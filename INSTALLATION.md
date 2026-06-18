# Installation Guide

**Current version:** 0.1.0
**Last updated:** 2026-06-18

---

## Phase 1 — v0.1.0 (this version)

Phase 1 requires only `custom:html-template-card`. No HA helpers need to be created yet.

### Step 1 — Install the required custom card

1. Open Home Assistant → HACS → Frontend
2. Search for **"HTML Template Card"**
3. Install it and reload your browser

Direct link: [lovelace-html-template-card on GitHub](https://github.com/PiotrMachowski/lovelace-html-template-card)

---

### Step 2 — Confirm your entity IDs

In Home Assistant go to **Settings → Devices & Services → your Zigbee valve device**.

The card expects two entities per valve:

| Entity | Example |
|--------|---------|
| `switch.{prefix}` | `switch.tap_lhs_lower_lawn_green` |
| `sensor.{prefix}_flow` | `sensor.tap_lhs_lower_lawn_green_flow` |

Note your prefix — everything between `switch.` and the end of the switch entity ID.

**Known prefixes for this installation:**

| Tap | Prefix |
|-----|--------|
| LHS Lower Lawn (Green) | `tap_lhs_lower_lawn_green` |
| LHS Upper Lawn (Blue) | `tap_lhs_upper_lawn_blue` |
| LHS Fence (Green&Yellow) | `tap_lhs_fence_green_yellow_2` |
| RHS Fence | `tap_rhs_fence_2` |
| Veg Trug | `tap_veg_trug` |

---

### Step 3 — Download the card YAML

Download [`releases/v0.1.0/card.yaml`](releases/v0.1.0/card.yaml) from this repository, or copy the raw YAML directly from that file.

---

### Step 4 — Adapt entity IDs

Open `card.yaml` in a text editor and do a **find and replace**:

- Find: `tap_lhs_lower_lawn_green`
- Replace: `your_actual_prefix`

You will need one copy of the card per valve, each with its own prefix.

---

### Step 5 — Add the card to your dashboard

**Option A — Via the UI editor**

1. Open your dashboard → Edit → Add card → **Manual card**
2. Delete the placeholder YAML
3. Paste the full contents of `card.yaml`
4. Save

**Option B — Via Claude (AI-assisted)**

Tell Claude:

> "Add the Zigbee water valve card for tap prefix [your_prefix] to my [dashboard name] dashboard, [tab name] view, [section name] section. Reference https://github.com/rhamblen/Zigbee-smart-water-valve for context."

Claude will need:
- Access to your Home Assistant via the HA MCP
- Your dashboard `url_path`, view index, and target section index
- The card YAML from this repository

See [docs/ai-context.md](docs/ai-context.md) for the full AI install brief.

---

### Step 6 — Verify

Open your dashboard. The card should show:

- A circular dial with your current flow rate (0.0 when valve is closed)
- CLOSED label and red border when the valve is off
- **OPEN** and **CLOSE** buttons that immediately toggle the valve
- ⚠ warning indicator dimly visible in the top-right (only turns amber if device is unavailable)

---

## Phase 2 — v0.2.0 (upcoming) — Consumption tracking

Phase 2 adds daily and monthly water totals. This requires creating HA helpers before installing the updated card.

### HA helpers to create

#### 1. Riemann sum integration (flow → cumulative litres)

In HA go to **Settings → Devices & Services → Helpers → Create helper → Integration — Riemann sum integral**.

Create one per valve:

| Field | Value |
|-------|-------|
| Name | e.g. `Tap LHS Lower Lawn total` |
| Input sensor | `sensor.tap_lhs_lower_lawn_green_flow` |
| Integration method | Left Riemann sum |
| Unit prefix | kilo |
| Unit of measurement | m³ |
| Max sub-interval | 00:01:00 |

This creates `sensor.tap_lhs_lower_lawn_total` (adjust name to match).

> **Why this is needed:** The Zigbee valve reports instantaneous flow rate (m³/h) but has no built-in cumulative counter. The integration helper accumulates it over time.

#### 2. Utility meters (daily and monthly reset)

In HA go to **Settings → Devices & Services → Helpers → Create helper → Utility meter**.

Create two per valve:

**Daily:**
| Field | Value |
|-------|-------|
| Name | e.g. `Tap LHS Lower Lawn daily` |
| Input sensor | `sensor.tap_lhs_lower_lawn_total` (from step 1) |
| Reset cycle | Daily |

**Monthly:**
| Field | Value |
|-------|-------|
| Name | e.g. `Tap LHS Lower Lawn monthly` |
| Input sensor | `sensor.tap_lhs_lower_lawn_total` (from step 1) |
| Reset cycle | Monthly |

Once created, install the v0.2.0 card YAML and update the entity references.

---

## Phase 3 — v0.3.0 (upcoming) — Countdown timer

Phase 3 adds the timer panel. Requires one HA timer helper per valve.

### Timer helper

In HA go to **Settings → Devices & Services → Helpers → Create helper → Timer**.

| Field | Value |
|-------|-------|
| Name | e.g. `Tap LHS Lower Lawn timer` |
| Duration | 00:30:00 (maximum, overridden by card) |

This creates `timer.tap_lhs_lower_lawn_timer`.

Phase 3 will also require an HA automation to close the valve when the timer finishes. A ready-made automation YAML will be included in the v0.3.0 release.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Red "Custom element doesn't exist" | html-template-card not installed | Install via HACS and reload |
| All values show `unknown` | Entity IDs don't match | Check entity prefix in HA device page |
| Warning light (⚠) always amber | Device offline or unavailable | Check Zigbee2MQTT bridge and device |
| OPEN/CLOSE buttons do nothing | JS blocked or entity_id mismatch | Verify prefix, check browser console |
| Flow shows 0.0 when valve is open | Device just opened or slow sensor | Wait 10–20 seconds; check Z2M device log |

---

## Updating to a newer version

1. Download the new `card.yaml` from the relevant release folder
2. In your dashboard editor, find the valve card and click Edit
3. Replace the content with the updated YAML
4. Apply your entity prefix find-replace again
5. Save

Or ask Claude:

> "Update my Zigbee water valve card for tap prefix [your_prefix] to the latest version from https://github.com/rhamblen/Zigbee-smart-water-valve"
