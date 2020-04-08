## Tutorial 1: Packet-Out Processing

The P4 Runtime Shell acting as a controller sends a Packet-Out message to the switch in the Mininet environment. Verify that the packet arriving at the host connected to the port you specify as the Out destination.

### Copy a file

Copy the packet-out message file (packetout.txt) from this tutorial to the /tmp/ether_switch directory you have created for your work. Let's take a look inside the file.

```bash
$ cp 1to2.txt /tmp/ether_switch
$ cat /tmp/ether_switch/packetout.txt
packet {
  payload: "\377\377\377\377\377\377\377\377\377\377\377\377\000\0001234567890123456789012345678901234567890123456789012345678901234567890123456789"
  metadata {
    metadata_id: 1
    value: "\000\001"
  }
}
$
```
Packet-Out processing in the P4 Runtime is performed by specifying a 'packet' in a StreamMessageRequest message. The file packetout.txt is the exact content of this StreamMessageRequest message.
- 'payload' contains the entire data-link layer packet. Since octal " \377 \377" is " \xff" in hexadecimal notation, this payload means that the destination and source MAC addresses are both ff:ff:ff:ff:ff:ff, Protocol Type is 00, and some data follows it as ABCDE ....
- 'metadata' with metadata_id1 is the port specification for Packet Out destination. value: " \000 \001" means the destination is port 1.

### Packet-Out operation

Do the Request() function on the P4 Runtime Shell side. The Request() function is what I added to the standard P4 Runtime Shell. It reads the message content from the specified file then sends it to the switch as a P4 Runtime StreamMessageRequest message.
No return value or message is returned. The contents of the message to be sent are printed on the screen.

```bash
P4Runtime sh >>> Request("/tmp/packetout.txt")                                                                                             
packet {
  payload: "\377\377\377\377\377\377\377\377\377\377\377\377\000\0001234567890123456789012345678901234567890123456789012345678901234567890123456789"
  metadata {
    metadata_id: 1
    value: "\000\001"
  }
}

P4Runtime sh >>> 
```

#### Packet detection

Monitor the eth0 port on h1 (h1-eth0) on Mininet to verify that the incoming packets are from the port 1 of the switch. It is convenient to add -XX to tcpdump to show the hex dump of the whole header.

```bash
mininet> h1 tcpdump -XX -i h1-eth0
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on h1-eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
```
Since it will be in the waiting state, perform the above Packet Out operation again. You will see the following display.
```bash
14:38:51.525754 Broadcast STP > Broadcast Unknown DSAP 0x40 Unnumbered, disc, Flags [Command], length 79
	0x0000:  ffff ffff ffff ffff ffff ffff 0000 3132  ..............12
	0x0010:  3334 3536 3738 3930 3132 3334 3536 3738  3456789012345678
	0x0020:  3930 3132 3334 3536 3738 3930 3132 3334  9012345678901234
	0x0030:  3536 3738 3930 3132 3334 3536 3738 3930  5678901234567890
	0x0040:  3132 3334 3536 3738 3930 3132 3334 3536  1234567890123456
	0x0050:  3738 3930 3132 3334 3536 3738 39         7890123456789
```



You have now confirmed the Packet-Out processing.



## Next Step

#### Tutorial 2: [Packet-In Processing](t2_packet-in.md)

