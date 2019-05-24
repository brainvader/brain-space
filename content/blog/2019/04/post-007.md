+++
title = "React入門 (4) イベントハンドリングとState"
description =  "React入門"
date = 2019-04-15T12:50:22Z

[taxonomies]
categories = ["programming"]
tags = ["React"]
+++

# イベントハンドリングとState

## イベントハンドリング

　コンポーネントにはメソッドが定義できる. これがそのままJSXに結合することができる. またクラス・メソッドは通常バインドされない. アロー関数と思えば良い. そうしないとメソッドとして定義した関数の内部に別のスコープが発生し, thisの生合成がなくなる. そのため明示的にバインドしてやる必要がある.

```jsx
<button onClik={this.clickHandler.bind(this)}>Click</button>
```

　ちなみにcreate-react-appを使うとクラス・フィールド構文というのが使われる. これを使うと明示的にbindをする必要はない.

## setState

　通常ユーザーインタクラションは内部状態を変更を伴う. ただしstateは直接変更せずsetState関数経由で行う必要がある.

```javascript
this.state(name: 'NewName');
```

　またpropsとstateは非同期に更新されるので両者に依存するような書き方はできない.

```javascript
// Wrong Case
this.setState({
  counter: this.state.counter + this.props.increment,
});

// Correct Cases
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));

this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});
```

## まとめ

　内部状態の変更を伴うイベント・ハンドラの利用例は以下のようになる.

```javascript
class MyButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = { available: true };
  }

  clickHandler() {
    this.setState((state, props) => ({
      available = !state.avaialble;
    }));
  }

  render() {
    return (
      <button onClick={this.clickHandler.bind(this)}>Click Me</button>
    );
  }

  ReactDOM.render(<MyButton />, document.getElementById('root));
}
```
