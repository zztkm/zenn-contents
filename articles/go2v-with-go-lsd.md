---
title: "go2v を使って Go を V に変換する"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vlang", go]
published: true
---

## ⏰ はじめに

 [go2v](https://github.com/vlang/go2v) というGo のコードを V に変換してくれるソフトウェアを試してみたので紹介します。


## 🍡 依存なしのプログラム

今回は mattn さんが作った [go-lsd](https://github.com/mattn/go-lsd) を V に変換してみます。
go-lsd はレーベンシュタイン距離アルゴリズムの Go の実装です。


go-lsd のコードはこれです。
https://github.com/mattn/go-lsd/blob/master/lsd.go

まずそのまま変換しようとしましたが、以下の点でうまくいきませんでした。

- 引数の型が省略してあると、引数が部分的に消えてしまう


:::details そのまま変換したときの出力

```v
module lsd

pub fn distance(lhs_1 []rune) int {
	mut rl1, rl2 := lhs_1.len, rhs.len
	mut costs := []int{len: rl1 + 1}
	for j := 1; j <= rl1; j++ {
		costs[j] = j
	}
	mut cost, last, prev := 0, 0, 0
	for i := 1; i <= rl2; i++ {
		costs[0] = i
		last = i - 1
		for j_1 := 1; j_1 <= rl1; j_1++ {
			prev = costs[j_1]
			cost = 0
			if lhs_1[j_1 - 1] != rhs[i - 1] {
				cost = 1
			}
			if costs[j_1] + 1 < costs[j_1 - 1] + 1 {
				if costs[j_1] + 1 < last + cost {
					costs[j_1] = costs[j_1] + 1
				} else {
					costs[j_1] = last + cost
				}
			} else {
				if costs[j_1 - 1] + 1 < last + cost {
					costs[j_1] = costs[j_1 - 1] + 1
				} else {
					costs[j_1] = last + cost
				}
			}
			last = prev
		}
	}
	return costs[rl1]
}

pub fn string_distance(lhs string) int {
	return distance(lhs.bytes(), rhs.bytes())
}
```
:::

なので引数の型指定を省略せずに書くようにします。

手直しの差分はこちら
https://github.com/zztkm/vlsd/commit/225b22b594b0fc50b105cbedc57571c070d2b34e


引数を修正して変換した結果

```v
module lsd

pub fn distance(lhs_1 []rune, rhs_1 []rune) int {
	mut rl1, rl2 := lhs_1.len, rhs_1.len
	mut costs := []int{len: rl1 + 1}
	for j := 1; j <= rl1; j++ {
		costs[j] = j
	}
	mut cost, last, prev := 0, 0, 0
	for i := 1; i <= rl2; i++ {
		costs[0] = i
		last = i - 1
		for j_1 := 1; j_1 <= rl1; j_1++ {
			prev = costs[j_1]
			cost = 0
			if lhs_1[j_1 - 1] != rhs_1[i - 1] {
				cost = 1
			}
			if costs[j_1] + 1 < costs[j_1 - 1] + 1 {
				if costs[j_1] + 1 < last + cost {
					costs[j_1] = costs[j_1] + 1
				} else {
					costs[j_1] = last + cost
				}
			} else {
				if costs[j_1 - 1] + 1 < last + cost {
					costs[j_1] = costs[j_1 - 1] + 1
				} else {
					costs[j_1] = last + cost
				}
			}
			last = prev
		}
	}
	return costs[rl1]
}

pub fn string_distance(lhs string, rhs string) int {
	return distance(lhs.bytes(), rhs.bytes())
}
```

良さそうに見えるんですが以下の点でエラーがあります。

- distance に渡す値が []u8 になっている
- 変数 last, prev が immutable になっている

上記の問題を修正したときの差分は以下
https://github.com/zztkm/vlsd/commit/28189e57c263cfa7c872fc235b2eb1bd7551f254

これで問題なく利用できるようになりました🎉

せっかくなので、GitHub で公開しています。

https://github.com/zztkm/vlsd


終わりです。ここまで読んでくれてありがとうございました！


TODO
- Go の標準ライブラリに依存したコードを変換してみる

