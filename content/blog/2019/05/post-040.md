+++
title = "Learn OpenGL With Rust (3) Triangle"
description = "三角形の描画"
date = 2019-05-25T13:05:17Z

[taxonomies]
categories = ["programming"]
tags = ["OpenGL", "Computer Graphics", "Rust"]
+++

# OpenGLで三角形の描画

## Graphics Pipline

 以下のようなパイプラインを経て画像が出力されるらしい.

+ Vertex Shader
+ Shape Assembly
+ Geometry Shader
+ Rasterization
+ Fragment Shader
+ Tests and Blending

 シェーダは小さなプログラムでGPUで実行される. OpenGLの場合はGLSL(OpenGL Shading Language)が使われる.

### 頂点シェーダと頂点属性

　頂点シェーダは頂点に付随するデータを入力とします. 中でも座標データは重要です. 他にも頂点カラーや頂点法線など任意の情報を持たせることができます. GLSLでこのデータを保持する変数のようなものを頂点属性(Vertex Attribute)と呼んでいます.

### Primitive Assembly

　頂点組み立てと日本語でいうらしいです. 頂点をつないで実際の図形を作るステージです.

### Geometry Shader

　いわゆる細分化(Tessellation)が行われるステージです. オプションでOpenGLのサブセットのOpenGL ESやWebGLにはありません.

### Rasterization

　ベクターデータをラスター化します. この時点ではフラグメント{{ ref(id=1) }}と呼ばれるデータが出力され次のステージへと渡されるようです. またフラグメント・シェーダで処理される前にクリッピングが行われます.

#### Fragment

　フラグメントは以下のような情報が含まれているようです(よく分かりませんが・・・).

+ それ以前に行われた頂点処理に関する情報
+ ウィンドウ空間座標
+ ステンシル値
+ 表面・裏面
+ Multisample coverrage masks
+ プリミティブの識別子
+ ユーザー定義クリッピング距離
+ プリミティブ・レイヤー & ビューポート・インデックス

### Clipping

　カメラで見えている範囲(View Frastum)以外のフラグメントを捨てます.

### Fragment Shader

　フラグメントから最終的にピクセルを生成するステージです.  各種テスト(depth, stencil, そしてalphaテスト)やブレンディングも行います.

## RustとOpenGL

　パイプラインはGPU上で実行されるのでRustからは直接操作できません. ではどうやってOpenGLを利用したアプリケーションを作るのでしょうか? それはOpenGLレンダリング・システムへ依頼する形で行います. OpenGLはそのための公開APIとも言えます.

### OpenGLシステム

　コンピュータの描画処理は特別なハードウェアが担当しています. 一般にビデオカードと呼ばれるものがそれです. Intelなどは内臓GPUといってCPUの内部にGPUをを統合したIntel HD Graphicsシリーズを出していますが, いずれにしてもGPUという描画専用のハードウェア実装が存在していることに変わりはありません. 前者をdGPU(discrete GPU), 後者をiGPU(integrated GPU)とも呼んだりする. また最近ではeGPU(external GPU)という外付けのグラフィック・アクセラレータも発売されている.

　さて, 私たちが日頃OpenGLといって触るのはOpenGL APIです. コンピュータのCPUが計算処理の実体であるようにこうした描画処理を担うハードウェアが背後に存在しているわけです. このようなハードウェアも含んだシステム全体をOpenGLレンダリング・システムと呼んだりします. このシステムをコントロールするのがOpenGL APIと呼ばれるものです. そんなわけでOpenGLにせよその他の描画システムにせよ(DirectXとかMetalなど), CPU側から描画システム(GPU)をコントロールするクライアント-サーバー・システムだったりします.

　ところでOpenGLというライブライをインストールした記憶がある人はいないと思います. これはGPUのドライバーに付属しているからだったりします. このようにGPUを操作するOpenGLというのは従来のプログラムとはだいぶ違うものなのです. 以下少しだけOpenGLの概要を説明します.

### Context, State, And OpenGL Objects

