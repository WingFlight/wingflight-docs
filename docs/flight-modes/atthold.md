# Attitude Hold

Attitude Hold (ATTHOLD) holds the aircraft's commanded attitude when you
release the sticks, rather than the aircraft continuing to fly whatever
attitude it was last commanded (typical for a fixed-wing aircraft with no
self-leveling). This gives a self-leveling-like behavior on demand,
independent of full autopilot features.

Unlike [Auto Hover](auto-hover.md), Attitude Hold doesn't bound stick
authority to any particular orientation -- it freezes and holds whatever
attitude the aircraft happens to be in, at any orientation, not just
vertical or level.

## Each axis holds independently

Roll, pitch and yaw each track or freeze based on **their own** stick. An
axis whose stick is inside **Deadband** (percent stick deflection) is held
at the attitude it was in when the stick centered; an axis whose stick is
outside the deadband gets full unmodified authority and continuously
re-captures its target, so the next freeze is seamless whenever that stick
settles again.

That means you can fly one axis while the others stay held:

- Work the ailerons alone for a clean axial or rifle roll while pitch and
  yaw stay locked.
- Ride the elevator through a high-alpha attitude while roll and yaw stay
  held, so torque roll can't creep in while you're not touching the
  ailerons.

## Tuning

**Gain** sets how aggressively it corrects back to the held attitude,
**Deadband** is the stick threshold above, and **Max Rate** caps how fast
it's allowed to rotate the aircraft while doing so. These are on the
[Profiles](../configurator/tabs/profiles.md) tab.

Attitude Hold works at reduced strength (about a quarter) before the
aircraft is airborne, so you can tilt the airframe by hand on the bench and
see a real, gentler correction. See
[On the bench](auto-hover.md#on-the-bench) for the details, which are shared
with Auto Hover.
