# Thrust Vector Attitude Hold

Thrust Vector Attitude Hold (THRUST VECTOR ATTITUDE HOLD) holds attitude
and heading on the Thrust Vector loop only, independent of whatever the
aerodynamic control surfaces are doing. This is aimed at jets and other
aircraft with a vectored-thrust nozzle: engage it and the nozzle holds
heading/attitude on its own while ailerons, elevator, and rudder stay in
plain rate/acro under your stick -- useful at low airspeed or post-stall,
where the surfaces have little authority but the vectored thrust still
does.

It only has an effect while **THRUST VECTOR** is also engaged. It has its
own switch and its own target, independent of [Attitude Hold](atthold.md),
[Auto Hover](auto-hover.md), and Angle/Horizon mode -- engaging any of
those still stabilizes the main Roll/Pitch/Yaw loop (and so the control
surfaces) as normal, and does not move TV Hold's own target. Enable both
switches together if you want the whole aircraft, surfaces included, to hold
attitude.

Like Attitude Hold, TV Hold doesn't bound stick authority to any particular
orientation, and it runs the same hold engine: each axis tracks or freezes on
**its own** stick, holding the attitude it was in once that stick is inside
**Deadband** and the axis has stopped rotating, and giving full unmodified
authority above the deadband while continuously re-capturing its target, so a
future freeze is seamless. See [Attitude Hold](atthold.md) for how the
release, give-up and I-term behaviour work; they are the same here.

It defers automatically to Angle Mode, GPS Rescue, Failsafe, GPS Loiter and
GPS RTH whenever one of those is active -- those already drive the shared
setpoint to a safety-leveled value, and TV Hold never fights that with its own
frozen target. Auto Hover and Attitude Hold do not switch it off: the
Thrust Vector loop reuses the main loop's final setpoint, so while either is
active, TV Hold works on top of that setpoint.

## Tuning

**Gain** sets how aggressively TV Hold corrects the thrust-vector output
back to the frozen attitude. **Max Rate** caps how fast it's allowed to
rotate the held target while doing so. Both are configured on the
[Thrust Vector](../configurator/tabs/thrust-vector.md) tab, under
**Attitude / Heading Hold** -- separate from Attitude Hold's own
Gain/Deadband/Max Rate on the [Profiles](../configurator/tabs/profiles.md)
tab, since this is an entirely independent hold engine tuned for the
thrust-vector actuators specifically. Like the rest of the Thrust Vector
tab, Gain/Deadband/Max Rate are per Thrust Vector profile.
