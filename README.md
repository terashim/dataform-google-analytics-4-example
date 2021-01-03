Dataform による Google アナリティクス 4 エクスポートデータの変換パイプライン
======================================================================

[Google アナリティクス 4 プロパティの BigQuery Export](https://support.google.com/analytics/answer/9358801?hl=ja) をデータソースとするデータ変換パイプラインの [Dataform](https://dataform.co/) サンプルプロジェクト

## 構成

### Dataform データセットの分類と規約

このプロジェクトではパイプラインを大きく以下の４段階に分ける。この段階に応じて Dataform データセット定義ファイルのフォルダ分け、およびテーブル接頭辞を定める。

1. **データソース**
    - フォルダ: [`definitions/source`](./definitions/source)
    - パイプラインの起点となるローデータを格納する。
2. **中間データ**
    - フォルダ: [`definitions/stage`](./definitions/stage)
    - 加工・整形途中のデータを格納する。
    - 基本的にパイプライン内部でのみ使用することとし、分析用途には使用しない。
    - テーブル接頭辞 `stg__` を付ける。
3. **データウェアハウス**
    - フォルダ: [`definitions/warehouse`](./definitions/warehouse)
    - 加工・整形済みデータを格納する。データマートに比べて粒度が細かく大規模なデータを持つ。
    - アドホック分析におけるクエリ対象となる。また、データマートの上流テーブルとして再加工される。
    - テーブル接頭辞 `dwh__` を付ける。
4. **データマート**
    - フォルダ: [`definitions/mart`](./definitions/mart)
    - 集約・軽量化されたデータを格納する。データウェアハウスに比べて粒度が粗く小規模なデータを持つ。
    - 目的に応じて作成される。
    - テーブル接頭辞 `dm__` を付ける。

### Google アナリティクス 4 データパイプラインの概要

このプロジェクトでは [Google アナリティクス 4 プロパティの BigQuery Export](https://support.google.com/analytics/answer/9358801?hl=ja) の出力テーブルをデータソースとし、そこからページビューイベントを抽出・加工したデータセット [`dwh__google_analytics_pageview_events`](./definitions/warehouse/dwh__google_analytics_pageview_events.sqlx) と、さらにそれを集約した日次レポート [`dm__google_analytics_daily_traffic`](./definitions/mart/dm__google_analytics_daily_traffic.sqlx) を作成する。

定義されるデータセットは以下の４つ。各データセットの詳細についてはそれぞれの SQLX ファイル内のドキュメントを参照のこと。

- データソース: [`events_*`](./definitions/source/events_wildcard.sqlx)
- 中間データ: [`stg__google_analytics_events`](./definitions/stage/stg__google_analytics_events.sqlx)
- ページビューイベント: [`dwh__google_analytics_pageview_eventsdwh__google_analytics_pageview_events`](./definitions/warehouse/dwh__google_analytics_pageview_events.sqlx)
- 日次トラフィックレポート: [`dm__google_analytics_daily_traffic`](./definitions/mart/dm__google_analytics_daily_traffic.sqlx)

このサンプルプロジェクトではウェブサイトのトラフィックデータ分析を想定しているため、ページビューイベントを抽出したデータセットを定義している。その他の分析（Eコマースやモバイルアプリの分析など）を行いたい場合はそれに適したデータセットを作成する。

### 設定項目

このリポジトリを fork して以下の値を変更すれば、異なる Google アナリティクス 4 プロパティや異なる BigQuery プロジェクトに対して同じパイプラインを構築できる。

- Google アナリティクス 4 プロパティ ID: `includes/constants.js` の `GA4_PROPERTY_ID` で指定
- Google アナリティクス 4 のエクスポート先 BigQuery プロジェクト ID: `includes/constants.js` の `GA4_EXPORT_PROJECT_ID` で指定
- パイプラインのデプロイ先 BigQuery プロジェクト ID: `dataform.json` の `defaultDatabase` で設定、または `environments.json` で上書き
- パイプラインのデプロイ先 BigQuery データセット ID: `dataform.json` の `defaultSchema` で設定、または `environments.json` で上書き

## 参考

- [Google アナリティクス 4 プロパティの BigQuery Export - アナリティクス ヘルプ](https://support.google.com/analytics/answer/9358801?hl=ja)
- [Dataform | Manage data pipelines in BigQuery](https://dataform.co/)
- Dataform ドキュメント 和訳: [GitHub terashim/dataform-docs-ja](https://github.com/terashim/dataform-docs-ja)
