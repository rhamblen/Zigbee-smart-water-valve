# v1.1.0 — Scheduler integration contract (docs only)

This release **documents an existing capability** as a stable public interface. There
is **no change to the card** — `card.yaml` is identical to v1.0.1.

## What's documented

The per-valve countdown timer and its auto-close automation (both shipped in **v0.3.0**)
are now declared as a contract that external schedulers can rely on to **offload** a
timed valve run to this card:

| Contract | Entity / behaviour |
|----------|-------------------|
| Per-valve countdown timer | `timer.{prefix}_timer` (`restore: true`) |
| Auto-close on finish | `automation.close_tap_when_timer_finishes_{prefix}` — `timer.finished` → `switch.turn_off` on `switch.{prefix}` |

A scheduler opens `switch.{prefix}`, calls `timer.start` on `timer.{prefix}_timer`, and
waits for the valve to report `off`. This card's finished automation performs the close,
and the pool-fill **volume safety cutoff** will close the valve too if a litre target is
reached first — so the scheduler inherits volume protection for free.

**Naming convention is the interface:** `switch.{prefix}` ⇒ `timer.{prefix}_timer`.

## Consumer

[Garden Watering Scheduler mk2](https://github.com/rhamblen/garden-watering-scheduler-mk2)
uses this contract to schedule watering without managing valve timing itself. (Plain
on/off valves with no timer should use
[mk1](https://github.com/rhamblen/garden-watering-scheduler).)

## Upgrade

No action required. If you already run v0.3.0+, the contract is already satisfied.
