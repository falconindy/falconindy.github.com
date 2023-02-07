---
title: "Optimizing Bootup With mkinitcpio"
layout: article
categories: articles
---

Recently, I've seen a bunch of questions to the tune of "how do I cut back on the number of modules in my initramfs?" To be brutally honest, this is sort of an annoying question. In general, the type of person who asks this question doesn't understand what the autodetect hook is doing and fails to realize that it's doing a 90% effective job of exactly this. lsinitcpio would have happily shown you exactly what's on the image. In addition, this sort of mindless pruning doesn't really cut back on boot time (stop using xz compression!) and only serves to remove functionality from your initramfs. In case you're still bent on doing this the manual way, I'll outline what's involved.

h2. What's it take to boot?

Perhaps surprisingly little. In the simplest case, mounting your root partition requires drivers for:

* storage bus (PATA, SATA, SCSI, etc)
* block device
* filesystem

For purposes of simplicity, I'll ignore things like mdadm, lvm, and crypto stacks. With only 3 modules (plus dependencies), the kernel is able to discover your storage bus, create a block device for the drives attached, and understand the underlying filesystem. It's really that simple. In my case, if I didn't have these boiled into my kernel, it'd be simply: ahci, sd_mod, and ext4.

h2. How do I know what I need?

There's a variety of tools at your disposal to figure this out. For starters, 'mkinitcpio -M' will scan your PCI bus and probe your root filesystem, returning a tidy list of modules, sans dependencies. You can be almost sure that you'll find your answers in this list. This isn't very useful though. You might not recognize these modules by name. So, take a look at something like 'lspci -vk'. In particular, search for something like SATA or PATA in the output.

<pre>
<code>
00:1f.2 SATA controller: Intel Corporation 82801JI (ICH10 Family) SATA AHCI Controller
        Subsystem: ASUSTeK Computer Inc. P5Q Deluxe Motherboard
        Flags: bus master, 66MHz, medium devsel, latency 0, IRQ 66
        I/O ports at 9c00 [size=8]
        I/O ports at 9880 [size=4]
        I/O ports at 9800 [size=8]
        I/O ports at 9480 [size=4]
        I/O ports at 9400 [size=32]
        Memory at f7dfc000 (32-bit, non-prefetchable) [size=2K]
        Capabilities: [80] MSI: Enable+ Count=1/16 Maskable- 64bit-
        Capabilities: [70] Power Management version 3
        Capabilities: [a8] SATA HBA v1.0
        Capabilities: [b0] PCI Advanced Features
        Kernel driver in use: ahci
</code>
</pre>

This looks promising! It even tells you 'ahci' is in use. Add that to the pile. You might have a motherboard with both PATA and SATA connectors. Make sure you grab the driver for both. We can confirm which one is in use soon enough.

Figuring out your needed block device driver is actually not guaranteed to be straightforward. We can employ the use of udevadm to knowledgeably walk around sysfs, and in particular, walk up the chain from our root device to the PCI backplane. Assuming /dev/sda1 is your root device, check out the following:

<pre>
<code>
$ udevadm info --attribute-walk -n /dev/sda1 | grep 'DRIVERS=="[^"]'
    DRIVERS=="sd"
    DRIVERS=="ahci"
</code>
</pre>

This gives us confirmation that we chose wisely with ahci, and it points out 'sd'. Based on what 'mkinitcpio -M' tells us, we can make an educated guess that this is the 'sd_mod' driver. Add that to the pile.

For your filesystem driver, it should literally be the name of the filesystem in use. There's a few exceptions -- reiser4 being one of them. Chances are if it's not a 1:1 match, you're already familiar with this particular gotcha.

h2. Alternative brute force method

There's another way of doing this with a bit less guessing involved. Simply make sure that your initramfs image is properly setup to boot (make sure it has udev and drivers for your keyboard), and reboot, appending 'break=postmount' to your kernel commandline. When you arrive at the rootfs shell prompt, run 'lsmod'. Ignoring anything like usb input drivers, that's what you need to boot. You'll, of course, also see module dependencies listed. Modules like ahci use libahci for functionality shared elsewhere, and ext4 uses mbcache and jbd2.

h2. Trim the fat

With your module list ready to go, it's time to tear apart mkinitcpio.conf. Since you're explicitly finding and loading modules, you're going to be very light on hooks. Based on the above, you could put together the following config:

<pre>
<code>
#
# /etc/mkinitcpio.conf
#

MODULES="ahci sd_mod ext4"
BINARIES="fsck fsck.ext4"
HOOKS="base"
</code>
</pre>

And that's it. We don't need udev, since anything in the MODULES variable will be explicitly loaded. mkinitcpio is also kind enough to do dependency resolution for us. I still advise you to keep (or add?) fsck to your image, as checking your filesystem before it's even mounted is greatly beneficial. I'll leave it as an exercise to the reader to figure out additional modules for things like: usb keyboards or raid/lvm root devices.

For your maiden voyage, I highly recommend creating a separate image in case you've forgotten something.

<pre>
<code>
# mkinitcpio -g /boot/initramfs-linux-tiny.img
</code>
</pre>

Either add another entry to your bootloader, or feel free to modify it on the fly at bootup. If this image isn't sufficient and init won't mount your root, go back through sysfs another time and check your work. Pick through the output of 'mkinitcpio -M' and check over what the modules do with 'modinfo'. The description may not be very useful, but the path within the module directory can be very telling.

That's pretty much all there is to it. A little bit of understanding about your hardware and some familiarity with the common kernel modules can go a long way.
