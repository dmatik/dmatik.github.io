#Switcher integration using WebAPI
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
### Discover Devices Info
1. Install latest Python (Currently 3.9).
2. Install "aioswitcher" module with the below command.
```
pip install aioswitcher
```
3. Download the [script](https://github.com/TomerFi/aioswitcher/blob/dev/scripts/discover_devices.py).
4. Run the following command from the folder you put the script, this will run the discovery for 30 seconds.
```
python discover_devices.py 30
```

Should look like this:
```
C:\Downloads\HA\aioswitcher-dev\scripts>python discover_devices.py 5
{   'auto_shutdown': '03:00:00',
    'device_id': 'XXXXXX',
    'device_state': <DeviceState.OFF: ('0000', 'off')>,
    'device_type': <DeviceType.MINI: ('Switcher Mini', '0f', <DeviceCategory.WATER_HEATER: 1>)>,
    'electric_current': 0.0,
    'ip_address': 'X.X.X.X',
    'last_data_update': datetime.datetime(2021, 7, 10, 19, 59, 22, 291648),
    'mac_address': 'XX:XX:XX:XX:XX:XX',
    'name': 'Switcher Mini 3472',
    'power_consumption': 0,
    'remaining_time': '00:00:00'}
```
