# AI Context — Zigbee Smart Water Valve Card

This file helps Claude (or any AI assistant) understand the repository structure so it can assist with modifications, updates, and publishing new versions.

---

## Repository purpose

This repository contains a Home Assistant Lovelace dashboard card for Zigbee smart water valves paired via **Zigbee2MQTT**. The card is a single `custom:html-template-card` rendered as a circular instrument dial with direct valve control buttons.

## File structure

```
/
├── README.md                        — Main project description, features, entity table, colour reference
├── CHANGELOG.md                     — Version history (Keep a Changelog format)
├── INSTALLATION.md                  — Step-by-step install guide including Phase 2 (helpers) and Phase 3 (timer) setup
├── hacs.json                        — HACS registry metadata
├── docs/
│   └── ai-context.md                — This file. AI/Claude reference for repo navigation
└── releases/
    └── v0.1.0/
        ├── card.yaml                — The full Lovelace card YAML (copy-paste ready)
        └── RELEASE_NOTES.md         — Release-specific notes for v0.1.0
```

## Card architecture

The card is a single `custom:html-template-card` with `ignore_line_breaks: true`. It uses:
- **Jinja2** for all live data (entity states and attributes)
- **Inline CSS** scoped under the `.zwv-` prefix (Zigbee Water Valve)
- **JavaScript onclick handlers** for interactivity: `hass-more-info` custom events and direct `hass.callService()` calls
- No external dependencies beyond `custom:html-template-card` itself

## Entity pattern

All valve entities follow this pattern (Zigbee2MQTT naming via HA):

| Entity | Pattern | Example |
|--------|---------|---------|
| Valve switch | `switch.{prefix}` | `switch.tap_lhs_lower_lawn_green` |
| Flow rate | `sensor.{prefix}_flow` | `sensor.tap_lhs_lower_lawn_green_flow` |
| Auto-close | `switch.{prefix}_auto_close_when_water_shortage` | (excluded from card — internal device setting) |

**Flow rate unit:** Device reports in **m³/h**. The card converts to L/min: `flow_m3h × 1000 / 60`.

## Known valve entity prefixes (this installation)

| Friendly name | Prefix | Notes |
|---------------|--------|-------|
| Tap - LHS Lower Lawn (Green) | `tap_lhs_lower_lawn_green` | Active |
| Tap - LHS Upper Lawn (Blue) | `tap_lhs_upper_lawn_blue` | Active |
| Tap - LHS Fence (Green&Yellow) | `tap_lhs_fence_green_yellow_2` | Use `_2` suffix — original is unavailable |
| Tap - RHS Fence | `tap_rhs_fence_2` | Use `_2` suffix — original is unavailable |
| Tap - Veg Trug | `tap_veg_trug` | Active |

> The `_2` suffix appears on devices that were re-paired in Zigbee2MQTT. The `_2` entity is the active one.

## Card state logic

| Condition | Dial border | Flow value | Label |
|-----------|-------------|------------|-------|
| `valve in ['unknown','unavailable']` | Dim grey, 60% opacity | `?` | UNAVAILABLE |
| `valve == 'off'` | Red (#ef5350) | `—` | CLOSED |
| `valve == 'on'` and `flow == 0` | Cyan (#4fc3f7) | `0.0` | OPEN |
| `valve == 'on'` and `flow > 0` | Cyan + pulse animation | flow L/min | FLOWING |

## Warning indicator

The ⚠ in the header glows amber (`#ff9800` with drop-shadow) when `valve in ['unknown','unavailable']`. Tapping opens HA more-info for the switch entity.

## Click interactions

| Element | Action |
|---------|--------|
| Dial | `hass-more-info` → `sensor.{prefix}_flow` (flow rate history) |
| Warning ⚠ | `hass-more-info` → `switch.{prefix}` |
| OPEN button | `hass.callService('switch','turn_on', {entity_id: 'switch.{prefix}'})` |
| CLOSE button | `hass.callService('switch','turn_off', {entity_id: 'switch.{prefix}'})` |
| Flow stat | `hass-more-info` → `sensor.{prefix}_flow` |

## CSS class reference

All classes prefixed `.zwv-`:

| Class | Purpose |
|-------|---------|
| `.zwv-dopen` | Dial — cyan border (valve open) |
| `.zwv-dclosed` | Dial — red border (valve closed) |
| `.zwv-dunavail` | Dial — grey, 60% opacity (unavailable) |
| `.zwv-flowing` | Dial — cyan pulse animation (water flowing) |
| `.zwv-fvopen` | Flow value — cyan text |
| `.zwv-fvclosed` | Flow value — red text |
| `.zwv-fson` | Status label — cyan (open/flowing) |
| `.zwv-fsoff` | Status label — red (closed) |
| `.zwv-fsunavail` | Status label — grey (unavailable) |
| `.zwv-won` | Warning indicator — amber glow |
| `.zwv-woff` | Warning indicator — dim grey |
| `.zwv-bopen` | OPEN button — green |
| `.zwv-bclose` | CLOSE button — red |

## Planned phases

| Phase | Version | Feature | HA helpers required |
|-------|---------|---------|-------------------|
| 1 | v0.1.0 ✅ | Valve control, flow rate | None |
| 2 | v0.2.0 | Daily/monthly consumption | `integration` (Riemann sum) + `utility_meter` × 2 per valve |
| 3 | v0.3.0 | Countdown timer | `timer` helper per valve + 1 automation |
| — | v1.0.0 | Pool fill mode, full docs | `input_number` + automation for volume cutoff |

## How to publish a new version

1. Create `releases/vX.Y.Z/` folder with `card.yaml` and `RELEASE_NOTES.md`
2. Update `CHANGELOG.md` with the new version section
3. Update `README.md` roadmap table to mark the version complete
4. Update version comment at top of `card.yaml`
5. Commit: `git commit -m "release: vX.Y.Z — description"`
6. Tag: `git tag -a vX.Y.Z -m "vX.Y.Z — description"`
7. Push: `git push origin main --tags`
8. On GitHub, create a Release from the tag, paste `RELEASE_NOTES.md` as the body, and attach `card.yaml` as a release asset

## How to modify the card via Claude

Tell Claude:

> "Update the Zigbee water valve card. [describe change]. Reference https://github.com/rhamblen/Zigbee-smart-water-valve for context."

Claude should:
1. Read this file (`docs/ai-context.md`) for repo context
2. Read `releases/vX.Y.Z/card.yaml` for the current card source
3. Apply the change to the card YAML
4. Update `CHANGELOG.md` — add entry under `[Unreleased]` or create a new version section
5. If deploying to HA: use `ha_config_get_dashboard` to locate the card, then `ha_config_set_dashboard` with a `python_transform` to replace the content string
6. Commit and push to GitHub

## How to add the card to a HA dashboard via Claude

Tell Claude:

> "Add the Zigbee water valve card for prefix [prefix] to my [dashboard] dashboard, [view] view, [section] section."

Claude should:
1. Call `ha_config_get_dashboard(url_path='...')` to locate the target section
2. Note the `config_hash` from the response
3. Build the card YAML with the correct prefix substituted
4. Call `ha_config_set_dashboard(python_transform='...', config_hash='...')` to insert the card
5. Confirm the card is visible at the expected location
