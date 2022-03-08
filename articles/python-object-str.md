---
title: "Python Object の文字列表現"
emoji: "🪵"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "python3"]
published: false
---

Python の object を `print(object)` したときなどにいい感じに表示されたけどなんでだろうと思ったので調べてみたことをまとめた記事です。

提案や指摘はコメントかTwitterで連絡もらえると助かります！

筆者の実行環境は以下のようになっています(OS関係)
- Windows 10
- Python3.9

# 目次

- 今回用いる例


# 📚今回用いる例

todo: なんかいい感じなやつを使う

今回は以下のような実装の架空の`User`クラスが存在するとして進めます。

ちなみにこのクラスの参考元は[discord.py](https://github.com/Rapptz/discord.py/blob/master/discord/user.py)です。

```python
class User:

	def __init__(self, name: str, id: int, tag: str):
		self.user = name
		self.id = id
		self.tag = tag

	def __str__(self) -> str:
		return f"{self.name}#{self.tag}"

	def send
```

# 🎮実際に動かしてみる


