## Tutorial 1: Packet-Out 処理

コントローラ役の P4Runtime Shell から Mininet 環境のスイッチに向けて Packet-Out メッセージを送ります。Out先として指定したポートに接続されたホストにパケットが届いていることを確認します。

### ether_switch.p4 のコンパイル

#### 作業場所の作成とファイルのコピー

作業用に /tmp/ether_switch ディレクトリを作り、この Tutorial にある P4 プログラム（ether_switch.p4）をコピーします。

```bash
$ mkdir /tmp/ether_switch
$ cp ether_switch.p4 /tmp/ether_switch
$ ls /tmp/ether_switch
ether_switch.p4
$
```

#### P4Cによるコンパイル

以下のようにしてP4C Dockerコンテナを起動します。

```bash
$ docker run -it -v /tmp/ether_switch/:/tmp/ yutakayasuda/p4c_python3 /bin/bash
root@f53fc79201b8:/p4c# cd /tmp      
root@f53fc79201b8:/tmp# ls
ether_switch.p4
root@f53fc79201b8:/tmp# 
```

ホストの /tmp/ether_switch ディレクトリと docker の /tmp を同期させていることに注意してください。

そこでp4cコンテナから見て /tmp以下に見えているはずの ether_switch.p4 をコンパイルします。

```bash
root@f53fc79201b8:/tmp# p4c --target bmv2 --arch v1model --p4runtime-files p4info.txt ether_switch.p4 
root@f53fc79201b8:/tmp# ls
ether_switch.json  ether_switch.p4  ether_switch.p4i  p4info.txt
root@f53fc79201b8:/tmp# 
```

ここで生成した p4info.txt と ether_switch.json を使って、あとで P4Runtime Shell を起動することになります。

## Mininet 環境の立ち上げ

P4Runtimeに対応した Mininet 環境を、やはりDocker環境で起動します。起動時に --arp と --mac オプションを指定して、ARP 処理無しに ping テストなどができるようにしてあることに注意してください。
```bash
$ docker run --privileged --rm -it -p 50001-50003:50001-50003 opennetworking/p4mn --arp --topo single,2 --mac
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
s1 の port 1 が h1 に、port2 が h2 に接続されていることが確認できます。
```bash
mininet> net
h1 h1-eth0:s1-eth1
h2 h2-eth0:s1-eth2
s1 lo:  s1-eth1:h1-eth0 s1-eth2:h2-eth0
mininet> 
```
h1 がスイッチにつながれているインタフェイス h1-eth0 の MAC アドレスは  00:00:00:00:00:01
h2 がスイッチにつながれているインタフェイス h2-eth0 の MAC アドレスは  00:00:00:00:00:02
であることが ifconfig コマンドで確認できるでしょう。

```bash
mininet> h1 ifconfig h1-eth0
h1-eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.1  netmask 255.0.0.0  broadcast 10.255.255.255
        inet6 fe80::200:ff:fe00:1  prefixlen 64  scopeid 0x20<link>
        ether 00:00:00:00:00:01  txqueuelen 1000  (Ethernet) 
(snip...)
```

## P4Runtime Shell と Mininet の接続

### P4Runtime Shellの起動

```bash
Cecil(133)% docker run -it -v /tmp/ether_switch/:/tmp/ yutakayasuda/p4runtime-shell-dev /bin/bash
root@d633c64bbb3c:/p4runtime-sh# . activate 
(venv) root@d633c64bbb3c:/p4runtime-sh# 

(venv) root@d633c64bbb3c:/p4runtime-sh# cd /tmp
(venv) root@d633c64bbb3c:/tmp# ls
ether_switch.json  ether_switch.p4  ether_switch.p4i  p4info.txt
(venv) root@d633c64bbb3c:/tmp# 
```
ここでもホストの /tmp/ether_switch ディレクトリと docker の /tmp を同期させていることに注意して下さい。それから、上の「. activate」処理はすぐこの後の操作で重要なので忘れないように。

### Mininet への接続

以下のようにして Mininet に接続します。IPアドレスは自身の環境に合わせて下さい。

```bash
(venv) root@d633c64bbb3c:/tmp# /p4runtime-sh/p4runtime-sh --grpc-addr 192.168.XX.XX:50001 --device-id 1 --election-id 0,1 --config p4info.txt,ether_switch.json
*** Welcome to the IPython shell for P4Runtime ***
P4Runtime sh >>>   

