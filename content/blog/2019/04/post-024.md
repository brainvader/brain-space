+++
title = 'Effective C# for Unity (1)'
description =  'Effective C#の読書メモ'
date = 2019-04-28T17:41:39Z

[taxonomies]
categories = ["programming"]
tags = ["C#"]
+++

# Effective C# for Unity (1)

## 第１章 C#言語イディオム

### 項目1ローカル変数の型をなるべく暗黙的に指定すること

　要はvarを使いましょうということらしい. varを行っても型推論自体はコンパイル時に行われるようになるだけで型付けの意義はなくならないそうです. 他の動的言語のように実行時に決定されるようになるわけではないということです.

　しかし内部的にはそうでもコードの可読性が低下するという懸念があります. また組み込みの数値型ローカル変数にvarを用いた場合にキャスティングが起きることが問題になります. これは暗黙的な型変換なのでエラーにはなりません. 勝手に精度が変わるという事態を招きます. この場合は全ての型を明示的に宣言すればコンパイラが警告を出してくれるようです. 一方, コンパイラに任せたいいケースもあって, 場合によってはパフォーマンスに影響を及ぼすこともあるようです.

+ 分かりやすい変数名をつける.
+ どうしても分かりにくい場合は明示的な型宣言を行う.
+ 数値型の場合は明示的に宣言した方が良い.
+ その他の型はvarを使ってコンパイラに任せる.

　著者は基準として,

> 開発者が初期化式から意味論的情報を明確に得られないようであればvarを使うべきではなく、型を明記してその情報を伝えるようにすべきです。

　を挙げています.

### constよりもreadonlyを使用する

　C#には２つの定数がある.

+ コンパイル時定数 (const)
+ 実行時定数 (readonly)

　実行時定数は柔軟性があるが, コンパイル時定数は速度的に若干有利とのことです. 後者はコンパイル時に置き換えられるのに対して, 前者は実行時評価を行う分オーバーヘッドがあるからのようです. ただしコンパイル時定数は

+ 数値
+ 文字列
+ null

　に限定されます. 実行時定数は全ての型を定数にすることができます. コンパイル時定数は仕様変更によって変わらないようなものに使うべきとのことです. パフォーマンスの向上は足したことがないらしいのでちゃんと計測した方がいいようです.<a href="ref-1"><sup>1</sup></a>

　以下のようなケースにコンパイル時定数を使うと良いらしいです.

+ 属性の引数<a href="ref-2"><sup>2</sup></a>
+ switch-caseのラベル
+ enum
+ リリースを跨いでも不変な値

 ゲームとかならPIとか重力定数はこれに該当しそうです.

 ### キャストにはisまたはasを使用すること

　その前に予備知識としてC#には値型と参照型がありますが, 値型も参照型のように振る舞うようです.<a href="ref-3"><sup>3</sup></a>

```Csharp
int x = 5;
object y = x;
int z = (int)y;
```

　object型というのは大雑把に言うとC#のルート・オブジェクトです.<a href="ref-4"><sup>4</sup></a>

> object 型は .NET での Object の別名です。 C# の統一型システムでは、すべての型 (定義済み、ユーザー定義、参照型、および値型) が、直接または間接的に Object を継承します。

　項目1で値型は明示的に型を宣言するとあるので, 自分で書く場合はそうすればいいんだと思います. でもフレームワークなどを使うとobject型を使わざる得ない場合があるようです. この項目の問題意識は型の不一致にどう対応するか?, コンパイラはどう言う動きをするか?です. 結論としてはasを使おうということのようです.

> むやみにキャストをするよりもas演算子の方が安全で、かつ実行時の効率も優れるため、as演算子が使用できる場合には常にasを使用するというのが正しい選択です。

　asやisが行うのは以下の2点です.

+ 実行時の型チェック
+ ボックス化

　 ボックス化は値型を参照型で振る舞うようにする処理のことです. 当然ですか型の変換は継承関係の縛りがあります. 全然関係ない型への変換はできません. ポリモーフィックな型のみに適用できます. 例外はasはボックス化された値をアンボックスする際にnull許容型へと変換するする時です. null許容型とはOptionalとかのことです.

```Csharp
object o = Factory.GetObject();
var i = o as int?;
```

　数値型はnullにはなりませんが, オプショナルとして変換することでnullを取れます. つまり参照先がnullならnullを返すのでnullチェックしましょうということです.

```Csharp
// o is an object type
object o = Factory.GetObject();
MyType t = o as MyType;
if (t != null) { /* do something with t */ }
```

　ではキャストは万能かというとそうでもありません. コンパイル時に決まる型の制約を受けます. 

> as演算子とキャストの一番の違いは、ユーザー定義の変換の扱いにあります。

　このユーザー定義の変換はコンパイル時の型に関してのみ作用します. oをキャストしてみましょう.

