# Setup and Gyro Off

Two modes turn the stabilization off. They do different things, and only
one of them is the right place to set up your surface throws.

| Mode | Stick to surface | Full stick gives |
|---|---|---|
| **SETUP** | Raw stick, no rates, no expo, no gyro | Full servo travel |
| **GYRO OFF** | Your rates and expo, no gyro | Less than full travel, set by F and the rates |

Both are assigned to a switch on the [Modes](../configurator/tabs/auxiliary.md)
tab. If both are on, SETUP wins. If failsafe starts, it takes over from
both, whatever the switches say.

Firmware before this change called them PASSTHROUGH and MANUAL. The box IDs
are the same, so existing switch assignments keep working.

## SETUP: set your throws here

In SETUP, full stick is full servo travel. That is the same limit every
other mode is held to: the stabilized modes can use up to this much
deflection and never more. So when a setup guide says "elevator 38° each
way", this is the mode to measure it in.

1. Remove the propeller, arm or enable servo output as you normally would on
   the bench, and switch to SETUP.
2. Move each stick to full deflection and measure the surface.
3. Adjust the servo's **Scale Neg** / **Scale Pos** on the
   [Servos](../configurator/tabs/servos.md) tab until full stick gives the
   throw you want. Use **Min** / **Max** only to keep the servo off a
   mechanical stop.

SETUP also works in flight as a raw bail-out: the sticks drive the surfaces
directly, with nothing from the flight controller in between.

## GYRO OFF: stabilization off, same sticks

GYRO OFF keeps your rates and expo but drops the gyro correction. It flies
the feedforward (F) part of normal flight on its own:

- Full-stick throw is `F × full-stick rate`. With the default F (75) and
  rates (250 °/s roll and pitch, 350 °/s yaw), that is about 47% of full
  travel on roll and pitch and 66% on yaw.
- It never drops below 30% of full travel at full stick, whatever F, the
  rates or an in-flight adjustment are set to.
- Throttle and GPS speed attenuation do not reduce it.

Because its throw follows F and the rates, don't use GYRO OFF to measure or
set throws.

### Why GYRO OFF looks smaller than normal mode on the bench

On the bench, normal mode moves the surfaces further than GYRO OFF, and
the gap grows while you hold the stick. Normal mode adds P and I to F, and
a model sitting still never rotates at the commanded rate, so P keeps
pushing and I keeps building. With the default tune, roll and pitch go from
about 0.55 to about 0.70 of full travel within a fraction of a second, and
yaw goes straight to full travel.

In flight the aircraft does rotate, P and I fall away, and normal mode
settles close to F × rate. That is what GYRO OFF gives, so the two feel
alike in the air. If normal mode still needs much more deflection than GYRO
OFF to reach the commanded rate in flight, F is too low for that aircraft.
See [Profiles](../configurator/tabs/profiles.md#f-and-gyro-off-mode).

The I-term is not built up while SETUP or GYRO OFF is active, so switching
back to a stabilized mode doesn't kick the aircraft.
