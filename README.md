# Foxess Home Assistant Automation with Amber Electric Pricing

A simple, effective Home Assistant automation for Foxess 15kW inverters optimized for Amber Electric dynamic pricing. This automation maximizes revenue from battery discharge while protecting battery health and respecting critical price events.

## Table of Contents
1. [Overview](#overview)
2. [Control Strategy](#control-strategy)
3. [Installation](#installation)
4. [Entity Requirements](#entity-requirements)
5. [Automation Rules](#automation-rules)
6. [Troubleshooting](#troubleshooting)

---

## Overview

This automation implements a **master control strategy** that prioritizes:
- **Revenue maximization** through intelligent battery discharge during high feed-in prices
- **Battery health** with critical SOC protection thresholds
- **Grid backup** through scheduled charging during low-price periods
- **Solar priority** ensuring solar energy is used before battery discharge

---

## Control Strategy

### **RULE 1: Primary (Highest Priority)**
**Hold battery when spike is forecasted**
- **Trigger:** Feed-in price > $1.00 AND forecast shows spike coming
- **Action:** HOLD battery (discharge = 0)
- **Exception:** If current price > $0.40 (immediate spike) → DISCHARGE at 15kW immediately

### **RULE 2: Secondary (Normal Operation)**
**Discharge battery when solar is low and prices are good**
- **Trigger:** Feed-in price > $0.15 AND solar generating < 2kW
- **Action:** DISCHARGE at 15kW (sell to grid)
- **Limit:** Stop discharge when SOC reaches 40% (safety minimum)

### **RULE 3: Grid Backup (1 PM - 2 PM Window)**
**Charge from grid during guaranteed low-price window**
- **At 1 PM:** If SOC < 40% AND general price < $0.06 → CHARGE from grid to 50% SOC
- **At 2 PM:** STOP grid charging, return to normal operation

### **RULE 4: Negative Pricing (≤ -$0.01)**
**Maximize charging when you're paid to consume**
- **Trigger:** Feed-in price ≤ -$0.01
- **Action:** CURTAIL solar (force_charge_power = 0)
- **Action:** CHARGE battery from grid at 15kW

### **RULE 5: Critical Protection**
**Prevent over-discharge and battery damage**
- **At SOC ≤ 10%:** Stop ALL discharge immediately
- **Resume discharge** only when SOC reaches 15%
- **At SOC ≤ 40%:** Stop discharge, revert to self-consumption only

### **RULE 6: Solar Priority**
**Always use solar first before battery**
- If solar generation > 2kW: Disable battery discharge
- Solar powers house load first, charges battery second, feeds excess to grid third

---

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/morgs1987/foxess-ha-amber-automation-morgs1987.git
   cd foxess-ha-amber-automation-morgs1987
   ```

2. **Copy files to Home Assistant:**
   ```bash
   cp automations.yaml ~/.homeassistant/automations.yaml
   ```

3. **Restart Home Assistant** or reload automations:
   - Settings → Developer Tools → YAML → Automations

4. **Verify entities exist** (see Entity Requirements below)

---

## Entity Requirements

Your Home Assistant setup must have these entities available:

### **Sensor Entities (Read-Only)**
| Entity | Purpose | Example Source |
|--------|---------|-----------------|
| sensor.home_assistant_general_price | Grid import price ($/kWh) | Amber Electric API |
| sensor.home_assistant_feed_in_price | Feed-in/export price ($/kWh) | Amber Electric API |
| sensor.home_assistant_feed_in_forecast | Next 30 min forecast ($/kWh) | Amber Electric API |
| sensor.home_automation_battery_soc_1 | Battery State of Charge (%) | Foxess Inverter |
| sensor.home_automation_solar_power | Current solar generation (W) | Foxess Inverter |

### **Binary Sensor Entities (Read-Only)**
| Entity | Purpose |
|--------|---------|
| binary_sensor.home_assistant_price_spike | True when price > $1.00 |

### **Control Entities (Write)**
| Entity | Purpose | Range |
|--------|---------|-------|
| number.home_automation_force_charge_power | Force battery charge power (W) | 0 - 15000 |
| number.home_automation_force_discharge_power | Force battery discharge power (W) | 0 - 15000 |
| number.home_automation_max_charge_current | Max charge current (A) | 0 - 100 |
| number.home_automation_max_discharge_current | Max discharge current (A) | 0 - 100 |
| number.home_automation_max_soc | Maximum battery SOC (%) | 0 - 100 |
| number.home_automation_min_soc | Minimum battery SOC (%) | 0 - 100 |
| number.home_automation_min_soc_on_grid | Min SOC when on grid (%) | 0 - 100 |
| select.home_automation_work_mode | Inverter mode | self-consumption / feed-in / backup |

---

## Automation Rules

### **Automation 1: Master Price Spike Strategy**
- Watches feed-in price for spikes > $1.00
- Holds battery (discharge = 0) when spike is forecasted
- Discharges immediately if current price > $0.40

### **Automation 2: Secondary Discharge Rule**
- Monitors solar power and feed-in price
- Discharges when: price > $0.15 AND solar < 2kW
- Stops at 40% SOC minimum

### **Automation 3: Grid Backup Charging**
- 1 PM: Start charging if SOC < 40% and price < $0.06
- 2 PM: Stop charging (back to normal mode)

### **Automation 4: Negative Price Response**
- Curtails solar when price ≤ -$0.01
- Charges battery from grid at 15kW

### **Automation 5: Critical Protection**
- Stops discharge at SOC ≤ 10%
- Resumes at SOC ≥ 15%

### **Automation 6: Solar Priority**
- Disables discharge when solar > 2kW
- Ensures solar is used first

---

## Troubleshooting

### **Issue: Automation not triggering**
- **Check:** Entity IDs match exactly (case-sensitive)
- **Check:** All sensors are reporting values (not unavailable)
- **Check:** Reload automations after any YAML edits

### **Issue: Battery discharges too much**
- **Check:** SOC protection rules are active (SOC ≤ 40%)
- **Check:** sensor.home_automation_battery_soc_1 is reporting correctly

### **Issue: Solar not prioritized**
- **Check:** sensor.home_automation_solar_power is reporting > 0 when generating
- **Check:** Solar priority automation runs after discharge automation

### **Issue: Grid charging not working at 1 PM**
- **Check:** Time-based trigger is set to 13:00 (1 PM in 24hr format)
- **Check:** sensor.home_assistant_general_price is below 0.06
- **Check:** SOC is below 40%

### **Enable Debug Logging**
Add to configuration.yaml:
```yaml
logger:
  logs:
    homeassistant.components.automation: debug
```

---

## Revenue Optimization Tips

1. **Monitor your feed-in price trends** - Adjust Rule 2 discharge threshold ($0.15) based on typical prices
2. **Check price forecasts daily** - Rule 1 protects battery during predicted spikes
3. **Set realistic SOC limits** - 40% minimum discharge prevents over-cycling
4. **Time your grid backup** - 1 PM charging window typically has lowest prices

---

## Support & Issues

If you encounter issues:
1. Check the Home Assistant Automation Docs
2. Review Amber Electric Integration
3. Check Foxess Integration
4. Create an issue on this repository

---

**Last Updated:** 2026-02-10 06:43:03 UTC
**Status:** Production Ready