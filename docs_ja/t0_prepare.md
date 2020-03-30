## Tutorial 0: 実験環境の準備

実験に先だって、P4 スイッチプログラムのコンパイルが必要です。それを使ってMininetを起動し、そこにコントローラ代わりとなる、P4 Runtime Shell を接続させます。

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

### Mininet 環境の立ち上げ

P4Runtimeに対応した Mininet 環境を、やはりDocker環境で起動します。起動時に --arp と --mac オプションを指定して、ARP 処理無しに ping テストなどができるようにしてあることに注意してください。
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

### P4Runtime Shell と Mininet の接続

#### P4Runtime Shellの起動

```bash
Cecil(133)% docker run -it -v /tmp/ether_switch/:/tmp/ yutakayasuda/p4runtime-shell-dev /bin/bash
root@d633c64bbb3c:/p4runtime-sh# . activate 
(venv) root@d633c64bbb3c:/p4runtime-sh# 

(venv) root@d633c64bbb3c:/p4runtime-sh# cd /tmp
(venv) root@d633c64bbb3c:/tmp# ls
ether_switch.json  ether_switch.p4  ether_switch.p4i  p4info.txt
(venv) root@d633c64bbb3c:/tmp# 
```
ここでもホストの /tmp/ether_switch ディレクトリと docker の /tmp を同期させていることに注意して下さい。それから、上の `. activate` 処理はすぐこの後の操作で重要なので忘れないように。

#### Mininet への接続

以下のようにして Mininet に接続します。IPアドレスは自身の環境に合わせて下さい。

```bash
(venv) root@d633c64bbb3c:/tmp# /p4runtime-sh/p4runtime-sh --grpc-addr 192.168.XX.XX:50001 --device-id 1 --election-id 0,1 --config p4info.txt,ether_switch.json
*** Welcome to the IPython shell for P4Runtime ***
P4Runtime sh >>>
```
動作確認としてテーブル一覧を表示させてみます。
```bash
P4Runtime sh >>> tables 
MyIngress.ether_addr_table

P4Runtime sh >>> 
```

以下にこのテーブルのフィールドやアクションの一覧を表示させてみます。ここで表示される id 情報に注意しておいてください。あとでP4Runtime メッセージを操作するときに出てきます。

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

このメッセージが表示された場合は、一度 P4Runtime Shell を終了して再度 Mininet に接続し直さないと、スイッチに対して働きかける処理、たとえばPacket I/Oやテーブルへのエントリ追加操作が効きません。

### Packet-Out/In に関係する情報を見る

今回実験に使用する ether_switch.p4 プログラムは Packet-Out, Packet-In のためのヘッダ記述と、それを扱う処理を含んでいます。以下にヘッダ記述を示しながら、これ以降の実験に登場するメッセージの中で登場する id の値の意味について最低限の説明をします。
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
P4Runtime では標準的にポート番号を9bit で表現しています。それに続いて7bit のパディングを定義して、全体では2バイトのメタデータをPacket I/O 処理では利用します。
このヘッダ記述をコンパイルすると、P4info Objectとして[ControllerPacketMetadata](https://p4.org/p4runtime/spec/master/P4Runtime-Spec.html#sec-controller-packet-meta) 情報が出力されます。その情報はp4info.txt ファイルを見れば確認できますが、例えばP4Runtime Shell でも以下のようにして得られます。

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
ここで表示されるmetadataの id 情報に注意しておいてください。あとでP4Runtime メッセージを操作するときに出てきます。

もう想像が付くでしょう。P4Runtime で何かしらスイッチのP4 Entityを指し示す事が必要な場合、このid を使います。例えばテーブル MyIngress.ether_addr_table は id: 33592100 として、例えば Packet-Out 処理を行う時の出力先ポートは、metadata の id: 1 （最終的には metadata_id: に変換される）として指定します。




## Next Step

#### Tutorial 1: [Packet-Out 処理](t1_packet-out.md)

