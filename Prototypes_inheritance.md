# Prototypes, inheritance

## Prototypal inheritance

プログラミングでは，何かを取ってそれを拡張することがしばしばある．
例えば，プロパティとメソッドを持つ`user`オブジェクトを持っているとする．
そしてそのいくつかをわずかに変更した`admin`や`guest`を作りたいとする．
コピーや再実装ではなく，単にその上に新しいオブジェクトを作成することで，`user`が持っているものを再利用したい．

javascript では，オブジェクトは特別な隠しプロパティ`[[Prototype]]`を持っており，それは`null`または別のオブジェクトを参照する．そのオブジェクトを"プロトタイプ"と呼ぶ．
`[[Prototype]]`は"魔法のような"意味を持っている．`object`からプロパティを読みたい時に，それが無い時，javascript では，自動的にプロトタイプからそれを取得する．プログラミングでは，"プロトタイプ継承"と呼ぶ．多くの言語機能やプログラミングテクニックではこれがベースになっている．
プロパティ`[[Prototype]]`は内部にあり隠されているが，セットする方法としていくつかある．

1 つは`__proto__`を使う方法である．

```javascript
let animal = {
  eats: true
};

let rabbit = {
  jumps: true
};

rabbit.__proto__ = animal;
```

`__proto__`は`[[Prototype]]`と同一ではない．これは getter/setter である．
もしも`rabbit`の中のプロパティを探し，それが無い場合，javascript は自動で`animal`から取得する．
これが`rabbit`がプロトタイプ的に`animal`を継承しているということになる．
実際には 2 つの制限がある．

1. 参照を循環させることはできない．
1. `__proto__`の値はオブジェクトまたは`null`になる．プリミティブのような，すべての値は無視される．
1. 1 つの`[[Prototype]]`しか存在しない．2 つ以上のものから継承できない．

プロトタイプはプロパティを読むためだけに使われる．
データプロパティ(getter/setter ではない)の場合，書き込み/削除の操作はオブジェクトで自動で動作する．
下の例では自身の`walk`メソッドを`rabbit`に割り当てている．

```javascript
let animal = {
  eats: true,
  walk() {
    /* このメソッドは rabbit では使われない */
  }
};

let rabbit = {
  __proto__: animal
};

rabbit.walk = function() {
  alert("Rabbit! Bounce-bounce!");
};
rabbit.walk(); // Rabbit! Bounce-bounce!
```

これ以降，`rabbit.walk()`はプロトタイプを使うこと無く，オブジェクトの中にすぐにメソッドを見つけそれを実行する．
getter/setter の場合，プロパティの読み書きをするとプロトタイプで参照されて呼び出される．

```javascript
let user = {
  name: "John",
  surname: "Smith",

  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  },
  get fullName() {
    return `${this.name} ${this.surname}`;
  }
};

let admin = {
  __proto__: user,
  isAdmin: true
};

alert(admin.fullName); // John Smith

admin.fullName = "Alice Cooper";
```

プロパティ`admin.fullName`はプロトタイプ`user`が getter を持っているのでそれが呼ばれ，setter もまた同様に呼ばれる．

`set fullName(value)`の内側の`this`の値は何になるのだろうか．プロパティ`this.name`と`this.admin`が書かれているのはどこか，答えはシンプルで`this`はプロトタイプの影響を全く受けない．
メソッドがどこにあるかは関係なく，オブジェクトの中でもそのプロトタイプ内でもメソッド呼び出しでは`this`は常にドットの前のオブジェクトである．
したがって，setter は実際に`this`として`admin`を使い，`user`ではない．

次に，継承されたオブジェクトの上でそれらのメソッドを実行し，大きなオブジェクトではなく継承したオブジェクトの状態を変更する．
例えば，ここでは`animal`は"メソッド格納域"を表現しており，rabbit はそれを使う
呼び出し`rabbit.sleep()`は`rabbit`オブジェクトに`this.isSleeping`をセットする．

