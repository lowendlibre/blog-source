+++
title = "From Zero to Kernel"
date = "2023-08-09T20:27:55-04:00"
author = "JustSoup321"
authorTwitter = "" #do not include @
cover = ""
tags = ["Kernel", "postmarketOS", "downstream"]
keywords = ["", ""]
description = "The story of getting postmarketOS running on the TCL A3."
showFullContent = false
readingTime = true
hideComments = false
color = "" #color from the theme settings
+++

# From Zero to Kernel

![TCL A3 Booting pmOS](/posts/images/bangkok.jpg)

Hi! This is going to be the first real post of the lowendlibre blog, so just bear with me if anything is weird. My name is JustSoup321, the maintainer of the [TCL A3 (tcl-bangkok_tf)](https://wiki.postmarketos.org/wiki/TCL_A3_(tcl-bangkok_tf)), and-

Alright. I know you just want to hear about postmarketOS, let's get to the good stuff.

### Introduction

I have been interested in postmarketOS for a long time, have been daily driving it on my Oneplus 6T for about half a year now. Safe to say, I love it. Thus, I was interested in getting it working on one of my devices. I chose a tablet I had with an unlockable bootloader. Safe to say, the stock bootloader was so glitchy it overwrote the boot partition and broke it. Although, through that endeavor, I found MTKClient. MTKClient is a really nice tool that allows the user to manipulate MediaTek SOCs to gain access to their partitions and some other nice boot parameters. As such, I was able to unlock my only other MediaTek device that originally had a non-unlockable bootloader, the TCL A3.

### Counting Kernels

Now, that is amazing, but an unlocked bootloader does not equal a postmarketOS port. No, that was only the beginning. Next I had to find the kernel source in the first place. So I searched and eventually found the [MediaTek Common Kernel](https://github.com/MediaTek-Labs/common-kernel-4.19), which seemed to support my SOC. Of course, this didn't work, so I thought all was lost. Turns out, TCL release their kernels on [SourceForge](https://sourceforge.net/projects/tcl-mobile/files/), so I was actually able to find a kernel for my device.

With the kernel, I could actually begin the porting process. Here is where I met another issue. The kernel doesn't compile. It seems TCL must have either used some kind of proprietary compiler (which, I know, is against GPL don't yell at me) or had some weird glue code. This was very bad. Without a compiling kernel, it was as good as no kernel at all. Thus, I had to put on my best kernel dev hat (they aren't that expensive actually) and get to work.

### Mo' Kernel Mo' Problems

Bad news. I don't know C.

I had to use my knowledge of Rust and guesswork to be able to understand what each function does and what is wrong with it. You may ask: "why didn't you just mainline if the downstream kernel is broken?" Well, little Jimmy (give anyone reading named Jimmy a jumpscare), I couldn't. See, MediaTek mt6762, which the TCL A3 uses, has zero, and I mean ZERO mainline support. There is very basic support for the mt6765, which mt6762 is compatible with, but that isn't enough to get a booting system, let alone find a compatible device tree. No, I had to fix the kernel.

And fix it I did! I was actually able to get the kernel compiling after about 30 hours of work. (I was left with a lot of patches in my kernel folder.) So all I had to do was select the correct DTS, flash, and watch the device bootloop...

### The DTB...

That was odd to me. Why using the stock Android kernel with a DTS that is literally named after the device does the device not boot? I checked through kernel configs and I swear I reread every single postmarketOS wiki page 5 times, yet no luck. It wasn't until in the porting Matrix room somebody pointed me to using `tar tvf` on the Alpine kernel package (which uses .apk and has nothing to do with Android's .apk; are you confused yet?). After doing that, it turns out the kernel was hard-coded to compile with the wrong DTBs.

Why that is and who at TCL decided upon that? We will never know. What we do know is that the device will not boot with a DTB that only has 7 lines in it and is a placeholder. Thus, I needed to figure out how to get the correct DTS to be compiled. I tried editing makefiles, I tried editing kernel options, I even tried using Clang as a compiler. I did have a little success creating a bootimg with the precompiled Android kernel and Android DTB, but nothing worked except the USB being able to say it's serial number is postmarketOS. But still, nothing worked. Nothing. Then I had an idea.

What if I just deleted the offending DTS and renamed the DTS I need compiled to the previously deleted DTS? It was definitely a dirty hack, but it actually worked! The device booted, the postmarketOS splash screen showed up, it found the rootfs, and XFCE4 booted for a moment and then crashed!

### "If you are lucky, your screen may give some clues that you are booted into pmOS."

That was to be expected though. Android doesn't use xorg, so it is no surprise that the device isn't very well optimized for it yet. I had to do that manually. So I plugged in my usb cord and fired up ssh and... it hung. There was no output and I can put no input. I was able to use the password, and ssh did connect to the device, but somewhere along the way the device starts ignoring the usb requests and leaves ssh, both figuratively and literally, hanging. Thus, I couldn't debug at all, and no matter what I tried, I still couldn't get ssh nor telnet (which is just another protocol to get a shell if you don't know what it is) to work. It seems the downstream kernel has a broken usb stack. It barely even works under Android.

Thus, we are now at current day, with my only choice to be mainlining.

### Conclusion

This is not the type of posts I have planned for this blog. I am going to be doing more technical posts soon enough, but I thought it best if I started with a full run-down of the process I have gone through. 90 hours of my life spent porting this device, and I am excited for the next 90.

Thanks for reading! Tschüss!
