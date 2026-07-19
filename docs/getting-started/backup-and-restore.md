# Backup & Restore

Back up your configuration before flashing new firmware, before major tuning
changes, and periodically once you have a working setup.

## From the Configurator

Configuration backups can be saved to and restored from a local file. Look
for backup/restore options in the CLI tab (a `dump`/`diff` snapshot) or via
the Configurator's dedicated backup functionality where available.

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
