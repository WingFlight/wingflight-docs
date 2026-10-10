# Flight Tuning

**Flight Tuning** holds the settings you change most at the field: PIDs,
rates and how the model feels. The pages are the same settings as the
Configurator's, so each section below links to the Configurator page that
explains them.

![The Flight Tuning menu](../assets/images/ethos-suite/flight-tuning-menu.png)

The menu scrolls: below **Advanced** there is a third row with **Thrust
Vector**.

![The Flight Tuning menu, scrolled down to Thrust Vector](../assets/images/ethos-suite/flight-tuning-menu-2.png)

Most of these pages belong to a PID profile, shown in the title as
*PIDs #1* and so on. To work on another profile, switch to it (from the radio,
or with [Select Profile](tools-and-logs.md#select-profile)) and the page
follows.

## PIDs

P, I, D, F and B for roll, pitch and yaw.

![PIDs page](../assets/images/ethos-suite/pids.png)

See [Profiles: PID Gains](../configurator/tabs/profiles.md#pid-gains).

## Rates

RC Rate, Rate and RC Expo for each axis, for the active rate profile (the
title shows its number).

![Rates page](../assets/images/ethos-suite/rates.png)

See [Rates](../configurator/tabs/rates.md#rate-shape-and-expo).

## Flight Feel

Gain, Decay and Relax per axis, plus the overall Throttle and Speed gains.

![Flight Feel page](../assets/images/ethos-suite/master_gains.png)

See [Profiles: Flight Feel](../configurator/tabs/profiles.md#flight-feel).

## Tune Advisor

Unlike the other pages, the Tune Advisor is the radio's own. While you fly in
rate mode, the flight controller measures how the model answers the sticks,
and each time you disarm the radio stores that flight's results. Pick an axis
to see how it responds and stops, and what the advisor suggests changing and
why. **Save** writes the suggested changes for that axis to the flight
controller; **Tool** clears the data it has collected.

![Tune Advisor page](../assets/images/ethos-suite/tune_advisor_changes.png)

See [Tune Advisor](tune-advisor.md) for how flights are collected and
combined, what each suggestion means, what **Save** checks before it writes,
and how in-flight adjustments affect it.

## Filters

The gyro lowpass filters (type, cutoff, and dynamic range) and the notch
filters. The page scrolls.

![Filters page](../assets/images/ethos-suite/filters.png)

See [Gyro](../configurator/tabs/gyro.md).

## PID Controller

How the PID loop handles error: in-flight error decay, error limits, I-term
relax, and the [Cross-Axis Relax](../flight-modes/cross-axis-relax.md) and
[Snap Relax](../flight-modes/snap-relax.md) settings. The page scrolls.

![PID Controller page](../assets/images/ethos-suite/pid_controller.png)

See also [Profiles: I-Term Relax](../configurator/tabs/profiles.md#i-term-relax).

## PID Bandwidth

Cutoff frequencies for the gyro, D-term and B-term, per axis.

![PID Bandwidth page](../assets/images/ethos-suite/pid_bandwidth.png)

## Gain Curves

A gain curve for roll, pitch, yaw, throttle and speed, and the speed range
the speed curve covers.

![Gain Curves page](../assets/images/ethos-suite/gain_curves.png)

See [Profiles: Gain Curves](../configurator/tabs/profiles.md#gain-curves).

## Rate Response

How quickly the model follows the sticks: response time, acceleration limit,
setpoint boost and its cutoff for each axis, and the dynamic ceiling gain.

![Rate Response page](../assets/images/ethos-suite/rates_advanced.png)

See [Rates: Dynamics](../configurator/tabs/rates.md#dynamics-expert-mode).

## Flight Modes

The self-levelling modes, one page each.

![Flight Modes menu](../assets/images/ethos-suite/autolevel_menu.png)

| Page | Settings | More |
| --- | --- | --- |
| Acro Trainer | Gain, and the bank and pitch limits | [Trainer Mode](../flight-modes/trainer.md) |
| Angle Mode | Gain, damping, and the bank and pitch limits | [Profiles: Flight-mode settings](../configurator/tabs/profiles.md#flight-mode-settings) |
| Att Hold | Gain, deadband and rate | [Attitude Hold](../flight-modes/atthold.md) |

![Angle Mode page](../assets/images/ethos-suite/autolevel_angle.png)

## Thrust Vector

For models with thrust vectoring: the same tuning pages again, for the thrust
vector profile, plus **Att/Hdg Hold**.

![Thrust Vector menu](../assets/images/ethos-suite/thrust_vector_menu.png)

![Thrust Vector Att/Hdg Hold page](../assets/images/ethos-suite/thrust_vector_hold.png)

See [Thrust Vector](../configurator/tabs/thrust-vector.md) and
[Thrust Vector Attitude Hold](../flight-modes/tv-hold.md).
