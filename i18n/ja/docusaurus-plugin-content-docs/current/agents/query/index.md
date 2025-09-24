---
title: クエリ エージェント
description: "自然言語理解を用いて複数の Weaviate コレクションにわたる複雑なクエリを処理する AI エージェントの概要。"
image: og/docs/agents.jpg
# tags: ['agents', 'getting started', 'query agent']
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import FilteredTextBlock from '@site/src/components/Documentation/FilteredTextBlock';
import PyCode from '!!raw-loader!/docs/agents/\_includes/query_agent.py';
import TSCode from '!!raw-loader!/docs/agents/\_includes/query_agent.mts';

# Weaviate クエリ エージェント: 概要

 Weaviate クエリ エージェント は、 Weaviate Cloud に保存されたデータを基に自然言語クエリへ回答するために設計された、あらかじめ構築済みのエージェント型サービスです。

ユーザーは自然言語でプロンプト／質問を入力するだけで、 クエリ エージェント が必要な処理をすべて行い、回答を返します。

![Weaviate Query Agent from a user perspective](../_includes/query_agent_usage_light.png#gh-light-mode-only "Weaviate Query Agent from a user perspective")
![Weaviate Query Agent from a user perspective](../_includes/query_agent_usage_dark.png#gh-dark-mode-only "Weaviate Query Agent from a user perspective")

## アーキテクチャ

 クエリ エージェント は Weaviate Cloud のサービスとして提供されます。

ユーザーがプロンプト／クエリを入力すると、 クエリ エージェント はそれと既知のコンテキストを解析し、自律的に検索を実行します。

:::tip Query Agent context
 クエリ エージェント は、コレクションとプロパティの説明を解析して、関連するクエリをどのように構築すべきかを理解します。<br/>

コンテキストには、以前の会話履歴やその他の関連情報が含まれる場合もあります。
:::

## クエリ エージェント: ワークフローの可視化

![Weaviate Query Agent at a high level](../_includes/query_agent_architecture_light.png#gh-light-mode-only "Weaviate Query Agent at a high level")
![Weaviate Query Agent at a high level](../_includes/query_agent_architecture_dark.png#gh-dark-mode-only "Weaviate Query Agent at a high level")

 クエリ エージェント は、以下の高レベルな手順に従います。

- 適切な生成モデル（例: 大規模言語モデル）を用いてタスクと必要なクエリを解析し、実行すべき正確なクエリを決定します。（ステップ 1 & 2）
- Weaviate にクエリを送信します。 Weaviate は指定された ベクトライザー 統合を使用して必要に応じクエリをベクトル化します。（ステップ 3–5）
- Weaviate から結果を受け取り、適切な生成モデルを用いてユーザーのプロンプト／クエリに対する最終回答を生成します。（ステップ 6）

その後、 クエリ エージェント はユーザーへ回答と、中間出力（ Weaviate からの検索結果など）を返します。

`Query Agent` という用語はシステム全体を指します。 クエリ エージェント は複数のサブシステム（マイクロサービスやタスクごとに責任を持つエージェントなど）で構成される場合があります。

![Weaviate Query Agent comprises multiple agents](../_includes/query_agent_info_light.png#gh-light-mode-only "Weaviate Query Agent comprises multiple agents")
![Weaviate Query Agent comprises multiple agents](../_includes/query_agent_info_dark.png#gh-dark-mode-only "Weaviate Query Agent comprises multiple agents")

## クエリ

 クエリ エージェント は 2 つのクエリタイプをサポートします。

- **`Search`**: 自然言語で クエリ エージェント を使って Weaviate を検索します。 クエリ エージェント は質問を処理し、 Weaviate で必要な検索を実行し、関連オブジェクトを返します。

- **`Ask`**: 自然言語で クエリ エージェント に質問します。 クエリ エージェント は質問を処理し、 Weaviate で必要な検索を実行して回答を返します。

## 基本的な使い方

以下はこの Weaviate エージェント の使用方法の概要です。詳細は [Usage](./usage.md) ページをご覧ください。

:::note Prerequisites

このエージェントは、 [Weaviate Cloud](/cloud/index.mdx) インスタンスと、対応するバージョンの Weaviate [client library](./usage.md#client-library) でのみ利用できます。

:::

### 使用例

Weaviate クライアントのインスタンスを クエリ エージェント に渡すと、 クエリ エージェント がクエリ実行に必要な情報をクライアントから取得します。

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START InstantiateQueryAgent"
            endMarker="# END InstantiateQueryAgent"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
        <FilteredTextBlock
            text={TSCode}
            startMarker="// START InstantiateQueryAgent"
            endMarker="// END InstantiateQueryAgent"
            language="ts"
        />
    </TabItem>

</Tabs>

次に、自然言語のクエリを入力します。 クエリ エージェント がクエリを処理し、 Weaviate で必要な検索を実行し、回答を返します。

### `Search` （検索のみ）

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START BasicSearchQuery"
            endMarker="# END BasicSearchQuery"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
    <FilteredTextBlock
            text={TSCode}
            startMarker="// START BasicSearchQuery"
            endMarker="// END BasicSearchQuery"
            language="ts"
        />
    </TabItem>

</Tabs>

 クエリ エージェント はフォローアップ クエリも処理でき、前回の応答を追加コンテキストとして利用します。

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START FollowUpQuery"
            endMarker="# END FollowUpQuery"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
    <FilteredTextBlock
            text={TSCode}
            startMarker="// START FollowUpQuery"
            endMarker="// END FollowUpQuery"
            language="ts"
        />
    </TabItem>

</Tabs>



### `Ask`（回答生成付き）

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START BasicAskQuery"
            endMarker="# END BasicAskQuery"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
     <FilteredTextBlock
            text={TSCode}
            startMarker="// START BasicAskQuery"
            endMarker="// END BasicAskQuery"
            language="ts"
        />
    </TabItem>
</Tabs>

## 参考資料

この Agent を使う方法の詳細については、[Usage](./usage.md) ページをご覧ください。

## 質問とフィードバック

import DocsFeedback from '/\_includes/docs-feedback.mdx';

<DocsFeedback/>

