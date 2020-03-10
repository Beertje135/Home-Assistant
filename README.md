# Bert's Home-Assistant
## My Home Assistant Config

UNDER CONSTRUCTION

This documents how I created my configuration and why I made some decisions.
Home Assistant is a way for making different Smart Applications talk to each other and this is the main reason I use this great software.

My wife is my best tester and gives me the best feedback.
If she gets annoyed with a light or an automation it means it isn't good enough or finished yet.
She is very patient but not so tech minded so the frontend (LoveLace) is not a primary concern for my setup.
I use Home Assistant for linking different home automation platforms like Apple Homekit and Amazon Alexa.
These will be the frontend if we need one. Everything is automated so the less we need to pull out our phones the better.
And I really believe home automation shouldn't be visible or difficult for the user.

The most important automation is the presence detection in our home.
I based this system on the post of Phil Hawthorne:
https://philhawthorne.com/making-home-assistants-presence-detection-not-so-binary/
Basically what this does is put the home in different modes based on who is home and when.
And these states will influence almost all other automations.
Eg. When I work from home (state = "Thuis (Werk)") only the kitchen (my office :-)) will be heated and radio will play not to loud.

The second most important automation is the light(hue and milight). We barely use any light switches in the house.
Automating lights without removing the manual overwrite is like walking on a tightrope.
What seems to work is to automate all the extra lights (table, floor lights) and leaving the ceiling lights to manual mode.
Eg. the kitchen led light will be automated (and is in 95% of the time enough light) but the ceiling lights are used on special occasions and as a failsafe.

The third and hardest automation is the heating(evohome).
We have a European Honeywell Evohome system in our home with different zones and schedules. Trying to save as much money on heating and not compromising on comfort while still relying on the logics of the Evohome system (pre heat, pre stop, shedule, etc.) was a great challenge.

Further we have radio(alexa), visitors(presence), doorbell(ESPHome), notifications(ios and alexa), tv (harmony and plex), gas and water usage(Smappee), solar panels(SolarEdge), vacuum(Roomba), etc.

## Integrations

## Custom Components

## Configuration Best Practices

### Naming Convention
### Scripts

Everywhere: what_all_action     e.g. light_all_off, music_all_on  
Room:       what_where_action   e.g. light_bathroom_on, music_kitchen_on  
Entity:     entity_action       e.g. arlo_light_on, alexa_bathroom_play_mnm  


## Templates

## Sensors

## Automations and Scripts

The structure of my config is based on a top down system.
My intention was to minimize the files that needed changes when one or more variables change.
Automations will point to scripts where possible.
I have two types of scripts, the Sequence scripts and the Service scripts. These are both regular scripts but I call them this way to illustrate the way I use them.

Automations --> Action points to a script where possible
Sequence Script --> Points to one or more Service scripts
Service Scripts --> Contain the actual action

The positive thing is that when you change a Service Script all Automations and Sequence Scripts are updated at once.
Another reason is the possibility to add different conditions to different actions in the sequence.
When a condition is not TRUE the sequence will stop, but you might want to cancel only one of the actions.
The downside is that sometimes you need to follow the road 'Automation'-->'Sequence Script'-->'Service Script' to find the action you need to change.

I also depend heavily on conditions. Some conditions are repeated in the automation and all scripts.
This is done so the automation isn't constantly fired and so we can use the script for multiple purposes.
Eg. movement will only fire the automation when it is dark and script will only fire if light is off.
Eg. the sequence script will turn on service script light 1 and service script light 2 but only if the light(s) are off. If we put these in one sequence and light 1 is already on the second light will not get turned on because the whole sequence is stopped.

### Example:
### Automation
```
- alias: Licht - Keuken - Aan
  trigger:
  - entity_id: binary_sensor.hue_sensor_keuken_motion
    platform: state
  condition:
  - condition: numeric_state
    entity_id: sensor.hue_sensor_keuken_light_level
    below: !secret lux_kitchen_low
  action:
  - service: script.turn_on
    entity_id: script.light_keuken_on
```
### Sequence Script
```
alias: Licht - Keuken - All - Aan
sequence:
  - service: script.turn_on
    entity_id:
      - script.milight_one_on
      - script.milight_bridge_one_on
```
```
alias: Licht - Keuken - Milight TV - Aan
sequence:
  - condition: numeric_state
    entity_id: sensor.hue_sensor_keuken_light_level
    below: !secret lux_kitchen_very_low
  - service: script.turn_on
    data_template:
      entity_id: >
        {%- if states('input_select.woning_status') in ['Slapen'] -%}
          script.milight_bridge_one_on_night
        {%- elif states('input_boolean.light_downstairs') in ['off'] -%}
          script.milight_bridge_one_on_standby      
        {%- elif states('group.bezoekers') in ['not_home'] and (states('sensor.time') <= '06:00' or states('sensor.time') >= '23:00') -%}
          script.milight_bridge_one_on_dimmed
        {%- elif states('input_boolean.media_kitchen') in ['on'] -%}
          script.milight_bridge_one_on_standby
        {%- else -%}
          script.milight_bridge_one_on_normal
        {%- endif %}
```
### Service Script
```
alias: Licht - Keuken - Milight TV - Aan - Night
sequence:
  - condition: numeric_state
    entity_id: sensor.hue_sensor_keuken_light_level
    below: !secret lux_kitchen_very_low
  - service: light.turn_on
    entity_id:
      - light.milight_keuken_bridge
    data:
      hs_color: [0,100]
      brightness: 200
```

## Homekit and Alexa
