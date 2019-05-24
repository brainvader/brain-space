+++
title = 'Rust入門 (2) テスト'
description =  'テストの書き方と実行方法'
date = 2019-04-27T16:39:11Z

[taxonomies]
categories = ["programming"]
tags = ["Rust"]
+++

# Rust入門 (2)

## 前提条件

　プロジェクトはlibraryとして生成します. プロジェクト名はgeometryとします(特に意味はないです).

```rust
cargo new geometry --lib
```

　ライブラリとしてプロジェクトを生成すると勝手にテストの雛形が生成されますので, 後はコマンド一発でテストが実行できます.

```bash
cargo test
```

## 単体テスト

　Cargoが自動で生成するテスト内容はこんな感じになります.

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

　#[cfg(test)]属性をつけてtestsモジュール以下にテストを記述します. #[test]属性がついた関数をテストと認識します. では実際に関数を定義してテストしてみましょう.

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(1, 2), 4);
    }

    #[allow(dead_code)] // to remove an unused-code warning
    fn no_test() {
        assert_eq!(2 + 2, 4);
    }
}
```

　addというなんの変哲も無い関数を定義してテストしますが, 失敗するはずです. useは外側の名前を取り込むための行です. もう一つのno_testは#[test]属性がついていないので, ただの関数定義ですので実行されません(#[allow(dead_code)]属性はunused codeの警告を消すため).

## 結合テスト

　今度は結合テストを行います. プロジェクト直下にtestsというフォルダを作り, 以下のファイルを保存します.

```rust
// integratin_test.rs
extern crate geometry;  // your project name by defaul
use geometry::*;

#[test]
fn test_add() {
    assert_eq!(add(3, 2), 5);
}
```

　なおtestsフォルダ以下のファイルは別々のクレートとして実行されます. テスト共通の処理などを記述したい場合はtestsフォルダ内に記述します. testスイートなどのsetupとかteardownを書けばいいのかもしれません. モジュールの扱い方の復習もかねて書いておきましょう.


```rust
// common.rs
pub fn setup() {
    // some setup code, like creating required files/directories, starting
    // servers, etc.
}
```

　モジュールにしたい場合は, 適当なフォルダ(ここではsharedとする)以下に以下のようなファイルを配置します.

```rust
// mod.rs
pub mod shared;
```


```rust
// shaerd.rs
pub fn teardown() {
  println!("called `shared::function()`");
}
```


　後はintegration_test.rsで呼び出すだけです.


```rust
extern crate geometry;
use geometry::*;

mod common;
mod shared;
use shared::shared::*;

#[test]
fn test_add() {
    common::setup();
    assert_eq!(add(3, 2), 5);
    teardown();
}
```

　ちょっと変な書き方ですが色々なやり方が網羅できたと思います.

## dev-dependencies

　テスト用の依存性はdev-dependenciesセクションに記述します.

```toml
[dev-dependencies]
pretty_assertions = "0.4.0"
```

　このクレートを#[cfg(test)]属性以下で導入します.

```rust
#[cfg(test)]
extern crate pretty_assertions
```

　こんな感じで外部のクレートを使ったテストも実行できます.

## ドキュメンテーション・テスト

　ドキュメントにコードを書くとテストしてくれるやつです. ドキュメントはMarkdownで書けるらしいです.

```rust
/// The next lines present detailed documentation. Code blocks start with
/// triple backquotes and have implicit `fn main()` inside
/// and `extern crate <cratename>`. Assume we're testing `doccomments` crate:
///
/// ```
/// use geometry::add;
/// let result = add(2, 3);
/// assert_eq!(result, 5);
/// ```

pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

## パニック・テスト

　パニックになることをテストしたい場合があります. 例としてゼロ除算でパニックが起こることをテストしましょう.

```rust
pub fn divide(numerator: u32, denominator: u32) -> u32 {
    if denominator == 0 {
        panic!("Divide-by-zero error");
    } else if numerator < denominator {
        panic!("Divide result is zero");
    }
    numerator / denominator
}
```

　後は単体テストと同じようにテストを追加すればいいのですが, その際#[should_panic]属性を追加します. これでパニックがおきた場合にテストが通ります. なおexpectedパラメータにはpanicマクロに与えるメッセージと同じものを指定します. これはテスト失敗がきちんと想定した場所で起きていることを確かめるのに使えるそうです.

```rust
#[test]
fn test_divide() {
    assert_eq!(divide(10, 2), 5);
}

#[test]
#[should_panic]
fn test_divide_by_zero() {
    divide(1, 0);
}

#[test]
#[should_panic(expected = "Divide result is zero")]
fn test_devided_by_larger_denominator() {
    divide(1, 10);
}
```

## テストを指定して実行

　テスト名を指定して実行することもできます.

```bash
cargo test test_name
```

## テストを除外する

　テストから除外するにはignore属性を利用します.
　
```rust
#[ignore]
```

　除外したテストをのみを実行する場合はignoreオプションを使います.

```bash
cargo run -- --ignored
```

## マクロ
　
　マクロを列挙する予定.

## Reference
+ [Testing on Rust By Example](https://doc.rust-lang.org/rust-by-example/testing.html#testing)
+ [Tests on Cargo Book](https://doc.rust-lang.org/cargo/reference/manifest.html#tests)
+ [Writing Automated Tests on The Rust Programming Language](https://doc.rust-lang.org/book/ch11-00-testing.html)
+ [テスト プログラミング言語Rust](https://doc.rust-jp.rs/the-rust-programming-language-ja/1.6/book/testing.html)
+ [What is rustdoc?](https://doc.rust-lang.org/rustdoc/what-is-rustdoc.html)
