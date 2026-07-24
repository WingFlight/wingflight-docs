# Profiles

The Profiles tab manages PID tuning profiles -- the actual stabilization
gains that control how tightly the flight controller holds your commanded
attitude.

Multiple profiles can be configured and switched between in flight (via an
[Auxiliary](auxiliary.md) mode mapping), which is useful for having distinct
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

### Quick troubleshooting

| What you're seeing in the air | Try |
|---|---|
| Floaty/mushy, wanders off your line | Raise P |
| Bounces or buzzes right after a sharp stick input | Lower P (or raise D) |
| Overshoots a gust or fast input before settling | Raise D |
| Servos growl, buzz, or run hot | Lower D, check [Gyro](gyro.md) filtering |
| Stick input feels laggy or rubbery | Raise F |
| Keeps drifting/rotating a moment after you center the stick | Lower I; if it's only on quick inputs, raise F instead |
| Snaps past the input and kicks back as you release the stick | Lower F |
| Quick flicks feel twitchy/nervous | Lower B |
| Quick flicks feel soft, no snap | Raise B |

Master Gain and its curve (below) scale P, I, D, and F together as one
percentage -- Boost is tuned independently per-axis and isn't affected by
Master Gain.

### Why these defaults

Out of the box:

| | P | I | D | F | B |
|---|---|---|---|---|---|
| Roll | 50 | 16 | 0 | 100 | 0 |
| Pitch | 50 | 16 | 0 | 100 | 0 |
| Yaw | 80 | 20 | 0 | 100 | 0 |

The standout choice is **D = 0 on every axis**. A fixed-wing control
surface doesn't live in a particularly noisy, high-vibration environment,
so the default tune leans on Feedforward (already set to a meaningful
F = 100 out of the box) for a responsive, instant-feeling stick rather than
D-driven damping. Add D deliberately if a specific airframe overshoots or
wallows in gusts -- and check [Gyro](gyro.md) filtering first, since D is
the term most likely to expose noise once it's non-zero.

I is also deliberately kept low relative to P (16-20 vs. 50-80) -- don't
read that gap as I being "weak." I isn't left to accumulate freely the way
a raw integrator would: I-Term Decay Time (6s by default) continuously
bleeds accumulated I-term error back off, capped by an I-Term Decay Max
Rate (35°/s), and I-Term Relax (level 22, cutoff 10Hz by default)
specifically suppresses I buildup while the stick is moving quickly, to
avoid bounce-back at the end of a roll or other fast maneuver. Because
something else is actively managing decay, a small I gain is enough to
hold a steady bias (wind, a CG offset) -- pushing I up to "match" P instead
just reintroduces the slow wallowing oscillation the decay/relax defaults
are there to avoid.

Yaw runs a bit hotter than Roll/Pitch (P 80 vs. 50, I 20 vs. 16) since
rudder authority and yaw stability vary more from airframe to airframe than
aileron/elevator response typically does.

Boost defaults to 0 -- it's a finishing touch layered on an already-working
Feedforward tune, not something you'd want fighting an airframe that hasn't
been flown and trimmed yet.

Master Gain defaults to 100% (no scaling) with no curve assigned on every
axis, and Throttle (TPA) likewise defaults to 100% with no curve -- so the
PID Gains table above is exactly what flies until you deliberately assign a
curve or an Adjustments knob.

## Master Gain

Master Gain sets one overall gain per axis (Roll, Pitch, Yaw) that scales the
whole PID loop for that axis, each with its own optional gain curve so the
scaling can vary with stick position rather than applying a single flat
multiplier. In expert mode a fourth row, Throttle (TPA), attenuates gains
across the throttle range using the same shared gain-curve pool.

Any of these gains can also be mapped to a transmitter switch/knob from the
[Adjustments](adjustments.md) tab for live in-flight tuning -- when a gain is
under live adjustment control, its row shows the current effective value
being commanded in place of the static configured number.
