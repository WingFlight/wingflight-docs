# Safety

Model aircraft can be dangerous, particularly on the test bench. Some simple rules:

* **Never** arm the aircraft with the propeller fitted unless you intend to fly.
* **Always** remove the propeller when setting up for the first time, flashing firmware, changing
  the mixer or servo settings, or if in any doubt.
* Keep clear of the propeller arc and the control surfaces when the aircraft is powered.

## Before installing

Read [Cli](../reference/cli-reference.md), [Controls](../reference/stick-commands.md), [Failsafe](../configurator/tabs/failsafe.md) and [Modes](../configurator/tabs/auxiliary.md). In
particular, read [Failsafe](../configurator/tabs/failsafe.md) and set a `Stage 2 - Failsafe Procedure`
deliberately: the flight controller can self-level, cut the motor and disarm, or fly home, once the link has
been down for `Guard Delay`, but it only reaches that stage after its own ~100ms/300ms detection and hold.
The receiver's own failsafe is still faster and does not depend on the flight controller at all, so configure
it too, and bench-test both before you fly.

Use the Receiver tab in the Configurator to check that your channels are centred at 1500 (1520 for
Futaba) and reach 1000 and 2000 at the ends of travel. If they do not, you may be unable to arm
(the endpoints are not reachable) or you may trigger failsafe unexpectedly. Adjust the endpoints
and sub-trims on your transmitter to get the range.

## Check the control directions

After any change to the mixer, servo reversal, or board orientation, check on the bench (with the
propeller removed) that every surface moves the right way for the stick, and that the stabilised
response, such as moving the aircraft by hand, opposes the motion. Check MANUAL and PASSTHROUGH
too, if you use them.
