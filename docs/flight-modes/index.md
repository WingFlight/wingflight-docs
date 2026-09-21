# Flight Modes & Features

WingFlight's flight modes are built around fixed-wing flight characteristics
-- most were designed or adapted specifically for aircraft with fixed
control surfaces, rather than inherited unchanged from Rotorflight's
helicopter-focused mode set.

| Mode / Feature | Summary |
|---|---|
| [Auto Hover](auto-hover.md) | Automated hover/attitude assist for aircraft capable of hovering flight, with free roll and optional throttle assist |
| [Attitude Hold](atthold.md) | Holds commanded attitude per axis when that stick is released |
| [Thrust Vector Attitude Hold](tv-hold.md) | Independent attitude/heading hold on the Thrust Vector loop only, decoupled from the main surfaces |
| [Auto Trim](auto-trim.md) | Captures trim automatically from sustained stick input |
| [Trainer Mode](trainer.md) | Assisted flight mode for beginners |
| [Traditional](traditional.md) | Zeroes the I-term for a snappy, no-hold rate-gyro feel, layered on top of any other stabilization |
| [Ready-to-Arm Wiggle](ready-to-arm-wiggle.md) | Visual servo-wiggle confirmation before arming |
| [Cross-Axis Relax](cross-axis-relax.md) | Reduces unwanted coupling between control axes |
| [GPS RTH and Loiter](gps-rth.md) | Experimental fixed-wing return-to-home and orbit, by banking and pitching under Angle-mode leveling |
| [GPS Rescue](gps-rescue.md) | Inherited from multirotors. Does not steer or hold altitude on a wing, and can disarm in flight. Not recommended |
| [Governor](governor.md) | Idle-hold or RPM governing of motor throttle response, from Off through fixed idle to full RPM Range control |
| [Backup RX Input](backup-rx-input.md) | Instant backup receiver takeover from a second RX port if the main RF link is lost |

Most modes are enabled and mapped to a transmitter switch from the
[Auxiliary](../configurator/tabs/auxiliary.md) tab. Backup RX Input
is the exception -- it's enabled by assigning a serial port's
function, not an aux switch; see its own page for setup.
