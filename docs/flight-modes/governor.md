# Governor

The Governor shapes motor throttle response, from holding a fixed idle
output through to actively governing motor/rotor RPM, rather than commanding
a raw linear throttle-to-motor-output mapping. This is distinct from PID
tuning -- it governs how throttle input translates to commanded motor
output, independent of attitude stabilization.

It's configured from the Governor section of the
[Motors](../configurator/tabs/motors.md) tab, but has no effect until the
**GOVERNOR** switch is assigned to a channel on the
[Auxiliary](../configurator/tabs/auxiliary.md) tab -- engaging that switch is
what actually activates idle hold or RPM governing, depending on the mode.

## Modes

| Mode | Behavior |
|---|---|
| Off | Disables the governor entirely; throttle passes straight through. |
| RPM Idle/Max | Holds a target idle RPM below the handover throttle using RPM feedback, with an optional max-RPM limit above handover to prevent overspeed. |
| Throttle Idle | Holds a fixed throttle percentage at idle; needs no RPM source, but won't correct for load or battery sag. |
| RPM Range | Maps the throttle stick across a Min RPM (0%) to Max RPM (100%) range, governing speed through the whole flight rather than just at idle. |

RPM Idle/Max and RPM Range modes require an RPM source: bidirectional DShot
telemetry, ESC telemetry, or a frequency sensor.

## Tuning

RPM-governing modes hold their target using a filtered P/I loop: P gain
corrects RPM error but always settles a little below target, and I gain
closes that remaining gap. Handover Throttle sets the stick position below
which the governor takes over (not used in RPM Range mode, which spans the
whole stick), and Throttle Ceiling caps the maximum throttle the governor
may command regardless of RPM error.

## Rapid throttle changes

RPM Idle mode learns the approximate throttle needed to hold the idle RPM
and uses it as a floor. A fast throttle chop below the handover point
settles near that learned value instead of momentarily cutting to 0% and
relying on the P/I loop to recover afterward, which reduces the risk of ESC
desync on abrupt throttle cuts. This estimate is learned in flight, so it
has nothing to floor against until the governor has held idle RPM steadily
at least once.

The max-RPM limiter (above handover) keeps its own state live for as long
as it's engaged, not just on the frames where it's actively pulling
throttle down, so brief, repeated excursions over the limit don't restart
from a cold response each time.
