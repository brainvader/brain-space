+++
title = "GatsbyJSでブログを作った時のメモ"
description =  "GatsbyJSでブログを作った時のメモ"
date = 2019-04-12T10:47:22Z

[taxonomies]
categories = ["programming"]
tags = ["GatsbyJS"]
+++

# GatsbyJSによるブログの作り方

## GatbyJSとは?

　React.jsベースの静的サイト・ジェネレータです. 画面の構成などをReactで作成してそこにコンテンツのデータを結合して静的サイトを生成します. RailsやLaravelといったWebフレームワークまたはWordpressのようなCMSはリクエストが来てからテンプレートとデータを結合してページを返しますが, 静的サイト・ジェネレータはあらかじめ全てのページを生成しておくプログラムです. このサイト生成の処理時間がいらないのでページの描画を高速にすることができます. では管理が面倒なのかというとそうでもないです. GatsbyJSでは様々なデータをソースとして使えます. 普通のデータベースやWordpressもこの中に含まれており, 既存のプロジェクトを静的サイトに変更することも比較的容易にできます.

## ブログってどんな機能が必要?

　ブログにもいろんな種類があるわけですがプログラムのとなるとだいたい以下のような機能が必要になると思います.

+ タグ
+ コードのハイライト
+ 数式の表示

　この程度の機能ならGatsbyJSでも簡単に実現できます. RailsやLaravelといったWebフレームワークのようにReactでビューが組めますし, 動的な機能もReactのコンポーネントとして定義できます.

## 静的サイト生成の基本的な流れ

　GatsbyJSではデータ取得してReactで作ったビューと結合して静的サイトを作成します.

## データソースの種類

　データソースは色々あります.

+ Wordpress
+ Contentful
+ Markdown

