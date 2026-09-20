# GPS Rescue

!!! danger "Not suitable for fixed-wing aircraft"
    GPS Rescue is inherited from Betaflight, where it was written for
    multirotors. On a fixed-wing aircraft it does **not** fly the aircraft
    home. It can also **disarm the aircraft in flight**. Do not use it as
    your failsafe plan. Use [GPS RTH](gps-rth.md) instead, and treat that as
    experimental too.

## What it actually does today

When the **GPS RESCUE** mode is switched on (with a GPS fix), the
firmware:

- forces roll and pitch into Angle-style leveling, so the wings go level;
- steers **pitch** with a speed controller written for multirotors. If the
  ground speed is below `gps_rescue_ground_speed` it asks for more nose-down
  pitch, up to the `gps_rescue_angle` limit, which on an airplane is a dive
  rather than more thrust.

It does **not**:

- turn the aircraft toward home. The heading (yaw) command the rescue
  calculates is never applied, and no bank is commanded;
- control the throttle. The throttle and altitude controllers still run, but
  nothing sends their output to the motor. Throttle stays with your stick;
- climb to, or hold, the rescue altitude.

So the result is a wings-level glide whose pitch follows a speed controller
that cannot control speed, with no steering.

## It can disarm in flight

The rescue state machine calls a disarm, which stops the motor, in
these cases:

- rescue is switched on without a home fix;
- rescue is triggered by a failsafe while closer than `gps_rescue_min_dth`
  to home;
- the sanity checks abort the rescue (see below);
- in the final landing phase, the accelerometer magnitude passes a
  threshold. On a wing that can happen from turbulence or a maneuver long
  before the aircraft is anywhere near the ground.

## Failsafe does not start it

Older documentation described GPS Rescue as something failsafe starts when
the radio link is lost. That does not happen in WingFlight: the failsafe
stage that would start it is disabled in the firmware. See
[Failsafe](../configurator/tabs/failsafe.md). The only way to enter GPS
Rescue is a switch mapped to **GPS RESCUE** on the
[Auxiliary](../configurator/tabs/auxiliary.md) tab.

## Settings

It has no Configurator UI. The `gps_rescue_*` settings exist in the CLI:

- `gps_rescue_min_sats` (default 8) -- minimum satellite count required.
- `gps_rescue_min_dth` (default 100m) -- below this distance to home, a
  failsafe rescue disarms rather than starting.
- `gps_rescue_sanity_checks` (on by default) -- aborts a rescue in
  progress, and disarms, if the GPS data stops making physical sense or the
  aircraft is flying away from home.
- `gps_rescue_allow_arming_without_fix` (off by default) -- keeps arming
  blocked without a GPS fix.
- `gps_rescue_ground_speed`, `gps_rescue_angle` and the
  `gps_rescue_velocity_*` gains shape the pitch behaviour described above.
- The remaining altitude, ascend/descend, landing and `gps_rescue_throttle_*`
  settings configure parts of the state machine whose output is not applied
  to the aircraft.

## What to use instead

[GPS RTH](gps-rth.md) and GPS Loiter steer by banking and hold altitude by
pitch, the way a fixed-wing needs. They are new and experimental, and leave
the throttle to you. Neither is a complete recovery on its own.
