---
title: 生成 AI
sidebar_position: 51
# image: og/docs/integrations/provider_integrations_xai.jpg
# tags: ['model providers', 'xAI', 'generative', 'rag']
---

# Weaviate と xAI の生成 AI

:::info Added in `v1.30.0`
:::

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import FilteredTextBlock from '@site/src/components/Documentation/FilteredTextBlock';
import PyConnect from '!!raw-loader!../_includes/provider.connect.py';
import TSConnect from '!!raw-loader!../_includes/provider.connect.ts';
import PyCode from '!!raw-loader!../_includes/provider.generative.py';
import TSCode from '!!raw-loader!../_includes/provider.generative.ts';

Weaviate の xAI API 連携を利用すると、xAI のモデル機能に Weaviate から直接アクセスできます。

[コレクションを設定](#configure-collection)して xAI の生成 AI モデルを使用すると、Weaviate はお客様の xAI API キーを用いて 検索拡張生成 (Retrieval Augmented Generation, RAG) を実行します。

具体的には、Weaviate が検索を実行して最も関連性の高いオブジェクトを取得し、それらを xAI 上の生成モデルに渡して出力を生成します。

![RAG integration illustration](../_includes/integration_xai_rag.png)

## 必要条件

### Weaviate 構成

お使いの Weaviate インスタンスは、xAI 生成 (`generative-xai`) モジュールで構成されている必要があります。

<details>
  <summary>Weaviate Cloud (WCD) ユーザー向け</summary>

この連携は、Weaviate Cloud (WCD) のサーバーレスインスタンスではデフォルトで有効になっています。

</details>

<details>
  <summary>セルフホストユーザー向け</summary>

- モジュールが有効になっているかを確認するには、[クラスターメタデータ](/deploy/configuration/meta.md) をご確認ください。
- Weaviate でモジュールを有効にする方法は、[モジュール設定方法](../../configuration/modules.md) ガイドをご覧ください。

</details>

### API 資格情報

この連携を使用するには、有効な API キーを Weaviate に提供する必要があります。登録と API キー取得は [xAI](https://console.x.ai/) で行えます。

以下のいずれかの方法で Weaviate に API キーを渡してください。

- Weaviate が参照できる `XAI_APIKEY` 環境変数を設定する
- 下記の例のように実行時に API キーを渡す

<Tabs groupId="languages">

 <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyConnect}
      startMarker="# START XaiInstantiation"
      endMarker="# END XaiInstantiation"
      language="py"
    />
  </TabItem>

 <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSConnect}
      startMarker="// START XaiInstantiation"
      endMarker="// END XaiInstantiation"
      language="ts"
    />
  </TabItem>

</Tabs>

## コレクションを設定

import MutableGenerativeConfig from '/_includes/mutable-generative-config.md';

<MutableGenerativeConfig />

xAI の生成 AI モデルを使用するように [Weaviate インデックスを設定](../../manage-collections/generative-reranker-models.mdx#specify-a-generative-model-integration) します。

<Tabs groupId="languages">
  <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyCode}
      startMarker="# START BasicGenerativexAI"
      endMarker="# END BasicGenerativexAI"
      language="py"
    />
  </TabItem>

  <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSCode}
      startMarker="// START BasicGenerativexAI"
      endMarker="// END BasicGenerativexAI"
      language="ts"
    />
  </TabItem>

</Tabs>

### モデルを選択

Weaviate が使用する [利用可能なモデル](#available-models) を指定することもできます。以下はその設定例です。

<Tabs groupId="languages">
  <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyCode}
      startMarker="# START GenerativexAICustomModel"
      endMarker="# END GenerativexAICustomModel"
      language="py"
    />
  </TabItem>

  <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSCode}
      startMarker="// START GenerativexAICustomModel"
      endMarker="// END GenerativexAICustomModel"
      language="ts"
    />
  </TabItem>

</Tabs>

[利用可能なモデル](#available-models) のいずれかを [指定](#generative-parameters) できます。モデルを指定しない場合は [デフォルトモデル](#available-models) が使用されます。

### 生成パラメーター

以下の生成パラメーターを設定して、モデルの動作をカスタマイズします。

<Tabs groupId="languages">
  <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyCode}
      startMarker="# START FullGenerativexAI"
      endMarker="# END FullGenerativexAI"
      language="py"
    />
  </TabItem>

  <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSCode}
      startMarker="// START FullGenerativexAI"
      endMarker="// END FullGenerativexAI"
      language="ts"
    />
  </TabItem>

