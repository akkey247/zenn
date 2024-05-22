---
title: "Mac で Docker を使って Style-Bert-VITS2 を動かす"
emoji: "🔊"
type: "tech"
topics: [tts,音声合成]
published: true
---
# はじめに

感情豊かな音声合成ができると噂の Style-Bert-VITS2 を試してみたいと思ったのですが、どうやらWindows用らしく自分のMacでは使えないっぽい。
でも Docker を使えば動くらしいとの情報を頼りに頑張ったらできるようになったので、やり方をまとめました。
結果的に Dockerfile と docker-compose.yml を書いて起動するだけのお手軽構成でいけました。
学習はできませんが、音声合成を試してみることが可能です。Macで試したい方はぜひ。

# Dockerfile を作成

下記の内容で `Dockerfile` を作成する。

```dockerfile
FROM python:3.11-slim

# 作業ディレクトリを設定
WORKDIR /app

# 必要なツールのインストール
RUN apt-get update && \
    apt-get install -y git gcc

# リポジトリをクローン
RUN git clone https://github.com/litagin02/Style-Bert-VITS2.git

# 作業ディレクトリをクローンしたリポジトリの配下に移動
WORKDIR /app/Style-Bert-VITS2

# 初期設定
RUN python -m venv venv && \
    . ./venv/bin/activate && \
    pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118 && \
    pip install -r requirements.txt && \
    python initialize.py
```

# docker-compose.yml を作成

下記の内容で `docker-compose.yml` を作成する。

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
    command: bash -c ". venv/bin/activate && python server_editor.py --inbrowser --device cpu"
```

# Docker コンテナを起動

下記のコマンドで Docker コンテナを起動する。

```
$ docker-compose.yml up
```

起動したら `localhost:8000` にアクセスすれば Style-Bert-VITS2 を使うことができます。

最初は「こんにちは」くらいの短いテキストで試してみることをお勧めします。
自分の場合「おはようございます。良い天気ですね。」みたいなテキストを書いたら数分待たされたので。。

# 参考記事

[litagin02/Style-Bert-VITS2](https://github.com/litagin02/Style-Bert-VITS2/blob/master/README.md)  
[Style-Bert-VITS2をDockerでワンパン構築してみた](https://hamaruki.com/i-tried-building-style-bert-vits2-in-one-with-docker/)  
[Style-Bert-VITS2をDockerで動かす話](https://qiita.com/okada1220/items/6fe462fa13f116ec4a7c)  