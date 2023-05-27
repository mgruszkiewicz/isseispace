---
title: "Installing Linux on Chromebook (and other Apollo Lake devices)"
date: 2023-05-23T00:24:56+02:00
draft: false
cover: "images/2023-05-23-linux-on-apollo-lake/cover.jpg"
tags: ['linux', 'chromebook']
---

In this post i will focus on replacing the ChromeOS with Linux. Before i was running Eupnea on my chromebook, it was working alright, but the boot splashscreen was quite annoying (and recently i encourage a ‘softbrick’ after kernel update).

If you want to dualboot linux+chromeos i would sugest to stick with Eupnea on external storage, like fast SD card or USB stick.

## What do i need?

- Some patience
- Two USB drives - one with UEFI compatible linux of your choice - I will install EndeavourOS via Ventoy because i’m too lazy to configure a decent looking arch desktop on my own, and a second one to backup stock firmware. Technically you can do that on the same drive, but just to be safe, I recommend two separate ones.
- Enabled Developer mode, and disabled WP

## Step zero - validate coreboot support for your device

Visit [https://mrchromebox.tech/#devices](https://mrchromebox.tech/#devices) website and check if your device have support for ‘UEFI Firmware’. You probably can also do the same thing using WP_LEGACY, but i decided to just replace firmware in my chromebook.

## First step - installing coreboot

To install coreboot you have to disable WP (write protection), on chromebooks with CR50 chip you need to use SuzyQ cable or disconnect the batery and power up the laptop. You also need to have enabled developer mode in ChromeOS.

1. Open terminal (CTRL+ALT+T) in ChromeOS
2. Enter `shell`
3. Execute this command

```bash
cd; curl -LO mrchromebox.tech/firmware-util.sh && sudo bash firmware-util.sh
```

It will download and execute the firmware utility script.

1. Select second option `Install/Update UEFI (Full ROM) Firmware`
2. Utility will inform you about the possible consequences. Read it carefully.
3. The utility will also backup your existing firmware if you decided to go back to stock firmware. You need to connect a USB stick as a storage device for backup.
4. Proceed with the process.

You can find more information on [mrchromebox website](https://mrchromebox.tech/#fwscript)

After a successful flash, you can reboot your machine - you should see coreboot logo. Connect pendrive with OS and press ESC on boot screen. Enter `Boot Manager Menu` and select your device. Install your Linux distro as normal.

## Dummy output? Where is audio?

This is not a case for all chromebooks, but at least with my apollo lake based chromebook (HP Chromebook 13 G6 with Intel Celeron N3040) some audio things are missing from kernel. Not a expert on audio subsystem, but that is very annoying.

To have working audio on our linuxed chromebook we will use `audio-script` from Eupnea Linux Project.

You need to have `git` and Python <3.10 installed.

```bash
git clone https://github.com/eupnea-linux/audio-scripts
cd audio-scripts
./setup-audio --force-avs-install
```

The `--force-avs-install` is needed because AVS driver might be unstable and might damage speakers if you set the volume too high.

Just follow the prompts and install AVS driver. I also recommend installing headphone autoswitch.

### Troubleshooting not working speaker/headphones autoswitch
Check if `avs-auto-switcher` service is running
```
➜  ~ systemctl --machine=$(whoami)@.host --user status avs-auto-switcher
○ avs-auto-switcher.service - Automatically switch between avs headphones and speakers
     Loaded: loaded (/usr/lib/systemd/user/avs-auto-switcher.service; enabled; preset: enabled)
     Active: inactive (dead)

```
if it is inactive, try enabling it using command 
```
systemctl --machine=$(whoami)@.host --user enable --now avs-auto-switcher
```
and check status once again
```
➜  ~ systemctl --machine=$(whoami)@.host --user status avs-auto-switcher      
● avs-auto-switcher.service - Automatically switch between avs headphones and speakers
     Loaded: loaded (/usr/lib/systemd/user/avs-auto-switcher.service; enabled; preset: enabled)
     Active: active (running) since Fri 2023-05-26 16:02:22 CEST; 27s ago
   Main PID: 16402
      Tasks: 2 (limit: 4522)
     Memory: 772.0K
        CPU: 1.092s
     CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/avs-auto-switcher.service
             ├─16402 /bin/bash /usr/local/bin/avs-auto-switcher
             └─16403 acpi_listen

```
## Keyboard layout

EndeavourOS and a couple of other distros have builin layout for chromebook media keys, but if your distro is missing that layout, you can use script from 
[fascinatingcaptain](https://github.com/fascinatingcaptain)

```bash
git clone https://github.com/fascinatingcaptain/CBFixesAndTweaks
cd CBFixesAndTweaks
# make backup of original pc config file
sudo cp -n /usr/share/X11/xkb/symbols/pc /usr/share/X11/xkb/symbols/pc.bck

# copy new pc config file
sudo cp pc /usr/share/X11/xkb/symbols/

# update config
sudo rm -rf /var/lib/xkb/*
```

On my EndeavourOS (xfce4) install out-of-the-box keyboard brightness control was not working despite enabling that option in power management (and working in the battery settings), so i installed `light` and just bind these buttons to commands

By default, `light` require root, but you can `sudo chmod +s /usr/bin/light`, and it should be able to control backlight.

![XFCE keybinding settings](images/2023-05-23-linux-on-apollo-lake/keybinding.png)

## Enjoy?

On my Chromebook everything else was working fine OOTB.

- Sleep (after turning on lid close action in settings)
- HDMI output over USB-C
- SD Card reader
- Wi-Fi, Bluetooth
- Battery life looks on par with ChromeOS one
