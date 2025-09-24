---
title: クエリ エージェント
description: "自然言語理解を用いて複数の Weaviate コレクションに渡る複雑なクエリを処理する AI エージェントの概要。"
image: og/docs/agents.jpg
# tags: ['agents', 'getting started', 'query agent']
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import FilteredTextBlock from '@site/src/components/Documentation/FilteredTextBlock';
import PyCode from '!!raw-loader!/docs/agents/\_includes/query_agent.py';
import TSCode from '!!raw-loader!/docs/agents/\_includes/query_agent.mts';

# Weaviate Query エージェント：概要

Weaviate Query エージェントは、Weaviate Cloud に保存されたデータに基づいて自然言語クエリに回答するために設計された、事前構築済みのエージェント サービスです。

ユーザーは自然言語でプロンプトや質問を入力するだけで、Query エージェントが途中のすべてのステップを処理し、回答を返します。

![Weaviate Query エージェントのユーザー視点](../_includes/query_agent_usage_light.png#gh-light-mode-only "Weaviate Query エージェントのユーザー視点")
![Weaviate Query エージェントのユーザー視点](../_includes/query_agent_usage_dark.png#gh-dark-mode-only "Weaviate Query エージェントのユーザー視点")

## アーキテクチャ

Query エージェントは Weaviate Cloud 上のサービスとして提供されます。

ユーザーがプロンプトやクエリを入力すると、Query エージェントはそれと既知のコンテキストを解析し、自律的に検索を実行します。

:::tip Query Agent context
Query エージェントはコレクションとプロパティの説明を解析し、関連するクエリを構築するための理解を深めます。<br/>

コンテキストには、以前の会話履歴やその他の関連情報が含まれる場合もあります。
:::

## Query エージェント：ワークフローの可視化

![Weaviate Query エージェントの高レベル構成](../_includes/query_agent_architecture_light.png#gh-light-mode-only "Weaviate Query エージェントの高レベル構成")
![Weaviate Query エージェントの高レベル構成](../_includes/query_agent_architecture_dark.png#gh-dark-mode-only "Weaviate Query エージェントの高レベル構成")

Query エージェントは次の高レベル ステップで動作します。

- 適切な生成モデル (例: 大規模言語モデル) を用いてタスクと必要なクエリを解析し、実行すべき具体的なクエリを決定します。(ステップ 1 & 2)
- Weaviate へクエリを送信します。Weaviate は指定されたベクトライザー統合を使用してクエリを必要に応じてベクトル化します。(ステップ 3-5)
- Weaviate から結果を受け取り、適切な生成モデルを用いてユーザーのプロンプト／クエリへの最終回答を生成します。(ステップ 6)

その後、Query エージェントは回答とともに、Weaviate から取得した検索結果などの中間出力も返します。

`Query Agent` という用語はシステム全体を指します。Query エージェントは内部的に複数のサブシステム (マイクロサービスや各タスクを担うエージェントなど) で構成される場合があります。

![Weaviate Query エージェントは複数のエージェントで構成される](../_includes/query_agent_info_light.png#gh-light-mode-only "Weaviate Query エージェントは複数のエージェントで構成される")
![Weaviate Query エージェントは複数のエージェントで構成される](../_includes/query_agent_info_dark.png#gh-dark-mode-only "Weaviate Query エージェントは複数のエージェントで構成される")

## クエリ実行

Query エージェントは 2 種類のクエリをサポートします。

- **`Search`** 自然言語で Query エージェントを通じて Weaviate を検索します。Query エージェントが質問を処理し、必要な検索を Weaviate で実行して関連オブジェクトを返します。

- **`Ask`** 自然言語で Query エージェントに質問します。Query エージェントが質問を処理し、必要な検索を Weaviate で実行して回答を返します。

## 基本的な使い方

ここでは Weaviate エージェントの使用方法を概観します。詳細は [Usage](./usage.md) ページをご覧ください。

:::note Prerequisites

このエージェントは、[Weaviate Cloud](/cloud/index.mdx) インスタンスと、対応バージョンの Weaviate [クライアント ライブラリ](./usage.md#client-library) でのみ利用できます。

:::

### 使用例

Weaviate クライアントのインスタンスを Query エージェントに渡すと、エージェントはクライアントから必要な情報を取得してクエリを実行します。

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

次に、自然言語でクエリを入力します。Query エージェントがクエリを処理し、必要な検索を Weaviate で実行して回答を返します。

### `Search` (取得のみ)

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

Query エージェントは、前回の応答を追加コンテキストとして使用しながら後続の質問にも対応できます。

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



### `Ask` （回答生成付き）

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

## 追加リソース

より詳細なこのエージェントの使用方法については、[使用方法](./usage.md) ページをご覧ください。

## 質問とフィードバック

import DocsFeedback from '/\_includes/docs-feedback.mdx';

<DocsFeedback/>

