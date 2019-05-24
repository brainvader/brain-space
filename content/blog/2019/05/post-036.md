+++
title= 'Rust入門 (5) 複数のバイナリを持つパッケージの作成'
description= '複数のバイナリを指定する'
date = 2019-05-14T23:51:53Z

[taxonomies]
categories = ["programming"]
tags= ["Rust"]
+++

# 複数のバイナリを管理する

## binファイルの作成

　binというファイルがいるようです.[<sup>1</sup>](#ref-1)

## Cargo.tomlに複数のbinセクションを指定する. 

　次にCargo.tomlに以下のように書く.

```toml
[[bin]]
name = "project_name"
src  = "src/main.rs"

[[bin]]
name = "second"
src  = "src/bin/second.rs"
```

　後はnameで指定した名前をオプションとして渡す.

```sh
cargo run --bin second
```

## feature属性の利用

 すでに[OpenGL](/2019/04/post-017)の初回で説明した.

## Reference
1. <a name="ref-1"></a> [rust_multibin](https://github.com/timglabisch/rust_multibin)
