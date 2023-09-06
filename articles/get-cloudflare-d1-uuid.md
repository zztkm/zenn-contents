---
title: "Cloudflare D1 の UUID を取得するスクリプト"
emoji: "🥝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["d1", "wrangler"]
published: true
---

## 結論

こんな感じのスクリプトになります。

```bash
wrangler d1 list --json | jq -r '.[] | select(.name == "d1-example") | .uuid'
```

d1 list コマンドには json で書き出すオプションがあるので、それを jq でパースしています。
`name == "d1-example"` の部分はプロジェクトごとに置き換えてください。

## ユースケース

なんでこんなことをするのかというと、Cloudflare D1 のマイグレーションに、[sqldef (sqlite3def)](https://github.com/k0kubun/sqldef/) を使いたいのですが、sqliite3def は db_name にパスを指定する必要があります。
 
そこで、ローカルにある D1 (sqlite db) を参照したいのですが、Cloudflare D1 のローカルDBのパスは `.wrangler/state/v3/d1/${UUID}/db.sqlite` という形式になっているので、UUID を取得する必要があります。

:::message
UUID なんざプロジェクト固有の値なのだから、固定しておけば良いのでは？という意見は受け付けておりません。自分はこれを書きながらそのことに気づきました。
:::

自分の場合はこれを Makefile でやりたかったので以下リンク先のように実装しています。

https://github.com/zztkm/hono-sqlc-d1-example/blob/main/Makefile

これはこれで Makefile 実行の度に D1 UUID の取得をしやがるので、工夫したいなとは考えてます(shell script にしといた方が使いやすいかな 🤔)。

## おわりに

D1 + sqldef + sqlc スタックは結構良い感じなので、これからも使っていきたいです。

もっといい感じなソリューションがあればぜひ教えてください！


## 参考

- https://github.com/chimame/workers-remix-d1/blob/main/scripts/migration.sh