```Csharp
object o = Factory.GetObject();
MyType t;
t = (MyType)o
```

　コンパイラからはoはobject型に見えます. この場合object型からMyTypeへの変換演算子が定義されていないとコンパイルは変換の仕方がわかりません. 静的に決定できない場合コンパイラは型を確かめるために実行時に型をチェックするコードを生成します. このコードはoがMyTypeかどうかをチェックします. oがMyTypeに変換可能かをチェックするコードではないのです.

　キャストは例外処理が必要というのも避けるための理由のようです.

> 事前に適切なチェックを行うことで回避できるのであれば、例外のキャッチをなるべく避けるべきという教訓もあります。

　asならnullチェックですみます. ユーザー定義演算子がある型をasで変換した場合はコンパイル・エラーが出るのもasが使った方がいい理由のようです. 両者の違いは大雑把に言うとこんな感じでしょうか.

+ asは継承関係から変換を実行する.
+ キャストは静的に決まる型で型変換演算子を実行する. 静的に決められない場合は動的な型チェックはするが型変換演算子は使わない.

　なお継承関係にはないが, 型変換演算子が定義されている場合はasだとエラーになるそうです. 最後は型チェックです.

+ isはポリモーフィズムな型情報
+ GetType()は厳密な型情報

　isの場合は親クラスなどにも該当しますが, GetTypeは呼び出したがインスタンスの型が戻り値になります.

　Enumerable.Cast<T>は古いタイプのキャストをやるそうですがよくわかりませんでした.

### 項目4 string.Format()を補間文字列に置き換える

　Formatメソッドの問題は実行してチェックしないとわからない点だそうです. 置換される文字列との対応関係も見にくいです. 一方補完文字列は可読性も高いです.

```Csharp
Console.WriteLine($"円周率は{Math.PI}");
```

　問題点はSQLとかでクエリ・パラメータは展開されて文字列になってしまうので丸見えになるのでやめましょうってことです.

### 項目5 カルチャ固有の文字列よりもFormattableStringを使用すること

　国際化の話のようです.

### 項目6 文字列指定のAPIを使用しないこと

　nameof演算子について延々と.

### 項目7 デリゲートを使用してコールバックを表現する

　タイトル通りの内容です.

### 項目8 イベントの呼び出し時にnull条件演算子を使用すること

　イベント・ハンドラをスレッド・セーフにするためにnull条件演算子を使うと良いよと言う話のようです. null演算子は式がnullでない場合のみInvokeを実行します.

```Csharp
public class EventSource {
  private EventHandler<int> Updated;

  public void RaiseUpdates() {
    counter++;
    Updated?.Invoke(this, counter);
  }

  private int counter;
}

```

### 項目9 ボックス化およびボックス化解除を最小限に抑える

　ボックス化とアンボックス化はオーバーヘッドがあると言う話です. 値型をSystem.object型として扱う場合に生じる問題のようです. 

+ コレクション中に値型を格納する
+ System.Objectに定義されたメソッドを呼び出す
+ System.Objectにキャストする

　のようなケースに気をつける必要があるようです. 特にString型やCollectionの場合には注意が必要そうです. String型の場合はtoStringを呼び出すとボクシングがおきます. 補間文字列を使いましょう. コレクションは要素を取り出したつもりがコピーをもらったと言うことがおきます.

```Csharp
var copied = some_collection[0];
```

　対処法としては不変な値型を使おうということです.<a href="ref-5"><sup>5</sup></a> オブジェクトの内部状態(値型)を変更する場合は, 新規にオブジェクトを生成するわけです.

### 項目10 親クラスの変更に応じる場合のみnew修飾子を使用すること

　C#ではnew修飾子と言うのをメソッドにつけることができます.

```Csharp
public new void AMethog() { /* ... */ }
```

　こうすると親クラスと違う実装を定義できます. これは仮想関数のoverrideと似た機能です. 通常はこちらをつかうべきです.

+ 仮想関数は動的に最適なメソッドを呼び出す.
+ new修飾子は静的に結び付けられる.

　基本はやはり仮想関数をつかうべきのようです. new修飾子を使うのは

> 派生クラスですでに使用済みのメソッド名が、新しいバージョンの親クラスに定義されたメンバと競合した場合にはnew修飾子を指定します。

　例えばライブラリでもフレームワークでも必要な機能がないので継承して付け足しました. ところが更新版でその機能が取り込まれたような場合です. 機能が同じならいいですが違う場合もありえます. もちろん自作のメソッド名を変更しても良いですが, この影響は広範囲に及ぶ可能性があります. new演算子なら一箇所の修正ですみます. 当然前者の方が基本的には良いプラクティスですが, そうもいかない場合はnewが使えると言う話です.

## 第2章 リソース管理

### 項目11.NETのリソース管理を理解する