（動作確認としてテーブル一覧を表示させてみます）
P4Runtime sh >>> tables 
MyIngress.ether_addr_table

P4Runtime sh >>> 
```
以下にこのテーブルのフィールドやアクションの一覧を表示させてみます。ここで表示される id 情報に注意して置いてください。あとでメッセージを作成するときに使います。

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

#### コネクションが切れる場合があります

しばらく放置しておくと、以下のようなメッセージが出ることがあります。

```bash
P4Runtime sh >>> CRITICAL:root:StreamChannel error, closing stream
CRITICAL:root:P4Runtime RPC error (UNAVAILABLE): Socket closed
```

このメッセージが表示された場合は、一度 P4Runtime Shell を終了して再度 Mininet に接続し直さないと、コマンド操作などが効きません。

## Packet out 処理によるパケットの送出

### 準備

作業用に作った /tmp/ether_switch ディレクトリに、この Tutorial にある packetout.txt をコピーします。

```bash
Cecil(284)% cat /tmp/ether_switch/packetout.txt
packet {
  payload: "\377\377\377\377\377\377\377\377\377\377\377\377\000\000ABCDEFGHIJKLMN"
  metadata {
    metadata_id: 1
    value: "\000\001"
  }
}
Cecil(285)% 
```
P4Runtime におけるPacket-Out 処理は、StreamMessageRequest というメッセージで、packet を指定してやることで実現されます。packetout.txt ファイルはこのStreamMessageRequest メッセージの内容そのものに当たります。
- payload はデータリンク層のパケットがまるまる格納されています。8 進表記 "\377\377" は16進表記での "\xff" ですから、このペイロードは、宛先・送り元 MAC アドレスがともに ff:ff:ff:ff:ff:ff で、Protocol Type が 00 であり、そのあとにABCDE… と幾らかデータが続くことを意味します。
- metadata_id 1 の metadata は Packet Out 先のポート指定です。 value: "\000\001" はつまり、転送先が port 1 であることを意味します。

### Packet Out 操作

P4Runtime Shell 側で Request() 関数を起動します。Request() 関数は私が標準の P4Runtime Shell に追加した機能です。指定されたファイルからメッセージ内容を読み取り、これをP4RuntimeのStreamMessageRequest メッセージとしてスイッチに送り込みます。
特に戻り値やメッセージは返ってきません。

```bash
P4Runtime sh >>> Request("/tmp/packetout.txt")                                                                                             
packet {
  payload: "\377\377\377\377\377\377\377\377\377\377\377\377\000\000ABCDEFGHIJKLMN"
  metadata {
    metadata_id: 1
    value: "\000\001"
  }
}

P4Runtime sh >>> 
```

#### パケット検出

上の操作によって、送り込んだパケットがスイッチの port 1 から送信されていることを、Mininet の h1 の eth0 ポート（h1-eth0）をモニタリングして確認します。tcpdump には -XX を付けるとヘッダごと hexdump してくれて便利ですね。

```bash
mininet> h1 tcpdump -XX -i h1-eth0
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on h1-eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
```
ここで待ち状態になるでしょうから、もう一度、上記の Packet Out 操作をしてください。すると以下のような表示が出るでしょう。
```bash
14:38:51.525754 Broadcast STP > Broadcast Unknown DSAP 0x40 Unnumbered, disc, Flags [Command], length 14
	0x0000:  ffff ffff ffff ffff ffff ffff 0000 4142  ..............AB
	0x0010:  4344 4546 4748 494a 4b4c 4d4e            CDEFGHIJKLMN
```



これで Packet-Out 処理の確認ができました。



## Next Step

##### Tutorial 2: [Packet-In 処理](t2_packet-in.md)

