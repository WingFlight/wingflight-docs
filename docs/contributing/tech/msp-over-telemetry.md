# MSP over Telemetry

The Ethos suite and the EdgeTX scripts configure the flight controller through MSP carried on the
telemetry link: FrSky S.Port, F.Port, FBUS (F.Port2), or CRSF. This page follows one request from the
radio to the firmware and back, names the code that handles each hop, and shows what bounds the speed.
It is written for people working on the firmware side or on the Lua transports.

The FrSky transports are the slow ones and get most of the attention here. CRSF carries up to 57 MSP
bytes per frame and is covered only for comparison.

## The path

```
radio Lua  --push frame-->  RF module  --RF downlink-->  receiver  --bus-->  FC
radio Lua  <--pop frame--   RF module  <--RF uplink---   receiver  <--bus--  FC
```

| Hop | S.Port (telemetry port) | FBUS / F.Port2 (RX port) | F.Port |
| --- | --- | --- | --- |
| Request reaches the FC | receiver transmits it on the bus addressed to physical ID `0x0D` | downlink frame to ID `0x0D`, frame ID `0x30` | telemetry-request frame, frame ID `0x30` |
| FC may transmit | only when polled on ID `0x1B` | when polled on `0x1B` or `0x0D` | after each telemetry request |
| Reply frame | `0x1B`, frame ID `0x32` | uplink written under `0x1B`, frame ID `0x32` | `0x00`, frame ID `0x32` |
| Firmware driver | `src/main/telemetry/smartport.c` | `src/main/rx/fbus.c` | `src/main/rx/fport.c` |

All three feed the same assembler, `src/main/telemetry/msp_shared.c`, which also serves CRSF.

## Frame format

A FrSky frame carries 6 bytes of MSP: a status byte and 5 bytes of message. The status byte is

| Bits | Meaning |
| --- | --- |
| 7 | error (replies only) |
| 6-5 | MSP version: `01` v1, `10` v2 |
| 4 | start of a message |
| 3-0 | sequence number, modulo 16 |

The message is the MSP frame without its `$M<` prefix and without a CRC, since the telemetry frame has
its own. The firmware accepts three layouts and answers in the one the request used:

| Version bits | Header | Used by |
| --- | --- | --- |
| `11` compact | command (16 bit), length (8 bit; `0xFF` then a 16-bit length for longer messages) | Ethos suite, EdgeTX scripts |
| `10` MSPv2 | flags, command (16 bit), length (16 bit) | older Wingflight Lua, Rotorflight Lua |
| `01` MSPv1 | length, command (8 bit), CRC byte at the end | older EdgeTX scripts, commands below 256 |

The compact header is 3 bytes, so the first frame carries 2 bytes of payload and every later frame
carries 5. The MSPv2 header is 5 bytes and fills the first frame on its own:

```
compact: frames = 1 + ceil((payload - 2) / 5)
MSPv2:   frames = 1 + ceil(payload / 5)
```

| Message | compact frames | MSPv2 frames |
| --- | --- | --- |
| read request, no payload | 1 | 1 |
| 11-byte reply | 3 | 4 |
| 12-byte write | 3 | 4 |
| 52-byte write (`MSP_TELEMETRY_CONFIG`) | 11 | 12 |
| 81-byte reply | 17 | 18 |
| 448-byte reply (mixer rule pool) | 91 | 91 |

A frame is lost, not retransmitted, when the sequence breaks. The firmware drops the rest of a request
whose chunk went missing and answers nothing, and the radio drops the rest of a reply.

## Firmware

### Assembly and chunking

`msp_shared.c` keeps one request buffer (192 bytes) and one reply buffer (640 bytes). A request bigger
than the buffer is answered with an error; a reply that overflows the buffer is replaced by an error
reply rather than sent truncated. The reply is handed back one chunk at a time by `sendMspReply()`.

A request that arrives while a reply is still going out replaces that reply, and the new reply starts
with its own START frame. The radio relies on this: it resends as soon as it sees a sequence break
instead of waiting for the old reply to drain.

After `MSP_EEPROM_WRITE` the FC answers the next 5 polls with empty frames before sending the reply, to
keep the bus quiet while flash is written.

### S.Port on the telemetry port

`handleSmartPortTelemetry()` runs from the telemetry task at 250 Hz. It reads the bytes the receiver
put on the bus, and `smartPortDataReceive()` recognises two things: a poll of `0x1B`, which makes the FC
clear to send, and a frame addressed to `0x0D`, which is a request chunk.

Each poll of `0x1B` sends one frame. A pending MSP reply chunk always wins over sensor data. After the
last chunk the FC sends no sensor data for 250 ms (`SMARTPORT_MSP_SILENCE_US`), which gives the
receiver's uplink buffer time to drain the reply before sensor frames compete for it.

