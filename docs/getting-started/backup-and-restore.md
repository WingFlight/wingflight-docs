# Backup & Restore

Back up your configuration before flashing new firmware, before major tuning
changes, and periodically once you have a working setup.

## From the Configurator

The **Firmware Flasher** wizard has backup and restore built in, and this
is the easiest route around a firmware update:

- On the **Backup** step (before flashing) you can capture your settings
  from the connected board over the CLI. The backup type defaults to
  **Dump** (the complete configuration) rather than Diff. A Diff backup
  starts by replaying the board's defaults, which on current firmware can
  produce a spurious parse error in the CLI output, so a full dump is the
  more dependable choice. **Diff** is still selectable, and you can turn
  the backup off entirely.
- The backup is held in memory and offered for restore on the **Restore**
  step once the new firmware boots, so you don't have to do anything by
  hand. Saving a copy to a file is optional but recommended in case
  something goes wrong.
- If the board isn't currently running WingFlight, the backup is still
  captured and saveable, but it isn't offered for automatic restore --
  another firmware's settings aren't guaranteed to carry over.
- Backup and restore need a normal serial connection; a board flashed over
  DFU alone can't be backed up this way. If the connection drops after the
  board reboots mid-restore, the wizard retries and offers a **Select Port**
  option to reconnect.

You can also save a `dump`/`diff` snapshot to a file from the CLI tab at
any time.

## Via the CLI

The most portable backup is a CLI dump:

```
diff all
```

This prints only the settings that differ from firmware defaults --
compact, and easy to review by eye. To capture a complete configuration
regardless of defaults, use:

```
dump all
```

Save the output to a text file. To restore, paste the same commands back
into the CLI followed by `save`.

!!! tip
    A `diff all` backup is the most useful format for sharing a tune with
    someone else or asking for tuning help, since it only shows what you've
    actually changed.
