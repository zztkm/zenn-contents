---
title: "はじめての C++ プロジェクト with Blend2D"
emoji: "🎄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp", "clangd", "blend2d"]
published: true
---

この記事は [zztkm振り返りカレンダー Advent Calendar 2024](https://adventar.org/calendars/10960) 2 日目の記事です。

第二回目は、自分が C++ プロジェクトを初めて立ち上げたときに行ったことをまとめてみようと思います。

## まとめ

- ビルドツールは [CMake](https://cmake.org/) と [Ninja](https://ninja-build.org/)
- ビルド補助ツールは Python
  - [uv](https://docs.astral.sh/uv/)
- Language server は [clangd](https://clangd.llvm.org/)

## 今回例にするプロジェクト

[Blend2D](https://blend2d.com/) という C++ で書かれた高性能な 2D ベクターグラフィックスエンジンを使って、クリエイティブコーディングをしたり、ダミー映像を作成したりしてみたかったので Blend2D をやってみたリポジトリです。そもそも C++ プロジェクトを作成するのも初めてだったので勉強用リポジトリとしても活用しています。

https://github.com/zztkm/hello-blend2d

このプロジェクトのビルドは Windows11 と Ubuntu 24.04 で問題なくビルドできることを確認しています。

## CMake

CMake は C++ コードをビルドするためのデファクトスタンダードなビルドシステムです。
なお僕は全然 CMake には詳しくなく、`CMakeLists.txt` も Blend2D の Getting Started を参考にというかほぼそのまま使ったので全然わかってません。
ただ、このままではこの先困るのでちゃんと CMakeLists を読み書きできるようになりたいなと思ってます。

https://cmake.org/

## Ninja

Ninja を使った理由

- いくつかの C++ プロジェクトを覗いたときに Ninja が使われているパターンがそこそこあった
- Ninja のマニュアルが読みやすかったから
  - <https://ninja-build.org/manual.html>
- Ninja を使うことで、`compile_commands.json` を生成できる
  - これはあとで出てくる clangd で使う
  - Windows のデフォルトジェネレーター `Visual Studio` では生成されないらしいが、よくわかっていない。

https://ninja-build.org/

## ビルド補助ツール

CMake と Ninja がビルドツールですが、このビルドにはいくつかの手順があり、それをいちいちやるのは大変なのでビルド手順を 1 コマンドにまとめるための Python スクリプトを書いています。

例とした hello-blend2d の README に書いてありますが、必要な依存関係をインストールした状態で `uv run build.py` を実行するだけで成果物となるバイナリをビルドする一連のコマンドを実行してくれます。

このアイデアは時雨堂の C++ プロジェクトからもらいました。

C++ のようにビルド成果物を得るまでの手順がそれなりあるプロジェクトでは Python のようなある程度どこの環境でも動くスクリプトを言語を利用してビルドツールを書くのは有用であると思いました。面倒なことは Python にやらせよう、です :D

今では uv などもあり、Python のインストールと管理がずいぶん楽になったので、Python を使いやすくなったこともこの選択の後押しになるのではないかと思います。

## clangd

clangd は C++ のための Language server です。
https://clangd.llvm.org/

インストールは <https://clangd.llvm.org/installation> を参照してください（各 Editor へのプラグイン導入方法も記載されています）。

ビルド中に実行されるコマンド `cmake -G "Ninja"` でビルドシステムを生成すると、`compile_commands.json` が生成され、clangd はそれを使って依存関係も含めたコード保管をすることができるようになります。

自分はプラグインのインストールが一番かんたんだった VS Code を選択しました。

:::message
VS Code を起動した状態で、`compile_commands.json` を生成しても、補完が効かない場合は、VS Code を再起動すると効くようになるはずです。
:::

## さいごに

hello-blend2d をビルドして `./app/build/app` を実行すると、コマンドを実行したディレクトリに output.png というファイルができます。

![output](https://i.gyazo.com/54212122fffd19822e12036b840e898b.png)

`app/app.cpp` をいじれば色々出力を変えて楽しめるのでぜひ試してみてください！

