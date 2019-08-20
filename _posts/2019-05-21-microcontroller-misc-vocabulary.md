---
layout: post
title: "Miscellaneous embedded software vocabulary and notions"
date: 2019-05-21 12:00:00 +0200
categories: software microcontroller 
---
Here are some concepts or tools that I had to learn about when working with microcontrollers.
Some of the explanations might be specific to Nordic's nRF family and NXP's Kinetis family of
microcontrollers since these are what our products are using.

## Bit-banding
Mapping a complete word of memory inside an alias region onto a single bit in a bit-band region. This
enables individual bits to be toggled atomically using a single LDR instruction
without performing a read-modify-write sequence.

See [arm's documentation](http://infocenter.arm.com/help/topic/com.arm.doc.100166_0001_00_en/ric1417773736773.html) about bit-banding on the Cortex-M series.

## SVDConv.exe
A data format and associated tool to describe memory mapped registers of peripherals in Arm
Cortex-M based microcontrollers. The tool allows silicon vendors to generate header files
to be used by software developers.

See [CMSIS documentation](http://www.keil.com/pack/doc/CMSIS/SVD/html/index.html).

## NAND Flash vs NOR Flash
At the core of flash memory are floating gate transistors, i.e. the gate is electrically
isolated with a control gate deposited above the floating gate. The control gate is only
capacitively coupled with the floating gate. Since the the floating gate is surrounded
by highly resistive material, the charge contained in it remains unchanged for long
periods of time without a power supply. The amount of charge stored can be modified using [Fowler-Nordheim
tunneling](https://en.wikipedia.org/wiki/Field_electron_emission#Fowler%E2%80%93Nordheim_tunneling) or [hot-carrier injection](https://en.wikipedia.org/wiki/Hot-carrier_injection).

When the floating gate is charged, this charge screens the electric field from the control gate, increasing the threshold voltage of the cell (V<sub>T1</sub>). This means that a higher voltage (V<sub>T1</sub>) needs to be applied to the control gate to make the channel conduct. The presence of a logical 0 or 1 is sensed by determining whether there is current flowing through the transistor (between the source and the drain) when a voltage V s.t. V<sub>T1</sub> < V < V<sub>T2</sub> is applied on the control gate. 

|Feature                      |NOR         |NAND        |
|-----------------------------|------------|------------|
|Random access                |Yes         |No          |
|Cell size                    |Larger      |Smaller     |
|Write speed                  |Slower      |Faster      |
|Random Read speed            |Faster      |Slower      |
|Erase speed                  |Much Slower |Faster      |
|Storage capacity             |Lower       |Higher      |
|Error Correcting Code        |Not needed  |Yes         |
|Endurance (erases per block) |100,000s    |10,000s     |

See [Floating-gate MOSFET](https://en.wikipedia.org/wiki/Floating-gate_MOSFET) on Wikipedia.
See [Flash 101](https://www.embedded.com/design/prototyping-and-development/4460910/Flash-101--NAND-Flash-vs-NOR-Flash) on www.embedded.com.