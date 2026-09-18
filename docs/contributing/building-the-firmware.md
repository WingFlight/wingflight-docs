# Building the Firmware

WingFlight firmware is built with the same ARM GCC toolchain and Makefile
structure as Rotorflight/Betaflight.

## Prerequisites

- A Linux, macOS, or WSL environment (native Windows builds of board
  firmware are not supported directly; the SITL simulator target is the
  exception, see [below](#simulator-sitl))
- `make`

## Installing the toolchain

```
make arm_sdk_install
```

## Building a unified target

```
make unified
```

This is the same build path used by the project's CI (see the
`release`/`snapshot` GitHub Actions workflows), producing `.hex` files under
`obj/`.

## Flashing a locally-built firmware

Locally-built `.hex` files can be flashed the same way as an official build
-- via the Configurator's Firmware Flasher, using **Load Firmware [Local]**
instead of the online build list.

## Targets

Boards are built as **unified targets** only. The old per-board targets
(the Matek F405/F411/F722/H743 and Nucleo F722/H743 builds) have been
removed in favor of the unified target configurations, so build guides and
examples use unified targets throughout. The one other supported target is
**SITL**, below.

## Simulator (SITL)

`TARGET=SITL` builds WingFlight as a native executable that talks to an
external flight-dynamics model (JSBSim, optionally with FlightGear visuals)
over UDP, so the firmware can be flown without a board. Unlike the ARM
builds, it can be built natively on Windows:

```
make mingw_sdk_install    # Windows only, once: installs MinGW-w64 GCC
make TARGET=SITL          # produces obj/main/wingflight_SITL.elf
```

On Linux/macOS the system `gcc` is used. RC input arrives over MSP, so a
joystick or the Configurator can connect to it over TCP; a few features
that need real hardware (LED strip, OSD, soft serial and similar) are not
built. The bridge, launcher, aircraft models, joystick tool and automated
checks live in the
[wingflight-sitl-hitl](https://github.com/WingFlight/wingflight-sitl-hitl)
repository. See the SITL README in the firmware repository
(`src/main/target/SITL/README.md`) for ports and `eeprom.bin` handling.
