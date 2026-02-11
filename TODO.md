# TODO - Home Assistant Automation Improvements

## Pending Changes

### SigEnergy Optimiser Enhancements

- [ ] **Monitor 99% battery emergency valve behavior (line 2056)**
  - At 99% SoC, allows HIGH export tier with ANY positive FIT price (even 0.01¢)
  - Bypasses all normal tier logic, forecast guards, and SoC checks
  - May be intentional emergency relief valve to prevent overcharge
  - Monitor in practice: does it trigger inappropriately at low FIT prices?

### Known Issues

- [ ] **amber_balance duplicate unique IDs**
  - Two sensors using same unique_id suffix "_v2_position"
  - AmberBalanceSensor and AmberMetricSensor conflict
  - Location: `custom_components/amber_balance/`

---

## Completed Changes

- ✅ Fixed FREE import logic to respect grid capacity limit (line 2240: `max()` → `cap_total_import` only) - 2026-02-11
- ✅ Removed unreachable dead code in import limit logic (`elif price_is_negative` branch) - 2026-02-11
- ✅ Increased reason text character limits (95→250 chars) - 2026-02-11
- ✅ Fixed import reason priority to show demand window blocking - 2026-02-11
- ✅ Made evening mode start time configurable (`evening_mode_hours_before_sunset`, default 1.0 hours) - 2026-02-11
- ✅ Fixed `positive_fit_override` to allow battery discharge at positive FIT (removed daytime PV surplus check) - 2026-02-11
- ✅ Added division by zero protection for export SoC span calculation (minimum 0.1) - 2026-02-11

---

*Add new items above this line. Move completed items to the bottom section.*
