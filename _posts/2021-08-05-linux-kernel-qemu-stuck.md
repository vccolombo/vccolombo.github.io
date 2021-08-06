---
title: "Linux Kernel stuck in QEMU: Decompressing Linux... Parsing ELF... forever"
excerpt: "Decompressing Linux... Parsing ELF... And nothing happens! What to do when a Linux Kernel does not work in QEMU."
date: 2021-08-05T22:05:30-03:00
categories:
  - Cybersecurity
tags:
  - Linux
  - Kernel
  - QEMU
---

I finally decided to learn about Linux kernel vulnerabilities! However, I won't talk about any vulnerability for now. Instead, I want to share a problem that I had while setting up my learning environment.

## The Problem

The problem happened when I compiled my own Linux Kernel version 4.10.6 (trying to follow [this guide](https://dangokyo.me/2018/10/11/linux-kernel-exploitation-setting-up-debugging-environment/). The blog is really good, go take a look later) and tried to make it work with QEMU. However, the kernel just stuck in what seemed an infinite loop of "Decompressing Linux... Parsing ELF...", with a lot of screen blinking. Below is a picture of the situation.

![Linux kernel stuck in QEMU](/assets/images/cybersec/linux-kernel-qemu-stuck.png)

Normally, what should happen is that this screen shows for a very brief time before continuing with a lot of boot messages, which is exactly what happened when I tested it with a 5.0+ kernel. So, I had an issue that only happened while trying to set up older kernels.

## Trying to find the cause

I will say already that I still haven't found the cause of the problem. My host machine was an Ubuntu 20.04, and I started by testing in which kernel versions the problem happened. I have confirmed (with some confidence) that no kernel < 5.0 was working while trying to run it in QEMU. However, I tried versions 5.4 and 5.10, and both presented no issues.

Then, I tried downgrading the version of some of the tools used. The first test was trying GCC version 7.5.0 (instead of 9.3, which was installed before). Long story short, it changed nothing. 

Then, I tried changing the QEMU version. Ubuntu 20.04 has QEMU 4 in its repositories, so I decided to downgrade it to QEMU 2. Again, no success.

At this time, I just decided to go full crazy and install Ubuntu 18.04 on my machine. And magically, everything worked. I tested compiling kernels 3.x, 4.x, and 5.x on the Ubuntu 18.04 host, and none of them presented issues.

However, I still couldn't find a way to make it work on Ubuntu 20.04. Nothing I did seemed to change the final result, the kernel was never booting.

To better narrow the cause, I compiled the 4.10.6 kernel in Ubuntu 18.04 and transferred the bzImage to the 20.04 installation. Running the kernel there, it worked and booted perfectly! So I narrowed the problem to something related to the compilation phase, and not a problem in QEMU.

## Fixing the issue

You are probably reading this to know how to fix the issue. Unfortunately, I don't know it either. It is probably related to a tool used in the kernel compilation having different versions in Ubuntu 18.04 and 20.04.

My suggestion is: first, make sure you are compiling it correctly. Follow the [syzkaller guide](https://github.com/google/syzkaller/blob/master/docs/linux/setup_ubuntu-host_qemu-vm_x86-64-kernel.md) on how to compile the kernel. If even so, it does not work, I suggest you find a way to do it in an Ubuntu 18.04 machine. Alternatively, you could install it on a Virtualbox VM, compile the kernel inside the VM, and transfer it back to the host.

## Bonus: PIC error when trying to compile an older kernel

Older kernels (I think it is 4.6.x and older, but don't quote me on that) do not support compilation with PIC/PIE enabled. However, recent versions of GCC enable this feature by default, which just breaks everything when you try to compile the Linux kernel. 

To fix the issue, you can apply [this patch](https://lists.ubuntu.com/archives/kernel-team/2016-May/077178.html), which works in kernel 4.x. I couldn't find a patch for older kernels, but it should be easy to change the Makefile and patch it manually.

To apply the patch, copy everything from the "From: ..." to "2.8.1" to a file `something.patch` inside the kernel folder, and run `git apply something.patch`. It should work now.