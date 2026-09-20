# Gyro

The Gyro tab configures gyro signal filtering: removing airframe/motor
vibration noise from the gyro signal before it reaches the PID loop, without
adding so much delay that the filtering itself starts to hurt tracking.

There are four different tools here, each catching noise a different way.
It's tempting to turn all of them up "to be safe," but every filter trades
noise removal for delay, and that delay comes straight out of how sharp and
locked-in the plane feels -- so the goal is the *least* filtering that keeps
things cool and quiet, not the most.

## Which filter should I use?

- If you can read real motor RPM (bidirectional DShot, or an RPM/frequency
  sensor) -- turn on the **RPM Filter** first. It uses the motor's actual
  RPM (and any gear ratio) to place notches exactly on the noise it's
  currently producing, so it introduces far less delay than guessing at a
  fixed frequency. `esc_sensor_protocol = FBUS` or `crsf_sensors_use_rpm =
  ON` ([CRSF Sensors](crsf-sensors.md)) can technically feed this too
  (nothing stops you setting either), but those update far too slowly and
  infrequently for RPM Filter's notches to track real noise -- **use at
  your own risk**, not a supported RPM source for filtering. It needs
  bidirectional DShot or a dedicated RPM/frequency-pin sensor to actually
  work as intended.
- If you don't have an RPM source -- the **Dynamic Notch Filter** is the
  next best option. It watches the gyro signal itself, hunts for noise
  peaks, and places notches on them automatically. More flexible, but
  reacts a beat slower than RPM Filter since it has to detect the peak
  first.
- The **Lowpass Filters** are the safety net underneath either one -- they
  don't need to know anything about your setup, they just cut everything
  above a cutoff. Leave Lowpass 1 on at its default; it's doing useful work
  even alongside RPM/Dynamic Notch filtering.
- The fixed **Notch Filters** are for one specific, known resonance you can
  see in a [Blackbox](blackbox.md) log that the other three aren't already
  handling -- not a first thing to reach for.

### Quick troubleshooting

| What you're seeing | Try |
|---|---|
| Servo/motor heat, high-frequency buzzing or growling | Enable RPM Filter or Dynamic Notch; if already on, raise RPM Filter Strength or lower Dynamic Notch Q |
| Mushy, laggy, "flying through jello" | Filtering is too aggressive for the setup -- try a lower RPM Filter Strength, raise Dynamic Notch Q, or raise the Lowpass cutoff |
| Twitchy at a specific throttle/RPM despite otherwise clean filtering | Look for a peak in Blackbox at that condition and target it with a fixed Notch Filter |

## RPM Filter

Uses the motor's real RPM (and configured gear ratio) to track exactly
where vibration is coming from and notch it out, rather than guessing --
this introduces substantially less delay than any filter that has to infer
noise from the gyro signal itself. It needs bidirectional DShot or an
RPM/frequency sensor; ESC telemetry (`esc_sensor_protocol = FBUS`) and
[CRSF Sensors](crsf-sensors.md) (`crsf_sensors_use_rpm = ON`) aren't fast
or precise enough, even though nothing stops you enabling either. Strength
(Low/Medium/High) trades more aggressive noise removal for more delay --
start at Medium and only go higher if you're still seeing heat or noise in
Blackbox.

## Dynamic Notch Filter

Tracks peak noise frequencies in the gyro signal and places notch filters
on them automatically -- the best option when there's no RPM source
available. Notch Count is 4-6 if this is the only filter running, or 2-4 if
it's working alongside an RPM Filter (which is already handling the main,
predictable noise). Notch Q controls how narrow/selective each notch is --
2.0-4.0 is the recommended range; going below 2.0 makes notches much wider
and adds significantly more delay. Min/Max Hz should bracket the actual
noise band -- Min just below your lowest-RPM noise frequency (but never
under 20Hz), Max about 10-20% above the highest noise frequency you expect.

## Lowpass Filters

Lowpass 1 is the one filter left on by default: a straightforward cutoff
that attenuates everything above its frequency, 1st or 2nd order. It can
also run in **Dynamic** mode, sliding its cutoff between a Min and Max
frequency instead of sitting fixed, so it can stay tighter at low
noise/RPM and open up (less delay) when there's less to filter. Lowpass 2
(expert mode) is a fixed second stage -- generally only needed around
100Hz if RPM Filter or Dynamic Notch is already doing the main work; without
either of those, two second-order lowpass stages are the recommended setup.

## Notch Filters

Two fixed notches for known, specific resonances that aren't already
handled by RPM/Dynamic Notch filtering -- each has a Center Frequency (where
attenuation is strongest) and a Cutoff Frequency (where the notch starts).
For example, a Center of 260Hz with a Cutoff of 160Hz attenuates roughly
160-360Hz, strongest at 260Hz. Find the frequency to target from a
[Blackbox](blackbox.md) gyro FFT rather than guessing.
