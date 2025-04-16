---
title: FPV drones
# layout: posts
# author_profile: true
# title: "Vi Everywhere"
# permalink: /life/pages/
excerpt: "notes on FPV drones"
# last_modified_at: 2016-11-03T11:13:12-04:00
collection: learning
categories:
  - learning
tags:
  - fpv
  - drones
  - hardware
  - radio

---
# TODO / MEMOs

- [ ] ExpressLRS presets might be required after flashing new firmware! 50Hz longrange/Cinematic
- [ ] BetaFlight ports -> might be required to be re-anabled (check below)
- [ ] BetaFlight PID -> throttle and motor setting -> reduce from 100% to 60-70%. don't go below 50%
# Batteries

## Li-Ion

- voltage, try not to go below 3.0V
- dead limit 2.5V 
- ok'ish limit 2.7V, as it is recovers back to 3V ish
- charge current - lower the better. 0.3-0.6A, depending on battery
# RX/TX

ELRS - do it via wifi: power off TX, power on RX, wait for wifihotspot -> password `expresslrs` or Launch ExpressLRS Wifi TX as well as TX hotspots from TX
ELRS configurator app: build RX and TX
You don't have to re-flash ELRS after betaflight firmaware update

Packet rate: 50hz -> longerst range
Telemetry: `STD`
Switch Mode: `Wide`

# Goggles
double tap and hold to power on
https://rotorriot.com/pages/downloads - range mod
# File transfer from Goggles and O3
- goggles - card reader
- o3, connect via USB, no batter required
# BetaFlight

## scripts


```sh
# HD display
set osd_displayport_device = MSP

# GPS rating on OSD
set osdgp-TODO

# soft landing
set exlanding

# DO NOT FORGET TO SAVE
save
```
## Ports
 after update RX port might need to be enabled

## 4.5 notes

via CLI
```
set osdgp-TODO
set exlanding
```

## 4.3 backup

### ports

UART2 used by TX/RX
<img width="1497" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/1708ea95-128d-4e90-89fe-06190ce8c635">

### configuration

<img width="1495" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/3fd6537b-5350-48fe-b28e-85e46c3f05ed">
<img width="1490" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/7966c5df-4164-471c-977f-480a0578c804">

### power / battery

<img width="1515" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/899fb0f3-8623-4267-8c47-959f7a4a1ffd">

### Presets

<img width="1482" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/252fc61d-8a3e-42bd-87f4-4ebd918f7381">


### PID

- throttle and motor setting
	- -> reduce from 100% to 60-70%. don't go below 50%
	- dynamic idle value `20` * 100

https://youtu.be/ShnYLKmFTog?si=69ox5MCgrGgmayzk
<img width="1479" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/c1537a08-2814-4025-bbca-19d5aa713751">
<img width="1473" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/8c1d3a51-7035-4428-9f55-e91ba0667b73">

### RX

<img width="1490" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/b7fc47fe-72cf-4286-904d-9e1bebb8b22c">

### Modes

<img width="1504" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/cefe335a-d256-49e8-a68f-eba5301dc3a7">

### Motors

<img width="1511" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/50aa458c-f2ea-49b8-8f44-d1dfa9b4da0b">

### OSD

- DJI goggles 2 (not o3 unit) need to be on v1.05 firmware
- in order to enable
```
get osd_displayport_device

set osd_displayport_device = MSP
```
<img width="1498" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/b2e11603-24a0-42ae-a21f-f4af48b5320e">

### LED

<img width="875" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/f085c38c-c11e-4c89-bba9-60f590cace83">

### BlackBox

<img width="1509" alt="image" src="https://github.com/friendlyantz/friendlyantz/assets/70934030/2c4b2304-c323-4241-8473-3308fac3fd6e">



## Rescue


- Stage 1
	- throttle: `set 1275`
- Stage 2
	- landing throttle `1230` tbc
	- horizontal speed 5-8m/s, same vertical?
# Info

- youtube - SetUP Chimera
