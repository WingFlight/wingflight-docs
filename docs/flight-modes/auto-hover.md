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

Auto Hover snaps the target straight to vertical at the heading you were
flying when you engaged it, and the aircraft rotates up to it at up to **Max
Rate** (120 deg/s by default). Roll is not part of the correction: your
aileron stick passes straight through the whole way, so you keep roll
control during the pull-up.

Because the target is vertical from the first instant, the pull-up is only
as gentle as Max Rate makes it. A low Max Rate gives a wide, slow arc; a high
one gives a hard snap. If the airframe can't out-thrust the hold, the pitch
correction stays pinned at Max Rate, which is also what triggers the
[throttle assist](#throttle-assist), if you have it on.

## Holding the hover

Pitch and yaw are always attitude-held, correcting back toward vertical
when the sticks are released. Roll (aileron) is not held at all: in a
nose-up hover the aircraft's roll axis coincides with the world vertical
axis, so roll is the pilot's spin/pirouette control, the same role yaw
plays in normal nose-level flight. It is a free rate pass-through, with the
same rates and feel as normal flight, so a held aileron gives continuous
rotation and a centered stick stops it where it is. Torque roll is not
corrected automatically; counter it with the aileron stick.

## Tuning

These fields are on the [Profiles](../configurator/tabs/profiles.md) tab,
in the Auto Hover section of Leveling Settings.

**Gain** sets how aggressively the aircraft corrects back to vertical/held
heading once the sticks are centered. **Max Angle** bounds how far
pitch/yaw stick input can deflect the held attitude away from vertical
before it springs back on release -- the same idea as Angle Mode's max
angle, just centered on vertical instead of level. **Max Rate** is a
safety clamp on how fast Auto Hover is allowed to rotate the aircraft
toward the held attitude, so it sets how hard the
[entry into vertical](#entering-the-hover) is. The default is 120 deg/s.
Lower it for a slower, gentler pull-up; raise it if the aircraft has the
authority and you want a snappier entry.

The **Auto Hover roll deadband** field no longer does anything. Roll used to
be held once the stick centered; it is now always a free pass-through. The
setting is kept so existing profiles and tools keep working.

## Throttle assist

By default throttle stays fully manual, and the mode has no awareness of
airspeed or whether the aircraft actually has enough thrust to sustain a
vertical hover. Engaging it without enough thrust available won't get the
aircraft to vertical: the pitch correction runs flat out at Max Rate and it
will likely sink or stall rather than hover.

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

If a safety mode (Failsafe, GPS Rescue, RTH, Loiter or Angle) takes over
and later releases, Auto Hover captures a fresh target from the aircraft's
current attitude instead of resuming a stale one.
