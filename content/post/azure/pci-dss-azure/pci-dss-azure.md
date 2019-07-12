---
title: "Azure上での PCI DSS 準拠サービスの構築"
date: 2019-07-08T00:00:00+09:00
tags: [Azure,PCIDSS]
description: "求められるサービスのセキュリティとAzure上での PCI DSS Blueprint を元にした準拠サービスの構築"
keywords:
  - "PCIDSS"
  - "Azure"
  - "Blueprint"
eyecatch: "/images/hugo/hugo.png"
draft: true
---

## 概要

<hr/>
**以下メモ**

なにかフリを置く、最後に書くかな。

以下概要

Azure上では概ね道具立ては揃っているので、PaaSなどを使うとオンプレより効率的にセキュアなサービスを構築できる。マネージドサービスを使って行くと良い。

<hr/>

クラウドのセキュリティは日々進歩しており、現時点でオンプレミスよりメリットがあるものも多い。個別の話をする前に、クラウド


## クラウドのセキュリティ上の利点

- クラウドでアプリケーションをホスティングすることのセキュリティ上の利点
- 他のクラウドサービスモデルに対するサービスとしてのプラットフォーム（PaaS）のセキュリティ上の利点を評価
- セキュリティをネットワーク中心からアイデンティティ中心のセキュリティアプローチに変更
- 一般的なPaaSセキュリティベストプラクティスの推奨事項を実装

