+++
title = 'React入門 (7) Reactプロジェクトの作成方法'
description =  'React入門'
date = 2019-04-16T11:50:22Z

[taxonomies]
categories = ["programming"]
tags = ["React"]
+++

# React プロジェクトの作成方法

## 種類

1. ライブラリとしてプロジェクトの追加[<sup>1</sup>](#com-1)
2. 0 から環境を構築して新規作成
3. ツールチェインを用いて React アプリとして新規作成[<sup>2</sup>](#com-2)

このポストでは最後の Create React App での SPA 用プロジェクトの作成を紹介したいと思います.

## ライブラリとしてプロジェクトの追加

React は SPA 用のライブラリと思われがちですが, jQuery などの既存の UI ライブラリと同じように使うことができます. ただこの場合は ES6 以降のブラウザが対応していない JavaScript の構文は使えません. 当然 JavaScript の拡張構文である JSX も使えません. これらを利用するためには webpack などの導入する必要がありますが, この場合はツールチェインを用いたやり方の方が良いと思います.

## 0 から環境を構築して新規作成

用途によるので一概に説明はできません. しかし比較的モダンな開発環境を簡単に手に入れるなら[Parcel](https://parceljs.org/)が良いでしょう. Parcel は Webpack と違ってほとんど設定をする必要がありません(zero configuration). ただこの場合は Create React App で良いのではという話も出てきます. ライブラリとして導入したけどモダンな仕様を試してみたいという時なんかに使えると思います.

React の環境構築を一から学びたいという場合はやはり[Webpack](https://webpack.js.org/)について勉強すると便利です. 何れにしても JavaScript のパッケージ・マネジャである npm や yarn を使うのは必須です.

インポート構文など ES6 以降の機能を使いたい場合は babel, 型を利用したい場合 TypeScript などのトランスパイラが利用できます. TypeScript については別の機会に書いてみたいと思います.

## ツールチェイン

SPA(Single Page Application)や SSR(Server Side Rendering)ウェブサイトや静的サイト, ネイティブ・アプリなど用途によっていくつかのツールが用意されています.

-   [Create React App](https://facebook.github.io/create-react-app/)
-   [Next.js](https://nextjs.org/): server side rendering site
-   [Gatsby](https://www.gatsbyjs.org/): Static Site Generator (SSG)
-   [React Native](https://facebook.github.io/react-native/): Native Application
-   [Electron](https://electronjs.org/): Desktop Application

## ツールチェインを用いて React アプリとして新規作成

まず create-react-app パッケージを導入します.

```bash
npm i -g install create-react-app
```

後は導入されたコマンドでプロジェクトを作成するだけです[<sup>3</sup>](#com-3).

```bash
npx create-react-app my-app
cd my-app
npm start
```

[localhost:3000](localhost:3000)ページを開くとサンプルページが表示されているはずです. 次に VSCode 限定ですが, コードのフォーマットの設定を行います[<sup>4</sup>](#com-4). 整形のためのツールであるprettierを導入します.

```bash
npm i --dev --exact prettier
npm i --dev eslint-plugin-prettier
```

　雛形プロジェクトの package.json には eslintConfig という項目があります. ここに prettier の設定を追加します.

```json
"eslintConfig": {
    "extends": "react-app",
    "plugins": [
        "prettier"
    ],
    "rules": {
        "prettier/prettier": "error"
    }
},
```

　これで整形はprettierがやってくれます. 後はルールを定義するだけです. .prettierrcとういファイルをプロジェクト直下に配置します.

```json
{
    "singleQuote": true,
    "tabWidth": 4,
    "trailingComma": "es5"
}
```

## まとめ

自分がやろうとしている用途によって柔軟に React を導入できる.

## 注釈

1. <a name="com-1"></a>. [Add React to a Website](https://reactjs.org/docs/add-react-to-a-website.html)  
2. <a name="com-2"></a>. [Create a New React App](https://reactjs.org/docs/create-a-new-react-app.html)  
<a name="com-3"></a> \*3. [Quick Start on React Create App](https://facebook.github.io/create-react-app/docs/getting-started#quick-start)
<a name="com-4"></a> \*4. [Using Prettier with VS Code and Create React App](https://medium.com/technical-credit/using-prettier-with-vs-code-and-create-react-app-67c2449b9d08)
