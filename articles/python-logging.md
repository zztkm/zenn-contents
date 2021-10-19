---
title: "Pythonの基本的なlogging"
emoji: "🪵"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["logging", "python", "python3"]
published: true
---

Python のロギング方法について [ログ出力のための print と import logging はやめてほしい](https://qiita.com/amedama/items/b856b2f30c2f38665701) という記事や他にも色々な記事でこういうロギングがあるよと紹介されています。

さまざまなロギング方法の中でどうすれば良いか迷ってしまう人もいるかもしれないと思い、僕が普段から実践している(多分)基本的なロギング方法を紹介しようかと思います。

アプリケーションによっては設定を追加したり、フォーマットを変更したりなどのカスタマイズをしますが、基本的な部分は変わらないと思うのでなにかの参考になれば幸いです。
記事の構成としてはライブラリとアプリケーションでわけて記述していきます。

適当に考えていたことや参考情報は [Scrap](https://zenn.dev/tkm/scraps/6a74a4e351495c) に記載しています。

**筆者の環境**

- Windows 10
- Python3.9

# 📚ライブラリでのロギング

ライブラリのロギング設定は基本的に [Logging HOWTO - Python](https://docs.python.org/ja/3/howto/logging.html#configuring-logging-for-a-library) の「ライブラリのためのロギングの設定」に従っています。

具体的には以下の`foo`ライブラリのようにします。

```
foo/
│
├── __init__.py
└── bar.py
```

**ライブラリを使うアプリケーションに影響しないように NullHandler のみを追加します。**

```python:__init__.py
import logging

logging.getLogger(__name__).addHandler(logging.NullHandler())
# 以下省略
```

```python:bar.py
import logging

logger = logging.getLogger(__name__) # これはファイルでも共通


def bar(msg: str):
    logger.debug(f"msg length: {len(msg)}")
    return msg
```

基本的にライブラリのロギング設定はこれで十分です。
ここからはなぜ上記のようにするのかを見ていきます。

## なぜNullHandlerを使うのか

[Logging HOWTO - Python](https://docs.python.org/ja/3/howto/logging.html#configuring-logging-for-a-library) の「ライブラリのためのロギングの設定」には以下のような記述があります。

> 注釈 ライブラリのロガーには、 NullHandler 以外のハンドラを追加しない ことを強く推奨します。これは、ハンドラの設定が、あなたのライブラリを使うアプリケーション開発者にも伝播するからです。アプリケーション開発者は、対象となる聴衆と、そのアプリケーションにどのハンドラが最も適しているかを知っています。ハンドラを 'ボンネットの中で' 加えてしまうと、ユニットテストをして必要に応じたログを送達する能力に干渉しかねません。


文章だけ読んでもなるほど〜といった感じで終わってしまうので、わかりやすいように NullHandler 以外のハンドラを追加してしまうとどのようにログに影響してしまうのかの例として「ライブラリ内で logging.basicConfig が設定されてしまっている場合(あんま見たことないけど)」を考えてみます。

まず、logging.basicConfig は以下のような仕様になっています。

> デフォルトの Formatter を持つ StreamHandler を生成してルートロガーに追加し、ロギングシステムの基本的な環境設定を行います。関数 debug(), info(), warning(), error(), critical() は、ルートロガーにハンドラが定義されていない場合に自動的に basicConfig() を呼び出します。
> この関数は、キーワード引数forceがTrueに設定されていない限り、ルートロガーがすでにハンドラを構成している場合は何もしません。

[logging --- Python](https://docs.python.org/ja/3/library/logging.html#logging.basicConfig)より (一部DeepLを用いて翻訳)

ここで重要なのは、「ハンドラを生成してルートロガーに追加」するという点と、「ルートロガーがすでにハンドラを構成している場合は何もしない」という点です。

悪い例として`hoge`というライブラリのソースコードで確認していきます。

:::details hogeライブラリのソースコード
```
hoge/
│
├── __init__.py
└── huga.py
```

実装は下記のようになっています。
```python:__init__.py
import logging

logging.basicConfig(level=logging.INFO) # ライブラリ内でハンドラを追加している
# 以下省略
```

```python:huga.py
import logging

logger = logging.getLogger(__name__)


def huga(msg: str):
    logger.info(f"{msg} huga!")
```

:::

hoge ライブラを使ったスクリプト例

```python:if-basicconfig-set-in-library.py
import logging

from hoge import huga

logging.basicConfig(level=logging.DEBUG)

logger = logging.getLogger(__name__)


def main():
    logger.debug("レベルをDEBUGに設定しているのに表示されへん")
    huga("Hello!")
    logger.info("表示される")


if __name__ == "__main__":
    main()
```

実行結果

```shell
❯ python if-basicconfig-set-in-library.py
INFO:hoge.huga:Hello! huga!
INFO:__main__:INFOは表示される
```

`logger.debug("レベルをDEBUGに設定しているのに表示されへん")`の行のログが表示されていません。

これは、呼び出した `hoge` ライブラリに書かれている basicConfig(level=INFO) によって追加されたハンドラがルートロガーに存在するので、`if-basicconfig-set-in-library.py` に書いた basicConfig(level=DEBUG) の設定が無視されてしまうという事象がおこったため、想定したログレベル=DEBUG以上のログを吐き出すことができなかったことが原因です。

このように NullHandler 以外のハンドラを追加したことで、ユーザーが意図したロギング設定の妨げとなってしまいました。

このような想定外の挙動をおこさせないためにもライブラリで設定するロガーには NullHandler を追加するようにしておきましょう。

:::message

もしプログラムが実行された順番を自分で確認したい場合は、以下のように trace モジュールなどで確認することができます(出力結果がめっちゃ多いので適当にファイル書き込みしてます)。

```shell
python -m trace --trace if-basicconfig-set-in-library.py > trace.log
```
参考リンク: [trace --- Python](https://docs.python.org/ja/3/library/trace.html)
:::


# 🎮アプリケーションでのロギング

次にアプリケーションでのロギング方法です。

これは標準ライブラリの logging を使う場合とサードパーティの [Loguru](https://github.com/Delgan/loguru) を使う場合でわけて記述します。

## 前提

ライブラリでのロギングで紹介した foo ライブラリの省略部分を書き加えて説明していきます。

:::details fooライブラリのソースコード

```
foo/
│
├── __init__.py
└── bar.py
```

ファイルの内容

```python:__init__.py
import logging

logging.getLogger(__name__).addHandler(logging.NullHandler())

from .foo import bar

__all__ = ["bar"]
```

```python:bar.py
import logging

logger = logging.getLogger(__name__)


def bar(msg: str):
    logger.info(f"msg length: {len(msg)}")
    return msg
```

:::


## logging を使用する場合

```python:logging-example.py
import logging

from foo import bar

logging.basicConfig(level=logging.DEBUG)

logger = logging.getLogger(__name__)


def main():
    logger.info("start")
    bar("Hello!")
    logger.info("finished")"

if __name__ == "__main__":
    main()
```

実行結果

```shell
INFO:__main__:start
DEBUG:foo.bar:msg length: 6
INFO:__main__:finished
```

めっちゃシンプル。
メッセージのフォーマットを変更したり、記録された時刻などを残したりしたい場合は設定をしてあげる必要があります。そんなときは Logging HOWTO - Python [ここらへん](https://docs.python.org/ja/3/howto/logging.html#changing-the-format-of-displayed-messages)の「表示されるメッセージのフォーマットの変更」などを確認してください。


## Loguru を使用する場合

今回のユースケースでは foo ライブラリは標準の logging モジュールを使っていて、loguru-example.py は Loguru を利用しています。
この場合、Loguru が logging の内容を受け取れるようにしなければなりません。そのやり方は Loguru の [REAMDME](https://github.com/Delgan/loguru#entirely-compatible-with-standard-logging) に記述されており、下記のような Handler を追加することで実現可能です。

```python:loguru-example.py
import logging

from loguru import logger

from foo import bar


# logging の内容を Loguru で受け取る処理を記述した Handler
class InterceptHandler(logging.Handler):
    def emit(self, record):
        try:
            level = logger.level(record.levelname).name
        except ValueError:
            level = record.levelno

        frame, depth = logging.currentframe(), 2
        while frame.f_code.co_filename == logging.__file__:
            frame = frame.f_back
            depth += 1

        logger.opt(depth=depth, exception=record.exc_info).log(level, record.getMessage())


logging.basicConfig(handlers=[InterceptHandler()], level=0)


def main():
    logger.info("start")
    bar("Hello!")
    logger.info("finished")

if __name__ == "__main__":
    main()
```

実行結果

```shell
❯ python loguru-example.py
2021-10-19 13:07:00.688 | INFO     | __main__:main:28 - start
2021-10-19 13:07:00.689 | DEBUG    | foo.bar:bar:7 - msg length: 6
2021-10-19 13:07:00.689 | INFO     | __main__:main:30 - finished
```

logging を扱うためにハンドラの定義が必要ですが、標準の logging とは違ってフォーマットに関する設定をしなくても、時間やファイルの行数などを出力してくれています。フォーマットも必要に応じて変更できるので、最終的にログをどのように運用するかによってフォーマットをいじったり出力先を調整したりしてください。

# おわりに

ライブラリのロギングとアプリケーションのロギングについて記述してきました。
ライブラリのロギングについては、紹介した方法で十分だと考えていますが、アプリケーション側はログの運用方法次第で工夫が必要になってくると思います。

Loguru については細かい設定なしに確認しやすいログが出力されるのでデバッグするときなどに重宝しています。設定をがんばる必要がないのが個人的にうれしいポイントです。

今回紹介した方法を元にみなさんの都合の良いロギング方法を見つけてみてください。

記事に対する意見があればコメントなどしてもらえると助かります！

# リファレンス

- [Logging HOWTO - Python](https://docs.python.org/ja/3/howto/logging.html#configuring-logging-for-a-library)
- [Logging クックブック - Python](https://docs.python.org/ja/3/howto/logging-cookbook.html)
- [Python Logging: A Stroll Through the Source Code - Real Python](https://realpython.com/python-logging-source-code/#library-vs-application-logging-what-is-nullhandler)

# CHANGELOG

## 2021-10-19
- 初公開