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
over that window and writes it as the new center point, the same
underlying value the [Servos](../configurator/tabs/servos.md) tab's Mid
field or the CLI `servo` command would set. Only servos actually fed by a
stabilized axis (roll/pitch/yaw) are touched -- outputs driven purely by a
raw RC channel, an override, or a logic condition are left alone, since
those aren't part of the drift this feature corrects for.

Flip the switch back off before disarming and the capture is abandoned,
restoring the previous center exactly -- nothing is written unless you
disarm *while* the new center is still active, at which point it's saved
the same way any other live-adjusted value is (on disarm).

For a manual alternative -- nudging an axis's center by a fixed amount per
switch flick, rather than capturing a sustained stick correction -- see
[In-Flight Trim](../configurator/tabs/servos.md#in-flight-trim) on the
Servos tab.
