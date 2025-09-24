---
title: GPT4All + Weaviate
sidebar_position: 10
image: og/docs/integrations/provider_integrations_gpt4all.jpg
# tags: ['model providers', 'gpt4all']
---

<!-- Note: for images, use https://docs.google.com/presentation/d/15opIcJuaIjEEcs_1Zm8B6pccox2p7_MHSjCnRv4dPfU/edit?usp=sharing -->

:::caution 非推奨の統合
この統合は非推奨で、将来のリリースで削除されます。新しいプロジェクトでは別のモデルプロバイダーの利用を推奨します。

ローカル AI モデルを統合する場合は、[Ollama](../ollama/index.md) または [ローカル HuggingFace](../huggingface/index.md) のモデル統合をご検討ください。
:::

GPT4All ライブラリを使用すると、ご自身のデバイス上で幅広いモデルを手軽に実行できます。 Weaviate は GPT4All ライブラリとシームレスに統合され、ユーザーは Weaviate データベースから互換性のあるモデルを直接活用できます。

これらの統合により、開発者は高度な AI ドリブンアプリケーションを容易に構築できます。

## GPT4All との統合

Weaviate は、ローカルでホストされた GPT4All API にアクセスすることで、互換性のある GPT4All モデルと統合します。

### ベクトル検索のための埋め込みモデル

![Embedding integration illustration](../_includes/integration_gpt4all_embedding.png)

GPT4All の埋め込みモデルは、テキストデータを意味とコンテキストを捉えたベクトル埋め込みに変換します。

[Weaviate は GPT4All の埋め込みモデルと統合](./embeddings.md) して、データのシームレスなベクトル化を実現します。この統合により、追加の前処理やデータ変換を行わずにセマンティック検索やハイブリッド検索を実行できます。

[GPT4All 埋め込み統合ページ](./embeddings.md)

## 概要

これらの統合を使用すると、開発者は Weaviate から直接強力な GPT4All モデルを活用できます。

その結果、AI ドリブンアプリケーションの構築プロセスが簡素化され、開発スピードが向上し、革新的なソリューションの創出に集中できます。

## はじめに

これらの統合を利用するには、GPT4All モデルをホストできるローカルの Weaviate インスタンスが必要です。

各統合ページにアクセスして、GPT4All モデルで Weaviate を設定し、アプリケーションで使用する方法をご確認ください。

- [テキスト埋め込み](./embeddings.md)

## ご質問とフィードバック

import DocsFeedback from '/_includes/docs-feedback.mdx';

<DocsFeedback/>

