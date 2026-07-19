# Building the Firmware

WingFlight firmware is built with the same ARM GCC toolchain and Makefile
structure as Rotorflight/Betaflight.

## Prerequisites

- A Linux, macOS, or WSL environment (native Windows builds are not
  supported directly)
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
