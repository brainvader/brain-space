+++
title= 'Unity入門 (2) Graphics入門'
description= 'UnityのGraphicsチュートリアルのメモ'
date = 2019-05-02T22:19:40Z

[taxonomies]
categories = ["programming"]
tags= ["Unity"]
+++

# Basic of Unity on Graphics

## Rendering Path

+ Forward Rendering
+ Deferred Rendering

### Forward Rendering [<sup>1</sup>](#ref-1)

　オブジェクトを１以上の複数のパスでレンダリングします. このパス数がライトの数で決まります.レンダリング速いのと, MSAA(Multi-Sample Anti-Aliasing)のようなテクニックが使えるそうです. ライトの数が増えるとそれだけパスが増えるので, たくさんのライトが必要なプロジェクトのは向かない. 透明の処理も速いらしいけどこの辺は理屈が分からない.

### Deferred Rendering [<sup>2</sup>](#ref-2)

　シェーディングとブレンディングを遅延して行う. ２つのパスからなる.

+ G-buffer pas
+ Lighting pass

　G-bufferはいくつかの頂点属性からなる複数のバッファーである. これとDepthバッファの情報を利用してLightingパスでライティングが行われる. つまりSceen Spaceで計算が行われる.

### レンダリング・パスの設定

　設定のやり方は２つある.[<sup>3</sup>](#ref-3)

> The rendering path used by your project is chosen on the Graphics window. Additionally, you can override it for each Camera.

+ Edior -> Project Settings -> Graphics category
+ Camera Object -> Cameraコンポーネント -> Rendering Path

![Rendering Path on Graphics Panel](../../img/Unity_002.png)  

![Rendering Path on Camera Component](../../img/Unity_003.png)

## Colo Space

### Color Spaceの種類

　カラー・スペースには２つの選択肢があるようです.

+ Gamma Space
+ Linear Space

![Differences between linear and gamma color space](https://docs.unity3d.com/uploads/Main/LinearRendering-LightingSphereLinearGamma.png)

　Linearの方が精確らしいです. 

> ...linear color space rendering gives more precise results.

　カラー・スペースは以下に影響するそうです. ~~またシェーダのライティング・モデルの入力として違う数値が渡されるので当然違う色がレンダリングされる.~~

+ ライティング計算のカラーの合成
+ テクスチャからの色の読み込み

　リニア・スペースの方が色としては正確です. 普通黒から白へのグラディエントを考えたら, 両色の間は線形に埋めるでしょう.

　ガンマ・スペースはより人間の目に近いカラー・スペースになるようです. 人間の目は明るいが感度が低いらしいです. 例えばろうそくを倍々に追加して行くと(1個, 2個, 4個...)輝度は指数関数的に増えるはずですが, 人間の目には線形に増えるようにしか見えないそうです[<sup>5</sup>](#ref-5). そうすると暗い色(黒に近い色)により多くの階調を割いた方が良いでしょう. 

　歴史的にモニターとディスプレイが同じような特徴を持っていたので[<sup>6</sup>](#ref-6)リニア・スペースだと不自然になるのでガンマ補正(Gamma Correction)という補正をかける必要があったそうです. 現代のデジタル化されたモニターこれを引き継いでガンマ補正後の信号(Gamma-Encoded Signal)を入力として受け取るという基準を採用しており, これをsRGBというそうです.

### 使い分け

　単純にリニア・スペースが採用できるハードウェアならリニアの方が良いということらしいです.

### More on Linear and Gamma Spaces

　Unityのドキュメントでは良く分からなかったので, [<sup>4</sup>](#ref-4)をベースにもう少しカラー・スペースについてい考えて見る そもそもガンマというのは累乗の指数のことです. この値によって補正をかけるわけです.

$$ x^{gamma}$$

　以下は$gamma = 0.45, 1.0, 2.2$の場合の画像です.

![適用される](https://static1.squarespace.com/static/525ed7ade4b0924d2f499223/t/575f4222e321408871d71230/1465860711911/Images+representing+various+gamma+values?format=750w)

　色としては中央が正しいです. この色をそのまま表示したいわけですが, 問題はモニターはsRGBを採用しているのでガンマ補正をして表示します. デジカメなどはこうした変換を前提にして保存しています. よってコンピュータ上は左側のデータを保持しています. それをモニターに出力するときに2.2辺りガンマ値を使って補正するそうです. そうすると中央に近い色が出ます. 

　[<sup>4</sup>](#ref-4)でgammaパイプラインとlinearパイプラインの比較が分かりやすいです. 物理的に正しい描画をするにはパイプラインの途中で入り込むガンマ補正による歪みを取り除く必要があります. 下の図では, まずマテリアルのカラーとテクスチャがすでにガンマ補正の影響を受けているのでそれを取り除いた後にシェーディングをして, 再度ガンマ補正をかけて表示している様子を表しています(下図).

![A comparison of Gamma and Linear pipelines](https://static1.squarespace.com/static/525ed7ade4b0924d2f499223/t/575f42e327d4bdc48f2261e4/1465860851928/An+image+comparing+gamma+and+linear+pipelines?format=1000w)

### ガンマ補正

　[<sup>5</sup>](#ref-5)によると昔のモニターにガンマ補正が必要だったのはブラウン管の特性らしいです.

> 昔のブラウン管テレビが陰極管の物理特性により入力信号の電圧に対して出力する輝度が比例しなかったので、信号側で補正する事にしたそうです。

　[<sup>6</sup>](#ref-6)によると結果として以下のような効果があったようです.

> CRTディスプレイが持つ冪関数的な濃度階調は、CRTに使われる三極管の性質によるものでもあったが、人間の視覚にとっては階調を均等に感じさせる効果があった（ヴェーバー‐フェヒナーの法則）。

### 設定

　Editor -> Project -> Settings -> PlayerのOther SettingsにColor Spaceという項目がある.

![Color Space](../../img/Unity_004.png)

## HDR

### 光に関係する単位

#### 種類

+ Lm (ルーメン)
+ Lux (ルクス)
+ cd (カンデラ)
+ nit (ニト)
+ W ()
+ Brightness (明るさ)
+ Luminance (輝度)
+ Illuminance (照度)
+ Lightness (明度)

#### ルーメン(全光束)

　光源(ランプ)の表面から発生する全ての光束の量を計測した単位です. つまり光の総量です. このことから全光束とも呼ぶようです. 積分球というので測定するようです.

[![積分球](https://www.diylabo.jp/images/column-121-02.jpg)](https://www.diylabo.jp/column/column-121.html)

　つまり,

> 「すべての方向に対して1カンデラの光度を持つ標準の点光源が1ステラジアンの立体角内に放出する光束」<a name="ref-12">12</a> 

　となります.

#### カンデラ(光度)

　単位立体角(sr)あたりの光束(ルーメン)です(Lm/sr). ある方向への光の強さ(放射強度)のことです. 実際の照明器具では, 中心光度とか最大光度を指すようです. 昔は1カンデラはだいたいろうそく一本の明るだったらしいです.

#### ルクス(照度)

　単位面積当たりに入射する光束のことで, 光源に照らされた場所で計測した値のようです. 単位は単位面積当たりの光度($cd/m^2$)となります. 光源が照らす場所が分かると照射角度が分かりますので, 実際にどれぐらい明るく照らせるかを表しています. このことから"照度"の単位として使われます.

#### ニト(輝度)

　ルクスは光があたっている場所に入射する光束でした. それを見る人の立場で表したのがニトです. つまりある方向から見た単位面積当たりの明るさ(ルクス)のことです.

#### ルクスとカンデラの関係

　 ルクスとカンデラには以下のような関係があります.

$$ Lm = \frac{cd}{m^2} $$

![ルクスとカンデラの関係](https://www.cateye.com/jp/enjoy_lights/img/3bb04ff285923a2ab6eb5e3f020077e22510e4f9.png)

#### まとめ

　以上の単位は以下のように整理できます.<a name="ref-11">11</a> 

![光源, 光束, 照度, 光度, 輝度の違い](https://www.daisaku-shoji.co.jp/iel/img/light_word.jpg)

#### 光束とは? 放射量と測光量の関係から<a name="ref-14">14</a> 

　ザクっと単位を確認したが良く分からないことも多い. 特に光束が良く分からなかった. これは以下のようが２つの概念が光に関して関係しているのが理由のようです.

+ 放射量(radiant quantities): 放射に伴う物理量の総称
+ 測光量(luminous quantities): 放射量を人間の目が感じる明るさに変換した量

　 まず光は究極的には電磁気的なエネルギーが伝播したもので, 簡単にいうとエネルギーの流れです.

> ..., ある着目したエリアを単位時間（例えば1秒間）にどれだけのエネルギーが通過していくかという放射量を「放射束」と言います。

　ある物理量が単位面積当たり単位時間当たりに流れる量をフラックス(流量)といい, 放射エネエルギーの場合は

$$ W = J/s = A \times V $$

　と表せます. つまり放射束(luminous flux)とは放射エネルギーのフラックスと言えます. しかし, これは光束とは少し意味合いが違います. 光は電磁波なので単純に光に置き換えても良さそうですが, 実際には人間の目というセンサーに限定されます. これは暗に可視光=光という前提があることを意味します. 放射量を人間の目を基準に変換したものを測光量と呼びびます. 放射エネルギーに対応するのが光度エネルギーで, このフラックスを光束(luminous flux)と呼んでいます. この単位がルーメンなのです.

　両者は基本的には同じものなので, どちらもワットが単位として使われますが, 人間の目というセンサーの特性を考慮して使い分けられているようです.

> 「放射束」を「人間の眼」で見るということは、「人間の眼」の分光応答度特性、すなわち標準分光視感効率 V ( λ )で評価する、ということになります。この時に「人間の眼」が感じる明るさが「光束」です。

#### 放射量と放射エネルギー

　放射エネルギーはジュールで, じゃ放射量は何かというと放射量というカテゴリの中に, 放射エネルギー[J]や放射束[W]が含まれるようです. 放射量とは放射に関係する物理量の総称のようです.

#### 放射量と測光量の対応関係

<table>
  <thead>
    <tr>
      <th>放射量</th>
      <th>測光量</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>放射エネルギー　[J = W・s]</td>
      <td>光度エネルギー　[lm・s]</td>
    </tr>
    <tr>
      <td>放射束　[W]</td>
      <td>光束　[lm]</td>
    </tr>
    <tr>
      <td>放射強度　[W/sr]</td>
      <td>光度　[cd]</td>
    </tr>
    <tr>
      <td>放射輝度　[W/sr/m^2]</td>
      <td>輝度　[cd/m^2 = nt]</td>
    </tr>
    <tr>
      <td>放射照度　[W/m^2]</td>
      <td>照度　[lx = lm/m^2]</td>
    </tr>
  </tbody>
</table>

 [放射量の国際単位系](https://ja.wikipedia.org/wiki/%E6%94%BE%E5%B0%84%E3%82%A8%E3%83%8D%E3%83%AB%E3%82%AE%E3%83%BC)と[測光の国際単位系](https://ja.wikipedia.org/wiki/%E5%85%89%E5%BA%A6%E3%82%A8%E3%83%8D%E3%83%AB%E3%82%AE%E3%83%BC)を元に作成

#### 最大視感効果度<a name="ref-14">14</a>  <a name="ref-15">15</a> 

　放射量から測光量への変換に使う係数です. 

$$
\begin{aligned}
\Phi_{e} &= \int P(\lambda)\cdot d\lambda \\
\Phi_{v} &= K_{m}\int P(\lambda)\cdot V(\lambda) \cdot d\lambda 
\end{aligned}
$$

　ここで$P(\lambda)$[W/nm]を分光分布, $V(\lambda)$を標準分光視感効率です.

#### 分光視感効率

> このグラフは，物理的に同じエネルギーを持った単色光が，ある波長の時にどれだけの明暗量（正確には輝度）としてヒトに知覚されるかを相対的に表したもので..., 480nmの単色光は560nmの単色光よりも約0.1倍明るく見える（＝10倍暗く見える）...[<sup>7</sup>](#ref-7)

![標準分光視感効率](https://cdn-ak.f.st-hatena.com/images/fotolife/M/Mzawa2/20180211/20180211161554.jpg)


#### 輝度 (Luminance)

　分光視覚効率におけるある一点における明暗の量のことを輝度というらしいです. 以下のような式で定義されます(単位は$cd/m^2$ = nt).

$$ L = 683\int_{730\,nm}^{380\,nm}E(\lambda)V(\lambda)d\lambda$$

　積分は足し算のようなものなので線型性が成り立ちます. 線型性というとあれですが, 光源が増えると2倍, 3倍と増えて行くという感じでしょうか?

#### 光度 (Luminous Intencity)

　すでに説明しましたがステラジアン辺りのルーメンです. また輝度を面積で積分するとカンデラになります.

#### 光束 (Luminous flux)

　単位はルーメン(lm)で点光源の光度を全球面積分したような値です.

#### 照度 (Illuminance)

　光源の明るさとは別に光で照らされている場所の明るさです. 同じ光源でも照射される面との距離で変化します.

### HDRとは?

　High Dynamic Rangeの略ですが説明の前にLDR(Low Dynamic Range)の話をしましょう. 色はRGBの3チェネルから構成されており各チャネルは8ビットです. 色の組み合わせは$256^3$となります. 白色が(255, 255, 255)で黒色が(0, 0, 0)となります. さて1600万色も使えるわけですが, これは現実世界の色の組み合わせは無制限に存在します. 明るさHDRの色は量子化したものなので量子化誤差が存在するわけです. また現実世界にはこの色空間外にも色が存在します. 現実世界ほどではないけれどもこのLDRを超えた明るさをUnityは扱えます. 制約となるのはモニタの性能です. HDRを使うと色として浮動小数点数が扱えるようになりより精度の高い色を表現できます.

+ 室外と影のコントラスト
+ Bloom Effect
+ Glow Effect

### Tonemapping

　階調のリバランスをして値がクランプ(値をある範囲に収める処理)されないようにする変換のことのようです. つまり画面内の相対的な階調(tone)は維持されるが桁溢れのような現象は起きていない画面作りとなる. HDRカメラの映像をポスト・プロセスとかで上手に変換してくれるらしい.

### HDRが使えない場合

 + ハードウェアが対応していない時
 + MSAAをフォーワード・レンダリングで利用している時

### 設定

  カメラ・コンポーネントのHDRチェックボックスから行います.

## Reflection

　UnityはStarndard Shaderは物理ベース・レンダリング(PBR)を行います. マテリアルはSpecularity(鏡面性)/metalness(金属性)の基づいた反射性(reflectivity)があります. この程度に基づいて反射が再現されるわけです. 通常は

### Physically Based Rendering

　光の物理的特性を再現するように設定できるシェーダ・プログラムのことだと思います. 

+ Energy Conservation
+ Fresnel Reflection
+ Geometry Term

　などの原理に従うとのことです.

### Standard Shader

　いくつかのシェーダー・タイプ(シェーダの変数のこと?)があるそうです.

+ Diffuse
+ Specular
+ Bumped Specular
+ Reflective

　これら属性を組み合わせることで１つのスタンダード・シェーダを構成し, マテリアル・システムで利用されます. この統合されたシェーダ・モデルでは, 金属マテリアルを個別に作るのではなく, マテリアルの反射率を調整することで金属の見た目を再現するわけです.

### Pre-Rendering Reflection

　性能が低い場合はキューブ・マップに焼き込みしたテクスチャ・マップを使います. この値を元にブレンディングなどが行われるようです.

### Reflection Source

　反射する対象といことでしょうか. つまり鏡面に写り込むのはなんなのかを指定できるようです.

![反射源の設定](../../img/Unity_005.png)

### Reflection Probe

　サンプルなら別としてSkyboxだけ反射できる機能に意味はないでしょう. は任意の場所に置かれるオブジェクトが必ずしも全天を映し出すわけではありません. 遮蔽物なども当然映り込む, 室内なら空が映るわけはないのです. このような場合に使うのがリフレクション・プローブです. 元のオブジェクトの代わりにキューブ・マップを生成し, それを使って鏡面映り込みを近似的に再現するわけです.

+ Baked (焼き込み)
+ Custom (カスタム)
+ Realtime (リアルタイム)

　ベイクの場合は指定したオブジェクトだけが鏡面に映り込みます. リアルタイムは全てですが, 計算が大変らしいです.


## Light Sources

　光源とは要するに光を発するオブジェクトです.  ライトの作り方は3種類あります.

+ Light Objects
+ Ambient light
+ Emissive materials

　Ambientライトは環境光のことです. Emissiveマテリアルは

### 光源の種類 (Light Types)

　Unityには4種類の光源が存在します.

+ Directional Light
+ Point Light
+ Spotlithg
+ Aera Light

### Ambient Lighting

> This can be thought of as a global light source affecting objects in the scene from every direction.

　 要するに環境光です. シーン全体を明るくできます. <a name="ref-16">16</a> 

![Effect of Ambient Lighting](https://unity3d.com/sites/default/files/ambientlightab_0.png)

　設定はLighting Settingsから行います.

![Environmental Lighting](../../img/Unity_006.png)

### Emissive Materials (発光素材)

　エリア・ライトに似た性質を持つ光源になります. スタンダード・シェーダの発光属性から設定できます. ネオンとかに使えるそうです.<a name="ref-17">17</a> 

![Neon Effect](https://unity3d.com/sites/default/files/emissives_2.png)

　ぼんやりした光(soft light)なので基本的に静的なオブジェクトにだけ影響を与えます. 動いているオブジェクトにも光の影響を及したい場合は他の場合と同じようにライティング・プローブを使うと良いようです.

### Light Probs

　指定した位置での光の情報を記録します.　この近くをオブジェクトが移動する場合は, GIやどの光の効果が見込めるようになります.

## Unityのライティング

　光にはおおよそ2種類あります.

+ Direct Light (直接光)
+ Indirect Light (間接光)

　直接光は比較的簡単に扱えます. 一方, 間接光は計算が複雑になります. 光源から表面を到達した光が反射されて空間上を跳ね返りながらカメラに入ってきます. このパスを正確に計算すると物理的に正しい絵ができるようです. 

## Reference
1. <a name="ref-1"></a> [Forward Rendering Path Details](https://docs.unity3d.com/2019.1/Documentation/Manual/RenderTech-ForwardRendering.html)  
2. <a name="ref-2"></a> [Deferred shading rendering path](https://docs.unity3d.com/2019.1/Documentation/Manual/RenderTech-DeferredShading.html)  
3. <a name="ref-3"></a> [Rendering Paths](https://docs.unity3d.com/Manual/RenderingPaths.html)  
4. <a name="ref-4"></a> [GAMMA AND LINEAR SPACE - WHAT THEY ARE AND HOW THEY DIFFER](http://www.kinematicsoup.com/news/2016/6/15/gamma-and-linear-space-what-they-are-how-they-differ)  
5. <a name="ref-5"></a> [ガンマ補正のうんちく](https://qiita.com/yoya/items/122b93970c190068c752)  
6. <a name="ref-6"></a> [ガンマ値 Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%AC%E3%83%B3%E3%83%9E%E5%80%A4)  
7. <a name="ref-7"></a> [すっごくわかりにくい「明るさ」のはなし　〜Aqoursを添えて〜](http://13mzawa2.hateblo.jp/entry/2018/02/12/005402)  
8. <a name="ref-8"></a> [ルーメン、カンデラ、ルクスの違いってなぁに？](https://www.cateye.com/jp/enjoy_lights/unit.html)  
9. <a name="ref-9"></a> [カンデラ、ルーメン、ルクスの違いをスッキリ理解](https://www.diylabo.jp/column/column-121.html)  
<a name="ref-10">10</a> [ルーメン・カンデラ・ルクスの違い](https://www.monotaro.com/s/pages/productinfo/akarusa/)  
<a name="ref-11">11</a> [証明用語の基礎知識](https://www.daisaku-shoji.co.jp/iel/w_difference.html)  
<a name="ref-12">12</a> [明るさの単位「カンデラ」「ルーメン」「ルクス」の違い](https://www.irodorinet.jp/irodori-column/brightness-unit-of/)  
<a name="ref-13">13</a> [第7回　放射量 と 測光量（その4](https://www.ccs-inc.co.jp/museum/column/)  
<a name="ref-14">14</a> [第4回 放射量 と 測光量（その1）](https://www.ccs-inc.co.jp/guide/column/light_color/vol04.html)  
<a name="ref-15">15</a> [放射束 ウィキペディア](https://ja.wikipedia.org/wiki/%E6%94%BE%E5%B0%84%E6%9D%9F)  
<a name="ref-16">16</a> [Ambient Lighting on Unity Tutorial](https://unity3d.com/learn/tutorials/topics/graphics/ambient-lighting?playlist=17102)  
<a name="ref-17">17</a> [Emissive Materials on Unity Tutorial](https://unity3d.com/learn/tutorials/topics/graphics/emissive-materials?playlist=17102)  

## Reference
+ [Graphics Section on Unity Tutorial](https://unity3d.com/learn/tutorials/s/graphics)
+ [Optimizing graphics rendering in Unity games](https://unity3d.com/learn/tutorials/temas/performance-optimization/optimizing-graphics-rendering-unity-games)
+ [【Unity】Unity 5.5以降のRender Pathの設定](http://tsubakit1.hateblo.jp/entry/2016/09/25/233000)
+ [【Unity】ForwardレンダリングとDeferredレンダリングの違いを軽くまとめてみた](https://qiita.com/r-ngtm/items/a78ca2f72a36e2005da3)
+ [Deferred Shadingについてまとめてみた](http://esprog.hatenablog.com/entry/2016/03/17/142939)
+ [7 Tips For Better Lighting in Unity](https://medium.com/@EightyLevel/7-tips-for-better-lighting-in-unity-686694892ece)
