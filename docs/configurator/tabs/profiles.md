# Profiles

The Profiles tab manages PID tuning profiles -- the actual stabilization
gains that control how tightly the flight controller holds your commanded
attitude.

Multiple profiles can be configured and switched between in flight (via an
[Auxiliary](auxiliary.md) mode mapping), which is useful for having distinct
tunes for, e.g., calm cruising versus aggressive 3D/aerobatic flight on the
same airframe.

## Master Gain

Master Gain sets one overall gain per axis (Roll, Pitch, Yaw) that scales the
whole PID loop for that axis, each with its own optional gain curve so the
scaling can vary with stick position rather than applying a single flat
multiplier. In expert mode a fourth row, Throttle (TPA), attenuates gains
across the throttle range using the same shared gain-curve pool.

Any of these gains can also be mapped to a transmitter switch/knob from the
[Adjustments](adjustments.md) tab for live in-flight tuning -- when a gain is
under live adjustment control, its row shows the current effective value
being commanded in place of the static configured number.
