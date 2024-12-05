---
title: "DevTools を使ってサイトのパフォーマンスを改善してみた"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["devtools", "performance"]
published: true
---

この記事は [zztkm振り返りカレンダー Advent Calendar 2024](https://adventar.org/calendars/10960) 5 日目の記事です。

今日の記事は DevTools を使ってサイトのパフォーマンスを改善してみた、です。
自社サイトのパフォーマンス改善をやってみようと思ったきっかけは mizchi さんが書いた「DevTools の使い方を可能な限りスクショ付きで解説してみる」を読んで面白そうだと思ったからです。

https://zenn.dev/mizchi/scraps/c72e6a55deca18

## 最初にまとめ・感想

- Google Fonts を削除せずにパフォーマンス改善をするの難しそう
- Google Fonts を利用しない場合にウェブセーフフォントを選択するのは良い選択肢になる
- DevTools の扱い方を学ぶのに mizchi さんが書いた「DevTools の使い方を可能な限りスクショ付きで解説してみる」は最初の取っ掛かりとして助かる

## この記事の前提

- 自分（ひとり）の会社で、ウェブサイトには必要なことが書いてあればよく、Google Fonts はこれが良いなどの要件はありませんでした
  - なんとなくこれ良いなで選んだ Font を使っていました
- 今回は DevTools を用いてどう改善していくかの初手を実際にやってみることが目的
- Performance を 100 にするのが目標
  - すべての項目を 100 にするのはスコープ外

Lighthouse 計測環境
![Lighthouse 計測環境](/images/20241205-site-performance-env.png)

## やってみた

### まずは Lighthouse で計測

以下のスクショにある通り 98 点と惜しいスコアでした（html + css のサイトなのでもともとスコアが高いのはおいておく）。
せっかくなのであと 2 点を取りにいきます。

![パフォーマンス改善前のLighthouseスコア](/images/20241205-site-performance-kaizenmae.png)

![Lighthouse Scoring Calculatorスクショ](/images/20241205-lighthouse-scoring-calculator.png)

上の2枚のスクショから、このサイトは FCP (First Contentful Paint) がスコアを下げる要因 = 課題になっていることがわかりました。

### Lighthouse Diagnostics を読む

上からスコア影響が大きいものが並んでいるとのことなので、まずは一番上を見てみると、
どうやらレンダリングをブロックしてしまっているリソースがあるということがわかりました。

今回はこれを潰せばスコアが 100 になりそうということで潰していきます。

![Lighthouse Diagnostics](/images/20241205-lighthouse-diagnostics.png)

xz.style は new.css という OSS の CSS framework を利用していたときのもので、消し忘れだったのでこれはこの時点で削除しました。

### レンダリング ブロック リソースを排除する

今回レンダリング ブロック リソースになっているのは Google Fonts であることがわかりました。
Lighthouse にもリンクが記載されていますが、この問題に対処するときの参考リンクがあるので飛びます。

https://developer.chrome.com/docs/lighthouse/performance/render-blocking-resources?utm_source=lighthouse&utm_medium=devtools&hl=ja

リンク先のページを読むと preload リンクを使用して、スタイルを非同期で読み込むという対処法で改善できそうだとわかったので、実際に試してみます。

ここで安直に「google fonts preload」と検索し、出てきた[【令和最新版】Google Fontsの読み込み最適化の結論](https://www.tak-dcxi.com/article/optimization-of-google-font-loading/)を参考に HTML の head 要素を修正してみました。

すると、FCP は改善したのですが、CLS (Cumulative Layout Shift) が約 8 倍の数値になってしまい、Performance スコアが 89 に下がりました（現時点ではこの原因を理解できていません。

![GoogleFonts preload](/images/20241205-googlefonts-preload.png)

### Google Fonts を削除する

なる早で Performance スコアを 100 にしたかったというのもあり、自分の持っている知見では Google Fonts を使いながら 100 を目指すのは厳しいなと判断したので、今回は Google Fonts を使わないように修正していく方針に変更しました。

じゃあ、どんなフォントを使えばよいの？というのを調べました。

### ウェブセーフフォント

Web のことは mdn に聞こうと「基本的なテキストとフォントのスタイル設定」を読んでいたところ、ウェブセーフフォントというものがあることを知りました。

ウェブセーフフォントはほぼすべてのシステム(PC、モバイル)にインストールされているフォントを指す言葉だそうです。

https://developer.mozilla.org/ja/docs/Learn/CSS/Styling_text/Fundamentals#%E3%82%A6%E3%82%A7%E3%83%96%E3%82%BB%E3%83%BC%E3%83%95%E3%83%95%E3%82%A9%E3%83%B3%E3%83%88

今回は Font をインターネットから取得しないようにするので、ウェブセーフフォントを使っておくと良いのかなと考え、以下のように font-family を指定するように変更してみました。

```css
font-family: Helvetica, Arial, sans-serif;
```

### Google Fonts をウェブセーフフォントに変えた結果

Google Fonts からウェブセーフフォントに変更してみた結果、当然ですがレンダリングブロックリソースが排除されたため、FCP を改善することができ、目標だった Performance スコアを 100 にすることができました。

今回は自分の要件に Google Fonts がなかったしフォントにこだわりがない（大抵のシステムにインストールされているフォントを選択するのは良い判断だとも思いますが）、というのがあったので Google Fonts をやめることができました。しかし、現実には Google Fonts をやめられない場合もあると思うので、そういう場合はどう対応しているのかが気になりました。

![Lighthouse 最終結果](/images/20241205-owari.png)

## わからなかったのであとで調べたいこと

- Google Fonts を preload したときに、なぜ CLS に影響してしまったのか
- 仮に Google Fonts で特定の Font を使うことが要件の場合は、どういう対応が取れるのか

## 終わりに

今回は自分のウェブサイトを実験場に DevTools を使ってパフォーマンス改善をやってみました。DevTools の使い方の初歩の部分をやってみましたが、使い方が少しは理解できたんじゃないかと思います。
ここまで読んでくれてありがとうございました！

実際に試してみたときの感想ポスト
https://x.com/zztkm/status/1864323318515683816

使ったサイト
https://veltiosoft.com/

