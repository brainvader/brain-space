+++
title= 'Rust入門 (3) Thread'
description= 'RustのSafe Threadを学ぶ'
date = 2019-05-08T21:24:13Z

[taxonomies]
categories = ["programming"]
tags= ["Rust"]
+++

# Rustにおける平行処理

## 平行処理とは?

> Concurrency is the process of breaking a program down into units that can be executed in any arbitrary order. [<sup>1</sup>](#ref-1)

## スレッドとは?

　対してスレッドは順番に処理しないといけない処理をまとめたものです.

> A thread is a block of instructions that must be executed in order. [<sup>1</sup>](#ref-1)


## スレッドの種類

+ OS thread
+ Green Thread

 OSスレッドはOSによって管理されるスレッドです. プログラムは通常１つのプロセスを持ち, このプロセスが複数のスレッドを生成します. Rustは標準ライブラリでOSスレッドに対応している. 一方のグリーン・スレッドは仮想マシン上で管理されるスレッドらしいです.[<sup>4</sup>](#ref-4) スレッドは平行処理を実現してくれるが, データの競合が発生する. Rustで安全とはこのデータの競合をコンパイラ時に教えてくれることを指します. つまりコンパイル・エラーになるので, データ安全あコード以外はコンパイルできません.

## thread::spawnとjoin

  spawnはJoinHandleを返す. joinを呼び出すとスレッドの処理が終わるまで待ってくれる.

> The return type of thread::spawn is JoinHandle. A JoinHandle is an owned value that, when we call the join method on it, will wait for its thread to finish. [<sup>2</sup>](#ref-2)

## キャプチャ

　JavaScriptのアロー関数は独自のスコープを持たないが, その周囲(といっても先に宣言している必要があるが)で定義した変数などをそのまま使える. こういう動きを変数のキャプチャと呼ぶ. 関数ならローカル変数となり, 宣言文が必要になる. Rustのクロージャもこのような性質があるが, 通常は問題にならない. ただし平行処理の場合は, 所有権が問題になる. スレッドはメイン・スレッドとは切り離されて実行される.

　以下のようにクロージャの場合vは宣言されたわけではない. 外側のmain関数の変数がキャプチャされたことになる.

```Rust
use std::thread;

fn main() {
  let mut v = vec![1,2,3];
  thread::spawn(|| {
     v.push(4);
  });
}
```

　しかしこのクロージャの中身は別のスレッドとして実行される. 当然その間にメイン関数が終了する可能性がある. Rustはこれをちゃんとコンパイル時に教えてくれる. ここでキャプチャではなく所有権の移動が起きてほしい. つまり普通の関数のようにしたい. その場合moveを明示する.

```Rust
thread=:spawn(move || {
  v.push(4);
});
```

## データ共有
　
　スレッド間でデータを共有できたら平行処理の恩恵を最大限受けられる. Rustでは可変なデータの所有権を共有は基本的には制限されている. つまり可変な借用は１つしか作れない. これを回避する方法はいくつかあるが, スレッドの場合あArcとMutexを使う.

### Atomically reference counted (Arc)

 リファレンス・カウントで共有データの所有権を管理する. このカウンターが0になるまではデータは解放されない.

### Mutex (Mutual exclusion)

 Mutexはデータをアクセスをロックする排他制御を提供するライブラリです.

### Arc & Mutex

　この２つを使うと

+ データを複数のスレッド間で共有する (共有所有権)
+ データの競合を防ぐ (排他制御)

　を実現できる. Rcと同じように使う. cloneする毎に参照カウンタは増加していき, スレッドが終了すると減少する. 最終的に0になった時にv自体が解放される.

```Rust
// valid
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let v = Arc::new(Mutex::new(vec![1,2,3]));

    for i in 0..3 {
      let cloned_v = v.clone();
      thread::spawn(move || {
         cloned_v.lock().unwrap().push(i);
      });
    }
}
```

## スレッドはどのように作られるか. [<sup>5</sup>](#ref-5)


## Reference
1. <a name="ref-1"></a>  [Safe Concurrency with Rust](http://squidarth.com/rc/rust/2018/06/04/rust-concurrency.html)
2. <a name="ref-2"></a>  [Fearless Concurrency](https://doc.rust-lang.org/book/ch16-00-concurrency.html)
3. <a name="ref-3"></a>  [Fearless Concurrency with Rust](https://blog.rust-lang.org/2015/04/10/Fearless-Concurrency.html)
4. <a name="ref-4"></a>  [グリーンスレッド](https://ja.wikipedia.org/wiki/%E3%82%B0%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B9%E3%83%AC%E3%83%83%E3%83%89)
5. <a name="ref-5"></a>  [Where do Rust threads come from?](http://squidarth.com/rc/rust/concurrency/2018/06/09/rust-threads-detach.html)
