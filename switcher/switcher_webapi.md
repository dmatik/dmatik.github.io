# Switcher integration using WebAPI

## What
This is alternative option for integration of Switcher devices in Home Assistant.  
There are two existing components, both developed by [TomerFi](https://github.com/TomerFi) for that: 
* "switcher_aio" - Very old custom component and has issues with newer versions of HA.
* "switcher_kis" - Newer component, already part of official HA releases. However it has very limited functionality. Also some people report they are getting timeouts while trying to load it during HA startup.  

In this solution we will be using [**Switcher WebAPI**](https://github.com/TomerFi/switcher_webapi), running in docker, developed also by [TomerFi](https://github.com/TomerFi).

## Prerequisites
* Install and configure your Switcher device.
* Collect the following information from the device’s (Use this [script](https://github.com/TomerFi/aioswitcher/blob/dev/scripts/discover_devices.py) to retrieve the device_id, more on this below):
  * ip_address
  * device_id
* Install docker (preferably with docker-compose)

## How

