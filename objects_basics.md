# Objects: the basics

## Objects

javascript には 7 つの型があり，6 つは"プリミティブ"型
オブジェクトは任意のプロパティの一覧から成る連想配列である．
プロパティは`key:value`のペアである．

```javascript
let user = new Object(); // "オブジェクトコンストラクタ"
let user = {}; // "オブジェクトリテラル"構文
let user = {
  name: "John",
  age: 30,
  "likes birds": true
};
user.isAdmin = true;
delete user.age;
user["likes birds"] = false;
```

末尾のカンマを置くことですべての行が同じ表記になるため，
プロパティの追加/削除/移動が簡単になる．
オブジェクトリテラルでは角括弧を使うことができ，それを算出プロパティと呼ぶ.

```javascript
let fruit = prompt("which fruit to buy?", "apple");
let bag = {
  [fruit]: 5 // プロパティ名はfruitから取られる．
};
alert(bag.apple);
```

角括弧はドット表記よりも強力で，任意のプロパティや変数を許容する．
しかし書くのはドットよりも面倒である．
プロパティ名を知っていて単純な場合はドットを使い，複雑な何かが必要なとき，角括弧に切り替える．
オブジェクトプロパティならプロパティ名に予約語を使っても良い．
歴史的な理由から`"__protp__"`だけにはセットすることができない．

プロパティを変数と同じ名前にするユースケースは非常に一般的である．
それを短く書くためにプロパティの短縮構文がある．

```javascript
function makeUser(name, age) {
  return {
    name, //name: name
    age // age: age
  };
}
```

整数値のプロパティはソートされ，それ以外は作成した順になる．

オブジェクトとプリミティブの基本的な違いの１つは，オブジェクトは"参照によって"格納，コピーされることである．
オブジェクト変数のクローンを作る javascript の組み込みのメソッドがないため多少難しい．
実際めったに必要ではなく，参照によるコピーは大抵の場合モンダイではないが，もし本当に必要なら以下のようになる．

```javascript
let clone = {};
for (let key in user) {
  clone[key] = user[key];
}
// cloneは完全に独立したクローン
clone.name = "Pete";
```

あるいは`Object.assign(dest[, src1, src2, ...])`を使うことが出来る．
`let clone = Object.assign({}, user);`
しかしプロパティに他のオブジェクトが入っている場合再帰的にクローンを作成する必要がある．
標準のアルゴリズムとして，`Structured cloning algorithm`がある．
javascript のライブラリの`lodash`に`_.cloneDeep(obj)`というメソッドがありそちらを使う．

## Garbage collection

javascript のメモリ管理の主要なコンセプトは*到達性*である.
到達可能な値は何らかの形でアクセス可能，または使用可能な値である．
参照または参照のチェーンにより，ルートから到達可能であれば到達可能とみなされる．
javascript エンジンには**ガベージコレクタ**と呼ばれるバックグラウンドプロセスがある．
すべてのオブジェクトを監視し，到達不可能となったオブジェクトを削除する．
基本のガベージコレクションのアルゴリズムは"マーク・アンド・スイープ"と呼ばれる．

- ガベージコレクタはルートを取得し，それらを"マーク"する．
- 次にそこからすべての参照へ訪れ，"マーク"する．
- 次にマークされたオブジェクトへアクセスし，それらの参照をマークする．訪問されたすべてのオブジェクトは記憶されているので，将来同じオブジェクトへは 2 度訪問しないようにする．
- 訪れていない参照があるまで行う．
- マークされたオブジェクトを除いたすべてのオブジェクトを削除する．

javascript エンジンは多くの最適化を適用して実行を高速化しているので，実行に影響を与えていない．

- 世代コレクション
  オブジェクトは新しいオブジェクトと古いオブジェクトの 2 つのセットに分割される．多くのオブジェクトは出現し，仕事をするとすぐ不要になる．これらは積極的にクリーンアップされる．
  長時間生き残ったものは"古い"ものになり，それほど頻繁には検査されない．
