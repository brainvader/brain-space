+++
title = "崩壊3rdの研究"
description = "崩壊3rdの技術について勉強する"
date = 2019-07-14T02:18:12Z

[taxonomies]
categories = ["programming"]
tags = ["Unity"]
+++

## 動機

　崩壊3rdのアニメ風レンダリングの技術という公演ビデオがアップロードとスライド{{ ref(id=1) }}されてたので見て見たが分からないところも多かった. 分からないものの単純にすごいなと興味を惹かれて関心が出てきた.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ZpWsinhPFLM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


　技術自体はすでにアニメでプリレンダリング技術が使われているので, その応用なのかもしれませんがゲームへの応用でUnityが使われているというのがポイントになりそうです. ゲームなのでステージなどの話も含まれていますがとりあえずキャラクターに焦点を当てたいと思います. Unreal Engineには

[UNREAL ENGINE HAIR MATERIAL](https://www.mizugames.com/2015/09/09/unreal-engine-hair-material/)

　というのがあってこれも似たようなことができるようです.

## アニメ風とは?

　Toon Shadingと言われる技術で主にフラグメント・シェーダでそれっぽい見た目を再現することになるようです. 崩壊3rd自体はゲームのための技術なのでリアルタイム性も必要とされます. Stylizedという言い方もするようです. わざわざ"アニメ風"と呼ぶには付加的な要素が必要そうです.{{ ref(id=2) }}  だいたい以下の要素を満たせば良いのかなという印象を持ちました. 

+ 肌
+ 瞳
+ 髪
+ アウトライン

　特に線画の有無はアニメ調かどうかを決める大きな要素かなと思います. これがないと漫画のアニメ化というのはほぼ不可能でしょう. 肌の質感や瞳, そして髪の毛も独特の表現が要求されます.

## 技術スタック

　静止画ならテクスチャ・ペインティグとかで書き込むのが一番簡単だと思いますが, ゲームでは光源やカメラの位置でスペキュラの出方が変わってきます.

+ Multi-Channel Ramp Coloring
+ Anisotropic material
+ IBL (Image-Based Lightin)
+ Cascade shadow map shadow
+ View-based shadow mapping
+ PCSS (Percentage Closer Soft Shadows)
+ Anisotropic Lighting
+ High Quality Outline

### Multi-Channel Ramp

　これはアニメ風だけに関わらずStylizedなレンダリングやToon Shadingでも使われるようです.{{ref(id=3)}} 特徴的なのはマルチチャネルの色階調を使っていることらしいのですが, 具体的なやり方はよく分かりません. この辺はUnityを勉強しながらやる必要がありそうです. 単なるディフューズ・シェーディングだと影が汚くなるというのは, 以前blenderで似たようなことをやろうとして難しかった記憶がある. 動画をよく見ると独自のエディタ拡張が使われているようですが今ならShader Graphを使うと簡単にできるのかもしれません. とにかく一枚のランプ・テクスチャではなく多層化することでそれらしい外観が得られるようです. またランプの階調をコントロールすることでハード/ソフト・ライトのような様々なシェーディングを表現できるとのことです. 例外は顔で影をつけないようにするために, 頂点カラーをマスクとして使うことでディフーズ・シェーディングの効果を低減させるそうです. 頂点カラーを使う理由は分かりませんが, テクスチャより頂点カラーの方が解像度が低くなるからでしょうか.

+ color ramp
+ mask texture
+ soft/hard shadow
 
#### マスク・テクスチャ

　ランプ・シェーディングのマスク・テクスチャと言っているのがよく分からなかったです. マスク・テクスチャというと画像処理などで星型の切り抜きとかをするときに使われるテクスチャを思い出します. 顔をマスクで隠すように画像の一部を隠すわけです. 3Dモデルの場合はUVマップにこのマスクを施すことで一種のマーカーとして使います. こうすることでマテリアルの効果をマスクされた部分に限定できます.{{ ref(id=4)}} マスク・テクスチャを省略してマスクなんて呼ばれたりもします.

### テクスチャ・アトラス

　UVマップのことをテクスチャ・アトラスとか言ったりするようです.

### Cascade Shadow

　全く同じサイズのオブジェクトの影は本来同じように表示されるバズなのですが, 視点からの位置で変化します. しかしピクセルは以下の図のように等方的で均一です.{{ ref(id=5) }}

![Shadow cascades](https://docs.unity3d.com/uploads/Main/ShadMapFrustumDiagram.svg)

　この画像の場合手前は4ピクセル, 奥は20ピクセルになります. ところがこれらはどちらも同じ解像度のスクリーンに表示されます. これが公演中に言及されている"近くで見るとこれあまりよくない"という部分に繋がるとも割れます. これを改善するのがView-based shadow mapらしいです. 

### View-based Shadow Map

　具体的なことには言及はないのですが, 視錐体とバウンディング・ボックスが交わるところで使うらしいです.

### Eye Refraction (瞳による屈折)

　一般的な手法として黒目を凹ませるという手法があるようです. これはコッチミンナモーフとかいうのと同じでしょうか.{{ ref(id=6)}} しかしこれも近くで見るとあまり良くないらしいです. そのためシェーダで黒目のオフセットを計算して補正するということをやっているようです. {{ ref(id=7) }} Eye Parallax Refraction
ともいうそうです. {{ ref(id=8) }} Unityのアニメーション・プロジェクトであるShermanでも使われています. {{ ref(id=9) }}

### Light Caustic

　これはマスクで表現されるらしい. 他にもディフーズ・シェーディングの係数を逆向きに使ってるとか, フレネルも使ってるらしい(よく分からん).

### Anisotropic Shader

　計算方法はWard's Model of Anisotropic Reflectionというのが使われるようです. {{ref(id=10)}} よく分からないのが

### Anime style highlight

　光源とオブジェクトの位置によってハイライトをアニメーションさせる必要があります.

### Geometry Based Outline

　アウトラインは背面法というのがスタンダードな方法とされているようです. 後は頂点カラーも利用するようです. こちらは主に調整用に使われているようです. 後は視点とオブジェクトの距離に応じて輪郭線の幅を修正しているとのことです. Creaseエッジに関してはプリプロセスで検知し, メッシュ・アセットというのに保存しているようです. 描画はジオメトリ・シェーダを使っているとのことです.

+ Backface method (背面法)
+ Perspective Correction
+ Crease Shading

　"そのものの角"?

#### Highlight stripe

　Jitterマップを引き伸ばしてテクスチャを作り, これを適用することでハイライトの帯の幅を小刻みに調整するようです.

#### Jitter map

　正直Jitterで何を刺しているのかはよく分かりません. Jitterという現象はあるようですが, それは観測に伴って起きるので数式で表せるようなものではありません. データが同じ座標で重なるときにランダムなノイズを与えて位置をずらして表示する場合をJitter plotと呼ぶようです. 単純にランダムな高周波の波をテクスチャにしたものをJitter mapと呼んでいるだけという気もします. 

 Jitterプロットとすると適当にサインカーブとかを作ってそのグラフ常にランダムな数のデータをずらして表示すればいいのかもしれません. 色々と試行錯誤が必要なようです. 床井研究所では似たようなテクスチャをバンプ・マップとして使っています. {{ ref(id=11) }} こちらは"ノイズの画像をぼかして縦方向に引き伸ばした"と説明されています.

#### もう少しリアル寄りの髪の毛

　Shin JeongHoという人がArtStationに公開している髪マテリアルも勉強になります. {{ref(id=12)}} こちらはかなりリアルよりです. テクスチャを多層にするのではなくメッシュ自体を多層にするというのは非常に興味深い方法です.

#### Mesh Asset

　メッシュ・アセット?


### まとめ

　とりあえずキャラクターのリアルタイム・レンダリング技法だけを書き出して見ました.
　
## Footnote

{{ anchor(id=1) }} [【Unite Tokyo 2018】『崩壊3rd』開発者が語るアニメ風レンダリングの極意](https://www.slideshare.net/UnityTechnologiesJapan002/unite-tokyo-20183rd)
{{ anchor(id=2) }} 中国語が分からないので厳密な対応関係は不明です. またこの辺の技術の定義や差異もよく分かっていません.
{{ anchor(id=3) }} [Creating A Stylized Waterfall in Unity](https://80.lv/articles/creating-a-stylized-waterfall-in-unity-part-1/)
{{ anchor(id=4) }} [Using Texture Masks](https://docs.unrealengine.com/en-US/Engine/Rendering/Materials/HowTo/Masking/index.htmls)
{{ anchor(id=5) }} [Shadow cascades](https://docs.unity3d.com/Manual/DirLightShadows.html)
{{ anchor(id=6) }} [MMDモデルの目の構造について(備忘録)](https://ch.nicovideo.jp/wani275waka/blomaga/ar1419793s)
{{ anchor(id=7) }} [Eye shader: Physically based Refraction +[Source code]](https://forum.unity.com/threads/eye-shader-physically-based-refraction-source-code.441270/)
{{ anchor(id=8) }} [Eye Parallax Refraction](https://computergraphics.stackexchange.com/questions/4133/eye-parallax-refraction)
{{ anchor(id=9) }} [Introducing Sherman (Part 1) – a Unity project featuring Real time fur, HDRP and Visual FX Graph for animators](https://blogs.unity3d.com/2019/06/11/introducing-sherman-part-1/)
{{ anchor(id=10) }} [Cg Programming/Unity/Brushed Metal](https://en.wikibooks.org/wiki/Cg_Programming/Unity/Brushed_Metal#Ward's_Model_of_Anisotropic_Reflection)
{{ anchor(id=11) }} [床井研究室 第６回 異方性反射](http://marina.sys.wakayama-u.ac.jp/~tokoi/?date=20051019)
{{ anchor(id=12) }} [Realtime Short Hair](https://www.artstation.com/artwork/3NQX2)

## Reference

+ [細部まで徹底的にこだわったキャラクター造形～中国と日本が連携して挑んだ最先端CGアニメ『崩壊3rd』PV『女王降臨』Vol.1](https://cgworld.jp/feature/201809-cgw241hs-houkai3rd01.html)
+ [アニメ調レンダリングで学ぶUnityShader](http://unityshader.hatenablog.com/entry/2017/04/02/121220)
+ [A Game of Tricks IV – Stylized normal mapping](http://www.alkemi-games.com/a-game-of-tricks-iv-stylized-normal-mapping/)
+ [Texture Mask Shader in Unity Tutorial](https://lindenreid.wordpress.com/2018/02/25/texture-mask-shader-unity-tutorial/)
+ [Unityを利用した2D画像の回転マスク表現](https://styly.cc/ja/tips/unity-rotation-mask-shader/#i-5)
+ [瞳の作り方メイキング](https://ch.nicovideo.jp/wachi/blomaga/ar538057)
+ [THIRD PERSON ANIMATION AND CHARACTER CONTROL](http://johnmcelmurray.com/unity-education-1)
+ [Tutorial 49: Cascaded Shadow Mapping](http://ogldev.atspace.co.uk/www/tutorial49/tutorial49.html)
+ [Tutorial 42: Percentage Closer Filtering](http://ogldev.atspace.co.uk/www/tutorial42/tutorial42.html)
+ [Anisotropic Highlight Shader](https://wiki.unity3d.com/index.php/Anisotropic_Highlight_Shader)
+ [裏面ポリゴンでアウトラインを表示する](https://spphire9.wordpress.com/2014/04/05/%E8%A3%8F%E9%9D%A2%E3%83%9D%E3%83%AA%E3%82%B4%E3%83%B3%E3%81%A7%E3%82%A2%E3%82%A6%E3%83%88%E3%83%A9%E3%82%A4%E3%83%B3%E3%82%92%E8%A1%A8%E7%A4%BA%E3%81%99%E3%82%8B/)
+ [なぜなにリアルタイムレンダリング](https://www.slideshare.net/SatoshiKodaira/ss-69311865)
+ [Outline Shader](https://roystan.net/articles/outline-shader.html)
+ [頂点カラーによる輪郭検出](https://twitter.com/megamarsun/status/996667614377082881)
+ [[Realtime Short Hair](https://www.artstation.com/artwork/3NQX2)](http://web.engr.oregonstate.edu/~mjb/cs519/Projects/Papers/HairRendering.pdf)
+ [UnityHairShader](https://github.com/AdamFrisby/UnityHairShader)
