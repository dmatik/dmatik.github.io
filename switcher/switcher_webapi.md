# Switcher HA integration using WebAPI

## What
This is alternative option for integration of Switcher devices in Home Assistant.  
There are two existing components, both developed by [TomerFi](https://github.com/TomerFi) for that: 
* "switcher_aio" - Very old custom component and has issues with newer versions of HA.
* "switcher_kis" - Newer component, already part of official HA releases. However it has very limited functionality. Also some people report they are getting timeouts while trying to load it during HA startup.  

In this solution we will be using [**Switcher WebAPI**](https://github.com/TomerFi/switcher_webapi), running in docker, developed also by [TomerFi](https://github.com/TomerFi).

## Prerequisites
* Install and configure your Switcher device.
* Collect the following information from the device’s (Use [this](switcher_discovery.md) to retrieve the device_id):
  * ip_address
  * device_id
* Install docker (preferably with docker-compose)

## How

### WebAPI container
Setup Switcher WebAPI container in docker using its the [documentation](https://switcher-webapi.tomfi.info).  
This is example of working Docker Compose (change to your values):

```YAML

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

> If you are using ARM system, like PRI, please use the below image.    
> [https://hub.docker.com/r/avishayil/switcher_webapi](https://hub.docker.com/r/avishayil/switcher_webapi)

### RESTful commands
Define RESTful commands in HA, to be used in scripts.  
> Change to your IP and port below.

```YAML
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
