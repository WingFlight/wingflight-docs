# Motors

The Motors tab lets you test motor outputs individually (props off!) and
configure motor-related output settings such as protocol and idle behavior.
It also hosts the [Governor](../../flight-modes/governor.md) section, which
governs idle hold and RPM behavior separately from motor protocol/testing.
In RPM Idle/Max mode, **Idle Throttle Floor** is the minimum powered idle
output the governor is allowed to command below handover; in Throttle Idle
mode, the same field is the fixed idle output.

!!! danger
    Always remove propellers before testing motor outputs from this tab.

## Protocol

Older ESCs expect plain PWM; most modern ESCs support a digital protocol
like DShot, which is more precise and doesn't need the manual PWM-endpoint
calibration below. Check what your specific ESC supports before switching.

For non-DShot ESCs, **Unsynced PWM** decouples the ESC update rate from
the PID loop, running it instead at a fixed frequency you set (typically
50-250Hz -- most modern analog ESCs are happy at 250Hz).

## Throttle endpoints (PWM-only)

For PWM/analog protocols, three raw PWM values calibrate the throttle
range: **Minimum Command** (sent when the motor is stopped -- also what
lets the ESC arm), **Throttle Minimum** (idle), and **Throttle Maximum**
(full throttle). Get these wrong and a motor can spin even while disarmed,
so it's worth using **Realtime** mode (applies changes immediately) paired
with **Throttle Override** at 0%/1%/100% to directly verify Motor
Off/Low/High before trusting the values -- rather than guessing and
arming to find out.

## RPM source

Both the [Governor](../../flight-modes/governor.md)'s RPM modes and the
[Gyro](gyro.md) tab's RPM Filter need a real-time RPM reading -- from
bidirectional DShot telemetry, an ESC telemetry sensor, or a dedicated
frequency sensor. Configure whichever source your setup provides here so
it's available to both.
