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

* Table of contents (do not remove this line)
{:toc}

# Via Proxmox

[Great video tutorial](https://www.youtube.com/watch?app=desktop&v=1Un4zJJWUTE)

## Get the correct image

[make sure you get specific Proxmox image, not just x86 install or other VM image via this link](https://www.home-assistant.io/installation/alternative)

## Unzip and Copy image to the Proxmox server

use rsync, scp - refer my SSH tutorial

## Create VM

## copy the image to the VM

```sh
qm importdisk 100 /root/haos_image local-lvm --format qcow2
```
