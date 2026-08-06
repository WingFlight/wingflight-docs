# FBUS/S.Port Sensor ID Reference

Background reference for the [FBUS/S.Port Sensors](../configurator/tabs/fbus-sensors.md)
tab -- what the Physical ID and App ID numbers it shows actually mean.
These IDs come from FrSky's S.Port sensor addressing scheme, which FBUS
reuses; they're shared across the wider FrSky/OpenTX/EdgeTX ecosystem, not
specific to WingFlight.

## Physical IDs

A sensor's Physical ID identifies *what kind of sensor it is* -- fixed by
the sensor's own firmware/hardware, not something you configure. Multiple
sensors of different types can share a bus; each announces itself with its
own Physical ID.

| ID (hex) | ID (dec) | Name | Notes |
|---|---|---|---|
| `0x00` | 0 | VARIO2 | Variometer (altitude/climb rate) |
| `0x01` | 1 | FLVSS | Per-cell LiPo voltage sensor |
| `0x02` | 2 | CURRENT | Current/voltage sensor |
| `0x03` | 3 | GPS | Position, altitude, speed, course, time |
| `0x04` | 4 | RPM | RPM sensor (also reports temperature) |
| `0x05` | 5 | SP2UART_A | S.Port-to-UART bridge (host side) |
| `0x06` | 6 | SP2UART_B | S.Port-to-UART bridge (remote side) |
| `0x07` | 7 | FAS_150S | High-current sensor |
| `0x08` | 8 | SBEC | Smart BEC voltage/current sensor |
| `0x09` | 9 | AIR_SPEED | Airspeed (pitot) sensor |
| `0x0A` | 10 | ESC | ESC telemetry (voltage, current, RPM, temperature, consumption) |
| `0x0B` | 11 | PSD_30 | Power sensor |
| `0x0C` | 12 | XACT_SERVO | Smart bus-servo telemetry (current, voltage, temperature) |
| `0x10` | 16 | SD1 | SD card logger module |
| `0x12` | 18 | VS600 | FrSky VS600 module |
| `0x16` | 22 | GASSUIT | Gas engine sensor suite (RPM, temperature, fuel) |
| `0x17` | 23 | FSD | FrSky FSD module |
| `0x18` | 24 | GATEWAY | FrSky Gateway |
| `0x19` | 25 | REDUNDANCY_BUS | Redundancy-receiver bus sensor |
| `0x1A` | 26 | S6R | FrSky S6R stabilization receiver telemetry |

An unrecognized Physical ID shows up in the sensor table as `ID_<n>` rather
than a name -- the sensor is still read and can still be forwarded, WingFlight
just doesn't have a friendly name for it.

## App IDs

Within a single sensor's data stream, App IDs identify the specific *data
type* of each value it reports -- e.g. a GPS sensor reports latitude,
altitude, speed, course, and time as separate App IDs, all under the same
Physical ID. Related values are typically grouped in a contiguous 16-value
range (`BASE` to `BASE + 0x0F`); the low nibble often carries a per-frame
sequence bit and isn't meaningful on its own.

| Base (hex) | Base (dec) | Sensor | Data |
|---|---|---|---|
| `0x0200` | 512 | CURRENT | Current, 0.1A steps |
| `0x0210` | 528 | CURRENT | Voltage, 0.01V steps |
| `0x0220` | 544 | CURRENT | Current, 0.001A steps (high precision) |
| `0x0300` | 768 | FLVSS | Packed per-cell voltages, 0.002V steps |
| `0x0400` | 1024 | RPM | Temperature 1 |
| `0x0410` | 1040 | RPM | Temperature 2 |
| `0x0500` | 1280 | RPM | RPM value |
| `0x0800` | 2048 | GPS | Latitude/longitude (bit 31 selects which) |
| `0x0820` | 2080 | GPS | Altitude |
| `0x0830` | 2096 | GPS | Ground speed |
| `0x0840` | 2112 | GPS | Course |
| `0x0850` | 2128 | GPS | UTC date/time |
| `0x0B50` | 2896 | ESC | Voltage/current |
| `0x0B60` | 2912 | ESC | RPM/consumption |
| `0x0B70` | 2928 | ESC | Temperature |
| `0x6800` | 26624 | XACT_SERVO | Current/voltage/temperature |

For example, a GPS sensor's App IDs column showing `2048, 2080, 2096, 2128`
is lat/long, altitude, speed, and time -- everything except course, which
that particular GPS unit isn't reporting.
