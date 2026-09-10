# Oscillation Limiter

The Oscillation Limiter watches each axis's tracking error -- the gap
between what you're commanding and what the gyro is actually reporting --
for a sustained, periodic oscillation in a narrow, low-frequency band
(roughly 4-20Hz by default). This is the band where too much
[Master Gain](../configurator/tabs/profiles.md) on a control surface, alone
or combined with a gain curve, can push the rate loop into a self-sustained
buzz that a hard maneuver or a gust doesn't -- the detector specifically
looks for a periodic, sustained pattern in the *error*, not just fast or
large stick/gyro activity, so an aggressive snap roll or rudder waggle on a
well-tuned axis doesn't trigger it.

Once a real oscillation is confirmed (it has to persist for a moment, not
just spike once), the affected axis's gain is eased down -- never up --
one small step at a time for as long as the oscillation is still present,
stopping wherever that is rather than always cutting all the way to the
floor. It never eases back up mid-flight: once the oscillation clears, the
reduced gain holds for the rest of the flight, and if the same axis
oscillates again later it backs off further from there, cutting harder
each time it recurs. Only a new arm restores full authority -- this is a
deliberate ratchet, not a live auto-tuner.

**This is a safety net, not a tuning tool.** It exists to keep a
gain-induced oscillation from damaging the airframe if one shows up
unexpectedly (a new setup, a control-horn change, a Master Gain sweep via
[Adjustments](../configurator/tabs/adjustments.md) that goes too far) --
not to let you run hotter gains than you'd otherwise tune to and rely on
the limiter to catch it. It's disabled by default and hasn't been
validated across every airframe yet.

## Knowing it's engaged

If you're running the [Ethos Lua Suite](https://github.com/WingFlight/wingflight-lua-ethos-suite),
a telemetry sensor and a spoken radio alert report an active gain cut for
as long as it's happening, so you don't have to notice it purely by feel
or catch it after the fact in a blackbox log.

## Tuning

Configured on the [Profiles](../configurator/tabs/profiles.md) tab, in the
expert-mode settings column, per PID profile:

- **Detection Band Low/High** set the band-pass frequency range the
  detector watches. Keep this below the range your
  [Gyro](../configurator/tabs/gyro.md) filtering already handles, and below
  normal trim/attitude-hold drift.
- **Energy Threshold** is how strong the band-passed error has to get before
  a candidate oscillation starts being scored. Lower catches milder
  oscillations sooner but risks false positives on a rough airframe --
  expect to tune this per aircraft.
- **Gain Floor** is the minimum Master Gain scale the limiter may reach on
  that axis. The axis never loses all authority, even on a false positive,
  but a genuine oscillation can still be cut by up to this much.
- **Engage Time** is how long a candidate has to persist before the cut
  fully engages -- long enough to ignore a single gust or transient, short
  enough to step in before a real oscillation damages the airframe.

If you find yourself needing the limiter to actually catch something in
normal flight, that's a signal to revisit the static tune (Master Gain,
gain curve, or the PID Gains themselves) rather than to rely on the
limiter as the fix.
