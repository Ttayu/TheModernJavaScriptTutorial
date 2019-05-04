# Data types

## Methods of primitives

プリミティブは`string`, `number`, `boolean`, `symbol`, `null`, `undefined`の 6 つである．
必要に応じて追加の機能を提供する特別な"オブジェクトラッパー"が作られ，その後破棄されプリミティブだけが残る．
"オブジェクトラッパー"はプリミティブ型毎に異なり，`String`, `Number`, `Boolean`, `Symbol`と呼ばれる．(e.g. `variable.toUpperCase()`)
コンストラクタ`String/Number/Boolean`は内部でのみ利用する．
`new Number(1)`のように明示的にプリミティブのための"ラッパーオブジェクト"を作れるが，強く推奨しない．
`null`, `undefined`は例題でメソッドを提供していない．
形式的にはそれらのメソッドは一時的なオブジェクトを通して行われる．
しかし，javascript エンジンは内部的に最適化するようチューニングされているため呼び出しのコストはかからない．

## Numbers

メソッドを呼ぶための 2 つのドットをつけることがある．
`123456..toString()`などとする．1 つのドットは小数部分と考えるため．
数値処理の丸めには`Math`オブジェクトを使う．
値を`===`のように比較する特別な組み込みメソッド`Object.is`がある，

1. `Object.is(NaN, NaN) === true`
1. `Object.is(0, -0) === false`
   上記以外のすべてのケースでは`Object.is(a, b)`は`a === b`と同じである．

`"100px"`や`"12pt"`などから数値だけを取り出したいことをままある．
`Number()`だと`NaN`を返すので，`parseInt`, `parseFloat`を使う．
ただ，`a123`などは最初の文字で処理が止まってしまうので`NaN`を返す．

## Strings

内部のフォーマットは常に"UTF-16"である，
javascript は文字列を変更することはできない．
`indexOf`は見つかった位置を返し，見つからなかった場合は`-1`を返す．
部分文字列を探索するときには

```javascript
let str = "As sly as a fox, as strong as an ox";
let target = "as";
let pos = -1;
while ((pos = str.indexOf(target, pos + 1)) != -1) {
  alert(pos);
}
```

`if`テストの中では`indexOf`は少し不便である．

```javascript
if (str.indexOf(target)) {
  alert("We found it."); // 動作しない．
}

if (~str.indexOf(target)) {
  alert("We found it."); //動作する．
}
```

`~n` は `-(n+1)`と同じ意味であるため，`n == -1` のときだけ`0`になる．
通常明白でない方法を使用することは推奨されないが，このトリックは昔から広く使われている．

部分文字列を取得するのに`substring`, `substr`, `slice`がある．
`substr`はコアな javascript の仕様ではなく非ブラウザ環境では動かない可能性がある．
実際にはほぼどこでも動作するが．
ほとんどのケースで`slice`を使うのが良い．

言語に従って文字列を比較するためには`localCompare`を使う．
ほとんどの記号は 2 バイトのコードを持っている．
しかし 65536 通りしか許されないので，非常に珍しい記号は"サロゲートペア"と呼ばれる 2 バイト文字のペアでエンコードされている．
javascript が作られた時にサロゲートペアは存在していなかったため言語として正しく処理されていない．

## Arrays

javascript の配列はキュートスタックどちらとしても動作する．
`pop/push`，`shift/unshift`メソッドを使う．
配列は連続した順序付きデータを処理するため，javascript エンジン内部で注意深くチューニングされている．
メソッド`push/pop`は処理が早く，`shift/unshift`は遅い．
`shift`は配列の最初を操作するので 3 つのことをする必要がある．

1. インデックス 0 の要素を削除する．
1. すべての要素を左に移動する．
1. `length`プロパティを更新する．
   配列内の要素が増えれば増えるほど，移動に必要な時間とメモリ内の操作が増える．
   `pop`は他の要素のインデックスは変わらないので，`pop`メソッドは何も移動させる必要がない．そのため高速である．
   配列のループには`for..in`ではなく`for..of`を使う．
   `for..in`は*全てのプロパティ*を繰り返し処理する．もし配列のようなオブジェクトを処理する必要があるとき，それらの"余分な"プロパティが揉んだに成ることがある．
   また，`for..in`は汎用オブジェクトに対して最適化されているため，10 から 100 倍遅くなる．
   `length`プロパティは書き込み可能である．
   そのため，配列をクリアする最もシンプルな方法は`arr.length = 0;`とすることである．

