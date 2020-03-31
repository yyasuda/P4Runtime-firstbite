# P4Runtime-firstbite
Simple P4Runtime tutorial for starters

See [Japanese document](README_ja.md).

## Introduction

The code and data in this repository were created as an easy starting point for first-timers using P4 Runtime to control a P4 Switch. It is assumed that you have some knowledge of P4 and P4 Runtime. I hope it will be a good entry point for people who are trying it out for the first time.

## This tutorial does…

In this tutorial, you will try three things:.

- Packet-Out Processing
- Packet-In Processing
- Adding Entries to the Table

These experiments are performed in the following environments:

- Use P4 Runtime Shell as controller
- Use Mininet corresponding to P4 Runtime as switch
- Use open source p4c for P4 compilation

We have everything that runs in a Docker environment. At the first try, just use what is described in this document.

## Tools

In this tutorial, all experiments are performed in a Docker environment.

#### P4C (modified)

Docker Hub: [yutakayasuda/p4c_python3](https://hub.docker.com/repository/docker/yutakayasuda/p4c_python3)

The original [p4lang/p4c](https://hub.docker.com/r/p4lang/p4c) somehow python3 did not work properly. Am I doing something wrong??

#### P4Runtime-enabled Mininet Docker Image

Docker Hub: [opennetworking/p4mn](https://hub.docker.com/r/opennetworking/p4mn)
GitHub: [opennetworkinglab/p4mn-docker](https://github.com/opennetworkinglab/p4mn-docker)

#### P4Runtime Shell (modified)

Docker Hub: [yutakayasuda/p4runtime-shell-dev](https://hub.docker.com/repository/docker/yutakayasuda/p4runtime-shell-dev)
GitHub: [yyasuda/p4runtime-shell](https://github.com/yyasuda/p4runtime-shell)

The Original [P4Runtime Shell](https://hub.docker.com/r/p4lang/p4runtime-sh) did not include an implementation for Packet I/O, so added it and built it with Dockerfile.dev. And less and vim are added for convenience during development.

## Step by Step

All set. Here are the steps. I recommend that you try them in order.

### Tutorial 0: [Preparing the Environment](./t0_prepare.md)

The P4 switch program must be compiled before the experiment. Then start Mininet and start the P4 Runtime Shell, which is the controller replacement. Then connect them.

### Tutorial 1: [Packet-Out Processing](./t1_packet-out.md)

The P4 Runtime Shell acting as a controller sends a Packet-Out message to the switch in the Mininet environment. Verify that the packet arriving at the host connected to the port you specify as the Out destination.

### Tutorial 2: [Packet-In Processing](./t2_packet-in.md)

When a packet arrives with an unknown MAC address as the destination, the prepared P4 program will send it to the controller as a Packet-In. Here we see that if a host attached to the switch sends a ping packet, it becomes a Packet-In and is received by the P4 Runtime Shell.

### Tutorial 3: [Adding Entries to the Table](./t3_add-entry.md)

Packet I/O processing is performed using StreamMessageRequest / StreamMessageResponse messages of P4Runtime. P4 Runtime also has a WriteRequest message that can be used to update the contents of the P4 Runtime Entity, such as tables. Here, you register the MAC address in a table. And verify that a ping packet from the host attached to the switch outputs to the specified port accordingly.

### Tutorial 4: [Packet Round Trip](./t4_roundtrip.md)

After Tutorial 3, add another entry to the table to verify that the ping packets are correctly going back and forth between the two hosts.

## Next Step

This tutorial did not go into depth into the internal structure of the system, but instead focused on providing a straightforward entry point. I think you will dig in there next. Here are some of the most useful documents I've read so far.

- [P4Runtime Specification](https://p4.org/specs/) v1.1.0 [[HTML](https://p4.org/p4runtime/spec/v1.1.0/P4Runtime-Spec.html)] [[PDF](https://p4.org/p4runtime/spec/v1.1.0/P4Runtime-Spec.pdf)]
  (See; 6.4.6. ControllerPacketMetadata, 16.1. Packet I/O)
- P4Runtime proto p4/v1/[p4runtime.proto](https://github.com/p4lang/p4runtime/blob/master/proto/p4/v1/p4runtime.proto) 
  (See around WriteReuqest / StreamMessageRequest / StreamMessageResponse)
- P4Runtime proto p4/config/v1/[p4info.proto](https://github.com/p4lang/p4runtime/blob/master/proto/p4/config/v1/p4info.proto) 
  (Pay attention to ControllerPacketMetadata)
- Some documents about Protocol Buffer

- [P4<sub>16</sub> Portable Switch Architecture (PSA)](https://p4.org/specs/) v1.1 [[HTML](https://p4.org/p4-spec/docs/PSA-v1.1.0.html)] [[PDF](https://p4.org/p4-spec/docs/PSA-v1.1.0.pdf)]
  In the above P4 Runtime Specification, there are some statements that P4 Runtime assumes PSA to some extent, such as section 1.2 In Scope. I didn't have anything to do with PSA specific things in this tutorial, but it's worth reading if you're interested.

### CPU port changes

CPU port information is at the beginning of ether_switch.p4. If necessary, update it and recompile.

```C++
#define CPU_PORT 255
```

If you don't know the CPU port number of the switch you are testing, [P4Runtime-CPUport-finder](https://github.com/yyasuda/P4Runtime-CPUport-finder) may help.

