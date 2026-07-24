# ESC Programming

The ESC Programming tab provides passthrough access to compatible ESC
firmware's own configuration tools (e.g. BLHeli, BLHeli_S/Bluejay, AM32),
letting you configure ESC-side settings directly through the flight
controller's USB connection without a separate ESC programming card.

The flow is: pick your ESC's manufacturer/firmware, select which ESC to
target if more than one is connected, wait for detection, then edit its
parameters directly -- **Change ESC** at any point restarts this from the
beginning if you picked wrong or want to program a different ESC. The
aircraft must be disarmed throughout; programming is blocked while armed.
