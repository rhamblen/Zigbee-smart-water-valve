# v1.0.1 — Pool fill auto-reopen

## What's new

### Automatic valve reopen during pool fill

For long overnight fills (8–10 hours), the valve now reopens automatically if it closes unexpectedly before the target volume is reached. Unexpected closes include firmware glitches, Zigbee dropouts, and HA restarts.

**No card change** — this is a new HA automation only. The card YAML from v1.0.0 is unchanged.

---

## How it works

When the valve closes while pool mode is active:

1. A 30-second delay begins (gives the Riemann sum sensor time to settle)
2. Session volume is rechecked: `session = current_accumulator − start_accumulator`
3. If `session < target` → valve reopens (unexpected close)
4. If `session >= target` → valve stays closed (target was legitimately reached)

The same logic runs on HA startup, so a fill interrupted by an HA restart resumes automatically.

---

## Safe stop

To intentionally stop a pool fill:

1. **Tap 🏊** in the card header to disable pool mode
2. Close the valve (or leave it — pool mode off is sufficient)

> If you close the valve while pool mode is still active, the auto-reopen automation will reopen it after 30 seconds. Always disable pool mode first.

---

## Automation required (per valve)

Create via **Settings → Automations → New automation → Edit as YAML**:

```yaml
alias: Pool fill auto-reopen (LHS Lower Lawn Green)
description: Reopens valve if closed unexpectedly during active pool fill before target is reached
trigger:
  - platform: state
    entity_id: switch.tap_lhs_lower_lawn_green
    from: 'on'
    to: 'off'
  - platform: homeassistant
    event: start
condition:
  - condition: state
    entity_id: input_boolean.pool_fill_active_lhs_lower_lawn_green
    state: 'on'
action:
  - delay: "00:00:30"
  - condition: template
    value_template: >
      {% set session = [0, states('sensor.tap_lhs_lower_lawn_green_volume')|float(0) -
                       states('input_number.pool_fill_start_volume')|float(0)]|max %}
      {% set target = states('input_number.pool_fill_target')|float(0) %}
      {{ target > 0 and session < target }}
  - action: switch.turn_on
    target:
      entity_id: switch.tap_lhs_lower_lawn_green
mode: restart
```

Replace `tap_lhs_lower_lawn_green` with your valve's entity prefix. Create one automation per valve.

---

## No helpers required

This release adds no new HA helpers. All required helpers were introduced in v1.0.0.
