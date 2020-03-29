## Tutorial 1: Packet-Out 処理

コントローラ役の P4Runtime Shell から Mininet 環境のスイッチに向けて Packet-Out メッセージを送ります。Out先として指定したポートに接続されたホストにパケットが届いていることを確認します。

### ファイルのコピー

作業用に作った /tmp/ether_switch ディレクトリに、この Tutorial にあるパケットアウトのためのメッセージファイル（packetout.txt）をコピーします。少しファイルの中を見てみましょう。

```bash
$ cp 1to2.txt /tmp/ether_switch
$ cat /tmp/ether_switch/packetout.txt
packet {
  payload: "\377\377\377\377\377\377\377\377\377\377\377\377\000\000ABCDEFGHIJKLMN"
  metadata {
    metadata_id: 1
    value: "\000\001"
  }
}
$
```
P4Runtime におけるPacket-Out 処理は、StreamMessageRequest というメッセージで、packet を指定してやることで実現されます。packetout.txt ファイルはこのStreamMessageRequest メッセージの内容そのものに当たります。
- payload はデータリンク層のパケットがまるまる格納されています。8 進表記 "\377\377" は16進表記での "\xff" ですから、このペイロードは、宛先・送り元 MAC アドレスがともに ff:ff:ff:ff:ff:ff で、Protocol Type が 00 であり、そのあとにABCDE… と幾らかデータが続くことを意味します。
- metadata_id 1 の metadata は Packet Out 先のポート指定です。 value: "\000\001" はつまり、転送先が port 1 であることを意味します。

### Packet Out 操作

P4Runtime Shell 側で Request() 関数を起動します。Request() 関数は私が標準の P4Runtime Shell に追加した機能です。指定されたファイルからメッセージ内容を読み取り、これをP4RuntimeのStreamMessageRequest メッセージとしてスイッチに送り込みます。
特に戻り値やメッセージは返ってきません。コマンド実行後に、画面上には送信するメッセージの内容をprintしています。

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

#### Tutorial 2: [Packet-In 処理](t2_packet-in.md)

