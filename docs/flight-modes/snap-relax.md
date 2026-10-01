# Snap Relax

Pop tops, pinwheels and snaps all start the same way: roll, pitch and yaw
slammed in together. The wing stalls and autorotates, and the aircraft
rotates much faster than the stick asks for, often two to two and a half
times the commanded roll rate. A rate gyro sees that as an error and pushes
back. Without Snap Relax it can put opposite aileron in while you are still
holding full stick, and the I-term it builds up fighting the rotation comes
back as a bump when you centre the sticks.

Snap Relax spots that stick gesture and stops the gyro fighting the
rotation you asked for.

## How it works

- **Detection.** A snap is detected when the roll, pitch and yaw sticks all
  pass the **Stick threshold** within the **Entry window** of each other.
  A slower build-up of the same inputs, such as a rolling harrier, does not
  count.
- **While it is on.** On roll and pitch, P and D are reduced by **Strength**
  only where they push against the direction you snapped the stick in, and
  I stops building up against that direction. Anything that helps the
  rotation, the stick feedforward (F and B), and yaw stay as they are.
- **Exit.** When any of the three sticks drops below the threshold, full
  feedback comes back over the **Fade-out** time. Reversing the stick
  against the snap during the fade brings full feedback back at once, so
  the gyro helps you stop the rotation.

It works in every stabilized mode and needs no switch.

## Settings

Per PID profile. In the Configurator they are in
[Profiles](../configurator/tabs/profiles.md) → PID Settings (Expert Mode).
On the radio they are on PID Controller (Ethos) and Profile - Various
(EdgeTX).

| Setting | CLI | Range | Default |
|---|---|---|---|
| Strength | `snap_relax_strength` | 0-100 % | 100 |
| Stick threshold | `snap_relax_threshold` | 20-100 % | 60 |
| Entry window | `snap_relax_window` | 0-1000 ms | 400 |
| Fade-out | `snap_relax_hold` | 0-1000 ms | 150 |

Strength 0 turns it off.

## Tuning

- **The gyro still fights the entry**: lower the stick threshold or widen
  the entry window, so the snap is caught for the way you put the sticks in.
- **It triggers when you don't want it to**: raise the threshold or narrow
  the window.
- **The exit is sloppy, or the rotation carries on too long**: shorten the
  fade-out. Lengthen it if the gyro snatches as you release.

To see it working, log with `debug_mode = SNAP_RELAX` (see the
[CLI reference](../reference/cli-reference.md)).
