---
title: "NIC devices"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---


The gem5 simulator has two different Network Interface Cards (NICs)
devices that can be used to connect together two simulation instances
over a simulated ethernet link.

## Getting an list of packets on the ethernet link

You can get a list of the packet on the ethernet link by creating a
Etherdump object, setting it's file parameter, and setting the dump
parameter on the EtherLink to it. This is easily accomplished with our
fs.py example configuration by adding the command line option
--etherdump=<filename>. The resulting file will be named <file> and be
in a standard pcap format. This file can be read with
[wireshark](http://www.wireshark.org) or anything else that understands
the pcap format.

