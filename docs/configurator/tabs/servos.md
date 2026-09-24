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
what the datasheet supports. The default is 50Hz, the safe choice for any
servo; raise it only once you've confirmed your servos support it. It only takes effect after **Save and
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

## Balance Curve

When two servos drive the same surface (dual ailerons, split flaps) and
don't quite track each other, a per-servo **balance curve** trims one
servo's travel to match its partner. Balance curves are edited on the
[Curves](curves.md#servo-balance-curves) tab, not here. Any servo with a
non-flat curve shows a curve icon in its row; click it to jump straight to
that servo's curve. A bus servo that is cloning a PWM output (see
[below](#clone-pwm-outputs-to-bus-servos)) shows its source PWM servo's
curve, since that's the one actually shaping its output.

!!! note
    The **Geometry Correction** switch that older Configurator versions
    showed on this tab has been removed from the Configurator, so the
    correction can no longer be turned on or off from here.

## In-Flight Trim

Center points (**Mid**) don't have to be set from this tab or the CLI --
map **Servo Trim Roll**, **Servo Trim Pitch**, or **Servo Trim Yaw** on the
[Adjustments](adjustments.md) tab and trim live while flying instead. The
two adjustment modes behave differently:

- **Stepped** (a momentary switch you flick to walk the trim up or down)
  changes **Mid** itself, and the change is saved.
- **Mapped** (a knob or channel position that sets the trim directly) adds
  an offset to the servo's output *on top of* Mid. It never changes Mid, is
  not saved, and starts from zero every time the flight controller boots,
  following the knob's position from there.

Trimming an axis moves every servo whose [Mixer](mixer.md) rule
takes its input from that stabilized axis, not just one output -- each
servo's own Reverse flag above is respected, so e.g. two ailerons mixed
from opposite sides of the same roll input trim toward each other
correctly rather than both moving the same raw direction. Servos fed by a
raw RC channel, an override, or a logic condition rather than a
stabilized axis aren't touched -- the same rule
[Auto Trim](../../flight-modes/auto-trim.md) uses for its own capture.

With **Stepped**, each axis can move up to ±200μs away from its last
*saved* Mid before hitting the adjustment's own limit. Disarming with a
pending trim saves it automatically, the same as any other live-adjusted
value, and the ±200μs window then re-baselines to the new center, so
there's always fresh headroom to keep trimming across multiple flights
rather than being capped by the first save.

With **Mapped**, the offset on any one servo is limited to 20% of that
servo's Scale (the larger of Scale Neg and Scale Pos -- ±100μs at the
default 500μs), however far the knob is turned or whatever the channel
reads, and it is applied inside the servo's Min/Max travel limits. Because
it is never saved, a knob that is misread -- for example a channel that
isn't valid yet just after power-up -- can move a surface by at most that
much and leaves nothing behind once the reading is right again. It also
means the knob can't stack on top of its own saved result after a reboot.
The firmware also ignores a trim channel until the receiver link has been
continuously present for a full second (both at power-up and after a brief
link drop), so a channel that comes online later than the link itself can't
pull a servo off center in that gap.

Because a Mapped trim doesn't change Mid, the **Mid** field on this tab
doesn't move when you turn the knob. Servos that have a Servo Trim
adjustment set up show a badge in the **Trim** column (highlighted while
the adjustment is active, with its channel, e.g. `R CH9` for roll on
channel 9). When the firmware and Configurator both support it, the badge
is followed by the live offset, for example `+10` or `-25`, so you can see
a trim is in effect. It is display-only and never counts as an unsaved
change on this tab.

!!! warning "Stepped trim is best used in the air, not on the bench"
    Stepped trimming is meant for trimming while actually flying. Ground use over USB
    currently fights you on two fronts: this tab won't visibly pick up a
    center-point change made this way, so there's nothing to confirm/save
    from the Configurator, and having this tab open over USB blocks
    arming outright. See
    [firmware issue #17](https://github.com/WingFlight/wingflight-firmware/issues/17)
    for current status. Mapped trims are not affected: they need no arming
    and can be tried with the Configurator connected.

## Bus Servos

Enabling **SBUS Output** or **FBUS Master** on a serial port (see
[Configuration](configuration.md)) adds a second "Bus Servo Configuration"
table below the PWM one, covering 16 additional outputs (24 with
[24-channel F.Bus output](#24-channel-fbus-output) on) -- for digital bus
servos wired to that UART instead of individual PWM wires. Each bus output
has the same Min/Max/Scale/Speed/Reverse fields as a PWM servo, above.
Bus servo N is channel N on the bus.

### 24-channel F.Bus output

With a port set to **FBUS Master**, a **24-channel F.Bus output** switch
appears above the bus servo table. It is **OFF** by default, which sends the
standard 16-channel F.Bus frame. Turn it **ON** to send the 24-channel frame
and drive bus servos 17-24, for example FrSky F.Bus servos assigned to those
channels. Leave it off if anything on the bus only accepts the 16-channel
frame.

In the 16-channel frame, channels 17 and 18 are on/off only: they are on when
that bus servo's output is at 1500 µs or above, and they aren't listed in the
table. The 24-channel frame carries channels 1-24 as normal channels and has
no on/off channels.

With a 24-channel F.Bus receiver, receiver channels 19-24 are available too
(see [Receiver](receiver.md#channel-assignment)), so they can be mixed
straight to bus servos 19-24.

SBUS output always sends 16 channels plus the two on/off channels. The CLI
setting is `fbus_master_channels` (`16` or `24`); see the
[CLI Reference](../../reference/cli-reference.md#fbus-output).

### Clone PWM outputs to bus servos

**Clone PWM outputs to bus servos** is **ON** by default: bus channel 1
mirrors PWM servo 1's final output, bus channel 2 mirrors PWM servo 2, and
so on for every bus channel that has a PWM counterpart. This means any
mixer built for PWM outputs -- including one generated by the [Mixer
Setup Wizard](mixer.md) for a named model type -- drives the bus servos
too, without needing separate mixer rules for them. Bus channels beyond
your PWM output count (e.g. channel 9 on a board with 8 PWM outputs)
always run their own independent mixer rule regardless of this setting,
since there's no PWM output to mirror.

Turn it **OFF** for a custom model where the bus servos need mixer rules
of their own, distinct from the PWM outputs -- for example a different
control-surface layout on the bus side, or extra bus channels that don't
correspond 1:1 with your PWM outputs. With cloning off, a bus channel's
Min/Max/Mid/Scale/Speed/Reverse here and its own rule on the
[Mixer](mixer.md) tab (output numbers past your PWM count) take full
effect, exactly like a PWM servo does.

!!! note
    Turning this off doesn't change your existing PWM mixer rules -- it
    only changes whether the bus outputs *also* follow them. A common
    workflow is still to start from the Mixer Setup Wizard for the PWM
    side, then add or edit rules targeting the bus outputs on the
    [Mixer](mixer.md) tab afterward.
