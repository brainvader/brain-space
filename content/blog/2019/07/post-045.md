+++
title = "Goならわかるシステムプログラミング"
description = "Goで低レベルプログラミング"
date = 2019-07-06T23:02:16Z

[taxonomies]
categories = ["programming"]
tags = ["Go", "Rust"]
+++

## ファイル・ディスクリプタ

　これでアクセスできるらしい.

## io.Writerインターフェース

```go
io.Writer
```

　Rustではstd::io::Writeに対応する.

### バイト列

　Goではバイトを配列として表します.

```go
byteArray := []byte("ASCII")
```

　Rustではスライス型を使います.

```rust
let s: &str = "ASCII";
let bytes: &[u8] = s.as_bytes();
```

　もしくはバイト文字列リテラルを使う方法もある.

```rust
let s: &[u8] = b"ASCII";
```
　
　これもスライス型なので範囲で指定できます.{{ ref(id=1) }}

```rust
use std::io::prelude::*;
use std::fs::File;

fn main() -> std::io::Result<()> {
    let data = b"some bytes";

    let mut pos = 0;
    let mut buffer = File::create("foo.txt")?;

    while pos < data.len() {
        let bytes_written = buffer.write(&data[pos..])?;
        pos += bytes_written;
    }
    Ok(())
}
```

### std::io::Stdout構造体

　GoではWrite関数を使います.

```go
os.Stdout.Write([]byte("os.Stdout example\n"))
```

　RustではStdout構造体を持ちいいます. この構造体はWriteトレイトを実装しています.

```rust
use std::io::{self, Write};

fn main() -> io::Result<()> {
    io::stdout().write(b"os.Stdout example\n")?;

    Ok(())
}
```

### バッファリング

> バッファは処理速度や転送速度の差がある複数の情報機器やソフトウェアの間でデータをやり取りするときに、データを一時的に保存して差を速度差を埋めるための記憶装置や記憶領域のことです。{{ref(id=2)}}

　ストリーミングなどでも遅延をなくすために一定量のフレームデータがメモリ上にバッファとして蓄えられます. GoではBuffer構造体のWrite関数にバイト列を指定するだけでメモリ上にバッファリングできるようです. Rustにはそれに対応するものはないようです.

> ... It also provides no advantage when writing to a destination that is in memory, like a Vec\<u8\>. {{ ref(id=3)}}

　BufWriterとVec\<u8\>に性能差は無いようなので, Vec\<u8\>を使ってみることにする.

```rust
use std::io::Write;

fn main() {
    let mut buffer: Vec<u8> = Vec::new();
    let data: &[u8] = b"ASCII";
    buffer.write(data).unwrap();
    buffer.write(data).unwrap();
    println!("buffer: {:?}", buffer);
}
```

## Reference

{{ anchor(id=1) }} [Trait std::io::Write Examples](https://doc.rust-lang.org/std/io/trait.Write.html#examples)
{{ anchor(id=2) }} [バッファ/バッファリング](https://dictionary.goo.ne.jp/leaf/strmg/%E3%83%90%E3%83%83%E3%83%95%E3%82%A1%252F%E3%83%90%E3%83%83%E3%83%95%E3%82%A1%E3%83%AA%E3%83%B3%E3%82%B0/m0u/)
{{ anchor(id=3) }} [Struct std::io::BufWriter](https://doc.rust-lang.org/std/io/struct.BufWriter.html#method.with_capacity)
