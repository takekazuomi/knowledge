---
title: "PCI DSS Azure PaaS リライト版"
linkTitle: "PCI DSS"
weight: 10
description: >
  PCI DSS Azure 書き直し版、前のをマージする予定。
---

## はじめに

本稿は、PCI DSS準拠の某クレジットカードイシュアのカード会員システムをMicrosoft Azure 上でPaaSで作った際の知見に基づいています。情報セキュリティと言っても範囲は広いので、ここではテクノロジーカットにフォーカスします。それでも、完全には運用やポリシーを外せないので、PCI DSS要件をユースケース的に利用したドキュメントしています。ここで、紹介する技術要素は PCI DSS に限らない広いユースケースで適用可能です。

最初に２つだけ前提となる基本に簡単に触れます。

1つ目が、[情報セキュリティの定義](https://ja.wikipedia.org/wiki/%E6%83%85%E5%A0%B1%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3)、時と共に変遷はありますが、大本は変わらず、これは、頭文字を取ってCIAとして知られています。

- 機密性 (confidentiality): 情報へのアクセスを認められた者だけが、アクセスできる状態とする
- 完全性 (integrity): 情報が破壊、改ざん又は消去されていない状態を確保する
- 可用性 (availability): 情報へのアクセスを認められた者が、必要時に中断することなく、情報及び関連資産にアクセスできる

この３つを軸を、Azure 上でどのように扱ったのかを説明します。

{{< figure src="images/01-cia.png" title="図1 JIS Q 27002(ISO/IEC 27002)" width="400" >}}

2つ目が、[共有責任モデル(Shared responsibility model)](https://docs.microsoft.com/ja-jp/azure/security/fundamentals/infrastructure#shared-responsibility-model)

これは、Microsoft（クラウドプロバイダー） と顧客の間で責任は共有され、それぞれ自己の責任部分がありますよという話です。分担は、主に[サービスモデル](https://en.wikipedia.org/wiki/Cloud_computing#Service_models)によって相互比重が異なります。原則的には、物理は Microsoft の責任で、アプリケーションは顧客の責任、其の間の部分は、IaaSだと顧客責任が多くなり、PaaS/SaaSだと顧客責任が少なくなります。

我々は、限られた顧客のリソースを最大限に活かすためには、PaaS/SaaSを利用しクラウドに最適化（＝クラウドネイティブ）した設計を取ることが重要だと考えています。そのため、PCI DSS の実装も最大限にAzureに最適化した設計としています。

{{< figure src="images/02-srmodel.png" title="図2 共有責任モデル(Shared responsibility model)" width="400" >}}


## Blueprint: PaaS Web Application for PCI DSSとは

Microsoftから、[PCI DSS のための PaaS Web アプリケーション](https://docs.microsoft.com/ja-jp/azure/security/blueprints/pcidss-paaswa-overview)というドキュメントが公開されていて、それを参考にしてサイトを構築しています。

{{< figure src="images/03-arch.png" title="図3 Azure PCI DSS アークテクチャー" width="600" >}}

この構成を情報セキュリティ (CIA) 対策の方針に当てはめると、下記のようなAzureのテクノロジーを使っていると言えるでしょう。

1. 認証、アクセス制御
  Azure Active Directory、RBAC, SQL Database アクセス制御等
2. 不正アクセス防止
  ネットワーク分離(VNet, NSG, ExpressRoute), セキュリティパッチ適応(App Service)
3. 監査、ログ、監視
  Azure Monitor
4. 暗号化 ー鍵管理
  Azure Key Vault

この順で概要を説明して行きます。

## 1. 認証、アクセス制御

Azure でオンプレと異なっていて、もっとも重要な点は。セキュリティの基本が、IDベースのセキュリティ[Id Based Security、Zero Trust](https://www.microsoft.com/security/blog/2018/12/17/zero-trust-part-1-identity-and-access-management/)となっていることです。Azure 自体が、Id Based Security であることだけでは無く、アプリケーションアーキテクチャ全体を、Id Based Securityで保護します。Zero Trust は、内部ネットワークといえども信頼せず、まず認証し事前に定義されたアクセスだけを許可するという考えです。本システムは、外向きだけでは無く企業ネットワーク側にも同一の方針を取り、全てを認証、アクセス制御で保護します。

Azure上にサービスを展開すると下図のような構成になります。一番下が、Azureの基盤層、其の上で、Azureのリソース層乗り、更に上にユーザーが作成したアプリケーションが乗ります。俗にインフラと言われる部分はが下２層、その上はアプリケーションです。

{{< figure src="images/05-3layer.png" title="図4 デプロイの3層構造" width="600" >}}

Azure リソースの操作は、Azure ADで認証されたオペレータが、Azure Resource Manager 経由で行います。操作は、RBACに基づいて可否を判定され、操作自体履歴がActivity Logとして保存されます。リソースの操作権限をRBACで定義しておくことで、アクセスを制御します。さらに、操作はAPIが提供されており、スクリプト化が出来ます。

{{< figure src="images/06-aadrbac.png" title="図5 AAD/RBAC/Resources" width="600">}}

### 踏み台サーバー

企業ネットワークからのアクセスの管理として運用管理用の踏み台サーバーを置きます。運用操作は必ず踏み台サーバーを経由することで、運用管理者の認証とアクセスを制御を実現しています。必要な場合は、踏み台サーバから、データーベースに接続することができます。其の場合も、オペレータの権限で、参照可能なテーブルやカラムを制限するなどが可能です。これは、運用要件応じて決めます。この場合に置いても、SQL Databaseの監査を併用することで、広いカバレッジの監査を実装することができます。

{{< figure src="images/07-bastin.png" title="図6 踏み台サーバー" width="600">}}

### まとめ

- リソースの構成、設定は、AAD認証下で行われる
- AAD認証と連動してアクセス制御（RBAC)される
- リソース操作はコードで実施できる

この３つを使うと、PCI DSS準拠の、Infrastructure as Code が実装でき、PCI DSSで必要な下記の項目の運用困難性が著しく下がります。

- 不正な構成変更の監査
- 構成基準の明確化
- 定期な構成確認の現実的な実行

さらに、Azure リソースには固定デフォルトパスワードは無いので、PCI DSS 要件2のデフォルトパスワード使用禁止も該当しません。

まさに、PCI DSSのために準備されたかのような環境と言えます。

### 課題

レガシーなシステムとの連携に置いて、 認証、アクセス制御が適応できない場合があります。その場合は、代替策として中継サーバーの設置、IPアドレス固定などの方策を取りますが、驚異分析、多重防御の原則が守られるように十分注意が必要です。

## 2. 不正アクセス防止

Azure の機能をうまく使ったアーキテクチャとするとこで多くの不正アクセスを防止することができる。

本システムでは、ネットワーク分離に、仮想ネットワーク、NSG、ExpressRouteを使い。既知の脆弱性からの防御には、Application Gateway+ WAF、セキュリティパッチの自動化（PaaS）を使うなどの手段を講じている。以下に

### Unauthorized Access (不正アクセス)

- AAD 認証を経由した不正アクセス
  - パスワード漏洩 → MFA、パスワードロック
  - 内部犯行 → 暗号化(SQL DB)、監査ログ(Azure Monitor)
- 経由しない不正アクセス
  - ネットワークセキュリティ		(VNet、NSG、WAF)
  - セキュリティパッチの自動化	(App Service Environment)

### ネットワーク分離と通信制限

仮想ネットワーク(VNet)で、専用のプライベートアドレス空間を用意し、外部からのアクセスを制限
VNetへのアクセスは、Azure Load Balancer (L4)、Application Gateway 経由とする。
Subnet 間通信を Network Security Group (NSG)で必要なものだけに制限。
SQL Database等のマネージド・サービスとの通信は、Virtual Network Service Endpoints で制限。

### 多段防御

攻撃を受けた場合、多段な防衛線は効果的です。なんからの問題で、防衛線の一部が脆弱な状態になっても複数あることで時間を稼ぐことができます。監視と多段防衛の両輪とすることで、驚異に対する抵抗力は劇的に向上します。

本システムでは、下記ように、ネットワーク通信を制限、既知の脆弱性に対する対策、データの暗号化の多段防衛としている。

- ネットワーク分離
  - Azure Load Balancer のポート制限
  - VNetにサービスを配備する
  - 役務毎にサブネットを割り当てる
  - サブネット間の通信はNSGで制限し、必要な通信だけを許可する
 
- 既知の脆弱性からの防御
  - WAF(Application Gateway)
  - セキュリティパッチの自動化（PaaS）

- データは暗号化して保存する
  - 鍵管理はKey Vaultを利用する
  - SQL Database の暗号化を利用する


### 脅威分析

驚異分析を行いアーキテクチャ、アプリケーション、業務設計に反映した。

{{< figure src="images/08-attack.png" title="図7 驚異分析" width="800">}}

ここに書くには数が多いので、以下に驚異分析から抜粋を記述する。

#### 1. Azure管理
- 脅威：Azureの管理者権限を取得すると、NSGなどAzueリソースの構成を変更することでアクセス権を得ることができる
- 軽減：
  - 通常業務のAzureアカウントと管理者アカウントを分け、最小権限のAzureアカウントで運用する
  - AzureアカウントのMFAを有効にする

#### 2. インターネットからApplication Gatewayへのアクセス
- 脅威：アカウントの偽装、脆弱性を使った特権の獲得
- 軽減：
  - httpsを利用し認証情報を保護
  - WAFによる既知の脆弱性の対策
  - NSGによる通信制御
  - ADD、RBACによる変更制限

#### 3. App Service Environmentからデータベースへのアクセス
- 脅威：アカウント漏洩、脆弱性を使った特権の獲得
- 軽減：
  - sslを利用したデータ保護
  - PaaSによるパッチ適応
  - NSGによる通信制御
  - ADD、RBACによる変更制限
  - Key Vaultによるデータベース接続情報の保護

#### 4. 運用管理者からデータベースへのアクセス
- 脅威：アカウント漏洩、脆弱性を使った特権の獲得
- 軽減：
  - 閉域網からアクセス
  - AAD によるアカウント管理

#### 5. 社内業務システムからの管理画面へのアクセス
- 脅威：アカウントの偽装、脆弱性を使った特権の獲得
- 軽減：
  - インタネット経由と同様
  - 閉域網からのみのアクセスに制限
  - 個別アカウントの利用
  - 操作ログの監査基準での保存
  - 機密データの参照権限の制限

## 3. 監査、ログ、監視

Azureでは、主に３種類のログがあり、Application Insightsと、Log Analyticsという、２つのログストアが用意されている。

- Application Log
- Diagnostics Log(診断ログ）
- Activity Log

### Application Log
アプリケーションによって生成されたログ、Application Insightsのクライアントライブラリによってインスツルメント、あるいはコードで明示的にロギングされる

### Diagnostics Log(診断ログ）
リソースの生成するログ、リソース内で診断ログが生成される。仮想マシンとかでは、ゲストOSからログを収集する仕組みがある。
https://docs.microsoft.com/ja-jp/azure/monitoring-and-diagnostics/monitoring-diagnostic-logs-schema

### Activity Log

主に Azure Resource Manager で発生したアクティビティを記録する。リソースの作成、更新、削除や。正常性について情報が含まれる。

- Tenant log: Azure ADなどSubscriptionの外側で生成されるログ
- Resource log: NSGや、Storage などResources が生成するログ

{{< figure src="images/09-log.png" title="図8 監査、ログ" width="800">}}

PCI DSSの要件10では、「少なくとも１年は保管、最低3ヶ月間はオンラインで閲覧利用できるようにする」とある。そのため、監査ログはLog Analytics に保存し、リテンション期間を要件に合わせて設定している。

この２つのログストアは、同じクエリ言語（KQL）で非常に柔軟に参照可能で、監査だけで無く運用ツールとしても優れている。また、RBACによってアクセス可、不可を制御出来るので運用もしやすい。

この２つのログストアは一旦書き込んだものは改竄することが出来ないようになっているが、GDPR要件によってパージ（削除）ができるようになっているので注意が必要だ。パージは別権限になっているので、通常はパージ権限を落としたユーザーで運用するようにする。

## 4. 暗号化 ー鍵管理

鍵管理には、HSMをバックエンドに持った Azure Key Vault を利用する。
Key Vaultの管理プレーン、データプレーンに適切なアクセス権を設定し、監査ログを取る。そのKey Vaultでシークレット、鍵を一元管理することで、アプリケーションのセキュリティを向上させる。

PCI DSS要件3,4では、カード会員データを保存する時に、暗号化を選択している場合のみに鍵管理プロセスが求められているが、Azureでは鍵管理のコストが低いので、本システムではデータベースの接続情報、暗号化鍵、Service Principalが利用する証明書などを含めシステム上センシティブ情報となるものをKey Vaultに保存している。鍵管理を行うセキュリティオペレータを別途設けることで、システム上のセンシティブ情報を知るスタッフを減らす。これは、情報の局在化による驚異の低減という意味がある。

Key Vault は、オンプレでは高価だったHSMベースの鍵管理が、クラウドによって費用対効果が大きく改善された例と言える。

## さいごに

Azure プラットフォームを使うことで、従来に比べセキュリティ上安全なシステうが作りやすく成りました。一方で適切な設定、運用が従来のものと違うのも事実です。
多くのAzureサービスを使っているため、固有の難しさはありますが、アーキテクチャから運用を考慮したサービスとすることで、コストとセキュリティの両立という従来では難しかったことができるようになってきています。





