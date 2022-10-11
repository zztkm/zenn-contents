---
title: "V 言語で OS の種類を判定する"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vlang"]
published: true
---

## はじめに

クロスプラットフォーム対応のアプリケーションを書くときに OS の種類を判定して処理を分けるといったことをやりたくなるときがあると思います。
そこで、今回は V 言語での OS 判定処理の実装方法を紹介します。

## OS の判定方法

V 言語では条件付きコンパイルという機能を利用して、コンパイル時の条件分岐 (`$if`) を使って OS の種類を判定することができます。

```v
module main

fn main() {
	$if linux {
		println('linux')
	}
	$if macos {
		println('macos')
	}
	$if windows {
		println('windows')
	}
}
```

output:
```shell
# macos で実行した場合
> v run main.v
macos
```

しかし毎回このような処理を書くのは面倒です。

そこで `os.user_os()` という関数を使います。

```v
module main

import os

fn main() {
	os := os.user_os()
	println('Using $os')
}
```

output:
```shell
# macos で実行した場合
❯ v run main.v
Using macos
```

`os.user_os()` も最初に示した例と同様のコードで定義されています。

```v
// user_os returns current user operating system name.
pub fn user_os() string {
	$if linux {
		return 'linux'
	}
	$if macos {
		return 'macos'
	}
	$if windows {
		return 'windows'
	}
	$if freebsd {
		return 'freebsd'
	}
	$if openbsd {
		return 'openbsd'
	}
	$if netbsd {
		return 'netbsd'
	}
	$if dragonfly {
		return 'dragonfly'
	}
	$if android {
		return 'android'
	}
	$if termux {
		return 'termux'
	}
	$if solaris {
		return 'solaris'
	}
	$if haiku {
		return 'haiku'
	}
	$if serenity {
		return 'serenity'
	}
	$if vinix {
		return 'vinix'
	}
	if getenv('TERMUX_VERSION') != '' {
		return 'termux'
	}
	return 'unknown'
}
```

以上です。


## 参考

https://github.com/vlang/v/blob/master/doc/docs.md#if-condition

https://modules.vlang.io/os.html#user_os