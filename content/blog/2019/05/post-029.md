+++
title= 'Unity入門 (3) Lighting'
description= 'Unityのライティング関係資料のメモ'
date = 2019-05-03T22:19:41Z

[taxonomies]
categories = ["programming"]
tags= ["Unity"]
+++

# ライティングの基礎

## 動画

　とりあえずこれを見る.

<iframe width="560" height="315" src="https://www.youtube.com/embed/eGu9_8HS2uI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

　プロシージャル・スカイボックスの解説動画です. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/UIbkfVWYRWQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

　後半のPost Processing StackはUnityに統合されたので導入方法が変わっているようです. ちなみに現在は

Window -> Package Manager

　からPost Processingは導入するようです. 詳しくは以下を参照してください.

<iframe width="560" height="315" src="https://www.youtube.com/embed/5cjhNjtkpVE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## コーネル・ボックス

 コーネル・ボックスはライティングのために作られる特別な空間らしいです. ライティングってどういう状況で試したらいいのか分からないのでこういうのを作ってやってみるのもいいかもしれません.[<sup>5</sup>](#ref-5) [<sup>6</sup>](#ref-6)

> コーネルボックス（英語：cornell box）とは、3DCGのレンダリングの品質を見るために考案された環境である。[<sup>7</sup>](#ref-7)

## Unityのライティング

　光にはおおよそ2種類あります.

+ Direct Light (直接光)
+ Indirect Light (間接光)

　直接光は比較的簡単に扱えます. 一方, 間接光は計算が複雑になります. 光源から表面を到達した光が反射されて空間上を跳ね返りながらカメラに入ってきます. このパスを正確に計算すると物理的に正しい絵ができるようです. 

　この差は穴から入る光の効果などを表現するに顕著に現れます. 以下はライトなし(左), 直接光のみ(中央), そして間接光あり(右)の例です.

![Comparison of lightin effect](https://unity3d.com/sites/default/files/lightingintroduction.png)

## リアルタイム化事前計算か?

　以下のライトはリアルタイムで更新される直接光に寄与するそうです. これらは動的なオブジェクトにもっぱら使うようです.

+ Directional Light
+ Spot Light
+ Point Light

![スポット・ライトの例](https://docs.unity3d.com/uploads/Main/RealtimeSpotlight.png)

　間接光は光の反射などでできるる影や光を表現できます.

## GI (Global Illumination)

　Unityのマニュアルにはシステムと書いていますが, 間接光も取り入れたモデルを元に描画を行うシステムということでしょう.

> Global Illumination (GI) is a system that models how light is bounced off of surfaces onto other surfaces (indirect light) rather than being limited to just the light that hits a surface directly from a light source (direct light).

### 事前計算とライト・プローブ

　普通にGIを計算するととんでもない時間がかかり, ゲームのフレームレートが落ちるなどの不都合が生じます. そのため静的なオブジェクトについては事前に光の影響を計算しておきます. 

+ Baked GI (ベイクGI)
+ Precomputed Realtime GI (リアルタイムGI)

　Precomputed Realtime GIは現在ではRealtime Global Illuminationと呼ばれているようです.

### リアルタイムGI

　Lighting Precomputeというプロセスで, 光の跳ね返りを計算しておきます. 一種のルックアップ・テーブルで, ランタイム時に参照されます. ベイクGIが単純にテクスチャにライトマップとして情報を焼き込むのとは違いリアルタイムでGIを更新できます. この情報はLighting Data Assetに保存されるらしいです.

 応用例としてはA time of dayシステムがあります.

 ![A time of day system](https://docs.unity3d.com/uploads/GlobalIllumination/timeofdaycycle.gif)

### ベイクGI

　こちらは古典的な方法で, 単純に焼き込みを行います. テクスチャなので光との相互作用などは全く働かないです. ある時点で作り出された光と影を色として色としてテクスチャに保存しているだけです.

![Lightmapped Scene and Lightmap](https://docs.unity3d.com/uploads/GlobalIllumination/Lightmap.png)

　直接光からの影響は受けるものの, 基本的にはこれがこのまま表示されて途中で変更することはできません. ただし性能が低いデバイスでも使えるようです.

### Enlighten

　Unityで使われるGI事前計算のためのバックエンドの名称です. これを最適に使ええるかで, 事前計算の効率に影響するようです.

## Lighting Setting

### 環境に関する設定

　Lightinの設定はWindow -> Rendering -> Lighting Settingsにあります. 資料によって違う場合が多いので最新のマニュアルを読むのが良さそうです(それでも更新が間に合わない場合はあるっぽいですが).

+ スカイボックスと太陽
+ 環境光 (Environmental Lighting)
+ 環境映り込み (Environmental Reflection)

　環境映り込みは高い反射率が設定されたマテリアルが適用されたオブジェクへ周囲の環境が映り込む現象である. 極端な例でいうと鏡のような表面をしたオブジェクトということである.

### GIの設定

　Realtime 

![Settings for GI](../../img/Unity_007.png)

### その他

　FogとかProgress CPUとか.

### リアルタイムGIとベイクGIの併用

　リアルタイムGIとベイクGIを両方使うMixedというのはとても思いです. 4コアフルでした. 

## 光源

+ Point Light
+ Spot Light
+ Directional Light
+ Area light
+ Emissive materials

### Directional Light

　Unityのプロジェクトを作るとデフォルトでダイレクト・ライトが１つシーンに追加されています. これが実は太陽として設定されています. 

![Default Sun Source](../../img/Unity_008.png)  

またアンビエント・ライトの元もスカイボックスなので, デフォルトのライトの色を買えると環境光の色も変わります.

![Ambient Source](../../img/Unity_009.png)  

### Area Light

　柔らかい光が出せるライトです. 逆二乗則で減衰します. 注意点は計算量が多いためか, ライトマップを使ったベイクのみでの利用となるようです(リアルタイムGIは使えない).

> Since the lighting calculation is quite processor-intensive, area lights are not available at runtime and can only be baked into lightmaps.

### Emissive Material

　エリア・ライトと同じような雰囲気の光を出せます. 逆二乗則に従う減衰などエリアライトと似た特性がありますがリアルタイムGIでも利用できます. ネオンなどに利用できます. 普通のジオメトリにマテリアルを設定して, Emission設定から光度を調整します.

> Emission will only be received by objects marked as ‘Static’ or “Lightmap Static’ from the Inspector.

### Ambient Lighting

 どこからともなくやってくる光をDiffuse LightingもしくはAmbient lightと呼ぶそうです. 色などを変更するとどこに影響しているかすぐに分かると思いますが, シーン全体の雰囲気を醸し出すのに使えそうです.

### ライティング・インスペクタ

　いじれば分かるのでパス.

### ライトの使い方

　ライトを追加してもシーン・ビューで変化がない場合は以下のアイコンから有効にできます.

![Lighting Effect In Scene View](../../img/Unity_10.png)

### Cookie(cucoloris)

　木々の影とか窓の格子の影をテクスチャで再現する.

+ ジャングル
+ barred window(格子付きの窓)

　などで使われる.

![Cookie In Action](https://docs.unity3d.com/uploads/Main/CookieExample.png)

 作り方は必要になったらやろう.

## 影

### シャドウ・マップ

　影は光で照らされていない領域です.

![How to create the shadow](https://docs.unity3d.com/uploads/Main/ShadowMapIntro.svg)

　ライトの視点からは見えない領域と言い換えることもできるようです.

![Camera View](https://docs.unity3d.com/uploads/Main/ShadowLightsEyeView.svg)

　こうした重なりの情報は深度バッファと呼ばれ, ジオメトリの重複を判断しています.

### シャドウ・アクネとバイアス

　まずはシャドウ・アクネがない場合は以下になります.

![Correct shadowing](https://docs.unity3d.com/uploads/Main/ShadowBiasGood.jpg)

　シャドウ・アクネができると縞模様のようなものが出現します.

![Shadowing with shado acne artifact](https://docs.unity3d.com/uploads/Main/ShadowBiasAcne.jpg)

 　オブジェクトの表面は相対的に滑らかなのですが, シャドウ・マップに格納する場合に発生する離散化誤差のようです.[<sup>8</sup>](#ref-8) こうした現象のためにバイアスが存在します.

 ![Bias setting](../../img/Unity_11.png)

  ただし適切に設定しないとlight bleedingが起きるそうです. これは影領域が光に侵食される現象です.

### パースペクティブ・アライアス

　シャドウ・マップはライトの視点から作られるもの出たが, シーン全体はカメラから見た映像になります. この場合, 以下の図のように近いものと遠い場所で解像度が違ってきます. このため手前のエリアではブロック・ノイズが発生します.

![Perspective Aliasing](https://docs.unity3d.com/uploads/Main/ShadMapFrustumDiagram.svg)


　解像度を手前に合わせることもできますが, これだとコンピュータ資源を食います.

### カスケード・シャドウマップ

　ではどうするかというとカメラから距離に応じて多解像度のシャドウ・マップを作ります. カスケード・シャドウ・マップ(Parallel Split Shadow Maps)とは多解像度のシャドウ・マップのことです. シェーディング・モードをShadow Cascadesに変更することで確認できます.

![Shading mode](../../img/Unity_12.png)

### シャドウ・パンケーキング

　照明の対象領域を限定するっぽいのだがよく分からなかった.
<!-- TODO: 理解する -->

![Shadow pancaking](https://docs.unity3d.com/uploads/Main/PancakingIdea.png)
　
### キャスト・シャドウ

　オブジェクト側で影が表面に生成されるか個別に設定できる.

## Lightmapping

### 種類

　3種類あるようです.

+ GPU-accelerated Lightmapper
+ CPU Progresive Lightmapper
+ Enlighten

　Lighting settingsから選択できる. デフォルトはCPUプログレス・ライトマッパーになっている.

![Lightmapper](../../img/Unity_13.png)

　後は飛ばした.

<!-- TODO: ちゃんと調べる -->

<iframe width="560" height="315" src="https://www.youtube.com/embed/tN33YqhfVtI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Lightmapperとは?

　ライト・マップはシャドウ・マップと逆に光をテクスチャに焼き込んだものです.

### Enlighten

 GI用のライティング・システムらしいです.

## UnityにおけるGI

　そのため2種類の方法があるそうです.

+ Realtime Global Illumination (Precomputed Realtime GI)
+ Baked GI
+ Mixed

### Baked GI/Lightmaps

　CGアニメ映画では普通に使われているGIですが, ゲームだとリアルタイム性が重要なので遅延するぐらいなら簡単なモデルで済ませることになります. しかし近年はその辺の要求も上がっているので何とかそれらしい映像を作る技術としてベイク(焼き込み)という技術が用いられています. やることは単純で, 事前に計算した間接光をテクスチャ(ライトマップと呼ばれる)として利用するわけです. ただし(というか当然の制約として)静的なオブジェクトにしか使えません. ソフト・シャドウとかいうものにも良いらしいです.

### Realtime Global Illumination

　Precomputed(事前計算)とRealtime(即時)という一見相反する言葉が含まれていますが, 事前計算リアルタイムGIでは事前に計算するのは光の軌跡です. 具体的な解説がないので良くわからないのですが, 設定された環境でとにかく全ての光の跳ね返りを事前に計算するそうです.

>  ..., but rather it precomputes all possible light bounces and encodes this information for use at runtime.

　この情報を使ってリアルタイムにGIを計算するようです.

> The final lighting is done at runtime by feeding the actual lights present into these previously computed light propagation paths.

 この方法なら計算自体はリアルタイムに行われるので対象は静的オブジェクトに限りません. またベイクされていないのでオブジェクトのマテリアルも変更可能です.
 
   計算量的にはベイクGI < 事前計算リアルタイムGIで, また影の品質もベイクGIの方が良いようです.

> While Precomputed Realtime GI also results in soft shadows, they will typically have to be more coarse-grained than what can be achieved with Baked GI unless the scene is very small.

### Mixed GI

 リアルタイムGIとベイクGIを両方使う設定ですが, 自分のMac mini(Late 2014)だと4コアがマックス近くになりました.

### 限界

　両方とも静的な環境で得た情報を使っているので動くオブジェクト(動的オブジェクト)には使えません. その場合はプローブを使うそうです.

## Lighting Settings

　GIの設定とかはWindow -> RenderingにあるLighting Settingsから行います.

### Sceneタブ

　シーン全体に関するGI(環境光)のセッチング

### Realtime Lighting

　Realtime LightingはRealtime GIの利用に関するチェックすボックスがあるだけです. これをチェックするとRealtime LightmapsタブにUVが展開されます.

![Lighting Settings Panel](../../img/Unity_001.png)

　当然これを切ってしまうと, 
　
### Debug Settings

　Auto Generateがデフォルトでチェックされている. この場合, Realtime LightingやMixed Lightingを有効にしていると自動的にライトマップの生成が行われる.

### ライト・プローブ

　空間中に設置された光の観測点です. ここを通過した光を保存して実行時に近似的に動的なGIが再現できます. 下図の黄色い点がプローブです. 赤線はプローブをつないだ線で構成される三角錐に照射対象が入るとライティングが計算されます.

![Light Probes](https://docs.unity3d.com/uploads/Main/LightProbes-0.png)

### リンギング

　背面にライトが突き抜けてしまうような現象です.

![Ringing](https://docs.unity3d.com/uploads/Main/class-LightProbeGroup-Ringing.png)　

### 

## 最適化において考慮する点

　色々な最適かのピットフォールがあるようです. 事前計算自体が何時間もかかったのでは効率化の意味が薄れます.

+ Appropriate Lighting Resolution
+ Charts
+ Probe lighting
+ Improve on auto UV unwrapping
+ Clusters
+ Lightmap Parameters


## Reference
1. <a name="ref-1"></a> [Introduciton to Precomputed Realtime GI](https://unity3d.com/learn/tutorials/topics/graphics/introduction-precomputed-realtime-gi?playlist=17102)
2. <a name="ref-2"></a> [Lighting strategy](https://docs.unity3d.com/Manual/BestPracticeMakingBelievableVisuals3.html)
3. <a name="ref-3"></a> [Lighting Modes](https://docs.unity3d.com/Manual/LightModes.html)
4. <a name="ref-4"></a> [Lighting overview](https://docs.unity3d.com/Manual/LightingInUnity.html)
5. <a name="ref-5"></a> [ライティングを理解するためにコーネルボックスを作って遊ぶ](http://tsubakit1.hateblo.jp/entry/2016/03/14/230904)
6. <a name="ref-6"></a> [コーネルボックスをつくってあそぼ](https://qiita.com/thiroaki/items/7a9213150b15810137f2)
7. <a name="ref-7"></a> [コーネルボックス](https://monobook.org/wiki/%E3%82%B3%E3%83%BC%E3%83%8D%E3%83%AB%E3%83%9C%E3%83%83%E3%82%AF%E3%82%B9)
8. <a name="ref-8"></a> [シャドウマッピングの基本](http://www.project-asura.com/program/d3d11/d3d11_008.html)
