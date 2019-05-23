+++
title = "Zola入門"
description = "Zolaによるブログ開発"
date = 2019-05-16T23:17:46Z

# draft = false
# aliases = [path to the new content]
# in_search_index = true
# template = "page.html"

[taxonomies]
categories = ["programming"]
tags = ["rust", "SSG"]
+++

## Zolaとは?

　このサイトはRust製の静的サイトジェネレータZolaによって開発されています. Zolaの導入はRustとCLI(CommandLine Interface)を導入するだけです. プラグイなどもなく簡潔で, Rust製だけあって高速です. 一方でAugmented MarkdownというMarkdownの拡張構文が用意されているため柔軟性も兼ね備えています. 本記事ではRust製の静的サイト・ジェネレータであるZolaの基本的な機能について解説しようと思います.

<!-- more -->

## 概要

### 設定

　config.tomlというファイルで管理します.

### zolaコマンド

　Zolaはシングル・バイナリなSSG(Static Site Generator)でコマンドライン・ツールでプロジェクトを管理します. その名もzolaと言います. zolaコマンドはシンプルなインターフェースでサブコマンドは３つしかありません.

+ init: プロジェクトの雛形生成
+ serve: 開発サーバー上での起動
+ build: 静的サイトの生成

### プロジェクトのファイル構造

　プロジェクトはinitサブコマンドで生成します.

```bash
zola init test-blog
```

　この後インタラクティブ・セッションでいくつかの質問がされます. とりあえず全部Yesでいいと思います. 設定はconfig.tomlで設定できます. 以下はzola init test-blogで生成される雛形です.

```bash
.
└── test-blog
    ├── config.toml
    ├── content
    ├── sass
    ├── static
    ├── templates
    └── themes
```

## Markdownによるコンテンツ管理

　コンテンツはmd形式(Markdownのファイル拡張子)で記述し, contentフォルダ以下に保存します. config.tomlで指定したbase_urlに続く形でファイル名を付加されパスが生成されます. 区切り文字はハイフンに変更されます. 


## セクションとページ

　コンテンツは階層化できます. やることは通常のプロジェクトと同じフォルダで階層を作るだけなのですが, Zolaのフォルダはセクションと呼ばれsection.htmlというテンプレートと結合して画面に表示することができます. その内容は_index.mdというファイルに記述します. セクションは非表示にすることもできます. あるフォルダが１つのコンテンツを表す場合はindex.mdというファイルを使います. 例えばaboutページを作成する場合about.mdとしてもいいですが, aboutフォルダの中にindex.mdを入れることもできます. この場合_index.mdがないページを孤児ページ(orphan page)と呼びます.

<!-- TODO: 孤児ページへURLをハードコードする以外でもアクセスできる? -->

### 前付 (front-matter)

　Zolaの全てのMarkdownファイルには前付けと呼ばれる部分があり, ここに設定などを書きます.

```md
+++
# front-matter
# here specify metadata
+++
# start page content from here
```

### SlugとPermalink

　パーマリンクはページのULR全体のことを指します. このURLの末尾のページの識別子をSlugと呼びます. 以下はsectionフォルダがある場合とない場合のパーマリンクにおけるslugの違いです. すでに述べましたが, ページ名やセクション名の区切り文字は全てハイフンに置き換えられます.

+ base_url/page-name/
+ base_url/section-name/

　もう少し具体的にいうとfirst_post.mdかpost/index.mdならhttps://example.com/first-postとなります.

### Asset colocation

　それぞれのページには画像やJavaScriptのような固有のアセットが必要な場合があります. sectionフォルダを使うとこれらを**asset colocation**として同一のファイルで管理できます. ちなみにアセットは全て使う必要ははなくビルドの際に除外設定することもできます.

```toml
# config.tomlに指定する!
ignored_content = ["*.hoge"]
```

　アセットはもう１つstaticというフォルダにも置けます.