- インクリメンタルコレクション
  多くのオブジェクトがあるとき，1 度にすべてのオブジェクトの集合をマークしようとすると時間がかかり，実行時に目に見える遅延を引き起こす可能性がある．なのでエンジンはガベージコレクションを小さく分割し，それらを別々に実行する．
  変更を追跡するため，多少の追加のメモリを必要とするが，大きな遅延ではなく小さな遅延になります．
- アイドルタイムコレクション
  ガベージコレクタは CPU がアイドル状態の時にのみ実行を試み，実行への影響を減らす．

他にも最適化の方法はあるが，エンジンによってばらばらである．

ガベージコレクションは自動で実行され，実行を強制したり，防ぐことはできない．
オブジェクトは到達可能な間，メモリ上に保持される．
参照されることはルートから到達可能であることと同義ではなく，島ごと除去される可能性がある．

## Symbol type

オブジェクトのプロパティのキーは文字列型，もしくはシンボル型のいずれかである．
シンボル値はユニークな識別子を表現する．

```javascript
let id = Symbol();
// シンボル名を与えることができ，デバッグ目的で便利である．
let id = Symbol("id");
```

Symbols は文字列への自動変換はしない．
もし，シンボルを文字列に変換したい場合`id.toString()`とする．

シンボルを使うと，オブジェクトに"隠れた"プロパティを作ることが出来る．
他のコードがアクセスしたり，上書きをすることはできない．

```javascript
let id = Symbol("id");

let user = {
  name: "John",
  [id]: 123 // 単に"id: 123"ではない．
};
```

シンボリックなプロパティは`for..in`ループに参加しない．
一方で`Object.assign`は文字列とシンボルの両方をコピーする．

時には同じ名前のシンボルを同じエンティティにしたいときがある，
*グローバルシンボルレジストリ*がある．
その中でシンボルを作り，あとでそれにアクセスすることが出来る．
レジストリ内でシンボルを作ったり読み込むためには`Symbol.for(key)`を使う．
レジストリ内のシンボルは*グローバルシンボル*と呼ぶ．
コード内のどこにでもアクセス可能なアプリケーション全体のシンボルが必要な場合これを使う．

```javascript
// グローバルレジストリから読む
let id = Symbol.for("id"); // symbol が存在しない場合，新たに作られる．
// 再度読み込み
let idAgain = Symbol.for("id");
// 同じシンボル
alert(id === idAgain); // true
```

グローバルシンボルは名前によってシンボルを返すのではなく逆方向の呼び出しである`Symbol.keyFor(sym)`もある．
内部ではシンボルのキーを探すためにグローバルシンボルレジストリを使っているので非グローバルのものは動作しない．
非グローバルのとき，`undefined`を返す．

```javascript
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");
// symbol から name を取得
alert(Symbol.keyFor(sym)); // name
alert(Symbol.keyFor(sym2)); // id
```

他にも多くのシステムシンボルが存在する．

- Symbol.hasInstance
- Symbol.isConcatSpreadable
- Symbol.iterator
- Symbol.toPrimitive

技術的にはシンボルは 100%隠れているわけではなく，すべてのシンボルを取得する`Object.getOwnPropertySymbols(obj)`がある．
シンボリックなものも含めてオブジェクトのすべてのキーを返す`Reflect.ownKeys(obj)`がある．

## Object methods, "this"

*アクション*はプロパティの中で関数で表現される．
オブジェクトのプロパティの関数は*メソッド*と呼ばれる．

```javascript
// メソッド例
let user = {
  name: "John",
  age: 30
};

user.sayHi = function() {
  alert("Hello!");
};

user.sayHi(); // Hello!
```

エンティティを表現するのにオブジェクトを使ってコードを書くことを`object-oriented programming`と呼ぶ．

オブジェクトリテラルでは，メソッドのための短縮構文がある．

