# v0.3.0 — Countdown Timer

## What's new

A countdown timer panel is now always visible on the card, sitting between the stats bar and the pool section.

### Timer features

- **Preset buttons** — 10 s (quick test), then 5 / 10 / 15 / 20 / 25 / 30 min
- **One tap starts everything** — pressing a preset opens the valve *and* starts the timer simultaneously
- **Live countdown** — ticks every second, stays accurate after a page reload or browser restart
- **Cancel & close valve** — one button cancels the timer and closes the valve immediately

The timer uses a native HA `timer` helper (`restore: true`), so it keeps running even if HA restarts or you close your browser. An automation fires when the timer finishes and closes the valve automatically.

## HA helpers required (in addition to Phase 2 helpers)

| Helper | Type | Entity ID |
|--------|------|-----------|
| Tap LHS Lower Lawn Green Timer | `timer` | `timer.tap_lhs_lower_lawn_green_timer` |

## Automation required

Create via **Settings → Automations → New automation**:

- **Trigger:** Event type `timer.finished`, Event data `entity_id: timer.tap_lhs_lower_lawn_green_timer`
- **Action:** Call service `switch.turn_off` → `switch.tap_lhs_lower_lawn_green`

## Adapting for other valves

Replace all occurrences of `tap_lhs_lower_lawn_green` with your valve's entity prefix. Create a matching `timer` helper and automation for each valve instance.
