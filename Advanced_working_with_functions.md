# Advanced working with functions

## Recursion and stack

最大の再帰の深さは javascript エンジンによって制限されている．
10,000 は確実で，エンジンによってはより多くの深さは可能であるが，100,000 は大多数の制限を超える．
これを緩和する自動最適化("末尾再帰")もある，
再帰的な考え方はコードがシンプルになり，保守が用意になるタスクはたくさんある．

関数の実行に関する情報はその*実行コンテキスト(execution context)*に格納される．
これは関数の実行に関する詳細を含む内部のデータ構造である．
今はどの制御フローであるか，現在の変数，`this`の値，その他いくつかの内部データを持つ．
1 つの関数呼び出しにはそれに関連付けられた実行コンテキストが 1 つだけある．
関数がネスト呼び出しをした場合，次のようなことが起こる．

- 現在の関数が一時停止する．
- 現在の関数に関連付けられている実行コンテキストは，*実行コンテキストスタック*と呼ばれるデータ構造で記録される．
- ネスト呼び出しを実行する．
- それが終わると，スタックから古い実行コンテキストが取り出され，停止したところから外部の関数が再開される．

再帰の深さはスタックのコンテキストの最大数と等しくなる，
メモリの容量に注意する必要がある．ループベースのアルゴリズムはメモリを節約する．

どんな再帰もループで書き直すことが出来る．
通常，ループのバリアントはより効率的にすることが出来る．
しかし，関数が条件によって異なる再帰サブコールを使用してその結果をマージするときや，分岐がより複雑な場合は書き直しが簡単ではないことがある．
再帰はより短いコードで書くことを可能とし，理解や保守をしやすくする．

順序付けられたオブジェクトのリストを保存したいとする．
配列を使う場合，"要素の削除"と"要素の挿入"の操作はコストが高い．
もし本当に速い挿入/削除が必要であれば，`連結リスト(linked list)`と呼ばれる別のデータ構造を選択することができる．
連結リスト要素は次の要素を持つオブジェクトとして，再帰的に定義される．

- `value`
- 次の連結リスト要素または末尾の場合は`null`を参照する`next`プロパティ．

```javascript
let list = {
  value: 1,
  next: {
    value: 2,
    next: {
      value: 3,
      next: null
    }
  }
};
```

配列と違って，大量の番号の再割り当てはなく，要素を簡単に組み替えることが出来る．
主な欠点は，番号では簡単に要素にアクセスできないことである．リストではアイテムの最初から始めて，N 個目の要素を取得するために，N 回 next を行う必要がある．
しかし，常にこのような操作が必要とは限らない．例えば queue や deque が必要なときである．
これらは両端から要素を非常に高速に追加/削除できる順序付けられた構造である．
リストの最後の要素を追跡するのに`tail`という別の変数を追加する価値もある．
要素の集合が大きいと，配列と速度の違いが大きくなる．

## Rest Parameters and spread operator

javascript の主な組み込み関数は任意の数の引数をサポートしている．

- `Math.max(arg1, arg2, ..., argN)`
- `Object.assign(dest, src1, src2, ..., srcN)`

関数はどのように定義されたかに関係なく，任意の数の引数で呼ぶことができる．

```javascript
function sum(a, b) {
  return a, b;
}
alert(sum(1, 2, 3, 4, 5));
```

"必要以上の"引数でもエラーにはならないが，最初の 2 つのみが使われる．
残りのパラメータは，3 つのドットを持つ関数定義で記述することが出来る．

残りのパラメータはすべての残っている引数をまとめるため，最後に書く必要がある．

```javascript
function f(arg1, ...rest, arg2){} //arg2 after ... rest ?!
```

インデックスによってすべての引数を含む`arguments`と言う特別な配列ライクなオブジェクトもある．
`arguments`は引数の数に関係なく，関数に指定されたすべての引数を取得するための唯一の方法である．
デメリットとして，`arguments`は配列ではなく，配列ライクで反復可能という点である．
したがって配列メソッドをサポートしないので`arguments.map(...)`呼び出しをすることができない．
また，すべての引数が常に含まれているので，残りのパラメータを部分的に取り込むことはできない．
したがって，これらの機能が必要な場合には"残りのパラメータ"が好ましい．

```javascript
function showName() {
  alert(arguments.length);
  alert(arguments[0]);
  alert(arguments[1]);
}

// 2, Julius, Caesar
showName("Julius", "Caesar");
// 1, Ilya, undefined
showName("Ilya");
```

アロー関数は`arguments`を持たない．もしアロー関数から`arguments`オブジェクトにアクセスすると，外部の通常の関数からそれらを取得する．

_スプレッド演算子_(Spread operator)は`...`を使い，反復可能なオブジェクト`arr`を引数のリストに展開する．
複数の iterables を渡すこともでき，通常の値とスプレッド演算子を組み合わせることも出来る．
スプレッド演算子は配列をマージするために使うことも出来る．

```javascript
let arr = [3, 5, 1];
let arr2 = [8, 3, -8];

alert(Math.max(3, 5, 1)); // 5
alert(Math.max(arr)); // Nan
alert(Math.max(arr[0], arr[1], arr[2])); // 不格好
alert(Math.max(...arr)); // 5
alert(Math.max(...arr, ...arr2)); // 8
alert(Math.max(1, ...arr, 25, ...arr2)); // 25
```

スプレッド演算子は内部的にはイテレータを使用して要素を集める．
スプレッド演算子は配列ライクでは動作せず，iterables でのみ動作する．
何かを配列に変換するタスクにおいては，`Array.from`がより適している．

## Closure

最初に 2 つの状況を考え内部の仕組みを少しずつ学んでいく．
1 関数`sayHi`は外部変数`name`を使う．関数を実行する時，2 つの値の内どちらが使われるだろう．

```javascript
let name = "John";
function sayHi() {
  alert("Hi, " + name);
}
name = "Pete";
sayHi(); // John もしくは Pete どちらが表示されるだろうか
```

このような状況はブラウザやサーバーサイドでの開発療法で一般的である．
関数が作られた時間よりも後，例えばユーザ操作やネットワークリクエストの後に実行がスケジュールされる場合がある．

2 関数`makeWorker`は別の関数を作り，それを返す．新しい関数は他の場所から呼び出すことが出来る．
作成された場所，呼び出し場所，あるいはその両方からの外部変数にアクセスできるか？

```javascript
function makeWorker() {
  let name = "Pete";

  return function() {
    alert(name);
  };
}

let name = "John";

//関数を作成する．
let work = makeWorker();

work(); // どれが呼び出される？ "Pete" (作成された場所のname) or "John" (呼び出された場所の name)
```

"変数"が技術的に何であるかを考える．
javascript では，すべての実行中の関数やコードブロック，スクリプト全体は*レキシカル環境(Lexical Environment)*と呼ばれる関連オブジェクトを持っている．
レキシカル環境オブジェクトは 2 つの部分から構成されている．

1. 環境レコード(Environment Record)．プロパティとしてすべてのローカル変数を持つオブジェクト．
1. 外部のレキシカル環境への参照する．通常，直近の外部のレキシカルなコードに関連付けられている．
   なので，変数は単に，特別な内部オブジェクト，環境レコードのプロパティである．
   "変数を取得または変更する"とは，"そのオブジェクトのプロパティを取得または変更する"ことを意味する．

