# 💧 Zigbee Smart Water Valve — Home Assistant Dashboard Card

A styled Home Assistant Lovelace dashboard card for **Zigbee smart water valves** paired via Zigbee2MQTT. Circular instrument dial, live flow rate, direct valve control, daily/monthly consumption tracking, and a countdown timer that auto-closes the valve.

> Designed for multi-valve gardens. One card per valve. Works on any Lovelace dashboard.

---

## Features

- **Circular meter dial** — live flow rate in L/min (converted from device-native m³/h); pulsing cyan glow when water is flowing
- **Valve state** — OPEN (cyan) / CLOSED (red) / UNAVAILABLE (dim)
- **OPEN / CLOSE buttons** — call the HA switch service immediately; no popup needed
- **Stats bar** — Flow | Today | This month; tap any cell for HA history popup
- **Countdown timer** — preset buttons (5–30 min) open the valve and start a native HA timer; auto-closes the valve when time expires; live countdown display
- **Pool fill mode** — volume-based safety cutoff; set a litre target, tap 🏊 to start tracking, progress bar fills as water flows, valve closes automatically when target is reached
- **Warning indicator** — lights amber when the device is unavailable
- **HA theme aware** — uses CSS variables (`--primary-text-color`, `--secondary-text-color`, etc.)
- **Multi-valve safe** — one card per valve; find-replace the entity prefix to configure each instance

---

## Versions

| Version | Status | Features |
|---------|--------|---------|
| v0.1.0 | ✅ | Valve control dial, live flow rate, OPEN/CLOSE buttons |
| v0.1.1 | ✅ | Pool mode toggle (localStorage), elapsed time display |
| v0.2.0 | ✅ | Daily & monthly water consumption (Riemann sum + utility meter helpers) |
| v0.3.0 | ✅ | Countdown timer with auto-close valve automation |
| v1.0.0 | ✅ | Pool fill mode — volume-based safety cutoff |

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| Home Assistant | Any recent version with Lovelace in storage (UI) mode |
| `custom:html-template-card` | Install via HACS → Frontend → search "HTML Template Card" ([GitHub](https://github.com/PiotrMachowski/lovelace-html-template-card)) |
| Zigbee water valve | Paired via Zigbee2MQTT; exposes `switch.{prefix}` and `sensor.{prefix}_flow` |

---

## Installation

See **[INSTALLATION.md](INSTALLATION.md)** for full instructions.

**Quick path — Claude-assisted:**

> "Help me install the Zigbee Smart Water Valve card from https://github.com/rhamblen/Zigbee-smart-water-valve — my valve entity prefix is `tap_lhs_lower_lawn_green`."

Claude will create all required helpers, the automation, and deploy the card.

**Quick path — manual:**

1. Install `custom:html-template-card` via HACS
2. Create the required HA helpers (see [INSTALLATION.md](INSTALLATION.md))
3. Download [`releases/v1.0.0/card.yaml`](releases/v1.0.0/card.yaml)
4. Find-and-replace `tap_lhs_lower_lawn_green` with your entity prefix
5. Add as a Manual card on your dashboard

---

## Entities used

| Entity | Role |
|--------|------|
| `switch.{prefix}` | Valve open/close control |
| `sensor.{prefix}_flow` | Volume flow rate (m³/h, displayed as L/min) |
| `sensor.{prefix}_flow_l_h` | Template sensor — L/h source for integration |
| `sensor.{prefix}_volume` | Riemann sum integration accumulator (L) |
| `sensor.{prefix}_daily` | Utility meter — daily reset (L) |
| `sensor.{prefix}_monthly` | Utility meter — monthly reset (L) |
| `timer.{prefix}_timer` | HA timer helper for countdown |
| `input_number.pool_fill_target` | Target fill volume in L (shared across valves) |
| `input_number.pool_fill_start_volume` | Session start accumulator — set automatically on pool mode activation |
| `input_boolean.pool_fill_active_{prefix}` | Pool mode on/off state |

---

## Colour reference

| Element | Normal | Active/Alert |
|---------|--------|-------------|
| Dial border | Dark blue `#2d6a9f` | Cyan `#4fc3f7` (open) / Red `#ef5350` (closed) |
| Dial glow | — | Pulsing cyan (flowing) |
| Warning light | Dim `#444` | Amber glow (unavailable) |
| Timer countdown | — | Cyan `#4fc3f7` |

---

## Changelog

See [CHANGELOG.md](CHANGELOG.md)

---

## Licence

MIT — free to use, adapt, and share. Attribution appreciated.
