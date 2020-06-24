---
title: Plugwise
description: Plugwise Smile platform integration.
ha_category:
  - Climate
  - Sensor
  - Switch
  - Water Heater
ha_iot_class: Local Polling
ha_release: 0.98
ha_codeowners:
  - '@CoMPaTech'
  - '@bouwew'
ha_config_flow: true
ha_domain: plugwise
---

This enables [Plugwise](https://www.plugwise.com) components with a central Smile gateway to be integrated. This integration talks locally to your **Smile** interface, and you will need its password and IP address.
The platform supports [Anna](https://www.plugwise.com/en_US/products/anna), [Adam (zonecontrol)](https://www.plugwise.com/en_US/zonecontrol) and [P1](https://www.plugwise.com/en_US/products/smile-p1) Smile products. See below list for more details.

Platforms available - depending on your Smile and setup are include:

 - `climate` (for the Anna and Lisa products, or a single Tom)
 - `sensor` (for all relevant products including the Smile P1)
 - `binary_sensor` (for domestic hot water and secondary heater)
 - `switch` (for Plugs)

The password can be found on the bottom of your Smile, it should consist of 6 characters. To find your IP address use the Plugwise App: 

 - Open the Plugwise App and choose the 'Settings'-icon (&#9776;) and choose 'HTML-interface'. 
 - Go to the (lower) 'Settings'-icon (&#9776;) and choose 'Preferences'. 
 - Choose 'System' then 'Networking' and your IP address will be shown.

## Entities

This integration will show all Plugwise devices present in your Plugwise configuration. In addition you will see a 'Smile' device representing your central Plugwise gateway (i.e. the Smile, Smile P1 or Adam).

For example, if you have an Adam setup with a Lisa named 'Living' and a Tom named 'Bathroom' these will show up as individual devices. For setups where an auxiliary heater (or heater/cooler) is present there will be an additional device representing it called 'Auxiliary'.

Centralized measurements such as 'power' for a P1, 'outdoor_temperature' on Anna or Smile will be assigned to your gateway device. Auxiliary Heater(/Cooler) measurements such as 'boiler_temperature' will be assigned to the Auxiliary entity.

## Configuration

To set up this integration, click Configuration in the sidebar and then click Integrations. Add a new integration using the "+" button in the lower right corner and look for 'Plugwise'. Click configure and you will be presented with a dialog requesting the Smile ID or password of your Smile and it's IP address. After you click submit, you will have the opportunity to select the area(s) where individual Smile appliances are located.

Depending on your `climate` setup an auxiliary entitiy will be added when there is information available about such Plugwise devices. If you have "plug"s (as in, pluggable switches that come with an Adam) those will be discovered as `switch`es. Various other measures of your setup will be available as `sensor`s or `binary_sensor`s.

Repeat the above procedure for each Smile gateway (i.e. if you have an Adam setup and a P1 DSMR you'll have to add two integrations).

### Services

#### Update Smile data

Forced update of data from your Smile can be triggered by calling the generic `homeassistant.update_entity` service with your Smile entity as the target.

```yaml
# Example script change the temperature
script:
  force_adam_update:
    sequence:
      - service: homeassistant.update_entity
        data:
          entity_id: climate.anna
```

#### Set HVAC mode (schedule)

Service: `climate.set_hvac_mode`

Available options include `auto` or `off`. The meaning of `auto` is that a schedule is active and the thermostat will change presets accordingly. The meaning of `off` is that there is no schedule active, i.e., the active preset or manually set temperature is to be used to control the climate of your house or rooms.
The last schedule that was active is determined the same way long-tapping the top of Anna works.

Example:

```yaml
# Example script change the temperature
script:
  lisa_reactive_last_schedule:
    sequence:
      - service: climate.set_hvac_mode
        data:
          entity_id: climate.lisa_bios
          hvac_mode: auto
```

Changing the active schedule can be done, though not easily:

- Deactivate the schedule (press the power button on the UI-card).
- Look up the available schemas in the `states` view (attribute `available_schemas`).
- Change the attribute `active_schema` to the actual name of your schema as available from the above attribute or your Plugwise App.
- Activate the schedule (press the calendar button on the UI-card).

#### Set temperature

Service: `climate.set_temperature`

Example:

```yaml
# Example script change the temperature
script:
  anna_set_predefined_temperature:
    sequence:
      - service: climate.set_temperature
        data:
          entity_id: climate.anna
          temperature: 19.5
```

#### Set preset mode

Service: `climate.set_preset_mode`

Available options include: `home`, `vacation`, `no_frost`, `asleep` & `away`.

Example:

```yaml
# Example script changing the active (or currently set by schedule) preset
script:
  anna_activate_preset_asleep:
    sequence:
      - service: climate.set_preset_mode
        data:
          preset_mode: asleep
```

### Supported devices

The current implementation of the Python module (Plugwise-Smile) includes:

Adam (zone_control):

 - v3.0
 - v2.3

 - Devices supported are Floor, Lisa, Tom, Koen and Plug - note a Koen always comes with a Plug (the active part) 

Anna (thermostat):

 - v4.0
 - v3.1
 - v1.8

Smile P1 (DSMR):

 - v4.0
 - v3.3
 - v2.5
