+++
title = 'React入門 (10) Refs'
description =  'React入門'
date = 2019-04-17T16:47:39Z

[taxonomies]
categories = ["programming"]
tags = ["React"]
+++

# Refs

## Referenceの取得

　通常親がこの要素へ何かするときはpropsで渡してやる. つまりこの内部状態を親へと持ち上げる必要がある.[<sup>1</sup>](#com-1) しかし以外にも以下のような場合にいくつかのケースで親が子コンポーネントの要素へを参照できると便利な場合がある.

+ focus
+ text selection
+ media playback


　なおrefsで取得されるのは実DOMへの参照です. コンポーネントのインスタンスではありません. そのため以下のような用途でも利用できるようです.


+ 仮想DOMを使わないサード・パーティ製のライブラリ

## createRefs

　まず参照を保存する変数をcreateRefで作成する. あとはrefs属性をつけたJSX要素に結合すればオッケーです. 実際のDOM要素はcurrentプロパティに保存されています.

```javascript
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // create a ref to store the textInput DOM element
    this.textInput = React.createRef();
  }

  focusTextInput = () => {
    // Explicitly focus the text input using the raw DOM API
    // Note: we're accessing "current" to get the DOM node
    this.textInput.current.focus();
  }

  render() {
    // tell React that we want to associate the <input> ref
    // with the `textInput` that we created in the constructor
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} />

        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```


　このコンポーネントは親から参照することができます.  


```javascript
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }

  componentDidMount() {
    this.textInput.current.focusTextInput();
  }

  render() {
    return (
      <CustomTextInput ref={this.textInput} />
    );
  }
}
```


　このようにfocusTextInputはCustomTextInputコンポーネントのメソッドですが, 参照を利用して親コンポーネントで利用することができます.


## Callback Refs

　通常はプロパティとして参照を定義しますが, コールバック関数を使って参照を取得することもできます. こうすることで参照時に処理を加えるなどの操作が可能になります. CustomTextInputをcallback refsで置き換えてみます.


```javascript
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // Focus the text input using the raw DOM API
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // autofocus the input on mount
    this.focusTextInput();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```


　親コンポーネントからコールバックを渡すこともできます.


```javascript
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}
      />
    );
  }
}
```


　これで子要素であるinputへの参照を取得できました.


## String Refs

　canvas要素を利用数場合は仮想DOMではダメなので, 実態へアクセスする必要があります. この場合にcanvas要素にrefs属性としてキーワードを設定して参照するような利用方法がありますが現在はLegacy APIとなっており上述のどちらかを使うようです.

　と言っても変更点はストリングをプロパティに変更するだけです.[<sup>2</sup>](#com-2)

```javascript
class CanvasExample extends React.Component {
  constructor(){
    super()
    this.canvas = React.createRef()                  //// 1 - create ref
  }

  componentDidMount() {
    this.drawRectInCanvas()
  }

  drawRectInCanvas(){
    const ctx = this.canvas.current.getContext('2d') //// 3 - access node using .current
    ctx.fillRect(5, 5, 200, 200)                     ////   - do something!
  }

  render(){
    return (
      <div>
        <canvas ref={this.canvas} />                 //// 2 - attach ref to node
      </div>
    )
  }
}
```

## Forwarding refs

　参照変数を子コンポーネントに送る方法である. やり方は簡単でfowardRefとして子コンポーネントを定義するだけである.

```javascript
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.label}
  </button>
));


class ParentComponent extends React.Component {
  constructor() {
    super();
    this.ref = React.createRef();
  }

  render() {
    retrun (
      <FancyButton ref={this.ref} label="Click Me"/>
    );
  }
}
```

## Higher Order Components (HOCs)

　高階関数を知っている人なら説明は不要かと思いますが, 公式ドキュメントには以下のように端的に説明されています. 

> Concretely, a higher-order component is a function that takes a component and returns a new component.[<sup>3</sup>](#com-3)

　ReduxのconnectやRelayのcreateFragmentContainerがHOCsに当たるそうです. コンポーネントは継承ができません. このことから共通部分を抽象化して取り出す(extract out)ができません. 

## 共通化

　では共通化するにはどうすればいいでしょうか? そのようなときに使うのがHOCsです. 以下のCommetListとBlogPostの共通部分を抜き出してみましょう.

```javascript
class CommentList extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // "DataSource" is some global data source
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // Subscribe to changes
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // Clean up listener
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // Update component state whenever the data source changes
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}
```

```javascript
class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id)
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id)
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
}
```


　ライフサイクル・メソッドを使ってDataSourceに購読するという同じ処理が含まれています. 継承や委譲は使えません. 

## Container Component


　ではどうするかというと, 入力コンポーネントを表示する新しいコンポーンネントを使います. このようなコンポーネントをContainer Componentと呼びます.


```javascript
// This function takes a component...
function withSubscription(WrappedComponent, selectData) {
  // container component
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ... that takes care of the subscription...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

　WrappedComponentにCommentListやBlogPostを渡してやると同じ結果になるのがわかると思います.

## Pure HOCs

　コンテナ・コンポーネントを使うときは渡したコンポーネントを改変しては行けません. 

```javascript
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it. Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

## Parameterized Container Component

　HOCsの別名のようです. コンテナ・コンポーネントはパラメータとして渡されたコンポーネントを表示する役割を持ちます. コンポーネントをパラメータ化していることからこのような名前でも呼ぶようです.


## connectと部分適用

　Reduxのconnect関数の仕組みを少し説明します.


```javascript
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```

　このように部分適用が利用されていてわかりにくいです. もちろん部分適用になっているのには理由があります. connectはHOCを返す関数で戻り値となるコンポーネントにCommentListコンポーネントを渡しています. もちろん左辺はコンポーネントになります.

```javascript
// connect is a function that returns another function
const enhance = connect(commentListSelector, commentListActions);
// The returned function is a HOC, which returns a component that is connected
// to the Redux store
const ConnectedComment = enhance(CommentList);
```

　こうすると関数のチェイン化や関数合成ができたりして便利らしいです.[<sup>4</sup>](#com-4) ReduxやRamdaと言ったライブラリではよく使われているようです.


## HOCsの適用

　HOCsの適用はコンポーネントの定義部分で行っては行けません. こうすると毎回新規作成されるため内部状態(state)その都度失われ, その下にぶら下がっている子コンポーネントにも影響が出ます.

　一例としてこんな感じにします.


```javascript
const ResultComponent = HOC(WrappedComponent);
// or
export default HOC(WrappedComponent);
```

## Forwarding refs with HOCs


　refsをコンテナ・コンポーネントを通じて内部コンポーネントに渡す場合refsは渡せません. LogPropsというコンテナ・コンポーネントでrefを渡そうとしてもrefはkeyと同様にpropsからアクセスできません. そこでコンテナ・コンポーネントをReact.forwardRefを使って返すといことを行います.


```javascript
function logProps(InnerComponent) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      const {forwardedRef, ...rest} = this.props;

      // Assign the custom prop "forwardedRef" as a ref
      return <InnerComponent ref={forwardedRef} {...rest} />;
    }
  }

  // Note the second param "ref" provided by React.forwardRef.
  // We can pass it along to LogProps as a regular prop, e.g. "forwardedRef"
  // And it can then be attached to the Component.
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}
```

## Uncontrolled Component

　<a href="/2019/04/post-009">フォーム</a>の回で説明したようにフォームのデータは一度コンポーネントのstateとして管理しました(Controlled Component). refを使うと同じことができます. このように内部状態で管理しない, refで直接DOM要素とやりとりするコンポーネントをuncontrolled componentと言うようです.

　またFile APIを使う場合は, uncontrolled componentを使うそうです. これはアップロードするファイルをReact側で制御しているわけではないからです. ユーザーが実DOMとのやり取りでファイルを指定するので, refでDOM要素からアクセする必要が出てきます.

```html
<input type="file" />
```

## まとめ

　ref属性を使うと実DOMへの参照を取得できます. ただしこれを子コンポーネントに渡す場合はFowarding Refを使って順送りしてやる必要があります. またHOCsの入力コンポーネントの参照を取得する場合にも使えます.


+++

1. <a name="com\-1"></a> [Lifting State Up](https://reactjs.org/docs/lifting-state-up.html)  
2. <a name="com\-2"></a> [CanvasRefExample.js](https://gist.github.com/chansiky/d68c58508eda3a78ba7754cebd43cc47)  
3. <a name="com\-3"></a> [Higher-Order Components](https://reactjs.org/docs/higher-order-components.html)  
4. <a name="com\-4"></a> [Convention: Maximizing Composability](https://reactjs.org/docs/higher-order-components.html#convention-maximizing-composability)

+++
