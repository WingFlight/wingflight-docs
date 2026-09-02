# GPS

The GPS tab configures GPS module settings and shows live GPS status
(fix type, satellite count, position, a live map).

## Module setup

**Protocol** selects how the GPS module talks to the flight controller:
NMEA or UBLOX over a serial port, MSP (a companion device relaying GPS over
MSP), FBUS (GPS position piggybacked on an FrSky FBUS link), or CRSF (GPS
position from a [CRSF Sensors](crsf-sensors.md) accessory) -- for the
latter two, the satellite signal-strength panel and baud/config options
below don't apply, since both carry an already-processed fix rather than
raw module output.

For NMEA/UBLOX modules, **Auto Baud** and **Auto Config** let the
Configurator detect the module's baud rate and push its own configuration
automatically -- leave both on unless you have a reason to manage the
module yourself. U-blox modules specifically also expose a **Ground
Assistance Type** (SBAS region -- EGNOS/WAAS/MSAS/GAGAN/auto/none) and a
**Galileo** toggle (swaps the QZSS constellation for Galileo, since a
module only tracks so many constellations at once).

**Home Point Once** controls when the home point is captured: off (the
default) resets it on every arm, so the point always matches your *most
recent* takeoff; on captures it only on the very first arm after
power-up, useful if you deliberately arm/disarm on the bench before walking
to your actual takeoff spot.
