---
collection: learning
tags:
  - software_engineering
  - learning
  - media
---

options:
# opt1 - on top of LXC container install (ZeroTier will be a pain)
# opt2 - VM (preferred)
---
3. install [docker](https://docs.photoprism.app/getting-started/troubleshooting/docker/#installation), easiest way below:
```sh
bash <(curl -s https://setup.photoprism.app/ubuntu/install-docker.sh)
```
# config

- [ ] Ensure you change password for 1. Client and MarianDB from _insecure_ default password
- [ ] Also when mounting volumes be explicit and state full path i.e. `/home/friendlyantz/Pictures/photoprism_originals` instead of `~/Pictures/photoprism_originals` as the latter can put `Pictures` forlder under `/root` instead of `/home/friendlyantz`
```yml
      - "/home/friendlyantz/Pictures/photoprism_originals:/photoprism/originals"               # Original media files (DO NOT REMOVE)
      - "/home/friendlyantz/Pictures/Archive:/photoprism/originals/archive" # *Additional* media folders can be mounted like this
```
> Also, 1st line is for originals, 2nd for additional folders. Left side is your local path, right side is the path inside the container and will be displayed in UI are `/archive` in this case.

### docker compose up

```sh
sudo docker compose up -d
```

#### set up auto boot docker compose up in case of reboot/powerloss
```sh
sudo vi /etc/systemd/system/photoprism_docker_compose_up.service
```
update accordingly and add
```
[Unit]
Description=BootPhotoprismAfterPowerLoss

[Service]
Type=simple
ExecStart=docker compose up -d
User=root
Restart=no
WorkingDirectory=/home/friendlyantz

[Install]
WantedBy=multi-user.target
EOF
```