以下の簡単なコードはレキシカル環境が 1 つだけある．
`let phrase = "Hello"; -> phrase: "Hello"(Lexical Environment) --Outer--> null`
これはスクリプト全体に関連付けられら所謂グローバルレキシカル環境である，
ブラウザの場合，全ての`<script>`タグは同じグローバル環境を共有する．
グローバルレキシカル環境は外部参照を持っていないので`null`である．

下記は`let`がどのように動作するかである．

1. スクリプト開始時はレキシカル環境は空である．
1. `let`で変数が定義される．今は初期値がないので`undefined`が格納される．
1. 値が代入される．
1. 新しい値を参照する．

要約すると，

- 変数は特別な内部オブジェクトのプロパティで，現在の実行ブロック/関数/スクリプトと関連付けられる．
- 変数を使った作業は実際にはオブジェクトのプロパティを使って作業する．

関数宣言は特別である．`let`変数とは異なり，実行がそこに到達した時に処理されるのではなく，レキシカル環境が作られたときに処理される．グローバルレキシカル環境では，スクリプトが開始される瞬間を意味する．
なので，定義される前に関数宣言を呼び出すことが出来る．
以下のコードでは，レキシカル環境は最初からからではないことを示している．
関数宣言なので`say`を持っており，その後`let`で宣言された`phrase`を取得する．

```javascript
// execution start --- say: function --outer--> null
let phrase = "Hello";

function say(name) {
  alert(`${phrase}, ${name}`);
}
```

呼び出しの中で`say`は外部変数を使う．
関数を実行する時，新しい関数のレキシカル環境が自動的に作られる．
そのレキシカル環境はローカル変数や呼び出しパラメータを格納するために使われる．

関数呼び出しの間，2 つのレキシカル環境がある．内部のものと外部のものである．

- 内部のレキシカル環境は現在の`say`の実行に対応する．1 つの変数`name`を持っており，それは関数の引数である．`say("John")`を呼び出した時，`name`は`John`である，
- 外部のレキシカル環境はグローバルレキシカル環境である．
  内部のレキシカル環境は外部のものへの外部参照を持っている．
  コードが変数にアクセスしたとき，最初に内部のレキシカル環境を探す．その次に外側を探し，チェーンの最後になるまで繰り返す．
  もし，変数がどこにもない場合，strict モードではエラーになる．
  `use strict`が無ければ．未定義変数への代入は下位互換性のために，新しいグローバル変数を作成する．

以上より，関数は外部変数を最新の値として取得する．
古い変数はどこにも保存されず，関数がそれらを必要とする時，自身または外部のレキシカル環境から現在の値を取得するので，最初の答えは`Pete`である．

1 つの呼び出しに 1 つのレキシカル環境である．新しい関数のレキシカル環境は，関数が実行するたびに作られることに注意．
関数が複数回呼び出された場合，各呼び出しには自身のレキシカル環境があり，ローカル変数とその実行に固有のパラメータがある．

レキシカル環境は仕様上のオブジェクトであり，コードからそのオブジェクトを取得したり，直接操作することはできない．
javascript エンジンにはメモリを節約するために使っていない変数を破棄したり，他の内部トリックを行うと言った最適化をする場合があるが，見える振る舞いは上で説明したとおりである．

新しいオブジェクトのプロパティ(外部関数がメソッドを持つオブジェクトを作成する場合)またはその自身の結果として，ネストされた関数を返すことが出来る．
どこにいても同じ外部変数には依然としてアクセスすることが出来る．

```javascript
function makeCounter() {
  let count = 0;
  return function() {
    return count++;
  };
}
let counter = makeCounter();

alert(counter()); // 0
alert(counter()); // 1
alert(counter()); // 2
```

`makeCounter`は各呼び出しで次の数値を返す`counter`関数を作る．これは擬似乱数生成器などでも使われる実践的な方法である．
内部関数が実行されると`count++`の変数は内側から外に検索される．

1. ネストされた関数のローカル変数
1. 外部関数の変数
1. ...さらにグローバルに到達するまで

この例では`count`はステップ 2 で見つかる．外部変数が変更されるとそれが見つかった場所で変更される．したがって，`count++`は外部変数を見つけ，それが属するレキシカル環境内でその値を増やす．

ここで 2 つの疑問がある．

1. `counter`を`makeCounter`に属していないコードからリセットすることは出来るのだろうか？
1. もし`makeCounter`を複数回呼び出した場合，それは`counter`関数を返す．それらは独立しているのか？それとも同じ`count`を共有しているのか？

答えは

1. `counter`はローカル変数の変数であり，外部からアクセスすることができない．なので方法はない．
1. すべての`makeCounter()`の呼び出しで自身の`counter`を持つ新しい環境のレキシカル環境が作られる．したがって`counter`の結果は独立している．

今やクロージャが一般的にどのように動作するかを理解したのでまとめる．

1. スクリプトが開始された直後は，グローバルレキシカル環境だけがある．誕生したすべての関数は作成されたレキシカル環境への参照を持ち，隠しプロパティ`[[Environment]]`を受け取る．このプロパティは関数が作られた場所を知る方法である．開始時点では`makeCounter`関数だけがある．
1. 次に，コードが実行され`makeCounter()`関数の呼び出しが行われる．`makeCounter()`関数の呼び出しの時点でその変数や引数を保持するためにレキシカル環境が作られる．このレキシカル環境は以下の 2 つを保持する．
1. ローカル変数を持つ環境レコード．
1. 外部のレキシカルへの参照
1. `makeCounter`実行中に，小さいネストされた関数が作られる．関数が関数宣言，関数式どちらを使って作られたのかは関係なく，すべての関数は作成されたレキシカル環境を参照する`[[Environment]]`プロパティを取得する．内部関数が作られているだけで，呼び出しはされていないことに注意する．
1. 実行が進み，`makeCounter`の呼び出しが終わると，結果(小さなネストされた関数)がグローバル変数`conuter`に代入される．
1. `counter()`が呼び出されると，"空の"レキシカル環境が作られる．ローカル変数を持っていないが`counter`の`[[Environment]]`はその外部参照として使われるので，それが作られた場所である`makeCounter`の呼び出しの変数にアクセスすることが出来る．
   一般的にレキシカル環境オブジェクトはそれを使用する可能性のある関数が存在する限り存続する．
1. `counter()`の呼び出しは`count`の値を返すだけでなく，その値も増やす．変更はその場で行われる．
1. 次の`counter()`の呼び出しも同じである．

したがって 2 つ目の結果は`"Pete"`である．
もし`makeWoker()`に`let name`が無い場合は結果は`"John"`になる．

`Closure`は外部変数を記憶し，それらにアクセスできる関数である．
いくつかの言語でそれは不可能，もしくはそれを実現するために特別な方法で関数を書く必要がある．
しかし javascript はすべての関数は必然的に自然にクロージャである．(例外はある．)

