# SBUS-In Fallback Receiver

SBUS-In Fallback lets a spare UART carry a second, independent SBUS
receiver ("satellite") purely as a backup for the main RF link. If the main
receiver's signal is lost, the flight controller takes over all RC
channels -- including aux/mode switches, so arm state and flight-mode
switching keep working -- from the SBUS-in port instead, bypassing the
normal failsafe stages entirely. The moment the main link's signal comes
back, control reverts to it automatically. Takeover and revert happen
within the main receiver's own signal-loss detection window (up to
~100ms), not on a per-missed-frame basis -- see [Behavior](#behavior).

This is a cheap way to get a basic backup-receiver setup (e.g. a small
FrSky/compatible satellite bound to a second transmitter module or a
different protocol entirely) without needing a full second RF system.

## Setting it up

1. Wire an SBUS-capable satellite receiver to a spare UART, then assign
   that port the **SBUS Input (Satellite Rx)** function from the port
   function dropdown on the Configuration tab's serial ports list.
2. Bind the satellite as you would any SBUS receiver. If it isn't decoding
   (Link stays down on the status page below), check wiring against the
   UART's pinout -- `sbus_input_pinswap` and `sbus_input_inverted` (both
   CLI-only for now, default `OFF`) cover boards where the port's natural
   RX pin isn't the one that's wired, or where the signal needs the
   opposite electrical inversion from the default SBUS assumption. These
   are independent of the main receiver's own `serialrx_pinswap`/
   `serialrx_inverted`, since this is a different physical UART.
3. Once a port is assigned, a new **SBUS Input** page appears under
   Diagnostics in the sidebar -- confirm it's working there: live
   channel values and link-up state, refreshed several times a second.
   See the [SBUS Input](../configurator/tabs/sbus-input-status.md)
   page for details.

## Behavior

- **Bypasses staged failsafe, but isn't sub-frame instant.** This does not
  go through the normal [Failsafe](../configurator/tabs/failsafe.md) stage
  machine at all -- no `failsafe_delay`/`failsafe_recovery_delay` involved.
  It reacts to the same signal-presence detection the main receiver already
  uses for its own signal-loss tracking, which is bounded to roughly 100ms
  after the last good main-link frame (not "next frame"). That's a
  deliberate choice: a feature-specific faster threshold could false-trigger
  a takeover on a legitimately slower RX protocol's normal frame spacing.
  It's still far quicker than doing nothing here -- an un-caught channel
  would otherwise hold its last value for 300ms before failsafe even
  declares it failed.
- **Full channel takeover.** All channels are taken from the SBUS-in port
  during fallback, not just roll/pitch/yaw/throttle -- aux switches
  (flight mode, arming, etc.) are driven from the fallback stick exactly
  as they would be from the main receiver.
- **Auto-revert.** As soon as the main receiver's signal is valid again
  (within that same ~100ms window), control reverts to it -- fallback does
  not latch for the rest of the flight.
- **Not a second failsafe system.** If *both* the main link and the
  SBUS-in link are down, ordinary staged failsafe behavior (hold/land/cut,
  per the [Failsafe](../configurator/tabs/failsafe.md) tab) takes over
  exactly as it would without this feature. SBUS-in fallback is a bridge
  for a single lost link, not a replacement for failsafe.
- A **Failsafe** switch on an aux channel still invalidates control
  channels even while SBUS-in fallback is otherwise healthy -- the switch
  is honored the same way regardless of which link is currently active.

## Bench-testing before you fly

As with any failsafe-adjacent behavior, test this on the bench (props off)
before relying on it in the air:

1. With both the main receiver and the SBUS-in satellite bound and
   powered, confirm the SBUS Input page shows the main link
   active and moving the SBUS-in satellite's sticks has no effect on
   outputs.
2. Power off (or walk the main receiver's transmitter out of range),
   and confirm control switches to the SBUS-in satellite within roughly
   100ms, arm state is preserved, and aux switches on the fallback radio
   work as expected.
3. Restore the main link and confirm control reverts to it within that
   same window.
4. Power off both receivers and confirm ordinary staged failsafe
   triggers as configured.
