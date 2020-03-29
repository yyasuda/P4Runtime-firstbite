# P4Runtime-firstbite
Simple P4Runtime tutorial for starters.

## はじめに

このリポジトリにあるコードやデータは、P4Runtime を使った P4 Switch の制御を初めて試す人達に、簡単な入り口となるチュートリアルとして作成されました。P4および P4Runtime については、ある程度の理解があることを仮定しています。実際に手元で試すのが初めて、という人にとって良い入り口となりますように。

## This tutorial does…

このチュートリアルでは、以下の三つのことを試します。

- Packet-Out 処理
- Packet-In 処理
- テーブルへのエントリ追加

これらの実験は、以下の環境で行います。

- コントローラ役には P4Runtime Shell を用いる
- スイッチ役には P4Runtime に対応した Mininet を用いる
- P4 コンパイルにはオープンソースの p4c を用いる

これら全て Docker 環境で動作するもので揃えています。最初はこのドキュメントに記述にあるものをそのまま使って下さい。

## Tools

このチュートリアルではすべて Docker 環境で実験を行います。

#### P4C (modified)

Docker Hub: [yutakayasuda/p4c_python3](https://hub.docker.com/repository/docker/yutakayasuda/p4c_python3)

オリジナルの [p4lang/p4c](https://hub.docker.com/r/p4lang/p4c) はどう言うわけか python3 が正しく動きませんでした。何か間違えてるんでしょうか。。

#### P4Runtime-enabled Mininet Docker Image

Docker Hub: [opennetworking/p4mn](https://hub.docker.com/r/opennetworking/p4mn)
GitHub: [opennetworkinglab/p4mn-docker](https://github.com/opennetworkinglab/p4mn-docker)

#### P4Runtime Shell (modified)

Docker Hub: [yutakayasuda/p4runtime-shell-dev](https://hub.docker.com/repository/docker/yutakayasuda/p4runtime-shell-dev)
GitHub: [yyasuda/p4runtime-shell](https://github.com/yyasuda/p4runtime-shell)

オリジナルの [P4Runtime Shell](https://hub.docker.com/r/p4lang/p4runtime-sh) にはPacket I/O のための機能が実装されていなかったのでそれを追加し、Dockerfile.dev でビルドしたものです。開発時の便利のためにless, vimを加えています。

## Step by Step

さあ準備が整いました。以下に一つずつ手順を示します。順番に試していくのが良いでしょう。

### Tutorial 0: [実験環境の準備](./t0_prepare.md)

実験に先だって、P4 スイッチプログラムのコンパイルが必要です。次にMininetを起動し、そこにコントローラ代わりとなる、P4 Runtime Shell を接続させます。

### Tutorial 1: [Packet-Out 処理](./t1_packet-out.md)

コントローラ役の P4Runtime Shell から Mininet 環境のスイッチに向けて Packet-Out メッセージを送ります。Out先として指定したポートに接続されたホストにパケットが届いていることを確認します。

### Tutorial 2: [Packet-In 処理](./t2_packet-in.md)

用意した P4 プログラムは宛先として未知のMACアドレスをもつパケットが来たら、それをPacket-In としてコントローラに送るように作られています。ここではスイッチに接続されたホストからpingパケットを送出すると、それがPacket-Inとなって P4Runtime Shell に受信されることを確認します。

### Tutorial 3: [テーブルへのエントリ追加](./t3_add-entry.md)

Packet I/O処理は、P4Runtime のStreamMessageRequest / StreamMessageResponse メッセージを用いて行われます。P4Runtimeには他にWriteRequestメッセージがあり、これを用いてテーブルなどのP4Runtime Entityの内容を更新することができます。ここではテーブルにMACアドレスを登録し、それに従ってスイッチに接続されたホストから送出されたpingパケットが、指定したポートに出力されることを確認します。

### Tutorial 4: [パケットの往復](./t4_roundtrip.md)

Tutorial 3 に加えて、もう一つエントリをテーブルに追加し、二つのホストの間でping パケットが正しく往復することを確認します。

## Next Step

このチュートリアルでは内部構造に触れることなく、分かりやすい入り口を示すことに集中しました。次はそこを入り口に、中を掘っていくことになるかと思います。これまでに私が読んで、特に有益だったドキュメントについて、幾つか挙げておきます。

- [P4Runtime Specification](https://p4.org/specs/) v1.1.0 [[HTML](https://p4.org/p4runtime/spec/v1.1.0/P4Runtime-Spec.html)] [[PDF](https://p4.org/p4runtime/spec/v1.1.0/P4Runtime-Spec.pdf)]
  （6.4.6. ControllerPacketMetadata の例、16.1. Packet I/O あたり）
- P4Runtime proto p4/v1/[p4runtime.proto](https://github.com/p4lang/p4runtime/blob/master/proto/p4/v1/p4runtime.proto) 
  （WriteReuqest / StreamMessageRequest / StreamMessageResponse あたりに注目）
- P4Runtime proto p4/config/v1/[p4info.proto](https://github.com/p4lang/p4runtime/blob/master/proto/p4/config/v1/p4info.proto) 
  （特に ControllerPacketMetadata ）
- Protocol Buffer に関するもの

- [P4<sub>16</sub> Portable Switch Architecture (PSA)](https://p4.org/specs/) v1.1 [[HTML](https://p4.org/p4-spec/docs/PSA-v1.1.0.html)] [[PDF](https://p4.org/p4-spec/docs/PSA-v1.1.0.pdf)]
  上記P4Runtime Specification では、1.2 In Scope をはじめ、P4RuntimeはPSAをある程度仮定している記述が散見されます。今回のチュートリアルには特に関係ありませんでしたが、気になる記述があれば読むと良いかと。

## 謝辞

このチュートリアルは、私が（特にWedge Switchにおける）Packet-In処理を試したいと思ったとき、簡単に試すためのドキュメントを見つけることができなかった事が動機となっています。現在の理解に至るまで、リバース・エンジニアリングのように大量の時間を使って仕様書やコードを読み、散在する情報を掻き集めてたくさんの試行錯誤をしました。

その作業の多くを手伝ってくれた石原さん、西さん、小谷さんに感謝。特にこのチュートリアルで利用している ether_switch.p4 は、はじめ[小谷さんが作成したもの](https://gist.github.com/daisuke-k/1714c176e62280cc8627dc5e96846e56)に機能追加したものです。

この資料がP4Runtimeに取り組もうとする初心者にとって、何かしらのお役に立ちますように。