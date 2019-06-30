+++
title = "OpenGL (1) レンダリング・パイプライン"
description = "概説レンダリング・パイプライン"
date = 2019-06-21T21:47:01Z

[taxonomies]
categories = ["programming"]
tags = ["OpenGL"]
+++

## レンダリング・パイプライン

レンダリング・パイプラインは OpenGL が GPU 上で実行する処理の流れです.

> the sequence of steps that OpenGL takes when rendering objects {{ ref(id=6)}}

以下のようなステージに分類できます. {{ ref(id=6) }}

1. Vertex Specification
2. Vertex Shader
3. Tessellation
4. Geometry Shader
5. Vertex Post-Processing
6. Primitive Assembly
7. Rasterization
8. Fragment Shader
9. Per-Sample Operations

多くのステージはオプションです. 例えばフラグメント・シェーダもオプショナルです. 一方で頂点シェーダは必須です. またシェーダを利用できるプログラマブルなステージ(programmable states)とハードウェアが自動的に実行してくれるてステージ(fixed-function stages)に分けることもできます. ラスター化(Rasterization)は後者の１つです.

#### OpenGL Shading Language

OpenGL のシェーダを記述するための C 風の言語です.

### Vertex Specification

レンダリング・パイプラインで処理される頂点属性と実際の値(データ)を設定するステージです. 頂点データは突き詰めると全て GPU がアクセスできる VRAM 上に確保された連続したメモリ領域に保存されます. OpenGL オブジェクトの VAOs(Vertex Array Objects)や VBOs(Vertex Buffer Objects)などを準備します.

### Vertex Rendering

glDrawArrays のようなコマンドで頂点の描画を指示します. 頂点言いましたが実際は指定したプリミティブを構築するために OpenGL に要素を送る処理のようです. 実際のデータは Vertex Specification ステージでバッファー・オブジェクトに送られて VRAM 上で管理されていますので, この場合 VRAM から OpenGL レンダリング・システムへと送られることになります.

> This command constructs a sequence of geometric primitives by transferring elements first through first + count − 1 of each enabled array to the GL. mode specifies what kind of primitives are constructed, ...

プリミティブは mode というパラメータで指定できます. 以下のようなプリミティブが指定できます.

- GL_POINTS
- GL_LINE_STRIP
- GL_LINE_LOOP
- GL_LINES
- GL_LINE_STRIP_ADJACENCY
- GL_LINES_ADJACENCY
- GL_TRIANGLE_STRIP
- GL_TRIANGLE_FAN
- GL_TRIANGLES
- GL_TRIANGLE_STRIP_ADJACENCY
- GL_TRIANGLES_ADJACENCY
- GL_PATCHES

いろんなプリミティブがありますが例えば三角形でも GL_TRIANGLES は頂点列を独立した三角形, すなわち辺を共有しないものとして描画します. 一方 GL_TRIANGLE_STRIP や GL_TRIANGLE_FAN は辺の共有が生じます. このステージは頂点で指定した図形をどう言う単位で描画していくかを行います. 実際にここで絵が描かれるわけではありません.

### Vertex Processing

このステージは３つのサブ・ステージから構成されています.

- Vertex Shader
- Tessellation
- Geometry Sahder

ただし Tessellation はバージョン 4.0 から Geometry Shader はバージョン 3.2 以上が必要です.

#### Vertex Shader

必須のプロセスで頂点に関する処理を実装します. 例えば座標変換などはここで行います.

#### Tessellation

Tessellation シェーダが対象にするのは GL_PATCHES だけです. この GL_PATCHES とは以下のように説明されています.

> The GL_PATCHES primitive type ... is a primitive with a user-defined number of vertices{{ ref(id=10)}}

このステージはさらに以下の三つに分解されます.

- TCS (Tessellation Control Stage)
- Tessellator or Tessellation Primitive Generator (TPG)
- TES (Tessellation Evaluation Shader)

TCS はどれぐらい分割するのかとか隣接関係を指定します. TPG は固定関数ステージ(fixed-function stage)で, 実際のテッサレーションを実行しプリミティブを生成します. ここでは quads, isolines, そして triangles といったプリミティブに応じて実行されるアルゴリズムが決まっています. では TES は何をするのかというと, TPG は各パッチを正規化された座標に写してテッサレーションを実行します. そこから実際の座標へ戻す変換がを行うのが TES の役割です.

利用例としてはパラメトリック曲線/曲面を描画するケースや距離に応じて LODs(Level of Details)を変化させたい場合などです. 何れにしてもプリミティブが GL_PATCHES の場合に実行されるシェーダです. 出力は基本的に頂点シェーダと同じです.

#### Geometry Shader (GS)

新しい幾何形状を作るシェーダです. 法線の可視化などに利用されるように頂点の座標以外から幾何形状を生成したりできます. つまりポリゴンを表示する場合の付加的なデータの可視化に使えます. こうした頂点から構成される図形とは別に図形を構築できることから Layered rendering と呼んでいます.{{ ref(id=11) }} 頂点シェーダのみで表示される図形の上にレイヤーを重ねるように GS で別の図形を描いていくわけです.

