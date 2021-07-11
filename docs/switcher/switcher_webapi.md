---
layout: default
title: WebAPI HA integration
nav_order: 3
has_children: false
---

# WebAPI HA integration

## What
This is alternative option for integration of Switcher devices in Home Assistant.  
There are two existing components, both developed by [TomerFi](https://github.com/TomerFi) for that: 
* "switcher_aio" - Very old custom component and has issues with newer versions of HA.
* "switcher_kis" - Newer component, already part of official HA releases. However it has very limited functionality. Also some people report they are getting timeouts while trying to load it during HA startup.  

In this solution we will be using [**Switcher WebAPI**](https://github.com/TomerFi/switcher_webapi), running in docker, developed also by [TomerFi](https://github.com/TomerFi).

## Prerequisites
* Install and configure your Switcher device.
* Collect the following information from all the Switcher device’s:
  * IP addresses
  * Device IDs (Use [this](switcher_discovery.md) to retrieve)
* Install docker (preferably with docker-compose)

## How

### WebAPI container
Setup Switcher WebAPI container in docker using its the [documentation](https://switcher-webapi.tomfi.info).  
This is example of working Docker Compose (change to your values):

```yaml
switcher_webapi:
  image: tomerfi/switcher_webapi:latest
  container_name: switcher_webapi
  hostname: switcher_webapi
  restart: always
  network_mode: bridge
  ports:
    - 8000:8000
  environment:
    TZ: "Asia/Jerusalem" 
```

> If you are using ARM system, like RPI, please use the below image.    
> [https://hub.docker.com/r/avishayil/switcher_webapi](https://hub.docker.com/r/avishayil/switcher_webapi)

### RESTful commands
Define RESTful commands in HA, to be used in scripts.  

{% raw %}
```yaml
rest_command:

  switcher_turn_on:
    url: http://{{switcher_web_api_ip}}:{{switcher_web_api_port}}/switcher/turn_on?id={{switcher_device_id}}&ip={{switcher_ip}}
    method: "POST"

  switcher_turn_on_timer:
    url: http://{{switcher_web_api_ip}}:{{switcher_web_api_port}}/switcher/turn_on?id={{switcher_device_id}}&ip={{switcher_ip}}
    method: "POST"
    payload: '{"minutes": "{{minutes}}"}'

  switcher_turn_off:
    url: http://{{switcher_web_api_ip}}:{{switcher_web_api_port}}/switcher/turn_off?id={{switcher_device_id}}&ip={{switcher_ip}}
    method: "POST"
```
{% endraw %}

### Sensors
Define RESTful Sensor and other Template sensors depending on it in HA.  
> Change to your WebAPI IP, Device ID, Switcher IP and port below.

{% raw %}
```yaml
sensor:
      - platform: rest
        resource: http://<<WEB_API_IP>>:8000/switcher/get_state?id=<<DEVICE_ID>>&ip=<<SWITCHER_IP>>
        name: Switcher WebAPI
        json_attributes:
          - state
          - time_left
          - auto_off
          - power_consumption
          - electric_current
        value_template: 'OK'

      - platform: template
        sensors:
          switcher_webapi_time_left:
            friendly_name: "Time Left"
            icon_template: mdi:timelapse
            value_template: >-
              {% if is_state("sensor.switcher_webapi_state", "off") %}
                  off
              {% else %}
                  {% set hour = states.sensor.switcher_webapi.attributes.time_left.split(':')[0] %}
                  {% set min = states.sensor.switcher_webapi.attributes.time_left.split(':')[1] %}
                  {% set sec = states.sensor.switcher_webapi.attributes.time_left.split(':')[2] %}
                  {{ hour | int }}h {{ min | int }}m
              {% endif %}
    
          switcher_webapi_auto_off_time:
            friendly_name: "Auto Off"
            icon_template: mdi:av-timer
            value_template: >-
              {% if is_state("sensor.switcher_webapi_state", "off") %}
                  off
              {% else %}
                  {% set hour = states.sensor.switcher_webapi.attributes.time_left.split(':')[0] %}
                  {% set min = states.sensor.switcher_webapi.attributes.time_left.split(':')[1] %}
                  {% set sec = states.sensor.switcher_webapi.attributes.time_left.split(':')[2] %}
                  {% set seconds = hour | int *3600 + min | int * 60 + sec | int * 1  %}
                  {{ ( now().timestamp() + seconds ) | timestamp_custom("%H:%M") }}
              {% endif %}

          switcher_webapi_electric_current:
            friendly_name: "Electric Current"
            icon_template: mdi:flash
            unit_of_measurement: A
            value_template: >-
              {{ state_attr('sensor.switcher_webapi', 'electric_current') }}

          switcher_webapi_power_consumption:
            friendly_name: "Power Consumption"
            icon_template: mdi:flash
            unit_of_measurement: kW
            value_template: >-
              {{ state_attr('sensor.switcher_webapi', 'power_consumption') }}

          switcher_webapi_state:
            friendly_name: "State"
            icon_template: mdi:shower
            value_template: >-
               {% if state_attr('sensor.switcher_webapi', 'state') == 'ON' %}
                  on
               {% else %}
                  off
               {% endif %}
```
{% endraw %}

### Input Select
Define Input Select in HA, to select the timings for the Turn On with timer script.

```yaml
input_select:
      switcher_timer_minutes_input_select:
          name: Timer minutes
          options:
              - 15
              - 30
              - 45
              - 60
```

### Scripts
Define scripts in HA for turning on the Switcher, with and without timers, and turning it off.

{% raw %}
```yaml
script:
      switcher_turn_on_timer_script:
          alias: On with Timer
          sequence:
              - service: rest_command.switcher_turn_on_timer
                data_template:
                  switcher_web_api_ip: <<WEB_API_IP>>
                  switcher_web_api_port: 8000
                  switcher_device_id: <<DEVICE_ID>>
                  switcher_ip: <<SWITCHER_IP>>
                  minutes: '{{ states("input_select.switcher_timer_minutes_input_select") }}'
              - service: homeassistant.update_entity
                entity_id: sensor.switcher_webapi

      switcher_turn_on:
          alias: Turn On
          sequence:
              - service: rest_command.switcher_turn_on
                data_template:
                  switcher_web_api_ip: <<WEB_API_IP>>
                  switcher_web_api_port: 8000
                  switcher_device_id: <<DEVICE_ID>>
                  switcher_ip: <<SWITCHER_IP>>
              - service: homeassistant.update_entity
                entity_id: sensor.switcher_webapi

      switcher_turn_off:
          alias: Turn Off
          sequence:
              - service: rest_command.switcher_turn_off
                data_template:
                  switcher_web_api_ip: <<WEB_API_IP>>
                  switcher_web_api_port: 8000
                  switcher_device_id: <<DEVICE_ID>>
                  switcher_ip: <<SWITCHER_IP>>
              - service: homeassistant.update_entity
                entity_id: sensor.switcher_webapi
```
{% endraw %}

### Switch
Define Switch in HA, which uses the sensor and scripts we defined before.

{% raw %}
```yaml
switch:
      - platform: template
        switches:

          switcher_webapi:
              friendly_name: Boiler
              icon_template: mdi:shower
              value_template: "{{ is_state('sensor.switcher_webapi_state', 'on') }}"
              turn_on:
                  service: script.turn_on
                  data:
                    entity_id: script.switcher_turn_on
              turn_off:
                  service: script.turn_on
                  data:
                    entity_id: script.switcher_turn_off
```
{% endraw %}

## UI
This is it. All that is left to define a nice UI and use all the above entities.
But this is for another guide.
