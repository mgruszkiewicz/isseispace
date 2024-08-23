---
title: MicroSD Card not visible on boot in linux
date: 2024-08-23T19:37:07+02:00
draft: false
tags: ['linux', 'chromebook', 'en']
---
I'm still using my chromebook with coreboot as a portable thinclient. Due to small built-in storage (only 16GB), i'm also using microSD card as a secondary storage for larger files and project. However i was strugeling with microSD card not being mounted at boot (waiting for device), my solution was to just physically eject and insert the card at boot, but it is pretty inconvient and is probably destroying the card pins.

## The solution
The solution was to create a systemd unit, that will run on boot before `local-fs.target`

```
# /etc/systemd/system/rescan-sd-card.service
[Unit]
Description=Remove and rescan MMC and PCI devices before mounting filesystems
DefaultDependencies=no
Before=local-fs.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo 1 > /sys/class/mmc_host/mmc0/device/remove'
ExecStartPost=/bin/sh -c 'echo 1 > /sys/class/pci_bus/0000:00/rescan'
RemainAfterExit=yes

[Install]
WantedBy=local-fs.target
```
Note: check if on your device the sdcard is also `mmc0` - you can check it just by looking at `lsblk`.

After creating the file at `/etc/systemd/system/rescan-sd-card.service`, we need to enable it
```
sudo systemctl enable rescan-sd-card.service
```

And now, on boot, the system will 'remove' the mmc device, and rescan pci bus. That solved the issue and now my OS is booting without user intervention.

## Reference
https://unix.stackexchange.com/questions/710377/microsd-card-not-found-at-boot-time-works-when-ejected-and-reinserted
