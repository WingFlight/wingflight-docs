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
Master Gain, I-Term Decay and I-Term Relax, plus an Expert Mode Gain Curves panel. See
[Profiles -- PID Gains](profiles.md#pid-gains) and
[Profiles -- Flight Feel](profiles.md#flight-feel) for what each one does
and how to tune it; the same reasoning applies here, just for the
thrust-vector loop instead of the main one, with its own values and its
own Adjustments functions (TV Master Gain, TV I-Term Decay and TV I-Term Relax
Roll/Pitch/Yaw).

One difference: the thrust-vector Master Gain (and so its gain curve) also scales F,
where the main loop's does not. The relax level is under PID Settings in
Expert Mode.

## Defaults

The thrust-vector defaults are tuned for a nozzle, not copied from the main loop:

| Setting | Default | Why |
| --- | --- | --- |
| P | 150 on every axis | A nozzle is symmetric, so pitch and yaw match. The main loop's higher yaw P makes up for a weak rudder. |
| I | 15 | Small, so I can't park the nozzle off centre. TV Hold already supplies the long-term correction. |
| F | 30 | In a hang there is no airflow to damp the nozzle, so rate feedforward over-drives it. P carries the response. |
| Error limit | 25 | Caps how far I can move the nozzle. |
| I-term decay | 0.30 s, no limit | The nozzle returns to centre quickly after a stick move. |
| Hold gain | 25 | Keeps TV Hold slower than the rate loop. |

A thrust-vector axis that no **Stabilized TV** mixer rule uses outputs zero and builds no I.

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
