# Auto Gain

A gyro gain that is right at cruise can be too much somewhere else in the
flight: in a fast dive, or with the prop blowing hard over the tail. The
surfaces then have more authority than the tune was set for, and the axis
starts to oscillate: a fast wag of the wings, tail or nose that goes away
when you slow down.

Auto Gain watches each axis for that oscillation. When it sees it, it turns
the gyro gain of that axis down until the oscillation stops, then brings it
back up when the cause has gone. It works in every stabilized mode.

It is off by default. It is a safety net, not a replacement for a good tune:
if it is cutting the gain in normal flight, lower the gain, or use
[TPA or GPS speed attenuation](../configurator/tabs/profiles.md#speed-gps-speed-attenuation)
to take it down where the oscillation happens.

## How it works

- **Detection.** Auto Gain looks at the correction the gyro sends to the
  surface (P and D). An oscillating axis drives the surface from one side to
  the other at a steady rhythm. Four swings in a row, each larger than the
  **Threshold** and between 2 and 25 times a second, count as oscillation.
  One gust and its rebound do not, and neither does anything slower.
- **Cutting.** Every further swing takes P and D on that axis down by 15 %,
  as far as the **Minimum gain**. The cut stops as soon as the swings are
  smaller than the threshold. I, and the stick feedforward (F and B), are
  not changed, so trim and stick response stay the same.
- **Recovery.** Once there has been no cut for 2 seconds, the gain comes
  back at the **Recovery** rate. If the cause is still there, as in a long
  dive, it finds the limit again: a few swings, a small cut, and another
  hold.
- **Stick movement.** Swings while you are moving the stick quickly are
  something you asked for, so they are not counted.

Auto Gain does nothing before takeoff, or in Passthrough and Manual. It does
not cover the thrust-vector loop.

## Switching it on

There are two ways, and they can be used together:

- **AUTO GAIN mode.** Add the **AUTO GAIN** mode in the Configurator's
  [Modes](../configurator/tabs/auxiliary.md) tab, or on the radio's Modes page,
  and put it on a switch. While the switch is on, Auto Gain runs on roll,
  pitch and yaw. This is the easy way to try it: fly the same pass with the
  switch off and on.
- **Per axis, always on.** Set `auto_gain` in the
  [CLI](../reference/cli-reference.md) for the axes you want. Those axes
  run Auto Gain whether or not the mode is set up.

## Settings

Per PID profile. For now they are set in the
[CLI](../reference/cli-reference.md).

| Setting | CLI | Range | Default |
|---|---|---|---|
| Roll, pitch, yaw on/off | `auto_gain` | 0 or 1 for each axis | 0,0,0 |
| Minimum gain | `auto_gain_min` | 20-90 % | 40 |
| Threshold | `auto_gain_threshold` | 5-200, in 0.1 % of surface travel | 20 |
| Recovery | `auto_gain_recovery` | 1-100 % per second | 5 |

To leave it on for roll and pitch without a switch:

```
set auto_gain = 1,1,0
save
```

## Tuning

- **The oscillation carries on for a moment before it is caught**: lower
  the threshold, so smaller swings count.
- **The gain is cut in rough air when nothing is oscillating**: raise the
  threshold.
- **The oscillation keeps coming back in a long dive**: lower the recovery
  rate, so the gain stays down for longer.
- **The model feels loose for too long afterwards**: raise the recovery
  rate, or raise the minimum gain.

To see it working, log with `debug_mode = AUTO_GAIN` (see the
[CLI reference](../reference/cli-reference.md)).
