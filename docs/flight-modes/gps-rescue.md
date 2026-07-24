# GPS Rescue

GPS Rescue automatically returns the aircraft toward its arm location when
triggered, typically as a [Failsafe](../configurator/tabs/failsafe.md)
response to lost radio link, provided a GPS fix is available. It has no
Configurator UI yet -- everything below is tuned entirely from the CLI
(`gps_rescue_*` settings).

GPS Rescue requires a working [GPS](../configurator/tabs/gps.md) fix before
it can be relied on -- test and tune rescue behavior deliberately (in open
space, with props at a safe altitude and heading) before trusting it as your
failsafe plan.

## What it actually does

On trigger, the aircraft climbs to `gps_rescue_initial_altitude` (default
50m), flies back toward the arm point at `gps_rescue_ground_speed` (default
20 m/s), then begins descending once within `gps_rescue_descent_distance`
(default 200m) of home, aiming to reach `gps_rescue_landing_altitude`
(default 5m) by the time it arrives. Climb/descent rates are separately
capped (`gps_rescue_ascend_rate`/`gps_rescue_descend_rate`, default 5 m/s
/ 1.5 m/s) so altitude changes stay gentle rather than snapping straight to
target. Throttle and velocity are each held by their own small PID loop
(`gps_rescue_throttle_p/i/d`, `gps_rescue_velocity_p/i/d`) around a
configured throttle range (`gps_rescue_throttle_min/max/hover`) -- these
rarely need touching unless the aircraft over/undershoots its climb or
approach speed.

## Safety gates

Rescue won't start (or will abort) if conditions look unreliable rather
than pressing on regardless:

- `gps_rescue_min_sats` (default 8) -- minimum satellite count required.
- `gps_rescue_min_dth` (default 100m) -- below this distance-to-home, a
  rescue is considered unnecessary/too close to be worth the maneuver.
- `gps_rescue_sanity_checks` (on by default) -- aborts a rescue already in
  progress if GPS-derived position/velocity stops making physical sense
  (e.g. implies impossible acceleration), rather than trusting bad data
  and flying further off course.
- `gps_rescue_allow_arming_without_fix` (off by default) -- keeps arming
  blocked without a GPS fix, so you don't take off believing rescue is
  available when it isn't.
