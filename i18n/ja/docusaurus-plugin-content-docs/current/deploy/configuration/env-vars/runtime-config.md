---
title: ランタイム構成管理
sidebar_label: ランタイム構成
sidebar_position: 1
image: og/docs/configuration.jpg
---

:::info ` v1.30 ` で追加
:::

Weaviate はランタイム構成管理をサポートしており、特定の環境変数を再起動なしで動的に更新・読み取りできます。この機能により、ニーズの変化に合わせてリアルタイムで設定を調整し、インスタンスを微調整できます。

## ランタイム構成のセットアップ

以下の手順でランタイム構成管理をセットアップします。

1.  **機能を有効化する**  
    環境変数 `RUNTIME_OVERRIDES_ENABLED` を `true` に設定します。

2.  **オーバーライドファイルを用意する**  
    ランタイムオーバーライドを含む設定ファイルを作成し、`RUNTIME_OVERRIDES_PATH` でそのパスを指定します。

    <details>
    <summary>構成オーバーライドファイルの例</summary>

    ```yaml title="overrides.yaml"
    maximum_allowed_collections_count: 8
    autoschema_enabled: true
    async_replication_disabled: false
    ```

    </details>

3.  **更新間隔を設定する**  
    `RUNTIME_OVERRIDES_LOAD_INTERVAL` に Weaviate が設定変更を確認する頻度を指定します（デフォルトは `2m`）。

4.  **インスタンスを再起動する**  
    セットアップを完了するために Weaviate インスタンスを再起動します。

### 構成変数

ランタイム構成管理を制御する環境変数は次のとおりです。

| Variable                          | Description                                                                                      | Type                 |
| :-------------------------------- | :------------------------------------------------------------------------------------------------ | :------------------- |
| `RUNTIME_OVERRIDES_ENABLED`       | 設定すると、ランタイム構成管理が有効になります。デフォルト: `false`                                 | `boolean`            |
| `RUNTIME_OVERRIDES_PATH`          | 構成オーバーライドファイルのパス                                                                   | `string - file path` |
| `RUNTIME_OVERRIDES_LOAD_INTERVAL` | 設定変更の有無を確認するリフレッシュ間隔。デフォルト: `2m`                                        | `string - duration`  |

## ランタイムで変更可能な環境変数

以下はランタイムで変更可能な環境変数と、オーバーライドファイルで使用する名前の一覧です。

| Environment variable name           | Runtime override name               |
| :---------------------------------- | :---------------------------------- |
| `ASYNC_REPLICATION_DISABLED`        | `async_replication_disabled`        |
| `AUTOSCHEMA_ENABLED`                | `autoschema_enabled`                |
| `MAXIMUM_ALLOWED_COLLECTIONS_COUNT` | `maximum_allowed_collections_count` |

各変数の詳細は [Environment variables](./index.md) ページをご覧ください。

## 運用と監視

ランタイム構成は設定ファイルの変更を監視する仕組みに基づいており、運用上の考慮が必要です。Weaviate が無効なランタイム構成ファイル（例: 不正な YAML）で起動しようとすると、プロセスは起動に失敗して終了します。

稼働中の Weaviate インスタンスでランタイム構成ファイルを変更し、その内容が無効だった場合、Weaviate はメモリに保持している最後に有効だった構成を使い続けます。エラーログとメトリクスにより構成読み込み失敗が示されます。

### メトリクス

Weaviate はランタイム構成の状態を監視するために以下の [メトリクス](../../configuration/monitoring.md) を提供します。

| Metric Name                                 | Description                                                                                                 |
| :------------------------------------------ | :----------------------------------------------------------------------------------------------------------- |
| `weaviate_runtime_config_last_load_success` | 直近の読み込みが成功したかどうかを示します（成功 `1`、失敗 `0`）                                             |
| `weaviate_runtime_config_hash`              | 現在アクティブなランタイム構成のハッシュ値。新しい構成が有効になったタイミングの追跡に役立ちます            |

### ログ

Weaviate はランタイム構成の変更を監視し、トラブルシューティングを支援するための詳細なログを提供します。

#### 構成変更

ランタイム構成値が正常に更新されると `INFO` ログが出力されます。例:

```
runtime overrides: config 'MaximumAllowedCollectionsCount' changed from '-1' to '7'  action=runtime_overrides_changed field=MaximumAllowedCollectionsCount new_value=7 old_value=-1
```

#### 構成検証エラー

Weaviate 稼働中に無効な構成が検出されると `ERROR` ログが出力されます。例:

```
loading runtime config every 2m failed, using old config: invalid yaml
```

### 障害モード

ランタイム構成管理は、データ破損やサイレントフェイルを防ぐために「早期・即時フェイル」ポリシーを採用しています。

**1. 無効な構成での起動**  
無効なランタイム構成ファイルで起動しようとすると、プロセスは起動に失敗して終了します。これにより、Weaviate が誤った設定で動作することを防ぎます。

**2. 稼働中の無効な構成**  
Weaviate 稼働中にランタイム構成ファイルが無効になった場合:

- Weaviate はメモリに保持している最後に有効だった構成を使用し続けます  
- エラーログとメトリクスで構成読み込み失敗が示されます  
- Weaviate がクラッシュしたりメモリ不足で停止した場合、構成が修正されるまで再起動に失敗します  

この設計により、ランタイムオーバーライドが失敗したときに環境変数のデフォルトにフォールバックして意図しない動作やデータ問題が発生するのを防ぎます。

以下は例となるシナリオです。

1.  環境変数 `MAXIMUM_ALLOWED_COLLECTIONS_COUNT` が 10 に設定されている  
2.  ランタイム構成 `MaximumAllowedCollectionsCount` がこれを 4 にオーバーライド  
3.  しばらくしてランタイム構成ファイルが無効になる  
4.  Weaviate は稼働中、最後に有効だった値（4）を使用し続ける  
5.  Weaviate がクラッシュすると、構成ファイルが修正されるまで再起動に失敗する  
6.  これにより、誤ったデフォルト値（10）で起動してしまうことを防止  

そのため、提供されているメトリクスとログを基に監視とアラートを設定し、構成問題をプロアクティブに特定・解決することが重要です。

