# Auxiliary (Modes)

The Auxiliary tab (sometimes called "Modes") maps flight modes and other
switchable features onto transmitter auxiliary channels -- defining which
switch position(s) activate ARM, [Auto Hover](../../flight-modes/auto-hover.md),
[Attitude Hold](../../flight-modes/atthold.md),
[Auto Trim](../../flight-modes/auto-trim.md),
[Trainer Mode](../../flight-modes/trainer.md), and any other mode
supported by the firmware.

Each mode card shows the live channel position (a marker on the slider) so
you can confirm a switch flip actually lands inside the range you've set,
without needing to fly to find out.

## Ranges and Links

A mode activates from one of two kinds of assignment, and a single mode can
have several of either:

- A **Range** watches one receiver channel -- the mode is active whenever
  that channel's value falls between the Min/Max you set, so a 3-position
  switch can, for example, leave a mode off, on, or on-with-a-different-
  range depending on position.
- A **Link** activates a mode automatically whenever a *different* mode is
  already active -- useful for bundling modes together (e.g. always
  enabling one mode alongside another) without needing a spare switch or
  matching their ranges by hand. ARM can't be linked to or from, and a mode
  already defined by a link can't itself be linked to (no chained links).

When a mode has more than one Range/Link, each one past the first gets an
**AND**/**OR** setting: the mode activates when *all* of its AND
conditions are true, or when *at least one* OR condition is true.

Changes here don't take effect on the flight controller until you hit
Save.
