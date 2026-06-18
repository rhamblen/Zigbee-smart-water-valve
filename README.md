# 💧 Zigbee Smart Water Valve — Home Assistant Dashboard Card

A styled Home Assistant Lovelace dashboard card for **Zigbee smart water valves** paired via Zigbee2MQTT. Displays live flow rate, valve state, and direct open/close controls — all in a single circular instrument dial card.

> Designed for multi-valve gardens. One card per valve. Works on any Lovelace dashboard.

---

## Features

- **Circular meter dial** — live flow rate in L/min (converted from device-native m³/h); pulsing cyan glow when water is flowing
- **Valve state** — OPEN (cyan) / CLOSED (red) / UNAVAILABLE (dim)
- **Direct OPEN / CLOSE buttons** — call the HA switch service immediately; no popup needed
- **Warning indicator** — lights amber when the device is unavailable
- **HA theme aware** — uses CSS variables (`--primary-text-color`, `--secondary-text-color`, etc.)
- **Multi-valve** — one card per valve; find-replace the entity prefix to configure each one

### Roadmap

| Version | Feature |
|---------|---------|
| v0.1.0 ✅ | Valve control dial, flow rate, open/close buttons |
| v0.2.0 | Daily & monthly water consumption totals (utility meter helpers) |
| v0.3.0 | Countdown timer — 5/10/15/20/25/30 min + 10 s test mode |
| v1.0.0 | Pool fill mode (volume-limited continuous run), full documentation |

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| Home Assistant | Any recent version with Lovelace in storage (UI) mode |
| `custom:html-template-card` | Install via HACS — [lovelace-html-template-card](https://github.com/PiotrMachowski/lovelace-html-template-card) |
| Zigbee water valve | Paired via Zigbee2MQTT; exposes `switch.{prefix}` and `sensor.{prefix}_flow` |

---

## Quick install

1. Install `custom:html-template-card` via HACS
2. Open your dashboard in edit mode → Add card → Manual card
3. Paste the contents of [`releases/v0.1.0/card.yaml`](releases/v0.1.0/card.yaml)
4. Find-and-replace `tap_lhs_lower_lawn_green` with your actual entity prefix throughout

Full guide: **[INSTALLATION.md](INSTALLATION.md)**

---

## Entities used

| Entity | Role |
|--------|------|
| `switch.{prefix}` | Valve open/close control |
| `sensor.{prefix}_flow` | Volume flow rate (m³/h, displayed as L/min) |

Default example prefix: `tap_lhs_lower_lawn_green`

To find your prefix: **Settings → Devices & Services → your valve device** — note the entity IDs listed.

---

## Your valves (example setup)

| Friendly name | Entity prefix |
|---------------|---------------|
| Tap - LHS Lower Lawn (Green) | `tap_lhs_lower_lawn_green` |
| Tap - LHS Upper Lawn (Blue) | `tap_lhs_upper_lawn_blue` |
| Tap - LHS Fence (Green&Yellow) | `tap_lhs_fence_green_yellow_2` |
| Tap - RHS Fence | `tap_rhs_fence_2` |
| Tap - Veg Trug | `tap_veg_trug` |

---

## Colour reference

| Element | Normal | Alert |
|---------|--------|-------|
| Dial border | Dark blue | Cyan pulse (flowing) |
| Flow value | Cyan (open) | Red (closed) |
| Warning light | Dim grey | Amber glow (unavailable) |

---

## Changelog

See [CHANGELOG.md](CHANGELOG.md)

---

## Licence

MIT — free to use, adapt, and share. Attribution appreciated.
