# Blackbox

The Blackbox tab configures flight-data logging -- which fields are
recorded, logging rate, and (if fitted) onboard flash/SD storage management.
Blackbox logs are the primary tool for diagnosing tuning issues after a
flight, and are usually the first thing to attach when asking for tuning
help.

## When it logs

**Device** picks where logs go: onboard flash, an SD card, or an external
serial logger (each only appears if actually fitted/supported). **Mode**
then decides when logging is active: **Armed** (simplest -- logs every
flight), **Normal** (logs only when armed *and* a Blackbox switch is
active, for keeping only flights you deliberately flag), or **Switch**
(logs whenever that switch is active, armed or not). **Grace Period**
keeps logging briefly after disarming, specifically to catch the moments
right after a crash rather than cutting off exactly at disarm.

## Rate and fields

**Rate of Logging** is a divisor of the PID loop rate -- higher rates
capture more detail (useful for chasing a specific fast oscillation) at
the cost of filling storage faster; lower rates log longer flights in the
same space. The field checkboxes choose which data categories are
recorded at all -- turn off what you don't need to save space, but when in
doubt for a tuning question, leave gyro/PID/setpoint data on since that's
what most tuning advice is read from.

**Debug Mode** adds a further 8 extra values on top of the normal fields,
for a specific diagnostic (its exact meaning depends on which mode is
selected) -- leave it on `NONE` unless you're chasing something specific
that calls for it.

## Flash storage

**Initial Erase** guarantees a minimum amount of free space before
recording starts, erasing old logs if needed -- note that erasing can take
a while on some flash chips, so recording doesn't begin until it finishes.
**Rolling Erase** instead overwrites the oldest logs once storage is full,
trading guaranteed history for never running out of space -- logs can gain
gaps if the chip is slow to erase mid-flight.
