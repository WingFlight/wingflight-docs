# Profiles

The Profiles tab manages PID tuning profiles -- the actual stabilization
gains that control how tightly the flight controller holds your commanded
attitude.

![Profiles tab](../../assets/images/configurator-profiles.png)

Multiple profiles can be configured and switched between in flight (via an
[Modes](auxiliary.md) mode mapping), which is useful for having distinct
tunes for, e.g., calm cruising versus aggressive 3D/aerobatic flight on the
same airframe.

## PID Gains

The PID Gains table holds the actual per-axis P, I, D, F (Feedforward), and B
(Boost) terms -- the static tune itself, as distinct from Master Gain below,
which scales it live rather than editing it directly.

Most people tune by feel in the air, not by reasoning about control theory,
so here's what each one is actually doing to the way the plane flies rather
than a textbook definition:

- **P** is how locked-in the plane feels to your stick and to gusts. Too
  little and it's floppy/mushy -- slow to respond, wanders off your line.
  Too much and it gets nervous: a fast buzz or bounce-back right after a
  sharp input.
- **I** is what holds a turn, a knife-edge, or a heading exactly, rather than
  approximately. Too little and it slowly bleeds off the line in wind or if
  the CG's slightly off. Too much and it wallows -- a slow wandering
  oscillation, or the plane keeps rotating a beat after you've already
  centered the stick.
- **D** is the shock absorber: it stops P's correction from overshooting and
  soaks up a gust before it upsets the plane. Too little and inputs
  overshoot, and gusts knock the plane around more than they should. Too
  much and servos start to growl, buzz, or run hot -- D is by far the most
  noise-sensitive term, so if you're chasing heat/buzzing here, check
  [Gyro](gyro.md) filtering first.
- **F** is what makes stick input feel instant instead of rubbery -- it
  pushes toward where you're asking the plane to go the moment you move the
  stick, rather than waiting for it to fall behind first. Too little feels
  delayed, and the plane keeps rotating briefly after you let go of the
  stick. Too much snaps past where you wanted and kicks back
  ("strikeback") as the stick returns to center.
- **B** is a finishing touch on top of F, reacting to how *fast* you move
  the stick rather than how far. Too little and quick flicks feel
  soft/rounded; too much and quick flicks get twitchy, sharing D's noise
  sensitivity.

### F and GYRO OFF mode

F also sets how far the surfaces move in
[GYRO OFF](../../flight-modes/setup-gyro-off.md) mode (stabilization off,
your rates and expo kept; called MANUAL in older firmware). GYRO OFF is
scaled through the same F-term the stabilized loop uses -- "stabilized
flight minus the gyro correction" -- so a well-tuned F puts its throw in
the neighborhood of what the stabilized modes settle to in flight. Judge the
match in the air: a stationary airframe on the bench never rotates, so
normal mode's P and I keep adding deflection there and the two never look
alike.

