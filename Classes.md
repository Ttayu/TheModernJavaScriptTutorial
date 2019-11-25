# Classes

## Class basic syntax

javascript には特別な構文とキーワード`class`がある．
javascript では`class`キーワードを使わずにクラスを作成する，いくつかのプログラミングパターンが有る．

### 関数クラスパターン

下のコンストラクタ関数は定義に従って"クラス"と考えることができる．

```javascript
function User(name) {
  this.sayHi = function() {
    alert(name);
  };
}

let user = new User("John");
user.sayHi(); // John
```

これは定義のすべてを満たす．

1. オブジェクトを作成する(`new`で呼び出し可能)ための"プログラムコードテンプレート"である．
1. 状態(パラメータから`name`)の初期値を提供する．
1. メソッド(`sayHi`)を提供する．
   これは**関数的なクラスパターン(functional class pattern)**と呼ばれる．
   関数的なクラスパターンでは，`this`に割り当てられていない`User`の中のローカル変数とネストされた関数は内側からは見えるが，外のコードからはアクセスすることができない．
   なので，簡単に内部関数や，変数を追加することができる．

```javascript
function User(name, birthday) {
  // User 内の他のメソッドからのみ見える．
  function calcAge() {
    return new Date().getFullYear() - birthday.getFullYear();
  }
  this.sayHi = function() {
    alert(name + ", age:" + calcAge());
  };
}

let user = new User("John", new Date(2000, 0, 1));
user.sayHi(); // John
```

このコードは変数`name`と`birthday`と関数`calcAge()`はオブジェクトに対しては内部，**private**である．
一方，`sayHi`は外部，**public**メソッドである．
このようにして，内部実装の詳細とヘルパーメソッドを外部コードから隠すことができ，`this`に割り当てられたものだけが外側に見えるようになる．

### ファクトリークラスパターン

`new`を全く使わずにクラスを作成することができる．

```javascript
function User(name, birthday) {
  // User 内の他のメソッドからのみ見える．
  function calcAge() {
    return new Date().getFullYear() - birthday.getFullYear();
  }
  return {
    sayHi() {
      alert(name + ", age:" + calcAge());
    }
  };
}

let user = User("John", new Date(2000, 0, 1));
user.sayHi(); // John
```

関数`User`はパブリックなプロパティとメソッドを持つオブジェクトを返す．
このメソッドの唯一のメリットは`new`を省略できることである．

### プロトタイプベースのクラス

これは最も重要で，一般的にベストな設計である．実際には上の 2 つはほとんど使用されない．
ここではプロトタイプを使って同じクラスを再度書く．

```javascript
function User(name, birthday) {
  this._name = name;
  this._birthday = birthday;
}

User.prototype._calcAge = function() {
  return new Date().getFullYear() - this._birthday.getFullYear();
};

User.prototype.sayHi = function() {
  alert(this._name + ", age:" + this._calcAge());
};

let user = new User("John", new Date(2000, 0, 1));
user.sayHi();
```

- コンストラクタ`User`は現在のオブジェクトの状態のみを初期化する．
- メソッドは`User.prototype`の追加されていく．
  メソッドは字句的に`function User`の中にはなく，共通のレキシカル環境を共有しない．もし，`function User`の内側で変数を宣言した場合，それらはメソッドには見えない．
  したがって，内部プロパティとメソッドはアンダースコア`"_"`が先頭に使いされているとい，広く知られている合意がある．`_name`や`_calcAge()`は技術的には単なる合意であり，外部からアクセスすることはできる．しかしながらほとんどの開発者は`"_"`の意味を理解しており，外部コードの中で prefix のついたプロパティやメソッドを触らないようにしている．

間接的なパターンと比較したときの利点は次のとおりである．

- 関数的なパターンでは，各オブジェクトにはすべてのメソッドの独自のコピーがある．コンストラクタで`this.sayHi = function(){...}`と他のメソッドの別のコピーを割り当てる．
- プロトタイプ的なパターンでは，すべてのメソッドはすべての`user`オブジェクトで共有される`User.prototype`にある．オブジェクト自身はデータだけを保持する．
  よってプロトタイプパターンはよりメモリ効率が良い．
  それだけでなく，プロトタイプを使うと継承を効率的にセットアップすることができる．
  すべての組み込みの javascript オブジェクトはプロトタイプを使っている．また，特別な構文構造がある．

