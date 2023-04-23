---
# layout: posts
# title: "Vi Everywhere"
permalink: /home-assistant/
excerpt: "home-assistant notes via Proxmox"
# last_modified_at: 2016-11-03T11:13:12-04:00
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
---

> WIP

The way I see the most optimal setup is to have several dedicated VMs:
1. for Home Assistant OS (HAOS gives you convenient add-on system, unlike Docker);
1. for Linux to host IoT stack (your DB, Grafana and other services);

and then use the Home Assistant integration to connect to your other devices VMs. This way, you can have a single point of entry for all of your devices, and you can use the Home Assistant UI to control them.

Action Plan:
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
- [ ] Install Zigbee2MQTT on Linux VM;

---

* Table of contents (do not remove this line)
{:toc}

# Install HAOS

[Great video tutorial on how to install HAOS on Proxmox](https://www.youtube.com/watch?app=desktop&v=1Un4zJJWUTE)

# Notes / Tips
## Get the correct image

[make sure you get specific Proxmox image, not just x86 install or other VM image via this link](https://www.home-assistant.io/installation/alternative)

## copy the image to the VM and create a VM(follow video tutorial)
use `rsync`, `scp` - refer my SSH notes

```sh
qm importdisk 100 /root/haos_image local-lvm --format qcow2
```

# Automations

## Laundry
