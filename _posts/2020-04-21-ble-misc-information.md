---
layout: post
title: "Miscellaneous BLE vocabulary and notions"
date: 2020-04-21 12:00:00 +0200
categories: software microcontroller bluetooth ble
---
Here are some concepts, resources or tools that I had to learn about when working with Bluetooth Low Energy (BLE).

# BLE Stack Overall Architecture
The BLE protocol stack can be split in 3 main layers:
1. **Host**: this layer sits below the application and enables it to communicate with other BLE devices in a standard and interoperatable way through high level APIs.
1. **Controller**: this layer implements the Link Layer (LL) low-level, real-time protocol which describes standard interoperatable over the air communication. It's responsible for scheduling packet reception and transmission, guaranteed delivery of data and control procedures.
1. **Radio**: the hardware which implements the required baseband blocks allowing the LL software to send and receive in the 2.4GHz band of the spectrum.

# Host Controller Interface
The Bluetooth Specification describes the format in which a Host must communicate with a Controller: the Host Controller Interface (HCI). This protocol can be implemented over several transports, e.g. UART, SPI or USB. This standardization makes it possible to combine Host and Controllers from different vendors.

# Generic Access Profile
The Generic Access Profile (GAP) defines 4 distinct roles of BLE usage:
1. Connection-less roles
    * Broadcaster
    * Observer
2. Connection-oriented roles
    * Peripheral
	* Central

# GAP Modes

# ATT

# GATT

# References

1. [https://docs.zephyrproject.org/latest/guides/bluetooth/bluetooth-arch.html](https://docs.zephyrproject.org/latest/guides/bluetooth/bluetooth-arch.html)