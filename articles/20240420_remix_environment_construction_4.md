---
title: "Remix環境構築④: MUIのテーマの設定とアプリバーの表示"
emoji: "😺"
type: "tech"
topics: [Remix,MUI]
published: true
---

# はじめに

この記事は、Docker上に Remix + Storybook + MUI の環境を構築する手順をまとめています。  
ちなみに React 初心者なので変なところあったら指摘お願いします。  

この記事は下記の記事からの続きです。  
https://zenn.dev/akkey247/articles/20240417_remix_environment_construction_3

# 現在の状態

前の記事で、MUIをインストールを行いました。  
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

# テーマの追加、アプリバーの追加

## `app/root.tsx` を修正

`app/root.tsx` を下記のように修正します。  

```tsx:app/root.tsx
import {
  Links,
  Meta,
  Outlet,
  Scripts,
  ScrollRestoration,
} from "@remix-run/react";
import { ThemeProvider, createTheme } from '@mui/material/styles';
import {
  CssBaseline,
  AppBar,
  Toolbar,
  Container,
  Typography,
} from '@mui/material';

const theme = createTheme({
  palette: {
    mode: 'dark',
    primary: {
      main: '#1976d2',
    },
  },
});

export function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <Meta />
        <Links />
      </head>
      <body>
        {children}
        <ScrollRestoration />
        <Scripts />
      </body>
    </html>
  );
}

export default function App() {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <AppBar position="static" color="primary" enableColorOnDark>
        <Toolbar>
          <Typography variant="h6">
            My App
          </Typography>
        </Toolbar>
      </AppBar>
      <Container>
        <Outlet />
      </Container>
    </ThemeProvider>
  );
}
```

__[補足]__

とりあえず、テーマを設定して、アプリバーを表示してみます。  
テーマはわかりやすいようにダークテーマにしました。アプリバーはコードはコードが長くなるので超シンプルにしています。  

## Remix を起動

下記のコマンドで Remix を起動します。  

```
$ docker-compose exec app npm run dev
```

## 表示を確認

`localhost:3000` にアクセスして、ダークテーマが設定されてアプリバーが表示されていればOKです。  

![](/images/20240420_remix_environment_construction_4__image1.png)

※ 「ボタン」は前回の記事で追加した部分です。

# 次の記事へ

この記事では、MUIのテーマの設定とアプリバーの表示までを行いました。  
次の記事ではコンポーネントの作成・使用までを行います。  

https://zenn.dev/akkey247/articles/20240417_remix_environment_construction_5
