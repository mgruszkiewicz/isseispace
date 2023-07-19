---
title: "Noob is trying to build Android kernel 4.14.x"
date: 2023-07-16T14:37:07+02:00
draft: true
tags: ['linux', 'android', 'en']
---
At the beggining of this article i would like to mention that i'm a noob when it comes to android custom kernel or rom development, so some of this informations can be obvious for some of you, but it was relatively hard to find one place with up-to-date information on this topic.

## Backstory - why i wanted to compile kernel for my phone?
My current phone is Xiaomi Mi 9 - i'm very happy with this phone, it is more than fast enough for my everyday needs, photo quality (espeically on gcam) is also pretty good. My main issue with this phone was under MIUI 12 - it had a couple of annoying bugs that will probably never get fixed:
* Sometimes, despite being muted, when taking a screenshot - the stock android sound and popup will appear.
* The screenshots are a bit darker than on screen (as the screenshot is captured at the first frame of the animation...)
* Sometimes the MIUI Gallery app is trying to index photos, and it might come across corrupted image - in that case it would just drain your battery and peg CPU retrying - solution: factory reset the phone or disallow access to storage/uninstall the appear

I decided that life is too short to bother with buggy software that i don't have a control on, and i didn't want to change phone, because beside this grapes it was working great!
I unlocked the bootloader, and install PixelOS 13 - since then - the phone is running better than on stock MIUI (feels quicker + battery last longer). Huge thanks to the mainteiner of PixelOS for cepheus - [balgxmr](https://forum.xda-developers.com/m/balgxmr.9196567/).

So - why i want to build kernel from scratch? Because after last OTA update there was no KernelSU-compatible kernel on telegram channel. Yes - i could just ask if he could release flashable zip with KSU, but why not try to build it myself?

KernelSU is a root solution that works in kernel mode and grant root permission to userspace application directly in kernel space. Compared to Magisk, i didn't had any issues with using mobile banking apps, google pay or McDonald's app (which was always the hardest to trick that my device was not rooted...)

## What do i need to build a kernel?
* Preferably Ubuntu-based distro
* Kernel source
* anykernel3 (optional - to create flashable zip)
* aarch64 toolchain
* arm32 toolchain
* patience

As a example for this writeup i will be using [kernel_xiaomi_cepheus](https://github.com/balgxmr/kernel_xiaomi_cepheus)

## File structure

using `--depth 1` to only download latest changes, as I only want to compile the kernel without any modifications (as the kernel from balgxmr already have a branch with KernelSU patch).
```
mkdir workdir && cd workdir
git clone https://github.com/balgxmr/kernel_xiaomi_cepheus --depth 1
git clone https://github.com/balgxmr/anykernel3 anykernel
git clone https://github.com/threadcolor/aarch64-linux-gnu --depth 1
git clone https://github.com/threadcolor/arm-linux-gnueabi --depth 1
chmod +x -R kernel_xiaomi_cepheus/build.sh kernel_xiaomi_cepheus/scripts/
```

Before continuning i noticed that the kernel have dos character newline encoding, so to get it working under unix, i needed to use tool `dos2unix`
```
find . -type f -print0 | xargs -0 dos2unix
```
[source](https://stackoverflow.com/a/11929475)

At first i had no idea where to get toolchain, as [https://gitlab.com/PixelOS-Devices/playgroundtc](https://gitlab.com/PixelOS-Devices/playgroundtc) was missing some `gcc` binaries - got the idea from [issue #2 in repo](https://github.com/balgxmr/kernel_xiaomi_cepheus/issues/2).

That's why i decided to use another prebuild binaries

## Building Docker container
My first try was on my desktop running EndeavourOS (Arch based), but i was quicly faced with weird dependencies issues, so i decided to build a Docker container which was running Ubuntu 22.04.

Also i will be using prebuild toolchain.

My `Dockerfile`
```
# workdir/Dockerfile
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Installing necessary packages
RUN apt update && apt upgrade -y && \
    apt install git clang g++-11 bc llvm make \
    build-essential libncurses-dev bison flex \
    libssl-dev libelf-dev zip lld -y \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir /work

WORKDIR /work

CMD tail -f /dev/null

```
My `docker-compose.yaml`  
we can also just use `docker run`, as there would be only one container, but to keep it simple with volume mounts and variables i will use docker-compose
```
# workdir/docker-compose.yaml
version: '3.7'

services:
  builder:
    container_name: builder
    build:
      context: .
    volumes:
      - ./kernel_xiaomi_cepheus:/work/kernel_xiaomi_cepheus
      - ./aarch64-linux-gnu:/work/aarch64-linux-gnu
      - ./arm-linux-gnueabi:/work/arm-linux-gnueabi
      - ./anykernel:/work/anykernel
      - ./out:/work/out
```

Build and start the container
```
docker-compose up -d
docker exec -it builder bash
```

## Trying to build the kernel
Thankfully, the kernel source did contain [build.sh](https://github.com/balgxmr/kernel_xiaomi_cepheus/blob/fe075fd2f6dd52abc5b728f64c9f9ad3d0c4ff12/build.sh) that helped me to figure out what i need to actually compile the kernel. This script probably can be used also for other kernels (just remember to change `DEFCONFIG`)
In `build.sh` i needed to change a couple of variables
```
export CLANG_PATH=/work/aarch64-linux-gnu/bin
export CLANG32_PATH=/work/arm-linux-gnueabi/bin
export PATH=${CLANG_PATH}:${CLANG32_PATH}:${PATH}
export CROSS_COMPILE=${CLANG_PATH}/aarch64-linux-gnu-
export CROSS_COMPILE_ARM32=${CLANG32_PATH}/arm-linux-gnueabi-

# Paths
KERNEL_DIR=`pwd`
REPACK_DIR=/work/anykernel
ZIP_MOVE=/work/out
```

Now in `kernel_xiaomi_cepheus` directory we can try to execute `bash build.sh` and see results.
Depends on your hardware, it could take some time - on my AMD Ryzen 5 5600G system it took around 6 minutes.

If everything went well, you should have a flashable zip in `workdir/out` on your host. 
