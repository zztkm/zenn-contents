---
title: "tcconfig コマンド逆引き"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["tcconfig", "network"]
published: true
---

この記事は [zztkm振り返りカレンダー Advent Calendar 2024](https://adventar.org/calendars/10960) 7 日目の記事です。

今回は tcconfig コマンドの逆引きというタイトルで、tcconfig のコマンドをユースケースごとに雑に紹介していこうと思います。
tcconfig は不安定なネットワーク環境を再現したい場合によく利用されているツールです、うまく使えばネットワーク起因の不具合の検知などに役立つはずです。
注意事項としては、これは逆引きでこういうケースで使えるよというのを示しているだけなので、詳細は下にリンクを貼っている tcconfig のドキュメントを参照してください。

https://tcconfig.readthedocs.io/en/latest/

## tcconfig インストール

tcconfig は pip を使ってインストールすることができます

```bash
sudo pip install tcconfig
```

tcconfig をインストールすると以下の3つのコマンドがインストールされます。tcconfig というツール名でインストールされるわけではないので注意してください（僕は最初にあれ？となりました...）。

- tcset
  - トラフィック制御ルールを設定する
- tcdel
  - トラフィック制御ルールを削除する
- tcshow
  - 現在のトラフィック制御設定を表示する

ではさっそく次からユースケースごとに書いてみます

## 入ってくるパケットに対して遅延あり、パケロスありの環境にする

`--direction` に incoming を指定するのが大事です。

tcset はデフォルトで outgoing 向きにトラフィック制御ルールを追加するので、入ってくるパケットに対してルールを設定したい場合は incoming と明示的に指定する必要があります。

例: 150ms の遅延、遅延時間の分布は +- 30ms (つまり遅延は 120 - 180 の間になる)、5% のパケロス率
```bash
tcset <device> --direction incoming --delay 150ms -delay-distro 30ms --loss 5%
```

出ていく方向に設定する場合は direction 設定は必要なしです（しても良い）。

## 帯域幅を制限する

`--rate` を使います

例
```bash
tcset <device> --rate 20Mbps
```

## パケット並び替えを設定する

パケットが並び替えられるように設定したい場合は `--reordering` [%] を使います。

例: 遅延は 200ms あるし、パケットの並び替え率も 10 % とひどいネットワーク状態
```bash
tcset <device> --delay 200ms --reordering 10%
```

## おわりに

また色々試したら追加していきます。

