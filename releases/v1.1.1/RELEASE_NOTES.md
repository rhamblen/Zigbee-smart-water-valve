# v1.1.1 — 2026-06-27

## Bug fix: water meter fault indicator false-positive

The ⚠ symbol on the water meter tile was glowing yellow even when there were no faults.

**Root cause:** the `has_faults` guard in the water meter card template did not include `no_alarm` in its exclusion list. The `sensor.garden_water_meter_faults` entity reports `no_alarm` when healthy, but the card only excluded `none`, `None`, `unknown`, `unavailable`, and `''` — so `no_alarm` was incorrectly treated as a fault condition.

**Fix:** `no_alarm` is now included in the exclusion list. The indicator is grey when the sensor state is `no_alarm` (or any other non-fault value) and amber only when a real alarm is present.

## No other changes

Card YAML, automations, helpers, and all other dashboard cards are unchanged from v1.1.0.
