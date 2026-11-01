# Backup RX Input

Backup RX Input lets spare UARTs carry up to two independent backup
receivers ("satellites") purely as fallbacks for the main RF link. If the
main receiver's signal is lost, the flight controller takes over all RC
channels -- including aux/mode switches, so arm state and flight-mode
switching keep working -- from the first backup that is receiving,
bypassing the normal failsafe stages entirely. The moment a higher-priority
link's signal comes back, control reverts to it automatically. Takeover and
revert happen within the main receiver's own signal-loss detection window
(up to ~100ms), not on a per-missed-frame basis -- see [Behavior](#behavior).

This is a cheap way to get a backup-receiver setup (e.g. a small
FrSky/compatible satellite bound to a second transmitter module or a
different protocol entirely) without needing a full second RF system. A
second backup gives three-way redundancy for models where a single lost
link is not an acceptable risk.

Each backup port speaks a single, configurable protocol, chosen from the
**Protocol** dropdown on its **Backup Receiver** box (see below): **SBUS**,
**FBUS** (also F.Port2 receivers), **F.Port**, **Jeti EXBUS**, or **TBS CRSF**
(Crossfire/ELRS). The two backups may use the same protocol or different
ones. By default (before a protocol is chosen) a port is reserved but not
decoding anything.

Flight-controller telemetry only ever travels over the main receiver. The
backup links are receive-only: nothing is sent to a backup receiver, whatever
its protocol.

## Priority

Failover is a fixed chain: **Primary**, then **Backup 1**, then **Backup 2**.
Whichever is highest in that order and currently receiving valid frames is
in control. There is no signal-quality comparison -- a backup's own
telemetry or RSSI is never read -- so the order you assign the ports is the
order they take over. Put the receiver you trust most as Backup 1.

## Setting it up

1. Wire an RX-capable satellite receiver to a spare UART, then assign
   that port the **Receiver: Backup 1** function from the port function
   dropdown on the Configuration tab's serial ports list. It's named to
   pair with the main receiver's own **Receiver: Primary** option -- this
   is a second, independent one. For a third receiver, wire it to another
   spare UART and assign that port **Receiver: Backup 2**.
2. On the [Receiver](../configurator/tabs/receiver.md) tab, a **Backup
   Receiver 1** box (and a **Backup Receiver 2** box, if that port is
   assigned) appears below the **Primary Receiver** box. Pick each
   satellite's protocol from its **Protocol** dropdown, then **Save &
   Reboot**.
3. Bind each satellite as you would any receiver of that protocol. If it
   isn't decoding (the box's **Link** badge stays down), check wiring
   against the UART's pinout -- the box's **Pin Swap** and **Inverted**
   switches cover boards where the port's natural RX pin isn't the one
   that's wired, or where the signal needs the opposite electrical
   inversion from the protocol's default assumption; **Half-Duplex** covers
   satellites that only expose a single, shared signal wire for this
   protocol. All three are independent of the main receiver's own
   equivalent settings and of the other backup's, since each is a different
   physical UART, and each protocol has its own natural wiring convention
   (SBUS is inverted by default; every other supported protocol isn't) --
   `OFF` always means "this protocol's normal wiring," not "no inversion" in
   an absolute sense. The box's **Detect** button tries every combination
   on the live port and reports the one that receives frames. Changing any
   of these needs **Save & Reboot** to take effect.
4. **Turn off telemetry on the backup receivers themselves.** These links
   are receive-only -- the flight controller never reads anything a backup
   receiver's own telemetry system might send, so this has no effect on
   the feature's flight-control behavior at all. It matters for a
   different reason: a second, independent telemetry stream is exactly
   what can confuse your radio's Lua scripts about which link is the
   real telemetry path back from the flight controller.
    - **FrSky/compatible satellites:** in the receiver's own RX options
      (on the transmitter, not the Configurator), turn telemetry off
      for this receiver.
    - **ExpressLRS receivers:** use the ELRS web configurator to
      disable telemetry on this receiver.
5. Once bound and decoding, each **Backup Receiver** box's **Link** and
   **Active Source** badges update live, and each entry in
   [Channel Assignment](../configurator/tabs/receiver.md#channel-assignment)
   grows a smaller meter per backup showing that receiver's own live
   channel value -- refreshed several times a second, the same readout
   used for bench-testing below.

All three receivers must use the same channel order: the backups are read
through the primary receiver's channel map.

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
  declares it failed. Between the two backups the hand-over is quicker: a
  backup counts as down after about 50ms without a valid frame, and Backup 2
  takes over the moment Backup 1 is down while it is still receiving.
- **Full channel takeover.** All channels are taken from the backup in
  control during fallback, not just roll/pitch/yaw/throttle -- aux switches
  (flight mode, arming, etc.) are driven from the backup stick exactly
  as they would be from the main receiver. Channels a backup doesn't
  carry go to stick center (1500 by default): for example CH19-24 when the
  main receiver sends 24 channels but the backup sends a 16-channel F.Bus
  frame. A mode or switch on those channels moves to its middle position on
  takeover, so keep safety-relevant switches on channels every receiver
  carries. F.Bus and F.Port2 backups carry up to 24 channels, SBUS and F.Port
  18, EX Bus and CRSF 16.
- **Auto-revert.** As soon as a higher-priority receiver's signal is valid
  again (the main receiver within that same ~100ms window, Backup 1 within
  ~50ms), control reverts to it -- a backup does not latch for the rest of
  the flight.
- **Not a second failsafe system.** If the main link *and* every backup
  link are down, ordinary staged failsafe behavior (hold/land/cut,
  per the [Failsafe](../configurator/tabs/failsafe.md) tab) takes over
  exactly as it would without this feature. Backup RX input is a bridge
  for lost links, not a replacement for failsafe.
- A **Failsafe** switch on an aux channel still invalidates control
  channels even while a backup link is otherwise healthy -- the switch
  is honored the same way regardless of which link is currently active.

## Arming check

Every configured backup receiver has to prove it works before you can fly
relying on it. Once the boot arming-grace window has passed, arming is
blocked -- the Status tab shows the **BACKUP_RX** arming-disable flag --
until each backup receiver that has a protocol selected has linked at least
once, so a mis-wired or dead backup can't go unnoticed until the moment you
need it.

The check only guards the **first arm after power-up**. Once you have armed
successfully, a backup that later drops out during the same powered session
will not block re-arming for the rest of that session; a battery change that
powers the flight controller off starts a new session and the check runs
again. A backup port left unconfigured (no protocol chosen) takes no part in
the check. A backup with a protocol chosen whose port fails to open counts as
configured but never linked, so it blocks the first arm until the wiring or
port conflict is fixed.

## Alerts on the radio

With the WingFlight Lua suite on an Ethos radio, the backup receivers' state
is also shown on the dashboard and announced:

- **BACKUP RX NO SIGNAL** (amber banner, "Backup receiver lost" /
  "Backup receiver OK"): at least one configured backup isn't receiving.
  Check this before takeoff. The **Rx Backup** diagnostics page shows
  which one.
- **BACKUP RX IN CONTROL** (red banner, "Backup receiver active" /
  "Main receiver restored"): a backup is flying the model.

Telemetry normally travels over the main link, so while the main link is
down the radio often can't hear the flight controller. See
[Radio Callouts and Alerts](../reference/radio-alerts.md) for when each
callout can reach you.

## Bench-testing before you fly

As with any failsafe-adjacent behavior, test this on the bench (props off)
before relying on it in the air:

1. Power up with the backup satellites bound and powered, and
   confirm the **BACKUP_RX** flag on the Status tab clears once every
   backup's **Link** badge comes up (arming stays blocked until then).
2. With every receiver bound and powered, confirm the Receiver tab's
   **Primary Receiver** box shows its **Active Source** badge as Main RX,
   and moving a backup satellite's sticks has no effect on outputs.
3. Power off (or walk the main receiver's transmitter out of range),
   and confirm control switches to Backup 1 within roughly 100ms, arm
   state is preserved, and aux switches on the backup radio work as
   expected. The **Active Source** badges name the backup in control.
4. With a second backup fitted, power off Backup 1 as well and confirm
   control moves to Backup 2; power Backup 1 back on and confirm it takes
   control back.
5. Restore the main link and confirm control reverts to it within that
   same window.
6. Power off every receiver and confirm ordinary staged failsafe
   triggers as configured.

## CLI settings

Backup 1 uses `rx_input_backup_provider`, `rx_input_backup_inverted`,
`rx_input_backup_halfduplex` and `rx_input_backup_pinswap`. Backup 2 uses
`rx_input_backup2_provider`, `rx_input_backup2_inverted`,
`rx_input_backup2_halfduplex` and `rx_input_backup2_pinswap`. The `serial`
command takes the port functions as numbers, 8388608 for Backup 1 and
16777216 for Backup 2; `serialdebug` shows them as `RX_BACKUP` and
`RX_BACKUP2`. See [Serial Ports](../reference/serial-ports.md) for the
details.
