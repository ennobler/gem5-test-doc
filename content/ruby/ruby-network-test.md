---
title: "Ruby Network Tester"
date: 2018-05-13T18:51:37-04:00
draft: false
weight: 1000
---

The Ruby Network Tester provides a framework for simulating the
interconnection network with controlled inputs. This is useful for
network testing/debugging, or for network-only simulations with
synthetic traffic (especially with the garnet network). The tester works
in conjunction with the [Network_test](Network_test "wikilink")
coherence protocol.

### Related Files

  - **configs/example/ruby_network_test.py**: file to invoke the
    network tester
  - **src/cpu/testers/networktest**: files implementing the tester.
      - **NetworkTest.py**
      - **networktest.hh**
      - **networktest.cc**

### How to run

First build gem5 with the [Network_test](Network_test "wikilink")
coherence protocol:

    scons build/ALPHA_Network_test/gem5.debug

Sample command to
    run:

    ./build/ALPHA_Network_test/gem5.debug configs/example/ruby_network_test.py  \
      --num-cpus=16 --num-dirs=16 --topology=Mesh --mesh-rows=4  \
     --sim-cycles=1000 \
     --injectionrate=0.01 \
     --synthetic=0 \
     --fixed-pkts \
     --maxpackets=1 \
     --garnet-network=fixed

| Parameter            | Description                                                                                                                                                                                                                                                                                                                                                                      |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **--num-cpus**       | Number of cpus. This is the number of source (injection) nodes in the network.                                                                                                                                                                                                                                                                                                   |
| **--num-dirs**       | Number of directories. This is the number of destination (ejection) nodes in the network.                                                                                                                                                                                                                                                                                        |
| **--topology**       | Topology for connecting the cpus and dirs to the network routers/switches. The *Mesh* topology creates a network with *--num-cpus* number of routers/switches. It requires equal number of cpus and dirs, and connects one cpu, and one dir each to one network router. More detail about different topologies can be found [here](Interconnection_Network#Topology "wikilink"). |
| **--mesh-rows**      | The number of rows in the mesh. Only valid when *--topology* is *Mesh* or *MeshDirCorners*.                                                                                                                                                                                                                                                                                      |
| **--sim-cycles**     | Total number of cycles for which the simulation should run.                                                                                                                                                                                                                                                                                                                      |
| **--injectionrate**  | Traffic Injection Rate in packets/node/cycle. It can take any decimal value between 0 and 1. The number of digits of precision after the decimal point can be controlled by *--precision* which is set to 3 as default in *ruby_network_test.py*.                                                                                                                              |
| **--synthetic**      | The type of synthetic traffic to be injected. Currently, 3 types of synthetic traffic: uniform-random (*--synthetic=0*), tornado (*--synthetic=1*), and bit-complement (*--synthetic=2*) can be injected by the tester code in *networktest.cc*.                                                                                                                                 |
| **--maxpackets**     | Maximum number of packets to be injected by each cpu node. This parameter only works when *--fixed-pkts* is declared. This is useful for network debugging.                                                                                                                                                                                                                      |
| **--fixed-pkts**     | Inject only a fixed number of packets (specified by *--maxpackets*) from each cpu. This is useful for network debugging. If this parameter is not declared, cpus will keep injecting at the specified rate till *--sim-cycles* cycles when the simulation ends.                                                                                                                  |
| **--garnet-network** | Enables the garnet network. This parameter can take two values: *fixed* and *flexible*. More details about these two garnet networks can be found [here](Interconnection_Network#Garnet_Networks "wikilink").                                                                                                                                                                    |

### Implementation of network tester

The network tester is implemented in networktest.cc. The sequence of
steps involved in generating and sending a packet are as follows.

  - Every cycle, each cpu performs a bernouli trial with probability
    equal to *--injectionrate* to determine whether to generate a packet
    or not. If *--fixed-pkts* is enabled, each cpu stops generating new
    packets after generating *--maxpackets* number of packets. The
    tester terminates after *--sim-cycles*.
  - If the cpu has to generate a new packet, it computes the destination
    for the new packet based on the synthetic traffic type
    (*--synthetic*).
      - 0 (Uniform Random): send to a destination directory chosen at
        random from the available directories.
      - 1 (Tornado): send to the destination directory half-way across X
        dimension. This only makes sense for Mesh or Torus topologies.
      - 2 (Bit-Complement): send to the destination directory whose
        location is the bit-complement of the source cpu location. This
        only makes sense for Mesh or Torus topologies.
  - This destination is embedded into the bits after block offset in the
    packet address.
  - The generated packet is randomly tagged as a ReadReq, or an
    INST_FETCH, or a WriteReq, and sent to the Ruby Port
    (src/mem/ruby/system/RubyPort.hh/cc).
  - The Ruby Port converts the packet into a RubyRequestType:LD,
    RubyRequestType:IFETCH, and RubyRequestType:ST, respectively, and
    sends it to the Sequencer, which in turn sends it to the
    [Network_test](Network_test "wikilink") cache controller.
  - The cache controller extracts the destination directory from the
    packet address.
  - The cache controller injects the LD, IFETCH and ST into virtual
    networks 0, 1 and 2 respectively.
      - LD and IFETCH are injected as control packets (8 bytes), while
        ST is injected as a data packet (72 bytes).
  - The packet traverses the network and reaches the directory.
  - The directory controller simply drops it.

