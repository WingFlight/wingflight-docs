# Trainer Mode

TRAINER provides **rate control with pitch and bank angle limits**, without
self-leveling. It is the closest Wingflight equivalent to the behavior Spektrum
calls SAFE Intermediate or Envelope mode. This is a behavioral comparison,
not an implementation of Spektrum's proprietary controller.

## Flying with Trainer

Inside the configured limits, the sticks command rotation rate as in normal
stabilized rate flight. Use aileron to establish a bank, center it to stop
commanding roll, and apply opposite aileron to roll out. Centering the sticks
does not command wings-level flight or capture an ATT HOLD target.

Near a limit, Trainer uses the current gyro rate to anticipate overshoot and
intervenes against commands that would carry the aircraft farther out. If an
angle is already beyond the limit, it commands a return toward the boundary.
Pilot input that helps return remains available.

| Mode | Stick behavior | Centered sticks | Pitch/bank envelope |
| --- | --- | --- | --- |
| ANGLE | Commands an attitude within the configured angle limit | Commands level flight | Limits the requested attitude |
| HORIZON | Rate control with added leveling that fades with stick deflection | Commands leveling | No enforced envelope |
| TRAINER | Rate control inside the limits | No deliberate self-leveling | Intervenes near/beyond the configured limit |

ANGLE is therefore closer to SAFE Beginner/Angle Demand; TRAINER is closer to
SAFE Intermediate/Envelope. HORIZON is a different blend, not an intermediate
angle-limited mode. See [Spektrum's SAFE setup guide](https://wiki.spektrumrc.com/spektrum/safe-setup-guide)
for its distinction between angle demand and envelope protection.

## Setup

- Enable and calibrate the accelerometer. The firmware only offers TRAINER
  when an accelerometer is available and trainer support is built in.
- Assign **TRAINER** to a switch range in [Auxiliary (Modes)](../configurator/tabs/auxiliary.md).
  If an older Configurator hides it, enable Expert Mode. Updated Configurators
  show TRAINER without Expert Mode.
- Set **Gain** and **Angle limit** in [Profiles → Trainer (angle limits)](../configurator/tabs/profiles.md#trainer-angle-limits).
  Older Configurators call this group **Acro Trainer** and require Expert Mode.
- In the Ethos suite, assign TRAINER under **Controls → Modes** and adjust
  **Acro Trainer → Gain** and **Trainer limits → Bank / Pitch** under
  **Flight Tuning → Advanced → Autolevel**. On older firmware, use the shared
  **Max** field instead; the per-axis fields are disabled.

With **MSP API 22.4 firmware and updated clients**, ANGLE and TRAINER each have
independent limits: **bank 10–90°** and **pitch 10–75°**, symmetric in both
directions. These match the configurable ranges documented for SAFE angle
demand and envelope protection. They do not reproduce Spektrum's proprietary
control algorithm or guarantee identical flight response.

Existing profiles retain their shared limits until an axis is changed. On a
fresh setup, TRAINER inherits **20°** for both axes and ANGLE inherits **55°**.
There is no universal SAFE default to copy across different aircraft. A legacy
shared pitch limit above 75° remains effective until overridden; new explicit
pitch limits are capped at 75°.

Trainer **Gain** controls correction strength when the limit is exceeded;
its default stored value is **75**. Firmware also provides the CLI setting
`acro_trainer_lookahead_ms` (default **50 ms**) for predictive intervention.

### CLI and compatibility

The per-profile settings are `angle_roll_limit`, `angle_pitch_limit`,
`acro_trainer_roll_limit` and `acro_trainer_pitch_limit`. **0** means inherit
`angle_level_limit` or `acro_trainer_angle_limit`, respectively. Nonzero
values use a minimum effective angle of 10°, with the axis maximum above.
The shared settings remain available for older clients and CLI backups.

The new values are stored separately, so upgrading does not reset PID profiles.
They follow profile selection, copying and reset. Updated clients show the
effective angles and preserve inheritance when an axis is not edited. Older
clients cannot edit the independent limits; their shared-limit changes affect
only axes still inheriting. Upgrading the Configurator/Ethos suite is therefore
recommended when using independent limits.

Use non-overlapping switch ranges for ANGLE, HORIZON and TRAINER. These are
alternative modes, not layers: AUTO HOVER, ATT HOLD, ANGLE and HORIZON each
take priority over TRAINER when their mode switches overlap. Selecting HORIZON
and TRAINER together does not produce self-leveling with Trainer limits.

## Limits of the protection

The angle boundary is a control objective, not a guarantee that the aircraft
cannot cross it. Momentum, wind, tuning and available control authority can
cause overshoot. Pitch-angle limits do not measure angle of attack or prevent
stalls, and Trainer does not manage throttle, altitude or terrain clearance.

Trainer's angle correction does not depend on the airborne estimate or motor
output, so cutting throttle does not disable its limits during a glide.