`new Array(number)`への呼び出しは与えれられた長さの配列を作り，要素を持たない．
そのため，通常は角括弧を使う．

## Array methods

配列の要素の削除は`delete`を使わない．
`arr.splice(str)`を使う．これは追加，削除，また要素の挿入も出来る．
`arr.forEach`は配列の全要素に対して関数を実行することが出来る．

```javascript
arr.forEach(function(item, index, array) {
  // ...itemに対してなにか処理をする．
  alert(`${item} is at index ${index} in ${array}`);
});
```

オブジェクトの配列を持っているとき，特定条件を持つオブジェクトを見つけるのに`arr.find`が便利である．

```javascript
let users = [
  {id: 1, name: "John"},
  {id: 2, name: "Pete"},
  {id: 3, name: "Mary"},
];

let user = users.find(function(item, index array){
  return item.id == 1
});

alert(user.name); // John
```

`find`は単一の要素を探すがそれが多い場合`arr.filter(fn)`を使う．
これはマッチした要素の配列を返す．

`arr.map(fn)`は配列の各要素で関数を呼び出し，結果の配列を返す．
`arr.sort(fn)`は組み込みでソートアルゴリズムの実装を持っている．(殆どの場合最適化されたクイックソートである．)
配列かどうかの確認には`typeof`ではなく，`Array.isArray(value)`を使う，前者では`array`を`object`と返すため．

ほとんどのメソッドは`"thisArg"`をサポートする．これは殆ど使われない．

## Iterables

反復可能な(iterables)オブジェクトは配列の汎化である．
これは`for..of`ループで任意のオブジェクトを使用できるようにするための概念である．

`range`を反復可能にするために(`for..of`を動作させるために)は，`Symbol.iterator`という名前のメソッドをオブジェクトに追加する必要がある．

- `for..of`が始まると，そのメソッドを呼び出す．
- メソッドは iterator(メソッド`next`を持つオブジェクト)を返さなければならない．
- `for..of`が次の値を必要とするとき，そのオブジェクトの`next()`を呼び出す．
- `next()`の結果は`{done: Boolean, value: any}`の形式である必要がある．`done=true`は繰り返しが終わったことを示す．

以下は`range`の完全な実装である．

```javascript
let range = {
  from: 1,
  to: 5
};

// 1. for..of の呼び出しは最初にこれを呼び出す．
range[Symbol.iterator] = function() {
  // 2. これはiteratorを返す．
  return {
    current: this.from,
    last: this.to,

    // 3. for..of ループにより各繰り返しで next() が呼ばれる．
    next() {
      // 4. オブジェクト {done: .., value: ..}を返す必要がある．
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      }
      return { done: true };
    }
  };
};

for (let num of range) {
  alert(num); // 1, then 2, 3, 4, 5
}
```

- `range`自身は`next()`メソッドは持っていない．
- 代わりに別のオブジェクト，イテレータは`range[Symbol.iterator]()`の呼び出しで生成され，反復を処理する．

技術的には，コードをシンプルにするためにそれらをマージして`range`自身をイテレータとして使うことも出来る．

```javascript
let range = {
  from: 1,
  to: 5,

  [Symbol.iterator]() {
    this.current = this.from;
    return this;
  },

  next() {
    if (this.current <= this.to) {
      return { done: false, value: this.current++ };
    }
    return { done: true };
  }
};
```

この`range[Symbol.iterator]()`は`range`オブジェクト自身を返す．欠点はオブジェクトに対して同時に 2 つの`for..of`ループを実行することができないという点である．イテレータが 1 つしか無いので，オブジェクトは繰り返し状態を共有する．

文字列も反復可能であり，サロゲートペアも正しく動作する．

通常 iterables の内部は外部のコードからは隠れている．`for..of`ループがあり，それが動作する．
もう少し深く理解するために，明示的なイテレータを作る．
`for..of`と同じ方法で文字列を反復処理するが，直接呼び出しを行う．

