---
title: "Azure 上での実装と課題"
date: 2019-07-08T00:00:00+09:00
tags: [Azure,PCIDSS]
description: ""
keywords:
  - "PCIDSS"
  - "Azure"
  - "Blueprint"
eyecatch: "/images/hugo/hugo.png"
draft: true
weight: 20
---


## 実装と課題

Azure上での構築で話題になったものをベースに気がついたものを列挙した。

Azureは日々進歩し、.NET Coreでの実装をベースに各技術に深掘りする。コードがあるものはコードを入れ、下記のドキュメントに関連する記述があるものは随時入れる。

* [Azure のセキュリティのドキュメント](https://docs.microsoft.com/ja-jp/azure/security/)
* [Azure アーキテクチャ センター](https://docs.microsoft.com/ja-jp/azure/architecture/)


1. KeyVaultと、Microsoft.Extensions.Configuration
2. シークレットの管理と更新（Azure App Configuration に期待できる面）
3. KeyVaultのアクセス権管理、RBACとAccess Policy
4. Application InsightsとMicrosoft.Extensions.Logging
5. Azure BatchとWebJobの使い分け
6. ASEとAppServiceの使い分け
7. Application InsightsとLog Analytics
8. Application InsightsのTelemetry 種別と使い分け（LogとMetrics、TraceとEventなど）
9. Application GatewayとASEの接続
10. Application GatewayのWAF
11. ER,VNet,NSGによるネットワーク分離
12. 踏み台サーバの役割と実装
13. 監査ログで網羅したい範囲
14. 運用アカウント管理、Azure AD、RBAC
15. DevOpsとアプリケーション管理
16. 監査ログとLog Analytics（GDPR要件）
17. 証明書の管理（システム内には複数の証明書がある、適切なタイミングで更新が必要）
18. KQLの利用（EUC的な側面もある）
19. Security Centerの範囲
20. Azure BatchからのKVへのアクセス（MSI非対応リソースの例としての）
21. 開発環境と本番環境の分離（赤間さんのドキュメントでも同じことをしてたはず）
22. AppServiceのsandbox（ネットワーク分離とは別の分離技術）
23. Azure Batch によるサーバーレスバッチの実装

more