　OpenGLコンテクストは何かは説明しにくいですが, OpenGLを利用するアプリケーションが依存する情報です. この情報に基づいてOpenGLレンダリング・システムは描画を行います. 個々のデータはステートと呼ばることから, OpenGLは巨大なステート・マシーンと言われたりもするようです. ステートはコンテクストに紐づけられたOpenGLオブジェクトによって管理されています. このオブジェクトはOpaque(不透明)なオブジェクトと呼ばれ, アプリケーション側からは見えません. 見えないというのは普通の構造体とかクラスのインスタンスのように生成してそのプロパティにアクセスしてメソッドを利用してと言ったことは自由にはできないということです. オブジェクトの生成とコンテクストへの結合, 削除, そして目的のステートへのデータの送信などをAPIを通じて依頼します. 後はGPU側でブラック・ボックス的に処理されます.

### 関数

　究極的にはステートに何かするのがOpenGL APIの目的ですからステーへの操作に応じて３つに分類できるようです.

+ ステートの設定
+ ステートの問い合わせ
+ ステートの描画

### シェーダ入門

　ジオメトリ・シェーダはオプションでした. 必須のシェーダとしては以下の２つです.

+ 頂点シェーダ (gl::VERTEX_SHADER)
+ フラグメント・シェーダ (gl::FRAGMENT_SHADER)

　その後このシェーダをコンパイルしてい, シェーダ・プログラム・オブジェクトを生成します. おおよその流れは以下のようになります.

1. シェーダの読み込み
2. シェーダのコンパイル
3. シェーダのリンク
4. プログラム・オブジェクトへのリンク

　シェーダではプログラマブルなパイプラインで実行される処理を定義するだけです. 後はOpenGLレンダリング・システムが勝手に呼び出してくれます.

## 三角形の頂点データの定義

　三角形の幾何データは単なるRustの配列です.

```rust
let vertices: [f32; 9] = [
    -0.5, -0.5, 0.0,
     0.5, -0.5, 0.0,
     0.0,  0.5, 0.0
];
```

　ポイントはデータ型は単精度の浮動小数点なところです. これはハード側の制約で一般的に単精度の浮動小数点しか扱えません. 当然高い精度が必要な分野(金融や科学技術計算)では倍精度の浮動小数点演算も必要ですが, ゲームなどのグラフィックスではそこまでの精度は要求されず, むしろ滑らかに動く方が重要視されています. 後価格も安く済ませられます.

### Buffer Object

　OpenGLの頂点座標などは非構造化された連続したメモリ領域に保存されます. このメモリ領域を管理するのがバッファ・オブジェクトの役割で, いくつかあるOpenGLオブジェクトの１つらしいです. CPUから頂点データなどを送信するとバッファ・オブジェクトの管理下に置かれるわけです. この方式は一度GPU側に送ってしまうとデータの変更がない場合はそれ以上送信する必要は無くなり効率的です. OpenGLの古いバージョンではそうではなかったようですが, よく分かりません. なお頂点に関するデータを管理する場合に**VBO(Vertex Buffer Object)**と呼ぶようです.

#### バッファ・オブジェクトの生成

　genはgenerateの略ですが, バッファ・オブジェクトは直接操作できません. つまりバッファ・オブジェクトの本体はGPU側で勝手に生成されます. 基本的にOpenGLオブジェクトは不透明(Opaqu)なオブジェクトと言われ直接操作することはできないようです. CPU側にはバッファ・オブジェクトの識別子だけが返されます. 以下のようにVBOと名づけられたりしますが, ポインターが実際にはアドレスだけをやりとりしているのと同様にidだけが受け渡しされます.

```rust
let mut vbo_id = 0; 
gl::GenBuffers(1, &mut vbo_id);
```

#### バインディング・ターゲット

　頂点バッファ以外にもバッファ・オブジェクトは存在します. この場合どのバッファ・オブジェクト用のデータなのかを指定する必要があります.

+ gl::ARRAY_BUFFER
+ gl::ELEMENT_ARRAY
+ gl::TEXTURE_BUFFER

　頂点バッファの場合はGL_ARRAY_BUFFERを指定します. もう少し詳しくいうとOpenGLコンテクストは複数のオブジェクトから構成されています. でかい構造体を想像すればいいでしょうか? このコンテクストのどの位置にオブジェクトを結合するのかをバインディング・ターゲットで指定します. なおバインディング・ターゲットはバインディング・ポイントとも呼ばれます.

#### 頂点座標データの転送

　VBOを直接指定するのではなバインディング・ターゲットを指定します. 非常に迂遠なやり方ですが, 何か意味があるのでしょうか? ドキュメントには

> When you're just creating, filling the buffer object with data, or both, the target you use doesn't technically matter.