Full-stick throw in GYRO OFF is F × the full-stick rate. With the default F
of 75 and 250 deg/s rates that is about 47% of full travel. Lower F or the
rates and it shrinks, but never below 30% of full travel. Set your surface
throws in [SETUP](../../flight-modes/setup-gyro-off.md#setup-set-your-throws-here)
mode, where full stick is full travel, not in GYRO OFF.

F can't be set below **50** on roll, pitch or yaw. The minimum is for the
stabilized modes, not GYRO OFF: F sets most of the surface deflection for a
commanded rate, and P and I only correct what is left. I is capped (by
`error_limit`), so with too little F the sticks run out of authority. With
the default P and I and F at 0, full stick reaches only about 100 of a
commanded 250 deg/s on roll and pitch, and only after a delay. At 50, I can
still make up the rest. The minimum applies in the Configurator, the CLI,
the Lua scripts and in-flight adjustments, and a saved profile below 50 is
raised to 50 when the flight controller starts. Thrust Vector F gains have
no minimum.

### Quick troubleshooting

| What you're seeing in the air | Try |
|---|---|
| Floaty/mushy, wanders off your line | Raise P |
| Bounces or buzzes right after a sharp stick input | Lower P (or raise D) |
| Overshoots a gust or fast input before settling | Raise D |
| Servos growl, buzz, or run hot | Lower D, check [Gyro](gyro.md) filtering |
| Stick input feels laggy or rubbery | Raise F |
| Keeps drifting/rotating a moment after you center the stick | Lower I; if it's only on quick inputs, raise F instead |
| Feels too locked in, pushes back after a manoeuvre (3D) | Shorten [I-Term Decay](#i-term-decay) |
| Wanders with gusts, doesn't feel pinned | Lengthen [I-Term Decay](#i-term-decay) |
| Heading wanders in knife-edge or hover | Lengthen yaw [I-Term Decay](#i-term-decay) only |
| Bounces back at the end of a roll or loop | Raise [I-Term Relax](#i-term-relax) on that axis |
| Sustained rolls or loops lose rate or drift off line | Lower [I-Term Relax](#i-term-relax) on that axis |
| Snaps past the input and kicks back as you release the stick | Lower F |
| Quick flicks feel twitchy/nervous | Lower B |
| Quick flicks feel soft, no snap | Raise B |

Master Gain and its curve (below) scale P, I, and D together as one
percentage -- F (Feedforward) and Boost are tuned independently per-axis
and aren't affected by Gain.

### Why these defaults

Out of the box:

| | P | I | D | F | B |
|---|---|---|---|---|---|
| Roll | 50 | 16 | 0 | 75 | 35 |
| Pitch | 50 | 16 | 0 | 75 | 35 |
| Yaw | 80 | 20 | 0 | 75 | 35 |

The standout choice is **D = 0 on every axis**. A fixed-wing control
surface doesn't live in a particularly noisy, high-vibration environment,
so the default tune leans on Feedforward (F = 75 and B = 35 out of the
box) for a responsive, instant-feeling stick rather than
D-driven damping. Add D deliberately if a specific airframe overshoots or
wallows in gusts -- and check [Gyro](gyro.md) filtering first, since D is
the term most likely to expose noise once it's non-zero.

I is also deliberately kept low relative to P (16-20 vs. 50-80) -- don't
read that gap as I being "weak." I isn't left to accumulate freely the way
a raw integrator would: [I-Term Decay](#i-term-decay) (0.60 s by default) continuously
bleeds accumulated I-term error back off, capped by an I-Term Decay Max
Rate (35°/s), and [I-Term Relax](#i-term-relax) (5, level 22°/s by default)
specifically suppresses I buildup while the stick is moving quickly, to
avoid bounce-back at the end of a roll or other fast maneuver. Because
something else is actively managing decay, a small I gain is enough to
hold a steady bias (wind, a CG offset) -- pushing I up to "match" P instead
just reintroduces the slow wallowing oscillation the I-Term Decay and Relax defaults
are there to avoid.

Yaw runs a bit hotter than Roll/Pitch (P 80 vs. 50, I 20 vs. 16) since
rudder authority and yaw stability vary more from airframe to airframe than
aileron/elevator response typically does.

F and B are split 75 / 35 to cut bounce-back at the end of a roll or loop.
F gives a steady surface deflection for as long as the stick is held, while B
only acts while the stick is moving: it kicks the surface when the stick
starts to move and kicks it the other way when the stick comes back to
centre, which stops the rotation crisply. The lower F leaves less steady
throw that has to unwind after the stick is released. If the stick feels
sluggish, raise F; if starts and stops feel too sharp, lower B.

F also sets GYRO OFF mode's throw (see [Setup and Gyro Off](../../flight-modes/setup-gyro-off.md)), so
lowering it reduces GYRO OFF deflection as well.

Flight Feel's Gain defaults to 100% (no scaling) with no curve on every axis,
and the Throttle and Speed rows likewise default to 100% with no curve -- so the PID
Gains table above is exactly what flies until you deliberately assign a
curve or an Adjustments knob.

## Flight Feel

Flight Feel is where to start tuning. The columns use their real names, one
value per axis, and the guide to the right of the table says in plain
language what each one does and which way to turn it:

| Column | What it changes | Raise it if... | Lower it if... |
|---|---|---|---|
| **Master Gain** [%] | How hard the axis pushes back against a disturbance | It feels soft or wanders | It oscillates or buzzes |
| **I-Term Decay** [s] | How long the axis holds on to a correction after a gust | It doesn't feel pinned | It feels too locked in or pushes back after a manoeuvre |
| **I-Term Relax** (1-10) | How strongly a fast roll or loop is stopped from bouncing back | It bounces back when you centre the stick | Long, sustained rolls or loops lose rate |

The PID Gains table above is for detailed tuning; most pilots only need
Flight Feel. Every column is also an [Adjustments](adjustments.md) function,
so you can sweep it on a knob in the air -- a column under live adjustment
shows the value being commanded.

### Master Gain

Master Gain sets one overall gain per axis (Roll, Pitch, Yaw) that scales the P, I
and D terms of that axis together -- F (Feedforward) and Boost stay at their
configured values. A gain curve can optionally shape it by stick position,
so gain tapers in or out as the stick moves away from centre -- see
[Gain Curves](#gain-curves). A **CURVE** badge on Master Gain shows when one is
shaping that axis.

The **Throttle** row scales all three axes' P, D, F and Boost with throttle
instead: surfaces in prop wash gain authority as throttle rises, so it is
usually used to reduce gain at high throttle. Because F is included, it also
keeps the roll and pitch rate you get for a given stick the same at any
throttle, not just the damping. Its gain curve, if any, is evaluated on
throttle.

To tune it, first set F so the model rolls at the commanded rate at low
throttle. Then log a few rolls at high throttle and set the curve's
high-throttle point to low-throttle rate ÷ high-throttle rate. For example,
if the model rolls 1.7× faster at full throttle, set about 60%.

Throttle and Speed together never scale the gains below 25%, so no curve can
leave the surfaces without throw. GYRO OFF mode is not attenuated.

### Speed (GPS speed attenuation)

The **Speed** row scales P, D, F and Boost on all three axes with GPS speed, on top of
the Throttle row. Control surfaces get more effective as speed rises, so a
tune that is right at cruise can wobble in a fast dive -- often with the
throttle closed, where the Throttle row can't help. Speed reduces gain as
the model goes faster.

It needs a GPS with a fix, and it does nothing until a **Speed** curve is
assigned in [Gain Curves](#gain-curves). The row only appears with firmware
that supports it (MSP API 22.10 or later).

- **Gain** (25-200%, default 100%) scales the whole curve.
- The curve's X axis runs from 0 to the **speed curve range** (10-600 km/h,
  default 150), set in the Gain Curves panel. Faster than that uses the
  curve's last point.
- Speed is 3D speed when the GPS reports it (u-blox), so vertical dives
  count; otherwise ground speed. It is smoothed, so gain follows speed with
  a short delay rather than in steps.
- If the GPS fix is lost, gain holds for 3 seconds, then eases back to 100%
  over a couple of seconds. When Speed engages (first fix, fix regained,
  profile change) it also eases in rather than jumping.

GPS speed is speed over the ground, not airspeed: flying into a strong wind
reads slower than the air speed over the surfaces, and downwind reads
faster. Leave some margin in the curve.

A typical start: a curve that stays at 100% up to your cruise speed and
falls to 60-70% at your fastest dive speed, with the speed curve range set a
little above that dive speed. The Effective PID Gains preview shows the
current Throttle and Speed attenuation on the bench; to see it in flight,
log `debug_mode = GAIN_ATTEN` with [Blackbox](blackbox.md).

### Gain Curves

Gain curves are an advanced shaping tool, so they are assigned in the
**Gain Curves** panel, which appears in Expert Mode, rather than in Flight
Feel. Pick a curve slot for Roll, Pitch, Yaw, Throttle and Speed (plus the
Speed curve's range in km/h); the shapes
themselves are edited on the [Curves](curves.md) tab. "-" means no curve,
so Master Gain applies as set. Whenever a curve is assigned, the Master Gain cell in
Flight Feel shows a **CURVE** badge, with the curve number in its tooltip,
so the shaping is visible even outside Expert Mode.

### I-Term Decay

I-Term Decay is the other half of how "locked" each axis feels. Master Gain
sets how hard the axis pushes back against a disturbance; I-Term Decay sets
how long it holds on to that correction before letting it go.

Anything faster than the I-Term Decay time (a gust, a bump, a wobble) is fully
corrected, and the model returns to where it was. Anything slower (a slow
drift, a trim offset, the attitude you've just flown into) is let go, so the
model never pulls you back towards where you were a few seconds ago.

| I-Term Decay | Feel |
|---|---|
| 0.10-0.30 s | Free. Close to a plain rate gyro; suits 3D flying. |
| 0.40-0.60 s | Locked but flyable. 0.60 s is the default. |
| 0.70-1.00 s | Very locked. Starts to hold trim errors and can push back at the end of a manoeuvre. |

The range is 0.01-1.00 s in 0.01 s steps. Longer than that would feel like
attitude hold, which is what [Attitude Hold](../../flight-modes/atthold.md)
is for. Axes don't have to match. Yaw is the one most worth setting apart:
a longer yaw I-Term Decay holds rudder lock in knife-edge and hover, while a shorter
roll I-Term Decay keeps rolls free.

I-Term Decay and I gain overlap: under a steady load, a longer I-Term Decay holds more I,
much as a higher I gain would. Set I gain for how firmly a gust is
corrected, then use I-Term Decay for how long the correction lasts. The ANGLE
and ATT HOLD modes manage this themselves while they are
holding, so I-Term Decay mainly shapes normal rate flight.

In the CLI this is `iterm_decay_time`.
Its maximum bleed rate, I-Term Decay Max Rate (35°/s), stays under PID
Settings in Expert Mode; leave it at the default.

### I-Term Relax

While the stick is moving quickly, I-Term Relax stops the I-term
building up from your own input, so the model doesn't bounce back at the end
of a roll, loop or snap. I-Term Decay deals with what the model remembers after a
manoeuvre; I-Term Relax stops it collecting the manoeuvre in the
first place.

It is a score from 1 to 10, default 5. **Higher means more relax, so less bounce-back**;
lower keeps more hold through long, sustained rolls and loops. Most
airframes end up between 5 and 9. If the model bounces back at the end of a
manoeuvre, raise it on that axis a step at a time.

It is always on for roll, pitch and yaw. Technically it sets the I-term
relax filter cutoff (`bounceback` in the CLI), from 50 Hz at 1 to 3 Hz at 10,
with 5 = 10 Hz. The relax **level** (default 22°/s, lower = stronger) is
under PID Settings in Expert Mode; leave it at the default unless
I-Term Relax alone doesn't remove the bounce-back.

## Trainer (angle limits)

**Trainer (angle limits)** exposes Trainer gain and independent bank/pitch
limits with API 22.4 firmware (bank 10–90°, pitch 10–75°). Older firmware
shows a single shared limit. Save a TRAINER assignment in
[Modes](auxiliary.md) to show this panel. Expert Mode is not required.
See [Trainer Mode](../../flight-modes/trainer.md) for how the limits affect flight.

## Flight-mode settings

ANGLE, TRAINER and ATT HOLD each have their own panel.
These panels are available in both basic and Expert Mode, and appear only for
modes with a saved switch range or linked-mode assignment. Visibility follows
the configuration, not the current position of the transmitter switch.

ANGLE provides leveling gain, damping and independent bank/pitch limits.

- **Angle Mode leveling gain** sets how hard the model is driven toward the
  stick angle: degrees per second of rotation per degree of error, ×10.
- **Angle Mode damping** (0–100%, default 25) takes that percentage of the
  measured roll and pitch rate off the leveling command, so the model settles
  on the target angle with less overshoot. It acts through the rate PID's
  feedforward, so it follows the rest of the tune. Higher values level more
  slowly; 0 turns it off. It needs firmware with MSP API 22.13 or later.

Angle mode never commands more roll or pitch rate than your rate profile's
full-stick rate. When Angle mode, failsafe or a GPS mode takes over, the
target starts at the model's current attitude and moves toward the stick angle
at that same rate, so the model rolls level smoothly instead of snapping.

[Attitude Hold](../../flight-modes/atthold.md) has its own Gain, Deadband and
Max Rate. See each mode's page for what these settings do.
