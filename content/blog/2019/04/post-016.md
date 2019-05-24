+++
title = 'Rust雑記 (1) Dpendency Inversion'
description =  'Rustについての雑多なメモ'
date = 2019-04-24T21:59:34Z

[taxonomies]
categories = ["programming"]
tags = ["Rust"]
+++

# Rust雑記 (1)

## RustでDependency Inversion

　Clean Architectureの日本語記事が増えて来た気がして, 大昔Pythonで失敗したことを思い出した. 当時はFlaskを使ってやって気がするが, Pythonでインターフェースに依存する書き方ができなくて詰んだのだった. PHPやScalaならできるんだろう(Javaは況や). 調べるとDIを使ったりもするらしい. そうするとLaravelなら実装できそうって話になるけど, あえてのRsutではって話です.

## トレイト

　トレイトとは直訳すると特性という意味になる. あるデータ型の特性を定義することになる. 特性って何というと振る舞いで, 要するに関数を定義する. 内容的におかしいのだが, とりあえず例を挙げておく.

```Rust
trait Hoge {
  fn do_something() -> ();
}

struct Data;

impl Hoge for Data {
  fn do_something() {
    println("Hello World");
  }
}
```

　他の言語で言う所のインターフェースということになるが, 少し違うのはトレイトは型らしい. なのでトレイトを継承(?)することができる.

```rust
trait Service {}
trait SpecialService: Serevice {}
```

## Associated Type

　インターフェースと違ってデータ型を定義できる. これを関連型という.[<sup>1</sup>](#com-1)

```rust
// Graph Trait
trait Graph<N, E> { /* function declarations */ }

// Graph Trait with Associated Types
trait Graph {
  type N;
  type E;

  /* function declarations */
}
``` 

　上はGenericを使ってグラフ型のトレイトにNodeとEdgeを定義したことになるが何かと煩雑になる. 


## Cakeパターン

　CakeパターンってのはScalaで考え出されたパターンらしい.[<sup>2</sup>](#com-2) トレイトと関連型を使うのだが実装はReference読んで. ポイントはトレイト型を関連型として持つトレイト型を作ること(実装見て).

## DIって何よ?

　内容読むとどの辺が依存性注入の話なのか良く分からない. Dpendency Inversionの話にしか聞こえない.

## 注釈
1. <a href="com-1"></a> [RustのAssociated Typeについて](https://qiita.com/tacke_jp/items/9c7617971dc341146c0f)
2. <a href="com-2"></a> [Scalaでいろんな DIを試してみる Cake 〜 Free+ReaderT](https://qiita.com/yasuabe2613/items/d1e0451024f61145f875)


## Reference 
+ [RustのDI](https://keens.github.io/blog/2017/12/01/rustnodi/)