　とにかくバインディング・ターゲットを介して, データの受け取り側(VBO)とデータの転送元が無事つながりました.

```rust
gl::BufferData(gl::ARRAY_BUFFER,
              (vertices.len() * mem::size_of::<GLfloat>()) as GLsizeiptr,
              &vertices[0] as *const f32 as *const c_void,
              gl::STATIC_DRAW);
```

#### バッファ・オブジェクトのまとめ

　データの"受け皿"としてバッファ・オブジェクトを作成します. 特に頂点データを扱う場合を頂点バッファ・オブジェクト(VBO)と呼ぶのでした. VBOを直接指定してデータを転送してもいいように思いますが, バインディング・ポイントを通じて行うのがOpenGLのやり方のようです. 対してシェーダ・オブジェクトは直接指定します.


#### 頂点属性

　各頂点ごとにデータが存在します. これは頂点シェーダによる処理の対象となります. これを頂点属性(Vertex Attribute)と呼びます. main関数の外側で定義します. よって頂点シェーダにはこのための変数が存在します. 頂点シェーダのインプットなのでinというキーワードを付けて表します. またlayoutで頂点属性の位置(location)を指定することもできます.

#### データのフォーマット

　頂点の座標データはすでにBufferDataコマンドで転送しています. しかしバッファ・オブジェクト(正確にはVBOが管理するメモリ)は構造を持ちませんので, シェーダがどのようにデータを解釈するかという情報はGPU側にはありません. 別途渡してやる必要があります.

```rust
gl::VertexAttribPointer(0, 3, gl::FLOAT, gl::FALSE, 3 * mem::size_of::<GLfloat>() as GLsizei, ptr::null());
```

+ index (location)
+ size per vertex
+ data type
+ normalized
+ stride
+ pointer

　さらに頂点属性配列を有効にする必要もあります.

```rust
gl::EnableVertexAttribArray(0);
```

　バンディング・ポイントの切り替えや頂点属性配列の有効・無効の切り替えは一見すると無駄な作業に見えますが描画に必要なデータ事前にビデオ・カード上にコピーしておいて, 必要な時に切り替えて使うと考えると効率的と言えそうです.

#### VAO (Vertex Array Object)

　今回は頂点属性は座標だけですが, 頂点カラーや頂点法線など頂点属性のデータの種類に制限はありません{{ ref(id=3) }}. また表示するオブジェクトの数も1個や2個ということはありません. ゲームなどでは非常に多くのオブジェクトを表示する必要があります. こうした設定の組み合わせを保存する方法としてVAOが用意されています.

```rust
let mut VAO = 0;
gl::GenVertexArrays(1, &mut VAO);
// bind VBO and define vertex attribute array (1)
gl::BindVertexArray(0);
```

　(1)でバッファ・オブジェクトの結合や頂点属性の定義などを行うと, 頂点属性の設定をVAOという塊で保存できるそうです. 表示されるのは3Dオブジェクトなので, VBOよりも粒度が大きいです. 対応するのはVAOと考えるとより自然な形で描画処理を記述できそうです.


#### 描画コマンド

　省略

#### EBOs (Element Buffer Objects)

　頂点の座標を指定する方法は四角形の場合に無駄が生じます. 頂点が重複するのです. OpenGLは三角形を組み合わせて図形を描画するので四角形の場合２つの三角形を組み合わせます. そうすると２つの三角形の辺が重なる部分が出てきます. 本来は４つのところ６つの頂点を描画する必要が出てきます. その時使うのがEBOです. インデックス・バッファとも呼ばれます. これもVAOでまとめて設定できます.

#### ワイアーフレーム・モード

　通常は塗りつぶしですが, 線で表示することもできます.

```rust
gl::PolygonMode(gl::FRONT_AND_BACK, gl::FILL);
gl::PolygonMode(gl::FRONT_AND_BACK, gl::LINE);
```

## Reference
{{ anchor(id=1) }} [Fragment](https://www.khronos.org/opengl/wiki/Fragment)
{{ anchor(id=2) }} [Purpose of binding points in OpenGL?](https://stackoverflow.com/questions/39921879/purpose-of-binding-points-in-opengl)
{{ anchor(id=3) }} 頂点属性の数にはハードウェアごとに制限があります. GL_MAX_VERTEX_ATTRIBSという定数で定義されています.
