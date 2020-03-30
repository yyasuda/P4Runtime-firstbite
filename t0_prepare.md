## Tutorial 0: Preparing the Environment

The P4 switch program must be compiled before the experiment. Then start Mininet and start the P4 Runtime Shell, which is the controller replacement. Then connect them.

### Compiling ether_switch.p4 

#### Create working directory and copy a file

Create a/tmp/ether _ switch directory for your work and copy the P4 program (ether_switch.p4) from this tutorial.

```bash
$ mkdir /tmp/ether_switch
$ cp ether_switch.p4 /tmp/ether_switch
$ ls /tmp/ether_switch
ether_switch.p4
$
```

#### Compiling by P4C

Launch the P4C Docker container.

```bash
$ docker run -it -v /tmp/ether_switch/:/tmp/ yutakayasuda/p4c_python3 /bin/bash
root@f53fc79201b8:/p4c# cd /tmp      
root@f53fc79201b8:/tmp# ls
ether_switch.p4
root@f53fc79201b8:/tmp# 
```

Note that we are synchronizing the/tmp/ether_switch directory on the host with the/tmp directory on the docker.

Now compile ether_switch.p4, which should be visible to the p4c container under /tmp.

```bash
root@f53fc79201b8:/tmp# p4c --target bmv2 --arch v1model --p4runtime-files p4info.txt ether_switch.p4 
root@f53fc79201b8:/tmp# ls
ether_switch.json  ether_switch.p4  ether_switch.p4i  p4info.txt
root@f53fc79201b8:/tmp# 
```

You will later launch the P4 Runtime Shell using the p4info.txt and ether_switch.json you generated.

### Launching the Mininet Environment

Start a Mininet environment that supports the P4 Runtime, again in a Docker environment. Note that at boot time, the --arp and --mac options allow you to do things like ping tests without ARP processing.
```bash
$ docker run --privileged --rm -it -p 50001:50001 opennetworking/p4mn --arp --topo single,2 --mac
*** Error setting resource limits. Mininet's performance may be affected.
*** Creating network
*** Adding controller
*** Adding hosts:
h1 h2 
*** Adding switches:
s1 
*** Adding links:
(h1, s1) (h2, s1) 
*** Configuring hosts
h1 h2 
*** Starting controller

*** Starting 1 switches
s1 .⚡️ simple_switch_grpc @ 50001

*** Starting CLI:
mininet>
```
You can see that port1 on s1 is connected to h1 and port2 is connected to h2.
```bash
mininet> net
h1 h1-eth0:s1-eth1
h2 h2-eth0:s1-eth2
s1 lo:  s1-eth1:h1-eth0 s1-eth2:h2-eth0
mininet> 
```
You can see;
h1-eth0 MAC address is 00:00:00:00:00:01 and
h2-eth0 MAC address is 00:00:00:00:00:02 by ifconfing command.
```bash
mininet> h1 ifconfig h1-eth0
h1-eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.1  netmask 255.0.0.0  broadcast 10.255.255.255
        inet6 fe80::200:ff:fe00:1  prefixlen 64  scopeid 0x20<link>
        ether 00:00:00:00:00:01  txqueuelen 1000  (Ethernet) 
(snip...)
```

## Connecting P4 Runtime Shell to Mininet

### Launching P4Runtime Shell

```bash
Cecil(133)% docker run -it -v /tmp/ether_switch/:/tmp/ yutakayasuda/p4runtime-shell-dev /bin/bash
root@d633c64bbb3c:/p4runtime-sh# . activate 
(venv) root@d633c64bbb3c:/p4runtime-sh# 

(venv) root@d633c64bbb3c:/p4runtime-sh# cd /tmp
(venv) root@d633c64bbb3c:/tmp# ls
ether_switch.json  ether_switch.p4  ether_switch.p4i  p4info.txt
(venv) root@d633c64bbb3c:/tmp# 
```
Note again that we are synchronizing the/tmp/ether_switch directory on the host with /tmp on the docker. Also, don't forget that the above `. activate` process is important for later operations.

