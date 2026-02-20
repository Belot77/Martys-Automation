# Changelog

All notable changes to the SigEnergy Solar Import & Export Control automation will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased]

### Fixed
- **Full Battery Surplus Detection Consistency** - Export and mode checks now both use uncurtailed solar potential when battery is full
  - Root cause: Early daytime `desired_export_limit` gate and `desired_ems_mode` fallback used measured `pv_kw`, which can be curtailed at 100% SoC
  - Impact: System could stay in `Maximum Self Consumption` and not raise export after sun intensity increased
  - Solution: At `battery_soc >= 99`, both branches now use `solar_potential_kw - load_kw` for surplus checks
  - Result: Export can ramp correctly during curtailment conditions while preserving below-99% behavior
  - Commit: 3246761

### Changed
- Solar Surplus Bypass thresholds are now fully configurable via blueprint inputs (enable/disable, start/continue multipliers)
- Added dedicated Solar Surplus Bypass section to blueprint UI
- Ongoing monitoring of full_battery_pv_export behavior

---

## [2.0.4] - 2026-02-13

### Fixed
- **Template Variable Order Error** - `productive_solar_end_ts` undefined causing 116 errors at 6:33am
  - Root cause: `morning_dump_active` (line 1865) referenced `productive_solar_end_ts` before it was defined (line 1967)
  - Forward reference in Jinja2 template caused UndefinedError when morning dump window opened
  - Solution: Moved `productive_solar_end_time` and `productive_solar_end_ts` definitions before `morning_dump_active`
  - Result: All template variables now defined before use, preventing runtime errors

---

## [2.0.3] - 2026-02-13

### Fixed
- **Morning Slow Charge Activating at Midnight** - Slow charge was enabled from midnight instead of morning
  - Root cause: Only checked time < noon, didn't verify it was actually morning
  - At 12:07am: Slow charge activated 7+ hours before sunrise, preventing overnight battery mode
  - Solution: Added two gating conditions:
    * Time check: Only after 5am
    * Sunrise proximity: Only within 2 hours of sunrise
  - Result: Slow charge now only activates when sunrise is approaching, not all night

---

## [2.0.2] - 2026-02-12

### Fixed
- **Status Message Threshold Mismatch** - Export status showed "blocked" while actually exporting
  - Root cause: Status checked `min_export_target_soc` (40%), control checked `effective_export_floor` (35% when boost)
  - At 40% battery with boost active: Status said "blocked below 35%", system actually exported 2.6kW
  - Solution: Status now uses `effective_export_floor` matching actual export control logic
  - Result: Status message now accurately reflects whether export is actually allowed/blocked

---

## [2.0.1] - 2026-02-12

### Fixed
- **Evening Export Boost Not Working** - Critical bug preventing evening export without PV surplus
  - Root cause: Line 2419 blocked export when `pv_surplus == 0` during "daytime"
  - At 6:50pm sun elevation 12.75° still registered as daytime, blocking export
  - Solution: Added `and not evening_export_boost_active` exception to allow boost export
  - Fixes: Evening export now allows aggressive discharge even after sunset when boost enabled
- **Template Syntax Error** - Comments using Python `#` syntax caused ValueError crashes
  - Root cause: Strategic comments at lines 2499-2502 output as literal text in Jinja2
  - Error: `float got invalid input '# PV Surplus Cap: ... 6.0'`
  - Solution: Changed to proper Jinja2 comment syntax `{# comment #}`
  - Affects: PV surplus cap comment block (lines 2499-2502)

### Documentation
- Added troubleshooting section for blueprint reload issues
- Added troubleshooting for template variable errors
- Documented requirement to restart automation after blueprint updates

---

## [2.0.0] - 2026-02-12

### Added
- **Configurable Forecast Safety Margins** - Two grouped settings replace 7+ hardcoded margins
  - `forecast_safety_charging` (default 1.25): Conservative for charge/dump decisions
  - `forecast_safety_export` (default 1.1): Less conservative for export decisions
  - Applied to: morning slow charge, morning dump, standby holdoff, export guards, daytime top-up
- **Morning Dump Forecast Protection** - Dump now verifies sufficient solar to refill battery
  - Calculates total PV from dump end until productive solar ends
  - Prevents dumping when forecast insufficient (avoids expensive grid import)
  - Consistent with morning slow charge forecast logic
- **Full Battery PV Export Tracking** - New `full_battery_pv_export` variable
  - Identifies when battery ≥99% with daytime PV surplus
  - Enables clearer status messages

