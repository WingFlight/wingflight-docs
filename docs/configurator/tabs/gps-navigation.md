# GPS Navigation

The GPS Navigation tab configures [GPS RTH and Loiter](../../flight-modes/gps-rth.md)
-- the fixed-wing navigation controller behind both, and behind the
Failsafe tab's [GPS Rescue](../../flight-modes/gps-rescue.md) procedure too.
It only appears once the **GPS** feature is enabled (Configuration tab).

These settings used to be CLI-only, then briefly lived on the Failsafe tab;
they moved here because they aren't failsafe-specific -- **GPS RTH** and
**GPS LOITER** (whether switched on by hand or started by the Failsafe
tab's GPS Rescue procedure) all use them.

## Arming without a GPS fix

When the Failsafe tab's GPS Rescue procedure or a **GPS RTH** switch is set
up, the first arm after power-up is blocked until the GPS has a fix (the
`GPS` arming disable flag). Re-arming later in the same power cycle doesn't
need one.

If you can't get a lock at the field, turn on **Allow arming without GPS
fix** (`gps_rescue_allow_arming_without_fix`, default OFF) at the top of this
tab. On the radio it is **Arm w/o GPS Fix** on the Lua suite's GPS
Navigation page.

The home point is only recorded at arming, with a fix and at least 5
satellites, so a flight armed without a fix has no home, even if the GPS
locks later:

- A GPS Rescue failsafe falls back to self-leveling with the motor off,
  like Land/Drop, instead of flying home.
- The **GPS RTH** switch flies level and doesn't turn toward home.
- **GPS LOITER** still works once a fix arrives, since it orbits where it
  was switched on.

Turn the setting off again once GPS is working, so the arming check
protects your next flights.

## Settings

| Field | Setting | Default | Meaning |
|---|---|---|---|
| RTH Altitude | `nav_rth_altitude` | 50m | Altitude RTH climbs or descends toward |
| Loiter Radius | `nav_loiter_radius` | 75m | Orbit radius for GPS LOITER |
| Loiter Direction | `nav_loiter_direction` | CW | Orbit direction, clockwise or counter-clockwise |
| Min Satellites | `nav_min_sats` | 6 | Minimum satellite count for navigation to run. Once navigating, one fewer is tolerated, and short dropouts are ridden through (see [GPS RTH and Loiter](../../flight-modes/gps-rth.md#how-they-work)) |
| Max Bank Angle | `nav_max_bank_angle` | 25° | Largest bank the navigation commands |
| Max Pitch Angle | `nav_max_pitch_angle` | 15° | Largest pitch the navigation commands |

Two more, under Expert:

| Field | Setting | Default | Meaning |
|---|---|---|---|
| Bearing Gain | `nav_bearing_kp` | 200 | Bank per degree of course error, percent |
| Altitude Gain | `nav_altitude_kp` | 100 | Pitch per metre of altitude error, percent |

See [GPS RTH and Loiter](../../flight-modes/gps-rth.md) for how these are
actually used, their defaults' practical effect, and known limits --
this tab is just where you set them. Home point capture (when RTH's "home"
is recorded) is a [GPS tab](gps.md) setting, not one of these.