それらは隠された`[[Environment]]`プロパティを使ってどこに作成されたのかを自動的に覚え，すべてが外部変数にアクセスできる．

面接でフロントエンドの開発者が「クロージャは何ですか」という質問を受けた時，クロージャの定義と javascript においてはすべての関数がクロージャであること，また`Environment`プロパティとレキシカル環境の仕組みと言った技術的に詳細な用語である．

今までは関数に焦点を当てていたが，レキシカル環境はコードブロック`{...}`にも存在する．
コードブロックが実行され，ブロック内のローカル変数を含む時にそれらが作られる．

レキシカル環境オブジェクトは，通常の値と同じメモリ管理ルールの対象である．
レキシカル環境は到達不能になると，メモリから削除される．
`[[Environment]]`参照は外部のレキシカル環境を生かし続ける．

これまで見てきたように，理論的には関数が生きている間，すべての外部変数も保持される．
しかし，実際には javascript エンジンはそれを最適化しようとする．(Real-life optimizations)
変数の使用状況を分析し，外部変数が使用されていないことがわかりやすい場合は削除される．
**V8(Chrome, Opera)の重要な副作用はこのような変数はデバッグでは利用できないことである．**
これはデバッガのバグではなく，V8 の特別な機能である．

## The old "var"

`var`は非常に古くからあるため，`let`, `const`とは異なる振る舞いをする．
`var`変数はブロックスコープを持たず，"関数全体"か"グローバル"のいずれがであり，ブロック外からでも見ることが出来る．
`var`は関数の開始時に処理される．

## Global object

javascript が作られた時，すべてのグローバル変数と関数を提供する"グローバルオブジェクト"と言う考え方があった．複数のブラウザ内スクリプトがその単一のグローバルオブジェクトを使用し，それを介して変数を共有することが計画されていた．
それ以来，javascript は大きく進化し，グローバル変数を介してコードをリンクする考えはそれほど魅力的ではなかった．そのため，現代のモジュールのコンセプトが採用された．
しかし，ブラウザでは"window"，Node.JS では"global"などグローバルオブジェクトはまだ仕様に残っている．

1. 仕様や環境で定義されている組み込み関数や値へのアクセスを提供する．例えば，`alert`を`window.alert`などとして呼ぶことが出来る．
1. グローバルな関数宣言と`var`変数のアクセスを提供する．しかし，グローバルオブジェクトをは`let/const`で宣言された変数は持っていない．

グローバルオブジェクトはグローバル環境レコードではない．
ES-2015 からはこれらのエンティティは分割されている．

Node.JS のようなサーバーサイド環境では`global`オブジェクトは稀にしか利用できない．
特定のグローバル変数または組み込みが存在するかどうかを確認する時に使われる．
例えばグローバル関数`XMLHttpRequest`が存在するかどうか確認したい時，`if (XMLHttpRequest)`とするとエラーが起きるが，`window.XMLHttpRequest`はただのオブジェクトプロパティであり，エラーにはならずに動作する．

```javascript
if (window.XMLHttpRequest) {
  alert("XMLHttpRequest exists!");
}
```

また正当な window から変数を取得する場合で利用され，これが最も有効なユースケースである．
ブラウザは複数のウィンドウやタブを開いている場合があり，また，`<iframe>`に別のものが埋め込まれている場合もある．
すべてのブラウザウィンドウは自身の`window`オブジェクトとグローバル変数を持っており，javascript を使用すると，同じサイト(同じプロトコル，ホスト，ポート)からのウィンドウが相互に変数にアクセスすることが出来る．

```html
<iframe src="/" id="iframe"></iframe>
<script>
  alert(innerWidth); // 現在のウィンドウの innerWidth プロパティを取得(ブラウザのみ)
  alert(Array); // 現在のウィンドウの配列を取得(javascriptコアのみ)

  iframe.onload = function() {
    // iframe ウィンドウの幅を取得
    alert(iframe.contentWindow.innerWidth);
    // iframe ウィンドウから組み込みの配列を取得
    alert(iframe.contentWindow.Array);
  };
</script>
```

ここで最初の 2 つの alert は現在のウィンドウを使い，あとの 2 つは`iframe`ウィンドウからの変数を取る．
`iframe`が同じプロトコル/ホスト/ポートから発信されている場合，任意の変数にすることが出来る．

`this`の値はグローバルオブジェクトである．

1. ブラウザにおいて，グローバル領域の`this`の値は`window`である．
1. 非 strict モードで`this`を使った関数が呼ばれた場合，`this`としてグローバルオブジェクトを取る．
1. strcit モードでは`this`は`undefinede`になる．

## Function object, NFE

javascript のすべての値は型を持っている．関数はオブジェクトである．
関数は呼び出し可能な"アクションオブジェクト"と見なすことができ，それらを呼び出すだけでなく，オブジェクトとして扱うことが出来る．プロパティの追加/削除/参照渡しなど．

関数オブジェクトには往々にして使用可能なプロパティが少ししか無い．
例えば，関数名は"name"プロパティとしてアクセスできる．
関数式として割り当てられる関数にも正しい名前を貼り付けることが出来る．
デフォルト値を通して行われた代入でも動作する．

```javascript
function sayHi() {
  alert("Hi");
}

let sayHi = function() {
  alert("Hi");
};

function f(sayHi = function() {
  alert(sayHi.name); // works!
})

alert(sayHi.name); // sayHi
```

仕様ではこの機能は"contextual name(文脈上の名前)"と呼ばれる．
関数がそれを提供しない場合，代入はコンテキストから見つけ出される．

オブジェクトメソッドも同様に名前を持っている．

正しい名前を見つける方法がない場合，空の文字列が吐き出される．
実際にはほとんどの関数は名前を持っている．

関数パラメータの数を返す別の組み込みのプロパティ"length"がある．
しかし，残りのパラメータ(`...`)はカウントされない．
`length`プロパティは他の関数上で動作する関数で内省のために使われることがある．
例えば，引数の有無で処理を変更させる関数など．

```javascript
function ask(question, ...handlers) {
  let isYes = confirm(question);

  for (let handler of handlers) {
    if (handler.length == 0) {
      if (isYes) handler();
    } else {
      handler(isYes);
    }
  }
}

// 肯定的な解答では，両方のハンドラが呼ばれる．
// 否定的な解答では，2つ目だけが呼ばれる．
ask("Question?", () => alert("You said yes"), result => alert(result));
```

これは所謂ポリモーフィズム(Polymorphism)と呼ばれる特定のケースである．
引数を型に応じて，または`length`に応じた動作を行う．

また独自のプロパティを追加することも出来る．

```javascript
function sayHi() {
  alert("Hi");

  sayHi.counter++;
}
sayHi.counter = 0; // 初期値
```

### Warning

プロパティは変数ではない．
`sayHi.counter = 0`のような関数に割り当てられたプロパティは，関数の中でローカル変数`counter`として定義されていない．
言い換えるとプロパティ`counter`と変数`let counter`は無関係なものである．

関数プロパティは時々クロージャを置き換えることができる．

