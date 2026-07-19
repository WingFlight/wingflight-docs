# Developer Guide

General guidance for contributing to WingFlight's firmware and configurator.

## Repositories

| Repo | Purpose |
|---|---|
| [wingflight-firmware](https://github.com/WingFlight/wingflight-firmware) | Flight controller firmware (C) |
| [wingflight-configurator](https://github.com/WingFlight/wingflight-configurator) | Configurator app (Svelte/Vite) |
| [wingflight-docs](https://github.com/WingFlight/wingflight-docs) | This documentation site (MkDocs) |
| [wingflight-artifacts](https://github.com/WingFlight/wingflight-artifacts) | Mirrored build artifacts (firmware binaries, etc.), used so the web Configurator can download them without hitting GitHub's release-asset CORS restrictions |
| [wingflight-targets](https://github.com/WingFlight/wingflight-targets) | Unified target board configs |

## Firmware changes

See [Building the Firmware](building-the-firmware.md) to get a local build
working first. Firmware PRs should build cleanly via `make unified` and
should not silently change saved-parameter layout without a parameter-group
version bump (see `PG_REGISTER` version numbers in `src/main/pg/`).

## Configurator changes

The Configurator is a Vite + Svelte app. See its repository's README for
local dev setup (`pnpm install`, `pnpm start`).

## Docs changes

This site is built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).
Every page is plain Markdown under `docs/`; the site nav is defined in
`mkdocs.yml`. To preview locally:

```
pip install -r requirements.txt
mkdocs serve
```
