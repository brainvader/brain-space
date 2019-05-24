+++
title = "React入門 (5) リストとmap"
description =  "React入門"
date = 2019-04-16T10:50:22Z

[taxonomies]
categories = ["programming"]
tags = ["React"]
+++

# リストとmap

## リストとキー

　よくあるTODOでもそうだが, 繰り返し要素(repetitive element)は有用です. Reactではmapが利用されます. この際要素のReactが要素を特定するためにキーが使われます. キーがない場合は警告が出ますので分かると思います.

```javascript
// Todo definition
const todos = [{id: 01, title = 'hoge'}];
// Create todo items 
const todoList = todos.map((todo, index) => (
    // Don't use index as key
    <li key={todo.id}>{todo.title}</li>
));

return <ul>{todoList}</ul>
```

　キーとしてindexを使うことはアンチ・パターンとされ推奨されていません[<sup>1</sup>](#com-1). 順番などが変わった場合にパフォーマンスに悪影響を及ぼすかららしいです.

```jsx
<li key={index}>{todo.title}</li>
```

## コンポーネントなリスト

　リスト項目をコンポーネント化することも行います. この場合内部のJSX要素で指定するべきでしょうか? それともコンポーネント側でするべきでしょうか. 答えばコンポーネント側です.

```jsx
<ListItem key={todo.id} title={todo.title}/>
```

## キーはヒント

　キーはReactが利用するヒントです. 他のプロパティと違いpropsからアクセスすることはできません. 明示的に渡してやる必要があります.

```jsx
<ListItem key={todo.id} id={todo.id} title={todo.title}/>
```

## まとめ

　繰り返し要素に対してはmapを使い, その場合はここの要素を特定するためにキーを用いる.

## 注釈
1. <a name="com-1"></a> [Index as a key is an anti-pattern](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318)
