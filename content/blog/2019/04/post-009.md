+++
title = "React入門 (6) フォーム"
description =  "React入門"
date = 2019-04-16T11:50:22Z

[taxonomies]
categories = ["programming"]
tags = ["React"]
+++

# フォーム

## フォームの基本

　フォームは通常以下のように定義されます. 問題はフォームはユーザーが入力したデータを保持するということです. これをコンポーネントで管理してやる必要があります. このようになコンポーネント化されたフォームをControlled Componentと呼びます.

```html
<form>
  <label>
    Name:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Submit" />
</form>
```

## Controlled Component

　input, textarea, そしてselectなどがControlled Componentの対象です. これらは全てvalue属性で値にアクセスできることが共通しています.

### 入力フォーム

　value属性に入力された値が保持されています.

```html
<form>
  <label>
    Name:
    <input id="myInput" type="text" name="name" value="John" onchange=changeHandler()/>
  </label>
  <input type="submit" value="Submit" />
</form>
```

　一方, value属性を上書きするとフォーム・フィールドに値が表示できます.

```javascript
document.getElementById('myInput').value = "Taro";
```

　これをコンポーネント化します. ポイントは二つでvalueとハンドラを定義するだけです.

```javascript
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

　ユーザー入力はハンドラを介して1度stateに保存され, それが更新時に反映されます. 初期値がある場合もstateにまず値を割り当てます. サブミットでサーバー側とやりとりする場合はSubmitイベントのハンドラで行えば良いです.

### テキスト・エリア

　テキスト・エリアもやることは同じです.

```jsx
<form onSubmit={this.handleSubmit}>
    <label>
        Essay:
        <textarea value={this.state.value} onChange={this.handleChange} />
    </label>
    <input type="submit" value="Submit" />
</form>
```

### 選択肢

　select要素も基本的に同じです.

```jsx
<form onSubmit={this.handleSubmit}>
    <label>
        Pick your favorite flavor:
        <select value={this.state.value} onChange={this.handleChange}>
        <option value="grapefruit">Grapefruit</option>
        <option value="lime">Lime</option>
        <option value="coconut">Coconut</option>
        <option value="mango">Mango</option>
        </select>
    </label>
    <input type="submit" value="Submit" />
</form>
```

　ちなみに複数選択の場合は配列を渡してやります.

```jsx
<select multiple={true} value={['Monday', 'Sunday']} />
```

## Uncontrolled Component

　input要素のtype属性をfileにした場合valueは読み込み専用なのでコンポーネントからは管理できません. こうしたコンポーネントをUncontrolled Componentと呼びます.

```jsx
<input type="file" />
```

## 合成フォーム

　通常フォームはもっと複雑なものです. この場合name属性を使ってどの属性からの入力かを判断します[<sup>1</sup>](#com-1).

```jsx
<form>
    <label>
        Is going:
        <input
        name="isGoing"
        type="checkbox"
        checked={this.state.isGoing}
        onChange={this.handleInputChange} />
    </label>
    <br />
    <label>
        Number of guests:
        <input
        name="numberOfGuests"
        type="number"
        value={this.state.numberOfGuests}
        onChange={this.handleInputChange} />
    </label>
</form>
```

　name属性の値を取得するにはtargetを使いますが,  computed property nameというのを使うと便利だそうです[<sup>2</sup>](#com-2).

```javascript
handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
}
```

 当然コンポーネント間を複数のControlled Componentに分割してもいいでしょう.

## まとめ

　Reactでフォームを作詞する場合は, value属性という共通の性質を用いてフォームの状態をコンポーネント側で受け持つようにします.

## Further Reading

+ [Uncontrolled Components](https://reactjs.org/docs/uncontrolled-components.html)
+ [Formik](https://jaredpalmer.com/formik/)

## 注釈
1. <a name="com-1"></a> [Handling Multiple Inputs](https://reactjs.org/docs/forms.html#handling-multiple-inputs)
2. <a name="com-2"></a> [Computed property namesSection](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names)