```javascript
function makeCounter() {
  // 次の代わり
  // let counter = 0

  function counter() {
    return counter.count++;
  }

  counter.count = 0;
  return conuter;
}
let counter = makeCounter();
alert(counter()); // 0
alert(counter()); // 1
counter.count = 10;
alert(counter()); // 10
```

`count`は今や外部のレキシカル環境ではなく，関数の中に直接格納されている．
主な違いは，`count`の値が外部変数にある場合，外部コードはそこにアクセスできないということである．
ネストされた関数だけがそれにアクセスすることが出来る．

名前付き関数式(Named Function Expression)は名前を持つ関数式の用語である．

```javascript
let sayHi = function func(who) {
  alert(`Hello, ${who}`);
};
```

`function`の後に名前`func`を追加しても，関数宣言にはならない．
なぜなら代入式の一部になっているためである．
名前`func`に関して 2 つの仕様がある．

1. 関数の内側から関数を参照することが出来る．
1. 関数の外側から見れない．

例えば以下の場合，下の関数`sayHi`は`who`が提供されていない場合，`"Guest"`で自身を再度呼び出す．

```javascript
let sayHi = function func(who) {
  if (who) {
    alert(`Hello, ${who}`);
  } else {
    func("Guest"); // 自身を再度呼び出すために，func を使用
  }
};

sayHi();

// しかしこれは動作しない．
func(); // Error, func は未定義
```

なぜ`func`を使うのか，内部でもう一度`sayHi`を使うと，`sayHi`の値が既に変更されている可能性があるためである．関数は別の変数になっており，コードがエラーを吐くように成るかもしれない．
これは，関数が外部のレキシカル環境から`sayHi`を取得するために起こる．
ローカルの`sayHi`が無いので，外部変数が使われる．呼び出しの瞬間，`sayHi`は`null`を表す．

```javascript
let sayHi = function func(who) {
  if (who) {
    alert(`Hello, ${who}`);
  } else {
    func("Guest"); // sayHiだとwelcomeに書き換えられた時にうまく動作しない．
  }
};

let welcome = sayHi;
sayHi = null;

welcome(); // Hello, Guest (ネスト呼び出しが機能する．)
```

これは動作する．なぜなら名前`func`は関数ローカルだからである．
それは外部のものではなく(外部から見えない)，この仕様は常に現在の関数を参照することを保証する．
外部コードは依然として変数`sayHi`または後の`welcome`を持っている，そして，`func`は内部の関数名であり，自身を呼び出すためのものである．

ここで説明された内部名の機能は関数式でのみ利用可能であり，関数宣言では利用できない．
関数宣言に対して，もう一つの"内部"の名前を追加する構文はない．
信頼できる内部名が必要なときには．それは関数宣言を名前付けされた関数式の形に書き換える理由に成る．

## The "new Function" syntax

関数を作る方法としてもう一つあるが，ほとんど使われていない．

```javascript
let func = new Function([arg1[, arg2[, ...argN]],] functionBody)
```

言い換えると関数パラメータが最初で本体が最後に来る．全ての引数は文字列である．
引数がない場合，1 つの引数(関数本体)だけになる．
`new Function`は任意の文字列を関数にすることが出来るため，例えばサーバから新しい関数を受け取り，それを実行することが出来る．
これはサーバからコードを受け取ったり，テンプレートから動的に関数をコンパイルするような非常に特定のケースで行われる．その必要性は通常，開発がかなり進んだ段階で発生する．

通常，関数は特別なプロパティ`[[Environment]]`でどこで生成されたかを覚えている．
それは作成された場所からレキシカル環境を参照する．
しかし，`new Function`を使用して作られた関数の場合，その`[[Environment]]`は現在のレキシカル環境ではなく，グローバルのレキシカル環境を参照する．

```javascript
function getFunc() {
  let value = "test";

  let func = new Function("alert(value)");
  // let func = function() { alert*value); }; // getFuncのレキシカル環境から
  return func;
}

getFunc()(); // error: valueは未定義
```

この`new Function`の動作は奇妙に見えるが，実践では非常に役に立つ．
本当に文字列から関数を作る必要がある場合をイメージしたとすると，
その関数のコードはスクリプト生成時には知られていないが実行中に認識される．
我々はサーバや別のコードからそれを受け取ることが出来る．
しかし，問題は javascript が本番環境に公開される前に，minifier(余分なコメントやスペースなどを削除することでコードを小さくする特別なプログラムで，ローカル変数をより短いものにリネームする．)を使用して圧縮されていることである．
例えば，関数が`let userName`を持っていた時，minifier はそれを`let a`などに置き換え，随所でそれを実行する．
もし`new Function`が外部変数へアクセスできる場合，`userName`を見つけることができない．
たとえ，`new Function`で外部のレキシカル環境へアクセスできたとしても minifiers で問題になる．
そのため，`new Function`の特殊なこの動作はエラーから守ることが出来る．

## Scheduling: setTimeout and setInterval

関数をすぐには実行させずに，ある時点で実行するようにしたいことがある．
これを"呼び出しのスケジューリング"という．
そのための 2 つのメソッドがある．

1. `setTimeout`は指定時間経過後，一度だけ関数を実行する．
1. `setInterval`は各実行の間は指定した間隔で，定期的に関数を実行する．
   これらのメソッドは javascript の仕様の一部ではないが，ほとんどの環境では内部スケジューラを持ち，それらのメソッドを提供する．
   特に，これらはすべてのブラウザと Node.JS でサポートされている．

### setTimeout

```javascript
let timerId = setTimeout(func|code, delay[, arg1, arg2...]);
```

`func|code`は関数もしくは実行するコードの文字列であり，通常は関数である．
コードの文字列も渡すことが出来るが，推奨されない．
`delay`は実行前の遅延時間で，ミリ秒単位である．
`arg1, arg2`は関数の引数である．

`setTimeout`の呼び出しは，実行を取り消すために使用できる"タイマー識別子"`timerId`を返す．
キャンセルするための構文は次のとおりである．

```javascript
let timerId = setTimeout(...);
clearTimeout(timerId);
```

`setInterval`も同じ構文を持っているが，関数を一回ではなく，定期的に与えられた時間間隔で実行する点が異なる．
これ以上の呼び出しを止めるためには`clearInterval(timerId)`を呼ぶ必要がある．

Chrome/Opera/Safari では，モーダルウィンドウを表示している間，timer の時間は止められている．

何かを定期的に実行するのに 2 つの方法がある．1 つは`setInterval`でもう 1 つは再帰的な`setTimeout`である．

```javascript
/*
 * let timerId = etInterval(() => alert('tick'), 2000);
 */
let timerId = setTimeout(function tick() {
  alert("tick");
  timerId = setTimeout(tick, 2000); // (*)
}, 2000);
```

上の`setTimeout`は現在の実行の最後の(\*)で次の呼び出しをスケジューリングする．
再帰的な`setTimeout`は`setInterval`よりも柔軟な処理を行うことが出来る．
例えば，5 秒毎にデータを確認するためにサーバへリクエストを送るサービスを書く必要があるとする．
しかし，サーバが高負荷である場合には間隔を 10, 20, 40 秒といったように増やしていく必要がある．

