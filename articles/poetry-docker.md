---
title: "Poetry で管理した依存を Docker コンテナ内でインストールする"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["poetry", "docker"]
published: true
---

## はじめに

[Poetry](https://python-poetry.org/docs/) を用いた依存関係管理をやりつつ、Docker コンテナでアプリケーションを動かしたかったので調べた備忘録としてこの記事を残しておきます。

また、もっと良いソリューションがあるよ！とか、こういうやり方もあるよ！とかあったらコメントや Twitter で教えてもらえると助かります。

## 例となるプロジェクト構成

以下のようなプロジェクトを例にします。
https://github.com/veltiosoft/mp7

```
.
├── docker-compose.yaml
├── Dockerfile
├── .env
├── Makefile
├── poetry.lock
├── pyproject.toml
├── README.md
├── src
│  ├── cogs
│  │  ├── __init__.py
│  │  ├── google.py
│  │  ├── text_channel.py
│  │  └── thread.py
│  ├── main.py
│  └── views
│     ├── __init__.py
│     └── button.py
└── tox.ini
```


## poetry export を使ってインストール

`poetry export` コマンドを利用します。

export コマンドについてはこちら: https://python-poetry.org/docs/cli/#export

```Dockerfile
ARG PYTHON_ENV=python:3.11-slim

FROM $PYTHON_ENV as build

RUN pip install poetry

RUN mkdir -p /app
WORKDIR /app

COPY pyproject.toml poetry.lock ./
RUN poetry export --only main --without-hashes -f requirements.txt -o /requirements.txt


FROM $PYTHON_ENV as prod

COPY --from=build /requirements.txt .
RUN pip install -r requirements.txt

COPY src /app
WORKDIR /app

EXPOSE 8080
ENTRYPOINT ["python"]
CMD ["main.py"]
```

### やってることの解説

今回の Dockerfile の構成は `build` ステージと `prod` ステージに分かれています。

ざっくり
- `build`: requirements.txt を生成するステージ
- `prod` : 依存関係のインストールとアプリケーションの起動

**build ステージ**

build ステージは以下のような流れで処理されます。

1. poetry のインストール
2. 作業ディレクトリの作成
3. pyproject.toml とpoetry.lock を作業ディレクトリにコピー
4. poetry で管理していた依存関係を `requirements.txt` に書き出し


export コマンドの解説
- `--only main`: main の依存関係のみを export 対象にする
- `--without-hashes`: export に hash を含めないためのオプション
	- hash についてはこちらを参照: https://pip.pypa.io/en/stable/cli/pip_hash/
- `-f requirements.txt`: export のフォーマット指定
- `-o /requirements.txt`: 今回は `/requirements.txt` に出力する

これで requirements.txt が作れました。

**prod ステージ**

あとは簡単で、prod ステージに build ステージで作成した requirements.txt をコピーして、`pip install` するだけです。


## 終わりに

他にもいい感じなやり方が見つかったり、誰かに教えてもらったときには記事を更新していこうと思います。

