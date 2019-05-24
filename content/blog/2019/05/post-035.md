+++
title= 'Rust入門 (4) CowとDerefについて'
description= 'CowとDerefについてのメモ'
date = 2019-05-14T22:33:19Z

[taxonomies]
categories = ["programming"]
tags= ["Rust"]
+++

# CowとDeref

## Cowとは?

　CowはClone-on-writeの略らしいです.[<sup>1</sup>](#ref-1)

> The type Cow is a smart pointer providing clone-on-write functionality: it can enclose and provide immutable access to borrowed data, and clone the data lazily when mutation or ownership is required.


　Cowはスマート・ポインタでーで, 書き込み時の複製機能を提供します: 借用されたデータを包み込むか不変なアクセスを提供します. 更に(データの)変更や所有権の取得が必要な時はその時になってからデータを複製できます.

> 'Cow' ... is a type that allows you to reuse data if it is not modified. [<sup>2</sup>](#ref-2)

　Cowは変更されないデータを再利用することができる型です.


> Cow is an abbreviation for Clone-on-write (usually Copy on Write, but copy has a different meaning in Rust), a cool idea that means we can use both an owned instance of something or a borrowed instance using the same code.[<sup>3</sup>](#ref-3)

　

## Derefとは?

## Reference
1. <a name="ref-1"></a> [std::borrow::Cow](https://doc.rust-lang.org/std/borrow/enum.Cow.html)
2. <a name="ref-2"></a> [The Secret Life of Cows](https://deterministic.space/secret-life-of-cows.html)
3. <a name="ref-3"></a> [Holy std::borrow::Cow!](https://llogiq.github.io/2015/07/09/cow.html)
