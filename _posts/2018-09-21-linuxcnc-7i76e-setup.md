---
layout: post
title: "How to Set Up a Mesa 7i76e Board with LinuxCNC"
date: 2018-09-21 12:00:00 +0200
categories: linuxcnc mesa
---
After reading this post you should be able to control a stepper motor with the LinuxCNC user
interface.

Required hardware and software:
* Mesa 7i76e board (see [http://store.mesanet.com/index.php?route=product/product&product_id=290](http://store.mesanet.com/index.php?route=product/product&product_id=290))
* LinuxCNC installed on a computer running a real time kernel (check the latency!)
* Stepper motor
* Stepper motor driver
* Power supply

## Wiring
Following [this wiring guide](https://forum.linuxcnc.org/media/kunena/attachments/3278/7i76_Anschluss.pdf):
1. Wire power (e.g. 24V) to TB1 (field power) and TB3 (unregulated logic power)
2. Wire the stepper driver to TB2 _Stepper 0_ inputs
3. Wire power (e.g. 24V) to the stepper driver
4. Wire the motor to its driver

## Ethernet configuration
Pick a static IP address (), by adding the following to `/etc/networking/interfaces`:
```
auto YOUR_INTERFACE_NAME
iface YOUR_INTERFACE_NAME inet static
address 192.168.1.122
netmask 255.255.255.0
```

Run the following command (ping before and after to see the difference), which sets the number of microseconds to delay
an RX interrupt to 0:
```
sudo ethtool -C YOUR_INTERFACE_NAME rx-usecs 0
```

If you use a Wifi connection on the same computer, make sure your address spaces do not clash. You can change the jumpers on the
7i76e so that it has its IP address set to 10.10.10.10 (W2 down, W3 up).

## LinuxCNC configuration
Download support software for the Mesa board (e.g. [here](http://www.mesanet.com/software/parallel/7i80.zip) for the 7i76e), and copy the hostmot2 drivers in `/lib/firmware/hm2`.

Launch `pncconf` and follow the instructions to configure your board, everything is quite self explanatory (don't forget
to power the Mesa board when trying to test your setup).

## MPG Wheel
You can have up to 2 MPGs connected to pins 16/17 and 18/19 by enabling software mode 2 for the 7i76e in your CONFIG_NAME.hal file
(see [https://forum.linuxcnc.org/media/kunena/attachments/3278/7i76_Anschluss.pdf](https://forum.linuxcnc.org/media/kunena/attachments/3278/7i76_Anschluss.pdf)):
```
sserial_port_0=2xxxxxxx
```
Then wire the MPG A & B inputs to inputs 16 and 17 on the 7i76e (18 and 19 for the second MPG). Wire 5V and GND from a stepper
output for instance.

Next add the following to `custom.hal` in your configuration folder:
```
setp joint.0.jog-vel-mode 0
setp joint.0.jog-enable TRUE
setp joint.0.jog-scale 1
net x-jog-counter <= hm2_7i76e.0.7i76.0.0.enc0.count
net x-jog-counter => joint.0.jog-counts
```

## References

1. [https://forum.linuxcnc.org/38-general-linuxcnc-questions/33969-7i76e-quick-setup](https://forum.linuxcnc.org/38-general-linuxcnc-questions/33969-7i76e-quick-setup)
1. [https://forum.linuxcnc.org/media/kunena/attachments/3278/7i76_Anschluss.pdf](https://forum.linuxcnc.org/media/kunena/attachments/3278/7i76_Anschluss.pdf)
1. [http://www.cnctablebuild.com/mesa-setup-part-2-software/](http://www.cnctablebuild.com/mesa-setup-part-2-software/)