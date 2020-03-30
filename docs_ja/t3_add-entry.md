## Tutorial 3: テーブルへのエントリ追加

Packet I/O処理は、P4Runtime のStreamMessageRequest / StreamMessageResponse メッセージを用いて行われます。P4Runtimeには他にWriteRequestメッセージがあり、これを用いてテーブルなどのP4Runtime Entityの内容を更新することができます。ここではテーブルにMACアドレスを登録し、それに従ってスイッチに接続されたホストから送出されたpingパケットが、指定したポートに出力されることを確認します。

### ファイルのコピー

作業用に作った /tmp/ether_switch ディレクトリに、この Tutorial にあるエントリ追加のためのメッセージファイル（1to2.txt）をコピーします。このメッセージは h2 の MAC アドレス 00:00:00:00:00:02 向けのパケットを port 2 に送るものです。少しファイルの中を見てみましょう。

```bash
$ cp 1to2.txt /tmp/ether_switch
$ cat /tmp/ether_switch/1to2.txt 
updates {
  type: INSERT
  entity {
    table_entry {
      table_id: 33592100
      match {
        field_id: 1
        exact {
          value: "\000\000\000\000\000\002" # Octal expression 
        }
      }
      action {
        action {
          action_id: 16838673
          params {
            param_id: 1 
            value: "\x00\x02" # Hexadecimal expression 
          }
        }
      }
    }
  }
}
$
```

テーブル id: 33592100 は MyIngress.ether_addr_table を、そこの match フィールドの id:1 は hdr.ethernet.dstAddr を意味します。続く value は 8 進数で 6 バイトぶん記述されており、つまり MAC アドレス 00:00:00:00:00:02 を意味します。

この辺りの情報は P4Runtime Shell を用いて確認することができます。

```bash
P4Runtime sh >>> tables["MyIngress.ether_addr_table"] 
Out[2]: 
preamble {
  id: 33592100
  name: "MyIngress.ether_addr_table"
(snip...)
```

同様に action_id: 16838673 は MyIngress.forward を指し、続く param_id: 1 の value は 16 進表記で 2 バイトぶん使って、出力先ポートが 2 であることを示しています。この時少し混乱するのは、P4 プログラムにおける packet_out_header_t の egress_port の定義は 9bit 幅であり、param_id: 1 で指定する value の 2 バイトの、下位9bitが使われる、ということです。

```C++
@controller_header("packet_out")
header packet_out_header_t {
    bit<9> egress_port;
    bit<7> _pad;
}
```

（私は最初ここが上位ビットから9bit ぶん取り出されると思い込んでしまい、'\x01\x00' と書いて期待通り動かず悩みました。）

### エントリ追加操作

P4Runtime Shell 側で Write() 関数を起動します。Write()関数はもともとP4Runtime Shell に用意されていたものです。指定されたファイルからメッセージ内容を読み取り、これをP4RuntimeのWriteRequest メッセージとしてスイッチに送り込みます。
特に戻り値やメッセージは返ってきません。

```bash
P4Runtime sh >>> Write("/tmp/1to2.txt")

P4Runtime sh >>> 
```
追加されていることをテーブルの中身を表示させて確認します。
```bash
P4Runtime sh >>> table_entry["MyIngress.ether_addr_table"].read(lambda a: print(a))
table_id: 33592100 ("MyIngress.ether_addr_table")
match {
  field_id: 1 ("hdr.ethernet.dstAddr")
  exact {
    value: "\\x00\\x00\\x00\\x00\\x00\\x02"
  }
}
action {
  action {
    action_id: 16838673 ("MyIngress.forward")
    params {
      param_id: 1 ("port")
      value: "\\x00\\x02"
    }
  }
}

P4Runtime sh >>>       
```

いまはエントリに追加されることを確認する方法だけを示しておきますが、以下のようにして実際に転送されていることを確認することもできます。

- Mininet 上で `h1 ping -c 1 h2` として何かしらパケットを送り出す（Tutorial 2 参照）
- h2 上での受信を `h2 tcpdump -XX -i h2-eth0` として確認する（Tutorial 1 参照）

### 参考：エントリの削除

以下のようにして登録されているエントリをすべて表示することができていました。
```bash
P4Runtime sh >>> table_entry["MyIngress.ether_addr_table"].read(lambda a: print(a))
```
同様に以下のようにして登録したエントリをすべて削除することができます。
```bash
P4Runtime sh >>> table_entry["MyIngress.ether_addr_table"].read(lambda a: a.delete())
```



## Next Step

#### Tutorial 4: [パケットの往復](t4_roundtrip.md)

