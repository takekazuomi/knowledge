---
title: "Azure Storage + Azure CDN で静的Webホスティング(Hugoから作成した静的なWEBサイトをデプロイ)"
date: 2019-05-10T16:53:17+09:00
tags: [Hugo,Azure]
description: "Hugoで作成した静的なWebサイトは、Azure上のカスタムドメインかつ無償の証明書付きでホスティングすることが可能。ストレージとアクセスの従量課金をふまえても月額100円未満。なんて便利で素敵な世の中になったのだ。"
keywords:
  - "Hugo"
  - "Azure"
  - "Azure Storage"
  - "Azure CDN"
eyecatch: "/images/hugo/hugo-on-azure.png"
---

## Azure Storage + Azure CDN で静的Webホスティング

Hugoで作成した静的なWebサイトは、Azure上のカスタムドメインかつ無償の証明書付きでホスティングすることが可能。
ストレージとアクセスの従量課金をふまえても月額100円未満。なんて便利で素敵な世の中になったのだ。
支払いを自動化できれば死後も永遠に残るサイトが作れる。

大枠の手順としては

- Azureのサブスクリプションを契約。
- Storage アカウントを作成。
- Storage にHugoで作成したコンテンツをデプロイする。
- CDNを作成。
- CDNにカスタムドメインを設定し、証明書をAzure側から無償提供されるように設定。

となる。


### 前提条件

- [こちらの記事](/howtohugo1/)で、Hugoの開発環境構築が済んでいる。
- publicフォルダ配下にhugo コマンドで静的なHTMLファイルを出力済み。

### Azureを使うメリット

- 高速なCDN配信サイトでもコストが格安。
- ストレージの料金＝1GB=10円(月)＋CDN経由での配信＝1GB=20円。
- VSCodeから、サイトのデプロイが可能。
- PaaS,Serverless,Azure ML,など周辺サービスと組み合わせてもGood。


### Microsoft Azureのアカウントを作る

[こちらから、](https://azure.microsoft.com/ja-jp/free/search/)Microsoft Azureのサブスクリプションを作成する。(15000円分の無料プランがついてくる)

![Azure アカウントを作成する](/images/azure/start-azure.png)

- Microsoft アカウントを作成する。

![Microsoft アカウントを作成](/images/azure/create-msaccount.png)

- Azureアカウント(サブスクリプション)を作成完了する。


### Microsoft Azure上にStorageを作成し、静的Webホスティング環境を構築する

- ストレージv2を作成する。確認及び作成をクリック。

![Azure Storagev2を作成](/images/azure/create-storage.png)


- ストレージv2の静的Webサイトを有効にする。

![Azure Storage で静的Webホスティングを有効にする](/images/azure/enable-staticwebsite.png)

- $Web コンテナにHTMLファイルを置くと、プライマリエンドポイントから「静的Webなサイト」にアクセス可能になる。(index.htmlはデフォルトドキュメントに指定したので省略可能で / でディレクトリ表示できる。)　

![$Web コンテナ](/images/azure/dollar-web.png)


- $Webコンテナは以下にデプロイした静的WebなサイトのURLは https://sigmanyan.z99.web.core.windows.net/ のようになる。


### ストレージの内容を高速配信するため Azure CDN を作成する。

- Azure CDN を作成する。CDNは複数から選べるのだが、ここでは「標準Microsoft」を選ぶ。

![Azure CDNを作成する](/images/azure/create-cdn.png)



### カスタムドメインで証明書無償で利用する

- Azure CDNにはカスタムドメインが設定可能。
- CNAMEでカスタムドメインとAzure CDN のURLを設定する。

```txt

cname www.sigmanyan.com.  sigmanyan.azureedge.net.

```

- Azure CDN でカスタムドメインを設定する。
- HTTPSを有効にする。今回はCDNマネージドで設定する。

![Azure CDNでHTTPSを有効にする](/images/azure/azure-cdn.png))

- 事前にCNAMEレコードを設定しておけば、数時間後にはドメインが認証され、証明書がデプロイされている。
- これで証明書は今後、Azure側が自動更新してくれる。非常に便利。



### VSCodeからAzure Storage(静的サイトホスティング)にデプロイする

- VSCode に storage プラグインをインストールしておく。

![VSCode でAzure Storage用エクステンションをインストールする](/images/azure/install-extension-azurestorage.png)

- storage プラグインを使うと　＋　からVSCode上から、ストレージに静的ファイル一式をデプロイすることができる。
- VSCode でリリース先のAzure Storageを選択する。

![VSCode でリリース先のAzure Storageを選択する](/images/azure/vscode-selectstorage.png)
![VSCode でリリース先のAzure Storageを選択する](/images/azure/vscode-selectstorage2.png)

- VSCode からリリース先のAzure Stotage コンテナを選択する。
![VSCode からリリース先のAzure Stotage コンテナを選択する](/images/azure/vscode-selectcontainer.png)
![VSCode からリリース先のAzure Stotage コンテナは以下のディレクトリを選択する](/images/azure/vscode-selectproductiondirectory.png)

- VSCode でリリース対象のローカルディレクトリ(public/production フォルダ配下)を選択する。

![VSCode でリリース対象のローカルディレクトリを選択する](/images/azure/vscode-selectpublicdirectory.png)


- $Webフォルダのコンテンツを削除して、リリースして良いか確認される。
![VSCode から静的Webホスティングのコンテンツを削除し、リリースする](/images/azure/vscode-deleteanddeploy.png)
![VSCode から静的Webホスティングのコンテンツを削除し、リリースする](/images/azure/vscode-deleteanddeploy2.png)

- VSCode から静的Webホスティングのリリースが完了する。
![VSCode から静的Webホスティングのリリースが完了する](/images/azure/vscode-deployfinish.png)


### VSCode以外のデプロイ方法だとStorage Explorerもある

[Storage Explorer](https://azure.microsoft.com/ja-jp/features/storage-explorer/)経由でもデプロイは可能。

- Storage Explorerだとディレクトリ表示できる。
- FTPライクなインターフェースになれていたり、特定のファイルだけという場合はこちらの方が使いやすい。

![VSCode以外のデプロイ方法だとStorage Explorerもある](/images/azure/storage-explorer.png)


### Azure Portal上からCDNのキャッシュをクリアする。

- Azure Portal 上のCDNから、キャッシュをクリアする。

![Azure CDNのキャッシュをクリアする](/images/azure/delete-azurecdncache.png)

- 数分おいてアクセスすると、キャッシュがクリアされ、最新のデプロイが反映されている。
- デプロイが反映されていない場合は、そもそもpublic\productionフォルダに最新のHTMLが出力されているか確認する。
- hugo コマンドを打ち忘れて、public\productionフォルダに最新のHTMLを出力し忘れていることが多い。


### リファレンス

- [Azure Storage での静的 Web サイト ホスティング](https://docs.microsoft.com/ja-jp/azure/storage/blobs/storage-blob-static-website)


- [チュートリアル:Azure CDN カスタム ドメインで HTTPS を構成する](https://docs.microsoft.com/ja-jp/azure/cdn/cdn-custom-ssl?tabs=option-1-default-enable-https-with-a-cdn-managed-certificate)
