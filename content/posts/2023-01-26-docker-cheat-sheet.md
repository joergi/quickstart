---
title: "Docker cheat sheet"
description: "a (hopefully always updated) docker cheat sheet"
categories: [docker, cheatsheet, howto]
tags: [docker,howto]
date: 2023-01-25

---

* Stop all containers - `docker stop $(docker ps -a -q)`
* Debug into running container - `docker exec -it container_name /bin/bash` 
* Debug into docker image - `docker run --rm -it image_name /bin/bash`
* Remove all docker dontainers - `docker rm -f $(docker ps -a -q)` 
* remove all images you need to download everything again) - `docker rmi -f $(docker images -q)`
* delete all docker network bridges - `docker network prune` 
