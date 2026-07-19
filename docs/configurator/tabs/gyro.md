# Gyro

The Gyro tab configures gyro signal filtering: low-pass filtering and
dynamic notch filtering to remove airframe/motor vibration noise from the
gyro signal before it reaches the PID loop.

Good filtering reduces motor/servo heat and noise-induced twitchiness
without introducing enough latency to hurt tracking. If you're not chasing a
specific noise problem visible in [Blackbox](blackbox.md) logs, the defaults
are a reasonable starting point.
