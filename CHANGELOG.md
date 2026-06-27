# Changelog

All notable changes to this project will be documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

Nothing planned yet.

---

## [1.1.1] — 2026-06-27

### Fixed
- **Water meter fault indicator false-positive** — the ⚠ symbol on the water meter tile was glowing yellow even when there were no faults. Root cause: the `has_faults` guard did not include `no_alarm` (the device's "no fault" state value) in its exclusion list. It now correctly treats `no_alarm` as a non-fault state, so the icon is grey when healthy and amber only on a real alarm.

---

## [1.1.0] — 2026-06-20

### Documentation
- **Scheduler integration contract** documented in README and `docs/ai-context.md`: the per-valve `timer.{prefix}_timer` helper and the `automation.close_tap_when_timer_finishes_{prefix}` (`timer.finished` → `switch.turn_off`) — both present since v0.3.0 — are now declared as a stable public interface that external schedulers use to offload a timed valve run to this card. The companion [Garden Watering Scheduler mk2](https://github.com/rhamblen/garden-watering-scheduler-mk2) consumes this contract.

### No functional change
- The card YAML is unchanged from v1.0.1. This is a docs-only release; the timer + finished automation behaviour is exactly as shipped in v0.3.0.

---

## [1.0.1] — 2026-06-18

### Added
- **Pool fill auto-reopen automation** — if the valve closes unexpectedly during an active pool fill (firmware glitch, Zigbee dropout, HA restart), a new automation reopens it automatically after a 30-second delay. Essential for long overnight fills of 8–10 hours where manual monitoring is not practical.

### Design decisions
- 30-second delay before the reopen check gives the integration sensor time to reflect the final volume. If the cutoff automation legitimately closed the valve (target reached), `session >= target` is still true after 30 s and the reopen is skipped.
- `homeassistant.start` trigger handles the HA-restart case where the valve may be found closed on boot with pool mode still active.
- `mode: restart` means repeated unexpected closes (e.g. repeated Zigbee dropouts) each restart the 30-second wait cleanly.
- **Safe stop:** to intentionally stop a pool fill, disable pool mode (tap 🏊) before closing the valve. The automation checks `input_boolean.pool_fill_active_{prefix}` and will not reopen when pool mode is off.

---

## [1.0.0] — 2026-06-18

### Added
- **Pool fill mode** — 🏊 button in card header reveals a pool fill tracking section
- **Volume-based safety cutoff** — HA automation closes the valve automatically when session volume reaches the configurable target
- **Session volume tracking** — Jinja2 computes filled/remaining litres from Riemann sum integration delta at each re-render; no JS required
- **Progress bar** — cyan gradient fill bar shows percentage of target reached; appears once a target is set
- **Tap-to-set target** — tapping the Target row opens the `input_number.pool_fill_target` more-info panel for easy editing
- **Pool mode state in HA** — moved from `localStorage` to `input_boolean.pool_fill_active_{prefix}`; pool section now shows/hides via Jinja2 and survives re-renders reliably on all devices
- **Start volume recorded automatically** — toggling pool mode on calls `input_number.set_value` for `pool_fill_start_volume` via a self-contained IIFE; session tracking resets each time pool mode is activated

### Changed
- **Pool toggle button** — onclick converted to self-contained IIFE (same pattern as timer buttons); no longer relies on `window['zwvTogglePool_'+prefix]` window function
- **Pool section content** — replaced elapsed-time-only display with Target / progress bar / Filled this session / Remaining / Valve open for rows
- Removed `zwv-pool-note` CSS class (no longer needed)
- Added `zwv-pool-clickable`, `zwv-pool-progress`, `zwv-pool-bar` CSS classes

### New HA helpers (per valve)
- `input_number.pool_fill_target` — target fill volume in L (0–20 000 L, step 50)
- `input_number.pool_fill_start_volume` — records accumulator reading at pool mode activation (set automatically)
- `input_boolean.pool_fill_active_{prefix}` — pool mode on/off state (replaces localStorage)

### New HA automation
- `automation.pool_fill_cutoff_{prefix}` — template trigger fires when session volume ≥ target; closes the valve

### Design decisions
- Pool mode state stored in HA (`input_boolean`) rather than `localStorage` so the pool section persists across re-renders, browsers, and devices
- Session volume computed as Jinja2 delta (`volume_L - pool_start`) — recalculated on every re-render; no JS polling needed
- `fill_pct` clamped to `[0, 100]` in Jinja2 using `|min`/`|max` filter chaining
- Pool toggle IIFE detects current state from Jinja2-rendered boolean (`!{{ 'true' if pool_active else 'false' }}`) to decide whether to also set start volume, avoiding a round-trip read of the input_boolean

---

## [0.3.0] — 2026-06-18

### Added
- **Countdown timer panel** — always-visible timer section between the stats bar and pool section
- **Preset buttons**: 10 s (test), 5 / 10 / 15 / 20 / 25 / 30 min; one tap starts the timer and opens the valve
- **Live countdown display** — reads `timer.finishes_at` attribute, updated every second client-side; matches reality even after a page reload or browser restart
- **Cancel & close valve button** — appears while timer is active; cancels the HA timer and turns the valve off
- **HA timer helper** — `timer.tap_lhs_lower_lawn_green_timer` (`restore: true` so it survives HA restarts)
- **HA automation** — `automation.close_tap_when_timer_finishes_lhs_lower_lawn_green` triggers on `timer.finished` event and calls `switch.turn_off`

### Design decisions
- Timer state (idle vs. active) is resolved at Jinja2 render time using `states('timer.*')` and `state_attr('timer.*','finishes_at')` — no extra helper entity needed
- `finish_ts` (Unix timestamp) is passed from Jinja2 to JS so the client countdown stays synchronised with the server timer without polling HA
- Starting a timer preset also turns the valve on — one tap is all the user needs
- Cancel button explicitly turns the valve off rather than relying solely on the automation, so cancelling mid-session reliably closes water flow
- `window['zwvStartTimer_'+prefix]` and `window['zwvCancelTimer_'+prefix]` follow the same IIFE-and-global-function pattern as the pool mode toggle to avoid multi-card collisions

---

## [0.2.0] — 2026-06-18

### Added
- **Daily consumption** — stats bar "Today" cell showing litres used since midnight (tappable → HA daily utility meter history)
- **Monthly consumption** — stats bar "This month" cell showing cumulative litres this calendar month (tappable → monthly utility meter history)
- **Three HA helpers per valve** — Riemann sum integration sensor (m³/h → cumulative m³), daily utility meter, monthly utility meter
- **Pool mode: today's usage** — pool fill section now shows today's meter reading as a real-time volume proxy
- **Stats bar dividers** — thin vertical lines between Flow / Today / This month cells

### Design decisions
- `float(-1)` sentinel on `states()` detects unavailable/unknown utility meters without a separate `in ['unknown','unavailable']` check — negative result renders `—`
- Display format: `< 10 L` shows one decimal place, `≥ 10 L` shows integer (avoids "1234.5 L" noise)
- `unit_time: h` in the Riemann sum integration correctly converts m³/h source to m³ accumulator
- Today's usage used as pool session proxy — simpler than tracking integration sensor delta at valve-open time

---

## [0.1.1] — 2026-06-18

### Added
- **Pool mode toggle** — 🏊 button in card header reveals a hidden pool fill section; tap again to hide. Off by default so everyday irrigation use is uncluttered.
- **Pool section** — displays how long the valve has been open (elapsed time, updated every 30 s client-side via `setInterval`). Volume tracking and safety cutoff placeholders visible, to be activated in v0.2.0.
- **localStorage persistence** — pool mode on/off state stored under key `zwv_pool_{prefix}` so it survives Jinja2 re-renders (triggered by HA entity state changes) without collapsing the section.
- **`last_changed_ts` Jinja2 variable** — seeds the elapsed-time JS timer on initial render; JS keeps it live between renders.
- Card deployed to **Garden tab** of the **my-home** dashboard (`tap_lhs_lower_lawn_green` instance).

### Design decisions
- Pool mode uses `localStorage` rather than an `input_boolean` helper — avoids requiring HA helper creation for a feature used once a year on one device
- IIFE pattern and `window['zwvTogglePool_'+prefix]` function name ensures multiple valve cards on the same page don't collide
- Elapsed time seeded from Jinja2 `last_changed_ts` so the display is correct on first render even before the 30 s JS interval fires

---

## [0.1.0] — 2026-06-18

### Added
- Initial release of the Zigbee Smart Water Valve dashboard card
- Circular instrument dial showing live flow rate in L/min (converted from device-native m³/h)
- Pulsing cyan glow animation on the dial when water is actively flowing
- Valve state display: OPEN (cyan) / CLOSED (red) / UNAVAILABLE (dim grey)
- Warning indicator (⚠) in card header — lights amber when device is unavailable; taps open HA more-info
- Direct **OPEN** and **CLOSE** buttons that call `switch.turn_on` / `switch.turn_off` immediately
- Tap dial to open HA more-info for the flow rate sensor (24 h history graph)
- Entity friendly name read automatically from `state_attr` — no hardcoded label needed
- Flow stat at bottom of card — tappable for flow sensor more-info
- CSS scoped under `.zwv-` prefix to avoid collisions with HA built-in styles
- Compatible with Zigbee2MQTT; entities follow `switch.{prefix}` / `sensor.{prefix}_flow` pattern

### Design decisions
- Flow rate converted from device-native m³/h to L/min for readability (`flow_m3h × 1000 / 60`)
- OPEN/CLOSE buttons call HA services directly rather than using more-info toggle, for faster control
- Unavailable state handled explicitly: dim dial, amber warning, flow shows `?`
- `switch.{prefix}_auto_close_when_water_shortage` intentionally excluded (internal device setting, not needed in daily use)
