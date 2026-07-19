# Flashing the Firmware

Firmware is flashed from the Configurator's **Firmware Flasher** tab.

## Picking a build

WingFlight firmware is currently published as numbered development
snapshots (e.g. `0.0.8`) rather than stable releases, while the project is
in its early stages. The Firmware Flasher lists available builds per target
board, pulled from the project's build history.

## DFU mode

Most WingFlight boards flash over USB using STM32's built-in DFU (Device
Firmware Upgrade) bootloader:

1. Put the board into DFU mode (usually a boot button held while powering on,
   or a dedicated bootloader pin/jumper -- check your board's documentation).
2. In the Configurator, select the **DFU** entry in the port picker.
3. Choose your target and firmware version.
4. Click **Flash Firmware**.

## Before you flash

If you're updating an existing setup rather than flashing a fresh board,
back up your configuration first (see [Backup & Restore](backup-and-restore.md))
or take a CLI `diff all` / `dump all` snapshot. A firmware update can change
parameter layouts between versions, and having a backup makes it much easier
to recover your tune afterward.