```javascript
// animal がメソッドを持っている．
let animal = {
  walk() {
    if (!this.isSleeping) {
      alert(`I walk`);
    }
  },
  sleep() {
    this.isSleeping = true;
  }
};

let rabbit = {
  name: "White Rabbit",
  __proto__: animal
};

// rabbit.isSleeping を変更する．
rabbit.sleep();
alert(rabbit.isSleeping); // true
alert(animal.isSleeping); // undefined
```

もし，`bird`, `snake`など`animal`から継承された他のオブジェクトを持っていた場合，それらもまた`animal`のメソッドへのアクセスを得る．しかし，各メソッドでの`this`は対応するオブジェクトであり，`animal`ではなく呼び出し時に評価される．
結果として，メソッドは共有されるが，オブジェクトの状態は共有されない．

## F.prototype

最新の javascript では`__proto__`を使ってプロトタイプをセットすることができたが，以前はそうではなかった．
以前は別(であり唯一)の方法として，コンストラクタ関数の`"prototype"`プロパティを使うことである．

`new F()`は新しいオブジェクトを作る．`new F()`で新しいオブジェクトが作られる時，オブジェクトの`[[Prototype]]`は`F.prototype`にセットされる．言い換えると，もし`F`がオブジェクト型の値を持つ`Prototype`プロパティを持っている場合，`new`演算子は新しいオブジェクトに対してそれを`[[Prototype]]`にセットする．ここで，`F.prototype`は`F`上の`"prototype"`と名付けられた通常のプロパティを意味していることに注意する．

```javascript
let animal = {
  eats: true
};

function Rabbit(name) {
  this.name = name;
}

Rabbit.prototype = animal;

let rabbit = new Rabbit("White Rabbit"); //rabbit.__proto__ == animal

alert(rabbit.eats);
```

`Rabbit.prototype = animal`は`new Rabbit`が生成される時，その`[[Prototype]]`へ`animal`を割り当てることを意味している．

すべての関数は明示的に提供されていなくても，`"prototype"`プロパティを持っている．デフォルトの`"prototype"`は`constructor`というプロパティだけを持つオブジェクトで，それは関数自体を指す．

```javascript
function Rabbit() {}
/* デフォルト prototype
 Rabbit.prototype = { constructor: Rabbit };
 */
alert(Rabbit.prototype.constructor == Rabbit);
```

`constructor`プロパティを使って既存のものと同じコンストラクタを使って新しいオブジェクトを作成することが出来る．

```javascript
function Rabbit(name) {
  this.name = name;
  alert(name);
}
let rabbit = new Rabbit("White Rabbit");
let rabbit2 = new rabbit.constructor("Black Rabbit");
```

これはオブジェクトを持っているが，どのコンストラクタが使われたか分からない場合(例えば 3rd party library が使われているなど)で同じ種類のものを使って別のオブジェクトを作る必要がある場合に便利である．
しかし，`"constructor"`に関する最も重要なことは javascript は正しい`"constructor"`値を保証しないということである．
もし，デフォルトプロトタイプ全体を置き換えると`"constructor"`は無くなってしまう．
したがって，`"constructor"`を維持するためには，全体を上書きする代わりに，デフォルト`"prototype"`に対して追加/削除を行う．

```javascript
function Rabbit() {}
// 全体を置き換えている
Rabbit.prototype = {
  jumps: true
};

let rabbit = new Rabbit();
alert(rabbit.constructor === Rabbit); // false
// 単に追加しているだけでデフォルトのRabbit.prototype.constructorは保持される．
Rabbit.prototype.jumps = true;
// もしくは代替として手動でconstructorプロパティを作る．
Rabbit.prototype = {
  jumps: true,
  constructor: Rabbit
};
```

## Naitive Prototypes

`"prototype"`プロパティは javascript 自身のコア部分で広く使われており，すべての組み込みのコンストラクタ関数はそれを使っている．

空のオブジェクトを出力してみる．

```javascript
let obj = {};
alert(obj); // "[object Object]" ?
```

文字列`"[object.Object]"`を生成するコードはどこにあるのか．それは`Object.prototype`が`toString`メソッドを持っている．
それは以下のようにしても確認することが出来る．

```javascript
let obj = {};
alert(obj.__proto__ === Object.prototype); // true
```

