+++
title = 'Learn OpenGL with Rust (1) Getting Started'
description =  'RustでOpenGL入門'
date = 2019-04-24T23:09:26Z

[taxonomies]
categories = ["programming"]
tags = ["Rust", "OpenGL"]
+++

# 第1回 Getting Started

## Learn OpenGLとは?

　C++でOpenGLに入門できるサイトです.[<sup>1</sup>](#com-1)

## Getting Started

　オリジナルは色々設定が大変みたいだが, Rustでは新規プロジェクト作ってglfw-rsを導入すればできます. 面倒な人はgit cloneでプロジェクトをローカル環境に持って来てもできるはずです. いずれにしろ下記のglfwの導入は必要です.

## glfw

　glfwだけ少し色々やる必要があります.[<sup>2</sup>](#com-2) まずglfw3をbrewでインストールします. この時cmakeが必要になります.

```rust
brew install glfw cmake
```

　後は[Cargo.toml](https://github.com/bwasty/learn-opengl-rs/blob/master/Cargo.toml)のdependenciesの項をコピーペすればオッケーです. バージョンが更新されているものもは更新しました.

## 条件付コンパイル (Conditional Compilation)

　 cfgマクロを使うと条件付きコンパイルができる.[<sup>3</sup>](#com-3) Cargo.tomでfeaturesでフラグを設定できる.

```toml
[features]
default =[
  "chapter-1"
]

# no extra dependencies
chapter-1 = []
```

　このフラグをcargoの--featuresオプションに渡してやる.

```sh
cargo run --features "chapter-1"
cargo build --features "chapter-1"
```

　モジュールの前にcfgマクロを定義すると, 引数として渡された条件を満たす場合にコンパイルされモジュールが読み込まれる.

```rust
#cfg[feature = "chapter-1")]
mod _1_getting_started;
```

　この場合_1_getting_startedが使えるようになる.


## モジュール

　次はモジュールが必要になる. 各章のソースはモジュールとして管理されている. \_1\_getting_startedにmod.rsがあり, ここでまとめて公開されている. ファイル名がモジュールとなる. ファイルに定義された関数はpubと定義されているものだけが公開される. useを使うとパスを省略できるので, 関数名だけを読込先で使える.

```rust
mod greate_module;
use great_module::*;
```

　とするとgreat_funをそのまま呼び出せる.
　

## サブ・メイン関数

　cargo runを--featuresオプション付きで実行すると条件付きコンパイルで章ごとにモジュールが取り込まれてビルドが実行される. しかしこれではいちいちmain.rsの内容を章の内容に合わせて書き直す必要がある. そのため引数でタイプ・キーで章を指定できるようにしている. 該当する章の"サブ・メイン関数"が実行される.[<sup>4</sup>](#com-4) 1章1節1項なら, 1\_1\_1を指定する.

  ```sh
  cargo run 1_1_1
  ```

　main.rsがビルドされて実行されるが, 実際に実行されるのは以下の部分になる.

  ```rust
  // sub main function for 1.1.1
  pub fn main_1_1_1() { /* omit */ }
  ```

## まとめ

　cargoがあるとはいえcmakeが必要だったりして, 結構前準備がめんどくさかった. Rustのこの辺は良くわかっていなかったので良い復習になった. またバイナリとしてプロジェクトを構築した場合に, 例をどうやって動的に切り替えて実行するかの良い例になっていて真似しようと思った.

## 補足

### glutin

　Rust版GLFWらしいですが使わないようです.

### glium

　OpenGLのラッパーです. 便利らしいのですがもうメンテナンスもされていないしので使わないようです.

## 注釈
1. <a href="com-1"></a> [Learn OpenGL](https://learnopengl.com/)
2. <a href="com-2"></a> [GLFW on Rust 環境構築](https://qiita.com/seibe/items/1c7388cfc216b29a8b75)
3. <a href="com-3"></a> [Conditional Compilation](https://doc.rust-lang.org/1.30.0/book/first-edition/conditional-compilation.html)
4. <a href="com-4"></a> こんな呼び方はしないが良い表現が思いつかない.


## Reference 
+ [RustのDI](https://keens.github.io/blog/2017/12/01/rustnodi/)
