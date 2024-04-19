---
title: "Remix環境構築③: MUIのインストール～表示まで"
emoji: "😸"
type: "tech"
topics: [Remix,MUI]
published: true
---

# はじめに

この記事は、Docker上に Remix + Storybook + MUI の環境を構築する手順をまとめています。  

この記事は下記の記事からの続きです。  
https://zenn.dev/akkey247/articles/20240417_remix_environment_construction_2

# 現在の状態

前の記事では、Storybookをインストールを行いました。  
現在下記のようなファイル構成となっています。(RemixやStorybookのバージョンなどによっても違うかも)  
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

# MUI の導入

## インストール

下記のコマンドで MUI 関連のパッケージをインストールします。

```
$ docker-compose exec app npm add @mui/material @mui/icons-material @emotion/styled @emotion/react
```

## `app/routes/_index.tsx` を修正

試しにMUIのボタンを配置してみます。  
`app/routes/_index.tsx` を下記のように修正します。  

```tsx:app/routes/_index.tsx
import type { MetaFunction } from "@remix-run/node";
import { Button } from '@mui/material';

export const meta: MetaFunction = () => {
  return [
    { title: "New Remix App" },
    { name: "description", content: "Welcome to Remix!" },
  ];
};

export default function Index() {
  return (
    <Button variant="contained">ボタン</Button>
  );
}
```

## Remix を起動

下記のコマンドで Remix を起動します。  

```
$ docker-compose exec app npm run dev
```

## 表示を確認

`localhost:3000` にアクセスしてみると下記のように一瞬ボタンがちゃんと表示されるが、すぐ普通のボタンに変わってしまいます。  

![](/images/20240417_remix_environment_construction_3__image1.png)

ブラウザのコンソールを見ると下記のような感じのエラーが出てます。  

```
Warning: Expected server HTML to contain a matching <meta> in <head>.
```

```
Uncaught Error: Hydration failed because the initial UI does not match what was rendered on the server.
```

```
Error: Hydration failed because the initial UI does not match what was rendered on the server.
```

これは、サーバー側とクライアント側でHTMLの構成が変わっているというエラーらしいので、これを解消していく。  

# ハイドレーションエラーを解消する

## entry.client.tsx を変更

```tsx:app/entry.client.tsx
/**
 * By default, Remix will handle hydrating your app on the client for you.
 * You are free to delete this file if you'd like to, but if you ever want it revealed again, you can run `npx remix reveal`
 * For more information, see https://remix.run/file-conventions/entry.client
 */

import { RemixBrowser } from "@remix-run/react";
import { startTransition, StrictMode } from "react";
import { hydrateRoot } from "react-dom/client";
import { CacheProvider } from '@emotion/react';
import createCache from '@emotion/cache';

function createEmotionCache() {
  return createCache({ key: 'css' });
}

const cache = createEmotionCache();

startTransition(() => {
  hydrateRoot(
    document,
    <StrictMode>
      <CacheProvider value={cache}>
        <RemixBrowser />
      </CacheProvider>
    </StrictMode>
  );
});
```

## 再び表示を確認

`localhost:3000` にアクセスしてみると下記のようにちゃんとボタンが表示される。エラーも解消されている。  

![](/images/20240417_remix_environment_construction_3__image2.png)

# 次の記事へ

この記事では、Storybookのインストールから表示までを行いました。  
次の記事ではMUIのインストールから表示までを行います。  
