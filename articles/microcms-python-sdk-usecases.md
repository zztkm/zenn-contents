---
title: "microCMS Python SDK ユースケースアイデア"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["microcms", "python"]
published: true
---

この記事は [microCMSでこんなことができた！あなたのユースケースを大募集 by microCMS - Qiita Advent Calendar 2024](https://qiita.com/advent-calendar/2024/microcms) シリーズ 1の12月12日の記事です。

前回の記事は、[@14kw](https://qiita.com/14kw)さんの [microCMSのコンテンツをSpreadsheetから一括更新する](https://14code.com/blog/20241210_0700/) でした。

実際に活用している現場での一括編集、一覧性の課題について技術で解決されている記事でしたし、feedback について触れられていたので、せっかくだから microCMS のコミュニティ Discord で議論してみても良いのではないかと思いました。

タイトルにあるとおり、今回はユースケースについて記事を書いてみようと思いますが、どのようなケースで使えるのかなとアイデアを出してみるのが記事の本題であり、実際に運用している話ではないのをご了承ください。

## microCMS Python SDK

https://github.com/zztkm/microcms-python-sdk

microCMS からもいくつか SDK が出ていますが、Python 向けの SDK はなかったので、今回実装を始めました（以前 V 言語向けの https://github.com/zztkm/microcms-v-sdk を実装した経験もあったので)。
microCMS はみなさんご存知の通り、ウェブサイトなどで利用されるのが主要なユースケースであり、それ以外の部分のユースケースはあまり表に出てきていないのかなーと思いました（個人の観測範囲内での感想なので、こういうところで使われているとかあればぜひ教えてください）。

自分はウェブサイトの制作をしたり、コンテンツ管理をしていたりするわけではないのですが、興味本位でmicroCMS を触り始めたのでせっかくなら microCMS 利用の幅が広がったらおもしろいだろうと思い、Python SDK の実装を始めた感じです。

## Python SDK ユースケース

さっそくですが microCMS Python SDK の利用シーンを雑に考えていこうと思います。

### 一括データ更新ツール

Python はかんたんなスクリプトなどでよく使われますが（当社比）、これは Python SDK を使うときのわかりやすい例だと思います。
例えば、microCMS の管理画面からコンテンツを更新するのではなく、一度ローカルに書き出して編集を行い、更新データを microCMS に適用したいケースで Python SDK が活躍するのではないかと思います。

大規模になると難しいかもしれませんが、ある程度の規模までなら Python は良いデータ処理ツールになるのではないかと思います。

### シナリオを JSON で管理するテストツール

僕は microCMS を管理画面つきの便利な JSON 管理ソリューションだと捉えているフシがありまして、JSON を扱うプログラムに対してなら microCMS はいろんな場面で使えるのではないかと思いました。

個人的に Python をテスト自動化のツールとしても活用しているのですが、例えば以下のようなことに扱えないかなと考えました。

- テストシナリオは JSON で変更可能
- テストシナリオのパラメータ調整は非エンジニアの方が担当 (microCMS の管理画面を活用)
- エンジニアは Python SDK を使ってテストツールと microCMS を接続する処理を実装
- テストツールは非エンジニアでも実行できる仕組みになっていて好きなタイミングで実行できる

まあ microCMS じゃなくても良いかもしれませんが、幅を広げるためのアイデアだと思って書いてみました。

### (やろうと思えば) 静的サイトジェネレーター

ウェブサイト制作では JS / TS のライブラリを活用して開発を行うのが主流なのかなと想像しますが（WordPress は除く)、静的サイトジェネレーターという選択肢もあります。
特に動きが必要ないウェブサイトであれば microCMS 前提の静的サイトジェネレーターとして実装すれば十分に実用的なものができるのではないかなと思います。
HTML はテンプレートエンジンを使えば生成できますし、css も自分で書くことができるなら問題ありません。ただ、ウェブ開発において JS / TS の資産を活用できないのは辛い部分ではあるなと思います...

## おわりに

以上雑にユースケースを想像してみましたが、使っていくなかでもっと広がっていくんではないかなと考えています。

本当は SDK も完成させて、ユースケースに沿ったツールも公開しておいてみたいなことをやるのが理想だったのですが全然間に合いませんでした。
しかし Python は [uv](https://docs.astral.sh/uv/) などのツールチェインの成長によって本当に使いやすくなったので活用シーンを広げていきやすいのではないかと思います。SDK を開発しようとするときはそのサービスに詳しくないといけませんから、せっかくなので microCMS についてもどんどん詳しくなれたらなと思っています。

明日の記事は [@trickstar13](https://qiita.com/trickstar13) さんの「管理画面の構築不要！Studioや静的サイトをmicroCMSで拡張して『本日の営業時間』を表示してみよう」です。楽しみです。