### Connecting to Mininet 

Connect to Mininet as follows. Adjust the IP address to your environment.
```bash
(venv) root@d633c64bbb3c:/tmp# /p4runtime-sh/p4runtime-sh --grpc-addr 192.168.XX.XX:50001 --device-id 1 --election-id 0,1 --config p4info.txt,ether_switch.json
*** Welcome to the IPython shell for P4Runtime ***
P4Runtime sh >>>
```
Let's display the table list to confirm that it works.
```bash
P4Runtime sh >>> tables 
MyIngress.ether_addr_table

P4Runtime sh >>> 
```

Below is a list of the fields and actions in this table. Note the id information displayed here. This information appears when you operate the P4 Runtime message.

```bash
P4Runtime sh >>> tables["MyIngress.ether_addr_table"] 
Out[2]: 
preamble {
  id: 33592100
  name: "MyIngress.ether_addr_table"
  alias: "ether_addr_table"
}
match_fields {
  id: 1
  name: "hdr.ethernet.dstAddr"
  bitwidth: 48
  match_type: EXACT
}
action_refs {
  id: 16838673 ("MyIngress.forward")
}
action_refs {
  id: 16803363 ("MyIngress.to_controller")
}
size: 1024

P4Runtime sh >>>  
```
#### Connection may be broken

If you leave it for a while, the following message may appear.
```bash
P4Runtime sh >>> CRITICAL:root:StreamChannel error, closing stream
CRITICAL:root:P4Runtime RPC error (UNAVAILABLE): Socket closed
```
If this message appears, you must exit the P4 Runtime Shell and reconnect to Mininet before doing any action on the switch, such as Packet I/O or adding entries to the table.

### Viewing information related to Packet-Out/In

The ether_switch.p4 program used in this experiment contains the header description for Packet-I/O and the handling of it. Here is a minimum description of the meaning of the id value that appears in the messages that appear in the following experiments, while showing the header description below.
```C++
@controller_header("packet_out")
header packet_out_header_t {
    bit<9> egress_port;
    bit<7> _pad;
}

@controller_header("packet_in")
header packet_in_header_t {
    bit<9> ingress_port;
    bit<7> _pad;
}
```
The P4 Runtime typically represents a port number in 9 bits, followed by 7 bits of padding, all of which consume 2 bytes of metadata in Packet I/O processing. Compiling this header description generates the [ControllerPacketMetadata](https://p4.org/p4runtime/spec/master/P4Runtime-Spec.html#sec-controller-packet-meta) information as a P4info Object. You can find this information in the p4info.txt file, but also in the P4 Runtime Shell can show them like below.

```bash
P4Runtime sh >>> p4info.controller_packet_metadata[0]
Out[46]: 
preamble {
  id: 67121543
  name: "packet_out"
  alias: "packet_out"
  annotations: "@controller_header(\"packet_out\")"
}
metadata {
  id: 1
  name: "egress_port"
  bitwidth: 9
}
metadata {
  id: 2
  name: "_pad"
  bitwidth: 7
}

P4Runtime sh >>> p4info.controller_packet_metadata[1]                                                                                             
Out[47]: 
preamble {
  id: 67146229
  name: "packet_in"
  alias: "packet_in"
  annotations: "@controller_header(\"packet_in\")"
}
metadata {
  id: 1
  name: "ingress_port"
  bitwidth: 9
}
metadata {
  id: 2
  name: "_pad"
  bitwidth: 7
}

P4Runtime sh >>>   
```
Note the metadata id information displayed here. This information appears when you operate the P4 Runtime message later.

Now, as you can guess. If the P4 Runtime needs to point to the switch's P4 Entity in some way, use this id. For example, the table MyIngress.ether_addr_table is specified as id: 33592100, and the output port for Packet-Out processing is specified as id: 1 (eventually converted to metadata _ id:) of metadata.


## Next Step

#### Tutorial 1: [Packet-Out Processing](t1_packet-out.md)

