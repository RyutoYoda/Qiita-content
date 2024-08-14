---
title: 感情分析とクラスタリングを用いたレビュー分析アプリを作ってみた
tags:
  - Python
  - 主成分分析
  - クラスタリング
  - SentimentAnalysis
  - 埋め込みベクトル
private: false
updated_at: '2024-08-14T10:15:45+09:00'
id: dc0abadadf4592cf087a
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

今回は、PythonとStreamlitライブラリを使用したレビュー分析アプリを構築する方法について解説します。基本的なアプリの説明として、ユーザーからアップロードされたレビューデータをテキスト処理し、感情分析とクラスタリングを行えるよう設計しました。さらに、結果を3次元プロットで可視化し、データをダウンロードする機能も備えています。

#### 実際に作成したアプリがこちらになります。ご自身の持つデータでも試してみてください。
https://reviewanalysisapp-lvayjg67ksjimoextf4mqy.streamlit.app/


## 使用するライブラリ

このアプリでは、以下のライブラリを使用しています。

- **Streamlit**: インタラクティブなウェブアプリケーションを簡単に作成するためのライブラリです。コードの各部分を実行しながら、即座にその結果をウェブブラウザに表示できます。
  
- **NumPy**: 数値計算を効率的に行うためのライブラリです。特にベクトルや行列操作に強力で、多くの機械学習アルゴリズムで使用されます。
  
- **Pandas**: データ解析に特化したライブラリで、データフレーム形式でデータを操作することができます。CSVやExcelファイルの読み込み、データのフィルタリング、集計などが簡単に行えます。
  
- **SentenceTransformers**: テキストの意味を保持しながらベクトルに変換するためのライブラリです。特に、事前学習済みのモデル（今回は`all-MiniLM-L6-v2`）を使用してテキスト埋め込みを生成します。
  
- **scikit-learn**: 機械学習アルゴリズムを提供するライブラリで、このアプリではクラスタリング（KMeans）や次元削減（PCA）に使用しています。
  
- **Plotly**: インタラクティブなグラフを作成するためのライブラリです。特に、3次元プロットの作成に優れています。
  
- **SnowNLP**: 中国語のテキスト処理に特化したライブラリですが、日本語や英語の感情分析にも応用可能です。感情スコアを0から1の範囲で計算し、テキストのポジティブ/ネガティブを判定します。
  
- **re**: 正規表現を使用してテキストデータを前処理するためのライブラリです。

## アプリの設定と初期化

アプリの設定は以下のコードで行っています。

```python
import streamlit as st

st.set_page_config(page_title="Review Analysis App", page_icon="📊")
```

`st.set_page_config`は、Streamlitアプリのタイトルやアイコン、レイアウトなどの基本設定を行います。この場合、アプリのタイトルを「Review Analysis App」、アイコンを「📊」に設定しています。

## スタイル設定

次に、HTMLとCSSを使用してアプリの見た目を整えます。

```python
st.markdown("""
<style>
body {
    font-family: 'Helvetica Neue', sans-serif;
}
</style>
""", unsafe_allow_html=True)
```

ここでは、アプリのフォントを変更しています。`unsafe_allow_html=True`を指定することで、HTMLやCSSを安全に適用できます。

## テキストの前処理

テキストデータを分析する前に、データをクリーンアップする必要があります。これには、テキストを小文字に変換したり、不要な文字や空白を削除することが含まれます。

```python
def preprocess_text(text):
    text = text.lower()  # 小文字に変換
    text = re.sub(r'\d+', '', text)  # 数字を削除
    text = re.sub(r'\s+', ' ', text)  # 不要な空白を削除
    text = re.sub(r'[^\w\s]', '', text)  # 特殊文字を削除
    return text
```

`preprocess_text`関数では、正規表現を用いてテキストデータをクリーンアップしています。`re.sub`関数を使うことで、テキスト中の数字や特殊文字を削除し、余分な空白を削除しています。

## ファイルのアップロードとデータフレームの作成

アプリでは、ユーザーがCSVまたはExcelファイルをアップロードし、それをPandasデータフレームとして読み込みます。

```python
uploaded_file = st.file_uploader("ファイルをアップロードしてください", type=["csv", "xlsx"])

if uploaded_file:
    if uploaded_file.name.endswith('.csv'):
        df = pd.read_csv(uploaded_file)
    else:
        df = pd.read_excel(uploaded_file)
    
    st.write("データプレビュー：", df.head())
```

この部分では、`st.file_uploader`を使用してファイルをアップロードし、`pandas.read_csv`または`pandas.read_excel`でデータを読み込みます。アップロードされたデータは、最初の5行を表示してプレビューします。

今回は擬似的に、ECサイトを想定した口コミデータを作成してみました。
![スクリーンショット 2024-08-13 18.16.14.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/1aa0dde1-961c-b48d-69ad-7d09ff4bcbf8.png)

## 埋め込みベクトルの生成

テキストをベクトルに変換するために、事前学習済みのSentenceTransformerモデルを使用します。

