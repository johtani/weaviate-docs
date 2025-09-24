---
title: 使用方法
sidebar_position: 30
description: "Query エージェントを実装するための技術ドキュメントと使用例"
image: og/docs/agents.jpg
# tags: ['agents', 'getting started', 'query agent']
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import FilteredTextBlock from '@site/src/components/Documentation/FilteredTextBlock';
import PyCode from '!!raw-loader!/docs/agents/\_includes/query_agent.py';
import TSCode from '!!raw-loader!/docs/agents/\_includes/query_agent.mts';

# Weaviate Query エージェントの使用方法

Weaviate Query エージェントは、 Weaviate Cloud に保存されたデータを基に自然言語クエリへ回答するために設計された、あらかじめ構築済みのエージェント型サービスです。  

ユーザーは自然言語でプロンプトや質問を入力するだけで、 Query エージェントがその間に必要なすべてのステップを実行し、回答を返します。

![ユーザー視点から見た Weaviate Query エージェント](../_includes/query_agent_usage_light.png#gh-light-mode-only "ユーザー視点から見た Weaviate Query エージェント")
![ユーザー視点から見た Weaviate Query エージェント](../_includes/query_agent_usage_dark.png#gh-dark-mode-only "ユーザー視点から見た Weaviate Query エージェント")

本ページでは、 Weaviate Cloud に保存されたデータを用いて、 Query エージェントで自然言語クエリに回答する方法を説明します。

## 前提条件

### Weaviate インスタンス

本エージェントは Weaviate Cloud インスタンスでのみご利用いただけます。 Weaviate Cloud インスタンスのセットアップ方法については [Weaviate Cloud のドキュメント](/cloud/index.mdx) を参照してください。

[Weaviate Cloud](https://console.weaviate.cloud/) 上の無料 Sandbox インスタンスで本エージェントをお試しいただけます。

### クライアントライブラリ

:::note 対応言語
現時点では、本エージェントは Python と TypeScript/JavaScript のみ対応しています。今後、他の言語サポートも追加予定です。
:::

Python では、オプション `agents` 付きで Weaviate クライアントライブラリをインストールすることで、 Weaviate Agents を利用できます。これにより `weaviate-client` パッケージと一緒に `weaviate-agents` パッケージがインストールされます。TypeScript/JavaScript では、 `weaviate-client` パッケージと併せて `weaviate-agents` パッケージをインストールしてください。

以下のコマンドでクライアントライブラリをインストールします。

<Tabs groupId="languages">
<TabItem value="py_agents" label="Python">

```shell
pip install -U weaviate-client[agents]
```

#### トラブルシューティング: `pip` に最新バージョンをインストールさせる

すでにインストール済みの場合、 `pip install -U "weaviate-client[agents]"` を実行しても `weaviate-agents` が [最新バージョン](https://pypi.org/project/weaviate-agents/) に更新されないことがあります。その際は、 `weaviate-agents` パッケージを明示的にアップグレードしてみてください。

```shell
pip install -U weaviate-agents
```

または [特定バージョン](https://github.com/weaviate/weaviate-agents-python-client/tags) をインストールします。

```shell
pip install -U weaviate-agents==||site.weaviate_agents_version||
```

</TabItem>
<TabItem value="ts_agents" label="JavaScript/TypeScript">

```shell
npm install weaviate-agents@latest
```

</TabItem>

</Tabs>

## Query エージェントのインスタンス化

### 基本的なインスタンス化

以下を指定します。

- 対象となる [Weaviate Cloud インスタンスの詳細](/cloud/manage-clusters/connect.mdx)（例: `WeaviateClient` オブジェクト）
- クエリ対象となるデフォルトのコレクション一覧

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

### コレクションの設定

クエリ対象のコレクション一覧は、次のように追加設定できます。

- テナント名（マルチテナントコレクションの場合は必須）
- クエリする対象ベクトル（任意）
- エージェントが使用するプロパティ名のリスト（任意）
- エージェントが生成したフィルターに加えて常に適用する [追加フィルター](#user-defined-filters)（任意）

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START QueryAgentCollectionConfiguration"
            endMarker="# END QueryAgentCollectionConfiguration"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
        <FilteredTextBlock
            text={TSCode}
            startMarker="// START QueryAgentCollectionConfiguration"
            endMarker="// END QueryAgentCollectionConfiguration"
            language="ts"
        />
    </TabItem>

</Tabs>

:::info Query エージェントがアクセスできる範囲は？

Query エージェントは、渡された Weaviate クライアントオブジェクトから認証情報を取得します。アクセス範囲は、 Query エージェントに渡されたコレクション名によってさらに制限できます。

たとえば、関連する Weaviate 認証情報のユーザーが一部のコレクションのみアクセス可能な場合、 Query エージェントもそのコレクションにしかアクセスできません。

:::

### 追加オプション

Query Agent は、次のような追加オプションを指定してインスタンス化できます。

- `system_prompt`: Weaviate チームが提供するデフォルトの system prompt を置き換えるカスタム system prompt（JavaScript では `systemPrompt`）。
- `timeout`: Query Agent が 1 件のクエリに費やす最大時間（秒）。サーバー側デフォルトは 60。

#### カスタム system prompt

Query Agent の挙動を制御するために、カスタム system prompt を指定できます:

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START SystemPromptExample"
            endMarker="# END SystemPromptExample"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
    <FilteredTextBlock
            text={TSCode}
            startMarker="// START SystemPromptExample"
            endMarker="// END SystemPromptExample"
            language="ts"
        />
    </TabItem>

</Tabs>

### ユーザー定義フィルター

論理 `AND` でエージェントが生成したフィルターと常に結合される永続的なフィルターを適用できます。

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START UserDefinedFilters"
            endMarker="# END UserDefinedFilters"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
    <FilteredTextBlock
            text={TSCode}
            startMarker="// START UserDefinedFilters"
            endMarker="// END UserDefinedFilters"
            language="ts"
        />
    </TabItem>

</Tabs>

### Async Python クライアント

async Python クライアントでの使用例については、[Async Python クライアントのセクション](#usage---async-python-client)を参照してください。

## クエリ実行

Query Agent は 2 種類のクエリ方式をサポートします:

- [**`Search`**](#search)
- [**`Ask`**](#ask)

### `Search`

Query Agent を使って Weaviate を `Search` し、自然言語で検索できます。Query Agent は質問を処理し、Weaviate で必要な検索を実行して、関連オブジェクトを返します。

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

#### `Search` レスポンス構造

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START SearchModeResponseStructure"
            endMarker="# END SearchModeResponseStructure"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
     <FilteredTextBlock
            text={TSCode}
            startMarker="// START SearchModeResponseStructure"
            endMarker="// END SearchModeResponseStructure"
            language="ts"
        />
    </TabItem>

</Tabs>

<details>
  <summary>Example output</summary>

```
Original query: winter boots for under $100
Total time: 4.695224046707153
Usage statistics: requests=2 request_tokens=143 response_tokens=9 total_tokens=152 details=None
Search performed: queries=['winter boots'] filters=[[IntegerPropertyFilter(property_name='price', operator=<ComparisonOperator.LESS_THAN: '<'>, value=100.0)]] filter_operators='AND' collection='ECommerce'
Properties: {'name': 'Bramble Berry Loafers', 'description': 'Embrace your love for the countryside with our soft, hand-stitched loafers, perfect for quiet walks through the garden. Crafted with eco-friendly dyed soft pink leather and adorned with a subtle leaf embossing, these shoes are a testament to the beauty of understated simplicity.', 'price': 75.0}
Metadata: {'creation_time': None, 'last_update_time': None, 'distance': None, 'certainty': None, 'score': 0.4921875, 'explain_score': None, 'is_consistent': None, 'rerank_score': None}
Properties: {'name': 'Glitter Bootcut Fantasy', 'description': "Step back into the early 2000s with these dazzling silver bootcut jeans. Embracing the era's optimism, these bottoms offer a comfortable fit with a touch of stretch, perfect for dancing the night away.", 'price': 69.0}
Metadata: {'creation_time': None, 'last_update_time': None, 'distance': None, 'certainty': None, 'score': 0.47265625, 'explain_score': None, 'is_consistent': None, 'rerank_score': None}
Properties: {'name': 'Celestial Step Platform Sneakers', 'description': 'Stride into the past with these baby blue platforms, boasting a dreamy sky hue and cushy soles for day-to-night comfort. Perfect for adding a touch of whimsy to any outfit.', 'price': 90.0}
Metadata: {'creation_time': None, 'last_update_time': None, 'distance': None, 'certainty': None, 'score': 0.48828125, 'explain_score': None, 'is_consistent': None, 'rerank_score': None}
Properties: {'name': 'Garden Bliss Heels', 'description': 'Embrace the simplicity of countryside elegance with our soft lavender heels, intricately designed with delicate floral embroidery. Perfect for occasions that call for a touch of whimsy and comfort.', 'price': 90.0}
Metadata: {'creation_time': None, 'last_update_time': None, 'distance': None, 'certainty': None, 'score': 0.45703125, 'explain_score': None, 'is_consistent': None, 'rerank_score': None}
Properties: {'name': 'Garden Stroll Loafers', 'description': 'Embrace the essence of leisurely countryside walks with our soft, leather loafers. Designed for the natural wanderer, these shoes feature delicate, hand-stitched floral motifs set against a soft, cream background, making every step a blend of comfort and timeless elegance.', 'price': 90.0}
Metadata: {'creation_time': None, 'last_update_time': None, 'distance': None, 'certainty': None, 'score': 0.451171875, 'explain_score': None, 'is_consistent': None, 'rerank_score': None}
```

</details>

#### ページネーション付き `Search`

`Search` は、大きな結果セットを効率的に扱うためにページネーションをサポートしています:

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START SearchPagination"
            endMarker="# END SearchPagination"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
    <FilteredTextBlock
            text={TSCode}
            startMarker="// START SearchPagination"
            endMarker="// END SearchPagination"
            language="ts"
        />
    </TabItem>

</Tabs>

<details>
  <summary>Example output</summary>

```
Page 1:
  Glide Platforms - $90.0
  Garden Haven Tote - $58.0
  Sky Shimmer Sneaks - $69.0

Page 2:
  Garden Haven Tote - $58.0
  Celestial Step Platform Sneakers - $90.0
  Eloquent Satchel - $59.0

Page 3:
  Garden Haven Tote - $58.0
  Celestial Step Platform Sneakers - $90.0
  Eloquent Satchel - $59.0
```

</details>

### `Ask`

`Ask` では、 Query Agent に自然言語で質問できます。 Query Agent は質問を処理し、 Weaviate 内で必要な検索を実行し、回答を返します。

:::tip クエリは慎重に検討しましょう
Query Agent は、入力されたクエリを基に戦略を立てます。可能な限りあいまいさを避け、完全かつ簡潔なクエリを心掛けてください。
:::

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

### 実行時にコレクションを設定する

クエリ対象のコレクション一覧は、クエリ実行時に名前のリスト、または追加設定付きで上書きできます。

#### コレクション名のみを指定する

この例では、 Query Agent に設定済みのコレクションを今回のクエリに限り上書きします。

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START QueryAgentAskBasicCollectionSelection"
            endMarker="# END QueryAgentAskBasicCollectionSelection"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
    <FilteredTextBlock
            text={TSCode}
            startMarker="// START QueryAgentAskBasicCollectionSelection"
            endMarker="// END QueryAgentAskBasicCollectionSelection"
            language="ts"
        />
    </TabItem>

</Tabs>

#### コレクションを詳細設定する

この例では、 Query Agent に設定済みのコレクションを今回のクエリに限り上書きし、必要に応じて追加オプションも指定します。例:

- 対象 ベクトル
- 表示するプロパティ
- 対象テナント
- 追加フィルター

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START QueryAgentAskCollectionConfig"
            endMarker="# END QueryAgentAskCollectionConfig"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
    <FilteredTextBlock
            text={TSCode}
            startMarker="// START QueryAgentAskCollectionConfig"
            endMarker="// END QueryAgentAskCollectionConfig"
            language="ts"
        />
    </TabItem>

</Tabs>

### 会話型クエリ

Query Agent は、 `ChatMessage` オブジェクトのリストを渡すことで複数ターンの会話をサポートします。これは `Search` と `Ask` の両方のクエリタイプで機能します。

`ChatMessage` で会話を構築する際、メッセージには次の 2 つのロールがあります。

- **`user`** - 人間のユーザーが質問やコンテキストを提供するメッセージを表します
- **`assistant`** - Query Agent の応答、または会話履歴内の AI アシスタントの応答を表します

会話履歴は、以前のやり取りからコンテキストを理解するのに役立ち、より一貫性のあるマルチターン対話を可能にします。適切な会話フローを維持するため、 user と assistant のロールを交互に配置してください。

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START ConversationalQuery"
            endMarker="# END ConversationalQuery"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
        <FilteredTextBlock
            text={TSCode}
            startMarker="// START ConversationalQuery"
            endMarker="// END ConversationalQuery"
            language="ts"
        />
    </TabItem>

</Tabs>

## ストリーム応答

Query Agent はストリーム応答にも対応しており、回答が生成されると同時に受け取ることができます。

ストリーミング応答は、次のオプションパラメーターでリクエストできます。

- `include_progress`: `True` に設定すると、 Query Agent はクエリ処理の進行状況をストリームします。
- `include_final_state`: `True` に設定すると、完全な回答を待たず、生成され次第ストリームします。

`include_progress` と `include_final_state` の両方を `False` に設定した場合、 Query Agent は進捗更新や最終状態なしで、生成された回答トークンのみをストリームします。

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START StreamResponse"
            endMarker="# END StreamResponse"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
        <FilteredTextBlock
            text={TSCode}
            startMarker="// START StreamResponse"
            endMarker="// END StreamResponse"
            language="ts"
        />
    </TabItem>
</Tabs>



## レスポンスの確認

Query Agent からのレスポンスには、最終的な回答に加えて追加の補足情報が含まれます。

補足情報には、実行された検索や集約、欠けている可能性のある情報、エージェントが使用した LLM トークン数などが含まれます。

### ヘルパー関数

レスポンスを読みやすい形式で表示するには、提供されているヘルパー関数（例: `.display()` メソッド）を試してください。

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

これにより、レスポンスと Query Agent が見つけた補足情報の概要が出力されます。

<details>
  <summary>出力例</summary>

```text
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────── 🔍 Original Query ────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                                                                                                                                                                   │
│ I like vintage clothes and nice shoes. Recommend some of each below $60.                                                                                                                                                                          │
│                                                                                                                                                                                                                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────────────────── 📝 Final Answer ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                                                                                                                                                                   │
│ For vintage clothing under $60, you might like the Vintage Philosopher Midi Dress by Echo & Stitch. It features deep green velvet fabric with antique gold button details, tailored fit, and pleated skirt, perfect for a classic vintage look.   │
│                                                                                                                                                                                                                                                   │
│ For nice shoes under $60, consider the Glide Platforms by Vivid Verse. These are high-shine pink platform sneakers with cushioned soles, inspired by early 2000s playful glamour, offering both style and comfort.                                │
│                                                                                                                                                                                                                                                   │
│ Both options combine vintage or retro aesthetics with an affordable price point under $60.                                                                                                                                                        │
│                                                                                                                                                                                                                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────── 🔭 Searches Executed 1/2 ─────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                                                                                                                                                                   │
│ QueryResultWithCollection(queries=['vintage clothing'], filters=[[]], filter_operators='AND', collection='ECommerce')                                                                                                                             │
│                                                                                                                                                                                                                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────── 🔭 Searches Executed 2/2 ─────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                                                                                                                                                                   │
│ QueryResultWithCollection(queries=['nice shoes'], filters=[[]], filter_operators='AND', collection='ECommerce')                                                                                                                                   │
│                                                                                                                                                                                                                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                                                                                                                                                                   │
│ 📊 No Aggregations Run                                                                                                                                                                                                                            │
│                                                                                                                                                                                                                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─────────────────────────────────────────────────────────────────────────────────────────────────────────────────── 📚 Sources ────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                                                                                                                                                                   │
│  - object_id='a7aa8f8a-f02f-4c72-93a3-38bcbd8d5581' collection='ECommerce'                                                                                                                                                                        │
│  - object_id='ff5ecd6e-8cb9-47a0-bc1c-2793d0172984' collection='ECommerce'                                                                                                                                                                        │
│                                                                                                                                                                                                                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯


  📊 Usage Statistics
┌────────────────┬─────┐
│ LLM Requests:  │ 5   │
│ Input Tokens:  │ 288 │
│ Output Tokens: │ 17  │
│ Total Tokens:  │ 305 │
└────────────────┴─────┘

Total Time Taken: 7.58s
```

</details>

### 検査例

この例では、以下を出力します。

- 元のユーザークエリ  
- Query Agent から提供された回答  
- Query Agent が実行した検索 &amp; 集約（存在する場合）  
- 欠けている情報

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START InspectResponseExample"
            endMarker="# END InspectResponseExample"
            language="py"
        />
    </TabItem>
    <TabItem value="ts_agents" label="JavaScript/TypeScript">
        <FilteredTextBlock
            text={TSCode}
            startMarker="// START InspectResponseExample"
            endMarker="// END InspectResponseExample"
            language="ts"
        />
    </TabItem>

</Tabs>

## 使用方法 - Async Python クライアント

async Python Weaviate クライアントを使用する場合、インスタンス化パターンはほぼ同じです。違いは `QueryAgent` クラスの代わりに `AsyncQueryAgent` クラスを使用する点です。

結果として得られる async パターンは以下のとおりです。

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START UsageAsyncQueryAgent"
            endMarker="# END UsageAsyncQueryAgent"
            language="py"
        />
    </TabItem>
</Tabs>

### ストリーミング

async Query Agent はレスポンスをストリームすることもでき、生成中の回答をリアルタイムで受け取れます。

<Tabs groupId="languages">
    <TabItem value="py_agents" label="Python">
        <FilteredTextBlock
            text={PyCode}
            startMarker="# START StreamAsyncResponse"
            endMarker="# END StreamAsyncResponse"
            language="py"
        />
    </TabItem>
</Tabs>

## 制限事項とトラブルシューティング

### 使用制限

import UsageLimits from "/\_includes/agents/query-agent-usage-limits.mdx";

<UsageLimits />

### カスタム Collection 説明

import CollectionDescriptions from "/\_includes/agents/query-agent-collection-descriptions.mdx";

<CollectionDescriptions />

### 実行時間

import ExecutionTimes from "/\_includes/agents/query-agent-execution-times.mdx";

<ExecutionTimes />



## 質問とフィードバック

import DocsFeedback from '/\_includes/docs-feedback.mdx';

<DocsFeedback/>

