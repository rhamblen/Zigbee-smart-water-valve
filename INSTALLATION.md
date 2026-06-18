# Installation Guide

**Current version:** v0.3.0  
**Repository:** https://github.com/rhamblen/Zigbee-smart-water-valve

---

## Choose your installation method

### Option A — Claude-assisted (recommended)

No manual YAML or helper setup. Claude reads the repository and configures everything for you.

**What you need first:**
- HACS installed in Home Assistant
- `custom:html-template-card` installed via HACS → Frontend → search "HTML Template Card"
- Claude with Home Assistant MCP connected

**Then tell Claude:**

> "Help me install the Zigbee Smart Water Valve card from https://github.com/rhamblen/Zigbee-smart-water-valve — my valve entity prefix is `tap_lhs_lower_lawn_green`. Add it to my [dashboard name] dashboard."

Claude will:
1. Read the card YAML from the repository
2. Create all required HA helpers (template sensor, integration, utility meters, timer)
3. Create the automation that closes the valve when the timer expires
4. Deploy the card to your dashboard

Replace `tap_lhs_lower_lawn_green` with your actual prefix (see [Finding your entity prefix](#finding-your-entity-prefix) below).

---

### Option B — Manual installation

#### Prerequisites

| Requirement | How to get it |
|-------------|--------------|
| HACS | https://hacs.xyz |
| `custom:html-template-card` | HACS → Frontend → search "HTML Template Card" |
| Zigbee valve paired via Zigbee2MQTT | Settings → Devices & Services → Zigbee2MQTT |

---

#### Finding your entity prefix

Go to **Settings → Devices & Services → your valve device**. You'll see two entities:

| Entity | Example |
|--------|---------|
| `switch.{prefix}` | `switch.tap_lhs_lower_lawn_green` |
| `sensor.{prefix}_flow` | `sensor.tap_lhs_lower_lawn_green_flow` |

Your prefix is everything after `switch.` — e.g. `tap_lhs_lower_lawn_green`.

---

#### Step 1 — Create HA helpers

You need **five helpers per valve**. Create them at **Settings → Devices & Services → Helpers → Create helper**.

> **Important:** Create them in order — each one feeds into the next.

---

##### 1a. Template sensor (converts m³/h → L/h)

Helper type: **Template → Sensor**

| Field | Value |
|-------|-------|
| Name | `Tap [name] flow L/h` |
| State template | `{{ states('sensor.tap_lhs_lower_lawn_green_flow') \| float(0) * 1000 }}` |
| Unit of measurement | `L/h` |
| Device class | (leave blank) |

Replace `tap_lhs_lower_lawn_green` with your prefix. This creates `sensor.tap_lhs_lower_lawn_green_flow_l_h`.

---

##### 1b. Integration sensor (accumulates litres)

Helper type: **Integration — Riemann sum integral**

| Field | Value |
|-------|-------|
| Name | `Tap [name] volume` |
| Input sensor | `sensor.tap_lhs_lower_lawn_green_flow_l_h` (from step 1a) |
| Integration method | Left Riemann sum |
| Unit time | Hours |
| Max sub-interval | 00:01:00 |

This creates `sensor.tap_lhs_lower_lawn_green_volume` (accumulates litres; resets to 0 on HA restart).

---

##### 1c. Daily utility meter

Helper type: **Utility meter**

| Field | Value |
|-------|-------|
| Name | `Tap [name] daily` |
| Input sensor | `sensor.tap_lhs_lower_lawn_green_volume` (from step 1b) |
| Reset cycle | Daily |

This creates `sensor.tap_lhs_lower_lawn_green_daily` — resets at midnight.

---

##### 1d. Monthly utility meter

Helper type: **Utility meter**

| Field | Value |
|-------|-------|
| Name | `Tap [name] monthly` |
| Input sensor | `sensor.tap_lhs_lower_lawn_green_volume` (from step 1b) |
| Reset cycle | Monthly |

This creates `sensor.tap_lhs_lower_lawn_green_monthly` — resets on the 1st of each month.

---

##### 1e. Timer helper

Helper type: **Timer**

| Field | Value |
|-------|-------|
| Name | `Tap [name] timer` |
| Duration | `00:30:00` (maximum; individual presets override this) |
| Restore | enabled |

This creates `timer.tap_lhs_lower_lawn_green_timer`. The `Restore` setting ensures the timer survives HA restarts.

> **Note:** HA may append `_timer` to the entity ID if the name ends in "timer" — check Settings → Helpers to confirm the actual entity ID.

---

#### Step 2 — Create the close-valve automation

Go to **Settings → Automations → Create automation → Start with an empty automation**.

Add:

- **Trigger:** Event → Event type: `timer.finished`, Event data: `entity_id: timer.tap_lhs_lower_lawn_green_timer`
- **Action:** Call service → `switch.turn_off` → Entity: `switch.tap_lhs_lower_lawn_green`

Or paste this YAML into the automation editor (three-dot menu → Edit as YAML):

```yaml
alias: Close tap when timer finishes (LHS Lower Lawn Green)
trigger:
  - platform: event
    event_type: timer.finished
    event_data:
      entity_id: timer.tap_lhs_lower_lawn_green_timer
action:
  - action: switch.turn_off
    target:
      entity_id: switch.tap_lhs_lower_lawn_green
mode: single
```

---

#### Step 3 — Download and adapt the card YAML

1. Download [`releases/v0.3.0/card.yaml`](releases/v0.3.0/card.yaml) from this repository
2. Open it in a text editor
3. Find-and-replace **all** occurrences of `tap_lhs_lower_lawn_green` with your actual prefix

You will need one adapted copy per valve.

---

#### Step 4 — Add the card to your dashboard

1. Open your Lovelace dashboard → click the pencil (edit) icon
2. Click **+ Add card** in the target section
3. Scroll to the bottom and choose **Manual card**
4. Delete the placeholder YAML and paste the full contents of your adapted `card.yaml`
5. Click **Save**

---

#### Step 5 — Verify

The card should show:

- **Circular dial** — live flow rate in L/min (0.0 when valve is closed)
- **CLOSED / OPEN** state and matching border colour
- **OPEN / CLOSE buttons** that immediately toggle the valve
- **Stats bar** — Flow | Today | This month (Today and This month will show `—` until the helpers produce their first reading)
- **⏱ Timer panel** — preset buttons that open the valve and start a countdown; auto-closes valve when time expires
- **🏊 Pool mode button** in the top-right header (hidden section; tap to toggle)

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| "Custom element doesn't exist" error | `html-template-card` not installed | Install via HACS and reload browser |
| All values show `unknown` or `unavailable` | Entity IDs don't match | Check the entity prefix — go to Settings → Devices → your valve |
| Today / This month shows `—` permanently | Helpers not created or wrong entity IDs | Verify all 4 sensor helpers exist and IDs match the card |
| Timer buttons do nothing | `html-template-card` or browser issue | Hard-reload the browser (Ctrl+Shift+R); check browser console for errors |
| Warning ⚠ always amber | Device offline | Check Zigbee2MQTT bridge status |
| Valve doesn't close when timer expires | Automation missing or wrong trigger | Check the automation is enabled and the entity ID matches |

---

## Adding more valves

Repeat Steps 1–4 for each additional valve, substituting that valve's entity prefix throughout. Each valve needs its own set of 5 helpers, its own automation, and its own card instance.

---

## Updating to a newer version

1. Download the new `card.yaml` from the relevant `releases/vX.Y.Z/` folder
2. Apply your entity prefix find-replace
3. Open your dashboard editor → find the valve card → Edit → replace the content → Save
