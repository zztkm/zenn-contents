---
title: "AWS CodeBuild でGo の CI を実行する"
emoji: "🐁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws", "codebuild", "go"]
published: true
---

## はじめに

AWS CodeBuild で Go 1.21.0 の CI を実行する環境を立てたのでそのメモです。

## 前提

- 利用する Docker イメージ
    - aws/codebuild/amazonlinux2-aarch64-standard:3.0
        - この環境イメージを CodeBuild で設定する
- Go のバージョン
    - 1.21.0

## Buildspec

point:
- Go 1.21.0 は公式イメージに含まれていないので自分でインストールする必要がある

```yaml
version: 0.2

env:
  variables:
    GOLANG_21_VERSION: "1.21.0"
phases:
  install:
    commands:
      - wget https://go.dev/dl/go${GOLANG_21_VERSION}.linux-arm64.tar.gz -O /tmp/golang.tar.gz
      - tar -xzf /tmp/golang.tar.gz -C /tmp
      - mv /tmp/go /usr/local/go21
      - rm -rf /usr/local/go
      - ln -s /usr/local/go21 /usr/local/go
  build:
    commands:
      - make build
      - make lint
      - make test
```

install phase は 公式 Docker イメージの [Dockerfile](https://github.com/aws/aws-codebuild-docker-images/blob/d6255f9e73e17db41c08f0980ae62c43f68eadf5/al2/aarch64/standard/3.0/Dockerfile#L257-L267) を参考にした（チェックサムの検証は省いたが、やった方が良いです...）。

## おまけ: Go 1.20 の場合

point:
- Go 1.20 & aws/codebuild/amazonlinux2-aarch64-standard:3.0 環境の場合も、Buildspec の工夫が必要。
- runtime-versions で `golang: 1.20` を設定しても、Go 1.20 を使えない
    - 理由: Go 1.20 symlink のパスが間違っているというバグがあるから
        - 問題箇所 [runtimes.yml](https://github.com/aws/aws-codebuild-docker-images/blob/d6255f9e73e17db41c08f0980ae62c43f68eadf5/al2/aarch64/standard/3.0/runtimes.yml#L25-L31)
- 解決策: Buildspec で symlink を張り直す

```yaml
version: 0.2

phases:
  install:
    commands:
      - rm -rf /usr/local/go
      - ln -s /usr/local/go20 /usr/local/go
  build:
    commands:
      - make build
      - make lint
      - make test
```

Bug を修正する PR はすでにでているので、近いうちに修正されると思います。
https://github.com/aws/aws-codebuild-docker-images/pull/655

## 参考情報

- [CodeBuild に用意されている Docker イメージ - AWS CodeBuild](https://docs.aws.amazon.com/ja_jp/codebuild/latest/userguide/build-env-ref-available.html)
- [使用可能なランタイム - AWS CodeBuild](https://docs.aws.amazon.com/ja_jp/codebuild/latest/userguide/available-runtimes.html)
- [aws/aws-codebuild-docker-images: Official AWS CodeBuild repository for managed Docker images](https://github.com/aws/aws-codebuild-docker-images/tree/master)

