# Tune Advisor Internals

How the Tune Advisor works, end to end: what the flight controller measures,
how the radio stores and combines it, the exact rules that produce the advice,
and exactly which parameters **Save** writes. For using it, see
[Tune Advisor](../../ethos-suite/tune-advisor.md).

The work is split in two:

| Side | Does | Source |
| --- | --- | --- |
| Flight controller | Measures how the model answers the rate loop and reports the numbers over MSP. Gives no advice and changes no settings. | `src/main/flight/tune_advisor.c`, `MSP2_WING_TUNE_ADVISOR` in `src/main/msp/msp.c` (wingflight-firmware) |
| Radio (Ethos Lua suite) | Reads the numbers after each flight, keeps the last 5 flights, works out the advice, and writes it to the flight controller when the pilot presses **Save**. | `tasks/tune_history.lua`, `lib/tune_history.lua`, `lib/msp_tune_advisor.lua`, `app/pages/tune_advisor.lua` (wingflight-lua-ethos-suite) |

The advice rules live on the radio so they can change without flashing the
flight controller.

## Firmware: what is measured

The feature is built when `USE_TUNE_ADVISOR` is defined, which
`common_pre.h` does on boards with more than 128 KB of flash.
`tuneAdvisorUpdate()` runs every PID loop, after the PID controller.

### Sampling

PID loops are averaged into one **sample every 10 ms** (100 Hz). Each sample
holds, per axis (roll, pitch, yaw):

| Value | Taken from |
| --- | --- |
| Setpoint (deg/s) | The PID controller's setpoint |
| Gyro (deg/s) | The PID controller's gyro rate |
| I-term | The PID controller's I output, in surface units |
| PID sum | The PID controller's full output, in surface units (1 = full travel) |
| Stick deflection | -1 to 1 |
| Saturated | Set if the mixer saturated that axis in any loop of the block |

plus the throttle (0 to 1).

### Which samples count

A sample is **valid** only if, in every PID loop of its block:

- the model is armed and airborne, and
- none of ANGLE, ATT HOLD, TRAINER, GPS rescue, LOITER, RTH, failsafe,
  PASSTHROUGH or MANUAL is active.

On top of that, the body must have been rotating: |roll| + |pitch| rate of at
least **40 deg/s** within the last **200 samples (2 s)**. Yaw is left out, so
taxi turns do not count as flying; on real wing logs they lowered the roll
correlation. This also keeps armed stick checks on the ground out.

Invalid samples are not used for anything below. The reported flight time is
the number of valid samples / 100, in seconds.

### Response (feed-forward match)

F sends the setpoint straight to the surfaces, so the ratio gyro / setpoint
says how well F fits the airframe: 1.0 means the model flies the rate it is
asked for; above 1 it turns faster than asked (F too high for the rates).

The model answers late (servos, aerodynamics), so the setpoint is compared
**delayed by 0 to 25 samples (0 to 250 ms)**. For each delay the firmware
keeps three running sums over usable pairs: Σ setpoint·gyro, Σ setpoint²,
Σ gyro². A pair is usable when:

- the current sample is valid and the axis is not saturated now, and
- the delayed setpoint came from a valid, unsaturated sample with
  40 ≤ |setpoint| < 200 deg/s.

From these, for each delay:

- **gain** = Σ setpoint·gyro / Σ setpoint² (least-squares slope through zero)
- **correlation** = Σ setpoint·gyro / √(Σ setpoint² · Σ gyro²)

Once a second the delay with the **highest correlation** becomes the best
lag. The reported `ffGain`, `ffCorr` and `ffCount` are those of the best lag,
and `ffLagMs` is the lag × 10 ms.

At the best lag the same ratio is also split into bands:

| Bands | Split by | Edges |
| --- | --- | --- |
| Setpoint bands | Size of the request, \|setpoint\| | 40–100, 100–200, 200+ deg/s |
| Throttle bands | Throttle | below 30%, 30–60%, 60% and above (only requests below 200 deg/s) |

Throttle matters because prop wash raises surface authority and TPA does not
scale F.

### Full stick

A sample counts as full stick when the stick deflection one best-lag ago was
at least **90%**. For those samples the firmware counts how many there were
(`fullCount`) and how many were saturated (`fullSatCount`), and keeps the rate
reached in the stick's direction against the rate asked:

- `fullRatio` = Σ rate reached / Σ |setpoint|
- `fullMaxRate` = the highest rate reached (deg/s)

Saturated samples are included here: this is what tells the radio the
surfaces ran out of travel.

### Stops (stick releases)

Per axis, a small state machine follows each stick input:

