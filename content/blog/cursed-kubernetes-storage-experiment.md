---
title: "How NOT to setup a RWX storage for Kubernetes cluster - SMB share on Mikrotik router"
date: 2025-01-17T21:53:32+01:00
draft: false
cover: "images/2025-01-15-cursed-storage/cover.jpg"
tags: ['en', 'kubernetes', 'lab']
---

Some time ago I decided to setup seperate homelab network in my house, as my previous _homelab_ server is more like a production homeserver which I don't want to disrupt. It is not much, but I wanted to build something small, energy-efficient and from the parts I already had.

But back to the topic that made me curious - I kinda needed some persistent storage for pods in lab. Then I remembered that the Mikrotik (RB951G-2HnD) router i'm using have a USB port and I saw an option in winbox to create SMB share. I think you can image where it is going.

# Setup Kubernetes cluster

For Kubernetes I will be using single-node Talos Linux. Didn't even used Terraform or anything fancy to deploy it, just created a virtual machine on Proxmox with nocloud image.  
What is a Talos Linux? Talos Linux is a minimal, immutable linux distribution that is designed for Kubernetes - it doesn't even have a SSH, and you do all the configuration over API.  

As this writeup is not a tutorial, I will skip setup of Talos Linux, but if i got you interested, take a look on [Talos Linux Getting Started guide](https://www.talos.dev/v1.9/introduction/getting-started/)

# Setup Samba share on Mikrotik
The cover picture was kinda a clickbait - for storage I will be using a slow microSD card, as at the time of writing, i didn't had any USB-powered storage that was stable on my mikrotik (i guess 2,5" SSDs or HDDs might be a bit too power hungry, or my adapters were not compatible with ROS)

I'm not a RouterOS expert, additionally the forum.mikrotik.com at the time of writing was down

![nginx error page showing 504 gateway timeout on mikrotik forum](images/2025-01-15-cursed-storage/Screenshot_20250115_221834.png)

but this _minor inconvenience_ will not prevent me from trying anyway.

First, let's see if our USB drive is detected
```
[admin@MikroTik] > disk print
Flags: B - BLOCK-DEVICE; M, F - FORMATTING; p - PARTITION
Columns: SLOT, MODEL, SERIAL, INTERFACE, SIZE, FREE, FS
#     SLOT        MODEL                     SERIAL                    INTERFACE                  SIZE           FREE  FS
0 B   usb1        MXTronics MXT USB Device  130818v01                 USB 2.00 480Mbps  7 744 782 336
1 BMp usb1-part1                            @1'048'576-7'744'782'336                    7 743 733 760  7 518 978 048  ext4
```
Cool, there is my USB. I pre-formatted this card on my PC to ext4.

```
[admin@MikroTik] /ip/smb/shares> add name=kube directory=/usb1-part1
[admin@MikroTik] /ip/smb/shares> print
Flags: * - DEFAULT
Columns: NAME, DIRECTORY, MAX-SESSIONS
#   NAME  DIRECTORY   MAX-SESSIONS
;;; default share
0 * pub   /pub                  10
1   kube  /usb-part1            10
[admin@MikroTik] > /ip/smb/shares/enable numbers=1
```

Next i enabled the SMB and added user
```
[admin@MikroTik] > /ip/smb/set enabled=yes
[admin@MikroTik] > /ip/smb/set interfaces=bridge
[admin@MikroTik] > /ip/smb/users/add name="cluster" password="pleasedontlook" read-only=no
```

And i tried to connect to share from my desktop
![](images/2025-01-15-cursed-storage/Screenshot_20250115_230537.png)
Success!

# Setup SMB CSI on Kubernetes

As of CSI driver setup, I just installed the driver using kubectl, following [official documentation](https://github.com/kubernetes-csi/csi-driver-smb/blob/master/docs/install-csi-driver-v1.16.0.md)

Created the StorageClass, statefulset and persistent volume for testing aaand...

```
csi-provisioner     Mounting command: mount
csi-provisioner     Mounting arguments: -t cifs -o dir_mode=0777,file_mode=0777,uid=1001,gid=1001,<masked> //192.168.200.1/kube /tmp/pvc-873655b4-c46b-4ef8-95db-7fa00a603be6
csi-provisioner     Output: mount error(95): Operation not supported
csi-provisioner     Refer to the mount.cifs(8) manual page (e.g. man mount.cifs) and kernel log messages (dmesg)
csi-provisioner  >
```

oh. In Talos console i can see, more specific error
```
kern:    info: [2025-01-15T22:18:38.016242704Z]: CIFS: Attempting to mount \\192.168.200.1\kube
 kern:     err: [2025-01-15T22:18:38.026805704Z]: CIFS: VFS: \\192.168.200.1 Dialect not supported by server. Consider  specifying vers=1.0 or vers=2.0 on mount for accessing
 older servers
```
So in smb StorageClass I added 
```
mountOptions:
- vers=2.0
[...]
```

and look!
```
❯ kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
storage-test-0   Bound    pvc-873655b4-c46b-4ef8-95db-7fa00a603be6   2Gi        RWO            smb            <unset>                 18m
❯ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-873655b4-c46b-4ef8-95db-7fa00a603be6   2Gi        RWO            Delete           Bound    default/storage-test-0   smb            <unset>                          2m41s

```
we got our PV on smb share provisioned!

Now let's enter the pod and do some basic tests. As i never used SMB CSI driver before, i was surprised that in `mount` i can see the whole path to smb server, but that makes sense.
```
/ # mount
[...]
//192.168.200.1/kube/pvc-873655b4-c46b-4ef8-95db-7fa00a603be6 on /mnt type cifs (rw,relatime,vers=2.0,cache=strict,username=cluster,uid=1001,noforceuid,gid=1001,noforcegid,addr=192.168.200.1,file_mode=0777,dir_mode=0777,soft,nounix,serverino,mapposix,rsize=65536,wsize=65536,bsize=1048576,echo_interval=60,actimeo=1,closetimeo=1)
```

# Testing out our monstrosity
Let's at first point out all the bottlenecks
* Using MicroSD card (class 4) in USB adapter
* Using not that powerful routerboard
* Using CIFS (+on linux)

## Random Read
As the first test, i used `fio` to perform random read
```
/mnt # fio --name=randread --ioengine=libaio --iodepth=16 --rw=randread --bs=4k --direct=0 --size=100M --numjobs=4 --runtime=240 --group_reporting
randread: (g=0): rw=randread, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=16
...
fio-3.38
Starting 4 processes
randread: Laying out IO file (1 file / 100MiB)
randread: Laying out IO file (1 file / 100MiB)
randread: Laying out IO file (1 file / 100MiB)
randread: Laying out IO file (1 file / 100MiB)
Jobs: 4 (f=4): [r(4)][100.0%][r=1893KiB/s][r=473 IOPS][eta 00m:00s]
randread: (groupid=0, jobs=4): err= 0: pid=26: Wed Jan 15 22:38:23 2025
  read: IOPS=382, BW=1531KiB/s (1567kB/s)(359MiB/240006msec)
    slat (usec): min=1159, max=600406, avg=8302.61, stdev=5622.61
    clat (usec): min=8, max=905809, avg=124681.33, stdev=28987.76
     lat (msec): min=6, max=912, avg=132.98, stdev=30.01
    clat percentiles (msec):
     |  1.00th=[  106],  5.00th=[  110], 10.00th=[  112], 20.00th=[  115],
     | 30.00th=[  117], 40.00th=[  120], 50.00th=[  122], 60.00th=[  125],
     | 70.00th=[  128], 80.00th=[  132], 90.00th=[  138], 95.00th=[  144],
     | 99.00th=[  159], 99.50th=[  186], 99.90th=[  684], 99.95th=[  869],
     | 99.99th=[  894]
   bw (  KiB/s): min=   56, max= 2408, per=100.00%, avg=1926.10, stdev=42.95, samples=1526
   iops        : min=   14, max=  602, avg=481.48, stdev=10.73, samples=1526
  lat (usec)   : 10=0.01%, 20=0.01%, 50=0.01%
  lat (msec)   : 10=0.01%, 20=0.01%, 50=0.02%, 100=0.10%, 250=99.59%
  lat (msec)   : 500=0.15%, 750=0.03%, 1000=0.09%
  cpu          : usr=0.13%, sys=0.65%, ctx=103998, majf=0, minf=102
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=99.9%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.1%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=91847,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=16

Run status group 0 (all jobs):
   READ: bw=1531KiB/s (1567kB/s), 1531KiB/s-1531KiB/s (1567kB/s-1567kB/s), io=359MiB (376MB), run=240006-240006msec
```
During the test, i saw around 60% of CPU utilization on router, so actually we might not be yet bottlenecked by router.
I'm actually quite suprised by the IOPS number

## Random Read/Write

```
/mnt # fio --randrepeat=1 --ioengine=libaio --gtod_reduce=1 --name=test --filename=random_read_write.fio --bs=4k --iodepth=64 --size=100M --readwrite=randrw --rwmixread=75
test: (g=0): rw=randrw, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=64
fio-3.38
Starting 1 process
Jobs: 1 (f=1): [f(1)][100.0%][eta 00m:00s]
test: (groupid=0, jobs=1): err= 0: pid=35: Wed Jan 15 22:43:11 2025
  read: IOPS=180, BW=722KiB/s (739kB/s)(75.1MiB/106574msec)
   bw (  KiB/s): min=    8, max= 2280, per=100.00%, avg=1046.69, stdev=887.64, samples=147
   iops        : min=    2, max=  570, avg=261.67, stdev=221.92, samples=147
  write: IOPS=59, BW=239KiB/s (245kB/s)(24.9MiB/106574msec); 0 zone resets
   bw (  KiB/s): min=    7, max=  824, per=100.00%, avg=407.48, stdev=284.06, samples=125
   iops        : min=    1, max=  206, avg=101.86, stdev=71.02, samples=125
  cpu          : usr=0.18%, sys=1.80%, ctx=21840, majf=0, minf=7
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=99.8%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.1%, >=64=0.0%
     issued rwts: total=19233,6367,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=64

Run status group 0 (all jobs):
   READ: bw=722KiB/s (739kB/s), 722KiB/s-722KiB/s (739kB/s-739kB/s), io=75.1MiB (78.8MB), run=106574-106574msec
  WRITE: bw=239KiB/s (245kB/s), 239KiB/s-239KiB/s (245kB/s-245kB/s), io=24.9MiB (26.1MB), run=106574-106574msec
```
yeah, not that surprising.

## Real-world application test?

I really wanted to make a benchmark using Postgres `pgbench`, but Postgres will just hang on creating initial data, so i gave up. I think we already know how good it would perform with the current setup.

## End words
Actually, the setup process on both RouterOS and Kubernetes part was pretty painless and i didn't even need to spend multiple hours on troubleshooting. Would I recommended that setup for file storage on PVC? Definitely not, but in a pinch, it might actually be useful as a shared storage for e.g. small configuration files that need R/W access (otherwise just use configmaps).


## References

https://learningbytutz.blogspot.com/2019/12/adding-hard-drive-to-mikrotik_25.html
https://dotlayer.com/how-to-use-fio-to-measure-disk-performance-in-linux/

