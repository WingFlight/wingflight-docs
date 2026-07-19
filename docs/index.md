# WingFlight

**WingFlight** is open-source flight controller firmware for fixed-wing aircraft --
sport planes, 3D and aerobatic aircraft, and flying wings -- along with the
WingFlight Configurator used to set it up.

WingFlight is a fork of [Rotorflight](https://rotorflight.org), refocused
exclusively on fixed-wing flight rather than helicopters. It shares
Rotorflight's MSP protocol, CLI, and configuration philosophy, but its flight
modes, mixer, and tuning tools are built around fixed-wing aerodynamics --
aileron/rudder/elevator surfaces, propeller motors, cross-axis coupling
between control surfaces, and glide/aerobatic flight characteristics --
rather than swashplates, tail rotors, and rotor governors.

!!! warning "Early-stage project"
    WingFlight is under active development. Firmware is currently distributed
    as numbered snapshots (`0.0.x`) rather than stable releases, and both the
    firmware and this documentation are evolving quickly. If something here is
    wrong or missing, please open an issue or a PR.

## Where to start

- New to WingFlight? Start with [Getting Started](getting-started/index.md).
- Setting up the app from a fresh install? See the
  [Configurator](configurator/index.md) section, organized tab-by-tab.
- Want to understand a specific flight mode (auto-hover, auto-trim, the
  throttle governor, etc.)? See [Flight Modes & Features](flight-modes/index.md).
- Looking for CLI commands or MSP details? See [Reference](reference/index.md).
- Want to build the firmware or configurator yourself, or contribute? See
  [Contributing](contributing/index.md).

## Getting help

Join the [WingFlight Discord](https://discord.gg/aEyyAJTXRw/) for setup help,
tuning advice, and project discussion.