`Array`, `Date`, `Function`のような，他の組み込みのプロトタイプもメソッドを保持している．
例えば，配列`[1, 2, 3]`を作る時，デフォルトの`new Array()`コンストラクタが内部で使われている．
なので，配列データは新しいオブジェクトに書き込まれ，`Array.prototype`はそのプロトタイプとなり，メソッドを提供する．これは非常にメモリ効率がいい．
仕様では全ての組み込みのプロトタイプは先頭に`Object.prototype`を持っている．なのですべては Object を継承している．

```javascript
let arr = [1, 2, 3];

// Array.prototypeから継承している？
alert(arr.__proto__ === Array.prototype); // true
// Object.prototype からは継承している？
alert(arr.__proto__.__proto__ === Object.prototype); // true
// トップの確認
alert(arr.__proto__.__proto__.__proto__); // null
```

プロトタイプのメソッドのいくつかは重複する可能性がある．例えば，`Array.prototype`はカンマ区切りで要素を表示する自身の`toString`を持っている．

```javascript
let arr = [1, 2, 3];
alert(arr); // 1, 2, 3 <- Array.prototype.toString
```

`Object.prototype`も同様に`toString`を持っているが，`Array.prototype`はチェーン内でより下位にあるので配列のバリアントが使われる．

Chrome developer console のようなブラウザ内のツールでも継承を表示できる．

他の組み込みオブジェクトも関数も同じように動作する．関数は組み込みの`Function`コンストラクタであり，`call/apply`は`Function.prototype`のメソッドである．

最も複雑なことは文字列や数値，ブール値などのプリミティブで起こる．それらはオブジェクトでは無く，プロパティへアクセスをしようとしたときに，組み込みのコンストラクタ`String`, `Number`, `Boolean`を使った一般的なラッパーオブジェクトが作られる．それはメソッドを呼んだあとに消える．
`String.prototype`, `Number.prototype`，`Boolean.prototype`として利用可能なものとしてプロトタイプに存在する．
値`null`と`undefined`はオブジェクトラッパーを持たず，利用可能なメソッドやプロパティは無い．

ネイティブプロトタイプは変更することが出来る．例えばあるメソッドを`String.prototype`に追加したときに，それはすべての文字列で使用可能に成る．

```javascript
String.prototype.show = function() {
  alert(this);
};

"BOOM!".show(); // BOOM!
```

しかしこれは一般的に悪い考え方で，プロトタイプはグローバルなため，コンフリクトを起こしやすいからである．
ネイティブプロトタイプの変更が認められるのはポリフィルのみである．
言い換えると，もし javascript エンジンがサポートしていない javascript 仕様上のメソッドがある場合，それを手動で実装し，組み込みのプロトタイプにそれを取り込むことが出来る．

```javascript
if (!String.prototype.repeat) {
  // プロトタイプに追加する．
  String.prototype.repeat = function(n) {
    return new Array(n + 1).join(this);
  };
}

alert("La".repeat(3));
```

既に，メソッドの借用については学んだ

```javascript
function showArgs() {
  // 配列からjoinを借り，引数コンテキストでそれを呼び出す．
  alert([].join.call(arguments, "-"));
}
showArgs("John", "Pete", "Alice");
```

`join`は`Array.prototype`にあるため，そこから直接呼び出すことが出来る．

```javascript
function showArgs() {
  alert(Array.prototype.join.call(arguments, "-"));
}
```

これは余分な配列オブジェクト`[]`を作らないのでより効率的である．

## Prototype methods, objects without **proto**

プロトタイプを操作する追加のメソッドを学ぶ

- `Object.create(proto[, descriptors])`-与えられた`proto`を`[[Prototype]]`として，また任意のプロパティディスクリプタで空のオブジェクトを作る．
- `Object.getPrototypeOf(obj)`-`obj`の`[[Prototype]]`を返す．
- `Object.setPrototypeOf(obj, proto)`-`obj`の`[[Prototype]]`に`proto`をセットする．

