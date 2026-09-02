# Governor

The Governor shapes motor throttle response, from holding a fixed idle
output through to actively governing motor/rotor RPM, rather than commanding
a raw linear throttle-to-motor-output mapping. This is distinct from PID
tuning -- it governs how throttle input translates to commanded motor
output, independent of attitude stabilization.

It's configured from the Governor section of the
[Motors](../configurator/tabs/motors.md) tab. Setting a mode other than
**Off** and assigning the **GOVERNOR** switch on the
[Auxiliary](../configurator/tabs/auxiliary.md) tab turns that switch into a
hard motor interlock, not just something that shapes the bottom of the
throttle curve:

- **Switch off** -- the motor is forced to 0% output. The throttle stick has
  no authority at all, at any position, until the switch is engaged. This is
  deliberate: it lets you arm and handle the model on the ground without the
  motor spinning, the same purpose an idle-up switch serves on a heli or
  helicopter-style transmitter setup.
- **Switch on** -- idle hold or RPM governing activates as described below,
  and above the handover throttle (or across the whole stick in RPM Range
  mode) the stick drives the motor normally.

If a mode other than Off is selected but no switch is assigned to
**GOVERNOR**, the interlock does not apply and the motor behaves as if the
governor were Off -- throttle passes straight through at all times. Assign
the switch if you want the mode you've configured to actually take effect.

## Modes

| Mode | Behavior |
|---|---|
| Off | Disables the governor entirely; throttle passes straight through. |
| RPM Idle/Max | Holds a target idle RPM below the handover throttle using RPM feedback, starting from the configured Idle Throttle Floor so the ESC is not commanded to zero during a rapid throttle chop. Above handover, the optional max-RPM limit can prevent overspeed. |
| Throttle Idle | Holds a fixed throttle percentage at idle; needs no RPM source, but won't correct for load or battery sag. |
| RPM Range | Maps the throttle stick across a Min RPM (0%) to Max RPM (100%) range, governing speed through the whole flight rather than just at idle. |

RPM Idle/Max and RPM Range modes require an RPM source: bidirectional DShot
telemetry or a frequency sensor. `esc_sensor_protocol = FBUS` or
`crsf_sensors_use_rpm = ON` ([CRSF Sensors](../configurator/tabs/crsf-sensors.md#rpm))
can also feed the governor -- nothing in firmware stops it -- but those
update far too slowly and infrequently for closed-loop RPM control to react
safely. Treat that as **use at your own risk**, not a supported governor
RPM source; see the [Gyro](../configurator/tabs/gyro.md#which-filter-should-i-use)
tab's own note on the same limitation for RPM Filter.

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
