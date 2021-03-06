+++
title = "非同期処理が分からない"
description = "プログラミング"
date = 2019-06-30T09:24:17Z

[taxonomies]
categories = ["programming"]
tags = ["low-level-programming"]
+++

## 非同期処理が分からなくなる原因

　非同期処理とは同期的ではない処理のことをだが, これが今ひとつ分からなかった. これは非同期処理にも実際にはいくつかの実現方法があることによる. 内容は[第1回　マルチスレッドはこんなときに使う (1/2)](https://www.atmarkit.co.jp/ait/articles/0503/12/news025.html#utm_term=share_sp)を主に参考にした.

### スレッド (thread of execution)

　ソフトウェア視点ではプログラムの処理を一本の糸と見立てたものをスレッドと言います. ハードではCPU(物理コア)を仮想化した論理コアのことをスレッドと言ったりします. Nコア/2Nスレッドなんて表記もあります. ややこしいのは物理コアと論理コアが両方とも1つであってもマルチ・スレッドはありえます. シングル・スレッドかマルチ・スレッドかの違いにハード(コア数)は直接関係ありません. とはいえシングル・コアとマルチ・コアでは引き出せる性能が全然違います.

### コンテクスト・スイッチ(実行文脈)

　OSがプロセスやスレッドを切り替える操作のことをこのように呼びます. シングル・コアでマルチ・スレッドを実現しようとした場合, 小刻みにコンテクスト・スイッチでスレッドを切り替えることでマルチ・タスクを実現しているようです.

### シングル・スレッド

　スレッドが1つしかありません. つまりプログラムは全て一本の線上で起きた出来事として処理されます. このモデルの問題点はスレッドを独占するプログラムが存在した場合, 他の処理は一切できなくなることです. JavaScriptなどはシングル・スレッドと言われます. しかし実際にはブラウザはマルチ・スレッドで処理を実行できるので, データのロードのような処理を行いながらもUI操作を並行して処理できます.

### マルチ・スレッド

　コア数によって振る舞いが違います. マルチ・コアなら別々のコアにスレッドを割り当てて並列に処理することが可能です. シングル・コアの場合にマルチ・スレッドを実現する場合は, コンテクスト・スイッチでスレッドを強制的に切り替えます. プログレス・バーを考えるとUIの入力を受け付けるスレッドとデータをロードするスレッドを入れ替えれば, データをロードしている間も定期的にUI用のスレッドが処理可能な状態になるためユーザーからは両者が全く同時に実行されているように感じるはずです. 総合的な処理時間を考えると, 別々に処理した方がコンテクスト・スイッチが不要な分高速です. 科学技術計算をさせたいなら計算中はコーヒーでも飲んでいる方がいいかもしれません.　しかしブラウジングしている時にロードが遅いからコーヒーを飲んで休憩しようという利用方法は現代では考えられないでしょう. 当然マルチ・コアなら両方の処理を別々のコアに割り当てると言うこともできます.

### 同時

　同時と言う言葉は誤解を招きやすい表現です. 特にコンピュータの世界の処理速度は人間が体感できないような速度なので同時と言うニュアンスに幅が生じます. マルチ・コアで並列に処理する場合も同時ですが, コンテクスト・スイッチで並行に処理している場合でも人間には同時と感じます.


### プロセス

　プロセスとはコンピュータが実行可能なアプリケーションの実体(インスタンス)のことです. 基本的にコア1つで実行できるプロセスは1つです.

### 並行処理

　いわゆるマルチ・タスクのことです. シングル・コアの時代でもブラウザ上で動画を見ながら計算を実行しつつ, プログラムや文章を書くと言うことが可能でした. これは OS が

### 並列処理

　並列処理はマルチ・コア CPUが一般化したことによって可能になった. 実際に処理を並列(別々のコア)で実行できる.

### 同期処理

　書いた順番に実行される処理のことです. スレッドに分割するようなことができないので, スケジューラで処理の順番を工夫する必要があります.

### 非同期処理

　非同期処理がややこしいのは上のような要素が絡み合うからです. 特にJavaScriptはブラウザというアプリの上でWebアプリを実行するためシングル・スレッドでも非同期処理が比較的簡単にできたりします. HTTPリクエストなどの非同期な処理はブラウザに任せて, JavaScriptはユーザー入力とかに集中するわけです. ブラウザのUIスレッドへのフックを書くためのまた近年はWeb Worker並列処理の仕組みも登場しているため, ますますややこしくなっています.

## まとめ

　非同期処理は思ったより奥深そうです. 言語によっても抽象化の仕方が違っていますし, 近年はマルチコアに対応したことから別スレッドで並列に処理するとかは割と当たり前のような状況もあるようです. 言語ごとに書き方を見ていく他ありませんが, 低レベルではスレッド作ったりコンテクスト・スイッチしたり色々しているようです. より深いレベルで理解するにはOSやハードのことを知らないといけないみたいです. JavaScriptでもWeb Workerの利用でマルチ・スレッドを並列で実行できる環境が整ったりしているようです. またシングル・スレッドだと言っても実際はブラウザは複数スレッドを使えたりとNode.jsではイベント駆動であるとかややこしいです. PythonもGIL(Global Interpreter Lock)とかの話があってめんどくさそうです(非同期とは直接関係ないですが, スレッドが並列かで使い勝手が違ってくるでしょう).

　これは他の構文にも言えることですが言語ごとに状況(抽象化の仕方)が違うので非同期処理をするときもドキュメントをちゃんと確認しようということです.

## Reference

- [プロセス、スレッドとは](http://u-kipedia.hateblo.jp/entry/2014/09/21/102823)
- [Is a light weight process attached to a kernel thread in Linux?](https://unix.stackexchange.com/questions/472300/is-a-light-weight-process-attached-to-a-kernel-thread-in-linux)
- [Operating systems](http://www.it.uu.se/education/course/homepage/os/vt18/)
- [図解】CPU のコアとスレッドとプロセスの違い・関係性、同時マルチスレッディング、コンテキストスイッチについて](https://milestone-of-se.nesuke.com/sv-basic/architecture/cpu/)
- [【図解】初心者にも分かるユーザー空間とカーネル空間, MMU/メモリ保護の仕組み](https://milestone-of-se.nesuke.com/sv-basic/architecture/user-space-kernel-space/)
- [非同期処理の基礎](https://www.slideshare.net/ufcpp/ss-34533225)
- [ややこしいのは物理コアと論理コアが両方とも1つでもマルチ・スレッドはありえます.](https://www.slideshare.net/ufcpp/asyncawait-114647813)
- [Scheduling In Go : Part I - OS Scheduler](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html)
- [コンテキストスイッチの仕組み](http://ossforum.jp/node/752)
- [Linux のしくみを学ぶ - プロセス管理とスケジューリング](https://syuu1228.github.io/process_management_and_process_schedule/process_management_and_process_schedule.html)
- [What are Threads?](https://www.studytonight.com/operating-system/multithreading)
- [Asynchronous programming. Blocking I/O and non-blocking I/O](https://luminousmen.com/post/asynchronous-programming-blocking-and-non-blocking)
- [OS の仕事はハードウェアをアプリから「隠す」こと？](https://ascii.jp/elem/000/000/629/629860/)

## Notes
{{ anchor(id=1) }} [Inside look at modern web browser](https://developers.google.com/web/updates/2018/09/inside-browser-part1)
