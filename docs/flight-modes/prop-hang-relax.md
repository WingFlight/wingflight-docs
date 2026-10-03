# Prop-Hang Relax

In a prop hang the propeller's torque tries to roll the aircraft the opposite
way to the prop. On a 3D model that torque roll is part of the flying. A rate
gyro sees it as an unwanted roll and builds up roll I-term until the ailerons
cancel the torque, so with the gyro on the model hangs without rolling at all.

Prop-Hang Relax spots a prop hang and holds the roll I-term back, so the
torque can roll the model again. It is on by default.

## How it works

- **Detection.** A hang is all of these at once, for half a second:
  - the nose within the **Angle** of straight up,
  - the model neither climbing nor sinking faster than 2 m/s, which is what
    tells a hang from a vertical up-line,
  - plain rate flight: not Angle mode, Attitude Hold, Trainer, GPS Rescue,
    RTH, Loiter or failsafe, which all need the roll I-term to hold their
    target.
- **While hanging.** Roll I stops building up, and the roll I it was already
  holding drains away over about half a second, so the ailerons stop holding
  the model against the torque. P and the stick still work: you can stop or
  steer the torque roll, and a gust is still damped. Roll only: the rudder
  and elevator keep steering the nose.
- **Exit.** When the hang ends, the roll I-term comes back over the
  **Fade-out** time.

It needs an altitude estimate, which on most flight controllers means a
barometer. Without one an up-line can't be told from a hang, so nothing is
relaxed.

It works in every plain rate mode and needs no switch.

## Settings

Per PID profile. In the Configurator they are in
[Profiles](../configurator/tabs/profiles.md) → PID Settings (Expert Mode).
On the radio they are on PID Controller (Ethos) and Profile - Various
(EdgeTX).

| Setting | CLI | Range | Default |
|---|---|---|---|
| Strength | `prop_hang_strength` | 0-100 % | 100 |
| Angle | `prop_hang_angle` | 5-45° | 20 |
| Fade-out | `prop_hang_fade` | 0-2000 ms | 500 |

Strength 0 turns it off. Lower values only partly hold the roll I-term back,
for a slower torque roll.

## Tuning

- **The model still won't torque roll**: check the hang is being detected
  (see below). If it is, raise Strength towards 100.
- **It torque rolls in vertical up-lines or slow climbs**: lower the Angle,
  so only a hang close to vertical counts.
- **The roll I-term snaps back when you fly out of the hang**: lengthen the
  fade-out.
- **You want a locked, still hang instead**: set Strength to 0.

To see it working, log with `debug_mode = PROP_HANG` (see the
[CLI reference](../reference/cli-reference.md)).
