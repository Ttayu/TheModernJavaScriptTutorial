# Object properties configuration

## Property flags and descriptors

オブジェクトはプロパティを格納することが出来る．
今まで，"key-value"ペアであったが，実際にはオブジェクトプロパティはより柔軟で強力である．
オブジェクトプロパティには`value`の他に，3 つの特別な属性(フラグと呼ばれる)がある．

- `writable`: `true`の場合は変更可能である．それ以外は読み取り専用である．
- `enumerable`: `true`だとループで列挙される．それ以外は列挙されない．
- `configurable`: `true`の場合，プロパティを削除したり，これらの属性を変更することが出来る．
  一般的にはこれらは姿を見せることが少ない．"通常の方法"でプロパティを作成する時，これらはすべて`true`である．
  まずそれらのフラグを取得する方法を見てみる．

メソッド`Object.getOwnPropertyDescriptor`でプロパティの完全な情報を参照することが出来る．
`let descriptor = Object.getOwnPropertyDescriptor(obj, propertyName)`
返却値はいわゆる"プロパティディスクリプタ"オブジェクトと呼ばれる．

```javascript
let user = {
  name: "John"
};

let descriptor = Object.getOwnPropertyDescriptor(user, "name");

alert(JSON.stringify(descriptor, null, 2));
/*
 * {
 * "value": "John",
 * "writable": true,
 * "enumerable": true,
 * "configurable": true,
 * }
 * */
```

`object.defineProperty`を使うことでフラグを変更することが出来る．
もしプロパティが存在する場合，`defineProperty`はそのフラグを更新する．
そうでなければ，与えられた値とフラグでプロパティを作る．
もしフラグが指定されていなければ`false`と見なされる．

`configurable`は一方通行の処理であり，一度 false に設定するとそれを戻すことはできない．

一度に多くのプロパティが定義できるメソッドとして`Object.defineProperties`もある．
また，一度にすべてのプロパティのディスクリプタを取得するには`Object.getOwnPropertyDescriptors`もある．
通常，オブジェクトをクローンする時，プロパティをコピーするために代入を使うが，これはフラグはこ p-をしないので，`Object.defineProperties`を使うほうが"より良い"クローンが作成できる．
もう 1 つの違いは`for..in`はシンボルプロパティを無視するが，`Object.getOwnPropertyDescriptor`はシンボリックなものを含むすべてのプロパティディスクリプタを返す．

プロパティディスクリプタは個々のプロパティのレベルで動作する．
そこには，オブジェクト全体へのアクセスを制限するメソッドもある．

- `Object.preventExtensions(obj)`: オブジェクトにプロパティを追加するのを禁止する．
- `Object.seal(obj)`: プロパティの追加，削除を禁止し，既存のすべてのプロパティに`configurable: false`を設定する．
- `Object.freeze(obj)`: プロパティの追加，削除，変更を禁止し，既存のすべてのプロパティに`configurable: false, writeable: false`をセットする．
- `Object.isExtensible(obj)`: プロパティの追加が禁止されている場合に`false`を返す．
- `Object.isSealed(obj)`: プロパティの追加，削除が禁止されており，すべてのプロパティが`configurable: false`を持っている場合に`true`を返す．
- `Object.isFrozen(obj)`: プロパティの追加，削除，変更が禁止されており，すべての現在のプロパティが`configurable: false, writable: false`の場合に`true`を返す．
  これらのメソッドは実際にはめったに使われない．

## Property getters and setters

プロパティには 2 種類ある．最初のプロパティはデータプロパティで，これまで使ってきた全てのプロパティがデータプロパティである．
2 つ目はアクセサプロパティである．それらは基本的に値の取得や，セットする関数であるが，外部コードからは通常のプロパティのように見える．

アクセサプロパティは"getter"と"setter"メソッドで表現される．オブジェクトリテラルでは，`get`と`set`で表される．

```javascript
let obj = {
  get propName() {
    // getter, obj.propNameを取得する時にコードが実行される．
  },
  set propName(){
    // setter, obj.propName = value 時にコードが実行される．
  }
}
```

`obj.propName`が読まれた時に getter は動作し，setter は割り当てられた時に動作する．
例えば，`name`と`surname`を持つ`user`オブジェクトがある．

```javascript
let user = {
  name: "John",
  surname: "Smith",

  get fullName() {
    return `${this.name} ${this.surname}`
  }

  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  }
};
alert(user.fullName); // John Smith
user.fullName = "Alice Cooper"
alert(user.name); // Alice
```

今，"John Smith"という値を持つ"fullName"プロパティを追加したいとする．もちろん既存の情報のコピペはしたくないのでアクセサを使用してそれを実装する．
外部からはアクセスプロパティは通常の変数に見える．値に第に有する際には`set`を利用する．

アクセスプロパティは get/set でのみアクセス可能である．
プロパティは"データプロパティ"か"アクセサプロパティ"のいずれかになるが，両方共にはならない．
プロパティが`get prop()`または`set prop()`で定義されていると，それはアクセサプロパティであり，getter もしくは setter で読まなければならない．

アクセサプロパティのディスクリプタはデータプロパティと比べて異なる．
アクセサプロパティには，`value`も`wraitable`もないが，代わりに`get`と`set`がある．
したがってアクセサディスクリプタには次のようなものがある．

- `get`-引数なしの関数で，プロパティが呼ばれた時に動作する．
- `set`-1 つの引数を持つ関数で．プロパティがセットされた時に呼ばれる．
- `enumerable`, `configurable`: データプロパティと同じ
  もし，`get`と`value`を同じディスクリプタで指定しようとするとエラーになる．

Getter/Setter は実際のプロパティ値のラッパーとして使用することで，それらをより詳細に制御することが出来る．例えば，`user`で短すぎる名前を禁止したい場合，`name`を特別なプロパティ`_name`に格納する事ができる．そして`setter`でその値をフィルタする．

```javascript
let user = {
  get name() {
    return this._name;
  },

  set name(value) {
    if (value.length < 4) {
      alert("Name is too short, need at least 4 characters");
      return;
    }
    this._name = value;
  }
};

user.name = "Pete";
alert(user.name); // Pete
```

技術的には外部コードは`user._name`を使うことで，直接 name にアクセスできる．
しかしアンダースコアで始まるプロパティは内部のもので，外部のオブジェクトから触るべきではないということは広く知られている．

getter と setter の裏にある素晴らしいアイデアの 1 つは，それらは"通常の"データプロパティを制御し，それをいつでも調整することが出来ることである．
例えば，データプロパティ`name`と`age`を使って user オブジェクトを実装したとする．

```javascript
function User(name, age) {
  this.name = name;
  this.age = age;
}
let john = new User("John", 25);
alert(john.age); // 25
```

しかし，遅かれ早かれそれを変更することがある．より正確にするために，`age`の代わりに`birthday`を格納することに決めるかもしれない．

```javascript
function User(name, birthday) {
  this.name = name;
  this.birthday = birthday;
}

let john = new User("John", new Date(1992, 6, 1));
```

さて，まだ`age`プロパティを使っている古いコードはどうすればいいか．`age`自体は`user`が持っていても良いデータである．
`age`の getter を追加すると問題が緩和される．

```javascript
function User(name, birthday) {
  this.name = name;
  this.birthday = birthday;

  // ageは現在の日付と誕生日から計算される．
  Object.defineProperty(this, "age", {
    get() {
      let todayYear = new Date().getFullYear();
      return todayYear - this.birthday.getFullYaer();
    }
  });
}

let john = new User("John", new Date(1992, 6, 1));
alert(john.birthday);
alert(john.age);
```

これで古いコードも機能する．
