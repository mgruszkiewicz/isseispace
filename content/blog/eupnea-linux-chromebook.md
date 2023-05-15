---
title: "Project Eupnea - native linux for chromebooks"
date: 2023-05-15T10:40:56+02:00
draft: false
cover: "/img/2023/eupnea.webp"
tags: ['chromebook', 'linux']
---

Let me start with that - second-hand chromebooks are awesome value for cheap, long-lasting battery, portable Netflix and SSH machine.

I’ve got my Chromebook about a year ago for 55 USD, it is a HP Chromebook 11a G6. I love how lightweight it is and that i can charge it from literally everything using USB-C ports (for real - it can even take charge from normal 5V USB charger). And the battery is awesome, it lasts a whole working day of moderate use.
I bought a chromebook mostly as an experiment, on how the ChromeOS works for “poweruser” - and to be honest, it isn’t that bad, mostly because of the Linux Debian container, but it is still very limiting.

So i went on the search of solutions of installing another OS on it - as i knew that pretty much any x86 chromebook can be flashed with coreboot from [mrchromebook.tech](http://mrchromebook.tech)

The issue are drivers for hardware. I really wanted to use some Linux distro, as i think it would work the best on hardware with that spec tier. But unfortunately, not all i2c hardware have great support, mostly audio, keyboard and touchpad. Actually on the Windows side things seems to be working a bit better, but i doubt Windows 10 on 16GB emmc would be a pleasant experience.

## Breath

Later i discover [Breath](https://github.com/cb-linux/breath) - it is a way to natively boot Linux without replacing chromebook firmware. All you need to do, is enable Developer Mode, create bootable USB and you can install it. Additionally, it is using Google ChromeOS kernel, so everything should be supported out-of-the-box. It does support creating bootable USB with

- Debian
- Ubuntu
- Arch
- Fedora

Unfortunately, the project was archived due to lack of active maintenance.

## Project Eupnea

[Eupnea](https://eupnea-linux.github.io/) is a fork of Breath, featuring all the features from Breath.

I decided to give it a try, first try was with EupneaOS - their own distro with a goal of mimicking ChromeOS look, and even Android emulator (via Waydroid). The distro is using KDE Plasma as a Desktop Environment.
I was pretty amazed on how booting process was painless - all it took was to enable developer mode in chromeOS and enable debugging options (that allow access to ChromeOS root shell), executing command that allow booting from USB unsigned OS, quick reboot, pressing CTRL+U, and in a couple of minutes i was on Linux KDE desktop running live from USB on my chromebook!

Later i decided to create my own image, based on Arch Linux and KDE Plasma (yeah, kde might not be the best choice for that class of hardware) - the process was pretty easy even on WSL2 environment. It generated a `.img` file that i could just `dd` onto flash drive.

Next couple of minutes, and i was on desktop of my image, next i decided to execute `install-to-internal` command, which, as you might guess, installs the OS to internal storage.

The best part of it - because you don’t need to replace the firmware - you can just go back to ChromeOS with recovery media.

After a reboot, i was on desktop of my arch install, running on chromebook - actually i wasn’t running that bad, considering that my chromebook have only a 6W TDP Celeron N3350. But one thing was missing - the audio.

### Audio setup

As my chromebook is running ‘apollo lake’ line of cpu i needed to use AVS drivers. The `setup-audio` script claim they are flaky and can burn out speakers, but personally I haven’t had any issues yet. Both the headphone jack and built-in speakers works.

### General usability

The first thing i wanted to do on my Linuxed chromebook was installation Steam. It can be done in Crostini, but on my 16GB EMMC, i was missing space and there aren’t any good ways of mounting external SD card in Crostini environment (there is a “Share for linux” option, but that limits transfer speeds to about 5MB/s…)

The game i wanted to try the most was Stardew Valley - and it works straight away!
But, i had before played this game via Crostini, and it was also working fine under ChromeOS.

Next thing was a internet browsing using Firefox, as day-to-day, it is my primary browser. It was also running pretty good, no stutters while playing youtube videos, Discord is also running “ok” - sometimes it takes a couple of seconds to switch serve. Netflix, after installing ‘netflix 1080p’ extension, also work great.

I’ve been running arch on my chromebook for about a month now, and i’m pretty happy with the result. It really brings up the usability of the laptop for me. The battery is also performing on par with the chromeos.