```javascript
let delay = 5000;
let timerId = setTimeout(function request() {
  ... send request ...
  if (request failed due to server overload) {
    // 次の実行のためにインターバルを増加させる．
    delay *= 2;
  }
  timerId = setTimeout(request, delay);
}, delay)
```

また，定期的に CPU を必要とするタスクを持っている場合には，実行にかかった時間を計測し次の呼び出しを計画することが出来る．
再帰的な`setTimeout`は実行の間の遅延を保証するが，`setInterval`は保証しない．
なぜなら，`setInterval`での`func`の実行にかかる時間はインターバルの一部を消費するためである．
そのため`func`の実行が予想していたよりも長くなり，100ms を超えてしまった場合，エンジンは`func`の完了を待ち，その後，スケジューラをチェックしてすぐに`func`を再度実行する．
`setInterval`は固定の遅延を保証する．

関数が`setInterval/setTimeout`に渡された時，内部参照がそこに作られ，スケジューラに保存される．
その場合，その関数への参照が他にない場合でも，関数はガベージコレクションの対象にはならない．
`setInterval`では`cancelInterval`が呼ばれるまで，関数はメモリ上に存在し続けている．
そのため，スケジュールされた機能がもう必要ないときは，たとえそれが非常に小さい機能だとしてもキャンセルしておくべきである．

特別なユースケースとして，`setTimeout(func, 0)`がある．
これは`func`をできるだけ速く実行するようにスケジュールする処理で，スケジューラは現在のコードが完了した後にそれを実行する．
なので，関数は現在のコードの"すぐ後"に実行するようにスケジュールされるため，言い換えると*非同期*処理である．

```javascript
setTimeout(() => alert("World"), 0);
alert("Hello");
```

これは`Hello`が最初で`World`が後に出力される．

`setTimeout`を使って CPU を必要とするタスクを分割するトリックがある．
例えば構文強調表示スクリプト(コードを色分けするためのもの)はかなり CPU が重くなる．
コードを強調表示するために，分析を実行し，多くの色の要素を作成し，テキストに追加する．
そこで，`setTimeout(..., 0)`を使って，最初の 100 行，次の 100 行といったように，長いテキストを小さく分割することが出来る．

```javascript
let i = 0;
let start = Date.now();

function count() {
  // 重い処理の一部を実行 (*)
  do {
    i++;
  } while (i % 1e6 != 0);
  if (i == 1e9) {
    alert("Done in " + (Date.now() - start) + "ms");
  } else {
    setTimeout(count, 0); // 新しい呼び出しをスケジュール (**)
  }
}

count();
```

1. 最初の実行:`i=1...1e6`
1. 2 回目の実行:`i=1e6+1...2e6`
1. ...が続き，`while`は`i`が`1e6`で均等に分割されているかどうかをチェックする．
   まだ終わっていない場合には(\*\*)で次の呼び出しがスケジュールされる．
   注目すべき点は両方のバリアントであり，`setTimeout`によりジョブを分割してもしなくてもスピードは同等である．全体のカウント時間に大きな違いはない．
   そこで`count()`の先頭にスケジューリングを移動させる．

```javascript
let i = 0;
let start = Date.now();

function count() {
  // 開始時にスケジューリングを移動する．
  if (i < 1e9 - 1e6) {
    setTimeout(count, 0); // 新しい呼び出しをスケジュール
  }

  do {
    i++;
  } while (i % 1e6 != 0);
  if (i == 1e9) {
    alert("Done in " + (Date.now() - start) + "ms");
  }
}

count();
```

これで`count()`を開始して`count()`をもっと呼ぶ必要があると知り，すぐにそれをスケジューリングするので実行時間が大幅に短縮される．

ブラウザではネストされたタイマーを実行できる頻度に制限がある．HTML5 標準では次のように書かれている. :"5 つのネストされたタイマーの後には間隔は少なくとも 4 ミリ秒に強制される."
サーバサイドではこの制限は存在せずブラウザのみである．

ブラウザ内のスクリプトの別の利点はプログレスバー等をユーザに表示できることである．
これはブラウザは通常スクリプトが完了した後に全ての"再ペイント"をするためである．

```HTML
<div id="progress"></div>
<script>
  let i = 0;
  function count() {
    for (let j = 0; j < 1e6; j++) {
      i++;
      // 現在のiを<div>に表示
      progress.innerHTML = i;
    }
  }
  count();
</script>
```

これを実行した場合，`i`の変更は count 全体が終わった後に行われる．
そこで`setTimeout`を使って小さく分割すると再描画を各実行の間で適用すること出来る．

```HTML
<div id="progress"></div>

<script>
  let i = 0;
  function count() {
    // 重い処理の一部を実行 (*)
    do {
      i++;
      progress.innerHTML = i;
    }while(i % 1e3 != 0);

    if (i < 1e9){
      setTimeout(count, 0)
    }
  }
  count();
</script>
```

ブラウザ内でのタイマーは，多くの理由で遅くなる可能性がある．

- CPU が過負荷になっている．
- ブラウザタブがバックエンドモードになっている．
- ラップトップがバッテリーモード

## Decorators and forwarding, call/apply

javascript は関数を扱う際，非常に柔軟性がある．関数は渡され，オブジェクトとして使われる．
これらの間の呼び出しを*転送*したり，それらを*装飾(デコレータ)*することが出来る．

### 透過キャッシュ(transparent caching)

CPU 負荷は高いが，その結果が不変である関数`slow(x)`を持っているとする．
言い換えると，同じ`x`の場合，常に同じ結果が変えってくる．
もし関数が頻繁に呼ばれた場合，再計算に余分な時間を費やすことを避けるために，異なる`x`の結果をキャッシュするとうれしい．
その機能を`slow()`に追加する代わりに，ラッパーを作る．

```javascript
function slow(x) {
  // CPUを大量に消費するジョブがここにある可能性がある．
  alert(`Called with ${x}`);
  return x;
}

function cachingDecorator(func) {
  let cache = new Map();

  return function(x) {
    if (cache.has(x)) {
      return cache.get(x);
    }

    let result = func(x);
    cache.set(x, result);

    return result;
  };
}

slow = cachingDecorator(slow);

alert(slow(1));
alert("Again: " + slow(1));

alert(slow(2));
alert("Again: " + slow(2));
```

上のコード中の`cachingDecorator`はデコレータであり，別の関数を取り，その振る舞いを変更する特別な関数である．
この考え方は，任意の関数で`cachingDecorator`を呼ぶことができ，それはキャッシングラッパーを返す．このような機能を使うことが出来る多くの関数を持つことが出来る．
また，関数に`cachingDecorator`を適用するだけで，メインの関数コードからキャッシュ機能を分離することでコードシンプルに保つことが出来る．

- `cachingDecorator`は再利用可能であり，別の関数に適用することも出来る．
- キャッシュロジックは分離されているので，`slow`自身の複雑性は増加しない．
- 必要に応じて複数のデコレータを組み合わせることができる．

### コンテキストのために，"func.call"を利用する

上で言及されたキャッシュデコレータはオブジェクトメソッドで動作するのに適していない．
例えば，下のコードでは，デコレーションの後，`worker.slow()`は動作を停止している．

