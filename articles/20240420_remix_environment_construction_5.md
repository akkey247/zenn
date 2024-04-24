---
title: "Remix環境構築⑤: コンポーネントの作成・使用"
emoji: "😺"
type: "tech"
topics: [Remix,SCSS]
published: true
---

# はじめに

この記事は、Docker上に Remix + Storybook + MUI の環境を構築する手順をまとめています。  
ちなみに React 初心者なので変なところあったら指摘お願いします。  

この記事は下記の記事からの続きです。  
https://zenn.dev/akkey247/articles/20240420_remix_environment_construction_4

# 現在の状態

前の記事で、MUIのテーマの設定とアプリバーの表示を行いました。  
現在下記のようなファイル構成となってます。(RemixやStorybookのバージョンなどによっても違うかも)  
`myapp` ディレクトリがコンテナと同期しているディレクトリで Remix, Storybook のルートディレクトリになっています。  
基本 `myapp` の中の操作になるのでファイルパスを書くときは `myapp` は無視してます。  

```
/
├- docker-compose.yml
└- myapp/
　　├- .storybook/
　　│ 　├- main.ts
　　│ 　└- preview.ts
　　├- app/
　　│ 　├- routes/
　　│ 　│ 　└- _index.tsx
　　│ 　├- entry.server.tsx
　　│ 　├- entry.client.tsx
　　│ 　└- root.tsx
　　├- node_modules/
　　├- public/
　　│ 　└- favicon.ico
　　├- .eslintrc.cjs
　　├- .gitignore
　　├- package-lock.json
　　├- package.json
　　├- README.md
　　├- tsconfig.json
　　├- vite.config.ts
　　└- vite.storybook.config.ts
```

# 1. `sass` のインストール

SCSSを使いたいので、まず下記のコマンドで `sass` をインストールします。  

```
$ docker-compose exec app npm add -D sass
```

# 2. コンポーネントの作成

## コンポーネントの `index.tsx` を作成

`app/components/MyModal/index.tsx` というファイルを下記のように作成します。  
ボタンを押すとモーダルウィンドウが表示されるというコンポーネントです。  

```tsx:app/components/MyModal/index.tsx
import * as React from 'react';
import { Button, Modal, Box } from '@mui/material';
import styles from './styles.module.scss';

export default function MyModal() {
  const [open, setOpen] = React.useState(false);

  const toggleModal = (newOpen: boolean) => () => {
    setOpen(newOpen);
  };

  return (
    <>
      <Button variant="contained" onClick={toggleModal(true)}>Open</Button>
      <Modal
        open={open}
        onClose={toggleModal(false)}
      >
        <Box className={styles.modalBox}>
          Hello World!!
        </Box>
      </Modal>
    </>
  );
}
```

__[補足]__

- この形のコンポーネントを作った理由は、MUIのコンポーネントを使ってみたいのと、動的なものを作ってみたかったからです。
- `app/components` ディレクトリがコンポーネントをまとめる場所として作っています。
- `MyModal` がコンポーネントの名前となります。
- React のコンポーネントは、このように `MyModal/index.tsx` のようにコンポーネント名のディレクトリの配下に `index.tsx` というファイルを作る方法と、 `MyModal.tsx` というファイルを作る方法の２通りがあるみたいです。

## コンポーネントのCSSを作成

`app/components/MyModal/styles.module.scss` を下記のように作成します。  

```scss:app/components/MyModal/styles.module.scss
.modalBox {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 400px;
    background-color: rgb(255, 255, 255);
    border: 2px solid rgb(0, 0, 0);
    box-shadow: rgba(0, 0, 0, 0.2) 0px 11px 15px -7px, rgba(0, 0, 0, 0.14) 0px 24px 38px 3px, rgba(0, 0, 0, 0.12) 0px 9px 46px 8px;
    padding: 32px;
    color: black;
}
```

__[補足]__

- CSSモジュールを使っています。 `.module.css` を付ければCSSモジュールと認識されます(今回はSCSSなので `.module.scss` )。
- Remix ではデフォルトで Vite が入っているのでCSSモジュールの設定は特に無しでCSSモジュールが使えます(Remix 2.2 以上)。
- SCSS も Vite が入っているので特に設定は必要なく使えます。

# 3. コンポーネントの使用

`app/routes/_index.tsx` を下記のように修正します。  

```tsx:app/routes/_index.tsx
import type { MetaFunction } from "@remix-run/node";
import MyModal from "~/components/MyModal";

export const meta: MetaFunction = () => {
  return [
    { title: "New Remix App" },
    { name: "description", content: "Welcome to Remix!" },
  ];
};

export default function Index() {
  return (
    <MyModal />
  );
}
```

__[補足]__

- `~/components/MyModal` のようにディレクトリを指定すればそのディレクトリのコンポーネントを使えます。
- `~` は `tsconfig.json` で設定されていますが、 `~` が `app/` を表すエイリアスになっています。

# 4. 動作確認

一旦うまくいっているかの確認のために Remix を起動して確認してみます。  

## Remix を起動

下記のコマンドで Remix を起動します。  

```
$ docker-compose exec app npm run dev
```

## 表示を確認

`localhost:3000` にアクセスして、コンポーネントが表示されていればOKです。  

![](/images/20240420_remix_environment_construction_5__image1.png)

# 次の記事へ

今回作ったコンポーネントは、MUIで動きがあるUIを作ってみたいのと、コンポーネントを作ってみたいという目的を果たすためのものだったのでかなり適当です。  
とりあえず動くことまで確認したかったという感じです。なので実用性があるのかは不明。  
実用性があるコンポーネントの作り方についてはこれから勉強します。  
自分と同じくらいの理解度の人の参考になれば幸いです。  

この記事では、コンポーネントの作成・使用までを行いました。  
次の記事ではコンポーネントをStorybookへ追加するまでを行います。  

https://zenn.dev/akkey247/articles/20240420_remix_environment_construction_6
