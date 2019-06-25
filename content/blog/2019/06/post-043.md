+++
title = "OpenGL (1) レンダリング・パイプライン"
description = "概説レンダリング・パイプライン"
date = 2019-06-21T21:47:01Z

[taxonomies]
categories = ["programming"]
tags = ["OpenGL"]
+++

## レンダリング・パイプライン

　レンダリング・パイプラインはOpenGLがGPU上で実行する処理の流れです.

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

　OpenGLのシェーダを記述するためのC風の言語です.

### Vertex Specification

　レンダリング・パイプラインで処理される頂点属性と実際の値(データ)を設定するステージです. 頂点データは突き詰めると全てGPUがアクセスできるVRAM上に確保された連続したメモリ領域に保存されます. OpenGLオブジェクトのVAOs(Vertex Array Objects)やVBOs(Vertex Buffer Objects)などを準備します.

### Vertex Rendering

　glDrawArraysのようなコマンドで頂点の描画を指示します. 頂点言いましたが実際は指定したプリミティブを構築するためにOpenGLに要素を送る処理のようです. 実際のデータはVertex Specificationステージでバッファー・オブジェクトに送られてVRAM上で管理されていますので, この場合VRAMからOpenGLレンダリング・システムへと送られることになります.

> This command constructs a sequence of geometric primitives by transferring elements first through first + count − 1 of each enabled array to the GL. mode specifies what kind of primitives are constructed, ...

　プリミティブはmodeというパラメータで指定できます. 以下のようなプリミティブが指定できます.

+ GL_POINTS
+ GL_LINE_STRIP
+ GL_LINE_LOOP
+ GL_LINES
+ GL_LINE_STRIP_ADJACENCY
+ GL_LINES_ADJACENCY
+ GL_TRIANGLE_STRIP
+ GL_TRIANGLE_FAN
+ GL_TRIANGLES
+ GL_TRIANGLE_STRIP_ADJACENCY
+ GL_TRIANGLES_ADJACENCY
+ GL_PATCHES

　いろんなプリミティブがありますが例えば三角形でもGL_TRIANGLESは頂点列を独立した三角形, すなわち辺を共有しないものとして描画します. 一方GL_TRIANGLE_STRIPやGL_TRIANGLE_FANは辺の共有が生じます. このステージは頂点で指定した図形をどう言う単位で描画していくかを行います. 実際にここで絵が描かれるわけではありません.

### Vertex Processing

　このステージは３つのサブ・ステージから構成されています.

+ Vertex Shader
+ Tessellation
+ Geometry Sahder

　ただしTessellationはバージョン4.0からGeometry Shaderはバージョン3.2以上が必要です.

#### Vertex Shader

　必須のプロセスで頂点に関する処理を実装します. 例えば座標変換などはここで行います.

#### Tessellation

　Tessellationシェーダが対象にするのはGL_PATCHESだけです. このGL_PATCHESとは以下のように説明されています.

> The GL_PATCHES primitive type ... is a primitive with a user-defined number of vertices{{ ref(id=10)}}

　このステージはさらに以下の三つに分解されます.

+ TCS (Tessellation Control Stage)
+ Tessellator or Tessellation Primitive Generator (TPG)
+ TES (Tessellation Evaluation Shader)

　TCSはどれぐらい分割するのかとか隣接関係を指定します. TPGは固定関数ステージ(fixed-function stage)で, 実際のテッサレーションを実行しプリミティブを生成します. ここではquads, isolines, そしてtrianglesといったプリミティブに応じて実行されるアルゴリズムが決まっています. ではTESは何をするのかというと, TPGは各パッチを正規化された座標に写してテッサレーションを実行します. そこから実際の座標へ戻す変換がを行うのがTESの役割です.

　利用例としてはパラメトリック曲線/曲面を描画するケースや距離に応じてLODs(Level of Details)を変化させたい場合などです. 何れにしてもプリミティブがGL_PATCHESの場合に実行されるシェーダです. 出力は基本的に頂点シェーダと同じです.

#### Geometry Shader (GS)

　新しい幾何形状を作るシェーダです. 法線の可視化などに利用されるように頂点の座標以外から幾何形状を生成したりできます. つまりポリゴンを表示する場合の付加的なデータの可視化に使えます. こうした頂点から構成される図形とは別に図形を構築できることからLayered renderingと呼んでいます.{{ ref(id=11) }} 頂点シェーダのみで表示される図形の上にレイヤーを重ねるようにGSで別の図形を描いていくわけです.

