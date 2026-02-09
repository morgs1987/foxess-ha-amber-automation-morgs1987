# Foxess Home Assistant Automation with Amber Electric Pricing Integration

## Overview
This document provides comprehensive instructions on how to set up and use the Foxess Home Assistant automation with Amber Electric pricing integration. It covers installation, configuration, automation rules, and troubleshooting.

## Table of Contents
1. [Installation](#installation)
2. [Setup Instructions](#setup-instructions)
3. [Entity Configuration](#entity-configuration)
4. [Automation Rules](#automation-rules)
5. [Troubleshooting Guide](#troubleshooting-guide)

## Installation
To install this automation, follow these steps:
1. Clone the repository:
   ```
   git clone https://github.com/morgs1987/foxess-ha-amber-automation-morgs1987.git
   ```
2. Navigate to the directory:
   ```
   cd foxess-ha-amber-automation-morgs1987
   ```
3. Install the necessary dependencies in your Home Assistant environment.

## Setup Instructions
1. **Amber Electric Integration**:  
   - Go to your Home Assistant configuration and set up the Amber Electric integration using the provided API keys from your Amber Electric account.

2. **Foxess Integration**:  
   - Set up the Foxess integration with your solar inverter by adding the required configuration in your `configuration.yaml` file.

3. **Load the Automation**:  
   - Add the automation found in this repository to your Home Assistant automation configuration.

4. **Restart Home Assistant** to apply the changes.

## Entity Configuration
1. Ensure that you have the following entities available in your Home Assistant:
   - `sensor.amber_electric_price` - Fetches current Amber Electric pricing.
   - `sensor.foxess_energy_status` - Monitors energy generation from your Foxess solar inverter.

2. Configure the entities in your `configuration.yaml` file as needed to customize your setup.

## Automation Rules
The following automation rules are configured:
1. **Adjust Energy Usage**: Automatically adjust appliances based on the current Amber Electric price. 

   ```yaml
   alias: Adjust Appliances Based on Amber Pricing
   triggers:
     - platform: state
       entity_id: sensor.amber_electric_price
   actions:
     - service: home_assistant.turn_on
       entity_id: switch.appliance_x
   ```

2. **Send Notifications**: Notify the user when prices drop below a certain threshold.
   ```yaml
   alias: Notify When Amber Price Drops
   triggers:
     - platform: numeric_state
       entity_id: sensor.amber_electric_price
       below: 0.20 # Example threshold
   actions:
     - service: notify.notify
       data:
         message: 'Amber pricing is low, consider using appliances now!'
   ```

## Troubleshooting Guide
- **Issue**: Automation not triggering.
  - **Solution**: Verify the entity states and ensure that the Amber Electric and Foxess integrations are correctly set up.
- **Issue**: API keys not working.
  - **Solution**: Double-check the API keys in your configuration and ensure they are valid.
- **Issue**: Home Assistant not reflecting the latest prices.
  - **Solution**: Review your setup and ensure that the polling interval is set appropriately in the Amber Electric integration settings.

## Conclusion
This README provides a detailed guide for integrating Foxess Home Assistant automation with Amber Electric pricing. For any further issues, check the Home Assistant community forums or the GitHub issues page of this repository.