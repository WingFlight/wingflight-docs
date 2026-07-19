# MSP Overview

MSP (MultiWii Serial Protocol) is the binary protocol the Configurator uses
to talk to WingFlight firmware over serial -- reading sensor data, reading
and writing configuration, and issuing commands.

On connection, the Configurator queries the firmware's API version, variant
identifier, and build/firmware version to confirm compatibility before
proceeding to the full configuration UI. If a connected firmware doesn't
identify as WingFlight, or reports a version outside the range this
Configurator build supports, the Configurator falls back to
[CLI-only mode](../getting-started/first-connection.md) rather than assuming
MSP compatibility it can't verify.

!!! note
    A full MSP command reference is planned as the project stabilizes --
    contributions welcome, see [Contributing](../contributing/index.md).
