## Tutorial 2: Packet-In 処理

用意した P4 プログラムは宛先として未知のMACアドレスをもつパケットが来たら、それをPacket-In としてコントローラに送るように作られています。ここではスイッチに接続されたホストからpingパケットを送出すると、それがPacket-Inとなって P4Runtime Shell に受信されることを確認します。

### 手順

以下のプロセスで Packet-In が行われていることを確認します。
1. P4Runtime Shell 側で Watch() 関数を起動する
2. Mininet 側で ping パケットを送出する
3. Watch() 関数がそれを受け取って画面に表示する

なお、Packet In されたデータはしばらくバッファに残っているので、処理 1, 2 の順を逆転して 2, 1 の順で作業しても問題なく Packet-In を目撃することができます。

### Watch() 関数の実行

P4Runtime Shell プロンプトで、Watch() 関数を実行します。
```bash
P4Runtime sh >>> Watch()
```
### ping パケットの送出

Mininet プロンプトで、ホスト h1 から h2 にめがけて ping パケットを一つだけ送出します。
```bash
mininet> h1 ping -c 1 h2
PING 10.0.0.1 (10.0.0.1) 56(84) bytes of data.
```
この状態では ping の返信が来ないので、ping コマンドはタイムアウトまで終了せずこの状態で止まります。

###  Watch() 側に受け取ったパケットが表示される

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
metadata_id: 1 は packet_in の ingress_port に当たり、スイッチの port 1 から入ってきたことが示されています。ここで h2 ping -c 1 h1 として実験すれば、port 2 から届いたことが確認できるはずです。

なお Watch() 関数は 3 秒でタイムアウトします。その時は以下のように表示されてWatch()関数が終了します。
```bash
P4Runtime sh >>> Watch()
None returned
P4Runtime sh >>>            
```



## Next Step

#### Tutorial 3: [テーブルへのエントリ追加](t3_add-entry.md)

