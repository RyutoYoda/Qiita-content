---
title: ローカルLLMの使用 - OllamaとOpen WebUIの連携について解説
tags:
  - 生成AI
  - 大規模言語モデル
  - LLaMA
  - ollama
  - ローカルLLM
private: false
updated_at: '2024-07-05T11:15:10+09:00'
id: ecdfbef8c73aae64aa45
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

このブログでは、ローカル環境で大規模言語モデル（LLM）であるOllamaをOpen WebUIと連携させて使用する方法を紹介します。Docker Composeを使用して、簡単に環境を構築する手順を詳しく解説します。手順に従うことで、ローカルで安全かつ効率的にLLMを活用できるようになります。また、ドキュメントを読み込ませてRAG（Retrieval-Augmented Generation）などの機能を使用する方法についても触れます。

### 前提条件

- Dockerがインストールされていること
- Docker Composeがインストールされていること
- NVIDIAのGPUを使用する場合、適切なドライバとCUDAがインストールされていること

## ステップ 1: Ollamaのインストールと実行
![スクリーンショット 2024-07-02 14.19.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/3eee9505-b862-b017-44c3-e8e4d561141b.png)
まず、Ollamaをローカル環境にインストールし、モデルを起動します。インストール完了後、以下のコマンドを実行してください。llama3のところは自身が使用したい言語モデルを選択してください。

```sh
% ollama run llama3
```

このコマンドにより、llama3モデルが起動し、コマンドライン上でllama3を使用できるようになります。

![スクリーンショット 2024-07-04 14.00.13.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/5572f538-f636-48e1-a28f-36bfdf0aee14.png)
## ステップ 2: プロジェクトフォルダの作成

次に、Docker Compose用のプロジェクトフォルダを作成します。ターミナルで以下のコマンドを実行してください。

```sh
mkdir ollama-webui
cd ollama-webui
```

## ステップ 3: 既存のコンテナの停止と削除

既に動作しているDockerコンテナがある場合、それを停止して削除します。以下のコマンドを実行してください。

```sh
docker stop $(docker ps -q)
docker rm $(docker ps -q -a)
```

## ステップ 4: Docker Composeファイルの作成

次に、Docker Composeを使用してOllamaとOpen WebUIを立ち上げるための設定ファイルを作成します。プロジェクトディレクトリに`docker-compose.yml`ファイルを作成し、以下の内容を記述します。

```yaml
version: '3.8'

services:
  ollama:
    image: ollama/ollama
    container_name: ollama
    volumes:
      - ollama:/root/.ollama
    ports:
      - "11434:11434"
    restart: unless-stopped
    # GPUを使用する場合、以下の行のコメントを外してください
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - capabilities: [gpu]

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3000:8080"
    volumes:
      - open-webui:/app/backend/data
    extra_hosts:
      - "host.docker.internal:host-gateway"
    restart: always

volumes:
  ollama:
  open-webui:
```

## ステップ 5: Docker Composeでコンテナを起動

作成した`docker-compose.yml`ファイルを使用して、OllamaとOpen WebUIのコンテナを起動します。以下のコマンドを実行してください。

```sh
docker-compose up -d
```

このコマンドにより、必要なイメージがダウンロードされ、OllamaとOpen WebUIのコンテナがバックグラウンドで起動します。
![スクリーンショット 2024-07-05 10.24.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/358cb5ca-2f06-7112-fe33-35bdca3d9338.png)

## ステップ 6: Open WebUIへのアクセス

コンテナが正常に起動したら、ブラウザで以下のURLにアクセスしてOpen WebUIを開きます。

```
http://localhost:3000/
```

初回アクセス時には、以下のような登録画面が表示されます。適当なユーザー名とメールアドレスを入力して登録を完了します。これらの情報はローカル環境でのみ使用され、サーバには送信されません。
![スクリーンショット 2024-07-05 10.32.24.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/a684b561-ea32-ea31-154f-2b404d9418bc.png)

## ステップ 7: モデルの選択と使用

Open WebUI上でモデルを選択し、実際に使用します。以下の手順に従ってください。

1. Open WebUIのインターフェースにアクセスします。
![スクリーンショット 2024-07-05 10.34.19.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/7182f88d-e4e2-42ff-9129-143f565571a2.png)
2. モデルの選択画面で、利用したいモデル（例：llama3）を選択します。
![スクリーンショット 2024-07-05 10.34.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/60641321-39a4-6cd8-ad1e-df6e6e6584f3.png)
3. 選択したモデルに基づいて、テキスト生成や質問応答を行います。テキストボックスに質問やプロンプトを入力し、モデルの出力を確認します。
![スクリーンショット 2024-07-05 10.45.12.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/2b5b485f-44ae-ff78-97ad-a28fbe40085d.png)


#### おまけ: ドキュメントを読み込ませてRAGを使用

Open WebUIでは、ドキュメントを読み込ませてRAG（Retrieval-Augmented Generation）を実行することもできます。これにより、特定のドキュメントやデータに基づいた質問応答が可能になります。
![スクリーンショット 2024-07-05 10.42.44.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/305bba6f-77b1-a10a-6128-c063a4c5450c.png)

## まとめ

以上で、ローカル環境でOllamaをOpen WebUIと連携させて使用するための設定が完了しました。Docker Composeを使用することで、簡単に環境を構築し、必要なモデルを実行できます。これにより、データのプライバシーを保ちながら、高性能なLLMを活用することが可能です。さらに、ドキュメントを読み込ませてRAG機能を使用することで、特定の情報に基づいた高度な質問応答が実現できます。

### 参考文献

- [Docker Documentation](https://docs.docker.com/)
- [Ollama GitHub Repository](https://github.com/ollama/ollama)
- [Open WebUI GitHub Repository](https://github.com/open-webui/open-webui)