　これ以外にもいろいろあります[*<sup>1</sup>](#ref-1). WordpressとContentfulはCMS(Content Management System)と呼ばれるアプリケーションです. 特にContentfulはHeadless CMSと呼ばれる必要なものです. 今回はMarkdownを使ってブログのコンテンツを管理する方法を説明しようと思います.

## Starter

　GatsbyJSにはいくつかスターター・プロジェクトが用意されています. 今回は0から作るのではなく[gatsby-starter-notes](https://www.gatsbyjs.org/starters/patricoferris/gatsby-starter-notes/)を使いました.

　プロジェクトはgatsby cliを使って生成します. newサブコマンド以下にプロジェクトの名前とスターターを渡せばオッケーです.

```bash
gatsby new project-name https://github.com/patricoferris/gatsby-starter-notes
```

　他にも色々なスターターがあり必要な機能から検索することもできます.

+ [Gatsby Starters](https://www.gatsbyjs.org/starters/?)

## gatsby-config.js

　GatsbyJSの設定ファイルです. ここに必要なプラグインを列挙したり, オプションを指定したりします. プラグインはnpmのパッケージとして管理されているのであらかじめインストールしておく必要があります.

## 必要なプラグイン

　GatsbyJSは便利なプラグインが豊富に提供されています. Markdownの記事を取得するには以下のようなプラグインが必要です.

+ [gatsby-source-filesystem](https://www.gatsbyjs.org/packages/gatsby-source-filesystem/?=markdown#gatsby-source-filesystem)
+ [gatsby-transformer-remark](https://www.gatsbyjs.org/packages/gatsby-transformer-remark/?=markdown#gatsby-transformer-remark)

　1つ目はローカルのファイル・システムからデータを取得するためのものです. 取得したファイルからファイル・ノードが作られます. 次にファイルの内容をtransformerというプラグインで変換します. 2つ目に上げたのがmarkdownを変換するプラグインで, MarkdownRemarkというノードに変換されます. 他にもJSONノードへ変換する[gatsby-transformer-json](https://www.gatsbyjs.org/packages/gatsby-transformer-json/?=json#gatsby-transformer-json)というのがあります.

```bash
npm install --save gatsby-source-filesystem gatsby-transformer-remark
```  

　ブログを作成するには他にもいろんな情報を記述する必要があります. 特にコードのハイライトと数式は欠かせません. 

- [gatsby-remark-prismjs](https://www.gatsbyjs.org/packages/gatsby-remark-prismjs/?=katex#gatsby-remark-prismjs)
- [gatsby-remark-katex](https://www.gatsbyjs.org/packages/gatsby-remark-katex/?=katex#gatsby-remark-katex)

```bash
npm install --save gatsby-remark-prismjs prismjs gatsby-remark-katex katex
```

### ノードとは?

　ノードというのはGraphQLで指定するフィールドのことととりあえず思えばいいです. GraphQLではデータ構造としてグラフを念頭に置いています. しかしデータソースはグラフ構造ではありません. これをGraphQLで扱えるようにグラフ構造へと変換するのがtransfomerの役割です. 変換後のデータにGraphQLでクエリを送りデータを取得するわけです. つまりクエリで指定するフィールドとノードが同じと言う意味です.

## Gatsby Node APIs

　GatsbyJSが提供するAPIsでgatsby-node.jsに記述します. 具体的には[onCreateNode](https://www.gatsbyjs.org/docs/node-apis/#onCreateNode)やcreatePagesという関数があります. 要するにフック関数[*<sup>2</sup>](#ref-2)と言うやつです.
　
　onCreateNodeはノードの作成時に呼び出されます. 今回使ったスターターではslugというノードの追加を行なっています. slugはURLの最後に付加される識別子で個別のページを表すのに使われます. そのためファイル名などから動的に生成する必要があります. createPagesは個別のページが作られるときに呼び出されます. この時点ではノードが生成されているのでクエリでどのようなデータが必要かを指定して, ページのテンプレートに渡します. ここから先はReactのデータ結合と同じです.

## スタイリング

　CSSを用いたスタイリングにもいくつかの書き方があります.

+ Globalスタイル
+ CSSモジュール
+ CSS-in-JS

　詳しくは[公式](https://www.gatsbyjs.org/tutorial/part-two/)で説明されています. 簡単に言うとGlobalスタイルはサイトやアプリ全体に適用されます. CSSモジュールはCSSファイルをimportステートメントで取り込んだコンポーネントだけに適用されます. 裏側ではwebpackのcss-loaderなどを利用しているようです. 最後のCSS-in-JSはその他の通りJavaScriptファイルにオブジェクト形式でCSSを定義すると言う方法です. 今回は公式でもオススメしているCSS-in-JSライブラリである[Emotion](https://www.gatsbyjs.org/docs/emotion/)を使いました.

　emotionを使うとcssという属性を使って直接記述できます. このままインラインになるわけではなく, cssで記述した部分はclassとして置き換えられ, 適当な名前が割り当てられるようです. CSSの名前を考えなくて良いのとスタイルも含めてコンポーネントで完結するのは読みやすいと思いますが, 問題はエディタの恩恵を得られないことでしょうか.

## Githubページへのデプロイ

　ホスティングはGithu Pagesというサービスを利用します. Github PagesはプロジェクトのWebサイトを公開するためのサービスです. 他にもアカウントを持っているユーザーのWebサイトを公開することもできます. やることは非常に簡単です(但しgitに関する知識が要求されます). 

1. Githubのアカウントに登録する.
2. Webページ用のリポジトリを作成
3. プロジェクト・フォルダをプッシュする.
4. ビルドした静的サイトをデプロイする.

　gitやGithubについてはここでは扱いませんが, デプロイは簡単です. まずgh-pagesと言うパッケージを入れます.

```bash
npm install gh-pages --save-dev
```

　後はgatsbyのdevelopサブコマンドを実行後にgh-pagesを実行すればいいです. package.jsonに以下を行を追加しましょう(scriptsがある場合はdeployの項目だけを追加する).

```json
{
  "scripts": {
    "deploy": "gatsby build --prefix-paths && gh-pages -d public"
  }
}
```

　scriptsに定義したコマンドはnpm run nameで実行できます. ただこのまえにGatsbyJSのプロジェクトをpushしておく方がいいです.

```bash
npm run deploy
```

　するとリポジトリにgh-pagesというリポジトリが生成されます. このリポジトリにビルド後の静的サイトが格納され公開されます.

## gatsby-config.js

```javascript
{
    module.exports = {
        siteMetadata: {
            title = 'Brain Space',
        },
        plugins: [
            options: {
                plugins: [
                    `gatsby-remark-prismjs`,
                    `gatsby-remark-katex`,
                    {
                        resolve: `gatsby-remark-images`,
                        options: {
                            maxWidth: 650,
                        },
                    },
                ],
            },
            {
                resolve: 'gatsby-source-filesystem',
                options: {
                    path: `${__dirname}/src/posts`,
                    name: 'posts',
                },
            },
        ],
    }
}
```

## 注釈
<a name="ref-1"></a> 1 [Sourcing Content and Data](https://www.gatsbyjs.org/docs/content-and-data/)  
<a name="ref-2"></a> 2 あるタイミングで呼び出される関数のこと.  

## Reference
+ [How Gatsby Works with GitHub Pages](https://www.gatsbyjs.org/docs/how-gatsby-works-with-github-pages/)