### Fixed
- **Solar Bypass Export Blocking** - Critical bug where excellent forecast (93kWh) couldn't export
  - Root cause: `export_tier_limit` blocked at 40% battery before bypass logic ran
  - Solution: Moved `solar_surplus_bypass` calculation before `export_tier_limit`
  - Added bypass exception to `min_export_target_soc` check
  - Commit: 304e147
- **Full Battery Solar Curtailment** - Battery at 100% causing PV to curtail (1kW actual vs 15kW potential)
  - Root cause: Chicken-and-egg loop - no export → PV curtailed → no surplus → no export
  - Solution: Use `solar_potential_kw` (uncurtailed) when battery ≥99% to calculate export limit
  - Prevents battery discharge while allowing full PV export
  - Commit: 40a5701

### Changed
- **Status Message Clarity** - Distinguished full battery export from tier-based limits
  - Was: "Exporting, Low tier 25.0kW @ 0.05c" (misleading - 25kW is not Low tier limit)
  - Now: "Exporting, Full battery 11.5kW @ 0.05c" (clear - shows PV surplus and reason)
  - Shows actual PV surplus (solar - load) instead of tier limit when battery full
  - Commit: 9d1c144
- **Improved Mode Status Messages** - Made special mode messages clearer
  - "dump active" → "morning dump"
  - "holdoff" → "charge holdoff"  
  - "conditions" → "low forecast"
  - Fixed FIT sensor unavailable handling (prevents "-999" display)
  - Commit: e018902

### Documentation
- **Attribution Added** - Credited Martin Pascoe as original author
  - Core automation logic and EMS control framework
  - Forecast-based optimization and price-responsive scheduling
  - Battery management and export control algorithms
  - Commit: 59308f3
- **Strategic Code Comments** - Added comments explaining complex logic
  - `solar_surplus_bypass`: 2×/1.25× battery threshold explanation
  - `full_battery_pv_export`: Emergency valve to prevent curtailment
  - `export_tier_limit`: Tier selection and override logic
  - PV surplus cap: Why `solar_potential_kw` vs `pv_kw` when battery full
  - Forecast safety margins: Consequence-based margin selection
  - Morning dump: Forecast protection calculation
  - Commit: 76b59ac
- **TODO.md Updated** - Comprehensive update for February 2026 changes
  - Moved 6 completed items to bottom with detailed descriptions
  - Updated pending items to reflect current code state
  - Commit: 76b59ac
- **README.md Created** - Comprehensive installation and configuration guide
  - Installation instructions with blueprint import
  - Configuration guide for all major settings
  - Status message reference
  - Troubleshooting guide
- **CHANGELOG.md Created** - Version history tracking

---

## [1.1.0] - 2026-02-11

### Fixed
- **FREE Import Grid Capacity** - Fixed max() logic to respect grid capacity limit
  - Changed line 2240 from using max() to `cap_total_import` only
  - Prevents exceeding grid import capability during free/negative pricing
- **Import Reason Priority** - Fixed demand window blocking not showing in status
  - Reordered checks to prioritize demand window message
- **Dead Code Removal** - Removed unreachable `elif price_is_negative` branch in import limit logic
- **Division by Zero Protection** - Added minimum 0.1 for export SoC span calculation
- **Positive FIT Override** - Fixed to allow battery discharge at positive FIT
  - Removed daytime PV surplus check that was blocking valid discharge

### Changed
- **Increased Character Limits** - Reason text fields expanded from 95 to 250 characters
  - Allows more detailed status messages without truncation
- **Evening Mode Timing** - Made start time configurable
  - New setting: `evening_mode_hours_before_sunset` (default 1.0 hours)
  - Previously hardcoded to 1 hour before sunset

---

## [1.0.0] - Initial Release

### Features
- Core battery management with forecast-based optimization
- Three-tier export control system (Low/Medium/High)
- Price-responsive import control
- Morning slow charge and morning dump modes
- Evening export boost with aggressive floor
- Standby holdoff (charge blocking when forecast sufficient)
- Solar surplus bypass for excellent forecast days
- PV safeguard to protect battery from over-exporting
- Demand window import blocking
- Cheap import daytime top-up
- Comprehensive status messaging
- Session tracking for import/export monitoring

### Integrations
- SigEnergy EMS control
- Solcast solar forecasting
- Amber Electric dynamic pricing

---

## Pending Development

See [TODO.md](TODO.md) for:
- Monitoring solar_surplus_bypass threshold behavior
- Monitoring full_battery_pv_export emergency valve
- Reviewing threshold values based on real-world usage

---

**Original Author:** Martin Pascoe  
**Repository:** https://github.com/Belot77/Martys-Automation  
**Maintained with community contributions**