1. **Input**: starts when |setpoint| leaves the centre band (5 deg/s). While
   the stick is out, the setpoint and gyro extremes are kept.
2. **Release**: when the setpoint returns inside 5 deg/s. The direction of the
   input is the side with the larger setpoint peak. At that moment the I-term
   is recorded, signed so that positive means opposing the input that just
   ended.
3. **Watch**: for the next **80 samples (0.8 s)**, the largest rate in the
   opposite direction is the **rebound**, and the largest opposite PID sum is
   the **counter-surface**. The rate peak in the input direction keeps
   updating for the first 10 samples, as the model coasts.
4. The release counts only if the input asked for at least **60 deg/s** and the
   model reached at least **60 deg/s**. If the stick moves out again within 15
   samples (0.15 s), it is not a stop and is dropped; after that, a new input
   ends the watch early and the release still counts.

For each counted release the firmware adds up:

| Value | Meaning |
| --- | --- |
| `releases` | Number of counted releases |
| `bigRebounds` | Releases with a rebound of 15% of the peak rate or more |
| `meanRebound` | Mean of rebound / peak rate |
| `meanOvershoot` | Mean of peak rate / peak setpoint |
| `meanCounter` | Mean peak counter-surface (PID sum) after the release |
| `meanIterm` | Mean I-term at the release, positive = pushing back against the input |

A small counter-surface with the I-term near zero points at the airframe
rebounding on its own (not enough damping). A positive I-term points at the
I-term pushing the model back.

### Accumulation and reset

Everything accumulates in RAM across flights until it is reset. It is **not**
saved and is lost at power-off. It is reset when:

- `MSP2_WING_TUNE_ADVISOR_CLEAR` is received (the radio sends it after saving
  each flight), or
- the model arms on a **different tune**. At each arm the firmware hashes the
  current PID and rate profile numbers, every PID gain (P, I, D, F, B, O) of
  the PID profile, Master Gain, I-term Relax, PID mode, and the RC rates, expo
  and super rates of all axes. If the hash differs from the previous arm, the
  statistics start again.

The tune is only checked at arming. A setting changed in flight (for example
by an adjustment function) is not noticed until the next arm.

## MSP interface

| Command | ID | Direction | Payload |
| --- | --- | --- | --- |
| `MSP2_WING_TUNE_ADVISOR` | `0x5F18` | Read | Request: `U8` axis (0 roll, 1 pitch, 2 yaw). Any other length or axis is an error. |
| `MSP2_WING_TUNE_ADVISOR_CLEAR` | `0x5F19` | Write | Empty. Resets all statistics. |

One axis per request keeps the reply (65 bytes) small enough for MSP over
telemetry. Firmware without the feature answers with an MSP error.

Reply, payload version 1. Ratios are signed 16-bit × 1000; counts saturate at
65535; all values little-endian.

| Offset | Type | Field |
| --- | --- | --- |
| 0 | U8 | Payload version (1) |
| 1 | U8 | Collecting now (1 = the current sample is valid) |
| 2 | U16 | Seconds of valid flight |
| 4 | U8 | Axis |
| 5 | U16 | P gain (current PID profile) |
| 7 | U16 | F gain |
| 9 | U16 | B gain |
| 11 | U8 | I-term Relax score for this axis (1–10) |
| 12 | U8 | RC rate for this axis (raw; the Rates page shows raw × 5 deg/s) |
| 13 | U16 | `ffCount` |
| 15 | S16 | `ffGain` × 1000 |
| 17 | S16 | `ffCorr` × 1000 |
| 19 | U16 | `ffLagMs` |
| 21 | 3 × (S16, U16) | Setpoint bands 40–100, 100–200, 200+: gain × 1000, count |
| 33 | 3 × (S16, U16) | Throttle bands <30%, 30–60%, 60%+: gain × 1000, count |
| 45 | U16 | `fullCount` |
| 47 | U16 | `fullSatCount` |
| 49 | S16 | `fullRatio` × 1000 |
| 51 | U16 | `fullMaxRate` (deg/s) |
| 53 | U16 | `releases` |
| 55 | U16 | `bigRebounds` |
| 57 | S16 | `meanRebound` × 1000 |
| 59 | S16 | `meanOvershoot` × 1000 |
| 61 | S16 | `meanCounter` × 1000 |
| 63 | S16 | `meanIterm` × 1000 |

The tune values at offsets 5–12 are the settings the numbers belong to, so a
client can work out new values without reading the profiles first. The radio
refuses a reply of any other length or version rather than guess at it.

## Radio: capturing each flight

The background task captures every flight, whether or not the Tune Advisor
page is open.