```javascript
// worker.slow のキャッシングを作成する
let worker = {
  someMethod() {
    return 1;
  },

  slow(x) {
    // 実際にはCPUの重いタスクがここにある
    alert("Called with " + x);
    return x * this.someMethod(); // (*)
  }
};

alert(worker.slow(1)); // オリジナルメソッドは動く
worker.slow = cachingDecorator(worker.slow); // キャッシングする
alert(worker.slow(2)); // Error: Cannot read property 'someMethod' of undefined.
```

行(\*)で`this.someMethod`にアクセスしようとして失敗する．これはラッパーがオリジナル関数を func(x)として呼び出すためである．
次のコードを実行しようとすると，同様の現象が観察される．

```javascript
let func = worker.slow;
func(2);
```

したがって，ラッパーは呼び出しを元のメソッドに渡すが，コンテキスト`this`を使用しないのでエラーに成る．

明示的に設定した`this`で関数を呼び出すことが出来る，特別な組み込みメソッド`func.call(context, ...args)`がある．
これは提供された最初の引数を`this`とし，`func`を実行する．
下のコードでは，異なるオブジェクトのコンテキストで`sayHi`を呼び出している．
`sayHi.call(user)`は`this=user`を提供する`sayHi`を実行し，次の行で`this=admin`をセットする．

```javascript
function sayHi() {
  alert(this.name);
}

let user = { name: "John" };
let admin = { name: "Admin" };

// 別のオブジェクトを"this"として渡すため，call を使用する．
sayHi.call(user); // this = John
sayHi.call(admin); // this = Admin
```

また，オリジナルの関数にコンテキストを渡しため，ラッパーの中で`call`を使うことが出来る．

```javascript
function cachingDecorator(func) {
  let cache = new Map();
  return function(x) {
    if (cache.has(x)) {
      return cache.get(x);
    }
    let result = func.call(this, x); // "this"は正しいものが渡される．
    cache.set(x, result);
    return result;
  };
}
```

どのように`this`が渡されているかを見ると

1. デコレーション後の`worker.slow`はラッパー`function(x){...}`である．
1. `worker.slow(2)`が実行される時，ラッパーを引数として`2`と`this=worker`を取る．
1. ラッパーの中では，結果がまだキャッシュされていないと仮定すると，`func.call(this, x)`はオリジナルのメソッドに現在の`this(=worker)`と現在の引数`(=2)`を渡す．

### "func.apply"で複数の引数を使用する．

`cachingDecorator`をより普遍的なものにする．
どのようにして複数の引数の`worker.slow`メソッドをキャッシュするのか

```javascript
let worker = {
  slow(min, max) {
    return min + max;
  }
};

worker.slow = cachingDecorator(worker.slow);
```

ここでは，それを解決するための 2 つのタスクがある．
最初は`cache`マップのキーに`min`, `max`両方の引数を使う場合がある．以前は 1 つの引数`x`に対し，単に`cache.set(x, result)`として結果を保存し，`cache.get(x)`でそれを取得する．しかし，今回の引数の組み合わせ`(min, max)`で結果を覚える必要がある．ネイティブの`Map`は 1 つのキーに 1 つの値のみを持つ．
これを解決するための方法として

1. より汎用性があり，複数のキーを許可する新しい(もしくは 3rd party を使って)マップライクなデータ構造を実装する方法である．
1. 入れ子のマップを使う．`cache.set(min)`は`(max, result)`ペアを格納する`Map`になる．なので，`cache.get(min).get(max)`で result を得ることが出来る．
1. 2 つの値を 1 つに結合する，今のケースだと`Map`として文字列`"min, max"`を使うことが出来る．柔軟性のために，デコレータに*ハッシュ関数*を提供することが出来る．

実用的な多くのアプリケーションでは，第 3 の策で十分なので，それで進める．
解決するための 2 つ目のタスクは，多くの引数を`func`に渡す方法である．
現在ラッパー`function(x)`は 1 つの引数を想定しており，`func.call(this, x)`はそれを渡す．

これは`this=context`として設定し，引数のリストとして配列ライクなオブジェクト`args`を使って`func`を実行する．

```javascript
func(1, 2, 3);
func.apply(context, [1, 2, 3]);
```

両方共，引数`1, 2, 3`が与えられた`func`を実行する．
`apply`は`this=context`のセットもする．
例えば`this=user`と引数のリストとして`messageData`で`say`が呼ばれると次のようになる．

```javascript
function say(time, phrase) {
  alert(`[${time}] ${this.name}: ${phrase}`);
}

let user = { name: "John" };

let messageData = ["10:00", "Hello"]; // time と phraseになる
// user は thisになり，messageData は引数のリストとして渡される．
say.apply(user, messageData);
```

`call`と`apply`の構文の唯一の違いは，`call`は引数のリストを期待し，`apply`はそれらの配列ライクなオブジェクトを期待している点である．
我々は既に，引数のリストとして配列(もしくは任意の反復可能な(iterable))を渡すことの出来るスプレッド演算子`...`を知っている．`call`でそれを使う場合，`apply`とほぼ同じ結果になる．

```javascript
let args = [1, 2, 3];
func.call(context, ...args); // スプレッド演算子として配列を渡すことは
func.apply(context, args); // apply を使うのと同じである．
```

より詳しく見た時に`call`と`apply`の使い方には若干の違いがある．

- スプレッド演算子`...`は`call`へのリストとして，反復可能(iterable)な`args`を渡すことが出来る．
- `apply`は配列ライク(array-like)な`args`のみを許可する．

したがって，これらの呼び出しはお互いを補完する．反復可能が期待されるところでは`call`が，配列ライクが期待されるところでは`apply`が動作される．

また，`args`が反復可能であり，配列ライクである場合は，`apply`のほうが 1 つの操作であるためより高速である．

`apply`の最も重要な用途の 1 つは次のように別の関数へ呼び出しを渡すことである．

```javascript
let wrapper = function() {
  return anotherFunction.apply(this, arguments);
};
```

これは*呼び出し転送(call forwarding)*と呼ばれる．
`wrapper`はコンテキスト`this`と`anotherFunction`への引数を取得し，その結果の戻り値を返す．
このような`wrapper`が外部コードが呼び出すと元の関数の呼び出しと区別できなくなるので，もっと強力な`cachingDecorator`にする．

```javascript
let worker = {
  slow(min, max) {
    alert(`Called with ${min}, ${max}`);
    return min + max;
  }
};

function cachingDecorator(func, hash) {
  let cache = new Map();
  return function() {
    let key = hash(arguments); // (*)
    if (cache.has(key)) {
      return cache.get(key);
    }

    let result = func.apply(this, arguments); // (**)

    cache.set(key, result);
    return result;
  };
}

function hash(args) {
  return args[0] + "," + args[1];
}

worker.slow = cachingDecorator(worker.slow, hash);

alert(worker.slow(3, 5)); // works
alert("Again " + worker.slow(3, 5)); // same(cached)
```

これでラッパーは任意の数の引数で動作する．

