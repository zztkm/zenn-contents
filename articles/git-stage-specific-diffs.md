---
title: "Git でファイル内の特定の差分だけをステージングする"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git"]
published: true
---

この記事は [zztkm振り返りカレンダー Advent Calendar 2024](https://adventar.org/calendars/10960) 9 日目の記事です。

今回の記事では Git でファイル内の特定の差分だけをステージングする方法を紹介します。

## git コマンドでのやり方

git コマンドで特定行の差分だけをステージングしたいときは、対話的なステージング機能を利用します。

https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%81%95%E3%81%BE%E3%81%96%E3%81%BE%E3%81%AA%E3%83%84%E3%83%BC%E3%83%AB-%E5%AF%BE%E8%A9%B1%E7%9A%84%E3%81%AA%E3%82%B9%E3%83%86%E3%83%BC%E3%82%B8%E3%83%B3%E3%82%B0

コマンドは `git add -p <対象ファイル>` です。

例えば、以下のような diff を持つファイルがあるとします

```diff
$ git diff src/main.rs
diff --git a/src/main.rs b/src/main.rs
index 9fc53e9..daf61c7 100644
--- a/src/main.rs
+++ b/src/main.rs
@@ -70,6 +70,9 @@ pub enum Commands {
     /// $ todo create "Buy milk" --description "Buy 2 milk" --due-date "2024-12-31"
     Add(todo::AddOptions),

+    /// Add todo (short version)
+    A(todo::AddOptions),
+
     /// Mark a todo as done.
     Done(todo::UuidOptions),

@@ -86,6 +89,9 @@ pub enum Commands {

     /// Edit todo.
     Edit(todo::EditOptions),
+
+    /// Edit todo (short version)
+    E(todo::EditOptions),
 }

 fn main() {
```

2箇所に diff がありますが、上の `A(todo::AddOptions),` の2行のみをステージングするときの様子を示します。

まず git add -p を実行すると `Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]?` と質問が表示されます。

選択肢の細かい説明はしませんが、たいていは `y` or `n` でステージする/しないを指定していくと思います(ご想像の通り y がステージする、n がステージしない、です)。

```bash
$ git add -p src/main.rs                                                                                                   21:46:19
diff --git a/src/main.rs b/src/main.rs
index 9fc53e9..daf61c7 100644
--- a/src/main.rs
+++ b/src/main.rs
@@ -70,6 +70,9 @@ pub enum Commands {
     /// $ todo create "Buy milk" --description "Buy 2 milk" --due-date "2024-12-31"
     Add(todo::AddOptions),

+    /// Add todo (short version)
+    A(todo::AddOptions),
+
     /// Mark a todo as done.
     Done(todo::UuidOptions),

(1/2) Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]? y
@@ -86,6 +89,9 @@ pub enum Commands {

     /// Edit todo.
     Edit(todo::EditOptions),
+
+    /// Edit todo (short version)
+    E(todo::EditOptions),
 }

 fn main() {
(2/2) Stage this hunk [y,n,q,a,d,K,g,/,e,?]? n
```

この結果、現在の状態を確認すると以下のように、src/main.rs がステージされている変更とされていない変更に分かれることがわかります。

```bash
~/dev/github.com/zztkm/todo main* 34s ❯ git status                                                                                                           21:49:31
ブランチ main
Your branch is up to date with 'origin/main'.

コミット予定の変更点:
  (use "git restore --staged <file>..." to unstage)
        modified:   src/main.rs

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md
        modified:   src/main.rs
```

ステージされている差分を確認すると、最初に言っていた通り A の部分だけステージされていることがわかります！

```diff
$ git diff --staged src/main.rs                                                                                            21:49:41
diff --git a/src/main.rs b/src/main.rs
index 9fc53e9..0b74a5e 100644
--- a/src/main.rs
+++ b/src/main.rs
@@ -70,6 +70,9 @@ pub enum Commands {
     /// $ todo create "Buy milk" --description "Buy 2 milk" --due-date "2024-12-31"
     Add(todo::AddOptions),

+    /// Add todo (short version)
+    A(todo::AddOptions),
+
     /// Mark a todo as done.
     Done(todo::UuidOptions),
```

## VS Code はもっとかんたん

以下のようにスクショの赤枠 1 番の部分のように色がついていますが、 これは git 上で差分がある場合にこのようになります。
この色付き線をマウスで左クリックします。

するとその部分にフォーカスした `Git ローカル作業の変更 - 1/2` のように表示されます。そこに赤枠 2 番のように `+` と書かれているボタンがあるのですが、これをクリックするとその部分だけステージすることができます。

![VSCode Git 部分的にステージングするやり方のスクリーンショット](https://i.gyazo.com/23c25960e2e121f4f2f0245a4821b1ed.png)


結果、git コマンドのときの同じように部分的にステージングできました。

![VSCode Git 部分的なステージング結果のスクリーンショット](https://i.gyazo.com/a9dc409ddf18898eb51674b78311e326.png)


