---
title: "Simple"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---


**More details of the gem5 Ruby Interconnection Network are
[here](Interconnection_Network "wikilink").**

## Simple Network

The default network model in Ruby is the simple network.

  - **Related Files**:
      - **src/mem/ruby/network/Network.py**
      - **src/mem/ruby/network/simple**
      - **src/mem/ruby/network/simple/SimpleNetwork.py**

## Configuration

Simple network uses the generic network parameters in Network.py:

  -   - **number_of_virtual_networks**: This is the maximum number of
        virtual networks. The actual number of active virtual networks
        is determined by the protocol.
      - **control_msg_size**: The size of control messages in bytes.
        Default is 8. **m_data_msg_size** in Network.cc is set to the
        block size in bytes + control_msg_size.

Additional parameters are specified in simple/SimpleNetwork.py:

  -   - **buffer_size**: Size of buffers at each switch input and
        output ports. A value of 0 implies infinite buffering.
      - **endpoint_bandwidth**: Bandwidth at the end points of the
        network in 1000th of byte.
      - **adaptive_routing**: This enables adaptive routing based on
        occupancy of output buffers.

## Switch Model

The simple network models hop-by-hop network traversal, but abstracts
out detailed modeling within the switches. The switches are modeled in
simple/PerfectSwitch.cc while the links are modeled in
simple/Throttle.cc. The flow-control is implemented by monitoring the
available buffers and available bandwidth in output links before
sending.

![Simple_network.jpg](Simple_network.jpg "Simple_network.jpg")
