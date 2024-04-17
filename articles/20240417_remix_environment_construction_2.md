---
title: "Remix環境構築: 2. Storybookのインストール～表示まで"
emoji: "😸"
type: "tech"
topics: [Remix,Storybook]
published: true
---

#　はじめに

この記事は下記の記事からの続きです。  
https://zenn.dev/akkey247/articles/20240417_remix_environment_construction_1

この記事の内容は下記の記事をかなり参考にしました。  
https://zenn.dev/m_ryosuke/articles/868eacfc1870c0

# Storybook の導入

## インストール

```
$ docker-compose exec app npx storybook@latest init
```

## Storybook　起動時のエラーを解消する

Storybook をインストールするとインストール完了と同時に Storybook が自動で起動しますが、その際に下記のようなエラーが発生します。  
(`Error: The Remix Vite plugin requires the use of a Vite config file` の部分)

```
Running Storybook

> storybook
> storybook dev -p 6006 --initial-path=/onboarding --quiet

@storybook/cli v8.0.8

(node:473) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)
info => Starting manager..
info => Starting preview..
=> Failed to build the preview
Error: The Remix Vite plugin requires the use of a Vite config file
```

エラー的には `vite.config.ts` で読み込んでいる Remix Vite プラグインで必要な設定ファイルが見つからない的な内容のようですが、要するに `vite.config.ts` は Remix 用に作ってあるので Storybook では使えんよということみたいです。  
なので、 Storybook 用に別の `vite.config.ts` を作成します。  

あと、上記のエラーの後に `Would you like to help improve Storybook by sending anonymous crash reports?` という質問が出ますが、これは「クラッシュレポートを送信しますか？」という内容なので回答はどっちでも大丈夫そうです。  

### Storybook 用の `vite.config.json` を作成

Storybook 専用に `vite.storybook.config.ts` というファイルを作成します。(別の名前でもOK)  

```ts:vite.storybook.config.ts
import { defineConfig, loadEnv } from 'vite';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '');
  process.env = { ...process.env, ...env };
  return {
    plugins: [tsconfigPaths()],
  };
});
```

### `.storybook/main.ts` を修正

`app/.storybook/main.ts` を修正して、さっき作った `vite.storybook.config.ts` を読み込むように変更します。  

```ts:main.ts
import type { StorybookConfig } from "@storybook/react-vite";

const config: StorybookConfig = {
  stories: [
    "../stories/**/*.mdx",
    "../stories/**/*.stories.@(js|jsx|mjs|ts|tsx)",
  ],
  addons: [
    "@storybook/addon-onboarding",
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@chromatic-com/storybook",
    "@storybook/addon-interactions",
  ],
  framework: {
    name: "@storybook/react-vite",
    options: {
      builder: {
        viteConfigPath: 'vite.storybook.config.ts',
      }
    },
  },
  docs: {
    autodocs: "tag",
  },
};
export default config;
```

## Storybook を起動

一旦うまくいっているかの確認のために Storybook を起動して確認してみます。  
下記のコマンドで Storybook を起動します。  

```
$ docker-compose exec app npm run storybook
```

## Storybook へのアクセスを確認

`localhost:6006` にアクセスして下記のようなページが表示されたらOKです。  

![](/images/20240417_remix_environment_construction_2__image1.png)

# 次の記事へ

この記事では、Storybookのインストールから表示までを行いました。  
次の記事ではMUIのインストールから表示までを行います。  