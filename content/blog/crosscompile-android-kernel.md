---
title: "Noob is trying to build Android kernel 4.14.x"
date: 2023-07-16T14:37:07+02:00
draft: false
tags: ['linux', 'android', 'en']
---
At the beginning of this article, i would like to mention that i'm a noob when it comes to android custom kernel or ROM development, so some of these information may be obvious to some of you, but it was relatively hard to find one place with up-to-date information on this topic.

## Backstory - why i want to compile the kernel for my phone?
My current phone is the Xiaomi Mi 9 - i'm very happy with this phone, it is more than fast enough for my everyday needs, and the photo quality (especially on gcam) is also pretty good. My main issue with this phone was under MIUI 12 - it had a couple of annoying bugs that will probably never get fixed:
* Sometimes, despite being muted, when taking a screenshot - the stock android sound and popup will appear.
* The screenshots are a bit darker than on screen (as the screenshot is captured at the first frame of the animation...)
* Sometimes the MIUI Gallery app is trying to index photos, and it might come across a corrupted image - in that case, it would just drain your battery and peg CPU retrying - solution: factory reset the phone or disallow access to storage/uninstall the app

I decided that life is too short to bother with buggy software that i don't have a control on, and i didn't want to change phone (..aand probably i just like to mess with stuff), because aside from this, it was working great!
I unlocked the bootloader, and install PixelOS 13 - since then - the phone is running better than on stock MIUI (feels quicker + battery last longer). Huge thanks to the mainteiner of PixelOS for cepheus - [balgxmr](https://forum.xda-developers.com/m/balgxmr.9196567/).

So - why i want to build the kernel from scratch? Because after the last OTA update, there was no KernelSU-compatible kernel on telegram channel. Yes - i could just ask if he could release flashable zip with KSU, but why not try to build it myself?

KernelSU is a root solution that works in kernel mode and grant root permission to userspace application directly in kernel space. Compared to Magisk, i didn't had any issues with using mobile banking apps, Google Pay or McDonald's app (which was always the hardest to trick that my device was not rooted...)

## What do i need to build a kernel?
* Preferably Linux-based OS (but can also be done on Windows or Mac!)
* Kernel source
* anykernel3 (optional - to create a flashable zip)
* aarch64 toolchain
* arm32 toolchain
* patience

As a example for this write up i will be using [kernel_xiaomi_cepheus](https://github.com/balgxmr/kernel_xiaomi_cepheus)

## File structure

using `--depth 1` to only download the latest changes, as I only want to compile the kernel without any modifications (as the kernel from balgxmr already has a branch with KernelSU patch).
```
mkdir -p workdir && cd workdir
git clone https://github.com/balgxmr/kernel_xiaomi_cepheus --depth 1
git clone https://github.com/balgxmr/anykernel3 anykernel
git clone https://github.com/radcolor/aarch64-linux-gnu --depth 1
git clone https://github.com/radcolor/arm-linux-gnueabi --depth 1
chmod +x -R kernel_xiaomi_cepheus/build.sh kernel_xiaomi_cepheus/scripts/
```

Before continuing i noticed that the kernel has dos character newline encoding, so to get it working under unix, i needed to use the tool `dos2unix`
```
find . -type f -print0 | xargs -0 dos2unix
```
[source](https://stackoverflow.com/a/11929475)

At first i had no idea where to get toolchain, as [https://gitlab.com/PixelOS-Devices/playgroundtc](https://gitlab.com/PixelOS-Devices/playgroundtc) was missing some `gcc` binaries - got the idea from [issue #2 in repo](https://github.com/balgxmr/kernel_xiaomi_cepheus/issues/2).

That's why i decided to use another prebuild binaries

## Installing dependencies on Arch-based OS
Some packages are named differently in arch repository than in debian/ubuntu, here is a list of what you need to install:
```
sudo pacman -S clang gcc bc llvm make ncurses bison flex openssl libelf lld zip
```

## Alternative way: Using Docker
If you are facing any issues with installing the dependencies, or you are on another OS and want to quickly setup a build environment, you could use Docker for that.

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

It is using a Ubuntu 22.04 as a base image, and it just install the necessary build dependencies. Also as a small hack - to keep it from shutting down, it will `tail` /dev/null.

My `docker-compose.yaml`  
we can also just use `docker run`, as there would be only one container, but to keep it simple with volume mounts and building i will use docker-compose
```
# workdir/docker-compose.yaml
version: '3.7'

services:
  builder:
    container_name: builder
    build:
      context: .
    volumes:
      - ./:/work/
```
This `docker-compose` will build `Dockerfile` from current directory, and mount files we have in our `workdir` to `/work` inside our Docker container.


Build and enter the container
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

Now in `kernel_xiaomi_cepheus` directory, we can try to execute `bash build.sh` and see results.
Depends on your hardware, it could take some time - on my AMD Ryzen 5 5600G system it took around 6 minutes.

If everything went well, you should have a flashable zip in `workdir/out` on your host. 
