# GPS Rescue

!!! note "Now the same thing as GPS RTH"
    GPS Rescue was inherited from Betaflight, where it was written for
    multirotors, and did not work correctly on a fixed-wing aircraft. As of
    [firmware#146](https://github.com/WingFlight/wingflight-firmware/pull/146),
    the **GPS RESCUE** switch and the Failsafe tab's GPS Rescue procedure
    both drive the same fixed-wing-native controller as
    [GPS RTH](gps-rth.md) instead -- see that page for how it actually
    behaves, the settings, and its known limits. This page just covers what
    changed and where GPS Rescue is used today.

## What it does now

Switching on **GPS RESCUE** (with a GPS fix and a home position) flies the
aircraft home and orbits it, exactly like switching on **GPS RTH** --
banking toward home, holding altitude by pitch, using the
[GPS Navigation](../configurator/tabs/gps-navigation.md) settings. There is
no separate configuration for GPS Rescue; it is the same controller,
reached by a second switch/trigger.

The old multirotor rescue code (a hover-throttle-learning descent
algorithm, wrong for a wing) is still in the firmware but is no longer
reachable from anywhere -- neither the **GPS RESCUE** switch nor the
Failsafe procedure can reach it any more. The `gps_rescue_*` CLI settings
still exist but have no effect.

## Where it's used

- **A switch** mapped to **GPS RESCUE** on the
  [Auxiliary](../configurator/tabs/auxiliary.md) tab, same as any other
  mode.
- **The Failsafe procedure.** Selecting **GPS Rescue** as the
  `Stage 2 - Failsafe Procedure` on the [Failsafe](../configurator/tabs/failsafe.md)
  tab starts this automatically if the radio link is lost and stays lost --
  see that page for the full staged behavior, including what happens
  without a GPS fix or home position, and how it ends.

## Known limits

Same as [GPS RTH](gps-rth.md): no throttle control, no wind or airspeed
correction, no altitude-managed powered landing or flare. Read that page's
Known Problems section too.
