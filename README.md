# Dockerize Your Dev

Starting point for turning a single VM into a containerized cloud development suite or lightweight container hosting environment. You can also just take the bones and basically do whatever you want with it.

Deploy any number of docker containers to almost any flavor Linux VM, as long as it supports `docker` and `docker-compose` your golden (although some may prove more difficult than others). Currently this repo is being developed on a [Ubuntu 18.04 LTS Minimal](https://cloud-images.ubuntu.com/minimal/releases/bionic/release/) VM on GCE ([n1-standard-2](https://cloud.google.com/compute/docs/machine-types)).

All the monthly fees for git, logging, monitoring, error monitoring, alerting, CI, etc etc really start to add up don't they... Well cancel a bunch of monthly fees with me, plus you don't even need to spend 100s of hours debugging to get it all working.

## Roadmap

- [x] Portainer (gui)
- [x] ELK Stack (logging)
- [x] cadvisor (monitoring)
- [ ] Grafana/Prometheus (monitoring)
- [ ] Gitea (git)
- [ ] Sentry (errors)
- [ ] Mattermost (chat)
- [x] NGINX (proxy)
- [x] LetsEncrypt (safety)
- [ ] Code Server (editor)
- [ ] Bit (code)

That is the goal anyways, its all containerized so add or subtract as you see fit.

Unfortunatly this setup with the SSL provisioning is not localhost friendly, also I am not actively supporting installing or running this on anything except a linux VM or if you are slick and run linux as your everyday OS you can get it working locally pretty easily. If you want to try it on Windows be my guest but I bet it won't be fun.

## Pre-Setup

1. On your host VM install `docker` and `docker-compose`, if you can't get this going probably this stack is not what your looking for anyways.

2. On the host make sure /etc/sysctl.conf has vm.max_map_count set to at least 262144 - `vm.max_map_count=262144`

3. Point all the subdomains you will be using to the public IP of your host VM, these are examples of what you may want to use.

* kibana.example.com
* cadvisor.example.com
* alertmanager.example.com
* portainer.example.com
* grafana.example.com
* prometheus.example.com

4. Go through all the compose.yml files in the root directory and find-replace your configured domains for NGINX/SSLs and email address for LetsEncrypt if you want emails from them about alerts on your SSLs.

```bash
find -type f -name "*-compose.yml" | xargs sed -i "s/example.com/yourdomain.com/g"
find -type f -name "*-compose.yml" | xargs sed -i "s/you@youremail.com/yourname@yourdomain.com/g"
```

5. (Optional) Setup / change / remove / add proxy configurations in the proxy/conf.d folder, they will all be mounted inside your NGINX container and used.

## Setup

1. Install the proxy

```bash
docker-compose -f proxy-compose.yml up -d
```

2. Install the logging

```bash
docker-compose -f logging-compose.yml up -d
```
** Optional ** Setup basic auth for kibana

```bash
sudo sh -c "echo -n '[username]:' >> /var/lib/docker/volumes/dockerize-your-dev_htpasswd/_data/kibana.example.com"
sudo sh -c "openssl passwd -apr1 >> /var/lib/docker/volumes/dockerize-your-dev_htpasswd/_data/kibana.example.com"
```

3. Install the monitoring

```bash
docker-compose -f monitoring-compose.yml up -d
```
** Optional ** Setup basic auth for cadvisor and alertmanager

```bash
sudo sh -c "echo -n '[username]:' >> /var/lib/docker/volumes/dockerize-your-dev_htpasswd/_data/cadvisor.example.com"
sudo sh -c "openssl passwd -apr1 >> /var/lib/docker/volumes/dockerize-your-dev_htpasswd/_data/cadvisor.example.com"
sudo sh -c "echo -n '[username]:' >> /var/lib/docker/volumes/dockerize-your-dev_htpasswd/_data/alertmanager.example.com"
sudo sh -c "openssl passwd -apr1 >> /var/lib/docker/volumes/dockerize-your-dev_htpasswd/_data/alertmanager.example.com"
```

4. Install the docker GUI

```bash
docker-compose -f gui-compose.yml up -d
```

5. Visit Portainer configured URL to setup your admin account UN and PW

#### Basic Auth

```bash
sudo sh -c "echo -n '[username]:' >> /var/lib/docker/volumes/dockerize-your-dev_htpasswd/_data/${VIRTUAL_HOST}"
sudo sh -c "openssl passwd -apr1 >> /var/lib/docker/volumes/dockerize-your-dev_htpasswd/_data/${VIRTUAL_HOST}"
```

## Attribution

https://github.com/jwilder/nginx-proxy

https://github.com/buchdag/letsencrypt-nginx-proxy-companion-compose

https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion

https://github.com/vegasbrianc/prometheus
