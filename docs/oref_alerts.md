---
layout: default
title: orefAlerts
nav_order: 3
permalink: /docs/oref_alerts
---

![pikud_haoref](images/pikud_haoref.png)

# oref-alerts-proxy-ms
 
Java Spring Boot MS to retrieve Israeli [Pikud Ha-Oref](https://www.oref.org.il/) so called "Red Color" alerts. <br/>
The project deployed on Docker Hub as [dmatik/oref-alerts](https://hub.docker.com/r/dmatik/oref-alerts).   
The source code is on [Github](https://github.com/dmatik/oref-alerts-proxy-ms).
<br><br>
Latest stable Node.js release: dmatik/oref-alerts:v0.0.2

## Usage
### Run from hub
#### docker run from hub
```text
docker run -d -p 49000:9001 --name oref-alerts dmatik/oref-alerts:latest
```

#### docker-compose from hub
```yaml
version: "3.6"
services:
    oref-alerts:
        image: dmatik/oref-alerts:latest
        container_name: oref-alerts
        hostname: oref-alerts
        restart: unless-stopped
        network_mode: "bridge"
        ports:
          - 49000:9001
        environment:
            TZ: "Asia/Jerusalem"
            LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB: "INFO"
```

### JSON Response Examples
#### Example for /current endpoint
```json
{
    "alert": true,
    "current": {
        "data": [
            "סעד",
            "אשדוד - יא,יב,טו,יז,מרינה"
        ],
        "id": 1621242007417,
        "title": "התרעות פיקוד העורף"
    }
}
```
#### Example for /history endpoint
```json
{
    "history": [
        {
            "data": "בטחה",
            "date": "17.05.2021",
            "time": "13:31",
            "datetime": "2021-05-17T13:32:00"
        },
        {
            "data": "גילת",
            "date": "17.05.2021",
            "time": "13:31",
            "datetime": "2021-05-17T13:32:00"
        }
    ]
}
```

### Home-Assistant

#### Sensors
##### Fetch the current alert
```yaml
sensor:
  - platform: rest
    resource: http://[YOUR_IP]:49000/current
    name: redalert
    value_template: 'OK'
    json_attributes:
      - alert
      - current
    scan_interval: 5
    timeout: 30
```

##### Fetch the last day history alerts
> **_NOTE:_** This responce is very long, while there is 255 characters limit in HA sensors. <br/>
> Hence adding it to the attribute, which does not have such limit.   

```yaml
sensor:
  - platform: rest
    resource: http://[YOUR_IP]:49000/history
    name: redalert_history
    value_template: 'OK'
    json_attributes:
      - "history"
    scan_interval: 120
    timeout: 30
```

#### Binary Sensors
##### Indicator for all alerts

{% raw %}
```yaml
binary_sensor:
  - platform: template
    sensors:
      redalert_all:
        friendly_name: "Redalert All"
        value_template: >-
          {{ state_attr('sensor.redalert', 'alert') == true }}
```
{% endraw %}

##### Indicator for specific alert

{% raw %}
```yaml
binary_sensor:
  - platform: template
    sensors:
      redalert_ashdod:
        friendly_name: "Redalert Ashdod"
        value_template: >-
          {{ state_attr('sensor.redalert', 'alert') == true and 
                    'אשדוד - יא,יב,טו,יז,מרינה' in state_attr('sensor.redalert', 'current')['data'] }}
```
{% endraw %}
