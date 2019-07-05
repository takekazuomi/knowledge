---
title: "Hugoを使いこなす1＝Windows10上にHugo開発環境を構築する"
date: 2019-04-30T16:53:17+09:00
tags: [Hugo,Azure]
description: "Windows10 上にHugo の開発環境を構築する。exeをダウンロードして、テンプレートを選べば即動くのがHugoの楽しいところ。"
keywords:
  - "Hugo"
  - "開発環境構築"
  - "Windows10"
eyecatch: "/images/hugo/hugo.png"
---

## 環境構築

Windows10 上にHugo の開発環境を構築する。exeをダウンロードして、テンプレートを選べば即動くのがHugoの楽しいところ。

### 前提条件

- Windows10でGitを使える環境にしておく。
- [Visual Studio Code(VSCode)](https://code.visualstudio.com/)をインストールしておく。


### Hugo（Windows版）のダウンロード

- 以下のディレクトリからHugoをダウンロードする。
- 今回はWindows版64bitをダウンロードし、解凍する。

https://github.com/gohugoio/hugo/releases


### ディレクトリの作成

- C:\Hugo フォルダを作成。
- 解凍した「hugo_バージョン名_Windows-64bit」フォルダをbinという名前に変えて、C:\Hugo\binに配置。

![binフォルダ配下](/images/hugo/setting-bin.png)


- C:\Hugo\sitesに「sites」フォルダを作成。ここではサイト名を「sigmanyan」とした。

![Hugoフォルダ配下](/images/hugo/setting01.png)


### 環境変数の設定

- コマンドプロンプト上でHugoコマンドを使えるようにするために、環境変数を設定する。
- Windows 10 の検索ウィンドウに「環境変数」と入力し、「システム環境変数の編集」を起動する。
- PathにC:\Hugo\bin を入力する

![環境変数の設定](/images/hugo/setting02.png)


### テーマをダウンロードし、設定する

- https://themes.gohugo.io/  から好きなテーマを探し、GitHubのパスを控える。
- 階層下のthemaのフォルダへ移動し、Git経由でテーマをダウンロードする。

```command
cd C:\Hugo\sites\themes 
git clone https://github.com/zwbetz-gh/cayman-hugo-theme.git themes/cayman-hugo-theme
```

- themeの中身は、丸々一つ上のディレクトリにもコピーし、編集を開始する。


### フォルダ構成を確認する

- 前の手順でthemesをGitから取得し、かつ一つ上のディレクトリにコピーを展開すると、Hugoは以下の通りのフォルダ構成となっている。それぞれ重要な役割があるので覚えておく。
- themeフォルダから一つ上の階層にコピーすると他のフォルダも作成されている場合がある。

```txt
├── archetypes
├── assets
├── config
├── content
├── data
├── layouts
├── static
└── themes
```

- archetypes=新規コンテンツをMarkdownで作る時に利用されるテンプレート。記事のタイトルやタグ等のメタ情報やユーザー定義情報を事前定義する。
- assets=Sass等を格納し、静的コンテンツ生成時にパイプライン経由でコンパイルする。
- config=「_default」「production」 のフォルダを作り、それぞれにConfig関連のファイル(config.toml)を配置して、環境を分ける。
- content=Markdown形式の記事ファイルをを格納する。階層構造にしてもよい。
- data=データを動的に使う場合、格納する(今回の例では使用していない)。
- layouts=レイアウトを格納する。themesで指定したテーマを上書きするものについて配置する。※直接themesフォルダはいじらないようにする。
- static=静的コンテンツファイルを格納する。imagesやpdfフォルダを作ってコンテンツ運用すればサイトの直下にコピーされる。
- themes=Gitからダウンロードしたテーマ。ダウンロード元は直接編集をしないため、こちらに格納しておく。


### 静的ファイルの使い方

- staticフォルダ配下にimagesフォルダやpdfフォルダを作り、画像ファイルやPDFファイルを配置する。
- ここに配置したファイルは　hostname/images/yourimage.jpg というようにルートに配置される。


### 記事を作成し、表示する。

- 今回は、contents/post配下に記事を作成していく。
- ただし、URLは http://localhost/testarticle1/  というように直下にコンテンツを作成していく。

```command
hugo new post/test_category/testarticle1.md
```

http://localhost:1313/testarticle1/ でコンテンツが表示される。


### Configの設定

- configフォルダ配下に　_default と production フォルダを用意する。
- それぞれの配下に config.toml を作成する。
- config.toml 内でpublishDir の値を _default production でそれぞれ設定する。
- 各configでbaseURLをそれぞれ設定しておき環境を使い分ける。

```toml
publishDir = "public/dev"
publishDir = "public/production"

baseURL = "http://localhost:1313"
baseURL = "https://www.sigmanyan.com"

```


### サーバーを起動し、デバッグする

- server コマンドで開発用のサーバーを起動し、テンプレートのビルドと記事の作成を行う。
- server -D コマンドだと、Draft状態の記事も含めてビルドする。

```command
hugo server -D
```

[-D, buildDrafts include content marked as draft](https://gohugo.io/commands/hugo/#options)

ドラフト状態のドキュメントでサーバーが起動する。

{{< figure src="/images/hugo/server-d.png" alt="server-d"  >}}




### 試しに編集してみる

- .mdを編集すると http://loacalhost:1313/testarticle1/ 上の内容もリアルタイム修正される。
- layouts配下を修正しても http://loacalhost:1313/testarticle1/ 上でリアルタイム修正される。
- public配下にデバッグ環境を生成している場合は、serverコマンドを使って再度renderToDiskしないと再生成されない模様。(物理ファイルがあってもなくてもデバッグできる。)


### dev環境とproduction環境のファイル一式を生成する

hugoは静的CMSなので、HTMLファイルを生成し、それをデプロイする必要がある。
※もしくはGitと連携し、Gitからデプロイする。

- public/production フォルダ配下にproduction環境のファイル生成する。こちらをリリースに使う。

```command
hugo 
```

{{< figure src="/images/hugo/public-production.png" alt="production環境のファイル生成する。"  >}}


- public/dev フォルダ配下以下にdev環境のファイル生成する。(こちらは使うケースが少ない。)

```command
hugo server --renderToDisk
```




### リファレンス

- [Hugoコマンドのリファレンス](https://gohugo.io/commands/)

- [Template Cayman](https://github.com/zwbetz-gh/cayman-hugo-theme#credits)

