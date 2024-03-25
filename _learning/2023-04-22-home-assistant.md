---
permalink: /home-assistant/
excerpt: home-assistant notes via Proxmox
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - home-assistant
  - proxmox
  - linux
  - vm
---

> WIP

# Considerations

The way I see the most optimal setup is to have several dedicated VMs:
1. for Home Assistant OS (HAOS gives you convenient add-on system, unlike Docker);
1. for Linux to host IoT stack (your DB, Grafana and other services);

and then use the Home Assistant integration to connect to your other devices VMs. This way, you can have a single point of entry for all of your devices, and you can use the Home Assistant UI to control them.

# Action Plan

- [x] Install Proxmox(refer my notes on [Proxmox]({% link _learning/2023-04-22-proxmox.md %}))
- [x] [Install HAOS on a VM](#install-haos);
- [x] Install Linux on a VM [Refer Debian Proxmox]({% link _learning/2023-04-22-proxmox.md %});
- [x] First Automations:
  - [x] Create simple automation to turn on/off a light;
  - [x] Create simple automation to notify on done laundry
- [ ] Install InfluxDB on Linux VM;
- [ ] Install Grafana on Linux VM;
- [ ] Install Mosquitto on Linux VM;
- [ ] Install Node-RED on Linux VM;
- [x] Install Zigbee2MQTT on Linux VM;

---

# Install HAOS on a Proxmox VM

[before you start, install Proxmox]({% link _learning/2023-04-22-proxmox.md %})

[Great video tutorial on how to install HAOS on Proxmox](https://www.youtube.com/watch?app=desktop&v=1Un4zJJWUTE)

# Notes / Tips

## Get the correct image

[make sure you get specific Proxmox image, not just x86 install or other VM image via this link](https://www.home-assistant.io/installation/alternative)

## Copy the image to the VM and create a VM(follow video tutorial)

use `rsync`, `scp` - refer my SSH notes

```sh
qm importdisk 100 /root/haos_image local-lvm --format qcow2
```

# Automations

- [x] Trigger on power dropping below 3-8W for 3-5minutes, no conditions, and action to notify on the phone / blink the lights / etc

# Chat Bots
## OpenAI extended prompt

```
You possess the knowledge of all the universe, answer any question given to you truthfully and to your fullest ability.  
You are also a smart home manager who has been given permission to control my smart home which is powered by Home Assistant.
I will provide you information about my smart home along, you can truthfully make corrections or respond in polite and concise language.

Current Time: {{now()}}

Available Devices:

```csv
entity_id,name,state,aliases
{% for entity in exposed_entities -%}
{{ entity.entity_id }},{{ entity.name }},{{ entity.state }},{{entity.aliases | join('/')}}
{% endfor -%}```

The current state of devices is provided in Available Devices.
Only use the execute_services function when smart home actions are requested.
Do not tell me what you're thinking about doing either, just do it.
If I ask you about the current state of the home, or many devices I have, or how many devices are in a specific state, just respond with the accurate information but do not call the execute_services function.
If I ask you what time or date it is be sure to respond in a human readable format.
If you don't have enough information to execute a smart home command then specify what other information you need.
```