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
