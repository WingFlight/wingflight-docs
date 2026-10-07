# Auto Trim

Auto Trim (ported conceptually from Rotorflight/iNav's BOXAUTOTRIM) captures
trim automatically from sustained stick input: hold a small, steady
correction in cruise flight, and Auto Trim gradually folds that correction
into the aircraft's trim so you no longer need to hold it.

This is particularly useful for dialing in trim on a new airframe without
repeated land-adjust-relaunch cycles.

## How the capture works

Flip the switch on while armed and hold a steady correction for about 2
seconds -- Auto Trim averages the *actual, already-mixed* servo output
over that window, and the difference from the servo's center becomes its
saved trim: the value in the [Servos](../configurator/tabs/servos.md#trim)
tab's **Trim** column. Mid and the Min/Max end stops don't change, and the
trim is limited to 20% of the servo's Scale like any other trim. Only servos actually fed by a
stabilized axis (roll/pitch/yaw) are touched -- outputs driven purely by a
raw RC channel, an override, or a logic condition are left alone, since
those aren't part of the drift this feature corrects for.

Flip the switch back off before disarming and the capture is abandoned,
restoring the previous trims exactly -- nothing is written unless you
disarm *while* the new trim is still active, at which point it's saved
the same way any other live-adjusted value is (on disarm).

If a Mapped [Servo Trim](../configurator/tabs/servos.md#trimming-in-flight)
knob is also in use, its offset is left out of the trim Auto Trim saves.
The captured trim is what the aircraft needs *without* the knob, so the
knob keeps working on top of it instead of being baked in and then applied
a second time.

A bus servo cloning a PWM output isn't trimmed itself: it already carries
its PWM servo's trim.

For a manual alternative -- nudging an axis's trim by a fixed amount per
switch press, rather than capturing a sustained stick correction -- see
[Trimming in flight](../configurator/tabs/servos.md#trimming-in-flight) on
the Servos tab. To make a trim part of the center, use **Move trims into
Center** on the [Servos](../configurator/tabs/servos.md#clear-trims-and-move-trims-into-center)
tab.

!!! note "Trying it on the bench"
    Auto Trim only starts while the aircraft is armed, and arming is blocked
    while the Configurator or the CLI is connected over USB, so it can't be
    tried that way. To check it on the ground, power the model from its
    battery with the props off and no USB, arm, tilt it so a surface moves
    off center and hold it there, switch Auto Trim on for a few seconds,
    and disarm with the switch still on. Then connect and check the Trim
    column on the [Servos](../configurator/tabs/servos.md#trim) tab. On an aircraft that is
    still and level the servo is already at center, so nothing visibly
    changes.

Auto Trim fixes a steady, always-there offset -- it isn't the right tool
for a pitch change that only shows up when flaps go down. If the plane
trims out fine clean but balloons (or dips) with flaps out, see
[Mixer → Flap-to-Elevator Compensation](../configurator/tabs/mixer.md#flap-to-elevator-compensation)
instead.
