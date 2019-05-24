+++
title= 'Learn OpenGL With Rust (2) GetString'
description= 'gl::GetStringで取得した情報の表示'
date = 2019-05-05T11:40:46Z

[taxonomies]
categories = ["programming"]
tags= ["Rust"]
+++

# GetStringでバージョンとレンダラ情報を取得して表示する

## 背景

　Learn OpenGL with Rust[<sup>1</sup>](#ref-1)で画面だけ表示しても面白みがないので, メタ情報へのアクセスを試みてみた. C++では[glGetString](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glGetString.xhtml)に以下のGLenumを渡すと文字列を返してくれる.

+ GL\_VENDER
+ GL\_VERSION
+ GL\_RENDERER
+ GL\_SHADING\_LANGUAGE\_VERSION

　gl-rsでは以下の定数を使う.

+ gl::VENDER
+ gl::VERSION
+ gl::RENDERER
+ gl::SHADING\_LANGUAGE\_VERSION

　Rustでもgl::GetStringに同じように渡すと文字列(String型とか&str型)が返ってくる.

```rust
let version = gl::GetString(gl::VERSION);
```  

　とお思いきや,

```rust
*const u8
```

　という型が返ってくる. これはraw pointerというやつである(Cのポインタって理解で良い?). またこの型には(当然)Displayトレイトが実装されていないので表示できない. クエスチョン・マークを使うと表示はできるのだが, ポインターなので値としてはアドレスが入っている.

```rust
// enclose an unsafe block when calling unsafe functions
unsafe {
  let version_addr = gl::GetString(gl::VERSION);
  println!("{:?}", version_addr);
};
```

　[Dereferencing a Raw Pointer
](https://doc.rust-lang.org/1.30.0/book/2018-edition/ch19-01-unsafe-rust.html#dereferencing-a-raw-pointer)を見るとそのままいけそうだが, 実は中身はC ABI(C Application Binary Interface)とかいうやつが返すUTF-8文字列らしいのです. OpenGLがCインタフェースへのラッパーなのだからある意味当然だが, これをRust側で安全に使うにはCStrというモジュールが必要になる.[<sup>2</sup>](#ref-2)

　CStrモジュールの[from\_ptr](https://doc.rust-lang.org/std/ffi/struct.CStr.html#method.from_ptr)を使うと,

> Wraps a raw C string with a safe C string wrapper.

　となる. これでRustの内部で使えるCStr型の変数となる. これはNull終端文字列への借用された参照らしい(意味不明).
　
```rust
use std::ffi::CStr;

let version = unsafe {
  let version = gl::GetString(gl::VERSION);
  println!("{:?}", version);
  CStr::from_ptr(version as *const i8)
};
println!("{:?}", version);
```

　参考にしたQiitaの記事では[to\_string\_lossy](https://doc.rust-lang.org/std/ffi/struct.CStr.html#method.to_string_lossy)使ってる. これは文字列が有効なものでない場合に適当な無効を表す文字列に置き換えてくれるらしい.[<sup>3</sup>](#ref-3)

### まとめ

　CStrはよくわからないこともあるが(COW型とか), C++とのやりとりのやり方として勉強になった. 後最初GetIntegervに渡すべきgl::MAJORとかgl::MINORを渡してSegment Faultをくらってたのは内緒です.

## Reference
1. <a name="ref-1"></a> [Learn Opengl with Rust](https://github.com/bwasty/learn-opengl-rs)
2. <a name="ref-2"></a> [Struct std::ffi::CStr](https://doc.rust-lang.org/std/ffi/struct.CStr.html#method.as_ptr)
3. <a name="ref-3"></a> [WindowsでRust環境を作ってGtk3でOpenGLする](https://qiita.com/ousttrue/items/ee617544ab737fc34c1d)
