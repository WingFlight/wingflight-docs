# Installing the Configurator

The WingFlight Configurator is the desktop/web application used to flash
firmware, configure the aircraft, and tune flight behavior.

## Web version (recommended for most users)

The Configurator runs directly in a Chromium-based browser (Chrome, Edge,
Brave, Opera) at **[cfg.wingflight.org](https://cfg.wingflight.org)** --
no installation required. It uses the browser's WebSerial and WebUSB APIs to
talk to your flight controller, so it needs:

- A Chromium-based browser (Firefox and Safari do not support WebSerial/WebUSB)
- A secure (HTTPS) connection, which `cfg.wingflight.org` provides automatically

Multiple versions are available under the same domain -- `master` tracks the
latest development build, and specific version numbers are available for
anyone who needs to stay on a known-working release.

## Desktop app

A desktop build (NW.js-based) is also available for Windows, macOS, and
Linux, for anyone who prefers a native app or needs functionality not yet
available in the web build. See the project's release page for download
links.

## Which should I use?

For most users, the web version is the easiest way to get started -- there's
nothing to install, and it's always up to date. The desktop app exists for
platform-specific needs and offline use.