```javascript
let user = {
  sayHi() {
    // "sayHi: function()" と同じ
    alert("Hello");
  }
};
```

この表記は完全に関数宣言と同一ではなく，オブジェクトの継承に関して微妙に違いがある．
が，ほぼすべてのケースでこの短縮構文は好まれる．

オブジェクトメソッドが処理するために，オブジェクトに格納されている情報にアクセスする必要があることは一般的である．
オブジェクトにアクセスするために，メソッドは`this`キーワードを使うことが出来る．
技術的には`this`なしでもオブジェクトへのアクセスも可能であるが間違ったオブジェクトへアクセスすることがあるので`this`を使う．

`this`はどの関数にでも使えてしまう．
`this`の値は実行時に評価され，それは何にでも成る．

```javascript
let user = { name: "John" };
let admin = { name: "Admin" };

function sayHi() {
  alert(this.name);
}

// 2つのオブジェクトで同じ関数を使う．
user.f = sayHi;
admin.f = sayHi;

// これらの呼び出しは異なる this を持つ．
user.f(); // John
admin.f(); // Admin
```

オブジェクトまったくなしで関数を呼び出すことも出来る．

```javascript
function sayHi() {
  alert(this);
}
sayHi(); // undefined
```

このケースでは，strict モードでは`this`は`undefined`になる．
非 strict モードでは，このようなケースでは`this`の値はグローバルオブジェクトになる．
一般的にオブジェクト無しで`this`を使う関数の呼び出しは，普通ではなく，プログラム上の誤りである．

複雑なメソッドの呼び出しは`this`を失う可能性がある．
これは呼び出しの内側の`"this"`が`undefined`になるためである，

```javascript
let user = {
  name: "John",
  hi() {
    alert(this.name);
  },
  bye() {
    alert("Bye");
  }
};

(user.name == "John" ? user.hi : user.bye)(); // Error!

// メソッドの取得呼び出しを2行に分ける．
let hi = user.hi;
hi(); // Error, this は undefined なので
```

`hi = user.hi`は関数を変数の中においている，そのため最後の行は完全に独立している．なので`this`がない．
`user.hi()`呼び出しを動作させるために，javascript はトリックを使う．
ドットは関数ではなく特別な参照型を返す．
参照型は"仕様上の型"であり，明示的にそれを使うことはできないが，言語の中で内部的に使われています．
参照型の値は 3 つの値の組み合わせで`(base, name, strict)`である．

- `base`はオブジェクト
- `name`はプロパティ
- `strict`は`use strict`が効いている場合は`true`
  `user.hi`へのプロパティのアクセスの結果は，関数ではなく参照型であり，
  `(user, "hi", true)`となる．
  代入`hi = user.hi`のような操作は参照型を破棄し，`user.hi`(関数)の値を渡すので`this`を失うことになる．

アロー関数は特別で`this`を持たない．
もし`this`参照した場合，外部の"通常の"関数から取得される．
例えばここで`arrow()`は外部の`user.sayHi()`メソッドから`this`を使う．

```javascript
let user = {
  firstName: "Ilya",
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow();
  }
};

user.sayHi(); //Ilya
```

これは別の`this`ではなく，外側のコンテキストから取り出したい場合に便利である．

## Object to primitive conversion

オブジェクトの場合，すべてのオブジェクトは真偽値のコンテキストでは`true`になるため，真偽値の変換は必要ではない．
プリミティブが必要なとき，`toPrimitive`を使って変換することが出来る．
変換には`"string"`, `"number"`, `"default"`の 3 つ`hint`がある．
`"default"`は操作がどんな型を期待しているか"よくわからない"時に起こる．
変換するために javascript は 3 つのオブジェクトのメソッドを見つけ呼び出そうとする．

1. メソッドが存在する場合，`obj[Symbol.toPrimitive](hint)`を呼び出す．
1. hint が`"string"`であれば，`obj.toString()`と`obj.valueOf()`を試す．
1. hint が`"number"`であれば，`obj.valueOf()`と`obj.toString()`を試す．

