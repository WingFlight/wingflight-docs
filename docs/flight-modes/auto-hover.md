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

Only pitch and yaw are attitude-held, correcting back toward vertical when
the sticks are released. Roll (aileron) is left as a free rate pass-through
rather than held -- in a nose-up hover the aircraft's roll axis coincides
with the world vertical axis, so roll becomes the pilot's spin/pirouette
control, the same role yaw plays in normal nose-level flight.

## Tuning

**Gain** sets how aggressively the aircraft corrects back to vertical/held
heading once the sticks are centered. **Max Angle** bounds how far
pitch/yaw stick input can deflect the held attitude away from vertical
before it springs back on release -- the same idea as Angle Mode's max
angle, just centered on vertical instead of level. **Max Rate** is a
safety clamp on how fast Auto Hover is allowed to rotate the aircraft
toward the held attitude -- most relevant the instant it engages from
forward flight, since it commands a large, sudden attitude change toward
vertical.

Auto Hover holds attitude and heading only -- throttle stays fully manual,
and the mode has no awareness of airspeed or whether the aircraft actually
has enough thrust to sustain a vertical hover. Engaging it without enough
thrust available will still command the pitch-up, and the aircraft will
likely stall or tumble rather than hover.
