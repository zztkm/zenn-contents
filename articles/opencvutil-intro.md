---
title: OpenCV の便利な関数ライブラリを作ってる
emoji: "📸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python3", "rust", "opencv", "pyo3"]
published: true
---

## はじめに

ちょっと前から、OpenCV を使って色々やっていたのですがこういう関数あったら便利だよなーというのをライブラリとして作ってみようと思いました。

また、最近 Rust と [PyO3](https://github.com/PyO3/pyo3) にも興味を持っていたので、それを使ってライブラリを作ってみることにしました。

## 実際に作ったもの

opencvutil というライブラリを作りました。
util という名前にしてしまったので、ちょっと汎用的な名前になってしまいましたが、ちょこちょこ関数を追加していく予定です。

https://github.com/zztkm/opencvutil

### 使い方のイメージ

普通の Python モジュールとして import して使うことができます。

opencvutil で カメラ index を取得して、ユーザーが選択したカメラを使って映像を表示する例です。
```python
import cv2
import opencvutil

# カメラの情報を取得
infos = opencvutil.camera_list()

# カメラの情報を表示し、ユーザーに選択させる
for i, info in enumerate(infos):
    print(f"Index {info['index']}: {info['name']}")

# カメラを選択
camera_index = int(input("使用するカメラの Index を指定してください: "))

# OpenCV でカメラを開く
cap = cv2.VideoCapture(camera_index)
while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    # 画像を表示
    cv2.imshow("frame", frame)

    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

cap.release()
cv2.destroyAllWindows()
```

## Rust PyO3 での Python ライブラリ開発時に参考になったリンク

基本はここ
- [The Rust Programming Language 日本語版 - The Rust Programming Language 日本語版](https://doc.rust-jp.rs/book-ja/)
  - そもそも Rust の基本ができていないので読みながら Rust のコードを書いた
- [PyO3 のドキュメント](https://pyo3.rs/v0.21.0-beta.0/)
- [PyO3 の examples](https://github.com/PyO3/pyo3/tree/main/examples)
- [ビルドツール Maturin のドキュメント](https://www.maturin.rs/)

## 今後の課題

- まだまだ関数が少ないので、関数を追加していくこと
  - 自分で OpenCV で遊んでいく中で必要な関数を追加していく
  - 何かアイデアがあったら issue で教えてもらえると嬉しいです
- Linux (Ubuntu) でのビルドが失敗するので、それを解決する
  - Rust, PyO3, Maturin は悪くない 
  - 今は Windows, Mac でしかビルドできていない
  - 自分の依存解決手段が悪いのか、必要なライブラリが見つからないというエラーが出てしまってる...
    - https://github.com/zztkm/opencvutil/actions/runs/8355321536/job/22870186105#logs
- stub ファイルの自動生成
  - 今は手動で書いているので、自動生成できるようにしたい
  - issue は上がってるようなので、それを追っていく
    - https://github.com/PyO3/pyo3/issues/2454