### クラスのためのプロトタイプベースの継承

2 つのプロトタイプのクラスを持っているとする．

```javascript
function Rabbit(name) {
  this.name = name;
}

Rabbit.prototype.jump = function() {
  alert(this.name + " jumps!");
};

let rabbit = new Rabbit("My Rabbit");

function Animal(name) {
  this.name = name;
}

Animal.prototype.eat = function() {
  alert(this.name + " eats.");
};

let animal = new Animal("My animal");
```

今，この`Rabbit`と`Animal`は完全に独立している．
しかし，`Rabbit`は`Animal`を拡張させたものにしたい．言い換えると，rabbit は animal をベースとし，`Animal`のメソッドへのアクセスを持ち，自身のメソッドでそれを拡張する必要がある．

今，`rabbit`オブジェクトのメソッドは`Rabbit.prototype`にあります．`Rabbit.prototype`にメソッドが見つからない場合，"フォールバック"として`"Animal.prototype"`を使ってほしい．
なので，プロトタイプチェーンは`rabbit`→`Rabbit.prototype`→`Animal.prototype`である必要がある．

```javascript
// 以前と同じAnimal
function Animal(name) {
  this.name = name;
}

// すべての animals 食べることができる．
Animal.prototype.eat = function() {
  alert(this.name + " eats.");
};

// 以前と同じRabbit
function Rabbit(name) {
  this.name = name;
}
Rabbit.prototype.jump = function() {
  alert(this.name + " jumps!");
};

// 継承チェーンを設定します
Rabbit.prototype.__proto__ = Animal.prototype;

let rabbit = new Rabbit("White Rabbit");
rabbit.eat(); // rabbitsも食べることができる．
rabbit.jump();
```

`rabbit`はまず`Rabbit.prototype`を検索し，次に`Animal.prototype`を検索する．そして完全性のために，もしメソッドが`Animal.prototype`に存在しない場合，`Object.prototype`で検索を継続する．`Animal.prototype`は通常のオブジェクトなのでそれを継承する．

よって

1. メソッドは`Class.prototype`に格納される，
1. プロトタイプはお互いから継承する．

## Class inheritance

"class"構造は綺麗で見やすい構文でプロトタイプベースのクラスを定義することができる．
`class`構文は汎用性があり，最初にシンプルな例から始める．
以下はプロトタイプベースのクラス`User`である．

```javascript
function User(name) {
  this.name = name;
}

User.prototype.sayHi = function() {
  alert(this.name);
};
let user = new User("John");
user.sayHi();
```

そしてこれは`Class`構文を使った場合である．

```javascript
class User {
  constructor(name) {
    this.name = name;
  }

  sayHi() {
    alert(this.name);
  }
}

let user = new User("John");
user.sayHi();
```

クラス内のメソッドはそれらの間にカンマを持たないことに注意しておく．新米の開発者はときどきそれを忘れて，クラスメソッドの間にカンマをおいてしまい動作しなくなる．これはリテラルオブジェクトではなく，クラス構文である．
`class`は正確に何をするのか．ここで`class User {...}`は実際には 2 つのことをしている．

1. `"constructor"`という名前の関数を参照する変数`User`を宣言する．
1. その定義の中にリストされているメソッドを`User.prototype`の中に置く．ここでは`sayHi`, `constructor`である．

```javascript
class User {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    alert(this.name);
  }
}

// Userは"constructor"関数である．
alert(User == User.prototype.constructor); // ture
// その"prototype"には2つのメソッドがある．
alert(Object.getOwnPropertyNames(User.prototype)); // constructor, sayHi
```

従って，`class`はコンストラクタとプロトタイプメソッドを一緒に定義する特別な構文である．

コンストラクタは`new`を必要とする．
通常の関数とは異なり，クラス`constructor`は`new`なしで呼ぶことはできない．

```javascript
class User {
  constructor() {}
}
alert(typeof User); // function
User(); // Error: Class コンストラクタ Userは `new`なしで呼ぶことはできない．
```

