---
title: "Hugoを使いこなす2＝Markdownでの編集"
date: 2019-05-01T18:53:17+09:00
tags: [Hugo,Azure]
description: "Hugoで記事を投稿するのはMarkdownで簡単にできる。文法は、Markdownのチートシート等を参考に。以下に一通り必要な編集例を記載する。"
keywords:
  - "Hugo"
  - "Markdown"
eyecatch: "/images/hugo/hugo.png"
---

## Markdownでの編集

Hugoで記事を投稿するのはMarkdownで簡単にできる。文法は、[Markdownのチートシート](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)等を参考に。以下に一通り必要な編集例を記載する。

### 見出し

見出しは # で記載する。

```markdown

# h1
## h2
### h3
#### h4
##### h5

```

### リスト

リストは - で記載する。

```markdown

- IPA
- PaleAle
- Lager

```

### リンク

リンクは、[]()で記載する。

```markdown

[Syntax Highlighting](https://gohugo.io/content-management/syntax-highlighting/)

```

### ソースコードの貼り付けとSyntax Highlightning

プログラム部分の貼り付けを行い、構造を色で区別(syntax highlighting)するには
[Syntax Highlighting](https://gohugo.io/content-management/syntax-highlighting/)
を参照のこと。

- 通常のソースコードは```で囲む。


- shortCode(後述) を含むソースコードを表示する場合は以下のようにshortCode内をコメントアウトする記法になる。

```html
{{</*/* instagram BWNjjyYFxVx */*/>}}
```

と記載する。出力される結果は、

```html
{{</* instagram BWNjjyYFxVx */>}}
```

となる。


### Tableを作る

Tableを作る場合は、Markdownで

```markdown

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

```

と記載する。出力される結果は、

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

となる。


### 引用する

引用する場合は、

```markdown

> あいうえおかきくけこ

> さしすせそたちつてと

```

と記載する。出力される結果は、

> あいうえおかきくけこ

> さしすせそたちつてと

となる。


### 罫線を引く

罫線を引く場合は、

```markdown

---

***
___

```

と記載する。出力される結果は、

---

***
___

となる。


## Markdownの中でShortCodeを使いこなす

Hugoにはshortcodeという機能があり、画像やTwitterへのリンクを張る際は、shortcodeを使いこなした方が、Markdownで重工に記載するより簡単に済む。


### 画像を張る＝shortCode「figure 」


```html
{{</* figure src="/images/iceland/iceland-car2.png" alt="Golden Circle はレイキャビクから北東に2時間ほどで日帰りコース" height="640" link="https://www.google.com" target="_blank" */>}}
```

と記載する。出力される結果は、

{{< figure src="/images/iceland/iceland-car2.png" alt="Golden Circle はレイキャビクから北東に2時間ほどで日帰りコース" height="640" link="https://www.google.com" target="_blank" >}}

となる。


### Youtubeを張る

```html
{{</* youtube w7Ft2ymGmfc */>}}
```

と記載する。出力される結果は、

{{< youtube w7Ft2ymGmfc >}}

となる。


### Instagramを張る

```html
{{</* instagram BWNjjyYFxVx */>}}
```

と記載する。出力される結果は、

{{< instagram BWNjjyYFxVx >}}

となる。


### Twitterを張る

```html
{{</* tweet 1020983858961965056 */>}}
```

と記載する。出力される結果は、

{{< tweet 1020983858961965056 >}}

となる。


### gistを張る

```html
{{</* gist spf13 7896402 */>}}
```

{{< gist spf13 7896402 >}}



## 設定周り

### aタグのリンクをtarget="_blank"にする

こちらはconfigの設定となる。

```toml config.toml
[blackfriday]
  hrefTargetBlank = true
```


### リファレンス

[Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
