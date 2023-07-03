---
title: "Echoでリクエストボディをロギングする"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["go", "echo"]
published: true 
---

## はじめに

Echoを使ったHTTPサーバーでリクエスト内容(主にボディ)をロギングしたいなと考えたので、その方法を調べたメモを残します。

ベースはこの記事を参考にしました。ありがとうございました。
[GoでRequest/Response BodyをロギングするHTTPサーバから見えてくるnet/http serverの実装 - RareJob Tech Blog](https://rarejob-tech-dept.hatenablog.com/entry/2021/11/15/100650)


### ログの方針

- json 形式のログを吐きたい
    - Datadog に転送して、Datadog logs で可視化する予定

## コード

ポイント
- Middleware は Handler の前に実行される
    - https://echo.labstack.com/middleware/
- io.ReadAll でリクエストボディを読み込むと、そのままでは Handler で読み込むことができなくなる
    - そのため logging したあと、リクエストボディを補填する必要がある
    - https://pkg.go.dev/io#NopCloser

```go
package main

import (
	"bytes"
	"io"
	"net/http"

	"github.com/labstack/echo/v4"
	"github.com/labstack/gommon/log"
)

// logBody はリクエストボディをログに出力するミドルウェアです
func logBody(next echo.HandlerFunc) echo.HandlerFunc {
	return func(c echo.Context) error {
		buf, _ := io.ReadAll(c.Request().Body)
		c.Logger().Info(string(buf))
		c.Request().Body = io.NopCloser(bytes.NewBuffer(buf))
		return next(c)
	}
}

func main() {
	e := echo.New()
	e.Logger.SetLevel(log.INFO)
    // ミドルウェアを追加
	e.Use(logBody)
	e.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "Hello, World!")
	})
	e.POST("/", func(c echo.Context) error {
		return c.JSON(http.StatusOK, map[string]interface{}{"message": "ショートカットに捧げよ"})
	})
	e.Logger.Fatal(e.Start(":1323"))
}
```

## おわりに

Go Echo を使っているときに、リクエストボディをロギングする方法を紹介しました。
より良い方法があれば参考にしたいのでぜひ教えてください！

今回使ったコード
https://github.com/zztkm/echo-logging-request-body-middleware