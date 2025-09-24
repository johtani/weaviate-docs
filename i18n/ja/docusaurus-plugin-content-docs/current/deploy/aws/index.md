---
title: AWS での Weaviate デプロイ
sidebar_title:
sidebar_position: 0
---

本セクションでは、Amazon Web Services (AWS) 上で Weaviate をデプロイして実行するための包括的なガイドを提供します。開発環境の構築、本番環境へのデプロイ、AWS サービスとの統合など、AWS エコシステムに合わせたインストールガイド、チュートリアル、ハウツー、リファレンス資料を確認いただけます。

## 本セクションの内容

- **インストールガイド:** さまざまな AWS サービスを用いた Weaviate デプロイを段階的に説明します。  
- **チュートリアル:** 一般的な AWS デプロイシナリオを端から端まで解説します。  
- **ハウツーガイド:** 特定の AWS 構成や統合に関するタスク指向の手順です。  
- **リファレンスドキュメント:** AWS 固有の設定オプション、ベストプラクティス、トラブルシューティングガイドです。  

## デプロイ方法

Weaviate には複数の AWS デプロイ方法があり、ユースケースや運用ニーズに応じて選択できます:

### Marketplace 提供

#### [AWS Marketplace - Serverless Cloud](../installation-guides/aws-marketplace.md)

AWS Marketplace から Weaviate Serverless Cloud を直接デプロイし、迅速にクラウド環境を構築しつつ AWS 請求と統合できます。  

この SaaS ソリューションは、以下のニーズを持つ AWS 顧客向けに設計されています:

- AWS 請求との統合  
- 特定リージョンへのデプロイによる規制要件への対応  
- インフラ管理なしでの簡単なセットアップ  

#### [AWS Marketplace - Kubernetes クラスター](../installation-guides/eks-marketplace.md)

AWS CloudFormation テンプレートを使用して AWS Marketplace 経由で Amazon EKS 上に Weaviate をデプロイします。これにより、EKS クラスター (単一ノードグループ)、ロードバランサーコントローラー、および EBS CSI ドライバーが CloudFormation テンプレートで自動的に作成されます。

#### 作成されるリソース:

- 単一ノードグループを持つ EKS クラスター  
- EKS 用ロードバランサーコントローラー  
- 永続ストレージ用 AWS EBS CSI ドライバー  
- 公式 Helm チャートでの Weaviate の最新選択バージョン  

**最適な用途**: 本番環境、セットアップの複雑さなくマネージド Kubernetes を利用したいチーム、エンタープライズグレードのデプロイ  

#### [AWS Marketplace - EC2 インスタンス](../installation-guides/ecs-marketplace.md)

AWS Marketplace を通じて Docker を使用し、単一 EC2 インスタンス上に完全動作する Weaviate をデプロイします。このオプションも CloudFormation テンプレートを使用し、Weaviate をすばやくプロトタイプおよびテストしたい開発者に最適です。

#### 仕様:

- 単一 EC2 インスタンス (デフォルト: m7g.medium)  
- Docker コンテナデプロイ  
- 月次契約 (AWS による即時請求)  
- テストおよび開発向け (エンタープライズサポートなし)  

### セルフマネージドオプション

#### [セルフマネージド EKS](../installation-guides/eks.md)

`eksctl` コマンドラインツールを使用して独自の EKS クラスターを作成および管理し、クラスター設定、スケーリング、管理を完全に制御できます。

#### 特徴:

- クラスター設定を完全に制御  
- カスタムオートスケーリングノードグループ  
- インスタンスタイプとストレージクラスを自由に選択  
- 永続ストレージ用 AWS EBS CSI ドライバーとの統合  

**最適な用途:** Kubernetes の専門知識を持つ組織、カスタムインフラ要件、最大限の柔軟性と制御  

### デプロイ比較

![デプロイ比較マトリックス](./img/deployment-matrix.png)

各デプロイオプションは、管理と制御のレベルが異なります:

- **Serverless Cloud:** 完全マネージド SaaS で自動スケーリング、インフラ管理不要。  
- **Marketplace EKS:** CloudFormation により事前構成されたインフラを持つマネージド Kubernetes コントロールプレーン。  
- **Marketplace EC2:** 月次課金の単一インスタンス Docker デプロイで開発に最適。  
- **セルフマネージド EKS:** EKS クラスターの設定と管理を完全に制御可能。  

## 質問とフィードバック

import DocsFeedback from '/_includes/docs-feedback.mdx';

<DocsFeedback/>

