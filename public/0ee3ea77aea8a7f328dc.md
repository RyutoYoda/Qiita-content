---
title: BigQueryでのパーティションテーブル作成とクエリ
tags:
  - BigQuery
  - パーティション
private: false
updated_at: '2024-06-13T13:18:02+09:00'
id: 0ee3ea77aea8a7f328dc
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
BigQueryでパーティションテーブルを作成し、クエリを実行する方法について説明します。パーティションテーブルを使用することで、大規模なデータセットのクエリパフォーマンスを向上させ、コストを削減することができます。今回は、サンプルデータを作成し、それを月ごとにパーティション化する方法を紹介します。

## サンプルデータの作成

まず、サンプルデータを作成します。以下のSQLクエリを使用して、サンプルデータを含むテーブルを作成し、データを挿入します。

```sql
-- サンプルデータを作成するテーブルを作成
CREATE OR REPLACE TABLE `project123.sample_dataset.sample_data` (
  id INT64,
  name STRING,
  value INT64,
  created_date DATE
);

-- サンプルデータを挿入
INSERT INTO `project123.sample_dataset.sample_data` (id, name, value, created_date)
VALUES 
  (1, 'Alice', 10, DATE '2023-01-01'),
  (2, 'Bob', 20, DATE '2023-01-02'),
  (3, 'Charlie', 30, DATE '2023-01-03'),
  (4, 'David', 40, DATE '2023-02-01'),
  (5, 'Eve', 50, DATE '2023-02-02'),
  (6, 'Frank', 60, DATE '2023-03-01'),
  (7, 'Grace', 70, DATE '2023-03-02'),
  (8, 'Heidi', 80, DATE '2023-04-01'),
  (9, 'Ivan', 90, DATE '2023-04-02'),
  (10, 'Judy', 100, DATE '2023-05-01');
```

このクエリは、`project123.sample_dataset`データセット内に`sample_data`というテーブルを作成し、10件のサンプルデータを挿入します。各レコードには、ID、名前、値、作成日が含まれています。
![スクリーンショット 2024-06-13 13.02.57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/ffadda2b-bc0e-40a3-1e72-d5feefd2a072.png)


### パーティションテーブルの作成

次に、作成したサンプルデータを基に、パーティションテーブルを作成します。ここでは、`created_date`を基に月ごとにパーティションを作成します。

```sql
-- パーティションテーブルを作成
CREATE OR REPLACE TABLE `project123.sample_dataset.partitioned_sample_data`
PARTITION BY DATE_TRUNC(created_date, MONTH) AS
SELECT id, name, value, created_date
FROM `project123.sample_dataset.sample_data`;
```

このクエリは、`created_date`を基に月ごとのパーティションを作成します。`DATE_TRUNC`関数を使用することで、日付を月単位に切り捨て、パーティション化を行います。
![スクリーンショット 2024-06-13 13.11.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/2ce92e93-0e2e-a359-6bdd-cfece06c5318.png)
テーブルの見た目に違いはありません。
![スクリーンショット 2024-06-13 13.15.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/4bed88e7-04af-b8d7-a31c-f90f64665c1b.png)

## パーティションテーブルのクエリ

パーティションテーブルをクエリする際には、特定のパーティションのみを対象とすることで、クエリのパフォーマンスを向上させることができます。以下のクエリでは、特定の日付のパーティションを対象としています。

```sql
-- 特定のパーティションをクエリ
SELECT * FROM `project123.sample_dataset.partitioned_sample_data`
WHERE created_date = '2023-02-01';
```
このクエリは、`created_date`が2023年2月1日のデータのみを取得します。パーティション化されたテーブルを使用することで、大量のデータから特定の範囲を効率的に抽出できます。
![スクリーンショット 2024-06-13 13.06.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/d2da628d-2d8e-9cfc-f8ae-ae8d1ecb4181.png)


## パーティションテーブルの利点

パーティションテーブルを使用することで、次のような利点があります。

1. **クエリパフォーマンスの向上**: 必要なデータのみをスキャンするため、クエリの実行速度が向上します。
2. **コストの削減**: スキャンするデータ量が減るため、クエリ実行にかかるコストを削減できます。
3. **管理の容易さ**: データを特定の範囲で分割することで、大規模データセットの管理が容易になります。

## 結論

BigQueryのパーティションテーブルは、大規模なデータセットのクエリパフォーマンスを向上させるための強力なツールです。今回紹介した方法を参考にして、パーティションテーブルを活用し、効率的なデータ処理を実現してください。
