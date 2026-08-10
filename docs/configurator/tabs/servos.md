# Servos

The Servos tab configures each physical servo output: center point, travel
limits/endpoints, and direction (reversing), on top of whatever mixing rules
send input to that output.

Set servo endpoints conservatively at first -- it's easier to increase travel
once you've confirmed nothing binds mechanically at the extremes than to
diagnose a stalled/straining servo after the fact.

## Endpoints and scale

**Min**/**Max** cap the servo's actual pulse-width travel, to keep the arm
off a mechanical stop at either extreme. **Scale Neg**/**Scale Pos** are a
separate pair of per-side scaling factors (500μs default for a normal
1520μs-center servo, 250μs for a narrow-band 760μs-center one) -- adjust
these instead of Min/Max specifically to correct a servo that throws
noticeably further on one side than the other.

## Rate, Speed, and Reverse

**Rate** is the PWM update frequency for this output -- 50Hz for analog
servos (higher can damage them), 100-560Hz for digital servos depending on
what the datasheet supports. It only takes effect after **Save and
reboot**, since the rate is set up at boot time, not applied live like most
other fields here.

**Speed** limits how fast the servo is allowed to move, expressed as
milliseconds needed for a 60° rotation -- 0 means unlimited. Useful for
deliberately slowing a specific output (e.g. retracts) independent of the
[Mixer](mixer.md) rule's own Speed limit on the same output.

**Reverse** flips direction if a surface moves the wrong way -- check
[Mixer](mixer.md) rule polarity first if only one of several surfaces on
the same axis is backwards, since Reverse here and a rule's own Reverse
checkbox both flip the same thing from different places.

## Geometry Correction

Compensates for the non-linear geometry of a servo arm sweeping through an
arc -- the further from center, the less each degree of rotation actually
moves a pushrod linearly. Only enable it for servos driving a linkage
through an arc; leave it off for anything already linear (e.g. a direct
belt/rack-driven surface), where there's no arc geometry to correct for.

## In-Flight Trim

Center points (**Mid**) don't have to be set from this tab or the CLI --
map **Servo Trim Roll**, **Servo Trim Pitch**, or **Servo Trim Yaw** to a
momentary switch on the [Adjustments](adjustments.md) tab (Stepped mode)
and nudge them live while flying instead.

Trimming an axis shifts Mid on every servo whose [Mixer](mixer.md) rule
takes its input from that stabilized axis, not just one output -- each
servo's own Reverse flag above is respected, so e.g. two ailerons mixed
from opposite sides of the same roll input trim toward each other
correctly rather than both moving the same raw direction. Servos fed by a
raw RC channel, an override, or a logic condition rather than a
stabilized axis aren't touched -- the same rule
[Auto Trim](../../flight-modes/auto-trim.md) uses for its own capture.

Each axis can move up to ±200μs away from its last *saved* Mid before
hitting the adjustment's own limit. Disarming with a pending trim saves
it automatically, the same as any other live-adjusted value, and the
±200μs window then re-baselines to the new center, so there's always
fresh headroom to keep trimming across multiple flights rather than
being capped by the first save.

!!! warning "Best used in the air, not on the bench"
    This is meant for trimming while actually flying. Ground use over USB
    currently fights you on two fronts: this tab won't visibly pick up a
    center-point change made this way, so there's nothing to confirm/save
    from the Configurator, and having this tab open over USB blocks
    arming outright. See
    [firmware issue #17](https://github.com/WingFlight/wingflight-firmware/issues/17)
    for current status.