　C#に置けるメモリ管理とGC(Garbage Collection)についてです. 基本的にはオブジェクトの管理は.NETフレームワークが全てを行ってくれます. マネージドヒープと呼ばれるメモリ領域で参照関係を管理し, 参照がなくなくなればガーベッジとみなされ削除されます. この際コンパクションといってメモリ領域が圧縮されます.

 非マネージ・リソースは開発者の責任で解放する必要があります. 解放する処理をそのリースを管理するクラスに実装するということです. そのためのメカニズムは二つあります.

 + ファイナライザ
 + IDisposableインターフェース

　ファイナライザはカーベージかどうかの判断はGCが行います. そしてファイナライザが不特定の時点(いつかは分からない)で執行されるようです. ファイナライザはC++のデストラクタと違ってタイミングをコントロールすることはできません. C++なら関数のスコープを使えば, ローカル変数にインスタンスを保持して関数から抜ける時にデストラクタが呼び出すことができます.

　通常このサイクルはGCのサイクルよりも遅れます.

![CGとファイナライザのサイクルの違い](../../img/C#_001.png)

　この遅れは単なる1サイクルではなく世代というGCサイクルを何回生き残ったかという重みで処理を最適化しています. 遅れるだけ長寿なオブジェクトになりGCの実行も後ずれしていきます. この辺はC++のようなプログラミングに慣れている人への注意だと思います.<a href="ref-6"><sup>6</sup></a>

　もうファイナライズの処理の流れをまとめておきましょう.<a href="ref-7"><sup>7</sup></a><a href="ref-8"><sup>8</sup></a>

1. ファイナライザが定義されたオブジェクト(F)をファイナライズ・リストへ登録する.
2. FがGCにガーベッジと判断されたらF-reachableキューへ突っ込む.
3. ファイナライザ専用スレッドでファイナライザを実行する.
4. GCがキューからオブジェクトを取り出す.

　2でガーベッジと判断しているにも関わらず, ここではコンパクションの対象にはならないわけです. ファイナライザは保険のようなものでどこかの時点では必ず解放してほしいようなリソースに対して行うと良いようです.

### 項目12 メンバには割り当て演算子よりもオブジェクト初期化子を使用すること

　要はメンバ面数は定義と同時に初期化しておけと言うことのようです. こうするとコンパイラがコンストラクの冒頭で定義した順番に挿入してくれるようです. 特に複数のコンストラクタが定義されている場合に有効です. 

　不要な場合は以下の三つです.

1. 0かnullで初期化する場合
2. 複数のコンストラクタで共通の初期化処理でない場合
3. 単純な初期化出ない場合(例外処理が絡むような場合)

 1の場合は単純にコンパイラがデフォルトでやってくれるからです. 問題はこの場合でも0初期化やnull初期化のコードをコンパイラは生成するしてしまうことにあります. 2で個別の初期化処理をどこかのコンストラクタに書いてもコンパイラはオブジェクト初期化子を挿入します. 3は自明と思います. コンストラクタに移しましょう.

### 項目13 staticメンバを適切に初期化すること

+ staticコンストラクタ
+ オブジェクト初期化子
+ Lazy<T>

　オブジェクト初期化子よりもstaticコンストラクタの方が良いそうです. 例外の処理が必要な場合, 特に問題になります. 静的メンバは全てのインスタンスで共有されるので初期化に失敗した場合全てのインスタンスが失敗します. コストが高い初期はLazy<T>を使うと良いそうです.

### 項目14 初期化ロジックの重複を最小化する

+ オーバーロード
+ デフォルト引数
+ プライベート・ヘルパー・メソッド

　初期化における共通のロジックはコンストラクタ初期化子を使うと良いらしいです. コピペとかプライベートなヘルパー・メソッドを使うのもだめだそうです. 理由はコンパイラによる最適化が受けられないからです. 開発者の認識では共通処理ですがコンパイラからしたらプライベート・メソッドの呼び出しです. コンストラクタなら共通した処理は省略されます. thisかbaseで共通処理用のコンストラクタを呼び出しましょう. もう一つ重大な欠陥はコンストラクタ外でreadonlyプロパティにアクセスできないことです!

　引数の組み合わせに対してはオーバーロードを使う場合もありますが, これは引数の組み合わせとともにコンストラクタが増えてしまいます. C#4.0で導入されたデフォルト引数を使うと解決できます. しかし引数名を指定する呼び出しの場合, APIに変更があった場合この引数名の変更も伴うことになります.

```Csharp
var size = new Size(width: 100, height:100);
```

　またジェネリック・クラスではパブリックで引数なしのコンストラクタが明示的に定義しておく必要があります. これをnew制約と言います.<a href="ref-9"><sup>9</sup></a> new()のように引数なしで呼び出せるコンストラクタのことです. なぜ一見関係なさそうなnew制約がここで出てきたかと言うと. 全てデフォルト引数が設定されている場合も全ての引数を省略できるからです. なぜならジェネリックの場合T型の引数のデフォルト値は設定できないからです.

　以下は掲載されていた初期化時の流れです. 2個目からのインスタンスの場合は5以下が実行されます.

1. static変数のメモリストレージが0に初期化される手順
2. static変数の初期化子が実行される手順
3. 親クラスのstaticコンストラクタが実行される手順
4. staticコンストラクタが実行される手順
5. インスタンス変数のメモリストレージが0に初期化される手順
6. インスタンス変数の初期化子が実行される手順
7. 適切な親クラスのコンストラクタが実行される手順
8. インスタンスコンストラクタが実行される

> 開発者が行うべきことは、すべての変数が意図した通りの値に初期化され、かつ初期化処理が1回だけ行われるようなコードを作成することです。

### 項目15 不必要なオブジェクトの生成を避けること

> ガベージコレクタにできるだけ仕事をさせないようにすべきです。

　ポイントは3点です.

1. 頻繁に呼び出されるメソッドでのインスタンス生成
2. Flyweightパターンの適用
3. 不変型へのBuilderパターンの適用

　例えばドローコールなどの頻繁に呼び出されるメソッドでインスタンス生成を行うべきではないでしょう. UnityのUpdateで毎回オブジェクトを作り直すのは非効率です. どこかにキャッシュして使い回しましょう. Updateの場合MonoBehaviorを継承したクラスのメンバ変数として保持しておけば良いでしょう.

　2つ目のBrushesクラスの例が上がっています. これも様々なところで描画用に使われるブラシを表すクラスです. 内部的には要求された色をlazyに生成するようです. このような広くアプリで共有されるインスタンスはstaticとして定義すると良いようです(可読性とかメンテナンスの関係で問題がありそうな気もしますが).

　最後の例はString型のような不変型の場合です. 

```Csharp
string msg = "Hello, ";
msg += thisUser.name;
msg += ". Today is ";
msg += System.DateTime.Now.ToString();
```

　加算代入演算子の前後に実際にはコピー操作をするコードが生成されます. ここでのコードの意図は文字列の構築なのでBuilderクラスを作りましょう.

### 項目16 コンストラクタ内では仮想メソッドを呼ばないこと

　アンチパターンといっても良いようで, とりあえず呼ばない方が良いようです.

### 項目17 標準的なDisposeパターンを実装する

+ マネージ・リソース
+ 非マネージ・リソース

　IDisposableインターフェースは

> アンマネージリソースを解放するためのメカニズムを提供します。<a href="ref-10"><sup>10</sup></a>

　しかし非マネージ・リソースが何を指すのかよくわかりません. これはネット上でも議論になったようです. ある記事<a href="ref-11"><sup>11</sup></a>では以下のように結論づけられていました.

| double[], string等 | マネージ・リソース |
| Stream, Bitmap等 | マネージ・リソース |
| (外部リソースの)ポインター等 | マネージ・リソース |
| ポインターが指す外部のリソース | 非マネージ・リソース |

　[CLRから見たリソースについて](https://blogs.msdn.microsoft.com/shozoa/2010/09/07/clr/)では以下のように分類しています.

> ..., シンプルに「GCの対象になるかどうか」ということです。

　しかし具体例がわかりません. 大雑把には.NET Frameworkの外のオブジェクトらしいのです. [What exactly are unmanaged resources?](https://stackoverflow.com/questions/3433197/what-exactly-are-unmanaged-resources/3433209)によると,

+ Open files
+ Open network connectins
+ Unmagend memory
+ In XNA: vertex buffers, index buffers, textures, etc.

　問題はこれらをラップしたクラスはマネージクラスになるようです. 公式では,

> The most common types of unmanaged resource are objects that wrap operating system resources, such as files, windows, network connections, or database connections.<a href="ref-12"><sup>12</sup></a>

　これらがメモリに読み込まれると管理しているオブジェクトはCommon Language Runtim(CLR)から管理されますが読み込まれた内容はヒープ領域に残りそうです.


## 第3章 ジェネリックによる処理

### 項目18 最低限必須となる制約を常に定義すること

　型制約がない場合多くのチェックが必要になります.　例としてIComparable<T>やIEquatable<T>制約を課した場合を紹介しています. 問題は制約は多すぎても少なすぎてもいけないということです.

> 最も一般的なものは、制約なしでも利用可能な機能であれば、それを制約とはしない方法です。

### 項目19 実行時の型チェックを使用してジェネリック・アルゴリズムを特化する

　何を訴えているのかよく分からなかったが, ジェネリック型で定義したとしても内部的に実行時に型をチェックして特化したアルゴリズムで処理するとパフォーマンスが改善するとこうこのようです. つまり型ごとにパフォーマンスチューニングを施すという話のようです. 互換性のあるインターフェースではジェネリックでも動きますがそれぞれに内部で使われるアルゴリズムが違うのでその辺を理解する必要もありそうでなかなか高度なトピックかなと思います.

### 項目20 IComparable<T>とIComparer<T>により順序関係を実装する

　クラスや構造体について順序関係を比較可能にしたいときは, .NET Frameworkが提供するインターフェスであるIComparable<T>やIComparer<T>を使おうということのようです. オレオレインターフェースを定義するなよってことかもしれません. ソート可能なオブジェクトを作りたいなと思ったらこんなのがあったと思い出しましょう.

### 項目21 破棄可能な型引数をサポートするようにジェネリック型を作成すること

　型に制約を貸すことのメリットは以下のように説明されています.

+ 制約をしていすることによって, 実行時エラーをコンパイル時エラーとすることができる
+ クラスの使用者に対してドキュメントを提供できる

　当然型引数できないことを制約にすることはできません. 問題はIDisposableを実装する型引数の場合に起こるそうです. つまり型引数がIDisposableを実装する場合はきちんと確認する必要があるのです.

> ジェネリッククラスの型引数に対するインスタンスを生成する場合、その型がIDisposableを実装するかどうか必ず確認するようにしてください。

　そしてそれが実装されていたら呼び出されるように実装する必要があります. 具体的には色々な方法が紹介されていました. usingブロックを使う簡単にチェックできるようです.

```Csharp
public interface IEngin {
    void DoWork();
}

public class EngineDriverOne<T> where T: IEngin, new() {
    public void GetThingsDone() {
        T driver = new T();
        // The compiler generates a hiddne local variable that stores a casted driver as IDisposable
        // The variable is null when it is not implemented IDisposable
        using(driver as IDisposable) {
            driver.DoWork();
        }

    }
}
```

　ではローカル変数ではなくて, メンバ変数の場合はどうすればいいでしょうか? 遅延初期化されるのでIsValueCreatedのような余計な処理が入っていますが, null合成演算子を使っています.

```Csharp
public class EngineDriverOne<T>: IDisposable where T: IEngin, new(){
    private Lazy<T> driver = new Lazy<T>(() => new T());
    public void GetThingsDone() => driver.Value.DoWork();
    public void Dispose() {
      if(driver.IsValueCreated) {
            var resource = driver.Value as IDisposable;
            // Call Dispose resource is not null!
            resource?.Dispose();
       }
    }
}
```

　そもそもdriverはEnginDriverOneが生成や消滅の管理をしなくてはいけないのかと言う視点も大事です. 以下はいわゆるDI(Dependency Injection)パターンです.

```csharp
public sealed class EnginDriverOne<T> where T: IEngin {
    private T driver;

    public EnginDriverOne(T driver) {
        this.driver = driver;
    }

    public void GetThingsDone() {
        driver.DoWork();
    }
}
```

### 項目22 ジェネリックの共変性と反変性をサポートする

　ジェネリック型の互換性の問題と考えるとわかりやすい項でした. 考えている状況はコード上で指定した型とランタイムの型が違う場合で, 両者が継承関係にある場合です.

+ 不変性 (Invariant)
+ 共変性 (Covariance)
+ 反変性 (Contravariance)

　A -> B -> Cの順で継承を表すとします. この場合戻り値がB型ならA型はダメですがC型なら返せます. こう言うのを共変戻り値と言うそうです. 一方, A型の引数の場合, B型は大丈夫ですが, C型の引数の場合B型は取れません. このような場合を反変引数と呼ぶようです.<a href="ref-13"><sup>13</sup></a> 前者はB(指定した型) -> C(実行時の型), 後者はB(実行時の型) -> A (指定した型)とコードの型と実際の型が変わっています.

　されこれをジェネリック型に拡張しましょう. 

+ X -> Yの場合にC<X> -> C<Y>ならC<T>は共変
+ Y -> Xの場合にC<X> -> C<Y>ならC<T>は反変

　こうした共変・反変をC#4.0以降ではin/out修飾子で制御できるようになったそうです. outは出力(戻り値)に対する型に, inは入力(引数)に対する型に使われます. 例を見てみましょう.<a href="ref-14"><sup>14</sup></a>

```Csharp
public interface IEnumerator<out T>
{
  T Current { get; } // get しかない ＝ 出力のみ
  bool MoveNext();
  void Reset();
}

IEnumerator<string> strEnum = new Enumerator<string>(); // (1)
IEnumerator<object> objEnum = strEnum; // (2)
// (3) ここでstring型 -> object型が成り立つ.
```

　 (1)でTはstring型になりますが, これを(2)でobject型に変換しています(IEnumerator<string> -> IEnumerator<object>). なのでCurrentの戻り値はstring型 -> object型ですが(3)より問題ないです. つまり

string型 -> object型 => IEnumerator<string> -> IEnumerator<object>

　が成立します. 上の定義より共変戻り値が成り立ちます. 今度は入力を考えましょう.

```Csharp
public interface IComparer<in T>
{
  int Compare(T a, T b); // T は引数としてしか使われない
}

IComparer<object> objComp = new Comparer<object>(); // (1)
IComparer<string> strComp = objComp; // (2)
// (3) ここでstring型 -> object型が成り立つ.
```

　(1)ではobject型です. それを(2)で代入していることからIConparer<object> -> IComparer<string>です. 例えばstrComp.Compare("aaa", "bbb")となりますが, 実際のCompareの引数の型はobject型です. (3)よりこれは成り立ちます. 

string型 -> object型 => IComparer<object> -> IComparer<string>

　より反変性が成り立ちます. 具体的なケースを考えて型の互換性をみてやる必要があると言うことのようです. 繰り返しになりますがout修飾子がつく例としては

```csharp
public interfaceIEnumerable<out T>: IEnumerable {
  IEnumerator<T>GetEnumerator();
}

public interfaceIEnumerator<out T>: IDisposable, IEnumerator {
  TCurrent { get; } //MoveNext()とReset()はIEnumeratorから継承
}
```

 B -> Aの時IEnumerator<B> -> IEnumerator<A>が成り立つかです.

```csharp
public static void CovarianceOutputs(IEnumerable<A> baseItems){
  foreach (var i in baseItems)
　Debug.Log(i)
}
```

  この場合普通にIEnumerable<B>を渡すと思います. そしてIEnumerable<B>はGetEnumeratorでIEnumerator<B>を返します. B -> Aの時IEnumerator<B> -> IEnumerator<A>が成り立つので共変戻り値の場合です. 今度はin修飾子が付く例としてIComparable<T>を考えて見ましょう.

```csharp
public interface IComparable<in T> {
  int CompareTo(T other);
}
```

　B -> Aのとき, IComparable<A> -> IComparable<B>が成り立つでしょうか? IComparable<A>なら

```csharp
int CompareTo(A other) { }
```

　となるはずです. ここにIComparable<B>が実装されたインスタンスは取れるかといことです. Aはベースクラスですのでポリモーフィックな型BやB'などのサブクラスが実行時に渡されても問題ありません. これが反変引数です. 

　デリゲートが引数として指定された場合は少し厄介です. out付き型は戻り値の型で, in付き型は引数の型として利用されます. つまりクラス中で以下のように使われます.

```csharp
// in T
void do(T_in item) { /* do something */ }
```

　ここに引数としてデリゲートを付け加えます.

```csharp
// class Example<in T_in, out T_out>
T_in a; // member variable
private T_out innerDo() { /*do something */ } // priavete method
void do(T_in item, Func<T_out, T_in> f) { /* do something */ }
```

　doの内部で以下のような処理がされるとします.

```csharp
a = f(innerDo())
```

 innerDoはT_out型ですし, aはT_in型です. ジェネリック型のクラスの前提に従うようにデリゲートの型引数の名前は決まるわけです.

### 項目23 型パラメータにおけるメソッドの制約をデリゲートとして定義する

  まず制約として指定でいないものがあります.

+ staticメソッド
+ 演算子のオーバーロード
+ 他のコンストラクタ

　こんな時はデリゲートを制約として使おうと言う話のようです.

> 通常の制約として指定できないような動作に依存するジェネリック型を定義できるようになります。

　インターフェースで良いのではと言う疑問に対しては以下のような利点を述べています.

> T型のオブジェクトを作成するインターフェイスIFactory<T>を引数で定義できます。IAdd<T>を定義すれば、Tに定義されたstatic演算子「"+"」（あるいは他のメソッド）を使用してオブジェクトに追加する機能を実装できます。しかしこれは十分な解決方法だとは言えません。多数の作業を追加しなければならず、元々の設計がぼやけたものになってしまいます。

> しかし、特定のジェネリッククラスやジェネリックメソッドでしか使用されないような制約をインターフェイスとして独自に作成しようとしているのであれば、デリゲートによるメソッド制約として作成した方が使用する側の手間も省くことができるでしょう

　実際の手間はこんな感じです.<a href="ref-15"><sup>15</sup></a>

> IAdd<T>を実装するクラスを作成し、IAdd<T>に必要なメソッドを定義し、ジェネリック型に対応するクローズジェネリッククラスを用意します。

　これをデリゲートにしてやれば目的は達成できます.

> しかし、ここまでする必要はありません。ジェネリッククラスで呼び出したいメソッドに一致するシグネチャを持ったデリゲートを定義すればよいのです。 

　　デリゲート使うと中身が骨抜きになると思っていましたが, インターフェースを使うよりもこうした場合には良いようです. 具体例としてZipメソッドを追加してみましょう. Zipメソッド自体はすでに存在しています.<a href="ref-16"><sup>16</sup></a>

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;

// Extension Method
class Extension {
    public static IEnumerable<TOutput> Zip<T1, T2, TOutput>
    (IEnumerable<T1> left, IEnumerable<T2> right, Func<T1, T2, TOutput> generator)
    {
        IEnumerator<T1> leftSeq = left.GetEnumerator();
        IEnumerator<T2> rightSeq = right.GetEnumerator();

        while (leftSeq.MoveNext() && rightSeq.MoveNext())
        {
            yield return generator(leftSeq.Current, rightSeq.Current);
        }
        leftSeq.Dispose();
        rightSeq.Dispose();
    }
}


public class Point {
    public double X { get; }
    public double Y { get; }

    public Point(double x, double y) {
        this.X = x;
        this.Y = y;
    }
}

public class ZipSequence : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        double[] xValues = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
        double[] yValues = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };

        List<Point> values = new List<Point>(
            System.Linq.Enumerable.Zip(
                xValues, yValues, (x, y) => new Point(x, y)
            )
        );

        foreach (var p in values)
            Debug.Log($"(x, y): ({p.X}, {p.Y})");
    }
}
```

### 項目24 親クラスやインターフェイス用に特化したジェネリックメソッドを作成しないこと

 特化したジェネリック・メソッドは自己矛盾を抱えてそうな言葉です.

> ジェネリックメソッドはあらゆるものに一致するため、親クラス用のメソッドよりも優先されるのです。

　この場合は親よりも汎化されて適用されているということです. 簡素な親子関係ならいいでしょうが, 複雑になると親子関係ををぶっ壊すことになります. 複雑になってくると必要なメソッドが呼べないとか呼ぶためには明示的に型変換が必要とかややこしい運用が求められるようになります. またその解消法として動的に型をチェックするのはあまり賢いやり方ではないようです. 動的な型のチェックは19項目で説明したような使い方もあるので必ずしも悪いわけではないです. ただ便利だからとジェネリック・メソッドを導入して, 複雑さを動的型チェックで解決するのはやめましょうということです. コンパイラが何をするのか理解した上で, 導入の判断をしましょうということです.

### 項目25 型引数がインスタンスのフィールドではない場合にはジェネリック・メソッドとして定義すること

　項目24の続きです. 型引数がインスタンスのフィールドというのはメンバ変数の型としてジェネリック型が必要かどうかということです. 必要なければジェネリック・メソッドを使おうといことです. クラス自体は非ジェネリック・クラスとしておいてメソッドだけをジェネリック・メソッドに置き換えるという手法です. ジェネリック・メソッドは型ごとに新しいインスタンスを生成することはないです. また使う側も指定する必要はなないです. ジェネリック・クラスの場合クラスの型を決める, インスタンスを生成する, そのインスタンスのメソッドを呼び出すという三段階が必要なのです.

> 特定の型の引数に一致するメソッドが定義されていれば, コンパイラはそのメソッドを呼び出します.

　つまり引数の型が分かっていれば後はコンパイラの仕事ということです. では両者をどうやって使い分ければいいのでしょうか? 以下のような判断基準が示されています.

> 型パラメータの値をクラスの内部状態として保持するかどうか
> ジェネリック・インターフェースを実装する型かどうか

　上は要はメソッドでメンバ変数を使ってるかどうかです. 使ってるならジェネリック・クラスに昇格します. ジェネリック・インターフェースの場合は, インターフェースが強要しているのでできないということでしょうか?.

### 項目26 ジェネリック・インターフェースとともに古いインターフェースを実装する

　.NET Frameworkにはジェネリックの機能が無かったので後方互換性(backword compability)も考慮しましょうと言う話でした.

> 新しいバージョンのインターフェースが優先して呼ばれるようにするには, IComparable<T>とIComparableのように, 非ジェネリックインターフェースを明示的に実装します。

### 項目27 最小限に制限されたインターフェースを拡張メソッドにより機能拡張する

　インターフェースは必要最小限に押さえておいて, 必要なら拡張メソッドで追加するのが良い習慣らしいです. インターフェースに多くの機能を実装すると, 利用する側の負担が増えることが理由らしいです.

> ...使用する側で実装すべきメソッドがすくなくなり, 機能としても充実するでしょう。　

　拡張メソッドはインターフェースの製作者以外の開発者がメソッドを追加できる仕組みらしいです. LinqのメソッドはIEnumerable<T>拡張メソッドらしいです. 項目23で実装したZipのような感じでになります.

 問題は拡張メソッドとクラスで定義したインスタンス・メソッドで間で起切るらしいのですがこの辺の話はよくわかりませんでした.

### 項目28 構築された型に対する拡張メソッドを検討すること

　構築された型が良く分かりませんでしたが, ジェネリックに型を適用したものっぽいです(List<int>とか). こうした型の場合派生クラスを作って機能を拡張するよりも, 拡張メソッドを使う方が柔軟性が高いようです. Linqのクエリとも相性がようようです.

 
　
### Examples

#### EventHandler
+ [A simple c# events and delegates tutorial](http://ikeptwalking.com/simple-c-events-delegates-tutorial/)
+ [How to use C# Events in Unity](https://www.gamasutra.com/blogs/VivekTank/20180703/321126/How_to_use_C_Events_in_Unity.php)

#### IDisposableインターフェース

　IDisposableインターフェースは得体がしれなかったが例を見るとそんなに悩むことでもなかったかなと思った.

　[この例](https://www.codeproject.com/Articles/413887/Understanding-and-Implementing-IDisposable-Interfa)では, StreamReaderがIDisposableインターフェースを実装している. つまりIDisposableを実装しているクラスはアンマネージ・リソースと言うことなのかもしれない. 

```csharp
TextReader tr = null;

