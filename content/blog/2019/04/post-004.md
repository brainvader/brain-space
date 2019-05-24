+++
title = "Ethereum Concept"
description =  "Ethereum"
date = 2019-04-14T10:50:22Z

[taxonomies]
categories = ["programming"]
tags = ["Ethereum", "BlockChain"]
+++

# Ethereum

## Blockchain

　暗号通貨の基盤となる技術らしいです. 一定の量のデータ(ブロック)を繋げた(チェーン)データ構造をしているからこのように呼ばれるらしい. ブロックに入るデータ構造を工夫する見たい.

## Ethereum

　Ethereumはアルトコインの一つらしいです. Ethereumのブロックはコントラクタという情報が収められているらしい. 取引が実行されるごとにブロックが増えていく. この時にガスといってetherという通貨が消費される. これはショバ代というかこのシステムを運営していくための費用みたいなものである. これがEthereumにコミットするインセンティブとなる.

　なんらかの引き金に応じて自動的に取引が実行されるというのが売りらしい. こうすることで何が期待されるのか? いくつかの例があった.

+ [カーシェアリング](https://rhizomebrain.net/2017/07/14/smartcontract_ethereum/)
+ [音楽配信契約](https://gaiax-blockchain.com/smart-contract#Etheruem)

　つまり個人で自販機のように条件が一致したら取引が自動的に実行され代金としてetherが入ってくるということのようだ. 取引は個人間で行われるが, この場合問題になるのは信用だ. 現状は調査をする必要があるがEthereumを使うと, 条件をコントラクタとして定義しておくとそれが満たされた時点で取引が自動で行われる. 相手が誰に信用性がないとかそんなことは関係ない. カーシェアリングの例とは特に面白くて, エンジンがかかった時点で取引開始となる. たとえ誰に貸してもとりあえず車のレンタル料は取り損ねることはない(事故とかの場合は良くわからないが).

　市場で個人対個人での取引ができるようになるということらしい. 例えばAmazonで売買を行うと販売業者はショバ代を払う必要があるだろう. これを客に直接は取引できれば仲介料は不要となる. 他にも色んな事例が紹介されている[<sup>1</sup>](#ref-1)

　この取引の安全性を担保しているのが分散台帳とも呼ばれるブロック・チェーンの技術ということになる. 分散台帳は参加者是認が取引の記録を保持している(実際はちょっと違うらしいが). そのため改ざん

## ether

　Ethereumにおける通貨の単位です. 

## アルトコイン

　ビットコイン以外の暗号通貨をこう呼ぶそうです. 現在さまざまな種類(1500以上とも言われる)が発行されているようです. すでに美少女擬人化なんかもされているぐらいです[<sup>2</sup>](#ref-2).


## 賢い契約とは?

　一般にスマート・コントラクタと呼ばれます. スマート・コントラクタは1994年にNick Szaboによって提唱された提唱された概念らしいです. 応用例としては自動販売機があるそうです.

> 書面上で作成された契約のみをさすのではなく、「取引行動全般」をさします。つまりあらゆる契約行動をプログラム化し、自動的に実行しようとするものがスマートコントラクトです[<sup>3</sup>](#ref-3).

　暗号通貨を使うと確かに実現できそうです.

## Plasma

　現状では全然わかりません.

## Further Reading

　探せばいくらでもありますが, 1がおすすめ.

1. [Blockchain Basics: A Non-Technical Introduction in 25 Steps](http://www.blockchain-basics.com/)
2. [How does blockchain work in 7 steps — A clear and simple explanation.](https://blog.goodaudience.com/blockchain-for-beginners-what-is-blockchain-519db8c6677a)
3. [What is Blockchain Technology? A Step-by-Step Guide For Beginners](https://blockgeeks.com/guides/what-is-blockchain-technology/)
4. [初心者向けプラズマ勉強法まとめノートー 0から3週間で学んだこと](https://medium.com/cryptoeconomics-lab/%E5%88%9D%E5%BF%83%E8%80%85%E5%90%91%E3%81%91%E3%83%97%E3%83%A9%E3%82%BA%E3%83%9E%E5%8B%89%E5%BC%B7%E6%B3%95%E3%81%BE%E3%81%A8%E3%82%81%E3%83%8E%E3%83%BC%E3%83%88%E3%83%BC-0%E3%81%8B%E3%82%893%E9%80%B1%E9%96%93%E3%81%A7%E5%AD%A6%E3%82%93%E3%81%A0%E3%81%93%E3%81%A8-f9d68a4d617f)

## Reference
1. <a name="ref-1"></a>[スマートコントラクトとはなにか：イーサリアムが起こすイノベーション](https://rhizomebrain.net/2017/07/14/smartcontract_ethereum/)
2. <a name="ref-2"></a>[仮想通貨革命クリプトカレンシーガールズ](https://crypto-currency-girls.com/)
3. <a name="ref-3"></a>[【５分で速攻理解する】そもそも「スマートコントラクト」って何？](https://www.bibitpost.com/archives/775)
