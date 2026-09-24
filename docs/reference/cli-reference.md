# CLI Command Reference

The CLI provides direct, text-based access to the flight controller,
available from the [CLI tab](../configurator/tabs/cli.md) in the
Configurator, or over any serial terminal at the configured baud rate.

## Common commands

| Command | Description |
|---|---|
| `help` | List available commands |
| `status` | Show system status |
| `version` | Show firmware version info |
| `dump` | Print the full configuration (including defaults) |
| `dump all` | Print the full configuration for all profiles |
| `diff` | Print only settings that differ from defaults |
| `diff all` | Print changed settings across all profiles |
| `save` | Save changes and reboot |
| `exit` | Exit the CLI (without saving) |
| `defaults` | Reset to firmware defaults |

## Backup and restore

`diff all` is the preferred backup format for day-to-day use -- compact, and
easy to review. Paste the saved output back into the CLI and run `save` to
restore it. See [Backup & Restore](../getting-started/backup-and-restore.md)
for the full workflow.

## Settings added in recent snapshots

Auto Hover and Attitude Hold settings are per-profile (set them after
selecting the profile you want to change):

| Setting | Range | Default | Description |
|---|---|---|---|
| `autohover_gain` | 0-250 | 50 | [Auto Hover](../flight-modes/auto-hover.md) correction strength |
| `autohover_max_angle` | 0-90 | 30 | Max stick deflection off vertical, degrees |
| `autohover_max_rate` | 0-1800 | 120 | Attitude-capture rate clamp, deg/s |
| `autohover_roll_deadband` | 0-100 | 5 | Unused; roll is always a free pass-through (kept for compatibility) |
| `autohover_throttle_assist_gain` | 0-100 | 0 | Throttle added per second under sustained saturation, percent of range; 0 = off |
| `autohover_throttle_assist_max` | 0-50 | 15 | Ceiling on the added throttle, percent of range |
| `autohover_throttle_assist_trigger_ms` | 0-2000 | 300 | Milliseconds of saturation before the assist starts |
| `atthold_gain` | 0-250 | 40 | [Attitude Hold](../flight-modes/atthold.md) correction strength |
| `atthold_deadband` | 0-100 | 5 | Per-axis stick percent below which that axis is held |
| `atthold_max_rate` | 0-1800 | 300 | Correction rate clamp, deg/s |

Stick deadbands are global (not per-profile). `roll_deadband` and
`pitch_deadband` replace the old shared `deadband` setting, which a diff
from older firmware will now reject as an invalid name:

| Setting | Range | Default | Description |
|---|---|---|---|
| `roll_deadband` | 0-100 | 5 | Roll stick deadband around center, µs |
| `pitch_deadband` | 0-100 | 5 | Pitch stick deadband around center, µs |
| `yaw_deadband` | 0-100 | 5 | Yaw stick deadband around center, µs |

## Bus servo output

| Setting | Values | Default | Description |
|---|---|---|---|
| `fbus_master_channels` | `8`, `12`, `16`, `24` | `24` | Channels F.Bus output sends. 8 uses the 8-channel frame, 12 and 16 the 16-channel frame, 24 the 24-channel frame |
| `sbus_out_channels` | `8`, `12`, `16` | `16` | Channels SBUS output sends, always in the 16-channel frame |

Channels past the count are sent at center. Also on the Servos tab; see
[Servos → Bus output channel count](../configurator/tabs/servos.md#bus-output-channel-count).

## Mixer rules

The `mixer rule` command takes an optional last argument, the rule's
[role](../configurator/tabs/mixer.md#rule-roles): `0` for none (the default
when omitted), `1` for Flap Compensation, `2` for Differential Thrust Yaw.
The full argument order is:

```
mixer rule <index> <operator> <input> <output> <weight> <offset> <weight-neg> <speed> <curve> <condition> <role>
```

Everything after `<offset>` is optional: `<weight-neg>` defaults to the
symmetric value, and the rest default to off. `dump` and `diff` print rules
in this form.

`<output>` is a number. Motors stay at 27-30, so bus servos 19-24 come after
them:

| Output | Meaning |
|---|---|
| 0 | None |
| 1-8 | PWM servos 1-8 (S1-S8) |
| 9-26 | Bus servos 1-18 (S9-S26) |
| 27-30 | Motors 1-4 (M1-M4) |
| 31-36 | Bus servos 19-24 (S27-S32) |

`<input>` is a number too. RC channels 19-24 were added after the
thrust-vector inputs, so they are 30-35 rather than following CH18 (26):

| Input | Meaning |
|---|---|
| 13-26 | RC channels 5-18 |
| 27-29 | Thrust-vector roll, pitch, yaw |
| 30-35 | RC channels 19-24 |

!!! note
    This page covers general CLI usage. A full per-command and per-setting
    reference is planned as the project stabilizes -- contributions welcome,
    see [Contributing](../contributing/index.md).