使用例はすでに紹介しましたが, 法線などの補助的なデータの可視化です. またプリミティブを処理対象にしていることから三角形の dissolve とか explode と呼ばれる表現も GS で処理することができるようです. また Vertex Displacement などの処理にも使えるかもしれません. ただし処理速度は遅いとか・・・.

##### GS instancing

#### Vertex post-processing

##### Transform Feedback

頂点プロセシングではバッファ・オブジェクトからデータを読み出し計算を行います. この結果をバッファ・オブジェクトに書き戻すことができます.

##### Clipping

視錐体の外部のプリミティブを削除し, 境界に位置するものは分割して新しい頂点を挿入します. 処理後の座標空間をクリップ・スペースと呼んだりします.

#### Perspective Divide

同次座標系の第 4 要素である w コンポーネントで x, y, z を割る処理を実行します. クリッピングで視錐体外の頂点は排除されているので視錐体を正規化する処理になります. この処理で正規化デバイス座標が得られます.

#### Viewport Transform

最終的に画像が出力されるエリアを viewport と呼びます. ウィンドウと思っても良いです. そのためこの変換で得られた座標空間をウィンドウ・スペース(スクリーン・スペース)と呼んだりするようです. ウィンドウのサイズが固定の場合は良いですが普通はスケールに追従する方が良いでしょう.

### Primitive Assembly

頂点からプリミティブを構築する固定関数プロセスです. このプロセスでのプリミティブは以下の 3 つです.

- Lines
- Points
- Triangles

またトランスフォーム・フィードバックを使う場合で, レンダリングが必要ない場合はここでプロセスを止められます.

#### Face Culling

構築した三角形が表面を向いているものが基本的には描画の対象となります. そのためこのプロセスでそれらをレンダリングの対象から除外します.

#### Rasterization

各プリミティブを画面サイズに応じてラスター化します. 通常ラスター化はベクター形式をピクセルに変換する処理を言いますが, この場合生成されるのはフラグメント(Fragment)と呼ばれるデータです. フラグメントには以下のようなデータが含まれています.

- スクリーン・スペース上での位置
- サンプル適用範囲(sample coverage)
- そのた VS や GS から出力されたデータ

#### Fragment Processing

ラスター化で出力されたフラグメントをフラグメント・シェーダ(FS)を使って処理します. FS の出力は,

- 色情報
- 深度値
- ステンシル値

です.

#### Per-Sample Processing

このプロセスはいくつかのプロセス(テストと呼ばれる)から構成されています.

- Pixel Ownership Test(ピクセル所有権テスト)
- Scissor Test
- Stencil Test
- Depth Test
- Color Blending
- フレームバッファへの出力

#### そのた

- Early Primitive Assembly
- Early Depth Test
- Occlusion Query

## OpenGL Context

OpenGL には様々な設定が存在します. こうした設定の組み合わせで違う出力を生み出すことができます. それぞれの設定の状態をまとめてコンテクストと呼びます. 様々な状態をまとめたものが OpenGL の実体であることから OpenGL は巨大な状態マシンとも呼ばれます. コードよりの視点で言うと OpenGLContext という名前の大きな構造体を思い浮かべてもいいと思います. 例えば複数の OpenGL ベースのアプリを作った場合それらは別々の状態を持つだろうことは想像できると思います. 要するに OpenGL ベースのアプリが依存している情報をコンテクストと読んでいるわけです.

ただ OpenGL のコンテクスト作成は OpenGL の仕様には含まれていません. そのため専用のコマンドなどもありません. これは OpenGL を動かすプラットフォーム(主に OS)が提供する API で管理されます. つまり Windows と Linux, そして Mac ではコンテクストの生成方法は異なることになります. こうした知識は複雑な割にグラフィックスとは直接関係はありませんし, 開発用マシンなどの環境にも依存してしまいます. これでは不便なので通常は GLFW のようなプラットフォームに依存しないライブラリを使います. このブログでも GLFW を使うことを前提にします.

#### Context State

コンテクストが管理する状態のことをこのように呼ぶようです. 仕様には以下のような情報を持っているとされています.

- プログラマブル・シェーダを表現しているオブジェクト
- シェーダが利用/生成するデータ
- 非プログラマブルな管理情報

#### GLFW

GLFW はウィンドウやコンテクストの生成/管理, そして入力の処理など OS ごとの依存性を吸収してくれる便利なライブラリです.

- ウィンドウの生成
- ウィンドウの管理
- コンテクストの生成
- イベントの管理

似たようなライブラリに GLUT や SFML, そして Qt などがあります. OS にはそもそも基本的なウィンドウを描画や管理をする機能(ウィンドウ・システム)が存在します. そうしないと GUI のような必須の機能が提供できません. OpenGL が担うのはこうしたウィンドウに表示する画像を生成することです. すでに述べたように OpenGL はコンテクストの生成はできません. また出力先のウィンドウを OpenGL からは指定することもできません. GLFW は本来なら面倒なウィンドウやコンテクストの生成/管理をやってくれる便利なライブラリということになります. つまり細かいことは気にしなくても良いということです.