The FC cannot use the `0x0D` slot on S.Port: the receiver transmits in that slot when it has a request
to deliver, and the FC has no way to know in advance whether it will.

### FBUS and F.Port2

The receiver streams control frames and follows each with a downlink frame that polls one physical ID.
`fbus.c` queues frames from the UART interrupt and `frameStatus()` sorts them. A downlink to `0x0D`
with frame ID `0x30` carries a request chunk; a downlink with frame ID `0x10` asks for data; `0x00` is a
null poll that is answered with an empty frame.

The FC answers no earlier than 500 µs after the downlink frame and not at all after 3 ms. The RX task
is woken for it by `rxFrameCheck()` in the scheduler loop, so the answer normally goes out within the
first millisecond. The reply chunk is written under the common ID `0x1B` whichever ID was polled, which
is what both Lua transports filter on.

Data polls of `0x0D` carry pending reply chunks too, not only the polls that brought a request. Sensor
data is only ever sent on `0x1B`. How much this gains depends on how often the receiver polls `0x0D`
between requests, which the debug mode below measures.

### F.Port

`fport.c` answers telemetry-request frames on the control line. A request chunk arrives as a telemetry
request with frame ID `0x30`; data and null requests are answered with the next reply chunk or sensor
frame. The timing constants match FBUS.

## What bounds the speed

For a round trip of one read request and one reply:

1. The radio pushes the request frame. The Ethos suite pushes one frame per background wakeup and
   keeps a frame the module refuses for the next wakeup. The EdgeTX scripts ask the module whether it
   has room first.
2. The RF downlink carries the frame to the receiver. This is the slowest hop for writes and is set by
   the RF protocol, not by anything in this repository.
3. The receiver delivers it on the next poll of `0x0D`.
4. The FC answers the next poll of `0x1B` (or, on FBUS, `0x0D`) with the first chunk, and one more chunk
   per poll after that. On S.Port a poll slot is 12 ms, and the receiver cycles through the IDs it has
   seen answer, so each extra answering ID lengthens the gap between polls of `0x1B`.
5. The receiver uplinks each frame to the radio. The uplink has its own rate and a small buffer; when
   polls outrun it, frames are dropped and the radio sees a sequence break.
6. The radio pops the frames on its next wakeups and reassembles the reply.

Worked example on S.Port, FC the only sensor: a 40-byte reply is 9 frames at one per 12 ms, about
110 ms after the first chunk, plus the two RF hops and the wakeup latency at each end. A page that
reads ten such settings spends over a second in replies alone.

A lost frame costs more than its own slot. The Ethos suite resends on the next wakeup after a sequence
break, which costs one round trip, and only waits its full retry window (0.8 s) when nothing at all comes
back, which is what a lost request frame or a lost START frame looks like.

## Measuring

Set `debug_mode = FBUS` and read the debug fields in the Configurator sensors tab or a Blackbox log:

| Field | Value |
| --- | --- |
| 0 | microseconds between downlink polls of `0x1B` |
| 1 | microseconds between downlink polls of `0x0D` |
| 2 | frame ID of the last poll of `0x1B` |
| 3 | frame ID of the last poll of `0x0D` |
| 4 | MSP request chunks received |
| 5 | MSP reply chunks sent |
| 6 | frames rejected by the checksum or type checks |
| 7 | microseconds from the end of the downlink frame to the FC's uplink |

Fields 0 and 1 are the ceiling on reply throughput: 5 payload bytes per poll. Field 3 shows whether a
receiver polls `0x0D` with data requests between MSP requests, which is what lets the FC use that slot
for reply chunks.

On the radio, the Ethos suite's *Developer* → *MSP Speed* tool reports the round-trip time of a read
over a chosen period, and its MSP log (Developer settings) records every frame with a `SEQ` line for
each sequence break, including the gap since the previous wakeup, which separates frames lost on the
link from frames lost to a full queue on the radio.

## Known limits

- The link, not the bus, sets the rate. Measured on an FBUS receiver, the receiver polls `0x1B` and
  `0x0D` every 21 ms each but turns 30 to 40% of the polls into null polls while a reply streams,
  because its RF uplink to the radio moves about one frame per 30 ms. Answering more polls on the FC
  therefore gains nothing; fewer frames per message is the only lever on the firmware side.
- A request chunk lost on the way to the FC is never acknowledged; the radio only finds out when its
  retry window expires.
- A reply bigger than 640 bytes cannot be sent at all; the FC answers with an error.
- The 250 ms sensor silence after a reply stalls dashboard sensors while a page is loading.