```javascript
let animal = {
  eats: true
};

// animal をプロトタイプとして新しいオブジェクトを作成する．
let rabbit = Object.create(animal);
alert(rabbit.eats); // true
alert(Object.getPrototypeOf(rabbit) === animal); // rabbitのプロトタイプを取得
Object.setPrototypeOf(rabbit, {}); // rabbitのプロトタイプを{}に変更
```

`Object.create`は任意の２つ目の引数を持っており，これはプロパティディスクリプタである．新しいオブジェクトに追加のプロパティを提供することが出来る．

```javascript
let rabbit = Object.create(animal, {
  jumps: {
    value: true
  }
});
alert(rabbit.jumps); // true
```

`for..in`でプロパティをコピーするよりも，よりパワフルにオブジェクトをクローンするために`Object.create`を使うことが出来る．

```javascript
// 完全に同一のobjの浅いクローン
let clone = Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
```

この呼出は列挙型，非列挙型，データプロパティ，setter/getter, `[[Prototype]]`のすべてのプロパティを含む`obj`の正確なコピーを作る．

`[[Prototype]]`を管理する方法は歴史的な理由によりたくさんある．

- コンストラクタ関数の`"prototype"`プロパティは非常に古くから
- 2012 年後半に`Object.create`が標準に登場．しかし，setter/getter が無かったため，ブラウザは非標準の`__prototype__`アクセサを実装した．
- 2015 年後半に`Object.setPrototypeOf`と`Object.getPrototypeOf`が標準に追加された．
  技術的にはいつでも`[[Prototype]]`の set/get が可能だが，通常はオブジェクト作成時に一度だけ設定を行い，変更はしない．また，javascript エンジンは高度に最適化されているので`Object.setPrototypeOf`や`obj.__proto__`でプロトタイプを変更することはとても遅い操作になる．

オブジェクトはキー/値ペアを格納するための連想配列として使える．
すべてのキーは`"__proto__"`を除いてうまく動作する．

```javascript
let obj = {};
let key = prompt("What's the key?", "__proto__");
obj[key] = "some value";
alert(obj[key]); // [object Object], "some value"ではない
```

`__proto__`プロパティは特別で，それはオブジェクトまたは`null`であり，文字列はプロトタイプになれない．しかしこの振る舞いは予期していないものでバグである．
もし連想配列としてオブジェクトを使いたい場合，リテラルのトリックを使ってそれを行うことが出来る．

```javascript
let obj = Object.create(null);
let key = prompt("What's the key?", "__proto__");
obj[key] = "some value";
alert(obj[key]); // "some value"ではない
```
`Object.create(null)`はプロトタイプなし`[[Prototype]]`が`null`の空オブジェクトを作る．
したがって，`__proto__`のために継承されたgetter/setterが無いので，通常のデータプロパティとして処理される．このようなオブジェクトを"非常にシンプルな"または"純粋な辞書オブジェクト"と呼ぶ．
欠点は，そのようなオブジェクトには組み込みのオブジェクトメソッドが無いことである．
ほとんどのオブジェクトに関連したメソッドは`Object.keys(obj)`のように`Object.something(...)`である．それらはプロトタイプにはないので，このようなオブジェクトで機能し続ける．

オブジェクトからキー/値を取得するには多くの方法がある．
プロトタイプを除いて，それらのほとんどはオブジェクト自体に作用する．
- `Object.keys(obj)/Object.values(obj)/Object.entries(obj)`-列挙可能な自身の文字列プロパティ名/値/キー値ペアの配列を返す．キーとして文字列を持つものだけをリストする．
- `Object.getOwnPropertySymbols(obj)`-すべての自身のシンボリックプロパティ名の配列を返す．
- `Object.getOwnPropertyNames(obj)`-すべての自身の文字列プロパティ名を返す．
- `Reflect.ownKeys(obj)`-すべての自身のプロパティ名の配列を返す．
これらのメソッドはすべてのオブジェクト自身で動作するが，プロトタイプからのプロパティはリストされない．

`for..in`ループは継承されたプロパティもループする．
```javascript
let animal = {
  eats: true
};

let rabbit = {
  jumps: true,
  __proto__: animal
}
```
