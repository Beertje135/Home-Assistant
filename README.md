# Home-Assistant
## My Home Assistant Config

UNDER CONSTRUCTION

This document describes how I created my configuration

## Structure

The structure of my config is based on a top down system.
My intention was to minimize the files that needed changes when one or more variables change.
Automations will point to scripts where possible.
I have two types of scripts, the sequence scripts and the Service scripts.

Automations --> Action points to a script where possible
Sequence Script --> Points to one or more Service scripts
Service Scripts --> Contain the actual action

The positive thing is that when you change a Service Script all Automations and Sequence Scripts are updated at once.
Another reason might be the possiblity to add some different conditions to different actions in the sequence.
When a condition is not TRUE the sequence will stop, but you might want to cancel only one of the actions.
The downside is that sometimes you need to follow the road 'Automation'-->'Sequence Script'-->'Service Script' to find the action you need to change.

### Example:
### Automation
```
- alias: Licht - Keuken - Aan
  trigger:
  - entity_id: binary_sensor.keuken_sensor_motion
    platform: state
  condition:
  - condition: numeric_state
    entity_id: sensor.keuken_sensor_light_level
    below: !secret lux_kitchen_low
  action:
  - service: script.turn_on
    entity_id: script.light_kitchen_all_on
```
### Sequence Script
```
alias: Licht - Keuken - All - Aan
sequence:
  - service: script.turn_on
    entity_id:
      - script.light_kitchen_milight_on
      - script.light_kitchen_milight_tv_on
```
```
alias: Licht - Keuken - Milight TV - Aan
sequence:
  - condition: numeric_state
    entity_id: sensor.keuken_sensor_light_level
    below: !secret lux_kitchen_very_low
  - service: script.turn_on
    data_template:
      entity_id: >
        {%- if states('input_select.woning_status') in ['Slapen'] -%}
          script.light_kitchen_milight_tv_on_night
        {%- elif states('input_boolean.light_downstairs') in ['off'] -%}
          script.light_kitchen_milight_tv_on_standby      
        {%- elif states('group.bezoekers') in ['not_home'] and (states('sensor.time') <= '06:00' or states('sensor.time') >= '23:00') -%}
          script.light_kitchen_milight_tv_on_dimmed
        {%- elif states('input_boolean.plex_kitchen') in ['on'] -%}
          script.light_kitchen_milight_tv_on_standby
        {%- else -%}
          script.light_kitchen_milight_tv_on_normal
        {%- endif %}
```
### Service Script
```
alias: Licht - Keuken - Milight TV - Aan - Night
sequence:
  - condition: numeric_state
    entity_id: sensor.keuken_sensor_light_level
    below: !secret lux_kitchen_very_low
  - service: light.turn_on
    entity_id:
      - light.milight_keuken_bridge
    data:
      hs_color: [0,100]
      brightness: 200
```