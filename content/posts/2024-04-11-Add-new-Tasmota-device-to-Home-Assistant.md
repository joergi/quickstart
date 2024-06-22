---
title: Add new Tasmota device to Home Assistant with MQTT"
description: "Adding a new Tasmota device to Home Assistant via MQTT and the console of the device"
date: 2024-04-11
---

## Pre-Requirements
To add a new Tasmota device to the Home Assistant (HA), I assume that:
- HA is installed
- An MQTT Broker is running and an user in HA is already registered/in use
- Already one other Tasmota device is connected to the HA (so you know it's working)
- The new Tasmota device is already connected to the local HA WiFi
- Already has upgraded to the newest firmware

## Todos:
### Give device a better name
Click `Configuration` -> `Configure Other` -> and enter at `Device Name` a name which makes sense
### Add device to MQTT
Click `Configuration` -> `Configure MQTT` -> and enter:  
- `Host` of the MQTT-Broker
- `Port` 
- `User` (DVES_USER) (this is the mqtt user you have in your HA)
- `Password` (this is the mqtt user you have in your HA)   

-> Click `Save`

### Make the device auto-discoverable 
Click `Console`  
Type: `Setoption19 0` and press Enter

### Set updating of the device to 20 instead of 2 min.
Click `Console`  
Type: `TelePeriod 20` and press Enter

I learned all this from [this YouTube Video](https://www.youtube.com/watch?v=-MovlMV4Ts8). so all credits go there!