---
title: "High kworker CPU usage on idle linux system"
date: 2026-03-22T18:46:48+01:00
draft: false
tags: ['en', '2026', 'proxmox', 'linux']
---

I'm running an HP Z230 SFF as my little homeserver for the last 5 years. It is not the most powerful, but it was energy-efficient enough, and was doing great for the things I'm running on it, so I kept using it. Recently I noticed that despite being idle, the power usage was higher than normal. Normally the power usage with idle workload oscillates around 25-30W, but it was at nearly 60W.
On Proxmox I didn't see that any VM was using much CPU, so I dug deeper. In `top` on the host, I noticed that only one core was maxed out, and the process `kworker/1:3+usb_hub_wq` was using a lot of CPU.

After some research on the internet, I first found a suggestion to disable some interrupts:
https://unix.stackexchange.com/questions/588018/kworker-thread-kacpid-notify-kacpid-hogging-60-70-of-cpu

But that didn't fix the issue for me. I didn't see any high values in the list, and even after disabling the highest one (which had a value of about "120"), it didn't change anything.

The thing that fixed it for me was unloading the `xhci_pci` driver using `modprobe`:

```bash
$ modprobe -r xhci_pci
$ dmesg
[...]
[  183.933736] xhci_hcd 0000:00:14.0: remove, state 4
[  183.933749] usb usb2: USB disconnect, device number 1
[  183.934564] xhci_hcd 0000:00:14.0: USB bus 2 deregistered
[  183.934586] xhci_hcd 0000:00:14.0: remove, state 1
[  183.934590] usb usb1: USB disconnect, device number 1
[  183.939166] xhci_hcd 0000:00:14.0: USB bus 1 deregistered
```

After that, I took a look at `top` and I didn't see the `kworker` again in the list of top CPU usage processes, so I added the driver to the blacklist, as I'm not using USB 3.0 devices on this host.
```bash
echo "blacklist xhci_pci" >> /etc/modprobe.d/pve-blacklist.conf
```
Please note that this behaviour is probably caused by some kind of hardware failure. As this is my test/home server, I'm accepting that fact, but in today's market, it is pretty hard to find a good deal for a replacement, newer homeserver due to the memory pricing, so good ol' Haswell with DDR3 will need to live for even longer.

Ref: https://community.frame.work/t/tracking-kworker-stuck-at-near-100-cpu-usage-with-ubuntu-22-04/23053/52
