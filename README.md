# Zenn MyPage
Check to https://zenn.dev/nyarufoy

# Zenn CLI
* [📘 How to use](https://zenn.dev/zenn/articles/zenn-cli-guide)# zenn-content

### 記事の作成

```shell
$ npx zenn new:article
```

### ファイル構成

```yaml
---
title: "" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: [] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
ここから本文を書く
```

### プレビュー

```shell
$ npx zenn preview 
```