# Failsafe

The Failsafe tab configures how the flight controller *detects* a lost
radio link and what value each channel takes once it has. That is all the
flight controller does on its own: it does not land, cut the motor or return
home by itself. See [What happens when the link is lost](#what-happens-when-the-link-is-lost),
and set up your receiver's failsafe as well.

Configure and bench-test failsafe behavior *before* flying -- turn off your
transmitter at a safe distance from the aircraft (props off) and confirm the
flight controller actually enters failsafe and behaves the way you expect.

!!! note
    If a [Backup RX Input](../../flight-modes/backup-rx-input.md)
    is configured, losing the main link doesn't reach any of the stages below
    on its own -- control switches to the backup receiver instead, within
    the main receiver's own ~100ms signal-loss detection window. The staged
    failsafe behavior described on this page only takes over if *both* the
    main link and the backup link are down.

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

## What happens when the link is lost

!!! warning "The flight-controller failsafe stages are disabled"
    WingFlight's inherited failsafe state machine (Betaflight's "stage 2":
    drop, auto-land, GPS Rescue) is switched off in the firmware. The
    settings `failsafe_procedure`, `failsafe_delay`, `failsafe_off_delay`,
    `failsafe_throttle_low_delay`, `failsafe_recovery_delay`,
    `failsafe_stick_threshold` and `failsafe_switch_mode` still exist in the
    CLI, but **they do nothing**. The flight controller does **not** disarm,
    land, cut the motor by itself, or return home when the link drops.

What the flight controller does do, on this tab's settings:

1. **Detects the loss.** No valid frame for about 100ms, a receiver failsafe
   flag, or a control channel outside the Pulse Width Limit counts as a bad
   signal.
2. **Holds for 300ms.** Each channel keeps its last good value.
3. **Applies Channel Fallback.** After 300ms each channel takes the value
   set above. With the defaults, roll, pitch and yaw go to center and
   throttle goes just below the off-throttle threshold, so the motor
   stops. Channels set to **Hold**, which is the default for everything
   after the four stick channels, keep their last value, so an arm switch
   or mode switch that was on stays on.
4. **Blocks arming** while there is no valid signal.
5. **Bypasses the [Governor](../../flight-modes/governor.md)**, so a
   governor switch held on cannot keep the motor running.

With roll, pitch and yaw at center, the surfaces go neutral, or the aircraft
levels if Angle, Horizon or another self-leveling mode is still selected by a
held switch. Nothing else happens.

## Setting up a safe failsafe

Because the flight controller will not act on its own, **the receiver's own
failsafe is what protects the aircraft**:

1. On your receiver or transmitter, set failsafe to throttle off (or idle for
   a glider) and a switch position that selects self-leveling (Angle).
2. Do this at the receiver, rather than relying on the flight controller
   fallback alone, because auxiliary channels default to Hold.
3. To make a mode switch fall back to a known position from the flight
   controller side, set that channel to **Set** with the value that selects
   the mode you want, or from the CLI:

   ```
   rxfail 4 s 1900
   ```

   `rxfail <channel> <a|h|s> [value]`, with the channel counted from 0. `a` is
   Auto (stick channels only), `h` is Hold, and `s` is Set.
4. Test it: turn the transmitter off with the props off, and check the
   surfaces and the motor. A receiver that keeps sending normal-looking
   frames when the link is lost cannot be detected by the flight controller.

[GPS RTH](../../flight-modes/gps-rth.md) can be mapped to a failsafe value the
same way. It is experimental, so test it thoroughly first. The GPS Rescue
mode does not work on a wing; see [GPS Rescue](../../flight-modes/gps-rescue.md).

## Failsafe switch

The **FAILSAFE** mode on the [Auxiliary](auxiliary.md) tab makes the flight
controller treat the four stick channels as invalid. After the 300ms hold
they take their Channel Fallback values, exactly as for a real signal loss.
Use it to test your fallback values, or as a panic switch.
