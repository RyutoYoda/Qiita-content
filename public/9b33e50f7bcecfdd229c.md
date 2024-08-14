---
title: Looker view定義の基本を理解
tags:
  - SQL
  - BI
  - データエンジニアリング
  - Looker
private: false
updated_at: '2024-06-19T11:06:57+09:00'
id: 9b33e50f7bcecfdd229c
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

Lookerでは、ビューを定義することによってデータベースのテーブルをモデル化し、データの可視化や分析を行うことができます。ビューの定義には、プライマリキーの指定や各カラムの定義、計算フィールド、メジャーの定義などが含まれます。この記事では、Lookerでのビューの定義方法について解説します。

## ビューの基本構造

ビューの定義は以下のような基本構造を持ちます：

1. `view` ブロック: ビューの名前とテーブル名を指定します。
2. `dimension` ブロック: 各カラムをディメンションとして定義します。
3. `measure` ブロック: 集計関数を使用したメジャーを定義します。

## プライマリキーの指定

ビューには必ず1つのプライマリキーを指定します。プライマリキーは各レコードを一意に識別するために使用されます。

### 例: `product_data` ビューの定義

以下は、架空の `product_data` テーブルをモデル化したビューの定義例です：

```lookml
view: product_data {
  sql_table_name: pd.product_data ;;

  dimension: product_id {
    primary_key: yes
    type: string
    sql: ${TABLE}."product_id" ;;  # ここを実際のプライマリキーのカラム名に変更してください
    label: "プロダクトID"
  }

  dimension: product_name {
    type: string
    sql: ${TABLE}."product_name" ;;
    label: "プロダクト名"
  }

  dimension: category {
    type: string
    sql: ${TABLE}."category" ;;
    label: "カテゴリ"
  }

  dimension: price {
    type: number
    sql: ${TABLE}."price" ;;
    label: "価格"
  }

  measure: total_sales {
    type: sum
    sql: ${TABLE}."sales" ;;
    label: "総売上"
  }

  measure: count {
    type: count
    label: "カウント"
  }
}
```

## 他に追加するべき情報

1. **ラベル (`label`) の追加**: 各ディメンションとメジャーに人間が理解しやすいラベルを付けます。
2. **説明 (`description`) の追加**: ディメンションやメジャーに対して詳細な説明を追加することで、他のユーザーがそのフィールドの目的や内容を理解しやすくします。
3. **フィルター (`filter`) の追加**: 特定の条件に基づいてデータをフィルタリングするフィールドを追加できます。
4. **パラメータ (`parameter`) の追加**: ユーザーが動的に値を入力できるようにするためのパラメータを追加できます。

### 例: 説明とフィルターの追加

```lookml
view: product_data {
  sql_table_name: pd.product_data ;;

  dimension: product_id {
    primary_key: yes
    type: string
    sql: ${TABLE}."product_id" ;;
    label: "プロダクトID"
    description: "各プロダクトを一意に識別するためのID"
  }

  dimension: product_name {
    type: string
    sql: ${TABLE}."product_name" ;;
    label: "プロダクト名"
    description: "プロダクトの名前"
  }

  dimension: category {
    type: string
    sql: ${TABLE}."category" ;;
    label: "カテゴリ"
    description: "プロダクトが属するカテゴリ"
  }

  dimension: price {
    type: number
    sql: ${TABLE}."price" ;;
    label: "価格"
    description: "プロダクトの価格"
  }

  measure: total_sales {
    type: sum
    sql: ${TABLE}."sales" ;;
    label: "総売上"
    description: "プロダクトの総売上額"
  }

  measure: count {
    type: count
    label: "カウント"
    description: "プロダクトの数をカウント"
  }

  filter: price_range {
    type: number
    sql: ${TABLE}."price" ;;
    label: "価格範囲"
    description: "特定の価格範囲のプロダクトをフィルタリング"
  }
}
```

## 終わりに

Lookerでのビューの定義は、データの可視化や分析を効率的に行うための重要なステップです。プライマリキーの指定、ラベルの追加、説明やフィルターの設定などを行うことで、他のユーザーがビューをより理解しやすく、使いやすくなります。このガイドを参考にして、自分のデータに応じたビューの定義を行ってください。