On each **disarm**:

1. Wait **5 s** (the flight-stats sync goes first).
2. Read roll, then pitch, then yaw, with **2 s** between one answer and the
   next request. Over S.Port and FBUS each reply takes about 15 telemetry
   frames, so the gaps keep sensors updating.
3. If a read fails, ask again for that axis only after **5 s**; axes already
   read are kept. Each read is sent at most 3 times. After **3** link errors
   the capture stops until the next disarm. The flight controller still holds
   the numbers, so that disarm saves them. If the flight controller answers
   with an MSP error (firmware without the Tune Advisor), the capture stops
   until the next disarm.
4. If the three replies report different flight times, the flight controller
   restarted or flew between reads, and the capture starts again.
5. Save the three axes as one flight, unless the flight time is 0 (no valid
   flight) or the same as the last flight saved for this model (the previous
   clear never arrived and nothing new was flown).
6. Send `MSP2_WING_TUNE_ADVISOR_CLEAR`, so the next flight is measured on its
   own.

Arming during a capture abandons it. The flight controller was not cleared,
so on the same tune the next disarm saves both flights as one; on a changed
tune the firmware resets at arming and the earlier flight is lost. A disarm
during a link loss is captured on reconnect, as long as the flight controller
was not powered off.

The page itself asks the flight controller for one axis when it opens, only
to learn whether the firmware has the Tune Advisor; an MSP error shows
*Needs newer firmware*.

### Files on the radio

All under `LOGS:/wfsuite/tune/<mcuId>/`, where `<mcuId>` is the flight
controller's ID (the same one that names the flight log folders):

| File | Contents |
| --- | --- |
| `history.csv` | One row per axis per flight, newest last. Only the last **5 flights** are kept. Columns: `date`, `flight_seconds`, `axis`, then every reply field in the order above (`p`, `f`, `b`, `relax`, `rc_rate`, `ff_count`, …, `thr0_gain`, …, `mean_iterm`). A file with a different header is started again. |
| `logs.ini` | The model name. |
| `changes.csv` | Every value **Save** wrote: `date,axis,setting,from,to`, raw values as the flight controller stores them. Appended, never trimmed, never erased by the page. |

### Combining flights

For each axis separately, the page starts from the newest flight and goes
back up to 5 flights, stopping at the first flight whose **tune** on that axis
differs from the newest one. The tune is: P, F, B, Relax and RC rate for that
axis.

- Counts are added (`ffCount`, band counts, `fullCount`, `fullSatCount`,
  `releases`, `bigRebounds`, seconds).
- `ffGain`, `ffCorr` and `ffLagMs` are averaged weighted by `ffCount`; each
  band's gain by its count; `fullRatio` by `fullCount`; the four stop means by
  `releases`.
- `fullMaxRate` is the highest of the flights.

The firmware's gains are weighted by setpoint² within a flight, so the
combined response is a close estimate. The stop figures combine exactly.

Only those five values decide which flights are combined. A change to I, D,
Master Gain or expo resets the flight controller's measurements at the next
arm, but flights before and after it are still combined on the radio.

## Radio: the advice rules

All rules work per axis, on the combined numbers. At most **3 changes** and 3
reasons are shown; a change and its reason are added together or not at all.

### Thresholds

| Name | Value | Used for |
| --- | --- | --- |
| Response data needed | `ffCount` ≥ 1000 (10 s) | Judging the response at all |
| Consistent enough | `ffCorr` ≥ 0.85 | Judging the response at all |
| Too fast / too slow | `ffGain` > 1.15 / < 0.85 | F and RC Rate change |
| F step limit | ±20% per step | F change |
| Full stick data needed | `fullCount` ≥ 100 | Full-stick RC Rate change |
| Surfaces at their limit | `fullSatCount` ≥ 50% of `fullCount` | Full-stick RC Rate change |
| Rate not reached | `fullMaxRate` < 80% of the rate asked | Full-stick RC Rate change |
| Stops needed | `releases` ≥ 10 | Judging stops at all |
| Bounce too big | `meanRebound` ≥ 0.12 | Stop changes |
| I-term pushing back | `meanIterm` ≥ 0.03 | Relax change |
| Throttle fact | both outer bands ≥ 300 samples, spread ≥ 25% | Information only |

### Response

Checked in this order:

1. `ffCount` < 1000: **Needs more flying** (shown as a percentage of 1000).
   The page asks for rolls, pitch inputs or rudder inputs. No change.
