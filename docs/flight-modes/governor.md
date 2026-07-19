# Throttle Range Governor

The Throttle Range Governor shapes motor throttle response across its
range, rather than commanding a raw linear throttle-to-motor-output
mapping. This is distinct from PID tuning -- it governs how throttle input
translates to commanded motor output, independent of attitude
stabilization.
