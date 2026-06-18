# Release notes — v0.1.1

**Released:** 2026-06-18

## What's new

- **Pool mode toggle** — a 🏊 button appears in the card header. Tap it to reveal the pool fill section; tap again to hide it. Hidden by default so the card stays clean for everyday irrigation use.
- **Pool mode section** — shows how long the valve has been open (elapsed time, updated every 30 s via JS). Volume tracking and safety cutoff placeholders are present, ready to be activated in v0.2.0.
- **localStorage persistence** — pool mode on/off state survives template re-renders (HA entity updates) without collapsing the section unexpectedly. Resets on browser storage clear.

## Upgrade from v0.1.0

Replace the `content:` value in your card with the updated one from `releases/v0.1.1/card.yaml`. The entity prefix find-replace remains the same.

Note: when find-replacing your entity prefix, replace ALL occurrences — the prefix now also appears in the JavaScript `var prefix='tap_lhs_lower_lawn_green'` line and the HTML element IDs.
