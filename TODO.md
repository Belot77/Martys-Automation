# TODO - Home Assistant Automation Improvements

## Pending Changes

### SigEnergy Optimiser Enhancements

- [ ] **Review solar_surplus_bypass thresholds**
  - Requires 2× battery capacity forecast to START bypass (e.g., 80kWh for 40kWh battery)
  - Only 1.25× capacity to CONTINUE once active (50kWh)
  - High thresholds mean this activates only with excellent solar forecasts
  - Monitor: Does starting threshold need adjustment based on local conditions?

- [ ] **Monitor full_battery_pv_export behavior**
  - At 99% SoC with PV surplus, allows high export limit regardless of FIT tier
  - Prevents solar curtailment when battery full
  - Uses solar_potential_kw (uncurtailed) in both export gating and mode fallback surplus checks when battery is full
  - Monitor: Verify it doesn't discharge battery unnecessarily

### Known Issues

- [ ] **amber_balance duplicate unique IDs**
  - Two sensors using same unique_id suffix "_v2_position"
  - AmberBalanceSensor and AmberMetricSensor conflict
  - Location: `custom_components/amber_balance/`

---

## Completed Changes

### February 2026 - Major Enhancements

- ✅ **Fixed evening export boost blocking bug** - 2026-02-12
  - Evening export was blocked when PV surplus = 0 even with boost enabled
  - At 6:50pm with sun elevation 12.75°, still considered "daytime" and blocked
  - Added `and not evening_export_boost_active` exception to line 2419
  - Allows aggressive evening export even without PV surplus when tomorrow's forecast sufficient

- ✅ **Fixed template comment syntax error** - 2026-02-12
  - Strategic comments using Python `#` syntax output as literal text
  - Caused ValueError: "float got invalid input '# PV Surplus Cap: ... 6.0'"
  - Changed to Jinja2 comment syntax `{# comment #}` at lines 2499-2502
  - Prevents automation crashes from template rendering errors

- ✅ **Added attribution to Martin Pascoe as original author** - 2026-02-12
  - Credited for core automation logic, EMS control framework, and optimization algorithms
  
- ✅ **Improved export status message clarity** - 2026-02-12
  - Distinguished between tier-based export limits (5/10/25kW) and full battery PV export
  - Added `full_battery_pv_export` tracking variable
  - Shows "Full battery 11.5kW" instead of misleading "Low tier 25kW"

- ✅ **Fixed solar bypass export blocking bug** - 2026-02-12
  - Solar bypass with 93kWh forecast was blocked by export_tier_limit at 40% battery
  - Moved solar_surplus_bypass calculation before export_tier_limit
  - Added bypass exception to min_export_target_soc check
  - Commit: 304e147

- ✅ **Fixed full battery solar curtailment** - 2026-02-12
  - Battery at 100% was causing PV curtailment loop (1kW actual vs 15kW potential)
  - Changed desired_export_limit to use solar_potential_kw when battery ≥99%
  - Prevents battery discharge while allowing full PV export
  - Commit: 40a5701

- ✅ **Implemented configurable forecast safety margins** - 2026-02-12
  - Replaced 7+ hardcoded margins (1.1×, 1.25×) with two grouped settings
  - `forecast_safety_charging` (default 1.25): Conservative for charge/dump decisions
  - `forecast_safety_export` (default 1.1): Less conservative for export decisions
  - Applied to: morning slow charge, morning dump, standby holdoff, export guards, daytime top-up
  - Commit: 40a5701

- ✅ **Added morning dump forecast protection** - 2026-02-12
  - Morning dump now checks if sufficient solar forecast to refill battery
  - Prevents dumping when forecast insufficient, avoiding expensive grid import
  - Consistent with morning slow charge forecast logic
  - Commit: 40a5701

- ✅ **Improved status message clarity for special modes** - 2026-02-11
  - "dump active" → "morning dump"
  - "holdoff" → "charge holdoff"
  - "conditions" → "low forecast"
  - Fixed FIT sensor unavailable handling (prevents "-999" display)
  - Commit: e018902

### Earlier Improvements

- ✅ Fixed FREE import logic to respect grid capacity limit (line 2240: `max()` → `cap_total_import` only) - 2026-02-11
- ✅ Removed unreachable dead code in import limit logic (`elif price_is_negative` branch) - 2026-02-11
- ✅ Increased reason text character limits (95→250 chars) - 2026-02-11
- ✅ Fixed import reason priority to show demand window blocking - 2026-02-11
- ✅ Made evening mode start time configurable (`evening_mode_hours_before_sunset`, default 1.0 hours) - 2026-02-11
- ✅ Fixed `positive_fit_override` to allow battery discharge at positive FIT (removed daytime PV surplus check) - 2026-02-11
- ✅ Added division by zero protection for export SoC span calculation (minimum 0.1) - 2026-02-11

---

*Add new items above this line. Move completed items to the bottom section.*
