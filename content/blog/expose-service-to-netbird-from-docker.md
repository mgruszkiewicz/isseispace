---
title: "Exposing service from Runpod.io (and others) to netbird network"
date: 2025-10-05T18:31:27+02:00
draft: false
tags: ['en', 'linux', 'docker']
---
Sometimes you need a better GPU for testing - service offerings like runpod.io or salad cloud is cool, for setup a quick docker container for testing, however currently both runpod and salad are missing the ability to setup some kind of sidecar, which would allow user to setup for example a vpn connection to internal network, so you can access the external container from your secure network and not just directly throught the internet.

Both services are offering a public ingress that will automatically route traffic to your containers, however, at least in the case of salad.com, you can only enable authentication by salad API key, which is pretty clunky while working with stuff like comfyui.

As a workaround to missing sidecar feature, you can do something, that is disgureged by pretty much anyone - run multiple processes in one container!
This solution is not perfect, and if you can, i would stick to sidecars, however, sometimes you do what you need to do.

In the beginning i mentioned two services - runpod.io and salad.com, however, that method should work everywhere, that you can run your own docker container and define some environmental variables.


## Okay, so how to do it?

The modification needed for docker image are not that big. We will be using `userspace` implementation in the netbird, instead of default kernel one.
As a example, i will be using `nginx` image, however you can use any other image as a base - you just need to define the correct startup command in the `start.sh` script.
{{< code language="docker" title="Dockerfile" >}}
FROM nginx:latest

# Netbird envs
ENV NB_FOREGROUND_MODE=true
ENV NB_USE_NETSTACK_MODE=true
ENV NB_ENABLE_NETSTACK_LOCAL_FORWARDING=true
ENV NB_DAEMON_ADDR=unix://netbird.sock

# Setup netbird
ADD https://github.com/netbirdio/netbird/releases/download/v0.59.2/netbird_0.59.2_linux_amd64.tar.gz /tmp/netbird.tar.gz
RUN cd /tmp/ && tar -xvzf /tmp/netbird.tar.gz && mv netbird /bin/ && chmod +x /bin/netbird

EXPOSE 80

WORKDIR /app

COPY start.sh .
RUN chmod +x /app/start.sh

CMD /app/start.sh
{{< /code >}}

{{< code language="bash" title="start.sh">}}
#!/bin/bash

netbird up --setup-key=$NB_SETUP_KEY --extra-dns-labels $NB_EXTRA_DNS_LABEL &

nginx -g "daemon off;"
{{< /code >}}

## What's going on here?

The trick is pretty simple - we're running netbird in userspace mode, which doesn't require kernel modules or special privileges. The `NB_USE_NETSTACK_MODE` flag tells netbird to use userspace networking instead of trying to create kernel interfaces, which is perfect for restricted container environments.

In the startup script, we launch netbird in the background with the `&` operator, then start nginx in foreground mode. The netbird process will connect to your netbird network using the setup key you provide, and from that point your container is accessible from other machines in your netbird network.

## Setting it up

You'll need to set two environment variables when launching your container:

- `NB_SETUP_KEY` - your netbird setup key (get it from your netbird dashboard)
- `NB_EXTRA_DNS_LABEL` - optional, but useful for giving your container a nice hostname in the network

For runpod, you can set these in the environment variables section when creating your pod. For salad, add them in the container configuration.

Once the container starts, give it a few seconds for netbird to establish the connection, then you should be able to access your service directly from any other machine in your netbird network - no public internet exposure needed!

## Final thoughts

Is this the cleanest solution? Nope. Does it work? Absolutely. Sometimes you just need to get stuff done, and this workaround lets you securely access your GPU workloads without exposing them to the entire internet or dealing with API key authentication in places where it doesn't make sense.

Just remember - if the platform adds proper sidecar support in the future, migrate to that. But until then, this gets the job done.
