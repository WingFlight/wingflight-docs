# CRSF Sensors

CRSF Sensors is a dedicated listener for a CRSF-protocol sensor accessory
(GPS, battery, barometer, per-cell voltage, or RPM module) wired to its own
spare UART -- separate from the receiver link itself. It's for accessories
that speak CRSF telemetry frames independently, not for reading data off
your main RX link, and it's unrelated to [Backup RX
Input](../../flight-modes/backup-rx-input.md)'s own CRSF option, which is
about failing over the *control* link, not reading a sensor.

## Setting it up

1. Wire the sensor's data line to a spare UART, then assign that port the
   **CRSF Sensors** function from the port function dropdown on the
   Configuration tab's serial ports list. The port runs at CRSF's fixed
   baud rate -- there's nothing to pick.
2. If the sensor doesn't show data once wired, try flipping
   `set crsf_sensors_pinswap = OFF` (it defaults **on**, since real-world
   CRSF sensor accessories commonly wire up the opposite way round from
   this port's natural, un-swapped pin) -- then `save` and reboot.
3. Open the **CRSF Sensors** tab (Diagnostics group) to confirm it's
   actually receiving before wiring anything downstream to it -- see
   [The diagnostic page](#the-diagnostic-page) below.

## What it feeds, and how to enable it

Nothing downstream is enabled automatically just by wiring the sensor up --
each consumer needs its own source set explicitly:

- **GPS** -- set **GPS Protocol** to CRSF on the [GPS](gps.md) tab (and
  make sure the GPS feature itself is enabled). Position, ground speed,
  heading, and satellite count come from the sensor's GPS frames.
- **Battery voltage/current** -- set **Voltage Meter Type** and/or
  **Current Meter Type** to **CRSF Sensor** on the [Power](power.md) tab.
  See [Battery source](#battery-source) below for which frame type
  actually feeds the voltage reading when a sensor sends more than one
  kind.
- **Barometer altitude** -- `set crsf_sensors_use_baro = ON` (CLI-only,
  not yet on this tab). This *replaces* an already-detected physical
  barometer's altitude reading with the sensor's each cycle -- it doesn't
  create a virtual barometer if no physical baro chip is fitted on the
  board at all.
- **RPM** -- `set crsf_sensors_use_rpm = ON` (CLI-only). See [RPM](#rpm)
  below -- **this one comes with a real caveat, read it before wiring
  anything to it.**

## RPM

`crsf_sensors_use_rpm = ON` is a direct override, the same shape as
`crsf_sensors_use_baro`: each motor's normal RPM value (whatever it would
otherwise be -- DShot telemetry, a frequency sensor, none) is replaced with
the sensor's decoded reading for that index, feeding straight into
`motor1Speed`/`motor2Speed` (blackbox, OSD, this firmware's own outbound
CRSF telemetry), RPM Filter, and the Governor's RPM modes alike. There's no
separate protocol selection involved, and no ESC Telemetry setting on the
Motors tab to touch.

!!! warning
    This is telemetry-rate, not motor-rate. It is **not** a substitute for
    bidirectional DShot or a dedicated RPM/frequency-pin sensor, and
    **use at your own risk** applies if you turn this on while RPM Filter
    or the Governor's RPM Idle/Max or RPM Range modes are active -- see
    the [Gyro](gyro.md#which-filter-should-i-use) tab's and the
    [Governor](../../flight-modes/governor.md)'s own notes on the same
    limitation. It's genuinely fine for OSD, blackbox, or telemetry
    display, where the update rate doesn't matter.

Only voltage/current has a source-selection setting
(`crsf_sensors_battery_source`, below) -- RPM has nothing else to choose
between, since only one CRSF frame type carries it.

## Battery source

A sensor can report battery data as an aggregate frame (voltage, current,
capacity used, and remaining % all together) or as a set of individual
cell voltages with no current data -- some sensors only ever send one or
the other, and a few devices marketed as separate "current sensor" and
"cell voltage sensor" products exist specifically because of that split.
`crsf_sensors_battery_source` (CLI-only) picks which one feeds
**Voltage Meter Type = CRSF Sensor** on the Power tab:

| Value | Behavior |
|---|---|
| `AUTO` (default) | Prefer the aggregate frame; fall back to summed cell voltages only if the aggregate one isn't present. |
| `CURRENT` | Always use the aggregate frame, no fallback -- named for the field only it has, since the cell-voltage frame carries no current data at all. |
| `VOLTAGE` | Always use summed cell voltages, even if an aggregate frame is also present on the bus. |

Current always comes from the aggregate frame regardless of this setting
-- there's nowhere else for it to come from.

!!! note
    If an individual cell reads back an implausible value (tens of volts
    on a single tap -- almost always a bad connector/balance-lead
    connection on the sensor itself, not a wiring or firmware issue), that
    one channel is excluded from the pack total rather than corrupting it.
    The resulting total will read *low* by roughly that cell's worth until
    the connection is fixed, rather than showing a dangerously wrong high
    reading.

## The diagnostic page

The tab only appears once a port is assigned the CRSF Sensors function,
and polls twice a second:

- **Link** -- whether the port opened, plus raw byte/sync/CRC counters.
  Bytes-received-but-zero-sync-frames points at a baud/protocol mismatch
  on the wire; sync-frames-but-zero-CRC-OK points at signal integrity
  issues; both at zero with a powered, wired sensor points at the physical
  connection itself (wrong pin, no shared ground, or genuinely not a CRSF
  device).
- **GPS / Battery / Barometer / Cell Voltages / RPM** -- each shows a
  **Receiving** or **No data** badge and the latest decoded values for
  whichever frame types the sensor actually sends. A card reading "No
  data" just means that particular frame type has never arrived -- most
  sensor accessories only send one or two of the five types this feature
  understands.

Only five CRSF frame types are decoded: GPS (`0x02`), RPM (`0x0C`), the
aggregate Battery Sensor (`0x08`), Altitude/vario (`0x09`), and per-cell
Voltages (`0x0E`, per the [TBS CRSF
spec](https://github.com/tbs-fpv/tbs-crsf-spec/blob/main/crsf.md)).
Anything else on the wire is safely ignored.

## CLI settings

| Setting | Default | Meaning |
|---|---|---|
| `crsf_sensors_timeout_ms` | `2000` | How long without a fresh frame before that sensor's data is dropped as stale. Range 500-10000. CLI-only. |
| `crsf_sensors_use_baro` | `OFF` | Replace a physical barometer's altitude reading with the sensor's, see [above](#what-it-feeds-and-how-to-enable-it). CLI-only. |
| `crsf_sensors_pinswap` | `ON` | Swap which physical pin this UART uses for RX, see [Setting it up](#setting-it-up). CLI-only. |
| `crsf_sensors_battery_source` | `AUTO` | Which frame type feeds Voltage Meter Type = CRSF Sensor, see [Battery source](#battery-source). CLI-only. |
| `crsf_sensors_use_rpm` | `OFF` | Override each motor's RPM with the sensor's reading -- see [RPM](#rpm) for the important caveat before using this for filtering or governing. CLI-only. |

None of these are on the Configurator UI yet -- set them from the CLI, then
`save`.
