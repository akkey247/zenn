---
title: "Remix環境構築: 1. Dockerコンテナ作成、Remixのインストール～表示まで"
emoji: "😸"
type: "tech"
topics: [Remix]
published: true
---

# はじめに

Remixをさわってみたいと思ってDocker上に Remix + Storybook + MUI の環境を構築したので、手順をまとめてみました。  
作って壊して試す用に作ったので、とりあえず欲しいパッケージだけつっこんでとりあえず動くとこまでという感じです。  
なるべく余計なコードや設定は書かずに必要最低限の内容で作成してます。  
ちなみに自分はReact自体にまだ慣れていないので、もしかすると所々おかしい部分があるかもしれません。変なところがあれば指摘いただけると助かります。  

# 1. プロジェクトディレクトリを作成

プロジェクト用のディレクトリを作成します。  

# 2. Docker環境作成

## `docker-compose.yml` を作成

プロジェクトディレクトリ内に `docker-compose.yml` を作ります。  

```yaml:docker-compose.yml
services:
  app:
    image: node:current-slim
    ports:
      - 3000:3000
      - 6006:6006
    environment:
      TZ: Asia/Tokyo
    working_dir: /usr/src/app
    volumes:
      - ./myapp:/usr/src/app
    stdin_open: true
    tty: true
```

__[補足]__  
- `ports` の設定は、 `- 3000:3000` はRemix用、 `- 6006:6006` はStorybook用のポート設定です。
- `working_dir` の設定でデフォルトの作業ディレクトリを指定してます。
- `volumes` の設定でホスト側の `myapp` というディレクトリがコンテナ内の `/usr/src/app` に同期されるようにしています。 `myapp` ディレクトリはコンテナを起動すると勝手に作成されるので事前に作っておく必要は無し。
- `node` のコンテナイメージは起動後すぐに終了してしまうので、 `stdin_open: true` と `tty: true` を入れることで起動し続けるようにしています。

## コンテナ起動

プロジェクトディレクトリ直下で下記のコマンドを実行してコンテナを起動します。  

```
$ docker-compose up -d
```

# 3. Remix を導入

## インストール

下記のコマンドで Remix をインストールします。  
`docker-compose exec` を使うとコンテナに入ってコマンドを実行しないで済むので楽です。  

```
$ docker-compose exec app npx create-remix@latest . --no-git-init
```

__[補足]__  
- `create-remix` では実行時に `git init` を行うか選択することが出来るが、今回使用している Docker イメージにはデフォルトでは Git が入っていないので `--no-git-init` を付けて `git init` を行わないようにする。 `git init` をしたい場合は事前にコンテナにGitをインストールしておく必要がある。
- `Install dependencies with npm?` と聞かれるのは `Yes` でOK。

## `package.json` を修正

`myapp/package.json` を下記のように修正します。  

```json:package.json
// 〜〜省略〜〜
"dev": "remix vite:dev --host --port 3000",
// 〜〜省略〜〜
```

__[補足]__
- `--host` を指定するのは、コンテナの外からのアクセスを許可してホスト側からアクセスできるようにするため。
- `--port 3000` でポート番号を3000番に設定している。3000版以外でもOK(ただし `docker-compose.yml` の `ports` の設定と合わせる必要がある)。

## Remix を起動

一旦うまくいっているかの確認のために Remix を起動して確認してみます。  
下記のコマンドで Remix を起動します。  

```
$ docker-compose exec app npm run dev
```

## Remix へのアクセスを確認

`localhost:3000` にアクセスして下記のようなページが表示されたらOKです。  

![](/images/20240417_remix_environment_construction_1__image1.png)

# 次の記事へ

この記事では、Dockerコンテナの作成とRemixのインストールまでを行いました。  
次の記事ではStorybookのインストールを行います。  