　使用例はすでに紹介しましたが, 法線などの補助的なデータの可視化です. またプリミティブを処理対象にしていることから三角形のdissolveとかexplodeと呼ばれる表現もGSで処理することができるようです. またVertex Displacementなどの処理にも使えるかもしれません. ただし処理速度は遅いとか・・・. 


##### GS instancing

#### Vertex post-processing

##### Transform Feedback

　頂点プロセシングではバッファ・オブジェクトからデータを読み出し計算を行います. この結果をバッファ・オブジェクトに書き戻すことができます.

##### Clipping

　視錐体の外部のプリミティブを削除し, 境界に位置するものは分割して新しい頂点を挿入します. 処理後の座標空間をクリップ・スペースと呼んだりします.

#### Perspective Divide

　同次座標系の第4要素であるwコンポーネントでx, y, zを割る処理を実行します. クリッピングで視錐体外の頂点は排除されているので視錐体を正規化する処理になります. この処理で正規化デバイス座標が得られます.

#### Viewport Transform

　最終的に画像が出力されるエリアをviewportと呼びます. ウィンドウと思っても良いです. そのためこの変換で得られた座標空間をウィンドウ・スペース(スクリーン・スペース)と呼んだりするようです. ウィンドウのサイズが固定の場合は良いですが普通はスケールに追従する方が良いでしょう.

### Primitive Assembly

　頂点からプリミティブを構築する固定関数プロセスです. このプロセスでのプリミティブは以下の3つです.

+ Lines
+ Points
+ Triangles

　またトランスフォーム・フィードバックを使う場合で, レンダリングが必要ない場合はここでプロセスを止められます.

#### Face Culling

　構築した三角形が表面を向いているものが基本的には描画の対象となります. そのためこのプロセスでそれらをレンダリングの対象から除外します.

#### Rasterization

　各プリミティブを画面サイズに応じてラスター化します. 通常ラスター化はベクター形式をピクセルに変換する処理を言いますが, この場合生成されるのはフラグメント(Fragment)と呼ばれるデータです. フラグメントには以下のようなデータが含まれています.

+ スクリーン・スペース上での位置
+ サンプル適用範囲(sample coverage)
+ そのたVSやGSから出力されたデータ

#### Fragment Processing

　ラスター化で出力されたフラグメントをフラグメント・シェーダ(FS)を使って処理します. FSの出力は, 

+ 色情報
+ 深度値
+ ステンシル値

　です.

#### Per-Sample Processing

　このプロセスはいくつかのプロセス(テストと呼ばれる)から構成されています.

+ Pixel Ownership Test(ピクセル所有権テスト)
+ Scissor Test
+ Stencil Test
+ Depth Test
+ Color Blending
+ フレームバッファへの出力

#### そのた

+ Early Primitive Assembly
+ Early Depth Test
+ Occlusion Query

## OpenGL Context

　OpenGLには様々な設定が存在します. こうした設定の組み合わせで違う出力を生み出すことができます. それぞれの設定の状態をまとめてコンテクストと呼びます. 様々な状態をまとめたものがOpenGLの実体であることからOpenGLは巨大な状態マシンとも呼ばれます. コードよりの視点で言うとOpenGLContextという名前の大きな構造体を思い浮かべてもいいと思います. 例えば複数のOpenGLベースのアプリを作った場合それらは別々の状態を持つだろうことは想像できると思います. 要するにOpenGLベースのアプリが依存している情報をコンテクストと読んでいるわけです.

　ただOpenGLのコンテクスト作成はOpenGLの仕様には含まれていません. そのため専用のコマンドなどもありません. これはOpenGLを動かすプラットフォーム(主にOS)が提供するAPIで管理されます. つまりWindowsとLinux, そしてMacではコンテクストの生成方法は異なることになります. こうした知識は複雑な割にグラフィックスとは直接関係はありませんし, 開発用マシンなどの環境にも依存してしまいます. これでは不便なので通常はGLFWのようなプラットフォームに依存しないライブラリを使います. このブログでもGLFWを使うことを前提にします.

#### Context State

　コンテクストが管理する状態のことをこのように呼ぶようです. 仕様には以下のような情報を持っているとされています.

