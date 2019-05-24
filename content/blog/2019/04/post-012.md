+++
title = 'React入門 (9) ポータルとモーダル'
description =  'React入門'
date = 2019-04-17T13:13:31Z

[taxonomies]
categories = ["programming"]
tags = ["React"]
+++

# ポータルとモーダル

## ポータルとは?

> Portals provide a first-class way to render children into a DOM node that exists outside the DOM hierarchy of the parent component.[<sup>1</sup>](#com-1)

　正直この説明では良く分からない. 通常DOMノードは親子関係を構築する. つまり子は親の下にぶら下がる親から子へと辿っていける構造になっている. しかしモーダルなどのいくつかのUIsでは動的に生成される必要がある.

## ユースケース

 公式では以下のようなユースケースを上げていました.

+ Dialog
+ Hovercards
+ Tooltips

## 構文

　createPortal[<sup>2</sup>](#com-2)と言うメソッドを使う.


```javascript
ReactDOM.createPortal(child, container);
```


　render[<sup>3</sup>](#com-3)と比較してみよう.


```javascript
ReactDOM.render(element, container[, callback])
```

　引数のcontainerは実際のDOMです(getElementByIdとかで取得する). renderはDOM構造の最上部で呼び出すと言うのが普通ですが, createPortalはサブツリーの好きなところで挿入できます. 特に

```css
.parent {
  overflow: hidden;
}
```

　とかz-indexなどが指定されている場合です. この場合普通に子要素を付け足すと, 隠れて見えなくなってしまいます. これでは具合が悪いので指定したDOM要素に追加できるのがポータルです. 言い換えると子要素が親を選べると言うことでもあります.

## モーダルの実装

　公式の例をそのままパクってきました.[<sup>3</sup>](#com-3) 通常モーダル・コンポーネントはAppコンポーネントの子要素となりますが, ポータルとして指定した要素の子要素になるように指定しています. 子が親を選べるとはこう言うことです. またモーダル・ビューということで要素も動的に生成・削除されています.


```html
<div id="app-root"></div>
<div id="modal-root"></div>
```

```javascript
// These two containers are siblings in the DOM
const appRoot = document.getElementById('app-root');
const modalRoot = document.getElementById('modal-root');

// Let's create a Modal component that is an abstraction around
// the portal API.
class Modal extends React.Component {
  constructor(props) {
    super(props);
    // Create a div that we'll render the modal into. Because each
    // Modal component has its own element, we can render multiple
    // modal components into the modal container.
    this.el = document.createElement('div');
  }

  componentDidMount() {
    // Append the element into the DOM on mount. We'll render
    // into the modal container element (see the HTML tab).
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    // Remove the element from the DOM when we unmount
    modalRoot.removeChild(this.el);
  }
  
  render() {
    // Use a portal to render the children into the element
    return ReactDOM.createPortal(
      // Any valid React child: JSX, strings, arrays, etc.
      this.props.children,
      // A DOM element
      this.el,
    );
  }
}

// The Modal component is a normal React component, so we can
// render it wherever we like without needing to know that it's
// implemented with portals.
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showModal: false};
  }

  handleShow = () => {
    this.setState({showModal: true});
  }
  
  handleHide = () =>{
    this.setState({showModal: false});
  }

  render() {
    // Show a Modal on click.
    // (In a real app, don't forget to use ARIA attributes
    // for accessibility!)
    const modal = this.state.showModal ? (
      <Modal>
        <div className="modal">
          <div>
            With a portal, we can render content into a different
            part of the DOM, as if it were any other React child.
          </div>
          This is being rendered inside the #modal-container div.
          <button onClick={this.handleHide}>Hide modal</button>
        </div>
      </Modal>
    ) : null;

    return (
      <div className="app">
        This div has overflow: hidden.
        <button onClick={this.handleShow}>Show modal</button>
        {modal}
      </div>
    );
  }
}

ReactDOM.render(<App />, appRoot);
```

## Event Bubbling

　親子関係が壊れるわけですがイベント・バブリングはどうなるのでしょうか? ドキュメントではモーダル・ビューで表示したボタンを押すと実際の親ではない, Appコンポーネントに反映されると言う例を示しています.[<sup>4</sup>](#com-4) 状態自体は仮想DOMが管理していますが, イベント・バブリングはReactツリー上で伝達されるそうです.

## まとめ

　Reactは仮想DOMを利用するライブラリで, 一般的な使い方なら実DOMを操作する必要はありません. しかし一部の特殊用途(子要素が親を選ぶ必要が出てくる場合)ではポータルを使って別の親の子要素とすることもできます. 一方この場合(実DOM上で親子関係が存在しない)でもイベントは仮想DOM(Reactツリー)上での親子関係が維持されて, 子要素のイベントが親へと伝達されます.

+++
1. <a name="com-1"></a> [Portals on advanced guides](https://reactjs.org/docs/portals.html)
2. <a name="com-2"></a> [createPortal](https://reactjs.org/docs/react-dom.html#createportal)
3. <a name="com-3"></a> [Modal Example on CodePen](https://codepen.io/gaearon/pen/yzMaBd)
4. <a name="com-4"></a> [Event Bubbling](https://reactjs.org/docs/portals.html#event-bubbling-through-portals)

+++
