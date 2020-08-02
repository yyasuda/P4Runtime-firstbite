## Tutorial 4: Packet Round Trip

After Tutorial 3, add another entry to the table to verify that the ping packets are correctly going back and forth between the two hosts, as follows;

<img src="t4_table.png" alt="attach:(table entry)" title="Table Entry" width="350">

###  Copy a file

The '1to2.txt' file we used in our add-entry experiment was a message to send a packet to port2 destined for h2 (MAC address 00:00:00:00:00:02). There is a message on '2to1.txt' file to make this return. Copy it to the /tmp/ether_switch directory you have created for your work.

The contents of the file are as follows.

```bash
$ cat /tmp/ether_switch/return.txt 
updates {
  type: INSERT
  entity {
    table_entry {
      table_id: 33592100
      match {
        field_id: 1
        exact {
          value: "\000\000\000\000\000\001" # Octal expression
        }
      }
      action {
        action {
          action_id: 16838673
          params {
            param_id: 1 
            value: "\x00\x01" # Hexadecimal expression
          }
        }
      }
    }
  }
}
$
```

Now you will see that it specifies that packets destined for the MAC address 00:00:00:00:00:01 should be sent to port1.

### Entry adding operation

As in Tutorial 3, on the P4 Runtime Shell side, the Write() function sends a WriteRequest message to the switch. As a result, you will see that there are two entries registered in total, for the round trip.

```bash
P4Runtime sh >>> Write("/tmp/2to1.txt")

P4Runtime sh >>> table_entry["MyIngress.ether_addr_table"].read(lambda a: print(a))   
```
### Ping round trip check
When you send a ping request with the round-trip entry set up, you can confirm that the ping response returns correctly. (In Tutorial 3, even if a ping packet was sent from h1 and reached h2, the ping command was waiting because the reply packet did not reach h1.)
```bash
mininet> h1 ping -c 1 h2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.959 ms

--- 10.0.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.959/0.959/0.959/0.000 ms
mininet> 
```
You can see that the Packet-In is not happening by the Watch() function times out without receiving any packet.
```bash
P4Runtime sh >>> Watch()

None returned

P4Runtime sh >>>
```



You have completed the series of tutorials. Congratulations!

## Next Step

Is [this](README.md#next-step) good for the next? 
