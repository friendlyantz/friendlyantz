---
permalink: /networking/
excerpt: my learnings and attempt to become CCNA chad
collection: learning
tags:
  - software_engineering
  - learning
  - networking
---

# DNS Reset

```sh
sudo resolvectl dns <interface> 1.1.1.1 1.0.0.1

# find interface via
ip link

# check
nslookup google.com
```

|            | Prime             | Secondary         |
| ---------- | ----------------- | ----------------- |
| CloudFlare | 1.1.1.1           | 1.0.0.1           |
| Troogle    | 8.8.8.8           | 8.8.4.4           |
| OpenDNS    | 208.67.222.222    | 208.67.220.220    |
| NordVPN    | 103.86.96.100<br> | 103.86.99.100<br> |
