---
title: "Hugoの面白さを簡潔にまとめる"
date: 2019-04-28T16:53:17+09:00
tags: [Hugo,Azure]
description: "記事の概要です。"
keywords:
  - "Hugo"
  - "Azure"
  - "Azure Storage"
eyecatch: "/images/hugo/hugo-on-azure.png"
---

## 静的CMS Hugo

![Huog](/images/hugo/hugo.png)

CMSの振り返りから、静的CMSに興味を持つところまでを前回書いた。
今回はその中から、チョイスしたHugoの面白さを簡潔にまとめる。

### Hugo とは

![GoHugo](/images/hugo/gohugo.png)

- 静的CMS。テンプレートとMarkdownで記載した記事をもとにHTML等の静的コンテンツを生成するタイプのCMS。
- テンプレートの実装を学べば、大概のことができる。
- サイトはこちら。 https://gohugo.io/
- 開発が活発に進んでいる。今のバージョンは0.56。
- コミュニティも活発。　https://discourse.gohugo.io/
- Gitでの管理に適していてGit経由でデプロイする。DevOps的なものも組みやすい。
- NetlifyやFirebaseなど、CDNを有する高速配信と相性が良い。
- Azure CDNと組み合わせても使いやすい。


### すぐ動く

Hugoの良さの一つ。Windows環境だとローカルに.exeをダウンロードして、環境変数を通せば実行可能となる。

### わかりやすいアーキテクチャ

- ディレクトリ構成
- Markdownからページ生成
- テンプレート実装(html,Front Matter)
- ホスティング先との連携

この辺をマスターすればすぐサイトがホスティングできる。

### ShortCodeが楽しい

- 1行でTwitterやInstagramを呼び出せる

```html
{{</* instagram BWNjjyYFxVx */>}}
```

```html
{{</* tweet 1020983858961965056 */>}}
```



### カスタマイズが容易

- HTMLのテンプレートはカスタマイズが容易
- Front Matter を使いこなす


### 静的ホスティングのメリットを享受

パフォーマンス、セキュリティ等非機能要件の大半をクリアできる。
CDNと組み合わせればサイトがダウンすることもない。


### テーマ(テンプレート)が豊富

テーマ(テンプレート)が豊富。500くらいある。このサイトも[Cayman](https://github.com/zwbetz-gh/cayman-hugo-theme#credits)というテーマをカスタマイズして作っている。

https://themes.gohugo.io/

![Hugo Theme](/images/hugo/hugo-theme.png)


### Azureの静的サイトホスティングと相性が良い

- [こちらにAzureでのホスティング方法を記載](/hugo-on-azure)したが、相性が非常に良い。
- 同一のURLで複数パターンレンダリングする必要がなく、生成する必要があるページ数が千くらいであればHugoで問題なく使えるだろう。


それでは、環境設定に移る。


### コミュニティ

https://discourse.gohugo.io/ から質問が可能。やり取りも活発。

![Hugo Forum](/images/hugo/hugo-forum.png)
