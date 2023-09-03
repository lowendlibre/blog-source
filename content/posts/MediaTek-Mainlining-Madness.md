+++
title = "MediaTek Mainlining Madness"
date = "2023-09-03T15:58:15-04:00"
author = "JustSoup321"
authorTwitter = "" #do not include @
cover = ""
tags = ["Kernel", "postmarketOS", "mainline"]
keywords = ["", ""]
description = "Progress report on the mainlining of the MediaTek mt6765 SOC."
showFullContent = false
readingTime = true
hideComments = false
color = "" #color from the theme settings
+++

# MediaTek Mainlining Madness

Long time no see! We have a lot to cover, so let's get right into it.

We have two devices booting the mt6765 kernel. The Nokia 3.1 Plus (Roo) and the TCL A3 (bangkok_tf). Out of these two devices, the Nokia 3.1 Plus has better support due to  being easier to get logs from.

### What is working?

You won't be replacing your daily driver just yet, but basic computer features are available.

- postmarketOS rootfs  
  - This is the big one. The kernel is able to load the rootfs from an sdcard. It might be able to load from the EMMC, but that is untested.
- USB 
  - Both connection and USB Networking works, allowing for internet connection.
- 4 CPU Cores
  - These are only half of the cores available on the mt6765, but they are enough for basic initialization.
- Haptics
  - I honestly have no idea why these work, but they do. Nice?
- SD-Card
  - The mounting of the sdcard and booting of operating systems of of it works.
- PMIC Buttons (Hardware Buttons)
  - You can actually click buttons and they do stuff. Wow.
- Real Time Clock
  - Basic ARM stuff that nobody cares about.
- Thermal
  - The CPU reports thermals of the device to the kernel. It doesn't report the correct values though, so this will require some tweaking.
- i2c
  - This was a pain to get working, but it was required for display.
- Backlight
  - The screen actually has brightness. Wow.
- Flashlight
  - Another one of those: I honestly have no idea why this works.
- Charger
  - The battery is able to get voltage higher than what it is discharging and can handle charging correctly. I am unsure if it is able to charge while suspended, as we don't have that implemented yet, so just assume it cannot.
- EMMC
  - This was a huge surprise to find working. Not even the mt6763 (which the vollaphone uses) has this working. Apparently the SOC handles EMMC in the same way it handles sdcards, so it was just a process of changing around a few values.
- Framebuffer
  - Simplefb works but causes a huge kernel panic, so it is almost useless for getting logs. It should be possible to fix it through some switching of the dts memory maps, but that is to be seen.

### Upstreaming

While I would love to be able to bring these patches to the true mainline Linux kernel, the way that the kernel handles the SOC is just incompatible with the main tree. Maybe in a few years time it would be possible if the code was cleaned up as much as possible, but even then, the dts files for the phones would not be accepted due to bootloader hacks.

### postmarketOS Upstreaming

On the other hand, JustSoup is in the process of writing a kernel package to send upstream to postmarketOS so new ports can make use of this mainline kernel fork. If you are interested in porting a new mt6765/mt6762 device to postmarketOS, keep an eye out for said kernel package, message JustSoup on Mastodon or Matrix, or just ask in the postmarketOS mainlining channel on Matrix.

### Conclusion

While this post was short, I hope it was enough to bring some light to our progress here at lowendlibre. If you have any questions, feel free to ask on the Mastodon post.

Until next time!
