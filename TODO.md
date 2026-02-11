# TODO - Home Assistant Automation Improvements

## Pending Changes

### SigEnergy Optimiser Enhancements

### Known Issues

- [ ] **amber_balance duplicate unique IDs**
  - Two sensors using same unique_id suffix "_v2_position"
  - AmberBalanceSensor and AmberMetricSensor conflict
  - Location: `custom_components/amber_balance/`

---

## Completed Changes

- ✅ Increased reason text character limits (95→250 chars) - 2026-02-11
- ✅ Fixed import reason priority to show demand window blocking - 2026-02-11
- ✅ Made evening mode start time configurable (`evening_mode_hours_before_sunset`, default 1.0 hours) - 2026-02-11

---

*Add new items above this line. Move completed items to the bottom section.*
