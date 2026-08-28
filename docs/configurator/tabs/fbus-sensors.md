# FBUS/S.Port Sensors

The FBUS/S.Port Sensors tab is a diagnostic and forwarding-control page for
boards acting as an **FBUS or S.Port master** -- reading telemetry directly
from third-party FrSky-ecosystem sensors (GPS, current sensors, RPM sensors,
smart servos, and more) wired to a spare UART, rather than talking to a
receiver on it. The tab only appears once a serial port is assigned the
`FBUS_OUT` or `SPORT_MASTER` function on the [Configuration](configuration.md)
tab.

## Reading sensors: direct use vs. forwarding

Sensors observed on the bus can be put to work two different (non-exclusive)
ways:

- **Direct use** -- the firmware reads certain sensor types itself and feeds
  them straight into existing systems. A FrSky current/voltage sensor can
  become a battery voltage/current source -- set **Battery Voltage/Current
  Source** to **FrSky Sensor** on the [Power](power.md) tab. A FrSky ESC
  sensor can become an ESC telemetry source (voltage, current, RPM,
  temperature, consumption); this one is currently CLI-only --
  `set esc_sensor_protocol = FBUS`. Nothing needs enabling on this page for
  direct use -- it happens automatically as soon as the sensor is observed.
- **Forwarding** -- relays a sensor's raw telemetry frames back out over the
  receiver link, so your transmitter/radio displays them directly. This
  matters most for sensor types the firmware doesn't consume itself (a
  vario or airspeed sensor, for example) -- forwarding is the only way to
  see those values on the radio. The **Forwarded** toggle on this page
  controls it.

## The sensor table

Each row is a sensor physical ID actually observed talking on the bus,
refreshed roughly once a second:

| Column | Meaning |
|---|---|
| Physical ID | The sensor's hardware-defined identity (e.g. GPS, FLVSS, ESC) -- see the [Sensor ID Reference](../../reference/fbus-sensor-ids.md) for the full list |
| Source | Which physical bus it was seen on -- FBUS or S.Port |
| Sensor Name | Resolved sensor name, or `ID_<n>` if the physical ID isn't recognized |
| App IDs | The specific data types seen from this sensor (e.g. altitude, speed) -- see the reference page for what the numbers mean |
| Packets | Running packet count since the last connect or **Clear** |
| Forwarded | Whether this sensor is currently relayed back to the receiver -- see [Forwarding](#forwarding) below |

**Clear** resets the observed-sensor table and packet counts -- useful after
rewiring, to confirm what's actually present now rather than what was seen
earlier in the session. It doesn't affect Forwarding configuration.

## Forwarding

Toggling **Forwarded** takes effect immediately -- no reboot needed -- and
persists across power cycles. Up to **8 sensors** can be forwarded at once;
toggling a 9th while all 8 slots are in use is refused, so turn one off
first.

The Forwarded toggle is only enabled when the receiver protocol itself is
FrSky-family -- **F.Port, F.Port2, or FBUS**, set on the
[Receiver](receiver.md) tab; otherwise it shows dimmed, and hovering it
explains why. It works by relaying frames over that protocol's own
telemetry return channel, which other protocols (CRSF, Ghost, and the rest)
don't have. Reading sensors as a bus master, and using them directly, both
work regardless of receiver protocol -- only forwarding to the radio needs
FrSky-family.

!!! note
    FBUS/S.Port master mode and the receiver's own RX protocol are
    independent: the master feature listens on its own dedicated UART, so
    it works alongside CRSF, ELRS, or any other receiver protocol. Only
    *forwarding* ties back to what the receiver link speaks.
