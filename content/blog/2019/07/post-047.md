+++
title = "崩壊3rdの研究 (2) Physically Based Refraction"
description = "崩壊3rdの技術について勉強する"
date = 2019-07-18T10:36:17Z

[taxonomies]
categories = ["programming"]
tags = ["Unity"]
+++

## 視差マッピング

　PBR(Phusically Based Refraction)のもう少し簡単な例を考えましょう. 何らかの計算でテクスチャを移動させるというのが視差マッピングと呼ばれます.{{ref(id=1)}} 計算方法はそれほど難しくはないです. 基本は{{ katex(body="k\cdot\mathbf{v}")}}という感じで, あるUV座標を視線ベクトルに沿ってk倍の位置までずらします. と言ってもUV座標は2次元ですのでxとyの値しかありません. これらの計算はフラグメント・シェーダで行われるのでラスタライズされた後です. 深度値は持っていますが, これはZテストでレンダリングしないピクセルを省くためです. 当然頂点の前後関係は変わらないので不自然な部分は生じますが, 逆にこれだけでそれっぽい絵が簡単に表現できるということでもあります. 実際に頂点を変更する場合は頂点シェーダかジオメトリ・シェーダを使えばいいのだと思います. {{ref(id=2)}}

　なぜではそれっぽい画面が生成できるのだろうか? これはUVがポリゴンに貼り付けられているのではなく, 実際にはテクスチャから対応する色をサンプリングしていることに原因がある.{{ref(id=3)}}

![バンプ・マッピングの概略図](https://news.mynavi.jp/article/graphics-17/images/005.jpg)

　視差マッピングには法線情報を提供するテクスチャとは別に高さ情報を提供するバンプ・マップ(高さマップ)を使う. このマップでマスクされた領域には高さの情報が加わる. 図でカメラか伸びる二つの視線がそれぞれ違う高さマップと交わって, その直下にあるテクスチャをサンプリングすることになっている. 一方視差マッピングでは(局面)で屈折して直進した場合とは違うテクスチャの色情報を採取しているのが見えると思う.

![視差マッピングの概略図](https://news.mynavi.jp/article/graphics-17/images/006.jpg)

　実際にはこの図からも誤差があることが分かるが"生々しさ"とリアルタイム性を両立するにはこで十分な表現力を持つわけです.

## Physically Based Refraction

　以上の視差マッピングの内容を踏まえると物理ベースの瞳での屈折の理解が進むと思います.

![Parallax Refraction](https://i.stack.imgur.com/a08kE.png)

　上の図とよく似ていることがわかると思います. ただしハイト・マップの代わりに頂点の座標を使います. 平面を浮かび上がらせるのとは逆で, 角度によってレンズの位置を奥に引っ込ませる処理が必要になるわけです. このポリゴンとレンズ(水晶体)までの空間をanterior chamber(前眼房)と呼ぶらしいです. 計算はややこしいですが公式化されていますので機械的に計算できます.

　具体的な計算方法などは[PHYSICALLY BASED PLANAR REFRACTION SHADER](http://popupasylum.co.uk/?p=1109)が詳しかったです. この記事でも紹介されているがGLSLにはrefractという組み込み関数が存在し, これを使っています.{{ref(id=4)}} ただUnityにはrefract関数がないので自分で実装する必要があります.

　Stack Exchangeにも[Eye Parallax Refraction Shader](https://gamedev.stackexchange.com/questions/131619/eye-parallax-refraction-shader)という質問が投稿されている. これも詳しい説明はないです.

　最後が[Eye shader: Physically based Refraction +[Source code]](https://forum.unity.com/threads/eye-shader-physically-based-refraction-source-code.441270/)でこれはUnityのスクリプトの実装があります. ここで紹介されている屈折のアルゴリズムはGLSLのビルトイン関数であるrefract関数やStack Exchangeで紹介されているアルゴリズムと同じようです. 実際に動くしこれが一番分かりやすいのです. おそらくこれでもそれらしく見せることはできそうですが, 問題は崩壊3rdの公演で紹介されている式とは全然違うということです.

## 崩壊3rdの場合

　崩壊3rdは方向余弦を計算しているように見えます.

{% katex(block=true) %}
\begin{aligned}
  &half3\,V = normalize(\_WorldSpaceCameraPos.xyz - o.objPos); \\
  &float3\,offset = float3(dot(V, uWorld), dot(V, vWorld), dot(V, eyeForward));
\end{aligned}
{% end %}

　この解釈が難しいのですが, 瞳の部分(黒目に相当)は平坦で角度によって膨らませる方法では無いかなぁと思いました. そしてその方法として視差マッピングが使えそうです.{{ref(id=3)}} 今黒目は平としているのでuWorldとvWorldはこの平面を構成するベクトルと考えられます. つまり接ベクトルと従法線ベクトルです.{{ref(id=5)}} それを視差マッピングの要領で視線ベクトル方向に持ち上げます. 前眼房の高さを考慮する必要もあるかもしれません.

## Footnote

{{anchor(id=1)}} [【Unity】視差マッピング](http://light11.hatenadiary.com/entry/2018/02/14/205444)
{{anchor(id=2)}} [[Unity3D] Intro to Geometry Shader](https://jayjingyuliu.wordpress.com/2018/01/24/unity3d-intro-to-geometry-shader/)
{{anchor(id=3)}}  [バンプマッピングの先にあるもの(1)～視差マッピング](https://news.mynavi.jp/article/graphics-17/)
{{anchor(id=4)}} [refract](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/refract.xhtml)
{{anchor(id=5)}} [法線マップと接空間](http://esprog.hatenablog.com/entry/2016/05/01/025634)

## Reference

+ [瞳の作り方メイキング](https://ch.nicovideo.jp/wachi/blomaga/ar538057)
+ [細部まで徹底的にこだわったキャラクター造形～中国と日本が連携して挑んだ最先端CGアニメ『崩壊3rd』PV『女王降臨』Vol.1](https://cgworld.jp/feature/201809-cgw241hs-houkai3rd01.html)
+ [Texture Mask Shader in Unity Tutorial](https://lindenreid.wordpress.com/2018/02/25/texture-mask-shader-unity-tutorial/)
+ [Vertex shader](https://www.febucci.com/2018/10/vertex-shader/)
+ [Lean OpenGL Parallax Mapping](https://learnopengl.com/Advanced-Lighting/Parallax-Mapping)
+ [3.Parallax mapping in Advanced shaders in Unity](https://www.habrador.com/tutorials/shaders/3-parallax-mapping/)
+ [A Closer Look At Parallax Occlusion Mapping](https://www.gamedev.net/articles/programming/graphics/a-closer-look-at-parallax-occlusion-mapping-r3262/)
