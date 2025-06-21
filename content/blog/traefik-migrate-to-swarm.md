---
title: "FYI: My HTTP routes are not visible after migrating from regular docker to docker swarm!!"
date: 2025-06-21T13:19:19+02:00
draft: false
tags: ['en', 'docker', 'linux']
---

Recently, during migration from one host to another, we had a idea to run the docker in swarm mode, so i future, when we needed more capacity, we could just add more nodes "without the complexity of kubernetes". However, after starting the working docker-compose in the docker swarm mode, i noticed that the traefik was not picking up the labels from the containers and i couldn't really figure out why.

The docker provider was replaced in config file with docker swarm, but it still wasn't working.

In the end, it was a pretty silly mistake on my part - in regular Docker, you add labels to the containers like that

```
services:
  laravel:
    image: mylaravelapp:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.laravel.rule=Host(`api.my-app.local`)"
      [...]
```

However, in Docker Swarm, labels are defined under `deploy` section
```
services:
  laravel:
    image: mylaravelapp:latest
    deploy:
      placement:
        constraints:
          - node.role == manager
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.laravel.rule=Host(`api.my-app.local`)"
        [...]
```

This took me too long to solve, I hope this blog entry will help someone out
