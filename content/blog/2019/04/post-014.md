+++
title = 'Rust入門 (1)'
description =  'Rust'
date = 2019-04-20T13:35:29Z

[taxonomies]
categories = ["programming"]
tags = ["Rust"]
+++

# Rust入門

## Hello World

　RustでHello Worldしてみましょう. 関数名は基本的にスネーク・ケース(snake case)です. 文字列はダブル・クオートで囲みます.[<sup>1</sup>](#com-1)

```rust
fn say_greeting() {
  println("Hello World");
}
```

## 注釈
1. <a href="com-1"></a> 実はダブル・クオートで囲むだけでは文字列型ではなく&`static strという型らしいです.
