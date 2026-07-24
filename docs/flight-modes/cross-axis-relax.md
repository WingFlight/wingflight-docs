# Cross-Axis Relax

Cross-Axis Relax reduces unwanted coupling between control axes -- for
example, rudder input inducing unintended roll, or vice versa, due to
airframe geometry (a common characteristic on flying wings and some
tailless/rudder-heavy designs). Rather than fighting this coupling purely
through PID gains, Cross-Axis Relax specifically relaxes the response on a
secondary axis while a primary axis is being actively commanded.

This was added specifically to address fixed-wing cross-axis coupling (e.g.
rudder-to-roll on normal-stabilization airframes), and is one of the more
airframe-specific tuning tools -- see the [Mixer](../configurator/tabs/mixer.md)
tab for related mixing setup.

## Tuning

Off by default (Strength = 0) -- it does nothing until an airframe actually
shows the coupling it's meant to fix. **Roll Strength** and **Pitch
Strength** each set the maximum feedback reduction (%) applied to that axis
while yaw/rudder is active -- raise Roll Strength if rudder input visibly
rolls the aircraft and fighting it through roll PID gains alone feels
artificial; only touch Pitch Strength if rudder-to-elevator coupling
specifically feels artificially held back. **Level** sets how much yaw
stick input is needed to reach that maximum strength, and **Cutoff**
smooths the yaw-activity detector driving all of this, so relax engages
smoothly rather than snapping on/off with every small rudder twitch.
