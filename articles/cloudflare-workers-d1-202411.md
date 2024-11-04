---
title: "Cloudflare Workers D1 with Go スタック | 202411 版"
emoji: "🟢"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["go", "cloudflareworkers", "sqlc"]
published: true
---

Cloudflare Workers D1 アプリケーションを Go で書くときのお気に入りスタックを紹介します。

この記事は自分のブログに書いた [Cloudflare Workers D1 with Go](https://blog.tsurutatakumi.info/posts/cloudflare-workers-go-d1-202404) をリライトしたもので、内容はほぼ同じです（今見ても自分には良いスタックに思えたので）。

## TL;DR

- [syumai/workers](https://github.com/syumai/workers) は最高
  - D1 を sql.Open で扱うことができて嬉しい
- sql.Open で D1 を扱えるので、[sqlc](https://github.com/sqlc-dev/sqlc) をそのまま使える
- HTTP Router には [michi](https://github.com/go-michi/michi) がちょうど良い

## モチベーション

DB を使ったアプリケーションであれば自分は sqlc を使いたいというモチベーションがありました。

下の記事はちょうど2年くらい前の記事ですが、Go で Workers アプリケーションを書くことができるライブラリを syumai さんが作っているのを知っていたので Go で Workers を書きたいというモチベーションは強くありましたが、Hello World するくらいしかやってませんでした。

[Cloudflare Workersで簡単にGoのHTTPサーバーを動かすためのライブラリを作った](https://zenn.dev/syumai/articles/ca9n4e91eqljc44k6ebg)

しかし、最近 syumai さんのツイートで sql.Open で d1 を扱えるようになったというのを知り、D1 を使った実用的な Workers アプリが Go で書けるのではと色々試した結果、syumai/workers + michi + sqlc という組み合わせがいい感じだと思ったので共有します。

## 実際に動くもの

実際に動くものを見た方が早い人のために実装例のリンクを貼っておきます。

<https://github.com/zztkm/go-workers-d1-sqlc>

## sqlc を選んだ理由

自分は以前 SQL を書く仕事をしていたので、ORM を使わず SQL でクエリを記述するのに抵抗はありませんでした。
しかし、SQL で取得した結果を model にマッピングする実装を自分で書いていたときはマッピングミスなどをやっていました。
そんなときに出会ったのが sqlc でした（たしか時雨堂の V さんがツイートしていたのを見かけて知った気がする）。

書いた SQL から Type Safe なコードを生成してくれるツールというのはまさに自分が求めていたものでした。

今では sqlc が普通に技術選定の選択肢になっていて、以下のように使い分けています。

- 定型クエリは sqlc
- 動的なクエリが必要なときはクエリビルダ
  - Go でクエリビルダを使ったことはないけど、[goqu](https://doug-martin.github.io/goqu/) とか便利そう

## michi を選んだ理由

syumai/workers は HTTP Server を動かす基盤を提供してくれますが、ルーティング周りは自分で用意する必要があります。

このルーティング部分を解決するライブラリとして michi を選んだ理由は以下の機能概要を見て Cloudfare Workers で動かすのに合っているなと思ったからです。

![スクショ](/images/21262273c01d65c291aa27268fa2d847.png)
スクリーンショット引用元: [Go 1.22のEnhanced ServeMuxに合わせて設計されたルーティングライブラリmichi](https://zenn.dev/sonatard/articles/831b761a27b230) より

特に外部パッケージ依存がなく小さい実装であることと 、net/http と 100 % 互換性というのは Cloudfare Workers で使っていく上で個人的に嬉しい内容です。

なお、TinyGo の net パッケージが Go 1.21.4 までの対応で、michi の機能は Go 1.22 以降の Go の機能が必要なため、TinyGo でのビルドは現状対応できていません。

## syumai/workers を選んだ理由

これがないと Workers Go は始まらないので、基礎ライブラリとして重要です。

ユーザーから見れば、Go 標準機能を使って HTTP Server を動かし、D1 を扱えるので、net/http や database/sql のインターフェースで使えるライブラリの恩恵を簡単に受けることができます。

この設計のおかげで、自分が使いたい sqlc や michi を利用することができています。非常に素晴らしいです。

## 課題

- TinyGo でのビルドができない
  - Go 1.22 以降 net/http の機能が必要なため

まだ使い切れていないので、今後課題は更新していく。

## まとめ

syumai/workers のおかげで Cloudflare Workers で Go の HTTP サーバーを動かすハードルがめちゃくちゃ下がりました。

今は TinyGo でビルドできないですが、今後も syumai/workers + sqlc + michi という組み合わせで Workers アプリを書いていこうと思います。
