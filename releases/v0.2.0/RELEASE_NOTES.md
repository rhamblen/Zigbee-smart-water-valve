# Release notes — v0.2.0

**Released:** 2026-06-18

## What's new

- **Daily consumption stat** — "Today" stat at bottom of card shows litres used since midnight. Tap to open HA history for the daily utility meter.
- **Monthly consumption stat** — "This month" stat shows cumulative litres this calendar month. Tap for monthly history.
- **Stats bar dividers** — thin vertical lines separate the three stat cells (Flow / Today / This month).
- **Pool mode: Today's usage** — pool section now shows today's total as a real-time proxy for session volume.

## HA helpers created (LHS Lower Lawn Green only)

Three helpers were created via the HA Helpers UI:

| Helper | Entity ID | Purpose |
|--------|-----------|---------|
| Integration (Riemann sum) | `sensor.garden_tap_lhs_lower_lawn_green_tap_lhs_lower_lawn_green_volume` | Converts m³/h flow → cumulative m³ |
| Utility meter — daily | `sensor.garden_tap_lhs_lower_lawn_green_tap_lhs_lower_lawn_green_daily` | Resets at midnight |
| Utility meter — monthly | `sensor.garden_tap_lhs_lower_lawn_green_tap_lhs_lower_lawn_green_monthly` | Resets on 1st of month |

Note: the `garden_` prefix in entity IDs comes from the HA area assigned to the device. When setting up other valves, check the generated entity IDs and update the card accordingly.

## Upgrade from v0.1.1

1. Create the three helpers above (or equivalents for your valve prefix).
2. Replace the `content:` value in your card with the content from `releases/v0.2.0/card.yaml`.
3. Update the three helper entity ID strings inside the card to match what HA generated for your helpers.

## Notes

- The consumption stats will show `—` until the integration sensor has accumulated some data. This is normal on first install — run the valve briefly and the sensor will populate.
- Today's value resets at midnight. Monthly value resets on the 1st.
- Volume-based safety cutoff for pool mode is planned for v1.0.0.
