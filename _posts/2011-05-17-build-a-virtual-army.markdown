---
title: "Building a Virtual Army"
layout: article
categories: articles
---

Recently, in testing my latest new toy (geninit), I've needed to create a variety of different root device setups to put geninit through the proverbial ringer. Up until a few weeks ago, this would have been done through VirtualBox. However, I've never really been a huge fan of VirtualBox, increasingly due to my opposition of Oracle. The management is fairly straight forward, but the machines themselves feel fairly limited, and don't take full advantage of processor extensions like VT-x. A few years back, it was even the case that VirtualBox was recommending *not* to enable VT-x at all, as it would actually hinder performance.

The other obvious option is "QEMU":http://www.linux-kvm.org/, with KVM support. On a few occasions, I've poked around with QEMU, but was never really satisfied. Turns out, I really just didn't give it enough time.

QEMU has a lovely feature that allows it to emulate a serial console -- effectively giving you a VM in a terminal. In addition, it supports the "virtio":http://wiki.libvirt.org/page/Virtio family of devices, which allows for much better performance, particularly in the realm of I/O, where the typical bottlenecks lie. Now things start to get more appealing. With a little bit of bash, and a fair bit of time with some documentation, things were starting to come together. I figured I'd share what I came up with, in case anyone else finds themselves in a similar situation.

h2. Initial Setup

You'll need a few packages to get started: qemu-kvm, vde2, and iptables for now. You'll also, of course, want a liveCD for your favorite distro. Start by creating a qcow2 image, which will serve as the disk for your VM:

{% highlight bash %}
qemu-img create -f qcow2 imagename.qcow2 5G
{% endhighlight %}

qcow2 is the QEMU image format of choice, which supports compression, encryption, dynamic sizing, copy on write, and snapshots. There are other formats, but this is by far the winner for versatility. Creation should be instant.

{% highlight bash %}
modprobe -a kvm-intel tun virtio
{% endhighlight %}

Note that I'm using intel, but there also exists kvm-amd for you other folks. Make sure your processor actually supports this -- you can grep for 'vmx' in /proc/cpuinfo, which will hopefully return results in your processor flags. You'll also want to make sure that /dev/kvm is created with 'kvm' as group. Arch Linux provides a udev rule to do this by default. Your mileage may vary. Make sure that you add yourself to the kvm group and log out for the changes to take effect.

h2. Networking

We're going to be using "VDE":http://vde.sourceforge.net for networking support which will essentially create an internal VLAN for our guests. Start by creating the gateway for the VLAN:

{% highlight bash %}
vde_switch -tap tap0 -mod 660 -group kvm -daemon
{% endhighlight %}

This launches vde_switch, which creates a new network device: tap0. It doesn't yet have an IP, so we'll need to assign it:

{% highlight bash %}
ip addr add 10.0.2.1/24 dev tap0
ip link set dev tap0 up
{% endhighlight %}

Note that I could have picked any "RFC 4193 internal address":http://en.wikipedia.org/wiki/Private_network, just as long as its not the same network as my LAN.

With our gateway created, we need to allow traffic to forwarded properly through it:

{% highlight bash %}
sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -s 10.0.2.0/24 -o eth0 -j MASQUERADE
{% endhighlight %}

A few points of interest here. The first is that you'll want to add both these commands to a file that will be routinely read on bootup. I'll leave it up to the reader as an exercise as to find the distro specific and recommended location for these. The second is that the iptables rule should be allowing any traffic whose source is the same as network our gateway device's IP. The output for the rule, specified by -o, doesn't necessarily need to be defined. In the case of my laptop which sometimes uses wlan0 and sometimes usb0, I left this undefined and routing rules take care of finding the correct path.

h2. Launch It!

With networking in place and disks ready, we should be all set to launch the VM. I'd suggest making a small script out of this, as it'll be useful later on:

{% highlight bash %}
#!/bin/bash

mem='-m 1024'  # RAM allocation
cpus='-smp 2'  # CPU allocation
net='-net nic,model=virtio -net vde' # networking using virtio & VDE
drive='-drive file=/path/to/qcow,if=virtio' # disk image we created

qemu-kvm $mem $cpus $net $drive -cdrom /path/to/livecd.iso -boot d
{% endhighlight %}

Note that you don't currently have the ability to acquire an address for guest's network device via DHCP. I'll cover that later as an optional feature. For now, just assign a static IP from within the guest:

{% highlight bash %}
ip addr add 10.0.2.100/24 dev eth0
ip link set dev eth0 up
ip route add default via 10.0.2.1
echo 'nameserver 8.8.8.8' >> /etc/resolv.conf
{% endhighlight %}

Note how we're using the IP of the host's tap0 device as our default route, and assigning an IP on the same subnet. Install as per usual. Before rebooting, make sure that the serial console is setup. It needs to be defined in your bootloader, on the kernel command line, and possibly as a getty. There's quite a few flavors for the moving pieces here -- some simple googling should quickly lead to results.

Once the serial console is setup, you can boot the VM with the -nographic option, which should happily dump output into your terminal.

h2. DHCP Server

Because I'm lazy, I decided that my little VLAN needs dhcp. The official ISC dhcp server is one option, and requires very little setup, but I was convinced that using dnsmasq was a better solution. It provides a lightweight DHCP server as well as DNS caching, which my desktop can benefit from as well. With dnsmasq installed from your trusty repositories, fire up your favorite editor and open /etc/dnsmasq.conf. We only need to make a few small changes. dnsmasq needs to be set to listen on addresses and/or interfaces, as well as to specify a dhcp range. My chosen settings were:

{% highlight bash %}
listen-address=127.0.0.1
interface=tap0
dhcp-range=10.0.2.100,10.0.2.150,12h
{% endhighlight %}

Now I can add 127.0.0.1 as the primary nameserver for my desktop, and guests on my VLAN will have dhcp as well as the cached DNS goodness. You could easily take this one step further and add in a DNS server for the VLAN. The addition of dynamic DNS updates would be excellent for keeping track of your soldiers. A more blunt approach would be to simply create DHCP reservations. Note that for this approach, you would need to manually pick out MAC addresses for each of the VMs as they're always the same 52:54:00:12:34:56 across all guests.

h2. Tying it All Together

And last but not least, I expanded on the bash script I described earlier to start up the VM. It's posted in my bin-scripts repo on "Github":https://github.com/falconindy/bin-scripts/blob/master/qinit. Rather straight forward -- define a function called vm_GUESTNAME for each VM and add the appropriate options. All that's required is the drives array.

