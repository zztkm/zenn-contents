---
title: "Poetry でCERTIFICATE_VERIFY_FAILEDが出る時の対処法"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["poetry"]
published: true
---

:::message alert

以下で紹介する設定は信頼できるホストに対してのみ実施してください。
参考にする場合は自己責任でお願いします。

:::

pip の場合は以下の記事のように、`trusted-host` を設定することで証明書の検証をスキップすることができます。

poetry でも似たようなことができるのではないかと調査していたところ一応は実現できたので紹介しようと思います。

pip trusted-host の例
https://tex2e.github.io/blog/python/pip-trusted-host

trusted-hostについてはこちら
https://pip.pypa.io/en/stable/cli/pip/#trusted-host


## 今回信頼するホスト

以下の 2 つのホストを信頼できるソースとして設定に追加します。

- pypi.org
- files.pythonhosted.org

## 設定方法

poetry で依存管理しているプロジェクト内で以下のコマンドを実行していくだけです。

### 1. Package Source の追加

```
poetry source add --default pypiorg https://pypi.org/simple/
poetry source add files https://files.pythonhosted.org/
```

https://python-poetry.org/docs/repositories/#default-package-source

### 2. 追加したソースたちの証明書検証を無効化

```
poetry config certificates.pypiorg.cert false
poetry config certificates.files.cert false
```

https://python-poetry.org/docs/repositories/#custom-certificate-authority-and-mutual-tls-authentication

これで `poetry add pytest` のようにライブラリをインストールすることができるようになります。

## 設定の補足

### 1. Package Source の追加

:::message

`https://pypi.org/simple/` についてですが、こちらは `pypi` という名前でデフォルトで登録されているので `poetry config certificates.pypi.cert false` とコマンドを実行すれば証明書検証を無効化できると思ったのですができませんでした（すみませんが何故かはまだわかっていません...）。
そこで、同じURLで別のソースとして登録しデフォルトに設定してみたところ証明書検証を無効化することができました。

:::


## おわりに

今回紹介した設定はプロジェクトごとにソースの設定が必要となってしまい、pip.ini に trusted-host として設定すると同じようにグローバルに設定することはできませんでした。

別のやり方を見つけたり、説明できることが増えたら記事を更新する予定です。

ここまで読んでくれてありがとうございました 🍚

