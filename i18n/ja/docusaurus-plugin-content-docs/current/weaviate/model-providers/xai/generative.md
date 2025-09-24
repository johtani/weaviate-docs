---
title: 生成 AI
sidebar_position: 51
# image: og/docs/integrations/provider_integrations_xai.jpg
# tags: ['model providers', 'xAI', 'generative', 'rag']
---

# Weaviate での xAI 生成 AI

:::info `v1.30.0` で追加
:::

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import FilteredTextBlock from '@site/src/components/Documentation/FilteredTextBlock';
import PyConnect from '!!raw-loader!../_includes/provider.connect.py';
import TSConnect from '!!raw-loader!../_includes/provider.connect.ts';
import PyCode from '!!raw-loader!../_includes/provider.generative.py';
import TSCode from '!!raw-loader!../_includes/provider.generative.ts';

Weaviate と xAI の API 連携により、xAI のモデルの機能を Weaviate から直接利用できます。

[Weaviate コレクションを設定](#configure-collection) して xAI の生成 AI モデルを使用すると、Weaviate は xAI の API キーを用いて 検索拡張生成 (RAG) を実行します。

具体的には、Weaviate が検索を行い、最も関連性の高いオブジェクトを取得し、それらを xAI 上の生成モデルに渡して出力を生成します。

![RAG 統合の図解](../_includes/integration_xai_rag.png)

## 要件

### Weaviate の構成

お使いの Weaviate インスタンスには、xAI 生成モジュール (`generative-xai`) が有効化されている必要があります。

<details>
  <summary>Weaviate Cloud (WCD) のユーザー向け</summary>

この統合は Weaviate Cloud (WCD) のサーバーレスインスタンスではデフォルトで有効化されています。

</details>

<details>
  <summary>セルフホストユーザー向け</summary>

- モジュールが有効化されているかどうかを [クラスターメタデータ](/deploy/configuration/meta.md) で確認してください。  
- Weaviate でモジュールを有効化するには、[モジュールの設定方法](../../configuration/modules.md) ガイドに従ってください。

</details>

### API 資格情報

この統合を利用するには、有効な API キーを Weaviate に提供する必要があります。登録と API キーの取得は [xAI](https://console.x.ai/) で行ってください。

以下のいずれかの方法で Weaviate に API キーを渡します。

- Weaviate が参照できる環境変数 `XAI_APIKEY` を設定する  
- 以下の例のように実行時に API キーを渡す

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

## コレクションの設定

import MutableGenerativeConfig from '/_includes/mutable-generative-config.md';

<MutableGenerativeConfig />

xAI の生成 AI モデルを使用するには、次のように [Weaviate インデックスを設定](../../manage-collections/generative-reranker-models.mdx#specify-a-generative-model-integration) します。

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

### モデルの選択

Weaviate が使用する [利用可能なモデル](#available-models) のいずれかを、次の設定例のように指定できます。

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

以下の生成パラメーターを設定して、モデルの挙動をカスタマイズします。

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

モデルパラメーターの詳細については、[xAI API ドキュメント](https://docs.x.ai/docs/guides/chat#parameters)を参照してください。

## 実行時のモデル選択

コレクションの作成時にデフォルトのモデルプロバイダーを設定するだけでなく、クエリ実行時に上書きすることもできます。

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

生成 AI 連携を設定した後、[シングルプロンプト](#single-prompt)または[グループタスク](#grouped-task)方式で RAG 操作を実行できます。

### シングルプロンプト

![Single prompt RAG integration generates individual outputs per search result](../_includes/integration_xai_rag.png)

検索結果内の各オブジェクトごとにテキストを生成する場合は、シングルプロンプト方式を使用します。

以下の例では、`limit` パラメーターで指定した `n` 件の検索結果それぞれに対して出力を生成します。

シングルプロンプトクエリを作成する際、`{}` を使って Weaviate から言語モデルへ渡したいオブジェクトプロパティを補間できます。たとえば、オブジェクトの `title` プロパティを渡す場合は、クエリ内に `{title}` を含めます。

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

![Grouped task RAG integration generates one output for the set of search results](../_includes/integration_xai_rag.png)

検索結果全体に対して 1 つのテキストを生成する場合は、グループタスク方式を使用します。

言い換えると、`n` 件の検索結果がある場合でも、生成モデルはグループ全体に対して 1 つの出力を生成します。

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

デフォルトモデルは `grok-2-latest` です。

## 追加リソース

### コード例

コレクションで統合を構成すると、 Weaviate のデータ管理および検索操作は他のコレクションとまったく同じように機能します。モデルに依存しない以下の例をご覧ください。

- [How-to: コレクション管理](../../manage-collections/index.mdx) と [How-to: オブジェクト管理](../../manage-objects/index.mdx) のガイドでは、データ操作 ( i.e. コレクションおよびその内部のオブジェクトの作成、読み取り、更新、削除 ) の方法を説明しています。
- [How-to: クエリ & 検索](../../search/index.mdx) のガイドでは、検索操作 ( i.e. ベクトル、キーワード、ハイブリッド ) と 検索拡張生成 の実行方法を説明しています。

### 参考資料

- [xAI API ドキュメント](https://docs.x.ai/docs/introduction)

import DocsFeedback from '/_includes/docs-feedback.mdx';

<DocsFeedback/>

