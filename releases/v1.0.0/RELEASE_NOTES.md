# v1.0.0 — Pool Fill Mode

## What's new

Pool fill mode lets you fill a pool (or tank) to a precise litre target. The valve closes automatically when the target is reached.

### How it works

1. Tap **🏊** in the card header to activate pool mode
2. Tap the **Target** row and set your fill volume in litres (e.g. 8 500 L for a pool top-up)
3. Open the valve with OPEN or a timer preset
4. Watch the progress bar fill — **Filled this session** and **Remaining** update on every HA re-render
5. When the target is reached, the HA automation closes the valve automatically
6. Tap 🏊 again to dismiss the pool section

### Pool section display

| Row | Shows |
|-----|-------|
| Target | Volume target in L — tap to edit |
| Progress bar | Cyan gradient fill; appears once target is set |
| Filled this session | Litres used since 🏊 was tapped |
| Remaining | Litres left to reach target |
| Valve open for | Elapsed time since valve opened |

## HA helpers required (in addition to v0.3.0 helpers)

| Helper | Type | Entity ID |
|--------|------|-----------|
| Pool fill target | `input_number` | `input_number.pool_fill_target` |
| Pool fill start volume | `input_number` | `input_number.pool_fill_start_volume` |
| Pool fill active | `input_boolean` | `input_boolean.pool_fill_active_{prefix}` |

## Automation required

Create via **Settings → Automations → New automation**:

- **Trigger:** Template — value template:
  ```
  {% set session = [0, states('sensor.{prefix}_volume')|float(0) - states('input_number.pool_fill_start_volume')|float(0)]|max %}
  {% set target = states('input_number.pool_fill_target')|float(0) %}
  {{ target > 0 and session >= target }}
  ```
- **Condition:** State — `switch.{prefix}` is `on`
- **Action:** Call service `switch.turn_off` → `switch.{prefix}`

## Adapting for other valves

Replace all occurrences of `tap_lhs_lower_lawn_green` with your valve's entity prefix.

Note: `input_number.pool_fill_target` and `input_number.pool_fill_start_volume` are shared across valves. Create one `input_boolean.pool_fill_active_{prefix}` and one pool fill cutoff automation per valve.
