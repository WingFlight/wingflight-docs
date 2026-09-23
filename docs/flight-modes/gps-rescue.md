# GPS Rescue

!!! note "Now part of GPS RTH"
    GPS Rescue was inherited from Betaflight, where it was written for
    multirotors, and did not work correctly on a fixed-wing aircraft. It was
    first made to drive the same fixed-wing-native controller as
    [GPS RTH](gps-rth.md) instead
    ([firmware#146](https://github.com/WingFlight/wingflight-firmware/pull/146)),
    and the redundant **GPS RESCUE** switch that duplicated **GPS RTH** has
    since been removed entirely -- see the [GPS RTH](gps-rth.md) page for
    how the controller actually behaves, the settings, and its known
    limits.

The Failsafe tab's Stage 2 **GPS Rescue** *procedure* is unaffected and
still exists -- selecting it on the [Failsafe](../configurator/tabs/failsafe.md)
tab starts the same GPS RTH controller automatically if the radio link is
lost and stays lost. See that page for the full staged behavior, and
[GPS RTH](gps-rth.md) for how the controller itself works.

The old multirotor rescue code (a hover-throttle-learning descent
algorithm, wrong for a wing) is still in the firmware but has not been
reachable from anywhere for a while. The `gps_rescue_*` CLI settings still
exist but have no effect.