```javascript
let str = "Hello";
// for (let char of str) alert(char);
// と同じことをする．

let iterator = str[Symbol.iterator]();

while (true) {
  let result = iterator.next();
  if (result.done) break;
  alert(result.value);
}
```

ほとんど必要ではないが`for..of`よりも処理をよりコントロール出来る．
例えば繰り返し処理を分割したい場合など．

- 反復可能(iterables)は`Symbol.iterator`メソッドを実装したオブジェクトである．
- 配列ライク(Array-like)はインデックスと`length`を持ったオブジェクトである．
  もちろんその特性を組み合わせることは出来る．

```javascript
let arrayLike = {
  // インデックスとlengthを持っている．→ 配列ライク
  0: "Hello",
  1: "World",
  length: 2
};

// エラー(Symbol.iteratorはないので)
for (let item of arrayLike) {
}
// iterableかarray-likeか調べ新しい配列を作り全てのアイテムをコピーする．
let arr = Array.from(arrayLike);
alert(arr.pop());
```

`Array.from`の完全な構文では，オプションで"マッピング"関数を指定することが出来る．
`Array.from(obj[, mapFn, thisArg])`

多くの組み込みメソッドは"本当の"配列の代わりに iterable または array-like で動作することを想定している．

## Map, Set, WeakMap and WeakSet

Map は`object`と同じようにキー付されたデータ項目の集まりである．
主な違いは`Map`は任意の型のキーを許可することである．
オブジェクトは全てのキーが文字列に変換されるため，差別化ができている．

`Map`はキーとしてオブジェクトを使うことが出来る．
これは最も重要で特筆すべき`Map`の機能である．
`Map`は等価のテストをするために，SameValueZero アルゴリズムを使う．
大雑把には厳密等価`===`と同じだが，違いは`NaN`は`NaN`と等しいとみなされる点である．
なので`NaN`も同様にキーとして使うことが出来る．

`Map`を生成するとき，キー/値のペアを持つ配列(または別の itrable)を渡すことが出来る．
オブジェクトのキー/値のペアの配列をその形式で返す組み込みメソッド`Object.entries(obj)`があるので map の初期化を行うことが出来る．

```javascript
let map = new Map([["1", "str1"], [1, "num1"], [true, "bool1"]]);
let map = new Map(
  Object.entries({
    name: "John",
    age: 30
  })
);
```

Map の繰り返しは値が挿入された順で行われる．
`map.keys()`, `map.values()`, `map.entries()`, `map.forEach`

`Set`は値の集まりで，それぞれの値は一度しか現れない．
`Set`の代替はユーザの配列と`arr.find`を使って挿入ごとに重複をチェックするコードである．
しかし，このメソッドは全ての要素をチェックするため配列全体を見るのでパフォーマンスははるかに悪い．
`Set`は一意性チェックを高速に行うように，内部的に最適化されている．
`Set`の中の`forEach`メソッドは 3 つの引数を持っている．value, valueAgain, object
これは`Map`との互換性のためである．

`WeakSet`は javascript がメモリからそのアイテムを削除するのを妨げない`Set`の特別な種類である．
javascript エンジンはそれが到達可能な間，メモリ上に値を保持する．
`WeakMap/WeakSet`はキーオブジェクトのガベージコレクションを妨げることはない．
`WeakMap`のキーはプリミティブな値ではなくオブジェクトでなければならない．
オブジェクトをキーとして使用し，そのオブジェクトへの他の参照が他にない場合，自動的にメモリから削除される．
`WeakMap`は繰り返しとメソッド`keys()`, `values()`, `entries()`をサポートしない．
これは技術的な理由から制限がある．
もしオブジェクトが全ての他の参照を失った場合，自動的に削除される．しかし，技術的にはいつ*クリーンアップ*が発生するかは正確に指定されていない．
これは javascript エンジンが決定する．エンジンはすぐにメモリのクリーンアップを実行するか，待ってより多くの削除が発生したあとにクリーンアップするかを選択する．
したがって技術的には`WeakMap`の現在の要素数がわからないため，全体にアクセスするメソッドはサポートされていない．