もし，`alert(User)`のように出力すると，エンジンによって`"class User..."`や`"function User..."`と表示されるが，`User`は依然として関数であり，javascript 言語では別の"class"エンティティは無い．

クラスメソッドは非列挙型である．クラス定義は`"prototype"`の中のすべてのメソッドに対して`enumerable`フラグを`false`にセットする．オブジェクトを`for..in`したとき，通常クラスメソッドは出てきてほしくないので，これは良いことである．

クラスはデフォルトの`constructor() {}`を持っている．`class`構造の中に`constructor`が無い場合，空の関数が生成され，`constructor() {}`と書いたのと同じように動作する．

クラスは常に`use strict`である．クラス構造の内側のすべてのコードは自動的に strict モードである．

クラスは`getter/setter`も含む．以下はそれらを使って実装した`user.name`の例である．

```javascript
class User {
  constructor(name) {
    // setterを呼び出す
    this._name = name;
  }

  get name() {
    return this._name;
  }

  set name(value) {
    if (value.length < 4) {
      alert("Name too short.");
      return;
    }
    this._name = value;
  }
}

let user = new User("John");
alert(user.name); // John

user = new User(""); // Name too short.
```

内部的に，getter と setter もまた次のように`User`プロトタイプ上に作られる．

```javascript
Object.defineProperty(User.prototype, {
  name: {
    get() {
      return this._name;
    },
    set(name) {
      // ...
    }
  }
});
```

オブジェクトリテラルとは異なり，`class`の中で`property:value`割り当ては許可していない．メソッドと getter/setter のみである．その制限を緩和するために，仕様で進行中のものがいくつかあるがそれはまだない．
もしプロトタイプに非関数の値を置く必要が本当にある場合．`prototype`を手動で修正することができる．

```javascript
class User {}
User.prototype.test = 5;
alert(new User().test); // 5
```

従って，技術的にはそれは可能であるが，なぜそのようにしているかを知るべきである．このようなプロパティはクラスのすべてのオブジェクトの間で共有される．

"クラス内"の代替は getter を使うことである．

```javascript
class User {
  get test() {
    return 5;
  }
}
alert(new User().test); // 5
```

### クラス表現

関数と同様に，クラスは別の式の中で定義され，渡され，返される．
これはクラスを返す関数("クラスファクトリー")である．

```javascript
function makeClass(phrase) {
  // class を宣言しそれを返す．
  return class {
    sayHi() {
      alert(phrase);
    }
  };
}
let User = makeClass("Hello");
new User().sayHi(); // Hello
```

`class`はプロトタイプ付き関数の特別な形式であることを思い出すとこれは普通である．
また，名前付けされた関数表現のように，このようなクラスもまた名前を保つ場合がある．

```javascript
// 名前付けされたクラス式
let User = class MyClass {
  sayHi() {
    alert(MyClass);
  }
};

new User().sayHi(); // 動作する．MyClassの定義を表示する
alert(MyClass); // error, MyClassはclassの外からは見れない．
```

基本のクラス構文は以下のようになる．

```javascript
class MyClass {
  constructor(...) {
    // ...
  }
  method1(...) {}
  method2(...) {}
  get something(...) {}
  set something(...) {}
  static staticMethod(...){}
  // ...
}
```

`MyClass`の値は`constructor`として提供された関数であり，`constructor`が無ければ，空の関数である．
静的メソッドはクラスに結びつく関数が必要なときに使用されるが，そのクラスのオブジェクトに結びつく場合には使用されない．

## Static properties and methods

2 つのクラスがあるとする．
まず Animal

```javascript
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  run(speed) {
    this.speed += speed;
    alert(`${this.name} stopped.`);
  }
  stop() {
    this.speed = 0;
    alert(`${this.name} stopped`);
  }
}

let animal = new Animal("My animal");
```

そして Rabbit である，

```javascript
class Rabbit {
  constructor(name) {
    this.name = name;
  }
  hide() {
    alert(`${this.name} hides!`);
  }
}

let rabbit = new Rabbit("My rabbit");
```