#### OpenGL はどうやってインストールするのか?

OpenGL は Open Graphics Library の略です. つまりライブラリなのです. プログラミングをしていても他のライブラリと変わりません. ただ OpenGL というライブラリをインストールした経験のある人はいないと思います. OpenGL の処理は GPU 上で行われるので, そのドライバーに付属しているので個別にインストールする必要はないのです. なお OpenGL は１つしかありませんがバージョンによって機能が違ってきたり互換性がない場合があります. ハードウェアに搭載されているグラフィック・アクセラレータ(グラフィック・カードなど)によっては対応していない場合もあります.

#### OpenGL API

OpenGL はグラフィック・ハードウェアの API(Application Program Interface)です. 実際にコードで実装するのはグラフィック・ハードウェアに対する送信される命令です. これはドライバーによって GPU 用の命令に変換されます.

> translate OpenGL API commands into GPU commands {{ ref(id=8)}}

#### サーバーとしての OpenGL

GPU 上の OpenGL は別個のシステムと見ることもできます. つまり CPU を中心とするシステム(クライアント)に対してサーバーとしての役割を果たすわけです. この視点からすると OpenGL でのレンダリング・システムは client-server モデルと考えられます. 実際に仕様である core-pforile ではネットワーク透過だと謳われています.

> In this sense, the GL is network transparent.

#### OpenGL オブジェクト

OpenGL はいくつかの状態をまとめてオブジェクトとして値を管理しています.

###

### ラスタライザ

フラグメントの説明をする前に, フラグメントを得るためのプロセスを紹介しましょう. レンダリング・パイプラインの１つで各三角形をピクセルに変換する処理です. ポイントは各三角形ごとにラスター化されることです. 素朴に２つの三角形が重なる場合はどうするかという疑問が湧いてきます. これは当然両方ラスター化されます. 深度テストと言う処理で, どちらが前をピクセル毎に決定します. では立方体の場合はどうでしょうか? 立方体の一部はカメラから隠れていますので最終的には表示されません. 基本的にはラスター化の１つ前のプロセスであるプリミティブ・アセンブリでフェイス・カリング(face culling)という処理がほどこされ, こうした裏面が視点方向を向いているものは削除されます. そのため表面を向いている三角形のみがラスター化の対象になります(何らかの理由で裏面を表示する場合は覗きます). またここで対象となる三角形はすでに 2 次元化されていることに注意してください. これは頂点シェーダで射影行列を使って行います. ラスタライザはこうして表面だけの 2 次元に射影された三角形をフラグメントに変換していきます.

### フラグメント

ラスタライズ・プロセスにより生成される値でフラグメント・シェーダの入力になる値らしいです. ピクセルとの違いはフラグメントは色情報以外も持っていることです. 例えはスクリーン・スペースでの座標やこの後の深度テスト(depth test)に用いる z 値です. 深度テストがこの後で行われることから分かるようにこの時点ではピクセルかされた三角形には重なりが生じている可能性があります. フラグメントの値を取得するには gl_FragCood という vec4 型のビルトイン変数を使います. x と y はスクリーン・スペースでの座標で z が深度値になります. w 値は 1/w というクリップ・スペースでの w 値の逆数になります. これによると

### 深度値の計算

フラグメントには深度値(depth value)が含まれている. これは gl_FragCood からアクセスできる.

## Reference

{{ anchor(id=1) }} [Fragment on OpenGL Wiki](https://www.khronos.org/opengl/wiki/Fragment)
{{ anchor(id=2) }} [OpenGL 4 Shaders](http://antongerdelan.net/opengl/shaders.html)
{{ anchor(id=3) }} [OpenGL 4 Shaders](http://antongerdelan.net/opengl/shaders.html)
{{ anchor(id=4) }} [Pipeline Overview](http://beyondthegeek.com/2019/04/08/opengl-rendering-pipeline/)
{{ anchor(id=5) }} [Rasterization: a Practical Implementation](https://www.scratchapixel.com/lessons/3d-basic-rendering/rasterization-practical-implementation/rasterization-stage)
{{ anchor(id=6) }} [Rendering Pipeline Overview](https://www.khronos.org/opengl/wiki/Rendering_Pipeline_Overview#Face_culling)
{{ anchor(id=7) }} [OpenGL as a State Machine](http://what-when-how.com/opengl-programming-guide/opengl-as-a-state-machine/)
{{ anchor(id=8)}} [Portal:OpenGL Concepts](https://www.khronos.org/opengl/wiki/Portal:OpenGL_Concepts)
{{ anchor(id=9) }} [Vertex Buffer Objects](http://antongerdelan.net/opengl/vertexbuffers.html)
{{ anchor(id=10) }} [Patches](https://www.khronos.org/opengl/wiki/Tessellation#Patches)
{{ anchor(id=11) }} [Geometry Shader](https://www.khronos.org/opengl/wiki/Geometry_Shader)
