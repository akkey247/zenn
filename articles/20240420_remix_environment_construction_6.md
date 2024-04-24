---
title: "Remix環境構築⑥: コンポーネントをStorybookへ追加"
emoji: "😺"
type: "tech"
topics: [Remix,Storybook]
published: true
---

# はじめに

この記事は、Docker上に Remix + Storybook + MUI の環境を構築する手順をまとめています。  
ちなみに React 初心者なので変なところあったら指摘お願いします。  

この記事は下記の記事からの続きです。  
https://zenn.dev/akkey247/articles/20240420_remix_environment_construction_5

※ コンポーネントの作成自体は前回の記事でやっているのでそっちを参考にしてください

# 現在の状態

前の記事で、コンポーネントの作成・使用を行いました。  
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
　　│ 　├- components/
　　│ 　│ 　└- MyModal
　　│ 　│ 　　　├- index.tsx
　　│ 　│ 　　　└- styles.module.scss
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

# 1. コンポーネントをStorybookへ追加

## ストーリーファイルを作成

`app/components/MyModal/MyModal.stories.ts` というファイルを下記のように作成します。  

```ts:app/components/MyModal/MyModal.stories.ts
import type { Meta, StoryObj } from '@storybook/react';
import { fn } from '@storybook/test';
import MyModal from "~/components/MyModal";

const meta = {
  title: 'Example/MyModal',
  component: MyModal,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
  args: { onClick: fn() },
} satisfies Meta<typeof MyModal>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {},
};
```

## Storybookの `main.ts` を修正

`.storybook/main.ts` を下記のように修正します。  
`stories` の設定を `["../app/components/**/*.stories.@(js|jsx|mjs|ts|tsx)"]` のように修正します。  

```ts:.storybook/main.ts
import type { StorybookConfig } from "@storybook/react-vite";

const config: StorybookConfig = {
  stories: [
    "../app/components/**/*.stories.@(js|jsx|mjs|ts|tsx)",
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

__[補足]__

- `stories` の部分で指定したパスからコンポーネントが読み込まれます。

# 2. 動作確認

一旦うまくいっているかの確認のために Storybook を起動して確認してみます。  

## Storybook を起動

下記のコマンドで Storybook を起動します。  

```
$ docker-compose exec app npm run storybook
```

## Storybook へのアクセスを確認

`localhost:6006` にアクセスしてMyModalのコンポーネントが表示されていたらOKです。  

![](/images/20240420_remix_environment_construction_6__image1.png)

# 3. デフォルトのコンポーネントディレクトリを削除

必要が無ければ `stories` ディレクトリを削除します。  
`stories` はStorybookインストール時に自動で作成されるサンプルのコンポーネントディレクトリなので使わなければ削除して大丈夫です。  

# おわりに

Docker上に Remix + Storybook + MUI の環境を構築しました。  
とりあえず個人的に入れたいものは入れれました。参考になれば嬉しいです。  