`WeakMap`はオブジェクトが存在している間だけ存在するオブジェクトに対して何かを格納することが出来る．
どこかにオブジェクトを格納するメインの場所を持ち，オブジェクトが存続している間だけ関連する追加情報を保持する必要があるといった場合に便利である．(ユーザごとの訪問回数を保持するなど)
通常の`Map`だとユーザーが離れたあとのクリーンアップはうんざりするタスクであるが，`WeakMap`は自動的にクリーンアップするので，物事をよりシンプルにすることが出来る．

`WeakSet`も同様に振る舞う．

## Object.keys, values, entries

`keys()`, `values()`, `entries()`は一般的なメソッドである．
そのため，独自のデータ構造を作成するときに，それらを実装しておく必要がある．
通常のオブジェクトでは次のメソッドが使える．

- Object.keys(obj)
- Object.values(obj)
- Object.entries(obj)
  `obj.keys()`ではなく`Object.keys()`と呼ぶ必要がある．主な理由は柔軟性である．
  javascript はオブジェクトはすべての複雑な構造のベースである．
  独自の`orderr.values()`メソッドを実装する`order`という独自のオブジェクトがあるかもしれないが，それでも`Object.values(order)`を呼ぶことが出来る．
  2 つ目の違いは`Object.*`メソッドが単なる iterable ではなく"本当の"配列オブジェクトを返すことである．これは歴史的な理由である．

`Object.*`は`Symbol`を使っているプロパティを無視する．
通常それは便利であるが，もしこのようなキーも同様に扱いたい場合は`Object.getOwnPropertySymbols`を使う．これは Symbol を使っているキーのみの配列を返す．
`Reflect.ownKeys(obj)`は全てのキーを返す．

## Destructing assignment

分割代入(Destructing assignment)は，配列またはオブジェクトの中身を複数の変数に代入できる特別な構文である．デストラクタリング(非構造化/構造の分解)は，多くのパラメータとデフォルト値を持つ複雑な関数でもうまく機能する．

Array の非構造化

```javascript
let arr = ["Ilya", "Kantor"];

// デストラクタリング
let [firstName, surname] = arr;
// let [firstName, surname] = "Ilya Kantor".split(" ");

alert(firstName);
alert(surname);
```

分割は破壊的を意味しない．項目を変数にコピーすることによって，"非構造化(destructurizes)"するため．"分割代入(destructuring assignment)"と呼ばれる．配列自体は変更されない．

配列の不要な要素は，余分なカンマをつけることで捨てることが出来る．

```javascript
// 1番目と2番目と4番目の要素が不要の場合．
let [, , title] = ["Julius", "Caesar", "Consul", "of the Roman Republic"];
alert(title); // Consul
```

実際には配列だけでなく，右辺は任意の iterable に対して動作する．
左辺は任意の"割当可能なもの"を指定することが出来る．

```javascript
let user = {};

[user.name, user.surname] = "Ilya Kantor".split(" ");
alert(user.name); //Ilya
```

`Object.entries(obj)`などでも分割代入が使える．

最初の値を取得するだけでなく，それに続くすべての値も集めたい場合，"残りの場合"の取得を意味する 3 つのドット`"..."`をパラメータに追加することで実現できる．

```javascript
let [name1, name2, ...rest] = [
  "Julius",
  "Caesar",
  "Consul",
  "of the Roman Republic"
];

alert(name1); // Julius
alert(name2); // Caesar

alert(rest[0]); // Consul
alert(rest[1]); // of the Roman Republic
alert(rest.length); // 2
```

`rest`は残りの値が要素として格納されている配列である．利用時には分割代入の最後に来るようにする．

代入する変数の数よりも配列の要素数のほうが少ない場合，エラーにはならず，不足している値は`undefined`と見なされる．
値がなかった場合に"デフォルト"値を使いたければ`=`を使ってデフォルト値を指定することが出来る．
デフォルト値はより複雑な式や関数呼び出しにすることもでき，それらは値が提供されなかったときのみ評価する．

```javascript
let [name = "Guest", surname = "Anonymous"] = ["Julius"];

alert(name); // Julius
alert(surname); // Anonymous

// surnameだけpromptが実行される．
let [name = prompt("name?"), surname = prompt("surname?")] = ["Julius"];
alert(surname); // promptが得たもの
```

