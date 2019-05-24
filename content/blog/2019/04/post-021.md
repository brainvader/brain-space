+++
title = 'Elm入門 (1)'
description =  'Elm入門'
date = 2019-04-26T17:20:19Z

[taxonomies]
categories = ["programming"]
tags = ["Elm"]
+++

# Elm入門 (1)

## Elmとは？

　第1回でそんなものは分からない. ReduxがElmアーキテクチャを参考に作られていたらしい. また関数型のプログラミング言語だそうだ. AltJSなので最終的にはJavaScript(とHTML)が生成される. またRustの[yew](https://github.com/DenisKolodin/yew)というUIフレームワークもElmに触発されたプロジェクトらしい. 基本的な構造は(代数的データ型)のメッセージをパターン・マッチで分岐させて処理する.

## Elmリンク集

　とりあえず第1回は入門できそうなサイトをまとめておく.

1. [Elm について (はじめに)](https://guide.elm-lang.jp/) [<sup>1</sup>](#com-1)
2. [Beginning Elm](https://elmprogramming.com/)
3. [Elm Syntax](https://elm-lang.org/docs/syntax) [<sup>2</sup>](#com-2)　

## 型と関数

　関数は型注釈と定義から構成される. Msg型とModel型の引数を受け取って, Model型を返すという意味になる. カスタム型では代数型データ型を定義する. **バリアント**というらしい. この型情報を元にパターン・マッチで処理を分岐させる.


```elm
-- custom data type
type Msg = Increment | Decrement
-- type anottation
update : Msg -> Model -> Model
-- functin definition
update msg model = 
  -- pattern match
  case msg of
    Increment ->
      model + 1

    Decrement ->
      model - 1
```  


　型は実は関数で引数を取れたりもします. 垂直線で型と引数のペアが区切られると解釈しますのでRegular型とVisitor型が定義されていることになります. 両方String型を第1引数にとりますがRegular型はInt型を第2引数として取ります.


```elm
type User
  = Regular String Int
  | Visitor String
```  

## パターン・マッチ

　パターン・マッチでバリアントごとの処理を分岐します. またパターン・マッチに不要な引数は\_で省略できます.

```elm
toName : User -> String
toName user =
  case user of
    Regular name age ->
      name

    Visitor name ->
      name

toName(Regular "Thomas" 44)
toName(Visitor "Thomas")
```

## Msgパターン

　パターンというべきか分かりませんが, 良く使われます. パターン・マッチが無い言語では文字列などを使ってswitch文で同じことをしていたりしますが(Reduxが確かそうです), こちらの方が可読性は高いと思います. しかも型情報があるのでエラーも適切に出せます(たぶん).

```elm
-- custom data type
type Msg = Increment | Decrement
-- type anottation
update : Msg -> Model -> Model
-- functin definition
update msg model = 
  -- pattern match
  case msg of
    Increment ->
      model + 1

    Decrement ->
      model - 1
```

## Browser.sandboxとBrowser.element

　ElmはコードとRuntime Systemから成立している. Browser.sandboxはHTMLだけをRuntimeに送るらしい. RuntimeでDOMの変更の計算などを行います. その結果をメッセージとしてElmコードに返します. updateが受け取るメッセージはElmコードの外からやってくるということにようです.

 ![Browser.sandbox](https://guide.elm-lang.jp/effects/diagrams/sandbox.svg)

```elm
import Browser
main =
  Browser.sandbox { init = init, update = update, view = view }
```

　Browser.elemetはHtml以外にもコマンド(Cmd)やサブスクリプション(Sub)という値を送り出します. これらはRuntimeに対する指示です. 指示の結果をやはりメッセージとして返してきますので, それをupdateで受け取ります.

  ![Browser.element](https://guide.elm-lang.jp/effects/diagrams/element.svg)

```elm
import Browser

main =
  Browser.element
    { init = init
    , update = update
    , subscriptions = subscriptions
    , view = view
    }
```

  コードだけ見ているとどれとどれが繋がるのかは良くわかりませんが, Html Msg, Cmd Msg, そしてSub Msgという部分を見つけたらこれはRuntimeに指示を出しているのだと思えばいいと思います. Runtimeが計算や処理を行って, メッセージを返してくるわけです. メッセージはupdateで受け取って更新処理(モデルの更新など)を行います.

  こうした流れが分かると宣言的かつ疎結合に書けてElmアーキテクチャの見通しの良さが理解できると思います.

## HTTP

　

## 注釈
<a name="com-1"></a> 1. [2019年 Elmをはじめる人が最初に読むページ](https://qiita.com/arowM/items/5ec5853298fc880353b7)によると1を読むべしということなので1を読む. 少し読んだ感じでは非常に良く書かれているようだ. 同じ著者のElm本もある.
2. <a name="com-2"></a> [わざわざ学ばなくてもいいプログラミング言語一位Elm](https://qiita.com/ababup1192/items/4c39ff981642aded5d8e)によると文法が少ないらしい.

