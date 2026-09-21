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

## Letting go of a stick

An axis does not freeze the instant its stick returns to center. It keeps
tracking until it has actually stopped rotating (under about 15 deg/s), or
0.4 seconds have passed, whichever comes first, and only then freezes. If it
froze at the instant of release, it would pin the target to an attitude the
aircraft was still rotating through, and the hold would pull it back past
where you stopped. The 0.4 second cap stops a constant disturbance, such as
torque roll, from keeping an axis in tracking forever.

## I-term while holding

An axis that is holding a frozen target still lets its accumulated I-term
drain, but slowly -- at about a tenth of the normal rate, a time constant of
roughly 6 seconds at the default I-term decay time. That is slow enough that
a real steady disturbance, such as torque roll, is still held (the hold
simply re-grows the I-term it needs), but not zero, so stale I-term doesn't
keep the surfaces parked off-center when there is no error and no motion. An
axis that is tracking or settling drains its I-term at the normal rate, the
same as in ordinary flight.

## When a hold gives up

If a frozen axis has more than about 5° of error and has hardly moved
(under 5 deg/s) for 3 seconds, the correction is not achieving anything --
the aircraft is pinned on the bench, or the surface has no authority. The
axis then gives up and re-captures its target at the current attitude, so the
surfaces do not stay pegged. There is no beep or indication when it happens.

The surface then returns to center over a few seconds, because any build-up
of I-term is drained at the normal fast rate for 3 seconds after the
re-capture. A small steady error, under about 5°, never triggers this, so a
hold that is sagging slightly against a steady disturbance keeps holding.

## Known limitation: hands-off flight counts as "landed"

The firmware decides whether the aircraft is airborne from stick activity and
tilt, not from throttle or airspeed. With the sticks centered and the
aircraft within about 26° of level, it is treated as **landed**, and the
correction is cut to about a quarter (see
[On the bench](auto-hover.md#on-the-bench)). A level, hands-off aircraft in
Attitude Hold, or in Angle mode, therefore holds with much less authority
than one being flown. It returns to full authority as soon as you move a
stick, or the tilt passes about 37°. Keep this in mind when you tune the
Gain: judge it with a stick touched, not with hands off. The same applies
to Angle and Horizon mode.
