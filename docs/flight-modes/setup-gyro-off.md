# Passthrough and Manual

Two modes turn the stabilization off. They do different things, and only
one of them is the right place to set up your surface throws.

| Mode | Stick to surface | Full stick gives |
|---|---|---|
| **PASSTHROUGH** | Raw stick, no rates, no expo, no gyro | Full servo travel |
| **MANUAL** | Your rates and expo, no gyro | Less than full travel, set by F and the rates |

Both are assigned to a switch on the [Modes](../configurator/tabs/auxiliary.md)
tab. If both are on, PASSTHROUGH wins. If failsafe starts, it takes over from
both, whatever the switches say.

Firmware up to 0.0.32 called them SETUP and GYRO OFF. The box IDs are the
same, so existing switch assignments keep working. SETUP now names the
[setup state](#the-setup-state) instead.

## PASSTHROUGH: set your throws here

In PASSTHROUGH, full stick is full servo travel. That is the same limit
every other mode is held to: the stabilized modes can use up to this much
deflection and never more. So when a setup guide says "elevator 38° each
way", this is the mode to measure it in.

1. Remove the propeller, arm or enable servo output as you normally would on
   the bench, and switch to PASSTHROUGH.
2. Move each stick to full deflection and measure the surface.
3. Adjust the servo's **Scale Neg** / **Scale Pos** on the
   [Servos](../configurator/tabs/servos.md) tab until full stick gives the
   throw you want. Use **Min** / **Max** only to keep the servo off a
   mechanical stop.

The Configurator's setup wizard switches PASSTHROUGH on by itself for the
steps that measure throws, so you don't need a switch for it there.

PASSTHROUGH also works in flight as a raw bail-out: the sticks drive the
surfaces directly, with nothing from the flight controller in between.

## MANUAL: stabilization off, same sticks

MANUAL keeps your rates and expo but drops the gyro correction. It flies
the feedforward (F) part of normal flight on its own:

- Full-stick throw is `F × full-stick rate`. With the default F (75) and
  rates (250 °/s roll and pitch, 350 °/s yaw), that is about 47% of full
  travel on roll and pitch and 66% on yaw.
- It never drops below 30% of full travel at full stick, whatever F, the
  rates or an in-flight adjustment are set to.
- Throttle and GPS speed attenuation do not reduce it.

Because its throw follows F and the rates, don't use MANUAL to measure or
set throws.

### Why MANUAL looks smaller than normal mode on the bench

On the bench, normal mode moves the surfaces further than MANUAL, and
the gap grows while you hold the stick. Normal mode adds P and I to F, and
a model sitting still never rotates at the commanded rate, so P keeps
pushing and I keeps building. With the default tune, roll and pitch go from
about 0.55 to about 0.70 of full travel within a fraction of a second, and
yaw goes straight to full travel.

In flight the aircraft does rotate, P and I fall away, and normal mode
settles close to F × rate. That is what MANUAL gives, so the two feel
alike in the air. If normal mode still needs much more deflection than
MANUAL to reach the commanded rate in flight, F is too low for that
aircraft. See [Profiles](../configurator/tabs/profiles.md#f-and-manual-mode).

The I-term is not built up while PASSTHROUGH or MANUAL is active, so
switching back to a stabilized mode doesn't kick the aircraft.

## The setup state

SETUP is not a mode you put on a switch. It is the state the flight
controller is in while a setup tool is holding the model on the bench: the
Configurator's setup wizard (for as long as it is open), or a servo, mixer
or motor test override. While it lasts:

- Arming is blocked. The Status tab and CLI `status` show the **SETUP**
  arming-disable flag.
- The radio shows **SETUP** as the flight mode and says "Setup" once. It
  doesn't call out the modes the tool switches on, such as PASSTHROUGH for
  throws or ANGLE for the gyro check. See
  [Radio Alerts](../reference/radio-alerts.md).
- Nothing the tool holds is saved. If the Configurator is closed or the
  cable is pulled, the flight controller drops it within a few seconds.
