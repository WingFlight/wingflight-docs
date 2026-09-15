# Mixer

The Mixer tab defines how virtual control axes (roll, pitch, yaw, throttle)
map onto your aircraft's actual control surfaces and motor(s) -- the core of
what makes WingFlight a *fixed-wing* mixer rather than a helicopter swashplate
mixer.

Typical fixed-wing mixer setups include conventional aileron/elevator/rudder
layouts, flying wings (elevon mixing), V-tails, and multi-motor
configurations. The **Mixer Setup Wizard** builds a starting rule set from a
few questions (wing layout, aileron count, tail type) -- a much faster start
than building rules from scratch, and a good way to see a working example of
the patterns below.

If your airframe changes (e.g. converting ailerons to flaperons, or adding a
second motor), this is the tab to revisit.

If you're running digital bus servos (SBUS/FBUS output) instead of, or
alongside, PWM servos, note that they normally mirror your PWM outputs
automatically -- rules built here for PWM outputs drive both. See
[Servos → Bus Servos](servos.md#bus-servos) for how that works and when to
turn it off.

## Rules

Each rule reads one input (a stabilized axis, throttle, an RC channel, etc.)
and writes it to one output (a servo or motor), combined with whatever's
already on that output via an **Operator**:

- **Set** overwrites the output outright.
- **Add** sums onto whatever's already there.
- **Mul** multiplies whatever's already there.

Rules run top to bottom in list order, so this matters: the **first** rule
for a given output should normally be **Set** (it establishes the baseline),
and any later rule for that *same* output should normally be **Add**
(layering more input on top) -- a later **Set** silently wipes out
everything earlier rules wrote to that output, which is rarely what you
want. The Configurator flags both cases inline so a misordered rule set
doesn't quietly go unnoticed. This is exactly the pattern behind e.g.
elevon mixing: Pitch is **Set** on the elevon servo, then Roll is **Add**ed
on top of it.

Other per-rule fields:

- **Weight** (0-5000) is how strongly this rule's input drives the output --
  values above 1000 amplify the input rather than just passing it through.
- **Differential** (-100% to 100%) suppresses (positive) or boosts
  (negative) the *negative*-deflection side of this rule relative to the
  positive side -- the classic aileron-differential trick of reducing
  down-aileron throw to cut adverse yaw, without touching the input source
  itself.
- **Offset** (-2500 to 2500) adds a fixed bias to the output, independent of
  the input's value.
- **Speed** (0-60000) limits how fast this rule's output can move, for a
  slower/softer mechanical response on a specific surface.
- **Curve** assigns a reshaping curve from the [Curves](curves.md) tab to
  this rule's input before it's weighted onto the output.
- **Condition** gates the rule on a [Logic](logic.md) condition -- while
  false, the rule doesn't apply, and its row dims in the table so it's
  obvious at a glance which rules are actually contributing right now.

## Flap-to-Elevator Compensation

Many airframes pitch when flaps go down -- often a nose-up "balloon,"
sometimes the opposite. If you don't correct for it, the stabilizer will
quietly paper over a small version of this for you, which can hide the
problem during most of the flight. But that hidden correction has limits,
and it tends to run out right on landing approach when you're slow and
low -- the plane can suddenly pitch up (or down) on you during the flare,
with no warning it was coming.

The fix is to cancel the pitch change yourself, right in the mixer, so
the elevator moves with the flaps automatically:

1. Find the rule that drives your elevator servo from **Stabilized
   Pitch** -- this should already exist as a **Set** rule.
2. Add a new rule **below** it, on the *same* output, reading whatever
   input drives your flaps (an RC channel, or the same input your flap
   rule uses). Set its Operator to **Add**.
3. Adjust **Weight** (and **Reverse** if it moves the wrong way) until
   putting the flaps down no longer causes any pitch change with the
   stick centered.
4. If the pitch change isn't even across flap travel (common with
   multi-stage flaps), assign a [Curve](curves.md#mixer-curves) to the
   rule instead of relying on Weight alone.

Do this after you've already trimmed the plane for normal cruise flight
(via [Auto Trim](../../flight-modes/auto-trim.md) or the Servos tab's Mid
field) -- this rule is just for the extra pitch change flaps cause, not a
replacement for trimming the airframe itself.

## Axis Gain / Invert

**Axis Gain** scales *every* rule reading a given stabilized axis (Roll,
Pitch, Yaw) by one percentage (0-200%, 100% = unchanged) -- raise or lower
the combined throw of every surface on that axis (e.g. both aileron
servos) without re-balancing each rule's Weight individually. **Axis
Invert** flips every rule reading that axis at once, instead of editing
each rule's Reverse checkbox by hand.

Pair Axis Gain with **Control Surface Override** below it to calibrate an
exact throw: enable the override for an axis, command a known percentage
(e.g. 100%), measure the actual surface deflection with a protractor, then
adjust that axis's Gain until the measured angle matches your target.
Override only takes effect while disarmed.
