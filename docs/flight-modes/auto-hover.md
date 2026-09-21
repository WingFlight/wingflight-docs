# Auto Hover

Auto Hover is an assisted flight mode for aircraft capable of hovering
flight (e.g. 3D-capable fixed-wing aircraft hovering on the prop), providing
stabilization assistance to help hold a stable hover attitude.

Because hovering fixed-wing flight has fundamentally different control
characteristics from forward flight, Auto Hover applies distinct handling
from WingFlight's normal forward-flight stabilization -- enable it via the
[Auxiliary](../configurator/tabs/auxiliary.md) tab on a switch you can
reach quickly, since transitioning in and out of hover is often a deliberate
part of a maneuver.

## Entering the hover

Auto Hover doesn't snap the aircraft to vertical when you engage it.
Instead it works in two steps:

1. **Level and hold.** The target starts at the aircraft's current pitch
   and heading with the wings level. If you were banked, it rolls level
   first.
2. **Rotate to vertical.** The target's pitch then rotates up to vertical
   at half of **Max Rate** (60 deg/s at the default of 120, so about a
   second and a half from level), and the aircraft follows it. Roll is
   held level the whole way, so torque roll during the pull-up is corrected
   rather than left to build.

The rotation waits whenever the aircraft can't keep up: while it is more
than 10° out of level in roll, or while the pitch correction is pinned at
Max Rate. It carries on as soon as the aircraft catches up. So an airframe
that runs out of thrust part-way up is held where it got to instead of
being dragged past what it can do, and that pinned pitch correction is also
what triggers the [throttle assist](#throttle-assist), if you have it on.

Only the aileron stick frees roll during the rotation. If you engage
already within 30° of vertical there is nothing to rotate, and the hold
starts straight away.

## Holding the hover

Pitch and yaw are always attitude-held, correcting back toward vertical
when the sticks are released. Roll (aileron) works differently: in a
nose-up hover the aircraft's roll axis coincides with the world vertical
axis, so roll is the pilot's spin/pirouette control, the same role yaw
plays in normal nose-level flight. While the roll stick is deflected, roll
is a free rate pass-through. Once it returns to center, Auto Hover
captures the roll the aircraft is in at that instant and holds it,
correcting disturbance-driven drift -- torque roll, for example -- back to
it. Unlike pitch and yaw, roll has no fixed target: it holds whatever roll
you last stopped at.

## Tuning

These fields are on the [Profiles](../configurator/tabs/profiles.md) tab,
in the Auto Hover section of Leveling Settings.

**Gain** sets how aggressively the aircraft corrects back to vertical/held
heading once the sticks are centered. **Max Angle** bounds how far
pitch/yaw stick input can deflect the held attitude away from vertical
before it springs back on release -- the same idea as Angle Mode's max
angle, just centered on vertical instead of level. **Max Rate** is a
safety clamp on how fast Auto Hover is allowed to rotate the aircraft
toward the held attitude. It also sets the pace of the
[rotation up to vertical](#entering-the-hover), which runs at half of Max
Rate. The default is 120 deg/s. Lower it for a slower, gentler pull-up;
raise it if the aircraft has the authority and you want a snappier entry.

**Auto Hover roll deadband** (percent of roll stick, default 5) is the stick
deflection below which roll counts as centered and is held. Above it, roll
is a free pass-through, so pirouettes are untouched. Raise it if the hold
grabs when you didn't mean to stop spinning; lower it if a slightly
off-center stick is stopping the roll hold from engaging.

## Throttle assist

By default throttle stays fully manual, and the mode has no awareness of
airspeed or whether the aircraft actually has enough thrust to sustain a
vertical hover. Engaging it without enough thrust available won't get the
aircraft to vertical: the rotation stops where the airframe can't keep up,
and it will likely sink or stall rather than hover.

Throttle assist is an optional, off-by-default nudge for the borderline
case. When Auto Hover's pitch correction stays pinned at Max Rate for a
sustained period, it suggests the airframe can't out-thrust the hold at
your current throttle, so the assist ramps extra throttle in on top of your
stick. It is a bounded nudge, not a fix for a genuinely underpowered
airframe.

| Setting | Default | What it does |
|---|---|---|
| **Auto Hover throttle assist gain** | 0 (off) | Percent of throttle range added per second while the trigger condition holds. 0 disables the feature entirely. |
| **Auto Hover throttle assist ceiling** | 15 | Hard cap, in percent of throttle range, on how much can ever be added. The firmware also enforces an absolute 50% limit regardless of this value. |
| **Auto Hover throttle assist trigger time** | 300 ms | How long pitch correction must stay pinned at Max Rate before the assist starts ramping in. Filters out single gusts. |

Things worth knowing:

- The boost is always **added to your own throttle stick**, never a
  replacement for it, and it ramps in and back out rather than stepping.
- It only runs while the aircraft is airborne, so sitting nose-up on a
  bench stand can't drive the throttle up.
- It resets to zero the moment Auto Hover is switched off or is preempted
  by a safety mode, and never carries over into the next engagement.
- A Max Rate of 0 disables the attitude correction, and with it the
  assist.
- It is off whenever your throttle stick is at or below the off-throttle
  point, or the radio link is lost, and it ramps back in from zero when the
  stick comes back up. The assist can never spin the motor up on its own.

## On the bench

Auto Hover (and [Attitude Hold](atthold.md)) still work at reduced
strength before the aircraft is airborne -- roughly a quarter of their
in-flight authority, the same as Angle and Horizon modes -- so tilting the
airframe by hand shows a real, gentler correction. It's not fully live
until liftoff.

The [rotation to vertical](#entering-the-hover) follows the airframe, so on
the bench, where it can't actually rotate, the target only runs ahead until
the pitch correction is pinned at Max Rate (about 24° ahead at the default
Gain and Max Rate) and then waits there. It doesn't wind up toward a vertical
target the aircraft will never reach.

When you engage within about 30° of vertical there is no rotation, and roll
locks in two stages: between 30° and 10° from vertical it only *tracks*, and
it is only allowed to freeze, and start its settle countdown, inside 10°.
That keeps it from locking a roll the aircraft is still spinning through as
it settles. Once locked, the hold stays until the aircraft leaves the 30°
band.

When you release the roll stick, roll keeps tracking until it has actually
stopped rotating (under about 15 deg/s) or 0.4 seconds have passed, and only
then freezes. That is why letting go of the aileron mid-pirouette doesn't
snap the aircraft back past where you stopped.

If a safety mode (Failsafe, GPS Rescue, RTH, Loiter or Angle) takes over
and later releases, Auto Hover captures a fresh target from the aircraft's
current attitude instead of resuming a stale one.
