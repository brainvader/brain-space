+++
title = 'Unity Interactive Tutorialメモ (1)'
description =  'Unity Interactive Tutorialの備忘録です'
date = 2019-04-20T18:16:19Z

[taxonomies]
categories = ["programming"]
tags = ["Unity"]
+++

# Unity Interactive Tutorial

## TOC
+ [Play & Edit Mode](#sec-1)
+ [Game Object & Components](#sec-2)
+ [Tweaking Components](#sec-3)
+ [Prefab Power](#sec-4)

## <a href="sec-1">Play & Edit Mode</a>

### EditモードとPlayモード

　UnityはEditモードとPlayモードがあります. Editモードで編集を行い, Playモードではゲームのレビューができます.
　
### PlayモードとPlayボタン

　Unityエディタの中央上部に位置する三角形(右方向を指す矢印)をPlayボタンと呼び, これを押すとPlayモードが始まります. Editモードでは黒く塗りつぶされており, Playモード中は青く表示されます

## <a href="sec-2">Game Object & Components</a>

### Game Objects

　シーン状に表示されている要素(目で見える部分)は全てGame Objectです.

### Components

　Game Objectは通常何もしません. コンポーネントを追加して動作や見た目を定義することができます. 例えばRigidBodyコンポーネントを追加するとオブジェクトはゲーム世界で物理現象の影響を受けるようにできます.

### Inspector Window

　コンポーネントはパラメータをまとめたもので, Inspector画面で情報を確認したり編集できます.

### Hierarchy Window

　階層画面ではオブジェクトがツリー状に表示されます.

## <a href="sec-3">Tweaking Components</a>

### NavMeshAgentコンポーネント

　NavMeshAgentコンポーネントはUnity標準のAIシステムを利用できるコンポーネントです. Speedというパラメータを変更するとMPCキャラクタの動きを制御できます. このようにしてゲームの難易度などを調整できます.

### カスタム・コンポーネント

　当然自作のコンポーネントを追加することもできます.

## <a href="sec-4">Prefab Power</a>

### プロジェクト画面

　スクリプトやモデル(メッシュやスケルタル・メッシュ), 音声ファイルなどのアセットを管理する画面です.

### Prefab

　ゲーム・オブジェクトの雛形です. 一度作っておくと後はコピーによって同じオブジェクトを複製できます. プロジェクト画面から階層画面やゲーム・シーンにドラッグ・ドロップするだけでコピーされたインスタンスが生成できます.

## Reference
+ [Interactive Tutorials](https://unity3d.com/learn/tutorials/s/interactive-tutorials)

