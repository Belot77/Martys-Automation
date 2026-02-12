# SigEnergy Solar Import & Export Control

**Original Author:** Martin Pascoe  
**Repository:** Enhanced and maintained with community contributions

Intelligent battery-preserving automation for SigEnergy EMS that prioritizes having enough battery to reach sunrise, avoids paid energy where possible, and maximizes earnings from export/import opportunities.

---

## Features

### Core Functionality
- **Forecast-Based Optimization**: Uses Solcast solar forecasts to make intelligent charging/exporting decisions
- **Price-Responsive Control**: Dynamic import/export limits based on Amber Energy pricing
- **Battery Preservation**: Ensures sufficient charge to reach sunrise, avoiding expensive grid imports
- **Configurable Safety Margins**: Separate margins for charging vs export decisions based on consequence risk

### Operating Modes

#### Morning Management
- **Morning Slow Charge**: Gentle top-up before sunrise using cheap overnight rates (with forecast verification)
- **Morning Dump**: Export battery at high FIT prices before sunrise (with forecast protection to ensure refill)

#### Daytime Optimization
- **Solar Surplus Bypass**: When forecast is excellent (2× battery capacity), bypass normal export restrictions
- **Full Battery PV Export**: Emergency valve prevents solar curtailment when battery at 99%+
- **PV Safeguard**: Protects battery from over-exporting when forecast insufficient

#### Evening/Night Optimization
- **Evening Export Boost**: Aggressive evening export when tomorrow's forecast is strong
- **Standby Holdoff**: Blocks charging when solar forecast can reach target SoC
- **Demand Window Blocking**: Prevents import during configured high-demand periods

### Export Control
- **Three-Tier System**: Low/Medium/High export limits (default 5/10/25kW) based on FIT price
- **Dynamic SoC Scaling**: Export limit scales with battery state of charge
- **Forecast Guards**: Multiple protection layers prevent battery depletion

### Import Control
- **Negative/Free Price Import**: Automatically import when paid or free
- **Cheap Import Top-Up**: Daytime top-up when import price below threshold
- **Grid Capacity Limits**: Respects configurable import/export power caps

---

## Requirements

### Home Assistant Integrations
- **SigEnergy** - Battery/EMS control and sensor data
- **Solcast Solar** - PV forecast data (custom integration)
- **Amber Electric** - Dynamic pricing data (Australia)

### Required Helper Entities

Create these before installing the automation:

```yaml
# input_boolean.sigenergy_automated_export
# Enables/disables automated export control
input_boolean:
  sigenergy_automated_export:
    name: SigEnergy Automated Export
    icon: mdi:transmission-tower-export

# input_number.sigenergy_export_session_start_kwh
# Tracks export session starting point
input_number:
  sigenergy_export_session_start_kwh:
    name: SigEnergy Export Session Start
    min: 0
    max: 10000
    step: 0.1
    unit_of_measurement: kWh
    icon: mdi:counter

# input_number.sigenergy_import_session_start_kwh
# Tracks import session starting point
input_number:
  sigenergy_import_session_start_kwh:
    name: SigEnergy Import Session Start
    min: 0
    max: 10000
    step: 0.1
    unit_of_measurement: kWh
    icon: mdi:counter
```

---

## Installation

### 1. Install Blueprint

**Option A: Via URL (Recommended)**
1. Open Home Assistant
2. Navigate to Settings → Automations & Scenes → Blueprints
3. Click "Import Blueprint"
4. Paste URL: `https://github.com/Belot77/Martys-Automation/blob/main/blueprints/automation/Belot77/sigenergy_optimiser.yaml`
5. Click "Preview" then "Import"

**Option B: Manual Copy**
1. Copy `blueprints/automation/Belot77/sigenergy_optimiser.yaml` to your Home Assistant config
2. Restart Home Assistant or reload automations

### 2. Create Helper Entities
Add the helper entities shown above to your `configuration.yaml` or create via UI (Settings → Devices & Services → Helpers).

### 3. Create Automation from Blueprint
1. Settings → Automations & Scenes → Create Automation
2. Select "SigEnergy Solar Import & Export Control" blueprint
3. Configure all required sensors (see Configuration section)
4. Save automation

---

## Configuration Guide

### Essential Settings

**Control & Notifications**
- **Automated Export Switch**: `input_boolean.sigenergy_automated_export`
- **HA Control Switch**: Entity controlling Home Assistant mode (required)

**SigEnergy Sensors**
- Battery SoC, capacity, power sensors
- Solar production (current and potential)
- Load consumption
- EMS mode status

**Price & Forecast Sensors**
- Amber general/feedin price sensors
- Solcast forecast (today/tomorrow)

**Export/Import Limits**
- Grid capacity caps (e.g., 25kW export, 15kW import)
- Minimum transfer thresholds

### Key Configuration Options

#### Forecast Safety Margins (Stability & Hysteresis section)
- **Forecast Safety Margin - Charging** (default 1.25)
  - Used for: Morning slow charge, morning dump, standby holdoff, daytime top-up
  - Higher consequence if wrong → import expensive grid power
  - Increase if Solcast consistently over-predicts

