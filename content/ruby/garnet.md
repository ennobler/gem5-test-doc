---
title: "Garnet"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

**More details of the gem5 Ruby Interconnection Network are
[here](Interconnection_Network "wikilink").**

### Garnet Network Model

'''Note: This model of garnet is no longer supported in gem5. The
updated model is [garnet2.0](garnet2.0 "wikilink").

Garnet is a detailed interconnection network model inside gem5. The
details can be found in the [ISPASS 2009
Paper](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=4919636%7CGarnet).

If your use of Garnet contributes to a published paper, please cite the
following paper:

    @inproceedings{garnet,
      title={GARNET: A detailed on-chip network model inside a full-system simulator},
      author={Agarwal, Niket and Krishna, Tushar and Peh, Li-Shiuan and Jha, Niraj K},
      booktitle={Performance Analysis of Systems and Software, 2009. ISPASS 2009. IEEE International Symposium on},
      pages={33--42},
      year={2009},
      organization={IEEE}
    }

Garnet consists of 2 pipeline models: a detailed *fixed-pipeline* model,
and an approximate *flexible-pipeline* model.

The *fixed-pipeline* model is intended for low-level interconnection
network evaluations and models the detailed micro-architectural features
of a 5-stage Virtual Channel router with credit-based flow-control.
Researchers interested in investigating different network
microarchitectures can readily modify the modeled microarchitecture and
pipeline. Also, for system level evaluations that are not concerned with
the detailed network characteristics, this model provides an accurate
network model and should be used as the default model.

The *flexible-pipeline* model is intended to provide a reasonable
abstraction of all interconnection network models, while allowing the
router pipeline depth to be flexibly adjusted. A router pipeline might
range from a single cycle to several cycles. For evaluations that wish
to easily change the router pipeline depth, the *flexible-pipeline*
model provides a neat abstraction that can be used.

  - **Related
        Files**:
      - **src/mem/ruby/network/Network.py**
      - **src/mem/ruby/network/garnet/BaseGarnetNetwork.py**
      - **src/mem/ruby/network/garnet/fixed-pipeline**
      - **src/mem/ruby/network/garnet/GarnetNetwork_d.py**
      - **src/mem/ruby/network/garnet/flexible-pipeline**
      - **src/mem/ruby/network/garnet/flexible-pipeline/GarnetNetwork.py**

## Invocation

The garnet networks can be enabled by adding **--garnet-network=fixed**,
or **--garnet-network=flexible** on the command line, respectively.

## Configuration

Garnet uses the generic network parameters in Network.py:

  -   - **number_of_virtual_networks**: This is the maximum number of
        virtual networks. The actual number of active virtual networks
        is determined by the protocol.
      - **control_msg_size**: The size of control messages in bytes.
        Default is 8. **m_data_msg_size** in Network.cc is set to the
        block size in bytes + control_msg_size.

Additional parameters are specified in garnet/BaseGarnetNetwork.py:

  -   - **ni_flit_size**: flit size in bytes. Flits are the
        granularity at which information is sent from one router to the
        other. Default is 16 (=\> 128 bits). \[This default value of 16
        results in control messages fitting within 1 flit, and data
        messages fitting within 5 flits\]. Garnet requires the
        ni_flit_size to be the same as the bandwidth_factor (in
        network/BasicLink.py) as it does not model variable bandwidth
        within the network.
      - **vcs_per_vnet**: number of virtual channels (VC) per virtual
        network. Default is 4.

The following parameters in garnet/fixed-pipeline/GarnetNetwork_d.py
are only valid for fixed-pipeline:

  -   - **buffers_per_data_vc**: number of flit-buffers per VC in the
        data message class. Since data messages occupy 5 flits, this
        value can lie between 1-5. Default is 4.
      - **buffers_per_ctrl_vc**: number of flit-buffers per VC in the
        control message class. Since control messages occupy 1 flit, and
        a VC can only hold one message at a time, this value has to be
        1. Default is 1.

The following parameters in garnet/flexible-pipeline/GarnetNetwork.py
are only valid for flexible-pipeline:

  -   - **buffer_size**: Size of buffers per VC. A value of 0 implies
        infinite buffering.
      - **number_of_pipe_stages**: number of pipeline stages in each
        router in the flexible-pipeline model. Default is 4.

<!-- end list -->

  - **Additional features**
      - **Routing**: Currently, garnet only models deterministic routing
        using the routing tables described
        [earlier](Interconnection_Network#Routing "wikilink").
      - **Modeling variable link bandwidth**: The bandwidth_factor
        specifies the link bandwidth as the number of bytes per cycle
        per network link. ni_flit_size has to be same as this value.
        Links which have low bandwidth can be modeled by specifying a
        longer latency across them in the topology file (as explained
        [earlier](Interconnection_Network#Topology "wikilink")).
      - **Multicast messages**: The network modeled does not have
        hardware multi-cast support within the network. A multi-cast
        message gets broken into multiple uni-cast messages at the
        interface to the network.

## Garnet fixed-pipeline network

The garnet fixed-pipeline models a classic 5-stage Virtual Channel
router. The 5-stages are:

1.  **Buffer Write (BW) + Route Compute (RC)**: The incoming flit gets
    buffered and computes its output port.
2.  **VC Allocation (VA)**: All buffered flits allocate for VCs at the
    next routers. \[The allocation occurs in a *separable* manner:
    First, each input VC chooses one output VC, choosing input arbiters,
    and places a request for it. Then, each output VC breaks conflicts
    via output arbiters\]. All arbiters in ordered virtual networks are
    *queueing* to maintain point-to-point ordering. All other arbiters
    are *round-robin*.
3.  **Switch Allocation (SA)**: All buffered flits try to reserve the
    switch ports for the next cycle. \[The allocation occurs in a
    *separable* manner: First, each input chooses one input VC, using
    input arbiters, which places a switch request. Then, each output
    port breaks conflicts via output arbiters\]. All arbiters in ordered
    virtual networks are *queueing* to maintain point-to-point ordering.
    All other arbiters are *round-robin*.
4.  **Switch Traversal (ST)**: Flits that won SA traverse the crossbar
    switch.
5.  **Link Traversal (LT)**: Flits from the crossbar traverse links to
    reach the next routers.

The flow-control implemented is credit-based.

![Garnet_router.jpg](Garnet_router.jpg "Garnet_router.jpg")

## Garnet flexible-pipeline network

The garnet flexible-pipeline model should be used when one desires a
router pipeline different than 5 stages (the 5 stages include the link
traversal stage). All the components of a router (buffers, VC and switch
allocators, switch etc) are modeled similar to the fixed-pipeline
design, but the pipeline depth is not modeled, and comes as an input
parameter **number_of_pipe_stages**. The flow-control is implemented
by monitoring the availability of buffers at each output port before
sending.
