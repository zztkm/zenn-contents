---
title: "Poetry でCERTIFICATE_VERIFY_FAILEDが出る時の対処法"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["poetry"]
published: false
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

以下の 3 つのホストを信頼できるソースとして設定に追加します。

- pypi.org
- files.pythonhosted.org
- pypi.python.org
	- pypi.org にリダイレクトされます

## 設定方法

poetry で依存管理しているプロジェクト内で以下のコマンドを実行していくだけです。

TODO: pypi を cert false するだけで良いか検証する
```
poetry config certificates.pypi.cert false
```

### 1. Default Package Source の変更

https://python-poetry.org/docs/repositories/#default-package-source

### 2. 追加したソースたちの証明書検証を無効化

https://python-poetry.org/docs/repositories/#custom-certificate-authority-and-mutual-tls-authentication

## 設定の解説

### 1. Default Package Source の変更

まず、

### 2. 追加したソースたちの証明書検証を無効化

