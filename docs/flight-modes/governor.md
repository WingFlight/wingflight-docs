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
| RPM Idle/Max | Holds a target idle RPM below the handover throttle using RPM feedback, starting from the configured Idle Throttle Floor so the ESC is not commanded to zero during a rapid throttle chop. Above handover, the optional max-RPM limit can prevent overspeed. |
| Throttle Idle | Holds a fixed throttle percentage at idle; needs no RPM source, but won't correct for load or battery sag. |
| RPM Range | Maps the throttle stick across a Min RPM (0%) to Max RPM (100%) range, governing speed through the whole flight rather than just at idle. |

RPM Idle/Max and RPM Range modes require an RPM source: bidirectional DShot
telemetry, ESC telemetry, or a frequency sensor.

## Tuning

RPM Idle/Max uses **Idle Throttle Floor** as the minimum powered idle output
while the governor is active below handover. The RPM controller then adds P/I
correction above that floor to reach the configured Idle RPM. This keeps the
motor spinning instead of letting an RPM overshoot drive the command to 0%
throttle, which can make the ESC struggle to spool back up.

Set Idle Throttle Floor high enough that the motor keeps turning reliably
after a quick throttle chop, but not so high that the aircraft idles faster
than you want. If the floor alone produces more RPM than the configured Idle
RPM, RPM Idle/Max will not pull below the floor; lower the floor or raise the
Idle RPM target. Throttle Ceiling remains the hard maximum output the
governor may command, and also caps the effective idle floor.

RPM-governing modes hold their target using a filtered P/I loop: P gain
corrects RPM error but always settles a little below target, and I gain
closes that remaining gap. Handover Throttle sets the stick position below
which RPM Idle/Max or Throttle Idle takes over (not used in RPM Range mode,
which spans the whole stick).
