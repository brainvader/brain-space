+++
title = 'React入門 (8) 親子コンポーネントの関係'
description =  'React入門'
date = 2019-04-16T12:50:22Z

[taxonomies]
categories = ["programming"]
tags = ["React"]
+++

# コンポーネントの関係

## 関係

+ 親から子
+ 子から親
+ コンポーネント渡し
+ 合成

## Unidirectional Data Flow

　通常Reactではデータは親から子へと渡されます. propsには値以外にもハンドラなども渡すことができます. 多くの場合親コンポーネントは子コンポーネントについて知る必要はありません.

## childrenプロパティ

　propsにはchildrenプロパティを通じて子コンポーネントのJSX要素にアクセスすことができます. やり方は簡単で, 単純にコンポーネントに閉じタグを加えて, 親に渡したい内容を挟み込むだけです.

```jsx
<ParentComponent>
　 <h1>Inserted Content</h1>
   <p>This is inserted on the child component</p>
</ParentComponent>
```

　親ではこれをchildrenプロパティから取り出して展開します.

```jsx
<div>
  {props.children}
</div>
```

## コンポーネント渡し

　コンポーネントを引数として渡すこともできます.

```jsx
<ParentComponent
  first={<FirstChild/>}
  second={<SecondChild />}
/>
```

 同じようにParentComponentで展開します.

 ```jsx
 <div>
   {props.first}
   {props.second}
 </div>
 ```

 ## 合成

  独自のコンポーネントはextendsでは継承しません.

```javascript
class SuperComponent extends MyComponent { 
    // ...
}
```

　独自コンポーネントを拡張するには合成で行います. 合成といってもラッパー・コンポーネントを使うだけです.

```javascript
class Wrapper extends React.Component {
    render() {
        return (
            <CoreComponent name="Super-Component">
               <h1>Dump on the parent component</h1>
               <p>This is a content of the child component. Is that properly inserted?</p>
            </CoreComponent>
        );
    }
}
```

## まとめ

　JSX要素としてコンポーネントを使う場合は, createElementでPOJO(React要素とも言う)に変換されることを思い出しましょう. そうすればなぜpropsに渡せるのかがわかるはずです. 