```python
if st.button('埋め込みベクトルを生成'):
    try:
        with st.spinner('埋め込みベクトルを生成中...'):
            model = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")
            st.session_state.embeddings = model.encode(st.session_state.df[review_column].astype(str).tolist())
        
        st.success('埋め込みベクトルの生成が完了しました！')
    
    except Exception as e:
        st.error("埋め込みベクトルの生成に失敗しました。")
        st.error(str(e))
```
この部分では、`SentenceTransformer`を用いてテキストデータをベクトルに変換します。`encode`メソッドは、テキストを数値ベクトルに変換し、各テキストの意味的な情報を数値で表現します。

## クラスタリングと次元削減

ベクトル化されたテキストをクラスタリングし、PCA（主成分分析）を用いて次元削減を行います。

```python
if st.session_state.embeddings is not None:
    if st.button('クラスタリングと3次元プロットを実行'):
        try:
            # クラスタリングを実行
            kmeans = KMeans(n_clusters=st.session_state.num_clusters, random_state=42)
            st.session_state.df['cluster'] = kmeans.fit_predict(st.session_state.embeddings)
            
            # PCAを使用して3次元に可視化
            pca = PCA(n_components=3)
            pca_result = pca.fit_transform(st.session_state.embeddings)
            st.session_state.df['pca_one'] = pca_result[:, 0]
            st.session_state.df['pca_two'] = pca_result[:, 1]
            st.session_state.df['pca_three'] = pca_result[:, 2]
            
            # クラスタの色を指定
            color_sequence = px.colors.qualitative.T10
            fig = px.scatter_3d(
                st.session_state.df, x='pca_one', y='pca_two', z='pca_three',
                color='cluster', hover_data=[review_column],
                color_discrete_sequence=color_sequence[:st.session_state.num_clusters]
            )
            st.session_state.fig = fig
            st.plotly_chart(st.session_state.fig, use_container_width=True)
        
        except Exception as e:
            st.error("クラスタリングとプロットに失敗しました。")
            st.error(str(e))
```

`KMeans`は、指定されたクラスタ数に基づいてデータをクラスタリングします。次に、`PCA`を用いて高次元のデータを3次元に縮約し、`plotly`を使って3次元プロットを作成します。

![スクリーンショット 2024-08-13 18.15.17.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/d30a9c2c-82fa-8d59-7227-a51cb28f7bcf.png)

## 感情分析

レビューの感情を分析するために、SnowNLPライブラリを使用します。

```python
if st.session_state.embeddings is not None and st.button('感情分析を実行'):
    try:
        st.session_state.df['sentiment_score'] = st.session_state.df[review_column].astype(str).apply(lambda x: 2 * SnowNLP(x).sentiments - 1)
        st.session_state.df['sentiment'] = st.session_state.df['

sentiment_score'].apply(lambda x: 'positive' if x > 0 else 'negative')
        
        st.write("Sentiment Analysis結果：")
        st.write(st.session_state.df[[review_column, 'sentiment', 'sentiment_score']])
    except Exception as e:
        st.error("感情分析中にエラーが発生しました。")
        st.error(str(e))
```

`SnowNLP`は、レビューの感情スコアを計算し、テキストがポジティブかネガティブかを判定します。感情スコアは、おおよそ-1から1の範囲で計算され、0より大きい場合はポジティブ、小さい場合はネガティブと判定されます。

![スクリーンショット 2024-08-13 18.13.16.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/bd0b68ea-ec02-c3bf-5c6d-2e0015f9e50a.png)

以下は、ポジティブレビューを赤色、ネガティブレビューを青色で表示しています。

![スクリーンショット 2024-08-13 18.17.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/aa3bab54-6996-aef2-0222-b517168e6f65.png)

## データのダウンロード

最後に、分析結果をCSVファイルとしてダウンロードする機能を実装しています。

```python
if st.session_state.embeddings is not None and st.session_state.df is not None:
        try:
            def convert_df_to_csv(df):
                return df.to_csv(index=False).encode('utf-8')

            csv = convert_df_to_csv(st.session_state.df)
            st.download_button(
                label="データをCSVとしてダウンロード",
                data=csv,
                file_name='review_analysis_result.csv',
                mime='text/csv',
            )
```

ここでは、PandasデータフレームをCSVファイルに変換し、それをダウンロードできるリンクを提供しています。`st.download_button`を使用して、ユーザーが簡単にデータをダウンロードして共有できるようにしています。

![スクリーンショット 2024-08-13 18.29.05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/87e236a9-a8c4-c9dc-e566-823ea0ac55fb.png)

![スクリーンショット 2024-08-13 18.28.21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3364428/ac4a6d36-a96a-cf91-bfc9-b11c9e3765f3.png)
## まとめ

このアプリは、ユーザーが簡単にテキストデータを分析し、クラスタリングや感情分析を行えるように設計しています。特に、自然言語処理に有効なライブラリを駆使して、データ処理を実現しています。この記事で紹介した手法を応用すれば、さまざまなデータセットに対して有効な分析を行うことができます。