- 行(\*)では，`arguments`から 1 つのキーを作成するために`hash`を呼び出している．ここでは，引数(3, 5)をキー"3,5"に変化する単純な"結合"関数を使う．より複雑なケースでは他のハッシュ関数が必要になることもある．
- 次に，(\*\*)でラッパーが取得したコンテキストとすべての引数(どれだけ多くても良い)を元の関数に渡すため，`func.apply`を使っている．

### Borrowing a method

ハッシュ関数に小さな改善をしてみる．先の関数だと 2 つの引数にのみ依存しているが，任意の数の`args`を指定できる方がよい．自然な対応策として，`arr.join`を使うことである．

```javascript
function hash(args) {
  return args.join();
}
```

残念なことに，これは動作しない．なぜなら，`hash(arguments)`と`arguments`オブジェクトは両方共反復可能であり配列ライクではあるが本当の配列ではないからである．
そこで以下のようにする．

```javascript
function hash() {
  return [].join.call(arguments);
}
```

このテクニックをメソッドの借用(method borrowing)と呼ぶ．
これはネイティブメソッドである`arr.join`内部アルゴリズムが非常に単純であるためである．

## Function binding

オブジェクトメソッドで`setTimeout`を使ったり，オブジェクトメソッドを渡すような場合，"`this`を失う"という既知の問題がある．

`this`はあるメソッドがオブジェクトから他の場所に渡されることによって失われる．
`setTimeout`を例に用いると

```javascript
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, $(this.firstName)!`);
  }
};

setTimeout(user.sayHi, 1000); // Hello, undefined!

// 以下のように書き直せる
let f = user.sayHi();
setTimeout(f, 1000); // user コンテキストを失う．
```

これは`setTimeout`はオブジェクトとは別に関数`user.sayHi`を持っているためである．
ラウザにブおいて，メソッド`setTimeout`は少し特別である．関数呼び出しでは`this=window`を設定する．従って，`this.firstName`は存在しない`window.firstName`を取得しようとする．他の同様のケースでは通常`this`は`undefined`になる．

このタスクは非常に典型的であり，オブジェクトメソッドをどこか別の場所(ここではスケジューラに渡して)から呼び出したい場合である．それが適切なコンテキストで呼び出されることはどのように確認すればいいだろうか．

1 つの方法として，ラップされた関数を使うことである．

```javascript
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

setTimeout(function() {
  user.sayHi();
}, 1000);
// 短い記法
setTimeout(() => user.sayHi(), 1000);
```

これは外部のレキシカル環境から`user`を受け取り，メソッドを普通に呼び出すためうまくいく．
しかし，コードの構造に僅かな脆弱性がある．
仮に`setTimeout`が動く前に(上の例では 1 秒の遅延がある)，`user`の値を変更していると間違ったオブジェクトを呼び出してしまう．

```javascript
// 1秒以内に次が行われると
user = {
  sayHi() {
    alert("Another user in setTimeout!");
  }
};
// Another user in setTimeout!
```

次の解決策はこのようなことが起きないことを保証する．
`this`を固定できる組み込みメソッド`bind`がある．
`func.bind(context)`の結果は特別な関数ライクな"エキゾチックオブジェクト(exotic object)"である．
これは関数として呼ぶことができ，`func`に`this=context`を透過的に渡す事ができる．
言い換えると，`boundFunc`の呼び出しは，固定された`this`での`func`呼び出しである．

例えば以下での`funcUser`は`this=user`での`func`呼び出しを渡す

```javascript
let user = {
  firstName: "John"
};

function func() {
  alert(this.firstName);
}

let funcUser = func.bind(user);
funcUser(); // John
```

ここで`func.bind(user)`は`this=user`で固定された`func`の"バインドされたバリアント"となる．

すべての引数はオリジナルの`func`に"そのまま"渡される．

```javascript
let user = {
  firstName: "John"
};

function func(phrase) {
  alert(phrase + ", " + this.firstName);
}

// this を user にバインドする
let funcUser = func.bind(user);
funcUser("Hello"); // Hello, John (引数 "Hello" が渡され，this=user)
```

オブジェクトメソッドで試してみると

```javascript
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

let sayHi = user.sayHi.bind(user); // (*)

sayHi();

setTimeout(sayHi, 1000);
```

(\*)の行でメソッド`user.sayHi`を`user`にバインドしている．
`sayHi`はバインドされた関数であり，単独もしくは`setTimeout`に渡して呼ぶことができる．
引数がそのまま渡され，`this`だけが`bind`によって固定されていることがわかる．

```javascript
let user = {
  firstName: "John",
  say(phrase) {
    alert(`${phrase}, ${this.firstName}!`);
  }
};

let say = user.say.bind(user);
say("Hello");
say("Bye");
```

もしオブジェクトが多くのメソッドを持ち，それらをバインドする必要がある場合，すべてループでバインドすることが出来る．

```javascript
for (let key in user) {
  if (typeof user[key] == "function") {
    user[key] = user[key].bind(user);
  }
}
```

また，便利な大量バインディングのための機能も提供している．`_bindAll(obj) in lodash`

## Currying and partials

今までは`this`をバインドすることについて話していた．`this`だけでなく引数もバインドすることが出来る．めったにしか見ないが，便利な時がある．
bind の完全構文は以下のようになる．
`let bound = func.bind(context, arg1, arg2)`
これはコンテキスト(`this`として，)関数の開始引数をバインドすることが出来る．
例えば`mul(a, b)`を考えると

```javascript
function mul(a, b) {
  return a * b;
}
```

これをベースとした関数`double`を作るために`bind`を使ってみる．

```javascript
let double = mul.bind(null, 2);

alert(double(3)); // = mul(2. 3) = 6
alert(double(4)); // = mul(2, 4) = 6
```

`mul.bind(null, 2)`を呼び出す新しい関数`double`が作成される．それは`null`がコンテキストとして，`2`が最初の引数として固定されるからである．さらに，その他の引数は"そのまま"渡される．
この既存の関数の一部のパラメータを変更することで新しい関数を作ることを`関数の部分的な適用(partial function application)`と呼ばれる．
ここでは実際には`this`を使っていないが，`null`を指定する必要がある．
ここでのメリットはわかり易い名前(`double`, `triple`)で独立した関数を作れたことである．`bind`で固定されているため，毎回最初の引数を書く必要がない．
別のケースでは，非常に汎用的な関数を持っており，利便性のために汎用性を減らしたい時に部分適用は役に立つ．例えば関数`send(from, to, text)`を考えると`user`オブジェクトの内側で，その部分的なバリアントを使いたいなどのケースである:e.g. 現在のユーザーから送信を行う`sendTo(to, text)`など．

仮に，いくつかの引数を修正したいが，`this`をバインドしない場合はどうなるだろう．ネイティブの`bind`はそれを許可されていないため，コンテキストを省略して引数にジャンプすることはできない．
ただ，幸いにも引数だけをバインドする`partial`関数は簡単に実装することが出来る．

```javascript
function partial(func, ...argsBound) {
  return function(...args) {
    return func.call(this, ...argBound, ...args);
  };
}
// 使い方
let user = {
  firstName: "John",
  say(time, phrase) {
    alert(`[${time}] ${this.firstName}: ${phrase}!`);
  }
};