現時点では，完全に独立している．しかし，`Rabbit`は`Animal`を拡張したものにしたい．
他のクラスから継承するには括弧`{..}`の前に`"extends"`と親のクラスを指定する．
ここでは`Rabbit`は`Animal`を継承している．

```javascript
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  run(speed) {
    this.speed += speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }
  stop() {
    this.speed = 0;
    alert(`${this.name} stopped.`);
  }
}

// "extends Animal"を指定してAnimalから継承する．
class Rabbit extends Animal {
  hide() {
    alert(`${this.name} hides!`);
  }
}

let rabbit = new Rabbit("White Rabbit");
rabbit.run(5);
rabbit.hide();
```

これで`Rabbit`は`Animal`のコンストラクタをデフォルトで利用するのでコードは少し短くなった．
そして`run`をすることもできる．
内部では`extends`キーワードは`Rabbit.prototype`から`Animal.prototype`へとの`[[Prototype]]`参照を追加している．

したがって，`Rabbit.prototype`にメソッドが見つからない場合，javascript は`Animal.prototype`から取る．

クラス構文では単にクラスではなく，`extends`の後に任意の式を指定することができる．
例えば，親クラスを生成する関数を呼び出す．

```javascript
function f(phrase) {
  return class {
    sayHi() {
      alert(phrase);
    }
  };
}
class User extends f("Hello") {}
new User().sayHi(); // Hello
```

ここでは，`class User`は`f("Hello")`の結果を継承している．
多くの条件に依存したクラスを生成するための関数を使用し，それらから継承できるような高度なプログラミングパターンに対して，これは役立つ場合がある．

### メソッドのオーバーライド

今の所．`Rabbit`は`Animal`から`this.speed = 0`をセットする`stop`メソッドを継承している．
もし，`Rabbit`で自身の`stop`を指定すると，代わりにそれが使われるようになる．

```javascript
class Rabbit extends Animal {
  stop() {
    // ... これは rabbit.stop() のために使われる．
  }
}
```

しかし，通常は親メソッドを完全に置き換えるのではなく，その上に組み立てて，その機能の微調整または拡張を行うことを望んでいる．
クラスはそのために`"super"`キーワードを提供する．

- `super.method(...)`は親メソッドを呼び出す．
- `super(...)`は親のコンストラクタを呼び出す．

例えば Rabbit が止まったときに自動で気に隠れさせる処理を行うとする．

```javascript
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }

  run(speed) {
    this.speed += speed;
    alert(`${this.name} runs with speed ${this.speed}.`);
  }
  stop() {
    this.speed = 0;
    alert(`${this.name} stopped.`);
  }
}

class Rabbit extends Animal {
  hide() {
    alert(`${this.name} hides!`);
  }
  stop() {
    super.stop();
    this.hide();
  }
}

let rabbit = new Rabbit("White Rabbit");

rabbit.run(5);
this.stop();
```

これで`Rabbit`は処理の中で親`super.stop()`を呼び出す`stop`メソッドを持っている．

アロー関数には`super`が無い．もしアクセスすると外部の関数から取得される．

```javascript
class Rabbit extends Animal {
  stop() {
    setTimeout(() => super.stop(), 1000);
  }
}
```

アロー関数での`super`は`stop()`での`super`と同じである．なので意図通りに動作する．
しかし，通常の関数を指定するとエラーになる．

```javascript
// Unexpected super
setTimeout(function() {
  super.stop();
}, 1000);
```

### コンストラクタのオーバーライド

コンストラクタには少し用心が必要である．
今まで`Rabbit`は自身の`constructor`を持っていなかった．
仕様によるとクラスが別のクラスを拡張子，`constructor`を持たない場合，次のような`constructor`が生成される．

```javascript
class Rabbit extends Animal {
  // 独自のコンストラクタを持たないクラスを拡張するために生成される．
  constractor(...args) {
    super(...args);
  }
}
```

基本的にはすべての引数を渡して親の`constructor`を呼び出す．それは自身のコンストラクタを書いていない場合にのみ起こる．カスタムのコンストラクタを`Rabbit`に追加してみる．

