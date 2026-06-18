# Installation Guide

**Current version:** v1.0.0  
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

You need **eight helpers** (five per valve, plus three shared pool fill helpers). Create them at **Settings → Devices & Services → Helpers → Create helper**.

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

##### 1f. Pool fill target (shared — create once, not per valve)

Helper type: **Number**

| Field | Value |
|-------|-------|
| Name | `Pool fill target` |
| Minimum | `0` |
| Maximum | `20000` |
| Step size | `50` |
| Unit of measurement | `L` |
| Display mode | Box |
| Icon | `mdi:pool` |

This creates `input_number.pool_fill_target`. Set your target volume here before starting a pool fill.

---

##### 1g. Pool fill start volume (shared — create once, not per valve)

Helper type: **Number**

| Field | Value |
|-------|-------|
| Name | `Pool fill start volume` |
| Minimum | `0` |
| Maximum | `1000000` |
| Step size | `0.001` |
| Unit of measurement | `L` |
| Display mode | Box |
| Icon | `mdi:water-sync` |

This creates `input_number.pool_fill_start_volume`. Updated automatically when pool mode is activated — do not edit manually.

---

##### 1h. Pool fill active toggle (per valve)

Helper type: **Toggle**

| Field | Value |
|-------|-------|
| Name | `Pool Fill Active [valve name]` |
| Icon | `mdi:pool` |

For the LHS Lower Lawn Green valve: `Pool Fill Active LHS Lower Lawn Green` → creates `input_boolean.pool_fill_active_lhs_lower_lawn_green`.

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

#### Step 2b — Create the pool fill cutoff automation

Go to **Settings → Automations → Create automation → Start with an empty automation**.

Or paste this YAML into the automation editor (three-dot menu → Edit as YAML):

```yaml
alias: Pool fill cutoff (LHS Lower Lawn Green)
description: Closes valve when pool fill target volume is reached
trigger:
  - platform: template
    value_template: >
      {% set session = [0, states('sensor.tap_lhs_lower_lawn_green_volume')|float(0) -
                       states('input_number.pool_fill_start_volume')|float(0)]|max %}
      {% set target = states('input_number.pool_fill_target')|float(0) %}
      {{ target > 0 and session >= target }}
condition:
  - condition: state
    entity_id: switch.tap_lhs_lower_lawn_green
    state: 'on'
action:
  - action: switch.turn_off
    target:
      entity_id: switch.tap_lhs_lower_lawn_green
mode: single
```

---

#### Step 3 — Download and adapt the card YAML

1. Download [`releases/v1.0.0/card.yaml`](releases/v1.0.0/card.yaml) from this repository
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
- **🏊 Pool mode button** in the top-right header — tap to activate pool fill mode; shows Target / progress bar / Filled / Remaining / elapsed time

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

Repeat Steps 1–4 for each additional valve, substituting that valve's entity prefix throughout. Each valve needs its own set of 5 sensor/timer helpers (steps 1a–1e), its own `input_boolean.pool_fill_active_{prefix}` (step 1h), its own two automations (steps 2 and 2b), and its own card instance. The two `input_number` pool fill helpers (steps 1f–1g) are shared — create them once only.

---

## Updating to a newer version

1. Download `releases/v1.0.0/card.yaml` (or whichever new version)
2. Apply your entity prefix find-replace
3. Open your dashboard editor → find the valve card → Edit → replace the content → Save
