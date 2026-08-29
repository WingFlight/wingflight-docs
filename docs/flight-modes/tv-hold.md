# TV Attitude / Heading Hold

TV Attitude / Heading Hold (TV ATT HOLD) holds attitude and heading on the
Thrust Vector loop only, independent of whatever the aerodynamic control
surfaces are doing. This is aimed at jets and other aircraft with a
vectored-thrust nozzle: engage it and the nozzle holds heading/attitude on
its own while ailerons, elevator, and rudder stay in plain rate/acro under
your stick -- useful at low airspeed or post-stall, where the surfaces have
little authority but the vectored thrust still does.

It only has an effect while **THRUST VECTOR** is also engaged, and it's
completely decoupled from [Attitude Hold](atthold.md),
[Auto Hover](auto-hover.md), and Angle/Horizon mode -- engaging any of
those still stabilizes the main Roll/Pitch/Yaw loop (and so the control
surfaces) as normal, and has no effect on TV Hold one way or the other.
Enable both switches together if you want the whole aircraft, surfaces
included, to hold attitude.

Like Attitude Hold, TV Hold doesn't bound stick authority to any particular
orientation -- it freezes and holds whatever attitude the thrust-vector
target is in the instant every stick returns inside **Deadband** at once,
and gives full unmodified authority above the deadband on any stick,
continuously re-capturing its target so a future freeze is seamless. It
also defers automatically to Angle Mode, Auto Hover, GPS Rescue, Failsafe,
GPS Loiter, and GPS RTH whenever one of those is active -- those already
drive the shared setpoint to a safety-leveled value, and TV Hold never
fights that with its own frozen target.

## Tuning

**Gain** sets how aggressively TV Hold corrects the thrust-vector output
back to the frozen attitude. **Max Rate** caps how fast it's allowed to
rotate the held target while doing so. Both are configured on the Thrust
Vector tab, under **Attitude / Heading Hold** -- separate from Attitude
Hold's own Gain/Deadband/Max Rate on the
[Profiles](../configurator/tabs/profiles.md) tab, since this is an
entirely independent hold engine tuned for the thrust-vector actuators
specifically.
