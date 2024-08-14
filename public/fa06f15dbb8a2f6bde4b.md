---
title: ' Lookerでのモデル作成の基本'
tags:
  - SQL
  - Looker
  - LookML
private: false
updated_at: '2024-06-20T13:10:45+09:00'
id: fa06f15dbb8a2f6bde4b
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
Lookerはデータを効果的に可視化し、分析するための強力なツールです。このガイドでは、Lookerでの基本的なモデル作成手順を紹介します。架空のテーブル名と列名を使用し、簡潔なコードサンプルを交えて説明します。
## 1. データベース接続の設定

まず、Lookerがデータベースに接続できるように設定を行います。接続設定は、Lookerの管理コンソールで行います。接続名を `example_connection` としましょう。

1. Looker管理コンソールにログインします。
2. **Admin** -> **Connections** をクリックします。
3. **New Connection** をクリックし、接続情報を入力して保存します。

## 2. モデルファイルの作成

モデルファイルでは、データベース接続とインクルードするビューを指定します。

#### model.lkml

```lookml
connection: "example_connection" #　※ここを変えると繋がらなくなる可能性があるので、注意

include: "/views/*.view.lkml"  # views フォルダー内の全てのビューをインクルード

explore: orders {
  join: customers {
    relationship: many_to_one  # 多対一のリレーションシップを指定
    sql_on: ${orders.customer_id} = ${customers.id} ;;
  }
}
```

#### リレーションシップについて

`relationship` は、テーブル間のデータの関連性を指定します。Lookerでは以下のリレーションシップタイプを使用します：

- `many_to_one`: 多対一の関係。例えば、`orders` テーブルの複数の行が `customers` テーブルの一つの行に関連付けられている場合に使用します。
- `one_to_one`: 一対一の関係。例えば、`users` テーブルと `user_profiles` テーブルの各行が一対一で対応する場合に使用します。
- `many_to_many`: 多対多の関係。複数の行が互いに関連している場合に使用しますが、Lookerでは一般的に避けるべきです。

また、`sql_on` にはリレーションシップを確立するためのSQL条件を指定します。通常、主キーと外部キーを使用してテーブルを結合します。例えば、`orders` テーブルの `customer_id` 列と `customers` テーブルの `id` 列を結合します。

## 3. ビューファイルの作成

ビュー定義では、テーブル名と各列をディメンションやメジャーとして指定します。

### orders.view.lkml

```lookml
view: orders {
  sql_table_name: my_database.orders ;;  # 実際のテーブル名を指定

  dimension: id {
    primary_key: yes
    type: number
    sql: ${TABLE}.id ;;
    label: "Order ID"
  }

  dimension: order_date {
    type: date
    sql: ${TABLE}.order_date ;;
    label: "Order Date"
  }

  dimension: customer_id {
    type: number
    sql: ${TABLE}.customer_id ;;
    label: "Customer ID"
  }

  measure: total_amount {
    type: sum
    sql: ${TABLE}.order_amount ;;
    label: "Total Amount"
  }
}
```

### customers.view.lkml

```lookml
view: customers {
  sql_table_name: my_database.customers ;;  # 実際のテーブル名を指定

  dimension: id {
    primary_key: yes
    type: number
    sql: ${TABLE}.id ;;
    label: "Customer ID"
  }

  dimension: name {
    type: string
    sql: ${TABLE}.name ;;
    label: "Customer Name"
  }
}
```

## 4. モデルとビューのデプロイ

1. **Develop** -> **Manage LookML Projects** に移動します。
2. 該当するプロジェクトを選択します。
3. **Validate LookML** ボタンをクリックして、エラーがないことを確認します。
4. **Deploy to Production** をクリックしてデプロイします。

## 5. Exploreの使用

デプロイが完了したら、LookerのUIで新しいExploreを使用してデータを可視化します。

1. **Explore** メニューから **orders** を選択します。
2. 必要なディメンションやメジャーを選択して、データを表示します。

## まとめ

以上が、Lookerでの基本的なモデル作成手順です。これで、データベース接続の設定からモデルとビューの定義、デプロイまでの流れが理解できたと思います。適切なリレーションシップを定義することで、複数のテーブルからデータを引き出し、効果的に可視化することが可能になります。
