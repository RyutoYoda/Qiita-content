---
title: Cloud Storageにデータをアーカイブする方法：TNOアプローチの解説
tags:
  - CloudStorage
  - TNO
private: false
updated_at: '2024-06-22T13:37:52+09:00'
id: 0661bdb4c2fbf65fb2ff
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに 

データをGoogle Cloud Storageにアーカイブする際、特に機密性の高いデータを扱う場合には「Trust No One（TNO）」アプローチを使用することが推奨されます。このアプローチにより、クラウドプロバイダーのスタッフがデータを解読できないようにすることができます。本記事では、TNOアプローチを使用してデータを暗号化し、Cloud Storageに安全に保存する方法を解説します。

## 基本的な概念と原則

### Cloud Storage
Google Cloudのオブジェクトストレージサービスで、バイナリーデータなどの様々なデータを保存、取得することができます。耐久性とスケーラビリティがあり、データのアーカイブやバックアップに適しています。

### Trust No One（TNO）
機密データのセキュリティアプローチで、データを所有者自身が暗号化し、キーを保持します。クラウドプロバイダーや他の第三者がデータを解読できないようにすることを目的としています。

### 顧客指定の暗号化キー（CSEK）
Google Cloud Storageで使用できるセキュリティ機能の一つで、ユーザー自身が管理する暗号化キーを使用します。このキーはGoogleでは管理されず、ユーザーのみが管理します。

### .boto設定ファイル
gsutilコマンドラインツールの設定ファイルであり、CSEKなどの設定を指定することができます。

### gsutil
Google Cloud Storageと通信するためのPython製コマンドラインツールで、データのアップロードやダウンロードなどの操作を行います。

### gcloud kms keys
Google Cloud KMSで暗号キーを作成するコマンドです。キーのローテーションや暗号化、復号化が可能です。

## 要件を満たすためのステップ

### 1. gcloud kms keys createを使用して共通鍵を作成
まず、共通鍵を作成します。この鍵を使用してデータを暗号化します。

```bash
# KMSキーリングを作成
gcloud kms keyrings create my-key-ring --location global

# KMS暗号キーを作成
gcloud kms keys create my-key --location global --keyring my-key-ring --purpose encryption
```

### 2. gcloud kms encryptを使用して各アーカイブファイルを暗号化
次に、作成したKMSキーを使用してアーカイブファイルを暗号化します。

```bash
# ファイルを暗号化
gcloud kms encrypt --location global --keyring my-key-ring --key my-key \
  --plaintext-file my-archive-file.txt --ciphertext-file my-archive-file.txt.enc
```

### 3. gsutil cpを使って、暗号化された各ファイルをCloud Storageバケットにアップロード
暗号化されたファイルをCloud Storageにアップロードします。

```bash
# 暗号化されたファイルをアップロード
gsutil cp my-archive-file.txt.enc gs://my-bucket/
```

### 4. .boto設定ファイルに顧客指定の暗号化キー（CSEK）を指定
.boto設定ファイルにCSEKを指定して設定します。

```ini
# .boto設定ファイルの例
[GSUtil]
encryption_key = <base64-encoded encryption key>
```

### 5. gsutil cpを使用して、各アーカイブファイルをCloud Storageバケットにアップロード
先ほどと同じように、gsutil cpコマンドでファイルをアップロードします。この際、CSEKが適用されます。

```bash
# CSEKを使用してファイルをアップロード
gsutil cp my-archive-file.txt.enc gs://my-bucket/
```

### 6. セキュリティチームのみがアクセスできる別のプロジェクトにCSEKを保存
セキュリティチームのみがアクセス可能なプロジェクトにCSEKを安全に保存します。

```bash
# CSEKを保存するためのプロジェクトに移動
gcloud config set project my-security-project

# キーを保存（例：Secret Managerを使用）
echo -n "<base64-encoded encryption key>" | gcloud secrets create my-csek --data-file=-
```

## 結論

以上の手順で、機密データをTNOアプローチで暗号化し、Cloud Storageに安全に保存することができます。この方法により、クラウドプロバイダーのスタッフがデータを解読するリスクを排除し、データの機密性を保護することができます。