</Tabs>

モデル パラメーターの詳細については、[xAI API ドキュメント](https://docs.x.ai/docs/guides/chat#parameters) を参照してください。

## 実行時のモデル選択

コレクション作成時にデフォルトのモデル プロバイダーを設定するだけでなく、クエリ実行時に上書きすることもできます。

<Tabs groupId="languages">
  <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyCode}
      startMarker="# START RuntimeModelSelectionxAI"
      endMarker="# END RuntimeModelSelectionxAI"
      language="py"
    />
  </TabItem>
  <TabItem value="js" label="JS/TS Client v3">
    <FilteredTextBlock
      text={TSCode}
      startMarker="// START RuntimeModelSelectionxAI"
      endMarker="// END RuntimeModelSelectionxAI"
      language="ts"
    />
  </TabItem>
</Tabs>

## 検索拡張生成

生成 AI 連携を設定した後、[単一プロンプト](#single-prompt) または [グループタスク](#grouped-task) のいずれかの方法で RAG 操作を実行します。

### 単一プロンプト

![単一プロンプトの RAG 連携は検索結果ごとに個別の出力を生成します](../_includes/integration_xai_rag.png)

検索結果に含まれる各オブジェクトごとにテキストを生成するには、単一プロンプト方式を使用します。

以下の例では、`n` 件の検索結果それぞれに対して出力を生成します。ここで `n` は `limit` パラメーターで指定します。

単一プロンプト クエリを作成する際は、ブラケット `{}` を使用して、 Weaviate が言語モデルに渡すオブジェクト プロパティを埋め込みます。たとえばオブジェクトの `title` プロパティを渡す場合、クエリ内に `{title}` を含めてください。

<Tabs groupId="languages">

 <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyCode}
      startMarker="# START SinglePromptExample"
      endMarker="# END SinglePromptExample"
      language="py"
    />
  </TabItem>

 <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSCode}
      startMarker="// START SinglePromptExample"
      endMarker="// END SinglePromptExample"
      language="ts"
    />
  </TabItem>

</Tabs>

### グループタスク

![グループタスクの RAG 連携は検索結果全体に対して 1 つの出力を生成します](../_includes/integration_xai_rag.png)

検索結果全体に対して 1 つのテキストを生成するには、グループタスク方式を使用します。

言い換えると、検索結果が `n` 件ある場合、生成モデルはそのグループ全体に対して 1 つの出力を生成します。

<Tabs groupId="languages">

 <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyCode}
      startMarker="# START GroupedTaskExample"
      endMarker="# END GroupedTaskExample"
      language="py"
    />
  </TabItem>

 <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSCode}
      startMarker="// START GroupedTaskExample"
      endMarker="// END GroupedTaskExample"
      language="ts"
    />
  </TabItem>

</Tabs>

## 参考情報

### 利用可能なモデル

 Weaviate では、[xAI の API](https://docs.x.ai/docs/models) にある任意の生成モデルを使用できます。

既定のモデルは `grok-2-latest` です。

## 追加リソース

### コード例

コレクションでインテグレーションを設定すると、 Weaviate におけるデータ管理および検索操作は他のコレクションとまったく同じように機能します。以下のモデル非依存の例をご覧ください:

- [How-to: コレクションの管理](../../manage-collections/index.mdx) と [How-to: オブジェクトの管理](../../manage-objects/index.mdx) の各ガイドでは、データ操作 (すなわちコレクションおよびそれらに含まれるオブジェクトの作成、読み取り、更新、削除) の方法を示しています。
- [How-to: クエリ & 検索](../../search/index.mdx) ガイドでは、ベクトル、キーワード、ハイブリッド検索に加え、検索拡張生成の実行方法を示しています。

### 参考情報

- [xAI API ドキュメント](https://docs.x.ai/docs/introduction)

import DocsFeedback from '/_includes/docs-feedback.mdx';

<DocsFeedback/>

