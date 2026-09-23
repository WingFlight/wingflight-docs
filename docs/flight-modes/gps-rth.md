# GPS RTH and Loiter

!!! warning "Experimental"
    RTH and Loiter are new and have not been flight-verified. Test them at a
    safe height, in open space, with your hand ready on the mode switch,
    before you rely on them -- including before relying on GPS Rescue or a
    GPS Rescue failsafe procedure, which now use this same controller.

!!! note "Loiter direction and RTH altitude sign, both fixed"
    Earlier firmware orbited the opposite way to `nav_loiter_direction` (`CW`
    orbited counter-clockwise and vice versa), and pitched down instead of up
    when below the RTH altitude. Both are fixed. If you set either as a
    workaround, set it back and check on the first flight.

RTH and Loiter are the fixed-wing navigation modes. Both need a
[GPS](../configurator/tabs/gps.md) with a fix, and both are switched on from
the [Auxiliary](../configurator/tabs/auxiliary.md) tab (**GPS RTH** and
**GPS LOITER**). [GPS Rescue](gps-rescue.md) and the Failsafe tab's GPS
Rescue procedure now use this same controller too.

- **GPS RTH** flies back to the point where the aircraft was armed, and
  climbs or descends toward the RTH altitude.
- **GPS LOITER** orbits the point where the mode was switched on, at the
  altitude it was flying then. If both switches are on, RTH wins.

## How they work

Neither mode is an autopilot. Each one gives the Angle-mode self-leveling a
bank and a pitch target, so the aircraft is always stabilized as it is in
Angle mode.

- **Steering.** Bank follows the difference between the GPS course over
  ground and the direction of the target: the bigger the error, the more the
  aircraft banks, up to a maximum bank. Loiter first flies to the loiter
  radius, then circles it.
- **Altitude.** Pitch follows the altitude error, up to a maximum pitch.
  There is no throttle control.
- **Throttle stays yours.** The aircraft needs enough throttle to climb.
  Neither mode adds or removes power.
- **Your sticks still work.** Stick input is added on top of the navigation
  target, so you can nudge the aircraft.
- **No wind or airspeed correction.** Course is taken from GPS ground track,
  which is unreliable at very low speed.
- **If the GPS gets unhealthy** or has fewer than `nav_min_sats` satellites,
  navigation stops. The aircraft is left in Angle-style leveling with no
  navigation target.

Switching either mode on yourself, from the Auxiliary tab, is one way in.
The other is the [Failsafe](../configurator/tabs/failsafe.md) tab's GPS
Rescue procedure, which starts RTH automatically if the radio link is lost
and stays lost -- see that page for the full staged behavior.

## Settings

Set on the [GPS Navigation](../configurator/tabs/gps-navigation.md) tab, or
from the CLI:

| Setting | Default | Meaning |
|---|---|---|
| `nav_rth_altitude` | 50 | RTH altitude, metres |
| `nav_loiter_radius` | 75 | Loiter radius, metres |
| `nav_loiter_direction` | CW | Loiter direction |
| `nav_min_sats` | 8 | Minimum satellites |
| `nav_max_bank_angle` | 25 | Largest bank the navigation commands, degrees |
| `nav_max_pitch_angle` | 15 | Largest pitch the navigation commands, degrees |
| `nav_bearing_kp` | 200 | Bank per degree of course error, percent |
| `nav_altitude_kp` | 100 | Pitch per metre of altitude error, percent |

The bank and pitch the navigation asks for is added to your stick input and
then limited by the Angle-mode limit (`angle_level_limit`, 55 by default;
API 22.4 can override it independently with `angle_roll_limit` and
`angle_pitch_limit`).
With the defaults, 12.5 degrees of course error is enough to reach the 25
degree maximum bank, which is a firm turn. Lower `nav_bearing_kp` if the
aircraft weaves from side to side while tracking.

## Known problems

- **The loiter does not correct its radius.** It circles at whatever radius it
  entered, so a wind can push it off the circle.
- **Slow flight.** At a low ground speed or in strong wind the GPS course is
  not a good guide to heading.
- **No altitude-managed powered landing or flare.** RTH and Loiter (and GPS
  Rescue) hold altitude and steer; they don't fly a landing. As a failsafe
  procedure, GPS Rescue hands off to the same self-level-and-cut ending as
  the other procedures once its own delay elapses.

These are recorded as finding M-2 (radius, slow-flight) in the firmware
repository's [Flight Dynamics review](../contributing/tech/flight-dynamics.md),
which also has the full signal chain and sign conventions if you're changing
this code.
