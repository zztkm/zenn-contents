---
title: "uv で Python 入門"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python3", "rye", "uv"]
published: false
---

## はじめに

この記事は uv を使って Python に入門することを目的としています。

uv の細かい説明はドキュメントを読んでもらうのが良いと判断し、ドキュメントに書かれていることは省略し、参考リンクを貼る形で進めていきます。

uv に興味を持った方はぜひ公式ドキュメントを読んでみてください！

uv リソース

- <https://docs.astral.sh/uv/>
- <https://github.com/astral-sh/uv>

## uv とは

ほぼドキュメントへの参考リンクにしようとは思いますが、流石に何も紹介しないのは良くないので、簡単に紹介します。

> An extremely fast Python package and project manager, written in Rust.
<https://docs.astral.sh/uv/> より引用

公式サイトで上記のように紹介されている通り、Rust で書かれた非常に高速な Python パッケージとプロジェクトマネージャーです。

以下のようなことができます

- Python の管理
  - uv を使って お好きなバージョンの Python をインストールすることができる
- Python プロジェクト管理
  - `rye` や `poetry` のようにプロジェクトを管理することができる
- ツール管理
  - Python で開発されたコマンドラインツールなどを管理することができる

## Python 管理

uv は Python をインストールし、バージョンを素早くきりかえることができます。
TODO: ここの文章をもうちょいわかりやすくというか、実態にあった言い方をしたい

ここを説明するのに重要なドキュメント
- https://docs.astral.sh/uv/guides/install-python/
- https://github.com/indygreg/python-build-standalone
- https://docs.astral.sh/uv/concepts/python-versions/#pypy-distributions
