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

The negative thing is that sometimes you need to follow the road 'Automation'-->'Sequence Script'-->'Service Script' to find the action you need.
The positive thing is that when you change a Service Script all Automations and Sequence Scripts are updated at once.
Another reason might be the possiblity to add some different conditions to different actions in the sequence.
When a condition is not TRUE the sequence will stop, but you might want to cancel only one of the actions.

### Example:
### Automation
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

### Sequence Script
- alias: Licht - All - Keuken - Aan
    sequence:
        - service: script.turn_on
          entity_id:
            - script.light_kitchen_milight_on_night
            - script.light_kitchen_milight_tv_on

### Service Script
alias: Licht - Keuken - Milight - Aan - Night
sequence:
  - condition: numeric_state
    entity_id: sensor.keuken_sensor_light_level
    below: !secret lux_kitchen_low
  - service: light.turn_on
    entity_id:
      - light.milight_keuken
    data:
      hs_color: [0,100]
      brightness: 200

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
