---
# layout: posts
# author_profile: true
# title: "Vi Everywhere"
permalink: /kamal/
# excerpt: "tbc"
# last_modified_at: 2016-11-03T11:13:12-04:00
collection: learning
tags:
  - software_engineering
  - learning
  - kamal
# toc: true
# toc_sticky: true
# toc_label: "My Table of Contents"
# toc_icon: "cog"
# header:
#   video:
#     id: _mrF0pp8LIE
#     provider: youtube
---


WIP
# TODO notes

```sh
kamal traefik reboot
```
- deploy lock
- SSL / Traefik SSL with Kamal [^2]
- Traefik will be replaced by Kamal Proxy in Kamal v2 [^3]

- from VM
```sh
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"

sudo docker logs -f container-id
```
- or remote
```sh
kamal app logs -f
```
- ssh limitations: if you have remote machines with different users than root and they are vary
	- dokku, cap rover, capistrano
---

# What Kamal offers

- zero-downtime deploys, 
- rolling restarts, 
- asset bridging, 
- local and remote builds, 
- auxiliary services management, 
- nice shortcuts for managing Docker in production. 
Kamal is Linux-flavour and language agnostic and will deploy any containerized application since it’s just a wrapper around Docker. (c)[^1]

---

Kamal is built around 
1. SSH, 
2. Docker, and
3. Traefik,

and is designed to deploy to servers that are already behind a load balancer by default. It can bootstrap your servers with Docker so you can use Kamal without a dedicated provisioning tool, but it’s not designed to do more than that.

---

# Limitations

1. can't run multiple applications on the same server.
2. No state of your servers beyond your configuration file. (Because of this, you cannot count on Kamal to decommission containers if you delete them from the config) [^1]
3. cannot auto-provision servers dynamically like Docker native Swarm mode
	1. ->  it doesn’t support auto-scaling.
4. No Load Balancer.

---
# Terminology

- service: A name for the service you are about to deploy, a name of your application
- server: A virtual or physical host running your application image
- role: A server role assigned to a group of servers running the same command like web or job
- accessory: A long-running auxiliary service like a database or microservice with an independent lifecycle
- volumes: A local mount directory or a Docker volume    
- directory: A local mount directory that’s created automatically

---
# Workflow / TLDR

```sh
gem install kamal
kamal init # init kamal deploy yaml. 
# Configure deploy.yml
# Configure ENV to include KAMAL_REGISTRY_PASSWORD(i.e. your Docker Hub) and RAILS_MASTER_KEY

kamal setup # 1st build and deploy, but can be 
kamal deploy # deploy update versions
```

---

 `kamal setup`:

1. ensures Docker is installed,  
2. pushes your environment from the environment files, 
3. boots accessories like databases,  
4. runs deploy step
---

 `kamal deploy` 

- starts a build process that will build the application out of a Dockerfile and push the image to the container registry with a service label and tag of the git commit (this label is required)
- boots Traefik on the hosts with the web role
- starts a new health check container named healthcheck-[SERVICE]-[VERSION] on port 3999 and checks if it passes the health check
- removes the health check container and checks for stale containers
- issues the application boot process with the new version
- removes old containers and images.

---

# health check `cord`

A cord file is created inside `.kamal/cords/[SERVICE]-web-[HASH]` on the host.

The container is started with this cord file mounted inside at `/tmp/kamal-cord` and the health check is modified to check for this file.
```sh
docker run (...)
	--volume $(pwd)/.kamal/cords/[SERVICE]-web-[HASH]:/tmp/kamal-cord 
	--health-cmd "(...) && (stat /tmp/kamal-cord/cord > /dev/null || exit 1)"
```

1. the requested version is tagged in Docker as :latest  
2. new assets are extracted  
3. asset volumes are synced  
4. old version is renamed to [OLD_VERSION]_replaced_[HASH] 
5. a cord is tied
6. the new version is run  
7. the old version cord is cut
8. the old version is stopped
9. assets are cleaned up

---
# Hooks

- pre-connect: Before connecting to hosts  
- pre-traefik-reboot: Before rebooting Traefik  
- post-traefik-reboot: After rebooting Traefik  
- docker-setup: After Docker is installed during bootstrap 
- pre-build: Before the image is built  
- pre-deploy: Before the application boot  
- post-deploy: After the application finished booting

---

# Deploy config

```yml
service: kamal-test
image: friendlyantz/kamal-test
servers:
	- 192.168.1.46
registry:
  username: friendlyantz
  password:
    - DOCKER_ACCESS_TOKEN=${DOCKER_ACCESS_TOKEN} # fetch from ENV
env:
  secret:
    - RAILS_MASTER_KEY # fetch from .env (avoid .env at all costs)
  clear:
    RAILS_LOG_TO_STDOUT: true # optional
```
---

# DNS

```sh
nslookup -q=ns yourwebsite.io
```
---
# Server Roles

1. web
2. job
3. scheduler
Kamal will only ever run one instance for each role on each specified host. 
If you need parallelism like running more workers on a single host, you have to implement it inside your image.

---
deploy.yaml

```yml
  # config/deploy.yml

...

  servers:
    web:
      hosts:
        - 165.232.112.197
      port: "3000:3000"
      cmd: bin/rails server
	  labels:
		traefik.http.routers.pinned-web.rule: Host(`pinnedjobs.com`)
		traefik.http.routers.pinned-web-secure.entrypoints: websecure
		traefik.http.routers.pinned-web-secure.rule: Host(`pinnedjobs.com`)
		traefik.http.routers.pinned-web-secure.tls: true
		traefik.http.routers.pinned-web-secure.tls.certresolver: letsencrypt
      options:
        network: "private"
```
---
options for CPU / Memory

```yml
# config/deploy.yml
...
servers:
	web:
		hosts:
		    - 165.232.112.197
		options:
			cpus: 1.5 
			cpuset-cpus: "0,3"
			memory: 1g
			memory-swap: 1.5g # actual swap is 0.5g,since 'mempry-swap' in this case is the amount of combined memory and swap that can be used.
			
			# We can also pass swappiness per container group by passing --memory-swappiness, from 0 (off) to 100 (maximum swappiness). Swap has to be enabled and exist for this to work.
```

Always leave a little bit of dedicated resources unassigned for debugging remotely

---

# Builder

```yml
kamal deploy -- version=[TAG]
```

```
kamal app images
```
---
# ENV
```
  # config/deploy.yml

...
env: 
	clear:
	    KEY_NAME: "value"
    secret:
		- TOP_SECRET
```


```sh
kamal envify
kamal envify --skip-push
```

An initial kamal setup does push the initial environment automatically, however, subsequent kamal deploy won’t push any environment variables on its own.

```sh
kamal env push
kamal app boot # redeploying to get env vars
```

```sh

```
---
---

----

[^1]: https://strzibny.gumroad.com/l/kamalbook
[^2]: https://github.com/basecamp/kamal/discussions/112
[^3]: https://github.com/basecamp/kamal/compare/main...kamal-proxy