- **Forecast Safety Margin - Export** (default 1.1)
  - Used for: Export forecast guard, PV safeguard, evening boost
  - Lower consequence if wrong → miss some export earnings
  - Adjust based on local forecast accuracy

#### Export Price Tiers
- **Low Tier Threshold**: FIT price where export starts (e.g., 0.05 $/kWh = 5c)
- **Medium Tier Threshold**: Mid-tier price (e.g., 0.10 $/kWh = 10c)
- **High Tier Threshold**: High-value export (e.g., 0.20 $/kWh = 20c)

#### Export Discharge Limits
- **Low Tier Limit**: Export power at low FIT (default 5kW)
- **Medium Tier Limit**: Export power at medium FIT (default 10kW)
- **High Tier Limit**: Export power at high FIT (default 25kW)

#### SoC Floors & Reserves
- **Export Min SoC**: Absolute export floor (e.g., 20%)
- **Min Export Target SoC**: Target minimum during day (e.g., 40%)
- **Sunrise Safety Factor**: Multiplier for overnight consumption estimate (default 1.0)

#### Import Control
- **Cheap Import Threshold**: Price to enable daytime top-up (e.g., 0.12 $/kWh)
- **Cheap Import Target SoC**: Top-up target (e.g., 80%)

#### Morning Modes
- **Morning Slow Charge**: Enable gentle pre-sunrise charging
  - Rate, target SoC, hours before sunrise
- **Morning Dump**: Enable pre-sunrise export at high FIT
  - Hours before/after sunrise for dump window

#### Evening Export Boost
- **Enable boost** for aggressive evening export when tomorrow looks good
- **Aggressive Floor**: Lower SoC limit during boost (e.g., 35%)

---

## Understanding Status Messages

### Export Messages
- **"Exporting, Low/Med/High tier Xkw @ Yc"** - Normal tier-based export
- **"Exporting, Full battery XkW @ Yc"** - Emergency PV export (battery ≥99%)
- **"Exporting, Solar bypass XkW (YkWh left)"** - Surplus bypass active (excellent forecast)
- **"Export blocked, below X% target"** - Battery below minimum target SoC
- **"Export blocked, saving for tomorrow"** - Forecast protection active
- **"Export blocked, FIT < X"** - Feed-in price below threshold

### Import Messages
- **"Importing, paid price=X"** - Paid to import (negative price)
- **"Importing FREE"** - Free import period
- **"Import blocked, charge holdoff"** - Forecast sufficient, waiting for solar
- **"Import blocked, demand window"** - High-demand period blocking

### Mode Messages
- **"Morning dump"** - Pre-sunrise battery export active
- **"Morning slow charge"** - Gentle pre-sunrise charging
- **"Overnight charge"** - Emergency charging when forecast insufficient

---

## Advanced Features

### Solar Surplus Bypass
When solar forecast is exceptional (≥80kWh for 40kWh battery), bypasses normal export restrictions to allow full PV export even when battery below target.

**Thresholds:**
- START: 2× battery capacity (e.g., 80kWh)
- CONTINUE: 1.25× battery capacity (50kWh) - hysteresis prevents flapping

### Full Battery Emergency Valve
At 99% SoC with PV surplus, allows high export limit regardless of FIT tier to prevent solar curtailment. Uses `solar_potential_kw` (uncurtailed potential) to calculate export limit.

### Forecast-Based Holdoff
Blocks unnecessary grid import when solar forecast can naturally reach target SoC. Uses configurable safety margin (default 1.25×).

---

## Troubleshooting

### No Export Happening
1. Check `input_boolean.sigenergy_automated_export` is ON
2. Verify battery SoC above minimum thresholds
3. Check FIT price above low tier threshold
4. Review status message for reason (forecast guard, SoC floor, etc.)

### Battery Not Charging Overnight
1. Verify forecast insufficient to reach target SoC
2. Check standby holdoff not blocking
3. Review morning slow charge settings if enabled

### Solar Curtailed When Battery Full
1. Check `full_battery_pv_export` is triggering (battery ≥99%)
2. Verify FIT price is positive
3. Check export limit and status message

### Forecast Safety Margins
- If importing too often: Decrease charging margin (e.g., 1.25 → 1.15)
- If missing export opportunities: Decrease export margin (e.g., 1.1 → 1.05)
- If running out of battery: Increase charging margin (e.g., 1.25 → 1.35)

---

## Contributing

Contributions welcome! Please:
1. Test changes thoroughly
2. Document behavior in comments
3. Update TODO.md and CHANGELOG.md
4. Credit Martin Pascoe's original work

---

## License

This automation builds upon Martin Pascoe's original work. Please respect attribution when sharing or modifying.

---

## Support

For issues or questions:
- Review TODO.md for known issues
- Check status messages for operational details
- Examine Home Assistant logs for errors
- GitHub Issues: https://github.com/Belot77/Martys-Automation/issues