```javascript
let user = {
  name: "John",
  money: 1000,

  [Symbol.toPrimitive](hint) {
    alert(`hint: ${hint}`);
    return hint == "string" ? `{name: "${this.name}"}` : this.money;
  }
};

// 変換動作の確認
alert(user); // hint: string -> {name: "John"}
alert(+user); // hint: number -> 1000
alert(user + 500); // hint: default -> 1500
```

`user`は変換に応じて文字列または金額になり，1 つのメソッドがすべての変換ケースを扱っている．

メソッド`toString`と`valueOf`は昔からあり，"通常の"文字列で名前付けされたメソッドである．

実際にロギングやデバッグ目的で"人が読める"オブジェクトの表現を返すような"あらゆる状況に対応できる"メソッドとしては，`obj.toString()`のみの実装で十分なことが多い．

## Constructor, operator "new"

コンストラクタ関数は技術的には通常の関数である．

1. 先頭は大文字で名前付けされる．
1. `"new"`演算子を使ってのみ実行されるべきである．

```javascript
function User(name) {
  this.name = name;
  this.isAdmin = false;
}
let user = new User("Jack");
```

1. 新しい空のオブジェクトが作られ`this`に代入される．
1. 関数本体を実行し，通常は`this`を変更し，それに新しいプロパティを追加する．
1. `this`の値が返却される．
   つまり`new User(...)`は次のようなことをする．

```javascript
function User(name) {
  // this = {}; (暗黙)

  // this へプロパティを追加
  this.name = name;
  this.isAdmin = false;

  // return this; (暗黙)
}
```

再利用可能なオブジェクト作成のコードを実装すること，それがコンストラクタの主な目的である．
技術的にはどのよな関数でもコンストラクタとして使うことが出来る．
"先頭が大文字"は関数が`new`で実行されることを明確にするためである．

1 つの複雑なオブジェクトの作成に関する多くのコードが有る場合コンストラクタでそれをラップすることが出来る．

```javascript
let user = new function() {
  this.name = "John";
  this.isAdmin = false;

  // ...ユーザ作成のための他のコード
  // 複雑なロジック，文やローカル変数などを持つかもしれない．
}();
```

コンストラクタはどこにも保存されず，単に作られて呼び出されただけなので 2 度は呼び出せない．
なのでこれは将来再利用すること無く，単一のオブジェクトを構成するコードをカプセル化することを目指している．

関数の中では`new.target`プロパティを使うことで，それが`new`で呼ばれたかそうでないかを確認することが出来る．
これは`new`の場合と，通常呼び出し両方の構文が同じように動作するようにするっために使用できる．

```javascript
function User(name) {
  if (!new.target) {
    return new User(name);
  }
  this.name = name;
}
let john = User("John"); // new User へのリダイレクト
```

このアプローチは構文をより柔軟にするためにライブラリの中で使われることがある．
だが，おそらくどこへでもこれを使うのは良いことではない．
`new`を省略すると，何をしているのかが少しわかりにくくなるため，新しいオブジェクトが作られるときは必ず`new`をつける．

通常コンストラクタは`return`を持たないが，もしあった場合,

- もし`return`がオブジェクトと一緒に呼ばれた場合`this`の代わりにそれを返す．
- もし`return`がプリミティブと一緒に呼ばれた場合，それは無視される．

```javascript
function BigUser() {
  this.name = "John";
  return { name: "Godzilla" }; // オブジェクトを返す．
}
alert(new BigUser().name); // Godzilla
```
通常コンストラクタはreturn文を持たない．主に完全性のためにオブジェクトを返す特殊な動作である．

オブジェクトを作るとき，コンストラクタ関数を使用することで高い柔軟性を得ることが出来る．
`this`にプロパティだけでなく，同様にメソッドを置くことが出来る．

