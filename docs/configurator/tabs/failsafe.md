# Failsafe

The Failsafe tab configures how the flight controller *detects* a lost
radio link and what to do with each channel once it has -- the front end
of failsafe. What the aircraft actually *does* next (glide to a landing or
cut the motor) is a firmware-level choice (`failsafe_procedure`) that isn't
exposed as a GUI control yet -- see below for how to check/change it from
the CLI.

Configure and bench-test failsafe behavior *before* flying -- turn off your
transmitter at a safe distance from the aircraft (props off) and confirm the
flight controller actually enters failsafe and behaves the way you expect.

## Pulse Width Limit

Sets the valid pulse-width range (default 885-2115μs, expert mode only) --
any channel reporting a value outside it is treated as failed and enters
failsafe individually, which is what actually feeds the per-channel
Channel Fallback behavior below and the overall link-loss detection.

## Channel Fallback

Per-channel behavior once that channel is judged failed:

- **Auto** drives it to a safe value automatically -- center for Roll,
  Pitch, Yaw; low for Throttle.
- **Hold** freezes it at its last known-good position.
- **Set** drives it to a specific value you choose.

## What happens next (CLI)

Once RX data has been invalid for `failsafe_delay` (default 15, in tenths
of a second -- 1.5s), the firmware runs whichever `failsafe_procedure` is
configured: **DROP** (cut output immediately -- the default) or
**AUTO-LAND** (attempt a controlled glide down). For AUTO-LAND,
`failsafe_off_delay` (default 10, 1s) then bounds how long the landing
phase runs before motors are finally cut. All of these are CLI settings
(`failsafe_procedure`, `failsafe_delay`, `failsafe_off_delay`) -- worth
checking and bench-testing even though there's no dedicated tab for them
yet.