try
{
    //lets aquire the resources here                
    tr = new StreamReader(@"Files\test.txt");

    //do some operations using resources
    string s = tr.ReadToEnd();

    Console.WriteLine(s);
}
catch (Exception ex)
{
    //Handle the exception here
}
finally
{
    //lets release the aquired resources here
    if (tr != null)
    {
        tr.Dispose();
    }
}
```

　usingブロックを使うと以下のように明示的なDisposeの呼び出しを省ける(Pythonのwith構文みたいなものか).

```csharp
TextReader tr2 = null;

//lets aquire the resources here                
try
{
    using (tr2 = new StreamReader(@"Files\test.txt"))
    {

        //do some operations using resources
        string s = tr2.ReadToEnd();

        Console.WriteLine(s);
    }
}
catch (Exception ex)
{
    //Handle the exception here
}
```

　trやtr2といったクラスへのポインタはGCで管理されるが, そのポインタが指すファイルの内容は解放されないのだと思う. そしてこうしたアンマネージ・リソースをメンバ変数とかに持つならそのクラスもIDisposableインターフェースを実装しましょうってことらしい. [もう１つ例](https://qiita.com/momotaro98/items/c4fe0fff0c173e879f2d)は[DbConnectionクラス](https://docs.microsoft.com/en-us/dotnet/api/system.data.common.dbconnection?view=netframework-4.8)で, IDisposableが実装されていると書かれている.

 後は自作クラスがこうしたアンマネージ・リソースを管理する場合だがこの書き方は公式に[Examples](https://docs.microsoft.com/en-us/dotnet/api/system.idisposable?view=netframework-4.8#examples)として乗っているので書く必要があるなら参照しよう.

  結論としてはドキュメント読めってっこった.


+ [イマイチ分かりにくいIDisposableの実装方法をまとめる。](https://clickan.click/idisposable/)
　
## 注釈
1. <a name="ref-1"></a> [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet)というのを勧めています.  
2. <a name="ref-2"></a> \[\]で括ったやつです.  
3. <a name="ref-3"></a> [object (C# リファレンス)](https://docs.microsoft.com/ja-jp/dotnet/csharp/language-reference/keywords/object)  
4. <a name="ref-4"></a> [値型と参照型とミュータブルとイミュータブルと](https://qiita.com/chocolamint/items/020143cf249505bf2ffc)  
5. <a name="ref-5"></a>　[ファイナライザには頼らない](https://qiita.com/jkr_2255/items/429648970ee104998d2a)  
6. <a name="ref-6"></a> [C# のメモリ管理](https://ufcpp.net/study/csharp/rm_gc.html)  
7. <a name="ref-7"></a> ファイナライザを理解する  
8. <a name="ref-8"></a> [ただしnew制約により初期化は10倍ぐらい遅いそうです.](https://ufcpp.net/study/csharp/sp2_generics.html#new-constrants)  
9. <a name="ref-9"></a> [IDisposable Interface](https://docs.microsoft.com/ja-jp/dotnet/api/system.idisposable?redirectedfrom=MSDN&view=netframework-4.8)  
<a name="ref-11">11</a> [マネージリソースとアンマネージリソース](http://d.hatena.ne.jp/hilapon/20100904/1283570083)  
<a name="ref-12">12</a> [Cleaning Up Unmanaged Resources](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/unmanaged)  
<a name="ref-13">13</a> [共変戻り値と反変引数](https://qiita.com/7shi/items/39a222b5928bf0b6f745)  
<a name="ref-14">14</a> [ジェネリクスの共変性・反変性](https://ufcpp.net/study/csharp/sp4_variance.html)  
<a name="ref-15">15</a> [C#のジェネリクスでできないこと](https://qiita.com/matarillo/items/25b837a419e2c82e1496)  
<a name="ref-16">16</a> [Enumerable.Zip(IEnumerable<TFirst>, IEnumerable<TSecond>, Func<TFirst,TSecond,TResult>) Method](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.zip?redirectedfrom=MSDN&view=netframework-4.8)  
