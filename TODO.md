# TODO - Home Assistant Automation Improvements

## Pending Changes

### SigEnergy Optimiser Enhancements

- [ ] **Fix FREE import logic bug (line 2240)**
  - Uses `max(cap_total_import, ess_max_charge_kw)` for FREE power (price = 0)
  - Should just use `cap_total_import` to respect user's grid capacity limit
  - Battery charge rate is controlled separately by `desired_ess_charge_limit`
  - Impact: Could exceed configured grid import limits during FREE power periods

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
- ✅ Fixed `positive_fit_override` to allow battery discharge at positive FIT (removed daytime PV surplus check) - 2026-02-11
- ✅ Added division by zero protection for export SoC span calculation (minimum 0.1) - 2026-02-11

---

*Add new items above this line. Move completed items to the bottom section.*
