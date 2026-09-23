# GPS Navigation

The GPS Navigation tab configures [GPS RTH and Loiter](../../flight-modes/gps-rth.md)
-- the fixed-wing navigation controller behind both, and behind the
Failsafe tab's [GPS Rescue](../../flight-modes/gps-rescue.md) procedure too.
It only appears once the **GPS** feature is enabled (Configuration tab).

These settings used to be CLI-only, then briefly lived on the Failsafe tab;
they moved here because they aren't failsafe-specific -- **GPS RTH** and
**GPS LOITER** (whether switched on by hand or started by the Failsafe
tab's GPS Rescue procedure) all use them.

## Settings

| Field | Setting | Default | Meaning |
|---|---|---|---|
| RTH Altitude | `nav_rth_altitude` | 50m | Altitude RTH climbs or descends toward |
| Loiter Radius | `nav_loiter_radius` | 75m | Orbit radius for GPS LOITER |
| Loiter Direction | `nav_loiter_direction` | CW | Orbit direction, clockwise or counter-clockwise |
| Min Satellites | `nav_min_sats` | 8 | Minimum satellite count for navigation to run |
| Max Bank Angle | `nav_max_bank_angle` | 25° | Largest bank the navigation commands |
| Max Pitch Angle | `nav_max_pitch_angle` | 15° | Largest pitch the navigation commands |

Two more, under Expert:

| Field | Setting | Default | Meaning |
|---|---|---|---|
| Bearing Gain | `nav_bearing_kp` | 200 | Bank per degree of course error, percent |
| Altitude Gain | `nav_altitude_kp` | 100 | Pitch per metre of altitude error, percent |

See [GPS RTH and Loiter](../../flight-modes/gps-rth.md) for how these are
actually used, their defaults' practical effect, and known limits --
this tab is just where you set them. Home point capture (when RTH's "home"
is recorded) is a [GPS tab](gps.md) setting, not one of these.
