# Adjustments

The Adjustments tab configures in-flight tuning: mapping a transmitter
channel/switch to live-adjust a setting (like a PID gain or rate) without
reconnecting the Configurator. Useful for fine-tuning while flying, or for
A/B comparing two settings values in the air.

Adjustable functions are grouped by category -- Master Gains
([Profiles](profiles.md)), Servo Trims (roll/pitch/yaw), Accelerometer Trim,
Setpoint Boost, and others -- and any function currently under live
adjustment control shows its effective value on the owning tab while the
switch/channel is active.

Adjustments made this way aren't saved to the flight controller
automatically -- they're meant for freely trying values in the air without
committing to any of them. If a value you find mid-flight is worth
keeping, go set it permanently (e.g. on [Profiles](profiles.md)) and Save
from there.

## Enable Channel and Value Channel

Each adjustment slot has two channels:

- **Enable Channel** gates whether the slot is active at all -- it only
  takes effect while this channel's value falls within the range you set
  (or set it to **ALWAYS** to skip gating entirely and let Value Channel
  control it unconditionally).
- **Value Channel** is read once the slot is enabled, and drives the
  function according to whichever mode you pick below. Its **AUTO**
  option reuses the Enable Channel itself as the value source, for the
  common case of one switch/knob doing both jobs.

## Mapped vs. Stepped

- **Mapped** continuously maps the Value Channel's position across its
  range straight onto the function's Min/Max -- a knob or 3-position
  switch gives you a proportional live value, the same idea as a
  transmitter-side gain knob but applied on the flight controller side.
- **Stepped** instead treats the Value Channel like a momentary
  increase/decrease button: entering one sub-range nudges the value up by
  **Step** each time, the other sub-range nudges it down -- useful for a
  2-position switch you flick repeatedly to walk a value up or down
  without needing a channel that can hold a precise proportional position.
