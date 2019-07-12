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
<NOTE>

冒頭に、なにかフリを置く、最後に書くかな。

以下概要

あらすじ

ここでは、セキュリティ視点を中心にクラウドアプリケーションの構築を紹介する。まずは、クラウドのセキュリティ上の利点と背景を説明し、その後、高度なセキュリティ基準の例としてPCI DSSの概要を記述し。Azure上での実装としてPCI DSS Blueprint PaaS をもとに紹介する。

下記のことがどこかに書かれるはず

- クラウドでアプリケーションをホスティングすることのセキュリティ上の利点
- 他のクラウドサービスモデルに対するサービスとしてのプラットフォーム（PaaS）のセキュリティ上の利点を評価
- セキュリティをネットワーク中心からアイデンティティ中心のセキュリティアプローチに変更
- 一般的なPaaSセキュリティベストプラクティスの推奨事項を実装

</NOTE>
<hr/>

## クラウドのセキュリティ上の利点

オンプレミス環境では、ユーザーが物理層（データセンター、ハードウェア、ネットワーク）、仮想化レイヤーからアプリケーションまですべてのスタックを所有している。攻撃者はすべてのレイヤーの脆弱性を悪用することができ、ユーザー組織はリソースをセキュリティ保全に投資する必要がある。ここで重要なのは、セキュリティ投資はビジネスリスクとのバランスで決定される限られたリソースであることだ。

ユーザーは、クラウドにアプリケーションをホストすることで、クラウドベースのセキュリティ機能を利用して脅威の検出と対応にかかる時間を短縮することができ、責任をクラウドプロバイダーに移すことで、ユーザーはより広い範囲のセキュリティ保全を得ることができる。そして、責任範囲にセキュリティリソースを集中、もしくは予算を他のビジネス優先事項に割り当ることが可能となる。

