# Roll-Yaw Coupling

Most models yaw a little on their own when they roll, from aileron drag and
from the fin. In MANUAL you never notice: the nose does what it always does,
and your rudder habits already allow for it. A rate gyro sees that yaw as an
error and fights it with the rudder. The rudder then no longer just follows
your stick in a roll, and how much it fights depends on which way you roll.
Rolls stop tracking the way they do in MANUAL.

Roll-Yaw Coupling tells the yaw loop how much yaw the model makes by itself
when it rolls, so the gyro leaves that yaw alone. It is off by default.

## How it works

- Each loop, the yaw gyro's error has the configured share of the measured
  roll rate taken off. With Coupling at 20 and the model rolling at
  400 °/s, 80 °/s of yaw against the roll counts as normal and gets no
  correction.
- Only the gyro feedback on yaw (P and I) changes. The yaw stick still
  commands the same rudder, and any yaw beyond what the roll explains (a
  gust, a stall) is still corrected.
- It uses the roll rate the gyro measures, not the stick, so it also covers
  snaps and rolls that run faster than the stick asked for.
- It works in every mode that uses the gyro, and needs no switch.

## Settings

Per PID profile. In the Configurator it is in
[Profiles](../configurator/tabs/profiles.md) → PID Settings (Expert Mode).
On the radio it is on PID Controller (Ethos) and Profile - Various (EdgeTX).

| Setting | CLI | Range | Default |
|---|---|---|---|
| Coupling | `roll_yaw_coupling` | -100 to 100 % | 0 |

Positive values are yaw against the roll (roll right, nose yaws left), which
is the usual case. Negative values are for a model that yaws into the roll.
0 turns it off.

## Finding the value

The value is a property of the airframe, so measure it once from a log:

1. With a blackbox log running, fly a few fast axial rolls each way in MANUAL
   with the rudder centred.
2. In the log, compare the yaw gyro with the roll gyro during the rolls. The
   yaw rate divided by the roll rate, as a percentage, is the Coupling. If
   the yaw gyro has the opposite sign to the roll gyro, the value is
   positive.

As an example, one 3D model yawed at 23% of its roll rate in MANUAL, the
same with up, neutral or down elevator. In rate mode the yaw loop cancelled
about two thirds of it, with up to 22% rudder. That model wants a Coupling
of about 23.

Without a log, start at 15 to 20 and adjust:

- **The nose still wanders off the line in rolls, or the rudder feels
  different rolling left and right**: raise it.
- **The model now yaws against the roll more than in MANUAL**: lower it.
