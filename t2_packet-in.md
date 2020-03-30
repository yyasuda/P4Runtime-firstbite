## Tutorial 2: Packet-In Processing

When a packet arrives with an unknown MAC address as the destination, the prepared P4 program will send it to the controller as a Packet-In. Here we see that if a host attached to the switch sends a ping packet, it becomes a Packet-In and is received by the P4 Runtime Shell.

### Steps

Verify that Packet-In is performed in the following process.
1. Execute Watch() function on the P4 Runtime Shell side
2. Send a ping packets on Mininet side
3. Watch() function receives it and displays it on the screen

Note that the Packet-In data remains in the buffer for a while, so it is no problem to see the Packet-In by reversing the order of processing 1 and 2 and working in the order of 2 and 1.

### Excecuting Watch() function 

At the P4 Runtime Shell prompt, run the Watch () function.
```bash
P4Runtime sh >>> Watch()
```
### Sending a ping packet

At the Mininet prompt, send a single ping packet from host h1 to h2.
```bash
mininet> h1 ping -c 1 h2
PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data.
```
In this state, no ping replies are received, so the ping command does not finish until it times out and stops.

### Watch() shows received packets

```bash
P4Runtime sh >>> Watch()
Response message is:
packet {
  payload: "\000\000\000\000\000\002\000\000\000\000\000\001\010\000E\000\000T\023\223@\000@\001\023\024\n\000\000\001\n\000\000\002\010\000\337\220\000n\000\001(d\177^\000\000\000\000\253j\006\000\000\000\000\000\020\021\022\023\024\025\026\027\030\031\032\033\034\035\036\037 !\"#$%&\'()*+,-./01234567"
  metadata {
    metadata_id: 1
    value: "\000\001" 
  }
  metadata {
    metadata_id: 2
    value: "\000"
  }
}
```
The metadata_id: 1 corresponds to the ingress_port of packet_in. It means that the packet came in from port 1 of the switch. Now experiment with `h2 ping -c 1 h1` and you will see it coming from port 2.
Note that the Watch() function times out after 3 seconds. At that time, the following is displayed and the Watch() function ends.
```bash
P4Runtime sh >>> Watch()
None returned
P4Runtime sh >>>            
```



## Next Step

#### Tutorial 3: [Adding Entries to the Table](t3_add-entry.md)