![Cloud security advantages](https://docs.microsoft.com/en-us/azure/security/media/security-paas-deployments/advantages-of-cloud.png)


## Division of responsibility

クラウドプロバイダーとの責任分担を理解することが重要だ。オンプレミスではスタック全体を所有しているが、クラウドでは、一部の責任がクラウドプロバイダー(Microsoft)に移転する。どのような責任分担となるのかは、下記のresponsibility matrixを見てほしい。ユーザー(青)と、Microsoft(灰色)が担当しているSaaS、PaaS、およびIaaSの展開におけるスタックの領域を表している。

![responsibility matrix](https://docs.microsoft.com/en-us/azure/security/media/security-paas-deployments/responsibility-zones.png)

この図を下から見ていくと、物理レイヤーは、クラウドプロバイダーの責務となり、その上はクラウドプロバイダーとユーザーの共同責任、さらに上はユーザーの責任となっている。共同責任の部分は、展開モデルによって責任分担の範囲にバリエーションがあり、IaaSではユーザ責任部分が多くなり、SaaSではクラウドプロバイダーの責任部分が多くなる。限られたセキュリティーリソースの有効利用という観点では、SaaS,PaaS、IaaSの順で有効でいうことがわかる。ここでは、サービスを提供するにあたって、サービス要件の充足とユーザーの責任分担を最小化のバランスを考慮しアーキテクチャを構築する。<NOTE>コンテナについて書くべくかな</NOTE>

<NOTE>唐突なので、どこかに移動する</NOTE>
ここに、PCI DSSを例にして、Azure上でのセキュアサービスの構築を説明する。

![New PCI DSS Azure Blueprint makes compliance simpler](/images/pcidss/overview.png)

参考: [Securing PaaS deployments](https://docs.microsoft.com/en-us/azure/security/security-paas-deployments)

## PCI DSS 3.2 の概要とAzureでの実装

> PCIデータセキュリティスタンダード（PCI DSS：Payment Card Industry Data Security Standard）は、 クレジットカード情報および取り引き情報を保護するために2004年12月、JCB・American Express・Discover・マスターカード・VISAの国際ペイメントブランド5社が共同で策定した、クレジット業界におけるグローバルセキュリティ基準である。

[Wikipedia: PCIデータセキュリティスタンダード](https://ja.wikipedia.org/wiki/PCI%E3%83%87%E3%83%BC%E3%82%BF%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E3%82%B9%E3%82%BF%E3%83%B3%E3%83%80%E3%83%BC%E3%83%89)

現在は、PCI DSS 3.2.1 が最新で、セキュアなサービスを構築する上で必要なことのエッセンスが凝縮している。PCI DSSからは、セキュアなサービスと現実的なセキュリティへの取り組みのバランスを学ぶことができる。これは、PCI DSSはクレジットカードブランドが、クレジットカード番号を使った決済の手数料でビジネスをするという自分たちのビジネスモデルを守るために策定したセキュリティ基準であることから生まれた結果だ。クレジットカードでの取引がが減るほど非現実的（実装コストが高い）なセキュリティを要求すると手数料ビジネスという彼らのビジネスモデルが成立しない。また、クレジットカードの信頼性が失われるほど不正利用が増えた場合も同様にユーザーのカード利用が減ることが予想される。この２つの問題を現実的な路線で収束させるのことは、PCI DSS（カウンシル）に期待される役目であり、PCIデータセキュリティスタンダードはその成果物だ。また、このバランスは社会的な状況によって変動する。

上記のことを踏まえ、現時点でPCI DSSでなにが求められいるのかと、Azure PCI DSS Blueprint PaaSでの実装を解説する、随時元の規格（PCI DSS)を参照しながら読んで欲しい。

### 序文から（要件の前に書いてある部分）

PCI DSSには、12の要件があり、そちらに話が集中することが多いが、ここには良いことがいろいろ書いてあるので、少し紹介する。<NOTE>PCI DSSを読み直して内容を確認する</NOTE>

「ネットワークセグメンテーションは要件ではないが、ネットワークセグメンテーションを利用すると対象範囲の限定し、評価コスト、PCI DSSコントロールの実施、維持コスト、組織のリスクを低減することができる」とあり、これは多くのシステムに適応できる。PCI DSS曰く、多重防衛の１つとしてネットワークを使うのはコスト的に優れてるのでお勧めというわけ。

また、「PCI DSSの適応範囲は、PAN（カード会員番号）を扱っている部分。処理は、伝送、処理、保管を現す」。PCI DSSはカード会員番号のセキュリティ基準なのでこうなっているが、Webサイトのセキュリティを考えた場合、守るべく情報を厳密に定義し、「伝送、処理、保管」で、どのように扱うかうべきかを決めることがセキュリティー上有効である。

複数の要求レベルの異なったデータの扱いの明確化、ここでは、「カード会員データとセンシティブ認証データの２つに分けて定義」する旨が記述され、カード会員データ（カード番号、有効期限、カード会員名、サービスコード）は、業務上の必要があれば保存可、センシティブ認証データは、処理後破棄、保存不可と明記されている。

それ以外にも、「カード会員データの保存、処理または伝送に関するビジネスニーズおよびプロセスを明確にし、データフロー図を使用してカード会員データフローを文書化しろ」など。

ここで全体的に語られているのは、守るべき情報（守秘情報）を定義し、守秘情報のフロー、処理、ビジネス要件を明らかにすることの必要性です。こう考えると、もしかして必要になるかもしれないので、一緒に渡しておくとか、的な考えは守秘情報には適応できない旨を理解してシステム設計をするべきということが言えます。拡張性、柔軟性とは相反する要件とも言える。

PCI DSSには、代替コントロールという考え方がある、これは一種の回避作でビジネス要件上避けられない場合に用意された方策だ。本稿では、代替コントロールに関しては範囲外として扱わない。

このあたりは、全体的なアーキテクチャー上の思想、哲学の部分なので、Azureとはあまり関係ないと思われがちだが、この辺 [Azure Security Documentation](https://docs.microsoft.com/en-us/azure/security/)　見ると出てくる（と思う、詳細は後で）

### 安全なネットワークの構築と維持

要件1,2として、ネットワーク分離の話と、ネットワーク機材の適切な設定の要件が記述されている。

- 要件1：カード会員データを保護するために、ファイアウォールをインストールして構成を維持する
- 要件2：システムパスワードおよびその他のセキュリティパラメータにベンダ提供のデフォルト値を使用しない

Blueprintでは分離に、下記４つのテクノロジを利用している

1. VNet + Subnet + Network Security Group
2. App Service の Sandbox
3. Firewall/ Virtual Network Service Endpoints
4. Application Gateway

最初の方法では、仮想ネットワークをサブネットに分割しサブネットへのアクセスをNetwork Security Groupで制限することでネットワークをセグメント化する。セグメント化されたネットワークにコンポーネントを配置することで通信を制限し、通信を暗号化することでCDE の入出力を保護する。各サブネットには単一責務のコンポーネントのみを配置する。

2つ目の方法は、インターネット向けのフロント、内部的な管理サイト、バッチ（WebJob）の分離に使用した。これらのコンポーネントは、App ServiceのSandboxで分離され、お互いの通信は制御された方法だけに制限される。つまり、直接メモリを読んだり、相互にファイルなどのデータのやり取りをすることはできない。

3つ目は、SQL Database/Azure Storageなどのマネージド・サービスのアクセス制御で利用される。アプリケーションからSQL Databaseのアクセスは、SQL DatabaseのFirewallで特定のVNetからのアクセスのみを許可する設定としている。

また、Webアプリケーションへのアクセスは、Application Gateway（WAF）経由とし既知の脆弱性の問題を軽減する。本システムではWebアプリケーションは、専用サーバーと仮想ネットワークへの配置という観点からApp Service Environment を利用している。

Webアプリケーションの、HTTPS トラフィックは、カスタム ドメインの SSL 証明書を使用し、Azure内の通信はすべて暗号化する。Azure内で通信が傍受される可能性が低いが、ハード、ソフトともに暗号化のコストも低く導入障壁も低いことから全通信の暗号化を導入している。

Log Analytics が、広範にわたる変更のログ記録を提供することで、変更内容の確認、検証を可能とし設定の正確性を確保する。

<NOTE>
Blueprintを元に、追加のドキュメントとして、[Azure network security](https://docs.microsoft.com/en-us/azure/security/abstract-azure-network-security) を絡めて書く。

このあたりは、構成を考える。
</NOTE>

#### Application Gatewayの機能

WAF (OWASP 3.0ルールセット)に対応した Application Gateway を使用することで、セキュリティの脆弱性のリスクを軽減する。Application Gateway は、カード会員WEB用（インターネットアクセス可）と担当者画面用（ExpressRoute経由のみ）の２つがある。ぞれぞれのApplication Gatewayは、固有のApp Servicesのフロントに設定される。
Application Gatewayの他の機能には次のものがある。

1.	エンド ツー エンド SSL
2.	SSL オフロードの有効化
3.	TLS v1.0 および v1.1 の無効化
4.	Web アプリケーション ファイアウォール (WAF モード)
5.	OWASP 3.0 ルールセットを使用した検出モード
6.	診断ログの有効化
7.	カスタム正常性プローブ
8.	Azure Security Center と Azure Advisor による、保護と通知の提供。 Azure Security Center は評価システムも提供

**関連PCI DSS要件** 要件 1.3, 要件 1.3.1, 要件 1.3.2, 要件 2.2.3, 要件 4.1, 要件 6.1, 要件 6.6, 要件 付録A2

#### Network Security Groupの役割
サブネット毎にコンポーネントを分割配置し、各サブネットに専用のNetwork Security Group (NSG) を定義。通信を制限した、Blueprintでは下記の種類のNSGを定義している。

1.	ファイアウォールおよび Application Gateway WAF 用の DMZ ネットワーク セキュリティ グループ
2.	ジャンプボックス (要塞ホスト) 管理用の NSG
3.	App Service Environment 用の NSG
4.	Azure Batch用のNSG

各 NSG には、本システムの安全かつ適切な操作のために開かれる固有のポートとプロトコルを設定し、Log Analyticsにログを保存する。

**関連PCI DSS要件** 要件 1.2, 要件 1.2.1, 要件 6.3.1, 要件 6.3.2, 要件 6.4.1, 要件 6.4.2

### カード会員データの保護

要件3,4は、暗号化の話である、鍵管理と会員データの暗号化の話。鍵管理は、マネージドな鍵管理があるので、それを使う。アルゴリズム的には

- 要件3：保存されたカード会員データを保護する
- 要件4：オープンな公共ネットワーク経由でカード会員データを伝送する場合、暗号化する


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
