+++
title= 'Functional Programming in Rust'
description= 'Rustで関数型言語を学ぶ'
date = 2019-05-08T18:28:18Z

[taxonomies]
categories = ["programming"]
tags= ["Rust"]
+++

# Rustで関数型言語

## 概要

 [<sup>1</sup>](#ref-1)をベースに関数型言語の考え方をRustで学習する.

## Rustは関数型のプログラミング言語?

　RustはC++と同じくマルチ・パラダイム言語らしいです.

## 関数型言語って? (What is a functional programming?)

　Rustで関数言語風に書くには, 以下のような概念を利用します.

+ Generics
+ Higher Order Function
+ Functions as values
+ Ternary Operators
+ Iterator
+ Pattern Matching
+ Expression
+ Direct Constructor Expression
+ Tagged Union Expression [<sup>3</sup>](#ref-3)

　関数型言語では全ての項は式です. イテレータはコンポーザビリティを提供します.


### クロージャとは何か?

　クロージャは関数ではなく関数のように振る舞うオブジェクトのようです.

>  A closure is an object that acts as a function, ...

　以下のいずれかのトレイトを実装しているオブジェクトとも言えます.

+ fn
+ Fn
+ FnMut
+ FnOnce

 値として扱われるため, 引数として関数に渡したり戻り値として返すことができます. このような関数を高階関数(Higher order Function a.k.a HoF)と呼びます.

### パターン・マッチとタグ付き共有型

　C言語にも共有型がありましたが, 色々と問題があったようです.[<sup>4</sup>](#ref-4) Rustの共有型はタグ付きといって属性を追加できます. この属性もパターン・マッチの対象とすることができます.

### Complex rules ownership

　関数のローカル変数を戻り値とすることはRustでは基本的にはできません. なぜなら関数のスコープとともにローカル変数が解放されるからです. clone(deep copy)を使うこともできますが, これではゼロ・コストを謳うRustらしくはありません. ローカル変数は関数の依存性なので外部から参照として渡してやることで解決できます(これをRustは借用と呼んでいます). 関数は処理だけを行えばいいのです.

### Fearless Concurrency [<sup>5</sup>](#ref-5)

　Rustは型安全かつコンパイラに強制されます. 後者は特にコンパイル時に競合の発生やデッドロックを教えてくれます.

> All of this concurrency is type-safe and compiler-enforced.

<!-- TODO: 少し早かったのでRustを先にちゃんと理解する. -->

## Reference
1. <a name="ref-1"></a>  [Hands-On Functional Programming in RUST](https://www.packtpub.com/application-development/hands-functional-programming-rust)  
2. <a name="ref-2"></a>  [Hands-On-Functional-Programming-in-Rust](https://github.com/PacktPublishing/Hands-On-Functional-Programming-in-Rust)  
3. <a name="ref-3"></a>  [A quick tour of Rust’s Type System Part 1: Sum Types (a.k.a. Tagged Unions)](https://tonyarcieri.com/a-quick-tour-of-rusts-type-system-part-1-sum-types-a-k-a-tagged-unions)
4. <a name="ref-4"></a>  [How Rust Implements Tagged Unions](http://patshaughnessy.net/2018/3/15/how-rust-implements-tagged-unions)
5. <a name="ref-5"></a>  [Fearless Concurrency](https://doc.rust-lang.org/book/ch16-00-concurrency.html?highlight=concur#fearless-concurrency)
6. <a name="ref-6"></a>  [Safe Concurrency with Rust](http://squidarth.com/rc/rust/2018/06/04/rust-concurrency.html)
