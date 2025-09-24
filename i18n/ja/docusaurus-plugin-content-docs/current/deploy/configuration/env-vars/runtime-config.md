---
title: ランタイム構成管理
sidebar_label: Runtime configuration
sidebar_position: 1
image: og/docs/configuration.jpg
---

:::info Added in `v1.30`
:::

Weaviate はランタイム構成管理をサポートしており、特定の環境変数を再起動なしで動的に更新・読み取りできます。この機能により、設定をリアルタイムで調整し、ニーズの変化に合わせてインスタンスを微調整できます。

## ランタイム構成のセットアップ

ランタイム構成管理をセットアップするには、次の手順に従ってください。

1.  **機能を有効化する**  
    環境変数 `RUNTIME_OVERRIDES_ENABLED` を `true` に設定します。

2.  **オーバーライド ファイルを用意する**  
    ランタイム オーバーライドを含む構成ファイルを作成し、`RUNTIME_OVERRIDES_PATH` 変数でそのファイルを指定します。

    <details>
    <summary>構成オーバーライド ファイルの例</summary>

    ```yaml title="overrides.yaml"
    maximum_allowed_collections_count: 8
    autoschema_enabled: true
    async_replication_disabled: false
    ```

    </details>

3.  **更新間隔を設定する**  
    `RUNTIME_OVERRIDES_LOAD_INTERVAL` 変数で、Weaviate が構成変更を確認する頻度を設定します（デフォルトは `2m`）。

4.  **インスタンスを再起動する**  
    設定を反映させるため、Weaviate インスタンスを再起動します。

### 構成変数

ランタイム構成管理を制御するために使用する環境変数は次のとおりです。

| 変数                              | 説明                                                                                         | 型                    |
| :-------------------------------- | :------------------------------------------------------------------------------------------- | :-------------------- |
| `RUNTIME_OVERRIDES_ENABLED`       | 設定するとランタイム構成管理が有効になります。デフォルト: `false`                            | `boolean`             |
| `RUNTIME_OVERRIDES_PATH`          | 構成オーバーライド ファイルのパス。                                                          | `string - file path`  |
| `RUNTIME_OVERRIDES_LOAD_INTERVAL` | 構成変更を確認する更新間隔。デフォルト: `2m`                                                 | `string - duration`   |

## ランタイムで変更可能な環境変数

以下は、ランタイムで変更できる環境変数と、オーバーライド ファイルで使用する名前の一覧です。

| 環境変数名                          | ランタイム オーバーライド名       |
| :---------------------------------- | :-------------------------------- |
| `ASYNC_REPLICATION_DISABLED`        | `async_replication_disabled`       |
| `AUTOSCHEMA_ENABLED`                | `autoschema_enabled`               |
| `MAXIMUM_ALLOWED_COLLECTIONS_COUNT` | `maximum_allowed_collections_count` |

各変数の詳細については、[Environment variables](./index.md) ページをご覧ください。

## 運用とモニタリング

ランタイム構成は構成ファイルの変更を監視する仕組みに基づいており、運用上の考慮事項があります。  
無効なランタイム構成ファイル（例: 不正な YAML）で Weaviate を起動しようとすると、プロセスは起動に失敗して終了します。

稼働中の Weaviate インスタンスのランタイム構成ファイルを変更した際、新しい構成が無効だった場合でも、Weaviate はメモリに保持している最後に有効だった構成を引き続き使用します。構成の読み込み失敗はエラーログとメトリクスで確認できます。

### メトリクス

Weaviate はランタイム構成の状態を監視するために、次の [メトリクス](../../configuration/monitoring.md) を提供します。

| メトリクス名                                  | 説明                                                                                           |
| :-------------------------------------------- | :--------------------------------------------------------------------------------------------- |
| `weaviate_runtime_config_last_load_success`   | 直近の読み込みが成功したかどうかを示します（成功で `1`、失敗で `0`）                           |
| `weaviate_runtime_config_hash`                | 現在アクティブなランタイム構成のハッシュ値。新しい構成が反映されたタイミングを追跡するのに役立ちます |

### ログ

Weaviate はランタイム構成の変更を監視し、トラブルシューティングを行うための詳細なログを出力します。

#### 構成変更

ランタイム構成の値が正常に更新されると、`INFO` ログが出力されます。例:

```
runtime overrides: config 'MaximumAllowedCollectionsCount' changed from '-1' to '7'  action=runtime_overrides_changed field=MaximumAllowedCollectionsCount new_value=7 old_value=-1
```

#### 構成バリデーション エラー

Weaviate の稼働中に無効な構成が検出されると、`ERROR` ログが出力されます。例:

```
loading runtime config every 2m failed, using old config: invalid yaml
```

### フェール モード

ランタイム構成管理は、データ破損やサイレント フェールを防ぐため、「早期検出・即時停止（fail early, fail fast）」の原則に従っています。

**1. 無効な構成での起動** - 無効なランタイム構成ファイルで Weaviate を起動しようとすると、プロセスは起動に失敗して終了します。これにより、不正な設定で Weaviate が動作することは決してありません。

**2. 稼働中の無効な構成** - Weaviate が稼働している間にランタイム構成ファイルが無効になった場合:

- Weaviate はメモリに保持している最後に有効だった構成を引き続き使用します  
- エラーログとメトリクスで構成の読み込み失敗を示します  
- Weaviate がクラッシュするかメモリ不足になった場合、構成が修正されるまで再起動に失敗します  

この設計により、ランタイム オーバーライドの読み込みに失敗した際に環境変数のデフォルトへフォールバックすることを防ぎ、想定外の動作やデータの問題を回避します。

以下は一例です:

1.  環境変数 `MAXIMUM_ALLOWED_COLLECTIONS_COUNT` が 10 に設定されている  
2.  ランタイム構成 `MaximumAllowedCollectionsCount` で 4 にオーバーライドされている  
3.  しばらくしてランタイム構成ファイルが無効になる  
4.  Weaviate は稼働中、最後に有効だった値（4）を使用し続ける  
5.  Weaviate がクラッシュした場合、構成ファイルが修正されるまで再起動に失敗する  
6.  これにより、誤った環境変数のデフォルト値（10）で起動してしまうことを防ぐ  

以上の理由から、提供されているメトリクスとログを基にしたモニタリングとアラートを設定し、構成問題を早期に検知・解決することが重要です。

