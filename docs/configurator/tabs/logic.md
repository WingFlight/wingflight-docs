# Logic Conditions

The Logic Conditions tab exposes a small in-firmware logic engine: simple
conditional rules (based on flight mode, RC channel values, sensor values,
etc.) that can drive outputs like LEDs, beepers, or other logic conditions.
This is the tool to reach for when you need custom behavior that isn't a
built-in flight mode.

Define conditions here, then gate any [Mixer](mixer.md) rule on one from
its Condition column -- a gated rule contributes nothing while its
condition is false.

Each of the 16 condition slots is one operation applied to two operands.
Operands aren't limited to fixed values -- they can also read an RC
channel, whether a flight mode is active, a sensor (altitude, voltage,
current, RPM, RSSI, and others), the active profile, or *another
condition's current result*, which is what lets conditions chain into
more complex logic than a single comparison. Beyond the basic comparisons
(Equal, Greater/Lower Than, Approx Equal) and boolean combinators (And, Or,
Xor, Not), a few are worth knowing specifically: **Sticky** latches true
once triggered rather than following its input from moment to moment,
**Delay** only reports true once its input has held steady for a set time
(filtering out brief blips), and **Edge** reports true only briefly on a
transition rather than for as long as the condition holds.