#### _index.md

　_index.mdはセクション・ビュ(section.html)のデータ・ソースになります. renderとかtransparentという設定項目によって表示/非表示などを管理できます.

## Teraテンプレート・エンジン

　Zolaの内部では[Tera](https://tera.netlify.com/)と呼ばれるテンプレート・エンジンが使われています.{{ ref(id=3)}} Teraも当然Rust製です. Markdownに記述したコンテンツをビューと結合し生成物として静的ファイルを出力します. テンプレートはプロジェクト直下のtemplatesというそのままの名前のフォルダに保存します.

### スタンダード・テンプレート (標準テンプレート)

　ページの前付にはtemplateといってどのファイルをテンプレートとして用いるか指定するオプションがあります. しかし一部のコンテンツはデフォルのテンプレート名が決まっています.

+ index.html
+ section.html
+ page.html
+ カスタム・テンプレート

　上の３つは標準テンプレートと呼ばれファイルを作るとZolaが勝手にコンテンツと結びつけてくれます. index.htmlはホームページのためのテンプレートで, page.htmlはMarkdownと結合されてブログのページになります. section.htmlは主に_index.mdの内容と結合します.これ以外のテンプレートで表示したい場合はカスタム・テンプレートを作ってページ側で指定する必要があります.

```md
template = "custom.html"
```

### 変数

　テンプレートでは変数を用いてMarkdownの内容にアクセスできます. 定義もしていないしTeraのキーワードでも無いのでぱっと見はどこから来たのか分からなくて驚くかもしれませんが, 変数というやつです.

+ config
+ current_path
+ current_url
+ lang
+ section
+ page
+ paginator
+ toc
+ translations

　などがあります(他にもあるかもしれませんが). 例えばページの本文などをテンプレートに埋め込みたい場合は以下のようになります.

 ```jinja2
 <div class="post-content">
   {{ page.content | safe }}
</div>
 ```

  セクションの場合(section.html)はsectionという変数が使えます.

### Teraの構文

　ここではZolaで必要な程度のTeraの構文を紹介します. 実際はRustのデータとやりとりしたりするのですが, ZolaではMarkdownの内容を読み取るのが基本ですのでそこまで難しくありません.
 
#### 区切り文字

　3種類の区切り文字(delimiter)がある.

```jinja2
{# Expressions #}
{{ }} 

{# Statements #}
{% %}

{# Comment #}
{# #}
```

### ifブロック

```jinja2
{% if age > 9 or  num < 20 %}
    {{ name }} is a teenager.
{% elif age > 19 and age < 61 %}
　  {{ name }} is an adult.
{% else %}
    {{ name }} is an elder or a child.
{% endif %}
```

### forブロック

 forループの構文もあります. このブロックではloopという特殊な変数が使えます.

+ loop.index
+ loopindex0
+ loop.first
+ loop.last
  
　リストなどを使うのに使えるでしょう. 

```jinja2
{% for item in list %}
    <li>{{ loop.index }}: {{ item }}</li>
{% endfor　%}
```

　配列はよく有る角括弧(square brackets)を使った記法でも可能です.

```jinja2
{% for i in [1,2,3] %}
  num: {{ i }}
{% endfor %}
```

　forループははマップにも使えます. この場合はkey-valueペアを取り出します.

 ```jinja2
 {% for key, value in a_map %}
   map {{ key }} to {{ value }}
 {% endfor %}
 ```

 continueとかbreakも使えます.

### 外部テンプレートの読み込みとマクロ

```jinja2
{% import "./path/to/another_template.hteml" %}
```

 マクロの読み込みも基本的には同じですが, パスの指定ができません. マクロは関数のようなもので引数を指定することができます.

```jinja2
{# Macro Definition in#}
{% macro factorial(n) %}
   {% if n > 1 %}{{ n }} - {{ self::factorial(n=n-1) }}{% else %}1{% endif %}
{% endmacro factorial %}

{# Macro Call #}
{% import "factorial_macros.html" as factorial_macros %}
{{ factorial_macros::factorial(5) }}
```

### ブロック

 ブロック・タグで囲んだ領域は継承先で上書きすることができます.

```jinja2
{# parent.html #}
<title>{% block title %} {% endblock title %}</title>

{# child.html #}
{% extends "parent.html" %}
<title>{% block title %} Real Title {% endblock title %}</title>
```

　なお継承元の内容はsuper()で読み込むことができます.

### フィルタ, テスト, 関数

<!-- NOTE: テストと関数の違いは? -->

　いくつかTeraのビルトインの機能を紹介します. {{ ref(id=2) }}

```jinja2
{# filter #}
{% for product in products | reverse %}
  {{loop.index}}. {{product.name}}
{% endfor %}

{# test #}
{% if rating is divisibleby(2) %}
    Divisible
{% endif %}

{# function #}
```

### Zolaビルトイン・フィルタ

　いくつか機能がありますがドキュメントを見た方が早いと思います. {{ ref(id=4) }}

### Zolaのビルトイン・グローバル関数

　こちらも公式を見た方が早いです. {{ ref(id=5) }} 一応いくつか紹介しておきましょう.

+ get_page: path引数に渡したページの内容を取得する. 値はkey-valueペーアで, 本文以外にも前付やwordcountなど変数の情報も含んでいる
+ get_sectin: get_pageのセクション版
+ get_url: 指定したファイルのパーマリンクを取得する. CSSや動的にページのパーマリンクを挿入したい場合に使う.
+ get_taxonomy: kind引数で指定した全てのtaxonomy取得する・・・と書いてあるが, 内部の構造が良く分からない.
+ load_data: 指定したファイルからデータを読み込む. pathでファイルのパス指定して, formatでフィイル形式を指定する. tomlがデフォルト.
+ trans: 指定した言語への変換. 日本語は対応していないっぽいです.
+ resize_image: 画像のスケールなどができる.

#### load_data

 load_dataはurlを指定するとAjaxのようにJSONを取得することもできます.

```jinja2
{% set response = load_data(url="https://api.github.com/repos/getzola/zola", format="json") %}
{{ response }}
```


## Taxonomies

### Taxonomiesってそもそも何?

　Taxonomyを辞書で調べると分類法という意味があるようです. ブログをタグやカテゴリごとに分類する場合に使われる用語のようです. Zolaでは任意の分類法が指定できます(と言ってもただの文字列を指定するだけですが).

### タクソノミーの設定

　好きな分類名が使えますがタグとかカテゴリとかにしておくのがいいかもしれません. config.tomlにタクソノミーを定義します.

```toml
taxonomies = [
    {name = "categories", rss = false},
    {name = "tags", rss = false},
]
```

　ここで設定した名前がMarkdownの前付で利用できるようになります.

```md
+++
[taxonomies]
categories = ["programming"]
tags = ["rust", "SSG", "Zola"]
+++
```

　テンプレート側ではpage変数にtaxonomiesというプロパティがあるので, そこから名前が取得できます. 以下はブログで良くあるタグを表示するコードです. [get_taxonomy_url](https://www.getzola.org/documentation/templates/overview/#get-taxonomy-url)でタグが指定されたブログ・ポストの一覧画面へのパーマリンクを取得します. 

```jinja2
{% for tag in page.taxonomies.tags %}
    <a href="{{ get_taxonomy_url(kind="tags", name=tag) }}">#{{ tag }}</a>
{% endfor %}
```

　この場合#rustならrustでタグ付けされた記事一覧が表示されるということです. getとついていますがcreateというプレフィックスの方がいいのかもしれません. なぜならbase_url/tagとかbase_url/categoryとかbase_urlに続く形でカテゴリやタグの名前をくっつけてパーマリンクを作っているだけだからです.

## スタイリング

　スタイリングはSassが標準です. CSSを使う場合はstaticフォルダにcssというフォルダを作って起きます. cssしか使わない場合は, config.tomlのcompile_sass設定を無効にしておくといいかもしれません. その場合zola initを実行してもSassフォルダは生成されませんのでCSSファイルはstatic/css以下に置くことになります.

### 置き場所

　スタイル・ファイルは以下のどちらかに置きます.

+ staticフォルダ直下のcssフォルダ
+ sassフォルダ

　static以下のファイルはそのままコピーされます. css/base.cssというファイルがあって, これをビルドするとpublic以下にcss/base.cssがコピーされます. 一方, Sassはcssへ変換されpublic直下へ格納されます. この場合懸念されるのがSassファイルが大量にある場合どうなるのかという問題です. publicフォルダ自体をいじることはないですがすっきりとさせたい場合, もしくはモジューラな単位に分割したい場合は**Partial Sass**を使います. やることは単純で_patial.sassのようにプレフィックスとしてアンダースコアをつけるだけです.

　標準はSassフォルダなのですが僕はSassが分かりません. またCSSも変数の導入など便利になってきているのでそちらを勉強したいというのもありSassは使いませんのでstaticに基本的に置いときます. このCSSにアクセスするにはTeraグローバル関数を使います. 例えばstatic/css/base.cssへのURLを取得するなら, 

```jinja2
{% set css = get_url(path="css/base.css") %}
{{ css }}
```

 のようになります.

### CSSの新構文

+ 変数
+ インポート
+ グリッド

#### 変数 (カスタム・プロパティ)

　変数は:root内に定義するグローバル変数とローカルに定義するローカル変数があります. 変数名はハイフン２つで始まり, 呼び出す時はvar関数を使います.

```css
/* 変数の定義 */
:root {
  --div-width: 800px;
}

.theme-dark {
  --bg-color: #000000;
  --text-color: #ffffff;
}

/* 変数の利用 */
div {
  width: var(--div-width);
  background: var(--bg-color);
  color: var(--text-color);
}
```

## シンタックス・ハイライト

　ブログの機能としてシンタックス・ハイライトは欠かせません. Zolaはビルトインでシンタックス・ハイライトにも対応しています. config.tomlで有効にできます.

```toml
highlight_code = true
# highlight_theme = ""
```

　Rust製ということで, 当然Rustは対応しています.

```rust
fn main() {
  println!("Hello World!");
}
```

 そしてTomlも標準で対応しています.

```toml
toml = "Yes"
```

 Markdownもいけます.

```md
# Heading1

1. item
2. item
3. item

> 引用
```

　TeraのシンタックスはJinja2とほぼ同じっぽい(ちゃんと確認はしてません)ので, Jinja2用のテーマが

```jinja2
{% block %} {% endblock %}
```

　Pythonも大丈夫です.

```python
if __name__ == '__main__':
    print("Hello World!")
```

 その他の対応言語は公式ドキュメントを参考にしてください. {{ ref(id=8)}}

## Tips

### 草稿ページ

 書きかけのページは非表示にできます.

```toml
draft = false
```
　テンプレート側ではブール値として取り出せますのでチェックするといいと思います. 

```jinja2
{% if page.draft %}
{% endif %}
```

　テンプレートの構成によってはエラーが出たりします.

### TOC (Table Of Content)

### 内部リンク (Internal Links)

　Markdownはテンプレートに結合されてHTMLになります. そのため内部リンクの記法は存在しません(たぶん). Zolaでは特別な記法が定義されているようです. 指定するパスはcontentフォルダからの相対パスです. 公式の説明そのままですが, content/pages/about.mdなら以下のようになります. GatsbyJSではできなかった(プラグインがある?)ので一発でできるのはありがたいです.

```md
[awesome-link](./pages/about.html)
```

[third post](../post_57.md)

### Anchor Links

　アンカー・リンクを有効にするにはconfig.tomlとセクションの前付に以下を付け足します. ただデフォルでは見栄えはあまり良くないです.

```toml
insert_anchor_links = "left"
```

```md
+++
# left, right, or none
insert_anchor_links = "none"
+++
```

### Read-more機能

　ブログでさらに読む(Reading More)のような機能があります.{{ ref(id=1)}} ZolaではMarkdown内に以下のようなタグを仕込むと勝手にやってくれます.

```md
<!-- more -->
```

```jinja2
<div class="read-more">
  <a href="{{ page.permalink }}">Read more...</a>
</div>
```

### 画像と画像処理

　画像には保存場所によりいくつかの設定の仕方があります.{{ ref(id=6) }} 基本的にはブログの記事(Markdownファイル)と同じフォルダに入れるのがいいですが, 共有されるもの(ページのロゴとかfavicon, そしてJavaScriptなど)はstaticに配置するのがいいようです.{{ ref=(id=7)}}

　ファイルサイズの変更にはresize_image関数が便利です. これは直接使えませんがshortcodes内で使用してやると, Markdownでも利用できます.

```jinja2
<img src="{{ resize_image(path=path, width=width, height=height, op=op) }}" />
```

{{ resize_image(path="./blog/2019/05/ptiblock.jpeg", width=200, height=200, op="scale") }}


### ページネーション

　前のページ(previous)や次のページ(next)をつける機能です.

### ショート・コード (Shortcodes)

　ショートコードを使うとMarkdownを拡張できます. そもそもMarkdownではHTMLのタグが利用できますが, 混在すると少し読みにくいです. そこでショートコードと呼ばれる, コード片を定義しそれをMarkdownでテンプレートの用に利用できます. <del>たとえばZolaは~~で打ち消し線に変換されません(たぶん). そこでdelタグで文章を囲むことになります. </del> そこでstrike.htmlというショートコードを定義してみましょう. templates以下にshortcodesというファイルを作り, strike.htmlというファイルを作り以下のように定義します.

```jinja2
<del>{{ sentence }}</del>
```

　二重波括弧内部は関数の名前付き引数のように利用できます.

```md
{{/* strike(sentence="ダメ出し!!") */}}
```

　個人的な注意点ですが, 文字列の場合は二重引用符で囲むのを忘れないようにしましょう!! 後上の表示されているコードをコピペしてZolaのシンタックス・ハイライターにかけると展開されてしまいます. 二重波括弧の中をコメントにすると展開を防げます.

```
/* */
```

#### Anchor Links

　アンカー・リンクはMarkdownにはありませんのでそれを作ってみようと思います(ページ内リンクとも呼ぶようですが正式名称がよく分かりません). まずtemplates以下にshortcodesというフォルダを作成しさらに２つのファイルを作ります.

+ ref.html
+ anchor.html

 ```jinja2
 {# ref.html #}
 <a href="#ref-{{id}}"><sup>{{ id }}</sup></a>

 {# anchor.html #}
 {{ id }}. <a name="ref-{{ id }}"></a>
 ```
 
　shortcodesフォルダのファイルは自動的にZolaが認識します. これをMarkdown側で後は読み込みます.

```mdf
{{/* ref(id=2) */}}
{{/* anchor(id=2) */}}
```

#### Katex

　数式も記述する場合が有ると思います. Zolaのテーマの１つで有る[even](https://www.getzola.org/themes/even/)に対応していますのでそれを参考にしたいと思います. evenテーマではまずkatexの有効無効をコントロールできます.

```toml
[extra]
katex_enable = true
katex_auto_render = true
```

　後はテンプレートのヘッダーでkatexのJSファイルやCSSをインストールすればいけます. この記事を書いている時点でのKatexのバージョンは0.10.2です.

```jinja2
{% block js %}
      {% if config.extra.katex_enable %}
      <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/katex.min.css" integrity="sha384-9eLZqc9ds8eNjO3TmqPeYcDj8n+Qfa4nuSiGYa6DjLNcv9BtN69ZIulL9+8CqC9Y" crossorigin="anonymous">

      <script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/katex.min.js" integrity="sha384-K3vbOmF2BtaVai+Qk37uypf7VrgBubhQreNQe9aGsz9lB63dIFiQVlJbr92dw2Lx" crossorigin="anonymous"></script>
      <script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/contrib/mathtex-script-type.min.js" integrity="sha384-zWYbd0NBwgTsgIdFKVprSfTh1mbMPe5Hz1X3yY4Sd1h/K1cQoUe36OGwAGz/PcDy" crossorigin="anonymous"></script>
          {% if config.extra.katex_auto_render %}
          <script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/contrib/auto-render.min.js" integrity="sha384-kmZOZB5ObwgQnS/DuDg6TScgOiWWBiVt0plIRkZCmE6rDZGrEOQeHM5PcHi+nyqe" crossorigin="anonymous"
              onload="renderMathInElement(document.body);"></script>
          {% endif %}
      {% endif %}
{% endblock js %}
```

#### katexを利用する.

　Katexを埋め込むには以下のようなショートコードを使います.{{ ref(id=9)}}

```jinja2
<script type="math/tex{% if block %};mode=display{% endif %}">{{body}}</script>
```

 　ここで注意点はalignedやmatrixなどで複数業の数式を記述する場合です. Katexではこれらの記法はサポートされていますが, この場合&や改行のバックスラッシュなどが使えません. その場合はTeraのフィルタのsafeを使います.

```jinja2
<script type="math/tex{% if block %};mode=display{% endif %}">{{body | safe}}</script>
```

　Markdownでは以下のように書きます.

```jinja2
<!-- inline -->
{{/* katex(body="\KaTeX") */}}
<!-- multiline-->
{%/* katex(block=true) */%}
  \KaTeX \\
  \KateX
{%/* end */%}
```

　インライン要素では{{ katex(body="\KaTeX") }}という表示になります. 一方のブロック要素は, 

{% katex(block=true) %}
\begin{aligned}
  \KaTeX
\end{aligned}
{% end %} 

### auto-render extension

 auto-render拡張を有効にすると, 以下のような記法も使えるようです.

```md
\\(\frac{a}{b} \\)
\\[ \frac{a}{b} \\]
```

## Githubページへのデプロイ

## 注釈
{{ anchor(id=1)}} [Summary](https://www.getzola.org/documentation/content/page/#summary)
{{ anchor(id=2) }} [Built-ins](https://tera.netlify.com/docs/templates/#built-ins)
{{ anchor(id=3) }} [Askama](https://github.com/djc/askama)などRust製の[テンプレート・エンジン](https://crates.io/categories/template-engine)はいくつかあります.
{{ anchor(id=4) }} [Built-in filters](https://www.getzola.org/documentation/templates/overview/#built-in-filters)
{{ anchor(id=5) }} [Built-in global functions](https://www.getzola.org/documentation/templates/overview/#built-in-global-functions)
{{ anchor(id=6) }} [Static assets](https://www.getzola.org/documentation/content/overview/#static-assets)

{{ anchor(id=7) }} [Static assets](https://www.getzola.org/documentation/content/overview/#static-assets)
{{ anchor(id=8) }} [Syntax Highlighting](https://www.getzola.org/documentation/content/syntax-highlighting/)
{{ anchor(id=9) }} [math/tex Custom Script Type Extension](https://github.com/KaTeX/KaTeX/tree/master/contrib/mathtex-script-type)
{{ anchor(id=10) }} [Latex, Katex support #26](https://github.com/getzola/even/issues/26)

## Reference
1. [Zola](https://www.getzola.org/)
2. [Tera](https://tera.netlify.com/)
