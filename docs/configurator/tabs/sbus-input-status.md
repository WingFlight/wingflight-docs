# SBUS Input

SBUS Input is a read-only diagnostic page for the
[SBUS-In Fallback Receiver](../../flight-modes/sbus-input-fallback.md)
feature -- it has no settings of its own. It only appears in the
**Diagnostics** section of the sidebar once a serial port is assigned the
**SBUS Input (Satellite Rx)** function on the [Configuration](configuration.md)
tab's serial ports list.

| Field | Meaning |
|---|---|
| Link | Whether the SBUS-in port is currently decoding valid frames |
| Active Source | Which link is driving the aircraft right now -- Main RX or SBUS-In |
| Channels | Live channel values from the SBUS-in decoder, refreshed several times a second |

Use this page to confirm your fallback receiver is bound and wired
correctly, and to watch it take over live during a bench test -- see the
feature page's own [bench-testing procedure](../../flight-modes/sbus-input-fallback.md#bench-testing-before-you-fly).
