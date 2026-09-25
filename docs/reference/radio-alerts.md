# Radio Callouts and Alerts

The WingFlight Lua suite for Ethos radios reads the flight controller's
status over telemetry. It tells you about important changes in two ways:

- **Spoken callouts**, such as "Backup receiver active" or "GPS not
  responding".
- **A banner along the bottom of the dashboard** that stays up for as long as
  a problem lasts.

This page explains what each one means and what to do about it.

## What you need

- Telemetry working between the flight controller and the radio. New and reset
  configurations have telemetry on, and use custom CRSF telemetry for
  ELRS/Crossfire. See [Telemetry](telemetry.md).
- The **System Status** and **System Config** telemetry sensors selected on the
  [Receiver tab](../configurator/tabs/receiver.md#telemetry-sensors). They are
  in the default sensor list, and the suite's **Default** button on the radio
  selects them too.
- Firmware and Lua suite versions that include the status sensors. Older
  firmware does not send them, so none of the alerts on this page appear.

## The dashboard banner

When the flight controller reports a problem, a banner appears across the
bottom of the dashboard:

- **Red**: critical. Something is wrong now and needs your attention in the
  air.
- **Amber**: warning. Usually something to fix before you fly.

Only one banner shows at a time, the most important one. If other problems are
active too, the banner ends with a count, for example
**BACKUP RX IN CONTROL (+2)**. When the top problem clears, the next one takes
its place.

The banner stays up while the condition lasts. It isn't a pop-up that times
out, and there's nothing to dismiss.

## Status alerts

These come from the flight controller's status telemetry. In the table, "When
it starts" and "When it clears" are the callouts.

| Banner | When it starts | When it clears | What it means | What to do |
|---|---|---|---|---|
| **BACKUP RX IN CONTROL** (red) | "Backup receiver active" | "Main receiver restored" | The main receiver has lost its signal and the [backup receiver](../flight-modes/backup-rx-input.md) is now flying the model. See the note below. | Fly back towards you and land. Find out why the main link dropped before the next flight. |
| **FAILSAFE** (red) | (the flight mode callout says "Failsafe") | – | The flight controller is in a failsafe stage. You will rarely see this, because telemetry travels over the main link, so a link loss usually cuts telemetry too. | Follow your failsafe plan. See [Failsafe](../configurator/tabs/failsafe.md). |
| **BATTERY CRITICAL** (red) | (the low voltage callout covers this) | – | Pack voltage is below the critical level set on the [Power tab](../configurator/tabs/power.md), or, if consumption alerts are on, the pack capacity is used up. | Land now. |
| **GYRO OVERFLOW** (red) | "Gyro overflow" | – | The gyro has been driven past its measuring range, usually by violent vibration or a crash. The stabiliser can't trust its readings while this lasts. | Land and check the flight controller's mounting, props and motor balance. |
| **BACKUP RX NO SIGNAL** (amber) | "Backup receiver lost" | "Backup receiver OK" | A backup receiver is configured, but it isn't receiving. If the main link drops now, there is no backup. | Check that the backup receiver is powered and bound, and that its transmitter module is on. |
| **RTH UNAVAILABLE** or **LOITER UNAVAILABLE** (amber) | (the flight mode callout says "RTH, unavailable" or "GPS Loiter, unavailable") | – | The RTH or Loiter switch is on, but the mode can't fly. It needs the model armed, an accelerometer, a good GPS fix and, for RTH, a recorded home position. | Wait for a GPS fix before arming. See [GPS RTH and Loiter](../flight-modes/gps-rth.md). |
| **GPS NOT RESPONDING** (amber) | "GPS not responding" | – | The GPS was working earlier in this session and has stopped talking to the flight controller. This is a wiring or power fault, not a lost fix. | Check the GPS wiring and power. GPS modes will not work until it comes back. |
| **ACC NOT CALIBRATED** (amber) | – | – | The accelerometer has never been calibrated. Self-levelling, Attitude Hold and GPS modes level to the wrong attitude. | Calibrate the accelerometer on the [Setup tab](../configurator/tabs/setup.md) with the model level. |
| **TEST OVERRIDE ACTIVE** (amber) | – | – | A servo, motor or mixer test override from the Configurator is still on, so the outputs aren't under normal control. | Turn the override off in the Configurator. Arming is blocked while a servo or mixer override is on. |
| **REBOOT REQUIRED** (amber) | – | – | A setting was changed that only takes effect after a reboot. | Power cycle the flight controller. |
| **BLACKBOX FULL** (amber) | "Blackbox full" | – | The onboard [Blackbox](../configurator/tabs/blackbox.md) log storage is full, so this flight isn't being logged. | Download the logs if you want them, then erase the flash. |

!!! note "When you'll hear the backup receiver callouts"
    Telemetry reaches the radio through the main receiver. With ELRS,
    Crossfire and FrSky, telemetry uses the same radio link as control, so
    when that link drops, the radio usually loses telemetry too. It then can't
    hear "Backup receiver active". Ethos announces the telemetry loss instead,
    and when the link comes back you'll hear "Main receiver restored".

    You'll hear "Backup receiver active" when the main receiver keeps its
    radio link but stops delivering control to the flight controller, for
    example a broken signal wire while S.Port telemetry uses its own wire.

    "Backup receiver lost" and "Backup receiver OK" are the ones to rely on:
    they tell you before takeoff whether the backup can take over.

A problem that is already present when the radio connects shows its banner,
but isn't announced. You'll hear it when it changes after that. Link changes
must hold for a moment before they're announced, so a flickering link doesn't
chatter.

## Other status callouts

These callouts have no banner:

| Callout | What it means |
|---|---|
| "Trim captured" | [Auto Trim](../flight-modes/auto-trim.md) has measured the new trim. It's kept if you disarm with the Auto Trim switch still on. |
| "Trim saved" | You disarmed with the new trim captured, and it's being saved. |
| "Trim cancelled" | The Auto Trim switch went off before you disarmed, so the old trim is back. |
| "Control limit" | The stabiliser asked for full control surface travel: it has run out of authority. Off by default, because it can be chatty on 3D models. If you hear it often in normal flight, increase the surface throws or reduce the gains. |

## Flight mode and other callouts

These existed before the status alerts, and work the same way.

| Callout | When |
|---|---|
| "Armed" / "Disarmed" | The model is armed or disarmed. |
| Flight mode name ("Angle", "Horizon", "Att Hold", "Auto Hover", "RTH", "GPS Loiter", "GPS Rescue", "Failsafe", "Manual", "Passthrough", "Normal", …) | The active flight mode changes. "Unavailable" follows the name when a GPS mode is switched on but can't engage. "Traditional" is announced when it's switched on. |
| "GPS Fix" / "GPS Fix Lost" | The GPS gains or loses its position fix. Useful on the bench while you wait to arm. |
| "Profile" + number, "Rates" + number, "Thrust Vector" + number | The PID, rate or thrust vector profile changes. |
| "Battery" + capacity and cell count | The battery profile changes. |
| "Low Voltage" | Pack voltage is below the warning level, repeated at the set interval. |
| "Fuel" + percentage, "Low Fuel" | SmartFuel crosses a callout step, or reaches empty. |
| "ESC Temperature Warning", "BEC Voltage Warning", "RX Batt Voltage Warning" | Past the thresholds set on the radio. Each is off by default. |

## Turning callouts on and off

On the radio, open the WingFlight suite and go to
**Settings → Audio → Events**:

- **State callouts**: armed, flight mode, GPS fix and profile changes.
- **FC status**: the status alerts above.

| FC status switch | Covers | Default |
|---|---|---|
| Backup receiver | Backup receiver active / main restored, backup lost / OK | On |
| Gyro overflow | Gyro overflow | On |
| GPS not responding | GPS not responding | On |
| Blackbox full | Blackbox full | On |
| Autotrim | Trim captured / saved / cancelled | On |
| Control limit | Control limit | Off |

The switches only control the callouts. The dashboard banners always show.
