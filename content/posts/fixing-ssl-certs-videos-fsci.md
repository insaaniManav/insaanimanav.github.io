+++
title = "Fixing ssl certs on videos.fsci.in"
date = "2021-03-11"
draft = false
author = "Manav"
tags = ["linux", "docker", "ssl", "sysadmin"]
+++

A couple of days ago we got a message on the matrix channel of videos.fsci.in that the certs had expired and hence people had to access it via HTTP — which of course doesn't look good for the community. The certs were set to autorenew but for some reason they didn't.

So I decided to fix the certs today and document the process live for some reason.

I have no idea how this machine was configured since I wasn't around back then and we have no documentation whatsoever, so it is also my first time digging into this. I am hoping to fix the certs issue and then write some docs for people after me.

---

The first thing I noticed is that there is neither Apache nor Nginx installed on the main machine, which might mean that the setup is dockerised.

```bash
docker ps
```

I found out that a bunch of containers were running and one of them was **Traefik**, which is probably serving the entire thing.

So now let's try and get into the Traefik container to see how these certs were working:

```bash
docker exec -it f1895ec8ab20 /bin/bash
docker exec -it f1895ec8ab20 /bin/sh
```

Both returned an error saying no such command, which means there is no shell in these containers.

My next instinct was to do a simple `docker inspect` on the Traefik container to get it to reveal its secrets:

```json
"com.docker.compose.config-hash": "",
"com.docker.compose.container-number": "1",
"com.docker.compose.oneoff": "False",
"com.docker.compose.project": "peertube",
"com.docker.compose.service": "traefik",
"com.docker.compose.version": "1.25.0"
```

This confirmed it was set up using docker-compose. The `Binds` section told me the volume mounts were somewhere in `/root/peertube`:

```bash
sudo ls -l /root/peertube
total 80
-rw-r--r-- 1 root root 70600 Jun  9  2020 docker-compose.log
-rw-r--r-- 1 root root  2404 Jun  9  2020 docker-compose.yml
drwxr-xr-x 8 root root  4096 Jan  7 14:26 docker-volume
```

Which means it was set up using docker-compose itself. Nice.

So, as every IT person ever, let's try restarting all the containers:

```bash
docker-compose restart
```

Boom. The certs got renewed.

Thanks for sticking around for this very short adventure of mine.
