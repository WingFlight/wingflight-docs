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

!!! note
    This page covers general CLI usage. A full per-command and per-setting
    reference is planned as the project stabilizes -- contributions welcome,
    see [Contributing](../contributing/index.md).
