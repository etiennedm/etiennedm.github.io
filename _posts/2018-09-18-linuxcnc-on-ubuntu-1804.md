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

# Install Ubuntu 18.04
Head over to [Canonical's website](https://www.ubuntu.com/download/desktop) and download Ubuntu. Create a bootable USB with
[Etcher](https://etcher.io/), and follow the installation procedure.

# Compile and install the real time kernel
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
make menuconfig
```
Make sure that the **Preemption Model** is set to _Fully Preemptible Kernel (RT)_, and disable debugging features in **Kernel Hacking**,
especially _Check for stack overflows_ in **Memory Debugging**.

You can now build the kernel and install it.
```bash
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

# Download and compile LinuxCNC
Clone LinuxCNC git repository, install dependencies and compile:
```bash
git clone git://github.com/linuxcnc/linuxcnc.git linuxcnc-dev
cd linuxcnc-dev/src
./autogen.sh
# Install missing dependencies, YMMV, use apt search
sudo apt install pkg-config libudev-dev libmodbus-dev libusb-1.0-0-dev gtk2.0 yapps2 intltool
sudo apt install tcl-dev tk-dev bwidget libtk-img tclx python-gtk2 libreadline-gplv2-dev
sudo apt install python-tk libboost-python-dev mesa-common-dev libglu1-mesa-dev libxmu-dev
# Compile
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

# Test system latency
Run the latency test and make sure you get reasonable values:
```bash
latency-test # For worst case values
latency-plot # For a plot over time of the latency
```

Look into SMI issues: [http://wiki.linuxcnc.org/cgi-bin/wiki.pl?FixingSMIIssues#Alternate_Approach_smictrl](http://wiki.linuxcnc.org/cgi-bin/wiki.pl?FixingSMIIssues#Alternate_Approach_smictrl)

# References
[https://www.osadl.org/latest-stable-quick-rt-preempt-kerne.realtime-kernel-installation.0.html](https://www.osadl.org/latest-stable-quick-rt-preempt-kerne.realtime-kernel-installation.0.html)
[https://www.osadl.org/Realtime-Preempt-Kernel.kernel-rt.0.html](https://www.osadl.org/Realtime-Preempt-Kernel.kernel-rt.0.html)
[https://github.com/LinuxCNC/linuxcnc](https://github.com/LinuxCNC/linuxcnc)
[http://linuxcnc.org/docs/devel/html/code/building-linuxcnc.html](http://linuxcnc.org/docs/devel/html/code/building-linuxcnc.html)
[https://forum.linuxcnc.org/38-general-linuxcnc-questions/33492-unexpected-realtime-delay-on-task-0](https://forum.linuxcnc.org/38-general-linuxcnc-questions/33492-unexpected-realtime-delay-on-task-0)