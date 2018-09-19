---
layout: post
title: "Install LinuxCNC on a fresh Ubuntu 18.04 install"
date: 2018-09-18 12:00:00 +0200
categories: linux cnc linuxcnc
---
LinuxCNC live/install ready to use images are available on [the official website](http://linuxcnc.org/) but after installing it I ran into
issues with the networking interfaces (both wifi and ethernet would not show up with `ip addr`). I also find that I got
accustomed to the more modern UI of recent releases of Ubuntu VS the stock Debian Wheezy look which is what the current release
of LinuxCNC is shipping with.

So let's set up LinuxCNC on a fresh Ubuntu 18.04 install.

# Compiling and installing the kernel
First we'll need to patch the kernel for realtime usage. Visit [https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/](https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/)
and find the latest available stable kernel version number _version.major.minor_ (which should be displayed prominently [on this page](https://www.kernel.org/)).
At the time of writing this post it was __4.18.8__.

Then head over to [https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/](https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/)
and find the _version.major_ directory previously determined. In this directory you will find the necessary patch to compile the kernel
with realtime capabilities. The _minor_ revision number might not be available, find the closest one, in my case it was __4.18.7__.

Now download the [kernel source](https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/linux-4.18.7.tar.gz) and the corresponding [RT patch](https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/4.18/patches-4.18.7-rt5.tar.gz) in `/usr/src/kernels`:
```bash
mkdir /usr/src/kernels
cd /usr/src/kernels
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/linux-4.18.7.tar.gz
tar zxvf linux-4.18.7.tar.gz
mv linux linux-4.18.7 linux-4.18.7-rt5
wget https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/4.18/patch-4.18.7-rt5.patch.gz
gzip -d patch-4.18.7-rt5.patch.gz
patch -p1 <patch-4.18.7-rt5.patch
```

Now that we have the kernel source properly patched, it's time to build it! Enable CONFIG_PREEMPT_RT and if new options pop
up, decide whether to include them or not.
```bash
cp /boot/config-`uname -r` .config
make menuconfig
make
make modules_install install
```

Reboot and select the new kernel.