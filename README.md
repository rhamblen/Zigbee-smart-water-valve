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
| v1.0.1 | ✅ | Pool fill auto-reopen — valve restarts automatically if closed unexpectedly mid-fill |
| v1.1.0 | ✅ | Documented scheduler integration contract (timer + finished automation) — docs only, no card change |
| v1.1.1 | ✅ | Fix water meter fault indicator false-positive (no_alarm state now correctly shown as grey) |

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

## Pool fill mode

Pool fill mode turns the card into a volume-controlled fill controller — ideal for topping up a swimming pool or large tank overnight without manual monitoring.

### How to use

1. Tap **🏊** in the card header — the pool fill section expands
2. Tap the **Target** row and enter your target volume in litres (e.g. 8 500 L)
3. Open the valve with **OPEN** or a timer preset
4. The card shows:
   - **Target** — your set volume (tap to edit)
   - **Progress bar** — cyan fill showing % complete
   - **Filled this session** — litres pumped since pool mode was activated
   - **Remaining** — litres still needed
   - **Valve open for** — elapsed time since valve opened
5. When the target is reached, an HA automation closes the valve automatically
6. Tap 🏊 again to dismiss the pool section

### Overnight fill behaviour (auto-reopen)

For long fills (8–10 hours), three automations work together:

| Automation | Purpose |
|-----------|---------|
| **Pool fill cutoff** | Closes valve when target volume is reached |
| **Pool fill auto-reopen** | Reopens valve if closed unexpectedly before target is reached (firmware glitch, Zigbee dropout, HA restart) — checks after a 30-second delay |
| **Close valve on timer** | Closes valve when a countdown timer preset expires |

The auto-reopen automation distinguishes a legitimate cutoff (target reached → `session >= target` still true after 30 s → valve stays closed) from an unexpected close (`session < target` → valve reopens).

### How to stop a fill safely

1. Tap **🏊** to deactivate pool mode
2. Close the valve with **CLOSE** (optional — pool mode off is enough)

> Closing the valve while pool mode is still active will trigger the auto-reopen after 30 seconds. Always disable pool mode first.

### Session tracking

Session volume is computed as `filled = current_accumulator − start_accumulator`. The start accumulator is recorded automatically when you activate pool mode. Deactivating and reactivating pool mode (tapping 🏊 twice) starts a new session from zero.

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

## Scheduler integration contract

This card exposes a small, stable contract that external schedulers — e.g.
[Garden Watering Scheduler mk2](https://github.com/rhamblen/garden-watering-scheduler-mk2)
— use to **offload** a timed valve run to this card's own countdown timer instead of
managing the close themselves.

| Contract | Entity / behaviour |
|----------|-------------------|
| Per-valve countdown timer | `timer.{prefix}_timer` (`restore: true`) |
| Auto-close on finish | `automation.close_tap_when_timer_finishes_{prefix}` — triggers on `timer.finished` for that timer, calls `switch.turn_off` on `switch.{prefix}` |

A scheduler opens `switch.{prefix}`, calls `timer.start` on `timer.{prefix}_timer` with
the desired duration, then waits for the valve to report `off`. This card's
`timer.finished` automation performs the close — and the **volume safety cutoff** (pool
fill mode) will close the valve too if a litre target is hit first, so the scheduler
inherits that protection for free.

**The naming convention is the interface:** `switch.{prefix}` ⇒ `timer.{prefix}_timer`.
Renaming the timer or removing the finished automation will break schedulers that
depend on it — treat them as a public API. These objects have existed since **v0.3.0**;
**v1.1.0** only documents them as a contract (no functional change to the card).

> Schedulers that need plain on/off valves with no timer should use
> [Garden Watering Scheduler mk1](https://github.com/rhamblen/garden-watering-scheduler),
> which self-manages the watering delay.

---

## Changelog

See [CHANGELOG.md](CHANGELOG.md)

---

## Licence

MIT — free to use, adapt, and share. Attribution appreciated.
