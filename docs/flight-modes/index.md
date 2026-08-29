# Flight Modes & Features

WingFlight's flight modes are built around fixed-wing flight characteristics
-- most were designed or adapted specifically for aircraft with fixed
control surfaces, rather than inherited unchanged from Rotorflight's
helicopter-focused mode set.

| Mode / Feature | Summary |
|---|---|
| [Auto Hover](auto-hover.md) | Automated hover/attitude assist for aircraft capable of hovering flight |
| [Attitude Hold](atthold.md) | Holds commanded attitude, releasing stick input to level/hold |
| [TV Attitude / Heading Hold](tv-hold.md) | Independent attitude/heading hold on the Thrust Vector loop only, decoupled from the main surfaces |
| [Auto Trim](auto-trim.md) | Captures trim automatically from sustained stick input |
| [Trainer Mode](trainer.md) | Assisted flight mode for beginners |
| [Ready-to-Arm Wiggle](ready-to-arm-wiggle.md) | Visual servo-wiggle confirmation before arming |
| [Cross-Axis Relax](cross-axis-relax.md) | Reduces unwanted coupling between control axes |
| [Governor](governor.md) | Idle-hold or RPM governing of motor throttle response, from Off through fixed idle to full RPM Range control |
| [SBUS-In Fallback Receiver](sbus-input-fallback.md) | Instant backup receiver takeover from a second SBUS-in port if the main RF link is lost |

Most modes are enabled and mapped to a transmitter switch from the
[Auxiliary](../configurator/tabs/auxiliary.md) tab. SBUS-In Fallback
Receiver is the exception -- it's enabled by assigning a serial port's
function, not an aux switch; see its own page for setup.