分割代入はオブジェクトでも動作する．
右辺には変数に分割したい既存のオブジェクト，左辺には該当するプロパティのパターンを指定する．
単純なケースでは`{...}`に変数名を並べたものがある．
プロパティを別の名前の変数に代入したい場合，コロンを使ってセットすることが出来る．

```javascript
let options = {
  title: "Menu",
  width: 100,
  height: 200
};

let { width: w, height: h, title } = options;

//  width -> w
//  height -> h
//  title -> title
```

値がない可能性のあるプロパティについてはデフォルト値を設定することが出来る．
コロンと等号の両方を組み合わせることが出来る．

仮に，指定した変数よりも多くのプロパティをオブジェクトが持っていたらどうするか，
残りの演算子(3 つのドット)を使用するという仕様はほぼ標準であるが，まだサポートされていないブラウザもある．

```javascript
let options = {
  title: "Menu",
  height: 200,
  width: 100
};

let { title, ...rest } = options;

// now title="Menu", rest={height: 200, width: 100}
alert(rest.height); // 200
alert(rest.width); // 100
```

`let {...} = {...}`での変数は代入の直前に宣言されている.
`{title, width, height} = {...}`は動作しない．
javascript がメインコードフローの`{...}`をコードブロックとして扱うためである．
そのためコードブロックではないと javascript に示すためには，代入全体を括弧`(...)`で囲む飛鳥がある．

オブジェクトや配列に別のオブジェクトや配列入れ子になっている場合，より複雑なパターンを使用して，より深い部分を抽出するお供できます．

```javascript
let options = {
  size: {
    width: 100,
    height: 200
  },
  items: ["Cake", "Donut"],
  extra: true // 分割されない何かの追加データ
};

// わかりやすくするために，複数の行での分割代入．
let {
  size: { width, height },
  items: [item1, item2],
  title = "Menu" // オブジェクトには存在しない(デフォルト値が使われる．)
} = options;
```

左辺になかった`extra`を除いた`optinos`オブジェクト全体が該当する変数に代入される．
以下のようなこともある．

```javascript
// 変数全体からsizeを取り，残りは無視する．
let { size } = options;
```

ある関数が多くのパラメータを持っており，ほとんどがオプションであることがある．
以下は良くない関数の書き方である．
`function showMenu(title = "Untiled", width = 200, height = 100, items = []){}`
問題の 1 つは，どうやって引数の順番を覚えるかである．
コードがしっかりドキュメント化されていれば通常は IDE が助けてくれる．
他にもほとんどのパラメータがデフォルトで OK の場合の関数の呼び方である．
`showMenu("My Menu", undefined, undefined, ["Item1", "Item2"])`
これは読みにくく，より多くのパラメータを扱う場合非常に読みにくい．
そこで非構造化が役に立つ．オブジェクトとしてパラメータを渡し，関数はそれらを変数に分解する．

```javascript
let options = {
  title: "My Menu",
  items: ["Item1", "Item2"]
};

function showMenu({
  title = "Untitled",
  width = 200,
  height = 100,
  items = {}
}) {
  // title, items - optionsから取得
  // width, height - デフォルト値を利用
}

showMenu(options);
```

また入れ子のオブジェクトやコロンのマッピングを使った複雑な非構造化を使うことも出る．
このような分割代入は`showMenu()`に引数があることを前提している．
全ての値をデフォルトにしたい場合には空のオブジェクトを指定する必要がある．`showMenu({})`

## Date and time

`Data()`で日付や時刻を保存し管理するためのメソッドを提供する．
`Data(miliseconds)`, `Data(datestring)`, `Data(year, month, date, hours, minute, seconds, ms)`

### Warning

多くの javascript エンジンは標準ではない`getYear()`を実装しているがこれは非推奨である．
これは 2 桁の年を返す時があるので，年の取得には`getFullYear()`がある．

`Date`オブジェクトは範囲外の値を指定した場合，自動補正を行い，自動的に調節される．

日付の差分だけを測定したい場合，`Date`オブジェクトを使う必要はない．
現在のタイムスタンプを返す特別なメソッド`Date.now()`がある．
これは`new Date().getTime()`と同じであるが，中間の`Date`オブジェクトを作らないため，より速く，ガベージコレクションに負担をかけない．
これは主に利便性のために，ゲームやアプリのパフォーマンスが重要な場合に使われるので`Date.now()`を使うのがベターである．

