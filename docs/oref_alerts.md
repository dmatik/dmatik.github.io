---
layout: default
title: orefAlerts
nav_order: 3
permalink: /docs
---

# orefAlerts
 
Node.js RESTful API to retrieve Israeli [Pikud Ha-Oref](https://www.oref.org.il/) so called "Red Color" alerts. <br/>
The project deployed on Docker Hub as [dmatik/oref-alerts](https://hub.docker.com/r/dmatik/oref-alerts).

## Usage
### Run from hub
#### docker run from hub
```text
docker run -p 49000:3000 --name oref-alerts dmatik/oref-alerts:latest
```

#### docker-compose from hub
```yaml
version: "3.6"
services:
    oref-alerts:
        image: dmatik/oref-alerts:latest
        container_name: oref-alerts
        network_mode: "bridge"
        ports:
          - 49000:3000
        environment:
            TZ: "Asia/Jerusalem"
        restart: unless-stopped
```

### JSON Response Examples
#### Example for /current endpoint
```json
{
    "alert": "true",
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
#### Example for /last_day endpoint
```json
{
    "lastDay": [
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
