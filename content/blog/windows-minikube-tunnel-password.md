---
title: "Password prompt while trying to use 'minikube tunnel' (docker@127.0.0.1's password:)"
date: 2022-11-03T10:40:56+02:00
draft: false
cover: "images/2022-11-03-minikube-tunnel/cover.webp"
tags: ['minikube', 'kubernetes', 'k8s', 'windows']
---
## Issue

```
$ minikube tunnel
✅  Tunnel successfully started
📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...
🏃  Starting tunnel for service balanced.
docker@127.0.0.1's password:
```

## Solution
You need to add minikube's ssh key to your ssh agent
```
ssh-add C:\Users\user\.minikube\machines\minikube\id_rsa
```


## Issues with ssh-add

### Cannot use ssh-add, agent not running
Make sure ssh-agent is running
```
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

### Permissions for 'C:\Users\user\.minikube\machines\minikube\id_rsa' are too open.

In case you get "Permissions for 'C:\\Users\\user\\.minikube\\machines\\minikube\\id_rsa' are too open." you might need to copy `id_rsa` to desktop and change permissions
[https://superuser.com/a/1296046](https://superuser.com/a/1296046)