# Changelog

All notable changes to this project will be documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

### Planned — v0.3.0
- Countdown timer panel with presets: 10 s (test), 5 / 10 / 15 / 20 / 25 / 30 min
- Uses HA native `timer` helper — continues even when browser is closed
- Live countdown display reading `timer.finishes_at` attribute

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