CPU を必要とする機能について信頼できるベンチマークが必要な場合には注意が必要である．

```javascript
function diffSubtract(date1, date2) {
  return date2 - date1;
}

function diffGetTime(date1, date2) {
  return date2.getTime() - date1.getTime();
}

function bench(f) {
  let date1 = new Date(0);
  let date2 = new Date();

  let start = Date.now();
  for (let i = 0; i < 1e5; i++) f(date1, date2);
  return Date.now() - start;
}

alert("Time of diffSubtract: " + bench(diffSubtract) + "ms");
alert("Time of diffGetTime: " + bench(diffGetTime) + "ms");
```

この結果は`getTime()`を使うほうが遥かに速い．この場合は型変換がなく，エンジンが最適化をするのがとも簡単であるためである．
よってこれはまだ良いベンチマークではない．
`bench(diffSubtract)`を実行している時に，CPU は並列で何かをしていてリソースを消費しており，`bench(diffGetTime)`の実行時までにはその作業が完了していたと想像できる．
これは現代のマルチプロセス OS でのよくある問題である．
結果として，最初のベンチマークは 2 回目よりも利用できる CPU リソースが少なくなる．
**より信頼性のある高いベンチマークを行うには，ベンチマーク全体を複数開催実行する必要がある．**

```javascript
function bench(f) {...}

let time1 = 0;
let time2 = 0;

// メインループの前の"ヒートアップ"のために追加．
bench(diffSubtract);
bench(diffGetTime);

// bench(upperSlice)とbench(upperLoop)を交互に10回実行する．
for (let i = 0; i < 10; i++) {
time1 += bench(diffSubtract);
time2 += bench(diffGetTime);
}
```
現代のjavascriptエンジンは何度も実行される「ホットコード」に対してのみ最適化を適用する．(ほとんど実行されないものを最適化する必要はないためである．)
したがって上記の例では最初の実行は最適化されていない．
ヒートアップ(メインの実行の前の助走)を追加することも出来る．

### Warning
現代のjavascriptエンジンは多くの最適化を行う．それらは"人工的なテスト"の結果を"通常の使用"と比較して調整する可能性がある．非常に小さいものをベンチマークするときは特にそうである．
真面目にパフォーマンスを理解したいのであれば，javascriptエンジンの仕組みを学ぶ必要がある．
そして，マイクロベンチマークは全く必要ないことがわかる．

より高精度の時間計測が必要な時，javascript自身はマイクロ秒(100万分の1)での時間を計測する方法を持っていないが，ほとんどの環境はそれを提供している．例えば，ブラウザはマイクロ秒の精度でページ読み込み開始からのミリ秒を返す`performance.now()`を持っている．
Node.JSは`microtime`モジュールや他の方法を持っており，技術的にはどのデバイスや環境でも精度を上げることが出来る．

## JSON methods, toJSON
`JSON(JavaScript Object Notation)`は値とオブジェクトを表現する一般的な形式である．
* `JSON.stringify`: オブジェクトをJSONに変換する．
* `JSON.parse`: JSONをオブジェクトに変換する．
結果の`json`文字列はJSONエンコードされた，シリアライズされた(serialized)，文字列化された(stringified)，整列化された(marshalled)オブジェクトと呼ぶ．

JSONエンコードされたオブジェクトはオブジェクトリテラルと比べ，何点か重要な違いがある．
* 文字列にはダブルクウォートを使う．
* オブジェクトのプロパティ名もダブルクウォートであり，必須である．

JSONはデータのみのマルチ言語仕様なのでjavascript固有のオブジェクトプロパティの一部は`JSON.stringify`はスキップされる．
* 関数プロパティ(メソッド)
* シンボルプロパティ
* `undefined`を格納しているプロパティ
入れ子のオブジェクトもサポートされており自動的に変換される．

`JSON.stringify`の完全な構文は以下です．
`let json = JSON.stringify(value[, replacer, space])`

正規のJSONは簡単で信頼性があり，かつ非常に高速なパースアルゴリズムの実装を可能にするために厳格である．
