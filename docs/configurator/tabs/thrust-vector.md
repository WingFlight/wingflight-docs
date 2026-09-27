# Thrust Vector

The Thrust Vector tab configures the independent PID loop that drives a
vectored-thrust nozzle (`FEATURE_THRUST_VECTOR`). It targets the same
roll/pitch/yaw setpoint as the main flight loop, but with its own gains,
filters, and I-term state -- so it can be tuned, adjusted, or disabled
without touching the main [Profiles](profiles.md) tune at all.

This tab only appears once **THRUST VECTOR** is enabled as a feature.

## PID Gains and Flight Feel

The PID Gains and Flight Feel tables work exactly like their
[Profiles](profiles.md) counterparts -- P/I/D/F/B per axis, and per-axis
Gain, Curve, Lock and Bounce-back Suppression. See
[Profiles -- PID Gains](profiles.md#pid-gains) and
[Profiles -- Flight Feel](profiles.md#flight-feel) for what each one does
and how to tune it; the same reasoning applies here, just for the
thrust-vector loop instead of the main one, with its own values and its
own Adjustments functions (TV Gain, TV Lock and TV Bounce-back Suppression
Roll/Pitch/Yaw).

One difference: the thrust-vector Gain (and so its Curve) also scales F,
where the main loop's does not. The relax level is under PID Settings in
Expert Mode.

## Attitude / Heading Hold

**Gain**, **Deadband**, and **Max Rate** here configure
[Thrust Vector Attitude Hold](../../flight-modes/tv-hold.md) -- an
independent hold engine for the vectored nozzle only, engaged by its own
switch and completely decoupled from the main loop's Attitude Hold/Auto
Hover/Angle mode chain.

## Profiles

Like PID and Rate profiles, up to six Thrust Vector profiles can be stored
and switched between from the profile tabs at the top of the page, and
copied from one to another. A Thrust Vector profile switches independently
of both the [PID profile](profiles.md) and the [rate profile](rates.md) --
map it to its own [Adjustments](adjustments.md) channel/switch if you want
distinct thrust-vector tunes for different flight conditions (e.g. hover
versus forward flight) without also having to change your main PID tune.
