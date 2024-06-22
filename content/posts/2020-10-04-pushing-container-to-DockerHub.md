---
title: "Pushing container to Docker hub"
description: "a short intro how to push containers to docker hub"
categories: [docker, registry, howto]
tags: [docker, registry, howto]
date: 2020-10-04

---

First build an docker container and run it afterwards so you see it with `docker ps -a`
As an example, I use the container for the [MagpiDownloader](https://github.com/joergi/MagPiDownloader)
```bash
docker build -f Dockerfile . -t mag-pi-downloader
docker run -v $(pwd)/issues:/app/issues/ mag-pi-downloader -f 131
```

After that is done, have a look, for all the docker container:
```bash
$ docker ps -a
CONTAINER ID   IMAGE               COMMAND                  CREATED              STATUS                          PORTS     NAMES
ea901f38e328   mag-pi-downloader   "/bin/sh /app/entryp…"   About a minute ago   Exited (0) About a minute ago             exciting_feistel
```

As you can see the newest has the container id `ea901f38e328`

Now login to docker hub

```bash
docker login
```

and commit the container id
```bash
docker commit ea901f38e328 joergi/mag-pi-downloader
```

tag it with the current version and also as latest
```bash
docker tag joergi/mag-pi-downloader:latest joergi/mag-pi-downloader:v.0.4.6
```

You can see it now:
```bash
$ docker images
REPOSITORY          TAG       IMAGE ID       CREATED              SIZE
joergi/mag-pi-downloader   latest    f05223f9e942   About a minute ago   131MB
joergi/mag-pi-downloader   v.0.4.6   f05223f9e942   About a minute ago   131MB
```

now push it:
```bash
docker push joergi/mag-pi-downloader:latest
docker push joergi/mag-pi-downloader:v.0.4.6
```

after that, you see that both have the same sha:
```bash
$ docker push joergi/mag-pi-downloader:v.0.4.6
docker push joergi/mag-pi-downloader:latest
The push refers to repository [docker.io/joergi/mag-pi-downloader]
1c1b759e6be0: Layer already exists 
e6aea274ef6a: Layer already exists 
543eb78604ea: Layer already exists 
4f310c1102f1: Layer already exists 
e57e91b0668b: Layer already exists 
9fc74be027cd: Layer already exists 
b5472f656374: Layer already exists 
256d88da4185: Layer already exists 
v.0.4.6: digest: sha256:3cd8adc5eb542a673b56cc56c2cee8898542f6f5e2a0bdca88950d65fb24ed13 size: 2194
The push refers to repository [docker.io/joergi/mag-pi-downloader]
1c1b759e6be0: Layer already exists 
e6aea274ef6a: Layer already exists 
543eb78604ea: Layer already exists 
4f310c1102f1: Layer already exists 
e57e91b0668b: Layer already exists 
9fc74be027cd: Layer already exists 
b5472f656374: Layer already exists 
256d88da4185: Layer already exists 
latest: digest: sha256:3cd8adc5eb542a673b56cc56c2cee8898542f6f5e2a0bdca88950d65fb24ed13 size: 2194
```

and this is how it looks in [my docker hub](https://hub.docker.com/repository/docker/joergi/mag-pi-downloader/tags?page=1&ordering=last_updated)
![image](https://github.com/joergi/blog/assets/1439809/1fd77b3b-0542-4519-99e4-57a3c0b97933)

