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

## LinuxCNC configuration
Download support software for the Mesa board (e.g. [here](http://www.mesanet.com/software/parallel/7i80.zip) for the 7i76e), and copy the hostmot2 drivers in `/lib/firmware/hm2`.

Launch `pncconf` and follow the instructions to configure your board, everything is quite self explanatory (don't forget
to power the Mesa board when trying to test your setup).

## MPG Wheel
You can have up to 2 MPGs connected by enabling the smart serial port:
```
sserial_port_0=2xxxx
```
Then wire the MPG A & B inputs to the 7i76e 16 and 17 inputs (18 and 19 for the second MPG). Wire 5V and GND from a stepper
output for instance.

Next add the following to `custom.hal` in your configuration folder:
```
setp axis.x.jog-vel-mode 0
setp axis.x.jog-enable TRUE
setp axis.x.jog-scale 1
net x-jog-counter <= hm2_7i76e.0.7i76.0.0.enc0.count
net x-jog-counter => axis.x.jog-counts
```

## References

1. [https://forum.linuxcnc.org/38-general-linuxcnc-questions/33969-7i76e-quick-setup](https://forum.linuxcnc.org/38-general-linuxcnc-questions/33969-7i76e-quick-setup)
1. [https://forum.linuxcnc.org/media/kunena/attachments/3278/7i76_Anschluss.pdf](https://forum.linuxcnc.org/media/kunena/attachments/3278/7i76_Anschluss.pdf)
1. [http://www.cnctablebuild.com/mesa-setup-part-2-software/](http://www.cnctablebuild.com/mesa-setup-part-2-software/)