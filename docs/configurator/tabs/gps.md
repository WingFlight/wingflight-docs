# GPS

The GPS tab configures GPS module settings and shows live GPS status
(fix type, satellite count, position, a live map).

## When GPS is active

There is no GPS feature switch. GPS runs when the **Protocol** below
matches a port on the [Configuration](configuration.md) tab:

| Protocol | Port needed |
| --- | --- |
| NMEA or UBLOX | a port set to **GPS** |
| FBUS | a port set to **Bus: F.BUS Master** or **Bus: S.PORT Master** |
| CRSF | a port set to **Bus: CRSF Sensors** |
| MSP | none |

This tab appears as soon as any of those ports is set up, so you can pick
the matching protocol here. The [GPS Navigation](gps-navigation.md) tab,
GPS modes and GPS telemetry only appear once the protocol and port match.
Changes take effect after the save and reboot.

Firmware before MSP API 22.15 still has a GPS feature bit. The
Configurator sets it for you to match the protocol and port when you save.

## Module setup

**Protocol** selects how the GPS module talks to the flight controller:
NMEA or UBLOX over a serial port, MSP (a companion device relaying GPS over
MSP), FBUS (GPS position piggybacked on an FrSky FBUS link), or CRSF (GPS
position from a [CRSF Sensors](crsf-sensors.md) accessory) -- for the
latter two, the satellite signal-strength panel and baud/config options
below don't apply, since both carry an already-processed fix rather than
raw module output.

!!! warning "Only u-blox can navigate"
    [GPS RTH and Loiter](../../flight-modes/gps-rth.md) and the Failsafe
    tab's GPS Rescue only run on a **UBLOX** GPS. NMEA, FBUS and CRSF GPS
    still show position and feed telemetry, but navigation won't engage on
    them: they update too slowly or report too little about their fix to
    steer on. With one of those selected, this tab shows a warning. The
    radio dashboard does too if a GPS mode or GPS Rescue is set up. Arming
    isn't blocked.

    The GPS serial port speed matters too. A u-blox GPS on a 9600 baud
    port doesn't navigate either, because the port can't carry its
    output fast enough. Use 38400 baud or faster (the default is 57600)
    to get the full 10 Hz.

For NMEA/UBLOX modules, **Auto Baud** and **Auto Config** let the
Configurator detect the module's baud rate and push its own configuration
automatically -- leave both on unless you have a reason to manage the
module yourself. Auto Config sets a u-blox module to 10 Hz, or 5 Hz if the
module refuses 10 Hz or the port is slower than 38400 baud. With Auto
Config off the flight controller leaves the module's update rate alone
and doesn't check it, so set the module to 5 Hz or faster yourself if you
use GPS navigation. U-blox modules specifically also expose a **Ground
Assistance Type** (SBAS region -- EGNOS/WAAS/MSAS/GAGAN/auto/none) and a
**Galileo** toggle (swaps the QZSS constellation for Galileo, since a
module only tracks so many constellations at once).

**Home Point Once** controls when the home point is captured: off (the
default) resets it on every arm, so the point always matches your *most
recent* takeoff; on captures it only on the very first arm after
power-up, useful if you deliberately arm/disarm on the bench before walking
to your actual takeoff spot.

Wiring and configuring a u-blox or NMEA module by hand is covered in
[GPS Hardware and Module Setup](../../reference/gps-hardware.md). The
fixed-wing navigation modes that use the GPS are in
[GPS RTH and Loiter](../../flight-modes/gps-rth.md).

## Diagnostics

With a serial GPS (NMEA or UBLOX) and firmware that supports it, a
**Diagnostics** panel under Status shows the GPS port and its pins, whether
it opened and at what baud rate, the GPS state, and the byte, packet, error
and timeout counters. A badge in its header gives the state at a glance
(for example *Receiving*, *No data* or *Bad data*). When something is
wrong, a line says what to do: no port set to GPS, nothing received
(wiring or power), or data arriving but no valid messages (baud rate or
protocol). In Expert Mode the panel also shows the last bytes received
from the module. For the raw data and a baud rate scan, use the CLI command
`serialdebug`; see
[GPS troubleshooting](../../reference/gps-hardware.md#troubleshooting).