// 最初の引数を固定して何かを表示する部分的なメソッドを追加する．
user.sayNow = partial(
  user.say,
  new Date().getHours() + ":" + new Date().getMinutes()
);
user.sayNow("Hello"); // [10:00] Hello, John!
```

`partial(func[, arg1, arg2...])`の呼び出しの結果は次のように`func`を呼び出すラッパーである．

- `this`はそれが取得したものと同じである(`user`)
- 次に`...argsBound`を与える(`partial`呼び出しからの引数("10:00"))
- 次に`...args`である．ラッパーへ与えられた引数(`"Hello"`)である．

スプレッド演算子を使うと楽になっていることがわかる．また，`loadash`ライブラリの`_.partial`関数も実装されている．

"カリー化(Currying)"と呼ばれる別のものと，上で言及された関数の部分適用を混同する人もいる．
Currying は`f(a, b, c)`と呼び出し可能なものを，`f(a)(b)(c)`として呼び出しできるように変換することである．
2 変数関数に対する Carrying を行う`curry`を作ってみる．つまり`f(a, b)`を`f(a)(b)`に変換する．

```javascript
function curry(func) {
  return function(a) {
    return function(b) {
      return func(a, b);
    };
  };
}

// 使い方
function sum(a, b) {
  return a + b;
}

let curriedSum = curry(sum);
alert(curriedSum(1)(2)); // 3
```

上でわかるように，実装はラッパーの連続である．

- `curry(func)`の結果は`function(a)`である．
- `sum(1)`のように呼ばれるとき，引数はレキシカル環境に保存され，新しいラッパー`function(b)`が返却される．
- そして最終的に`sum(1)(2)`は`2`で`function(b)`を呼び，それは元の複数引数を取る`sum`を呼ぶ．
  `loadsh`の`_.curry`のようなカリー化のより高度な実装は，より洗練された処理を行う．それらすべての引数が提供された場合には関数が正常に呼び出されるるようなラッパーを返し，そうでないときは部分的を返す．

```javascript
functino curry(f) {
  return function(..args) {
    // もし args.length == f.lengthの場合(fと同じ引数がある場合),
    // 呼び出しをfへ渡す
    // それ以外はargsを最初の引数として固定する部分関数を返す．
  }
}
```

高度な Currying を使用すると簡単に関数を通常呼び出し可能にしつつ，部分適用をすることができる．このメリットを理解するために価値のある実例を見る必要がある．
例えば情報を整形して出力するロギング関数`log(date, importance, message)`を持っているとする．実際のプロジェクトでは，このような関数には，ネットワーク経由での送信やフィルタリングなど，他にも多くの便利な機能がある．
これを Currying すると，

```javascript
functino log(date, importance, message) {
  alert(`[${date.getHours()}:${date.getMinutes()}] [${importance}] ${message}`);
}
log = _.curry(log);
// この処理の後でも`log`は通常の方法で動く
log(new Date(), "DEBUG", "some debug");
// Curryingされた形式でも呼ぶことができる．
log(new Date())("DEBUG")("some debug");
// 便利な関数を作成する．
let todayLog = log(new Date());
todayLog("INFO", "message"); // [HH:mm] INFO message
// また変更できることも出来る．
let todayDebug = todayLog("DEBUG");
todayDebug("message"); // [HH:mm] DEBUG message
```

1. Currying しても何も失っていない(`log`)は以前のように呼び出し可能である．
1. いろんなケースに応じて便利な部分適用した関数を生成することが出来る．

また，高度なカリー実装を示す．

```javascript
function curry(func) {
  return function curried(...args) {
    if (args.length >= func.length) {
      return func.apply(this, args);
    } else {
      return function pass(...args2) {
        return curried.apply(this, args.concat(args2));
      };
    }
  };
}

function sum(a, b, c) {
  return a + b + c;
}

let curriedSum = curry(sum);
alert(curriedSum(1, 2, 3)); // 通常呼び出し 6
alert(curriedSum(1)(2, 3));
alert(curriedSum(1)(2)(3));
```

新しい`curry`は複雑に見えるが，`curry`の結果は`curried`のラッパーである．
これは 2 つの分岐がある．

1. 渡された`args`の数と元の関数で定義されている引数の数が同じか，より多い場合，単にそれを呼び出し`func`に渡す．
1. 部分適用を得る: そうでない場合は`func`はまだ呼ばれず代わりに別のラッパー`pass`が返される．これは`curried`を再度適用して新しい引数と一緒に前の引数を提供する．そのご新しい呼び出しでは新しい部分適用(引数が不十分な場合)か最終的に求めたい結果が得られる．

Currying は関数が固定数の引数を持つ必要がある．
また，定義上，Currying は`sum(a, b, c)`は`sum(a)(b)(c)`に変換するべきである．しかし，javascript での Currying のほとんどの実装は説明されているように高度であり，副引数のバリアントでも関数呼び出しが可能となっている．

## Arrow functions revisited

アロー関数について改めて考える．
アロー関数は関数を小さく書くための単なる簡略化ではない．
javascript は小さな関数を書く必要がある状況に満ちており，それはいろんな場所で実行される．
例えば，

- `arr.forEach(func)` - `func`は`forEach`によってすべての配列項目に対して実行される．
- `setTimeout(func)` - `func`は組み込みのスケジューラによって実行される．
  関数を作成してどこかに渡すのは javascript の真髄である．そしてこのような関数は通常現在のコンテキストから離れたくはない．

以前にもやったとおり，アロー関数は`this`を持っていない．もし，`this`がアクセスされた場合，それは外側から取られる．例えばそれを使って，オブジェクトメソッドの内側を反復することが出来る．

```javascript
let group = {
  title: "Our Group",
  students: ["John", "Pete", "Alice"],

  showList() {
    this.students.forEach(student => alert(this.titile + ": " + student));
  }
};

group.showList();
```

`forEach`ではアロー関数が使われているのでその中の`this.title`は外部メソッド`showList`と全く同じである．つまり，`group.title`である．もし通常の関数を使った場合にはエラーになる．

`this`を持たないことは当然別の制限を持っており，アロー関数はコンストラクタとして利用できないので`new`を呼び出すことはできない．

アロー関数は`arguments`変数も持たない．それは現在の`this`と`arguments`を使って呼び出しをフォワードする必要があるときなど，デコレータとしては素晴らしいものである．
例えば`defer(f, ms)`は関数を得て，`ms`ミリ秒で呼び出しを遅らせるラッパーを返す．

```javascript
function defer(f, ms) {
  return function() {
    setTimeout(() => f.apply(this, arguments), ms);
  };
}

function sayHi(who) {
  alert("Hello, " + who);
}

let sayHiDefferd = defer(sayHi, 2000);
sayHiDefferd("John"); // Hello, John 2秒後に
```

アロー関数を使わない場合は次のように成る．

```javascript
function defer(f, ms) {
  return function(...args) {
    let ctx = this;
    setTimeout(function() {
      setTimeout(function() {
        return f.apply(ctx, args);
      }, ms);
    });
  };
}
```

ここでは`setTimeout`内の関数がそれを受け取るために，余分な`args`と`ctx`を作る必要があった．
