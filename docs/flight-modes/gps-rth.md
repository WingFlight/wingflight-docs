# GPS RTH and Loiter

!!! warning "Experimental"
    RTH and Loiter are new and have not been flight-verified. A review of the
    firmware found that the **loiter direction setting appears inverted** and
    that the **RTH altitude hold may be reversed** (see
    [Known problems](#known-problems)). Test them at a safe height, in open
    space, with your hand ready on the mode switch, before you rely on them.

RTH and Loiter are the fixed-wing navigation modes. Both need a
[GPS](../configurator/tabs/gps.md) with a fix, and both are switched on from
the [Auxiliary](../configurator/tabs/auxiliary.md) tab (**GPS RTH** and
**GPS LOITER**).

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

The flight controller does not start either mode on its own when the radio
link is lost. See [Failsafe](../configurator/tabs/failsafe.md). To use
one as a recovery, map its switch to a receiver failsafe value on that
channel, and test that it works.

## Settings

These are CLI settings. They have no Configurator UI yet.

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
then limited by the Angle-mode limit (`angle_level_limit`, 55 by default).
With the defaults, 12.5 degrees of course error is enough to reach the 25
degree maximum bank, which is a firm turn. Lower `nav_bearing_kp` if the
aircraft weaves from side to side while tracking.

## Known problems

- **Loiter direction is inverted.** With `nav_loiter_direction = CW` the
  aircraft orbits counter-clockwise, and the other way around. Until this is
  fixed, set the opposite of the direction you want, and check on the first
  orbit.
- **RTH altitude may be reversed.** Pitch is positive nose-down in the
  firmware. The altitude controller commands a positive pitch when the
  aircraft is *below* the target, which would descend further. If the aircraft
  dives when it is below the RTH altitude, switch out of the mode at once and
  do not use it.
- **The loiter does not correct its radius.** It circles at whatever radius it
  entered, so a wind can push it off the circle.
- **Slow flight.** At a low ground speed or in strong wind the GPS course is
  not a good guide to heading.

These are recorded as findings H-3 and M-2 in the firmware repository's
[Flight Dynamics review](https://github.com/WingFlight/wingflight-firmware/blob/master/docs/FlightDynamics.md).
