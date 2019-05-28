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