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

## Version compatibility

Additive changes, such as a new read-only command, don't change the API
version: firmware that lacks a command answers it as unsupported, which is
how the Configurator tells. `MSP_SERVO_TRIM`, which reports the live runtime
servo trim, works this way.

Changes to an existing message's layout are different, because an old client
would misparse the reply, or write shifted bytes into a profile. Those bump
the API version and the client picks the layout from it. Snapshot **0.0.25**
moved the API to **22.3**: the always-zero placeholder bytes inherited from
Rotorflight's helicopter code were removed from `MSP_RC_TUNING`,
`MSP_PID_PROFILE`, `MSP_PID_TUNING`, `MSP_SETPOINT`, `MSP_TELEMETRY_CONFIG`,
`MSP_ESC_SENSOR_CONFIG`, `MSP_RC_CONFIG` and `MSP_SENSOR_CONFIG`. Use the
Configurator and Lua suites from the same snapshot as the firmware. An older
client that doesn't know 22.3 falls back to CLI-only mode.

Commands for features that were removed (VTX, camera control and the
rangefinder) are gone too, so `MSP_SONAR_ALTITUDE` and the others now answer
as unsupported instead of returning a constant.

!!! note
    A full MSP command reference is planned as the project stabilizes --
    contributions welcome, see [Contributing](../contributing/index.md).
