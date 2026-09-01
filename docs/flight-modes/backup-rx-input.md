# Backup RX Input

Backup RX Input lets a spare UART carry a second, independent RX
receiver ("satellite") purely as a backup for the main RF link. If the main
receiver's signal is lost, the flight controller takes over all RC
channels -- including aux/mode switches, so arm state and flight-mode
switching keep working -- from the backup port instead, bypassing the
normal failsafe stages entirely. The moment the main link's signal comes
back, control reverts to it automatically. Takeover and revert happen
within the main receiver's own signal-loss detection window (up to
~100ms), not on a per-missed-frame basis -- see [Behavior](#behavior).

This is a cheap way to get a basic backup-receiver setup (e.g. a small
FrSky/compatible satellite bound to a second transmitter module or a
different protocol entirely) without needing a full second RF system.

The backup port speaks a single, configurable protocol, chosen from the
**Protocol** dropdown on the **Serial RX #2** box (see below): **SBUS**,
**FBUS**, **F.Port**, **F.Port2**, **Jeti EXBUS**, or **TBS CRSF**
(Crossfire/ELRS). The feature was originally SBUS-only
in name too ("SBUS-In Fallback Receiver"); it's now built so more protocols
can be added later without changing how it behaves or is configured, hence
the more general name. By default (before a protocol is chosen) the port
is reserved but not decoding anything.

## Setting it up

1. Wire an RX-capable satellite receiver to a spare UART, then assign
   that port the **Serial Rx (Backup)** function from the port function
   dropdown on the Configuration tab's serial ports list. It's named to
   pair with the main receiver's own **Serial Rx** option -- this is a
   second, independent one.
2. On the [Receiver](../configurator/tabs/receiver.md) tab, a **Serial
   RX #2** box appears below the main **Serial RX #1** box. Pick the
   satellite's protocol from its **Protocol** dropdown, then **Save &
   Reboot**.
3. Bind the satellite as you would any receiver of that protocol. If it
   isn't decoding (the box's **Link** badge stays down), check wiring
   against the UART's pinout -- the box's **Pin Swap** and **Inverted**
   switches cover boards where the port's natural RX pin isn't the one
   that's wired, or where the signal needs the opposite electrical
   inversion from the protocol's default assumption; **Half-Duplex** covers
   satellites that only expose a single, shared signal wire for this
   protocol. All three are independent of the main receiver's own
   equivalent settings, since this is a different physical UART, and each
   protocol has its own natural wiring convention (SBUS is inverted by
   default; every other supported protocol isn't) -- `OFF` always means
   "this protocol's normal wiring," not "no inversion" in an absolute
   sense. Changing any of these needs **Save & Reboot** to take effect.
4. Once bound and decoding, the **Serial RX #2** box's **Link** and
   **Active Source** badges update live, and each entry in
   [Channel Assignment](../configurator/tabs/receiver.md#channel-assignment)
   grows a second, smaller meter showing the backup receiver's own live
   channel values -- refreshed several times a second, the same readout
   used for bench-testing below.

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
- **Full channel takeover.** All channels are taken from the backup port
  during fallback, not just roll/pitch/yaw/throttle -- aux switches
  (flight mode, arming, etc.) are driven from the backup stick exactly
  as they would be from the main receiver.
- **Auto-revert.** As soon as the main receiver's signal is valid again
  (within that same ~100ms window), control reverts to it -- the backup
  does not latch for the rest of the flight.
- **Not a second failsafe system.** If *both* the main link and the
  backup link are down, ordinary staged failsafe behavior (hold/land/cut,
  per the [Failsafe](../configurator/tabs/failsafe.md) tab) takes over
  exactly as it would without this feature. Backup RX input is a bridge
  for a single lost link, not a replacement for failsafe.
- A **Failsafe** switch on an aux channel still invalidates control
  channels even while the backup link is otherwise healthy -- the switch
  is honored the same way regardless of which link is currently active.

## Bench-testing before you fly

As with any failsafe-adjacent behavior, test this on the bench (props off)
before relying on it in the air:

1. With both the main receiver and the backup satellite bound and
   powered, confirm the Receiver tab's **Serial RX #1** box shows its
   **Active Source** badge as Main RX, and moving the backup satellite's
   sticks has no effect on outputs.
2. Power off (or walk the main receiver's transmitter out of range),
   and confirm control switches to the backup satellite within roughly
   100ms, arm state is preserved, and aux switches on the backup radio
   work as expected.
3. Restore the main link and confirm control reverts to it within that
   same window.
4. Power off both receivers and confirm ordinary staged failsafe
   triggers as configured.
