---
title: "shutil.rmtree で削除失敗するファイルに対する対策"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python3"]
published: true
---

この記事は [zztkm振り返りカレンダー Advent Calendar 2024](https://adventar.org/calendars/10960) 8 日目の記事です。

今回は Python で shutil.rmtree を利用したときに遭遇した問題の対応策を書きます。

## 起こった問題


- Windows で git ディレクトリを削除するのに shutil.rmtree を利用
- 普通に実行すると以下のようなエラーが出る
  - `PermissionError: [WinError 5] アクセスが拒否されました。: 'blend2d\\.git\\objects\\pack\\pack-927607be891be214450291520bdac1d9cdde99a9.idx'`
- ディレクトリ丸ごと削除したいので、`ignore_errors=True` ではだめ
  - これは単にエラーが起こった場合に無視するだけなので

## shutil.rmtree で削除失敗したときの対策

- 引数 onexc に以下のコード例のようにコールバックを設定する
  - これで確実に削除していけるはず
    - ダメなパターンはあるのだろうか？、あんま詳しくわかってない

```python
import os, stat
import shutil

def remove_readonly(func, path, _):
    "Clear the readonly bit and reattempt the removal"
    os.chmod(path, stat.S_IWRITE)
    func(path)

shutil.rmtree(directory, onexc=remove_readonly)
```
Python のドキュメントから引用しました: https://docs.python.org/3/library/shutil.html#rmtree-example

なんでだろうと思って公式ドキュメントを読みにいったらライブラリの説明としてちゃんと記載されていました(ご丁寧に example code つき)。

## どこで使うの？

自分の場合は C++ プロジェクトのビルド補助ツールを Python で書いていたときに、git clone してきた依存関係をクリーンインストールするためにディレクトリごと削除しようとしたときに git ディレクトリを削除できないという問題に遭遇したので、その削除処理で利用しました。

具体的にはこういう部分です
https://github.com/zztkm/hello-blend2d/blob/main/build.py#L27-L29

## おわりに

Python のドキュメントは結構充実していて、今まで何度も助けられている気がします。

明日のアドベントカレンダーの記事は何を書こうかな...