2. `ffCorr` < 0.85: **Too uneven to judge**. No change. Common on rudder.
3. `ffGain` above 1.15 or below 0.85: change **F and RC Rate together**:
    - new F = round(F × clamp(1 / `ffGain`, 0.8, 1.2)), limited to 50–1000
      (50 is the firmware's lowest F: MANUAL flies on F)
    - new RC Rate = round(RC Rate × old F / new F), limited to 1–200

    F × rate is what sets how far the stick moves the surfaces, so that stays
    the same; the setpoint now matches what the model does. This applies to
    yaw too.
4. Otherwise the response matches. Then, on **roll only**, if the full-stick
   conditions above all hold, RC Rate is set to what the model reaches:
   new RC Rate = round(`fullMaxRate` / 5), limited to 1–200. The rate asked
   is RC Rate × 5 deg/s. Pitch is left out because 3D flying parks at full
   elevator on purpose.

### Stops

1. `releases` < 10: **Needs more stops**. No change.
2. `meanRebound` < 0.12: shown as the bounce-back percentage. No change.
3. `meanRebound` ≥ 0.12, first matching case:
    - I-term pushing back (`meanIterm` ≥ 0.03), Relax below 10, and room for a
      change: new Relax = Relax + 1. A higher Relax score lowers the I-term
      relax cutoff (scores 1–10 are 50, 30, 20, 15, 10, 8, 7, 6, 5 and 3 Hz),
      so the I-term builds less during the input.
    - The response is still off (step 3 above applies): reason only, "fix F
      first".
    - Otherwise, with room for a change: new P = round(P × 1.2), limited to
      (P + 1) … 1000. The page also mentions B as an alternative but never
      changes B.

F with RC Rate is two changes, so a Relax change can still be added next to
them.

### Throttle fact

If the response is judged and both the low and high throttle bands have at
least 300 samples, the larger band gain / smaller band gain is computed. At
1.25 or more the page adds a reason such as "It turns 30% faster at high
throttle than at low". This is information only and is the first reason
dropped when there is no room.

## Radio: what Save writes

**Save** writes the changes for the **axis on screen only**. These are the
only parameters it can ever change:

| Setting (as the page names it) | CLI parameter | MSP message, field | Accepted range |
| --- | --- | --- | --- |
| PIDs > *axis* > P | `roll_p_gain`, `pitch_p_gain`, `yaw_p_gain` | `MSP_SET_PID_TUNING` (202), `<axis>_p` | 0–1000 |
| PIDs > *axis* > F | `roll_f_gain`, `pitch_f_gain`, `yaw_f_gain` | `MSP_SET_PID_TUNING` (202), `<axis>_f` | 50–1000 |
| Flight Feel > *axis* > Relax | `iterm_relax` (one value per axis) | `MSP_SET_PID_PROFILE` (95), I-term Relax for the axis | 1–10 |
| Rates > *axis* > RC Rate | `roll_rc_rate`, `pitch_rc_rate`, `yaw_rc_rate` | `MSP_SET_RC_TUNING` (204), `rcRates_<1-3>` | 1–200 |

All of them belong to the **active** PID or rate profile. Save never changes
I, D, B, Master Gain, expo, super rate, filters or any other setting, and
never touches another axis or profile.

### The write sequence

1. **Confirm.** The dialog lists every change exactly as the page shows it.
2. **Read** `MSP_PID_TUNING` (112), `MSP_PID_PROFILE` (94) and
   `MSP_RC_TUNING` (111). A failed read, or a reply too short to hold every
   field, stops here with nothing changed.
3. **Check**, and stop with nothing changed if any of these fail:
    - the model is disarmed;
    - the active PID and rate profile are the same as when Save was pressed;
    - P, F, B, Relax and RC Rate on that axis are exactly the values of the
      newest flight the advice came from (so a hand edit, a profile switch or
      advice already applied stops it);
    - every value about to change still holds its expected old value, and
      every new value is in range.
4. **Write** only the messages that contain a change, in the order PID tuning,
   PID profile, RC tuning. Each is a full-message write: every other field is
   sent back exactly as it was read in step 2.
5. **Save** with `MSP_EEPROM_WRITE` (250).
6. **Verify**: read all three messages again and check each changed field
   holds its new value.
7. Append each change to `changes.csv`.

If a write fails, nothing has been saved to EEPROM, and restarting the flight
controller undoes any part already sent. If the EEPROM save fails, the
changes are active until the next restart.

The saved flights are kept. Because the tune changed, the next flight starts
a new combination on the page, and the firmware resets its statistics when
it next arms.

## Clearing

The page's **Tool** button deletes `history.csv` for the model and sends
`MSP2_WING_TUNE_ADVISOR_CLEAR`. `changes.csv` and `logs.ini` are kept.