![Cloud security advantages](https://docs.microsoft.com/en-us/azure/security/media/security-paas-deployments/advantages-of-cloud.png)

ここに、PCI DSSを例にして、Azure上でのセキュアサービスの構築を説明する。

![New PCI DSS Azure Blueprint makes compliance simpler](/images/pcidss/overview.png)

参考: [Securing PaaS deployments](https://docs.microsoft.com/en-us/azure/security/security-paas-deployments)

## 第１章 PCI DSS の概要

PCI DSS 3.2 には、セキュアなサービスを構築する上で必要なことのエッセンスが凝縮している。PCI DSSで課題となっているものと対策への要求を現実的なセキュリティへの取り組みを学ぶことができる。

もともと、PCI DSSはクレジットカードブランドが、クレジットカード番号を使った決済の手数料でビジネスをするという自分たちのビジネスモデルを守るために策定したセキュリティ基準であることから生まれた結果だ。クレジットカードでの取引がが減るほど非現実的（実装不可能）なセキュリティを要求すると手数料ビジネスという彼らのビジネスモデルが成立しない。また、クレジットカードの信頼性が失われるほど不正利用が増えた場合も同様に利用量が減ることが予想される。この２つの問題を現実的な路線で収束させるのことは、PCI DSS（カウンシル）に期待される役目であり、その基準は社会的な状況によって変動する。

上記のことを踏まえ、現時点でPCI DSSでなにが求められいるのかを解説するので、随時元の規格（PCI DSS)を参照しながら読んで欲しい。

### 序文（要件の前に書いてある部分）

ここには良いことがいろいろ書いてある。

「ネットワークセグメンテーションは要件ではないが、ネットワークセグメンテーションを利用すると対象範囲の限定し、評価コスト、PCI DSSコントロールの実施、維持コスト、組織のリスクを低減することができる」とあり、これは多くのシステムに適応できる。PCI DSS曰く、多重防衛の１つとしてネットワークを使うのはコスト的に優れてるのでお勧めというわけ。

また、「PCI DSSの適応範囲は、PAN（カード会員番号）を扱っている部分。処理は、伝送、処理、保管を現す」。PCI DSSはカード会員番号のセキュリティ基準なのでこうなっているが、Webサイトのセキュリティを考えた場合、守るべく情報を定義することが重要である。

複数の要求レベルの異なったデータの扱いの明確化、ここでは、「カード会員データとセンシティブ認証データの２つに分けて定義」する旨が記述され、カード会員データ（カード番号、有効期限、カード会員名、サービスコード）は、業務上の必要があれば保存可、センシティブ認証データ

それ以外にも、「カード会員データの保存、処理または伝送に関するビジネスニーズおよびプロセスを明確にし、データフロー図を使用してカード会員データフローを文書化しろ」など。

ここで全体的に語られているのは、守るべき情報（守秘情報）を定義し、守秘情報のフロー、処理、ビジネス要件を明らかにすることの必要性です。こう考えると、もしかして必要になるかもしれないので、一緒に渡しておくとか、的な考えは守秘情報には適応できない旨を理解してシステム設計をするべきということが言えます。拡張性、柔軟性とは相反する要件とも言えます。

このあたりは、全体的なアーキテクチャー上の思想、哲学の部分なので、Azureとはあまり関係ないと思われがちだが、この辺 [Azure Security Documentation](https://docs.microsoft.com/en-us/azure/security/)　見ると出てくる（と思う、詳細は後で）

### 安全なネットワークの構築と維持

PCI DSS 3.2 では、要件1,2として、ネットワーク分離の話と、ネットワーク機材の適切な設定の要件が記述されている。ネットワーク分離をAzureでどう実装するの話を書く。

- 要件1：カード会員データを保護するために、ファイアウォールをインストールして構成を維持する
- 要件2：システムパスワードおよびその他のセキュリティパラメータにベンダ提供のデフォルト値を使用しない

分離には、下記３つのテクノロジを利用する。

1. Subnet +Network Security Group
2. App ServiceのSandbox
3. Firewall/ Virtual Network Service Endpoints

最初の方法は、仮想ネットワークをサブネットに分割しサブネットへのアクセスをNetwork Security Groupで制限することでネットワークをセグメント化する。セグメント化されたネットワークにコンポーネントを配置することで通信を制限し、通信を暗号化することでCDE の入出力を保護する。N-1からN-7までの各サブネットでは本方式を分離に利用した。

2つ目の方法は、カード会員WEB、担当者画面、バッチ（WebJob）の分離に使用されている。これらは、各App ServiceとしてSandboxで実行される。

3つ目は、SQL Database/Azure Storageなどのマネージド・サービスのアクセス制御で利用される。N-8のSQL Databaseのアクセスは、SQL DatabaseのFirewallで特定のVNetからのアクセスのみを許可する設定としている。

また、Webアプリケーションへのアクセスは、Application Gateway（WAF）経由とし既知の脆弱性の問題を軽減する。本システムを構成するカード会員WEBと担当者画面の２つのWebアプリケーションは、専用サーバーと仮想ネットワークへの配置という観点からApp Service Environment を利用する。２つのWebアプリケーションは、App Service のPaaS isolationによって分離され、相互に直接のやりとりを禁止する。

カード会員WEBと担当者画面では、本システム外からのHTTPS トラフィックは、カスタム ドメインの SSL 証明書を使用する。Log Analytics が、広範にわたる変更のログ記録を提供することで、変更内容の確認、検証を可能とし設定の正確性を確保する。

Blueprintを元に、追加のドキュメントとして、[Azure network security](https://docs.microsoft.com/en-us/azure/security/abstract-azure-network-security) を絡めて書く。

このあたりは、構成を考える。

### カード会員データの保護

- 要件3：保存されたカード会員データを保護する
- 要件4：オープンな公共ネットワーク経由でカード会員データを伝送する場合、暗号化する

要件1,2は、暗号化の話は、鍵管理と会員データの暗号化の話。鍵管理は、マネージドな鍵管理があるので、それを使う。アルゴリズム的には

暗号化全般の導入コストが低い、特にDBの暗号化。鍵管理＝鍵のアクセス権＋監査なので、このあたりをまとめて書くかな。

### 脆弱性管理プログラムの整備

- 要件5：アンチウィルスソフトウェアまたはプログラムを使用し、定期的に更新する
- 要件6：安全性の高いシステムとアプリケーションを開発し、保守する

PaaS使って、共同管理モデルを適応する。

[Develop secure cloud applications on Azure](https://docs.microsoft.com/en-us/azure/security/abstract-develop-secure-apps) などというドキュメントがあったりするので、良さそうな部分を使わせてもらう。

※ 改竄防止には、Run From Package が良かったなと思ったり

### 強固なアクセス制御手法の導入

- 要件7：カード会員データへのアクセスを、業務上必要な範囲内に制限する
- 要件8：コンピュータにアクセスできる各ユーザに一意の ID を割り当てる
- 要件9：カード会員データへの物理アクセスを制限する

Azureにおける分離の話を書くと長くなる。[Isolation in the Azure Public Cloud](https://docs.microsoft.com/en-us/azure/security/azure-isolation)

ここでは、特定のカード会員番号だけにアクセスできる人（エンドユーザー）と不特定多数のカード会員番号にアクセスできる人を分けて扱っていることが重要。脅威は内部にもある。

### ネットワークの定期的な監視およびテスト

- 要件10：ネットワークリソースおよびカード会員データへのすべてのアクセスを追跡および監視する
- 要件11：セキュリティシステムおよびプロセスを定期的にテストする

モニターと監査、[Azure security management and monitoring overview](https://docs.microsoft.com/en-us/azure/security/security-management-and-monitoring-overview) このあたりが、良いけど。構成的にもっと前にもっていったほうが良いかも。

ここには、Shared responsibility（共同責任モデル）、Role-Based Access Control、Antimalware、Multi-Factor Authentication、ExpressRoute、Virtual network gateways とかの話が書いてある。いろいろ新しいのが出すぎてるので、どうするか検討する。

この部分は、アクセス権を適切に設定し、ログを取る（監査）の二本立ての片方（２つめ）

### 情報セキュリティポリシーの整備

- 要件12：すべての担当者の情報セキュリティポリシーを整備する

Azure Blueprintは、どうやらシステム的に対応しようとしているようだけど、まだ出来てない感じ

## 第2章 Azure PCI DSS Blueprint (PaaS) とは

Azure PCI DSS Blueprint (PaaS)では、PCI DSSの課題に対してAzure 上でどのような実装をするのかを解説している。ここでは、PCI DSSの課題に対して、Blueprintはどのように扱っているのかを解説する。
（ここは全部流すとながくなりそう）
PANのデータフローとか

## 参考

参考リンク

### Azure Security and Compliance Blueprint

Azure Security DocumentationのPCI DSSの下は４つに別れてます

- [Data analytics/Analytics for PCI DSS](https://docs.microsoft.com/en-us/azure/security/blueprints/pcidss-analytics-overview)
- [Data warehouse/Data Warehouse for PCI DSS](https://docs.microsoft.com/en-us/azure/security/blueprints/pcidss-dw-overview)
- [IaaS web application/IaaS Web Application for PCI DSS](https://docs.microsoft.com/en-us/azure/security/blueprints/pcidss-iaaswa-overview)
- [PaaS web application/PaaS Web Application for PCI DSS](https://docs.microsoft.com/en-us/azure/security/blueprints/pcidss-paaswa-overview)

## まとまったドキュメント(PDF)の場所

[Azure セキュリティおよびコンプライアンス PCI DSS Blueprint](https://servicetrust.microsoft.com/ViewPage/PCIBlueprint)

## 2019/6/27 の PCI DSS Azure Blueprint のBlog

2019/6/27に、Azure Blueprint(preview) に、PCI-DSS v3.2.1 blueprint を追加するというアナウンスがありました。[New PCI DSS Azure Blueprint makes compliance simpler](https://azure.microsoft.com/en-us/blog/new-pci-dss-azure-blueprint-makes-compliance-simpler/)

![New PCI DSS Azure Blueprint makes compliance simpler](/images/pcidss/newpcidssbp01.png)

以下Blogからの概要です

Azure Blueprintsの入った、PCI-DSS v3.2.1のBlueprints では下記のPCI DSS コントロールへのマッピングが含まれている。

- 職務の分離: 購読所有者の権限を管理
- ネットワークおよびネットワークサービスへのアクセス: Azureリソースにアクセスできるユーザーを管理するための役割ベースのアクセス制御（RBAC）の実装
- ユーザの秘密認証情報の管理: 多要素認証が有効になっていないアカウントを監査
- ユーザーアクセス権の確認: レビュー用に優先順位を付ける必要があるアカウント（監査対象アカウント、昇格された権限を持つ外部アカウントなど）
- アクセス権の削除または調整 購読に対する所有者権限を持つ非推奨アカウントの監査
- 安全なログオン手順 多要素認証が有効になっていないアカウントの監査
- パスワード管理システム: 強力なパスワードの強制
- 暗号制御の使用に関する方針: 特定の暗号制御を実施し、弱い暗号設定の使用の監査
- イベントとオペレータのロギング: Diagnosticログは、Azureリソース内で実行された操作についてのinsightを提供
- 管理者とオペレータのログ: システムイベントが記録されていることの確認
- 技術的な脆弱性の管理: 不足しているシステムアップデート、オペレーティングシステムの脆弱性、SQLの脆弱性、およびAzure Security Centerの仮想マシンの脆弱性の監視
- ネットワーク制御: ネットワーク管理、制御、ネットワークセキュリティグループでの寛容なルールの使用を監視
- 情報伝達の方針と手順: Azureサービスとの情報転送が安全であることの確認

従来運用に任せた部分をできるだけ自動化しようという方向性を感じます。まだpreviewなので動向を見守りつつ今後に期待ですね。

## その他

- [What Does Shared Responsibility in the Cloud Mean?](https://blogs.msdn.microsoft.com/azuresecurity/2016/04/18/what-does-shared-responsibility-in-the-cloud-mean/)
- [Use the Microsoft Graph Security API](https://docs.microsoft.com/en-us/graph/api/resources/security-api-overview?view=graph-rest-beta)

