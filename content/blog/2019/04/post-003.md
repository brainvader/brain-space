+++
title = "React入門 (2) JSXとReactエレメント"
description =  "React 入門"
date = 2019-04-13T10:50:22Z


[taxonomies]
categories = ["programming"]
tags = ["React"]
+++

# JSX

## JSXとは?

　JavaScriptの拡張でHTMLタグのような記法を使います. コンポーネントはロジックとビューで構成されますが, このビューの部分をJSXとして記述します. Webフレームワークで使われるテンプレートとは逆にJavaScriptにビューを取り入れています. JSXの部分はテンプレートなので中括弧を使ってJavaScriptの変数や関数を埋め込むこともできます.

　HTMLに似ていますが基本的にはJavaScriptの拡張です. 属性名などもclassとclassNameのように違ってきます. コンポーネントの記述方法などは[前回](/2019/04/post-002/)の記事を参考にしてください.

　JSXはXSS(Cross-Site-Scripting)対策もされています. つまりHTMLタグが入力としてきた場合は文字列に変換されます. dangerouslySetInnerHTMLを使うとHTML要素として表示されますが危険が伴います. GatsbyJSなどではMarkdownの記事を変換してHTMLとして表示するのにdangerouslySetInnerHTMLが利用されています.

## JSXの正体

　JSXはReact固有の記法として導入されました(現在は他にも使われているかもしれませんが). JavaScriptの拡張といってもブラウザでは解釈できません. そのためコンパイラでブラウザが解釈可能な構文に変換する必要があります. コンパイル後はReact.createElementに変換されます.

```javascript
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

　この関数はReactエレメントを生成する関数ですが, ただのPOJO(Plain Old Javascript Object)です. DOM(Document Object Model)生成に使われるレシピとでも思えばいいと思います. 

## 仮想DOM

　実際のDOMは木構造をしています. React要素はその設計図なので何を作るかとか子要素は何かという情報を持っています. これを仮想DOMと言います. DOMとは別にこのようなデータ構造を挟むのは不経済と思うかもしれませんが, 実際には仮想DOMの方が早く処理できます[<sup>*1</sup>](#rel-1).

## Immutability (不変性)

　React要素はHTML要素と違って不変(immutable)です. 不変というのは変更があった場合は, 構成するノードを全て再描画し直します. これだと非効率と思うかもしれませんが変更は仮想DOM上で行われます. DOMの方は必要な部分だけが更新されます. こうした仮想DOMを用いた効率的なDOM操作がReactの魅力の1つです.

## ルートノード

　ReactエレメントからDOMを作りますが, ルート・ノードだけは別です. ルート・ノードはアプリケーションの入れ物で普通のHTMLです. このノードの下に最終的なDOM要素が追加されます. 名前とは裏腹に複数あっても良いそうです.

## まとめ

　JSXの正体はcreateElementが生成するReact要素でした. React要素はPOJOに過ぎず, これが組み合わさって仮想DOMを構成していました. 仮想DOM上に変更を限定し実際のDOMの変更からコードの変更を切り離すことでReactは高速な画面の更新を実現しています.



## 注釈
<a name="ref-1"></a> 1 ということになっていますが実際は知りません. 信じましょうw.


## Reference
+ [MAIN CONCEPTS](https://reactjs.org/docs/hello-world.html)
