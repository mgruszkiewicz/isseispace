---
title: "Solving issue with K3s System Upgrade exec /bin/upgrade.sh: exec format error"
date: 2024-08-02T22:37:07+02:00
draft: false
tags: ['linux', 'kubernetes', 'en']
---
Recently, I was trying to upgrade my toy K3s cluster using System Upgrade Plans operator. Applied example manifest, control-plane upgraded without issues, but pod for upgrading worker node was constantly restarting with error in logs `exec /bin/upgrade.sh: exec format error`.
I would _kinda_ expect that issue on ARM system (although we come a long way in the last couple of years!), but my nodes are running on x86 CPU, so what is happening?

# My troubleshooting steps

### Try to define image version with correct architecture
Looking at image manifest, it is a multi-arch image, so maybe we are pulling the wrong image (or there was an error on release)? I noticed that rancher is pushing single-arch images as well, so just in case I tried to define image for specific architecture (amd64)
```
apiVersion: upgrade.cattle.io/v1
kind: Plan
metadata:
  name: plan-k3s-agent-upgrade
  namespace: system-upgrade
spec:
  [...]
  upgrade:
    image: rancher/k3s-upgrade:v1.30.3-k3s1-amd64
```
The issue still persisted.

### Restarting node
![have you tried turning it off and on again?](images/2024-08-02-k3s/have-you-tried-turning-it-off-and-on-again.jpg)  
In my case, my K3s nodes are virtual machines running on Proxmox hosts, so I just tried to reboot the VM, but it didn't change anything.
I had a couple of PVC on that node which were using openebs-hostpath storage, so I didn't want to go straight to nuclear option (but kinda recommended option) to just delete old node and replace it with brand new one.

### Prune images on node
Maybe at first we pulled an image for a wrong architecture? Decided to prune unused images from that node, just to be safe.
```
root@k3s-2:~# crictl rmi --prune
Deleted: docker.io/rancher/k3s-upgrade:v1.30.3-k3s1
```

And after restarting the system upgrade operator, the update proceeded without issues, and both of my nodes were happily on v1.30.3.
```
➜ kg nodes
NAME         STATUS   ROLES                       AGE    VERSION
k3s-1   Ready    control-plane,etcd,master   138d   v1.30.3+k3s1
k3s-2   Ready    <none>                      138d   v1.30.3+k3s1
```

The fix in this case was pretty straight-forward, but maybe my troubleshooting steps will help someone in the future.
