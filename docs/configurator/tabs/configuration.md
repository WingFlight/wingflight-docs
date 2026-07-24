# Configuration

The Configuration tab covers system-level setup that doesn't belong to a
more specific tab: board/sensor alignment, ESC/motor protocol selection,
arming behavior, and other one-time-per-build settings.

Board alignment in particular is worth getting right early -- if the flight
controller isn't mounted flat/forward in the airframe, the alignment offsets
here correct for that so the IMU's reported attitude matches reality.

## Board Alignment

Board Alignment is the coarse correction for how the FC is physically
mounted, typically in 90° steps (sideways, upside down, rotated). Rather
than working out the rotation by hand, **Auto-Align FC** detects it for you:
place the aircraft flat and disarmed, press Start, then briefly lift the
tail by at least 20° -- the wizard reads the resulting accelerometer
movement and reports the detected roll/pitch/yaw. It requires a calibrated
accelerometer and only runs while disarmed.

## Mounting Trim

Mounting Trim is a separate, fine-grained correction (in degrees) for a
mounting surface that isn't perfectly level, applied *after* Board Alignment
rather than by adjusting it. Keeping the two independent matters because
Board Alignment's roll/pitch/yaw are composed in a fixed order -- once a
large 90°-step correction lands on one axis, a small correction entered as
more board alignment can visually show up on the wrong axis. Mounting Trim
always corrects the intended physical axis regardless of the board
alignment already set.

Roll and pitch mounting trim can be auto-detected the same way as board
alignment (place the aircraft level and disarmed, press Start, don't touch
it) -- yaw can't be measured from a resting accelerometer and stays manual.
If the residual tilt is too large (over 30°), that usually means Board
Alignment itself is wrong; run Auto-Align FC first, then retry.
