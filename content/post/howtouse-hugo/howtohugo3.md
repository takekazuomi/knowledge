---
title: "Hugoを使いこなす3=テンプレートのカスタマイズ"
date: 2019-05-02T21:53:17+09:00
tags: [Hugo,Azure]
description: "Markdownでの投稿になれたところで、デザインやHTML等のカスタマイズに入っていく。テンプレートとFront Matterに関する知識が必要になる。"
keywords:
  - "Hugo"
  - "Hugoテンプレート"
  - "FrontMatter"
eyecatch: "/images/hugo/hugo.png"
---


## テンプレートのカスタマイズ

[Markdownでの投稿になれたところ](/howtohugo2/)で、デザインやHTML等のカスタマイズに入っていく。
例えば、以下のような改修をする場合、実装するには、テンプレートとFront Matterに関する知識が必要になる。

- SNSのシェアボタンを置きたい。
- OGPを設定したい。
- KeyWordとDescriptionを設定可能にしたい。
- 記事一覧部分のデザインを変更したい。


### 基本的なルール

- 先述の通り、themesフォルダを直接編集はしない。
- テンプレートはthemesから拡張したいものをlayoutsフォルダに置いて編集する。
- 全てのページのbaseにあたるのが、baseof.html で、コードを見ると構成がわかりやすい。
- head.htmlでメタ、header-nav.htmlでヘッダ部分、mainが記事本体で、footer.htmlがフッター。
- share.html　でSNSへのシェア機能を呼び出し。(こちらの実装は後述)
- footerの後に、google anayticsの呼び出し。

```html
<!DOCTYPE html>
<html lang="{{ .Site.Language.Lang }}">
{{ partial "head.html" . }}
<body>
  {{ partial "header-nav.html" . }}

  <section class="main-content">
    {{ block "main" . }}{{ end }}
    {{ partial "share.html" . }}
    
    {{ partial "footer.html" . }}
  </section>

  {{ template "_internal/google_analytics_async.html" . }}

</body>
</html>
```


### SNSボタンを実装する

- CSSの設定(Font awesomeとcustom.css)
- layouts/partials/head.html に以下のCSSを設定する。
WebフォントからSNSアイコンの見た目を作る。

```html

<!-- snsicon -->
<link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.0.6/css/all.css">

```

- custom.cssを作成し、static/cssフォルダ配下に置く。

```html

<!-- custom css -->
<link rel="stylesheet" href="{{ "css/custom.css" | absURL }}">

```

- partial/share.html の作成

```html

{{ if ne .Params.share false}}
<!-- use font awsome -->
<section class="section sns_parent">
  <div class="container sns_section">
      <div class="sns_button twitter">
        <a href="http://twitter.com/intent/tweet?url={{ .Permalink }}&text={{ .Title }}" target="_blank" title="Tweet"><i class="fab fa-twitter"></i></a>
      </div>
      <div class="sns_button hatena">
        <a href="http://b.hatena.ne.jp/add?mode=confirm&url={{ .Permalink }}&title={{ .Title }}" target="_blank" title="hatena"><i class="fa fa-hatena"></i></i></a>
      </div>
      <div class="sns_button facebook">
        <a href="http://www.facebook.com/sharer.php?u={{ .Permalink }}&t={{ .Title }}" target="_blank" title="Facebook"><i class="fab fa-facebook"></i></a>
      </div>
      <div class="sns_button pocket">
        <a href="http://getpocket.com/edit?url={{ .Permalink }}&title={{ .Title }}" target="_blank" title="pocket"><i class="fab fa-get-pocket"></i></a>
      </div>
  </div>
</section>
{{ end }}


```

- share.htmlをbaseof.htmlから呼び出す。baseof.htmlのソースコードは↑に記載。


### ogp用のテンプレートを作成する。

TwiiterCardとOGPに対応するテンプレート、ogp.htmlを作成する。


```html

<!-- Twitter Card data -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@k1hash" />
<!-- The Open Graph protocol data -->
<meta property="og:url" content="{{ $.Permalink }}"/>
<meta property="og:type" content="article"/>
<meta property="og:title" content="{{ .Title }}"/>
<meta property="og:description" content="{{ .Description }}"/>
<meta property="og:image" content="{{ $.Site.Params.hostname }}{{ .Params.eyecatch }}" />

```

- head.html内で ogp.htmlを呼び出す。

```html

<!-- ogp -->
{{ partial "ogp.html" . }}

```


### KeywordとDescriptionを設定する

- この辺からFront MatterやPage Valiablesの知識が必要になる。
- Front MatterはMarkdownの記事上部にソースコードや変数を記載するためのエリア。
- Page Valiablesはページ内で利用することが可能な予約語もしくはユーザー独自の変数を使うことができる。
- KeywordとDescriptionは *Front Matter* に記載されているものを利用するようにする。
- KeywordはSEO的にはもう使われていないそうなので、実際には使わない予定。
- Keywordは配列を分割するように実装すると以下のようになる。

MarkdownのFront MatterにKeywordとDescriptionを入稿する。Keywordは配列形式で記載する必要がある。

```toml
---
title: "Hugo環境を作る(Windows 10)"
date: 2019-04-30T16:53:17+09:00
tags: [Hugo,Azure]
description: "記事の概要です。"
keywords:
  - "word1"
  - "word2"
  - "word3"
---
```

head.htmlに以下の通りに記載する。

```html

<meta name="description" content="{{ .Description }}">
<meta name="keywords" content="{{ delimit .Keywords ", " }}">

```



### URLコントロール

- 該当ドキュメントのFront Matterで　aliases: [/posts/my-old-url/]　と指定すると、旧URL(リダイレクト元)を設定でき、このドキュメントにリダイレクトしてくれる

- Front Matter でurl を指定すると任意のURLにすることができる



### リファレンス

- [Hugoで作ったサイトにシェアボタンを足した](https://aakira.app/blog/2018/08/share/)

- [Hugo のテンプレートで <meta> の keywords が配列になってしまうので調べました](https://va2577.github.io/post/133/)



## まだ解決していないこと


### 記事と同じ階層にある画像をpublicディレクトリにコピーする方法がわからない

おそらくfigureを使うと自動でコピーされる？


### Disqusの出し方がわからない

本番でもエラーが表示されている。
