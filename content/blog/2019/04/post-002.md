+++
title = "React入門 (1) Reactとコンポーネント"
description =  "React 入門"
date = 2019-04-12T10:50:22Z

[taxonomies]
categories = ["programming"]
tags = ["React"]
+++

# React入門

## Reactとは?

　Facebook製のUIライブラリです. 以下のような特徴があります.

+ Reactive
+ Component-based
+ Declarative
+ Virtual DOM
+ Bidirectional

　名前の通りリアクティブなUIを設計することができるライブラリです. ただReactを中心にしたエコシステムが存在し, JSXやHTMLタグ風のコンポーネントなどを変換するもあり実質UIフレームワークといっても良いものだと思います. 今回はコンポーネントについて説明したと思います.

## コンポーネント

　ロジックが定義されているのでMVCアーキテクチャでいうモデルに近いですが, JSXというHTML風の構文でビューも一緒に定義しています. ステートと呼ばれる内部状態を持ち, 定義された状態を変更してビューを描画します. 画面の中の一部を抽象化した部品です. 定義したコンポーネントはHTMLタグのようにJSX内に記述できます. コンポーネント自体はES6のクラスとして定義されていますが, 関数コンポーネントとしても定義できます. これはのクラスも関数も全てオブジェクト(連想配列)だというJavaScriptの特徴によります.

　コンポーネントは以下のように定義されます.

```javascript
import React from 'react';
class MyComponent extends React.Component {
    constructor(props) {
        super(props);
        this.name  = props.name;
        this.state = { money: 0 };
    }

    doSomething() {
        // write logic here
    }

    render() {
        return (
            <div>
              <h2>適当</h2>
              <p>
                私は適当なコンポーネントです. 名前は{this.name}です. 所持金は{this.money}です.
                よろしくお願いします.
              </p>
            </div>
        )
    }
}
```

　ほぼHTMLですがこのrender関数内でreturnされる部分をJSXと呼んでいます. 要するにテンプレートです. 内部状態にはthisでアクセスし, 中括弧でビューと結合します. コンストラクタの引数として渡されるのがpropsです.

### props vs state

　コンポーネントの定義は単なるクラス(関数)でした. ではどのように利用しデータの受け渡しはされるのでしょうか?

```javascript
<MyComponent name="Component Taro"/>
```

　一見HTMLですがこれがコンポーネントですが, 属性にあたる部分がが引数として扱われます. 一方stateは外部からはアクセスできません.

### 特徴

　コンポーネントとは,

+ Independent(独立性)
+ Reusable (再利用性)
+ Isolation (隔離性)
+ Pure Function (純粋関数)

　があるとされています[<sup>1</sup>](#ref-1). これらは要するにコンポーネントごとにテストや実行などが可能ということを意味します.

### 純粋関数

　Reactのコンポーネントはpropsに対して純粋関数として振舞うこととされています[<sup>2</sup>](#ref-2). 簡単にいうと引数を上書きしないということです. しかしJavaScriptの変数は可変です. propsも当然書き換えようと思えば書き換えれます. そのためこれはこれは規約です. propsは読み込み専用ということを覚えておきましょう. 上書き可能な変数(蓄積変数など)が必要な場合はstateを利用します.

### 種類

　いくつか定義の仕方があります.

+ 関数コンポーネント (RFC)
+ アロー関数コンポーネント (RAFC)
+ クラスコンポーネント (RCC)

　VSCodeなどではカッコ内の名前を打ち込むとスニペットが補完されたりします.

## 注釈
<a name="ref-1"></a> 1 [Components and Props](https://reactjs.org/docs/components-and-props.html)
<a name="ref-2"></a> 2 [Props are Read-Only](https://reactjs.org/docs/components-and-props.html#props-are-read-only)

## Reference
+ [MAIN CONCEPTS](https://reactjs.org/docs/hello-world.html)