+ プログラマブル・シェーダを表現しているオブジェクト
+ シェーダが利用/生成するデータ
+ 非プログラマブルな管理情報

#### GLFW

　GLFWはウィンドウやコンテクストの生成/管理, そして入力の処理などOSごとの依存性を吸収してくれる便利なライブラリです.

+ ウィンドウの生成
+ ウィンドウの管理
+ コンテクストの生成
+ イベントの管理

　似たようなライブラリにGLUTやSFML, そしてQtなどがあります. OSにはそもそも基本的なウィンドウを描画や管理をする機能(ウィンドウ・システム)が存在します. そうしないとGUIのような必須の機能が提供できません. OpenGLが担うのはこうしたウィンドウに表示する画像を生成することです. すでに述べたようにOpenGLはコンテクストの生成はできません. また出力先のウィンドウをOpenGLからは指定することもできません. GLFWは本来なら面倒なウィンドウやコンテクストの生成/管理をやってくれる便利なライブラリということになります. つまり細かいことは気にしなくても良いということです.


#### OpenGLはどうやってインストールするのか?

　OpenGLはOpen Graphics Libraryの略です. つまりライブラリなのです. プログラミングをしていても他のライブラリと変わりません. ただOpenGLというライブラリをインストールした経験のある人はいないと思います. OpenGLの処理はGPU上で行われるので, そのドライバーに付属しているので個別にインストールする必要はないのです. なおOpenGLは１つしかありませんがバージョンによって機能が違ってきたり互換性がない場合があります. ハードウェアに搭載されているグラフィック・アクセラレータ(グラフィック・カードなど)によっては対応していない場合もあります.

#### OpenGL API

　OpenGLはグラフィック・ハードウェアのAPI(Application Program Interface)です. 実際にコードで実装するのはグラフィック・ハードウェアに対する送信される命令です. これはドライバーによってGPU用の命令に変換されます.

> translate OpenGL API commands into GPU commands {{ ref(id=8)}}

#### サーバーとしてのOpenGL

　GPU上のOpenGLは別個のシステムと見ることもできます. つまりCPUを中心とするシステム(クライアント)に対してサーバーとしての役割を果たすわけです. この視点からするとOpenGLでのレンダリング・システムはclient-serverモデルと考えられます. 実際に仕様であるcore-pforileではネットワーク透過だと謳われています.

> In this sense, the GL is network transparent.

#### OpenGLオブジェクト

　OpenGLはいくつかの状態をまとめてオブジェクトとして値を管理しています.

###

### ラスタライザ

　フラグメントの説明をする前に, フラグメントを得るためのプロセスを紹介しましょう. レンダリング・パイプラインの１つで各三角形をピクセルに変換する処理です. ポイントは各三角形ごとにラスター化されることです. 素朴に２つの三角形が重なる場合はどうするかという疑問が湧いてきます. これは当然両方ラスター化されます. 深度テストと言う処理で, どちらが前をピクセル毎に決定します. では立方体の場合はどうでしょうか? 立方体の一部はカメラから隠れていますので最終的には表示されません. 基本的にはラスター化の１つ前のプロセスであるプリミティブ・アセンブリでフェイス・カリング(face culling)という処理がほどこされ, こうした裏面が視点方向を向いているものは削除されます. そのため表面を向いている三角形のみがラスター化の対象になります(何らかの理由で裏面を表示する場合は覗きます). またここで対象となる三角形はすでに2次元化されていることに注意してください. これは頂点シェーダで射影行列を使って行います. ラスタライザはこうして表面だけの2次元に射影された三角形をフラグメントに変換していきます. 


### フラグメント

　ラスタライズ・プロセスにより生成される値でフラグメント・シェーダの入力になる値らしいです. ピクセルとの違いはフラグメントは色情報以外も持っていることです. 例えはスクリーン・スペースでの座標やこの後の深度テスト(depth test)に用いるz値です. 深度テストがこの後で行われることから分かるようにこの時点ではピクセルかされた三角形には重なりが生じている可能性があります. フラグメントの値を取得するにはgl_FragCoodというvec4型のビルトイン変数を使います. xとyはスクリーン・スペースでの座標でzが深度値になります. w値は1/wというクリップ・スペースでのw値の逆数になります. これによると

### 深度値の計算

　フラグメントには深度値(depth value)が含まれている. これはgl_FragCoodからアクセスできる.

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
