# Receiver

The Receiver tab configures how the flight controller talks to your RC
receiver: protocol selection (serial RX protocols, PWM/PPM, RX-SPI, etc.),
channel mapping, and live channel monitoring.

## Protocol and signal options

Once a serial RX protocol is selected, three signal-level switches appear
that matter more than they look like they should -- get the wrong one and
you'll see no signal at all rather than a subtle problem:

- **Serial Inverted** un-inverts an inverted signal -- required for S.BUS
  and F.Port, among others. Not supported on F4-based boards.
- **Half Duplex** uses the Tx pin for both sending and receiving -- required
  for SBUS2, F.Port, SRXL, and SRXL2.
- **Pin Swap** swaps the Rx/Tx pins, for wiring that comes in reversed.

## SRXL2 full-size receivers

Some Spektrum full-size receivers (e.g. the AR6610T) need to see a valid
SRXL2 handshake within about 200ms of power-on, or they revert their
receiver port to plain PWM instead of SRXL2 -- which can be too fast for
the normal handshake to win the race during boot. When the FC's own SRXL2
unit ID is set to `0` (bus master), it now sends the first handshake
packets very early during boot, before this can happen, then continues
the normal handshake process as usual.

This is CLI-only -- set it with `set srxl2_unit_id = 0`. Note the
default is `1`, not `0`, so this needs setting explicitly for a receiver
that requires it.

## Channel Assignment

Different transmitter brands don't agree on which physical channel carries
which stick -- FrSky/ELRS/Futaba send `AETR` (Aileron, Elevator, Throttle,
Rudder) while Spektrum/JR/Graupner send `TAER` (Throttle first). The
**channel order preset** dropdown remaps for your brand in one click,
instead of manually reassigning each channel; the live bars next to each
channel let you confirm the remap actually matches real stick movement
before wiring up the [Mixer](mixer.md).

**RSSI** can be read from Auto (protocol-reported, if the link supports
it), the board's analog RSSI ADC input, or repurposed from any AUX channel
your receiver outputs an analog-ish RSSI value on.

## Telemetry Sensors

Selects which telemetry values are sent back to your transmitter/OSD, and
for protocols with a fixed transmission sequence (e.g. Smart Port), what
order they go out in -- data earlier in the list updates more often.
Available sensors depend on what the connected protocol and firmware build
actually support (e.g. RPM only appears with a configured RPM source).

## Virtual TX

For bench-testing without a real transmitter bound, a virtual RX input tool
is available (desktop app only) to simulate channel input.
