# Traditional

Traditional mode keeps the same rate-PID stabilization as normal flight --
same P/D/F/B gains, same profile -- but forces the I-term's contribution to
zero while it's active. Instead of the small, deliberately-managed I-term
described in [Profiles](../configurator/tabs/profiles.md#why-these-defaults)
gradually correcting a steady bias, there's no integral correction at all:
the servo snaps back to center the instant you release the stick, rather
than holding against any accumulated error. It's a more traditional
RC-gyro feel, closer to a simple rate gyro than a stabilization system that
leans toward holding attitude.

Because it layers on top of whichever stabilization is already active
(plain rate flight, [Angle, or Horizon](../configurator/tabs/auxiliary.md)),
it isn't a replacement for those modes -- switch it on alongside any of
them to strip out I on that axis without touching P/D/F/B or any other
mode's behavior. Turning it back off resumes I-term correction smoothly,
since the underlying accumulator keeps decaying/relaxing normally the whole
time it's suppressed -- there's no jump or reset.