```javascript
class Animal {
  constructor(name) {
    this.speed = 0;
    this.name = name;
  }
  // ...
}

class Rabbit extends Animal {
  constructor(name, earLength) {
    this.speed = 0;
    this.name = name;
    this.earLength = earLength;
  }
  // ...
}
// 動作しない
let rabbit = new Rabbit("White Rabbit", 10); // Error!
```

これではエラーになる．継承したクラスのコンストラクタは`this`を使う前に`super(...)`を呼び出す必要がある．
`javascript`では継承しているクラスのコンストラクタ関数とそのすべてで区別がある．継承しているクラスでは，該当するコンストラクタ関数は特別な内部プロパティ`[[ConstructorKind]]: "derived"`がつけられる．
違いとして，

- 通常のコンストラクタを実行するとき，`this`として空のオブジェクトを作る．
- 派生されたコンストラクタが実行されると親のコンストラクタがこのジョブを実行することを期待する．
  なので，もし独自のコンストラクタを作っている場合には`super`を呼ぶ必要がある．なぜならそれを参照する`this`を持つオブジェクトが生成されず，結果としてエラーになるためである．
  `Rabbit`を動作させるには`this`を使う前に`super`を呼ぶ必要がある．

```javascript
class Rabbit extends Animal {
  constructor(name, earLength) {
    super(name);
    this.earLength = earLength;
  }
}
// これは動作する．
let rabbit = new Rabbit("White Rabbit", 10);
```

### Super: internals, [[HomeObject]]

`super`が技術的にどう動くかを見る．オブジェクトメソッドが実行されるとき．`this`として現在のオブジェクトを取り出す．もし，`super.method()`を呼び出す場合，どうやって`method`を取得しているのか．実際`this.__proto__.method`で`this`の`[[Prototype]]`からメソッドは取得できない．
クラス無しで簡単に確認してみる．

```javascript
let animal = {
  name: "Animal",
  eat() {
    alert(this.name + " eats.");
  }
};

let rabbit = {
  __proto__: animal,
  name: "Rabbit",
  eat() {
    // super.eat() が動作する．
    this.__proto__.eat.call(this);
  }
};

rabbit.eat(); // Rabbit eats.
```

これはプロトタイプから`eat`を取り，現在のオブジェクトコンテキストでそれを呼び出す．シンプルな`this.__proto__.eat()`は現在のオブジェクトではなく，プロトタイプのコンテキストで親の`eat`を実行するためである．
ここで，もう 1 つのオブジェクトをチェーンに追加する．

```javascript
let animal = {
  name: "Animal",
  eat() {
    alert(this.name + " eats.");
  }
};

let rabbit = {
  __proto__: animal,
  eat() {
    // 親メソッドを呼び出す．
    this.__proto__.eat.call(this); // (*)
  }
};

let longEar = {
  __proto__: rabbit,
  eat() {
    this.__proto__.eat.call(this); // (**)
  }
};
longEar.eat(); // Error: 最大呼び出しスタックサイズを超えている．
```
コードはエラーになります．`longEar`の呼び出しをトレースすると，行`(*)`と`(**)`はともに`this`の値は現在のオブジェクト(`longEar`)である．すべてのオブジェクトメソッドはプロトタイプなどではなく，現在のオブジェクトを`this`として取得する．
したがって，行`(*)`と`(**)`はともに`this.__protp__`の値は`rabbit`である．それらは両方共`rabbit.eat`を呼び出す無限ループにはまります．
1. `longEar.eat()`の中で`(**)`は`this=longEar`となる`rabbit.eat`を呼び出す．
1. 次に`rabbit.eat`の行`(*)`でチェーンの中でより高次へ呼び出しを渡そうとするが`this=longEar`なので，`this=__proto__.eat`は再び`rabbit.eat`になる．
1. したがって無限に自身を呼び出すことになる．

### [[HomeObject]]
この問題を解決するために，javascriptは1つの関数のために特別な内部プロパティ`[[HomeObject]]`を追加している．
関数がクラスまたはオブジェクトメソッドとして指定されたとき，その`[[HomeObject]]`プロパティはそのオブジェクト自身になる．
つまり，メソッドはそれらのオブジェクトを覚えているため
## Private and protected properties and methods

## Extending built-in classes

## Class checking: "instance of"

## Mixins
