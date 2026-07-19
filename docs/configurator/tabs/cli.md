# CLI

The CLI tab gives direct, text-based access to the flight controller's
command-line interface -- the same interface documented in the
[CLI Command Reference](../../reference/cli-reference.md).

The CLI works independently of the normal MSP-based configuration protocol,
which makes it the fallback the Configurator switches to automatically when
it can't fully validate a connected firmware version (see
[First Connection](../../getting-started/first-connection.md)). It's also
the most reliable way to take a full backup (`dump all`) or a compact
changed-settings backup (`diff all`).
