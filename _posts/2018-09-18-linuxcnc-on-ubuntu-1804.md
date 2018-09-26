---
layout: post
title: "How to Set Up LinuxCNC on a Fresh Ubuntu 18.04 Install"
date: 2018-09-18 12:00:00 +0200
categories: linuxcnc
---
LinuxCNC live/install ready to use images are available on [the official website](http://linuxcnc.org/) but after installing it I ran into
issues with the networking interfaces (both wifi and ethernet would not show up with `ip addr`). I also find that I got
accustomed to the more modern UI of recent releases of Ubuntu VS the stock Debian Wheezy look which is what the current release
of LinuxCNC is shipping with.

So let's set up LinuxCNC on a fresh Ubuntu 18.04 install.

## Install Ubuntu 18.04
Head over to [Canonical's website](https://www.ubuntu.com/download/desktop) and download Ubuntu. Create a bootable USB with
[Etcher](https://etcher.io/), and follow the installation procedure.

Don't forget to install the SSH server if you plan on accessing your machine remotely:
```bash
sudo apt install openssh-server
```

Install the tools necessary to build software on your machine:
```bash
sudo apt install build-essential
sudo apt install git vim
```

## Compile and install the real time kernel
First we'll need to patch the kernel for realtime usage. Visit [https://www.kernel.org/](https://www.kernel.org/) and find the latest available stable kernel version number _version.major.minor_.
At the time of writing this post it was __4.18.8__.

Then head over to [https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/](https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/)
and find the _version.major_ directory previously determined. In this directory you will find the necessary patch to compile the kernel
with realtime capabilities. The exact _minor_ revision number might not be available, find the closest one, in my case it was __4.18.7__.

Now download the [4.18.7 kernel source](https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/linux-4.18.7.tar.gz)
(find all available versions [here](https://mirrors.edge.kernel.org/pub/linux/kernel/))
and the corresponding [4.18.7 RT patch](https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/4.18/patches-4.18.7-rt5.tar.gz)
(find all available versions [here](https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/))
in `/usr/src/kernels`:
```bash
mkdir /usr/src/kernels
cd /usr/src/kernels
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/linux-4.18.7.tar.gz
tar zxf linux-4.18.7.tar.gz
mv linux-4.18.7 linux-4.18.7-rt5
cd linux-4.18.7-rt5
wget https://mirrors.edge.kernel.org/pub/linux/kernel/projects/rt/4.18/patch-4.18.7-rt5.patch.gz
gzip -d patch-4.18.7-rt5.patch.gz
patch -p1 <patch-4.18.7-rt5.patch
```

Now that we have the kernel source properly patched, it's time to build it! Enable CONFIG_PREEMPT_RT and if new options pop
up, decide whether to include them or not.
```bash
cp /boot/config-`uname -r` .config
sudo apt install libncurses-dev bison flex
make menuconfig
```
Make sure that the **Processor type and feature -> Preemption Model** is set to _Fully Preemptible Kernel (RT)_, and disable debugging features in **Kernel Hacking**,
especially _Check for stack overflows_ in **Memory Debugging**.

You can now build the kernel and install it.
```bash
sudo apt install libefl-dev libssl-dev
# Compile
make -j 4 # Adjust based on the number of cores in your CPU
# Install kernel
make modules_install install
# Install headers (needed to build LinuxCNC)
make headers_install INSTALL_HDR_PATH=/usr
```

Reboot and select the new kernel when reaching Grub's screen.
```bash
reboot
```

If you don't see a Grub prompt, add or modify the following to `/etc/default/grub` (you'll need to sudo it):
```bash
GRUB_TIMEOUT_STYLE=menu
```
Then run `sudo update-grub`.

## Download and compile LinuxCNC
Clone LinuxCNC git repository, install dependencies and compile:
```bash
git clone git://github.com/linuxcnc/linuxcnc.git linuxcnc-dev
# Install missing dependencies, YMMV, use apt search
sudo apt install autoconf pkg-config libudev-dev libmodbus-dev libusb-1.0-0-dev gtk2.0 yapps2 intltool
sudo apt install tcl-dev tk-dev bwidget libtk-img tclx python-gtk2 libreadline-gplv2-dev
sudo apt install python-tk libboost-python-dev mesa-common-dev libglu1-mesa-dev libxmu-dev python-glade2
# Compile
cd linuxcnc-dev/src
./autogen.sh
./configure --with-realtime=uspace
make -j4
# Fix permissions
sudo make setuid
```

Run LinuxCNC tests:
```bash
source ../scripts/rip-environment
runtests
```

## Test system latency

### Benchmark 1 - LinuxCNC latency test
Run the latency test and make sure you get reasonable values, if not typically below 10us worst case latency,
read below for possible improvements.
```bash
latency-test # For worst case values (which is what we ultimately care about)
latency-plot # For a plot over time of the latency
latency-histogram # To plot a distribution of latencies

# Will run a Mesa 7i76e card, will only use the servo thread at 1kHz
latency-test 1ms 1ms
latency-histogram --nobase
```

### Benchmark 2 - cyclictest and stress-ng
Install cyclictest and stress-ng:
```bash
sudo apt install stress-ng rt-tests
```

First benchmark: you want to look at the **Max** value printed by cyclictest, which is the worst case latency in nanoseconds.
```bash
# Terminal 1
cyclictest --mlockall --smp --priority=80 --interval=200 --distance=0 --nsecs
# Terminal 2
stress-ng --cpu 4 --io 2 --vm 2 --vm-bytes 128M --fork 4 --timeout 10s
```

### BIOS options
Disable any options related to power management, hyper threading, C states...

### SMI
Clone smictrl repository:
```bash
git clone https://github.com/zultron/smictrl.git
sudo apt install libpci-dev
```

You will probably need to modify the Makefile to make it work (the implicit rule for building a program has a different order
for the linker flags):
```
CC = gcc
smictrl: smictrl.c
	$(CC) $(CFLAGS) smictrl.c -o smictrl $(LDFLAGS)
```

List and disable SMI flags:
```bash
sudo ./smictrl -v
sudo ./smictrl -s 0x0 # try to clear all
sudo ./smictrl -g -s 0x0 # try to clear all
```

## References
1. [https://www.osadl.org/latest-stable-quick-rt-preempt-kerne.realtime-kernel-installation.0.html](https://www.osadl.org/latest-stable-quick-rt-preempt-kerne.realtime-kernel-installation.0.html)
1. [https://www.osadl.org/Realtime-Preempt-Kernel.kernel-rt.0.html](https://www.osadl.org/Realtime-Preempt-Kernel.kernel-rt.0.html)
1. [https://github.com/LinuxCNC/linuxcnc](https://github.com/LinuxCNC/linuxcnc)
1. [http://linuxcnc.org/docs/devel/html/code/building-linuxcnc.html](http://linuxcnc.org/docs/devel/html/code/building-linuxcnc.html)
1. [https://forum.linuxcnc.org/38-general-linuxcnc-questions/33492-unexpected-realtime-delay-on-task-0](https://forum.linuxcnc.org/38-general-linuxcnc-questions/33492-unexpected-realtime-delay-on-task-0)
1. [https://rt.wiki.kernel.org/index.php/HOWTO:_Build_an_RT-application](https://rt.wiki.kernel.org/index.php/HOWTO:_Build_an_RT-application)
1. [http://linuxrealtime.org/index.php/Improving_the_Real-Time_Properties](http://linuxrealtime.org/index.php/Improving_the_Real-Time_Properties)
1. [https://wiki.linuxfoundation.org/realtime/documentation/howto/tools/cyclictest/start](https://wiki.linuxfoundation.org/realtime/documentation/howto/tools/cyclictest/start)
1. [http://people.seas.harvard.edu/~apw/stress/](http://people.seas.harvard.edu/~apw/stress/)