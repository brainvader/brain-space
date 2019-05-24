+++
title = "React入門 (3) ライフサイクル・フック"
description =  "React入門"
date = 2019-04-15T11:50:22Z

[taxonomies]
categories = ["programming"]
tags = ["React"]
+++

# ライフサイクル・フック

## Reactのライフサイクル

　正確にはReactのコンポーネントのライフサイクルで, レンダリングされることをマウントと呼ぶ. 公式の[Lifecycle Diagram](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)を参照してほしい.

## マウントとアンマウント

　コンポーネントがレンダリングされることをマウントと呼ぶ. 削除されることをアンマウントと呼ぶ.

## ライフサイクル・フックとは?

　Reactのアプリの任意のタイミングで呼び出される関数のことです. コンポーネントが利用するリソースの処理などを記述します.

## 種類

　　この他にもconstructorやrenderメソッドもライフサイクル・メソッドである.  これ以外にとりあえず以下の三つを覚えれば十分である.

+ componentDidMount
+ componentDidUpdate
+ componentWillUnmount

　constructorはマウントされる前に呼び出される. stateとpropsの初期化などに使います. renderを挟んでコンポーネントが表示された後にcomponentDidMountが呼び出されます. 動的に削除する前にcomponentWillUnmountが呼び出されます. コンポーネントの状態は変化するので変更されるたびにcomponentDidUpdateが呼び出されます.

## ユースケース

　公式ではタイマーの例を使って説明していました. 

1. ReactDOM.renader()にコンポーネントが渡されるとconstructorが呼び出される.
2. renderでReact要素が返される.
3. 表示後にcomponentDidMountが呼び出される.
3. componentWillUnmountメソッドの場合clearInterval関数のように後処理が必要な場合に使えます.

　setIntevalをcomponentDidMount時に設定して, 削除する場合に4を呼び出すという流れです.

```javascript
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

## まとめ

　ひとまずそういうものがあったというのを覚えておくと, 実際のアプリを作成するときよりReactらしいコードディングができます.

## Further Readind
+ [React.Component](https://reactjs.org/docs/react-component.html)
