---
title: "リファレンス"
date: 2019-07-08T00:00:00+09:00
tags: [Azure,PCIDSS]
description: ""
keywords:
  - "PCIDSS"
  - "Azure"
  - "Blueprint"
eyecatch: "/images/hugo/hugo.png"
draft: true
---

## Azure Security and Compliance Blueprint

Azure Security DocumentationのPCI DSSの下は４つに別れてます

- [Data analytics/Analytics for PCI DSS](https://docs.microsoft.com/en-us/azure/security/blueprints/pcidss-analytics-overview)
- [Data warehouse/Data Warehouse for PCI DSS](https://docs.microsoft.com/en-us/azure/security/blueprints/pcidss-dw-overview)
- [IaaS web application/IaaS Web Application for PCI DSS](https://docs.microsoft.com/en-us/azure/security/blueprints/pcidss-iaaswa-overview)
- [PaaS web application/PaaS Web Application for PCI DSS](https://docs.microsoft.com/en-us/azure/security/blueprints/pcidss-paaswa-overview)

## まとまったドキュメント(PDF)の場所

[Azure セキュリティおよびコンプライアンス PCI DSS Blueprint](https://servicetrust.microsoft.com/ViewPage/PCIBlueprint)

## 2019/6/27 の PCI DSS Azure Blueprint のBlog

2019/6/27に、Azure Blueprint(preview) に、PCI-DSS v3.2.1 blueprint を追加するというアナウンスがありました。[New PCI DSS Azure Blueprint makes compliance simpler](https://azure.microsoft.com/en-us/blog/new-pci-dss-azure-blueprint-makes-compliance-simpler/)

![New PCI DSS Azure Blueprint makes compliance simpler](images/newpcidssbp01.png)


### 以下Blogからの概要

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
