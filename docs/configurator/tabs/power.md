# Power

The Power tab configures battery voltage and current monitoring: cell count
detection/limits, voltage divider scaling, and current sensor scaling if
fitted.

Accurate voltage readings here feed the low-voltage warnings/beeper and any
battery-based failsafe behavior, so it's worth calibrating against a real
multimeter reading rather than trusting default scale values.

## Meter source

Voltage and current can each be read from **Battery ADC** (the board's own
onboard voltage divider / current shunt), **ESC Telemetry**, or a **FrSky
Sensor** -- or left **None** if not fitted. Only ADC sources need the Scale
and Divider fields below calibrated for your hardware; ESC telemetry and
FrSky sensors already report a computed value, so there's nothing to scale.
Changing the source itself needs a reboot to remap hardware pins -- the
rest of this tab applies live.

## Calibration Manager

Rather than computing a voltage divider or current-shunt scale by hand,
plug in a battery (props off!), measure its actual voltage/current with a
multimeter, and type those true values into the Calibration Manager --
it back-computes the corrected Scale for you from the difference between
what it's currently reading and what you measured. Only available for ADC
meter sources, and only meaningful if the Divider/Offset fields are already
roughly right first (leaving a value at 0 skips calibrating it).

## SmartFuel

A fuel-gauge charge estimate, more informative than raw voltage alone:
**Voltage** mode estimates remaining charge from pack voltage,
**Current** mode integrates measured mAh drawn against the configured
battery capacity, and **Combined** reports whichever of the two is more
pessimistic at any given moment (so an optimistic voltage reading under
load can't mask a pack that's actually further depleted). All modes except
Current need a working voltage sensor. Voltage Drop Rate and Charge Drop
Rate cap how fast the voltage-based estimate is allowed to fall, and Sag
Gain compensates for voltage sag under load -- raise it if SmartFuel reads
too pessimistic under load, lower it if too optimistic.
