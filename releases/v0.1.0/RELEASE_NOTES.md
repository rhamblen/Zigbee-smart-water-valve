# Release notes — v0.1.0

**Released:** 2026-06-18

## What's in this release

Initial release of the Zigbee Smart Water Valve dashboard card.

### Features
- Circular instrument dial — live flow rate in L/min (converted from device-native m³/h)
- Pulsing cyan glow animation on the dial when water is actively flowing
- Valve state display — OPEN (cyan) / CLOSED (red) / UNAVAILABLE (dim)
- Warning indicator (⚠) in header — lights amber when device is unavailable
- Direct **OPEN** and **CLOSE** action buttons that call the HA switch service immediately (no more-info popup needed)
- Tap the dial to open the HA more-info panel for the flow sensor (24 h history graph)
- Entity friendly name read automatically from HA — no hardcoded label

### Entities required
| Entity | Role |
|--------|------|
| `switch.{prefix}` | Valve control |
| `sensor.{prefix}_flow` | Volume flow rate (m³/h → displayed as L/min) |

### What's coming
- **v0.2.0** — daily and monthly consumption totals (requires utility meter helpers)
- **v0.3.0** — countdown timer with presets (5 min … 30 min + 10 s test mode)
