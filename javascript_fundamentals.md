# javascript Fundamentals

## Hello world

`<script>`タグを使用して HTML の任意の部分に挿入することができる.
外部スクリプトを利用する際は, `<script src="filename.js>"`とする．

### Information

原則として，最もシンプルなスクリプトだけを HTML 内に置く．
ファイル分割のメリットはブラウザがスクリプトをダウンロードしてそれをブラウザのキャッシュに保存するため．
同じスクリプトが必要な他のページはダウンロードする代わりにキャッシュから取得する．
そのため，ファイルは実際には 1 度だけ保存される．

### Warning

1 つの`<script>`タグは`src`属性と中身のコード両方を持つことはできない．
2 つのスクリプトに分ける必要がある．

```html
<script src="file.js">
  alert(1) // srcが設定されているのでこれは動作しない．
</script>
```

## Code structure

改行によって文が分割されるとしてもセミコロンを置く．
コメントはコードの規模を増加させるが全くの問題ではない．プロダクションサーバーへリリースする前にコードを minify する多くのツールが有り，それを使うことによってコメントを除去することが出来るので実行されるスクリプトには現れない．

## The modern mode, "use strict"

javascript は互換性の問題なしに進化してきた．**後方互換性**を守るため，問題のある挙動を廃止したくても後方互換性が崩れてしまうためできない．
スクリプトの一番先頭に明示的に`"use strict";`を有効することで，問題のある挙動を廃止することができる．
文字列で表現されているのもブラウザの後方互換性を保つためである．
常に`"use strict"`で始まるスクリプトを推奨する．

### Warning

`"use strict"`がスクリプトの先頭にあることを保証する必要がある
一度 strict mode に入ると`"use strict"`をキャンセルすることができない．

## Variables

一行で複数の変数を宣言することは出来るが，可読性のために 1 変数 1 行にする．
一般的には変数名には**camelCase**が使われる．
ハードコーディングされた定数は**UPPER_CASE**が使われる．
実行時に評価される定数には変数と同じく**camelCase**が用いられる．

## Data types

1. number
1. string
1. boolean
1. null
1. undefined
1. object
1. symbol
   typeof でどの型が格納されているかを知ることが出来る．
   typeof(null)で object 型を返すがこれは言語の既知のエラーで実際には object ではない．

## Type Conversions

### Warning

javascript ではゼロの文字列`"0"`や`" "`を boolean に変換すると`true`である．

## Operators

## Comparisons

通常の等価演算子`==`は両辺の変数を数値に変換してから等価かどうかをチェックする.
そこで厳密等価演算子`===`を使って型変換無しで等価をチェックする．
厳密な等価`===`以外の比較演算子(`>=, >, <, <=`)については例外的な注意を払い，`undefined/null`の比較を行う必要がある．
変数が`undefined/null`を持つ可能性がある場合には，それらを別々にチェックする必要がある．

## Interaction: alert, prompt, confirm

ブラウザ固有の interaction な関数

- alert(message)
  メッセージのある小さいウィンドウを*モーダルウィンドウ*という．
  モーダルはそのウィンドウを処理するまで他のボタンを押すことができないことを意味する．
- prompt(title, [, default])
  OK/CANCEL ボタンを持つ小窓を表示する
  default は入力フィールドの初期値である．
- result: boolean = confirm(question)
  OK/CANCEL の 2 つのボタンを持つモーダルウィンドウを表示する．

## Conditional operators: if, '?'

たとえ 1 つの文しかない場合でも if を使用するときは可読性のために波括弧でコードブロックを使用することを推奨する．

## Logical operators

## Loops: while and for

break/continue のためのラベル

```javascript
for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    let input = prompt(`Value at coords (${i}, ${j})`, "");
    // ここで終了して下にあるDoneに移動したい
  }
}
alert("Done!");
```

```javascript
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    let input = prompt(`Value at coords (${i},${j})`, "");

    // 文字から文字またはキャンセルされた場合、両方のループから抜ける
    if (!input) break outer; // (*)

    // 値に何かをする処理...
  }
}
alert("Done!");
```

ラベルは`goto文`ではなく，コードの任意の位置に移動できるわけではない．
`break/continue`の移動はループの中からのみ可能

## The "switch" statement

## Functions

古いバージョンだとデフォルト引数がサポートされていなかったので
以下のような方法で実現されていた.

```javascript
function showMessage(from, text) {
  // type 1
  if (text === undefined) {
    text = "no text given";
  }
  // type 2
  text = text || "no text given";
}
```

関数が値を返却しない場合，`undefined`を return したことと同値
1 つの関数で明確に 1 つのアクションを行う．

## Function expressions and arrows

ask の引数をコールバック関数または単にコールバックと呼ぶ.
関数を渡し，あとで呼び戻すことを期待しているため．

```javascript
function ask(question, yes, no) {
  if (confirm(question)) yes();
  else no();
}

ask(
  "Do you agree?",
  function() {
    alert("You agreed.");
  },
  function() {
    alert("You canceled the execution.");
  }
);
```

関数宣言はスクリプト/コードブロック全体で使用できる．
コードブロックが実行する前に処理される.

```javascript
sayHi("John"); // Hello, John

function sayHi(name) {
  alert(`Hello, ${name}`);
}
```

関数式はその式が実行された時に作られ，それ以降で利用可能になる．

```javascript
sayHi("John"); // エラー!

let sayHi = function(name) {
  // (*) no magic any more
  alert(`Hello, ${name}`);
};
```

コードの中で関数式よりも関数宣言のほうが調べるのが少し簡単であり，コードを体系化する自由度が増す，
たいていのケースでは関数の宣言が望ましい．コード構成の柔軟性が増し，通常は読みやすくなるため．
関数宣言がそのタスクに適さないときのみ，関数式を使うべきである．

関数を作成するための方法としてアロー関数がある．
より非常にシンプルで簡潔な構文であり，関数式よりも優れています．
多くの文字を書くのが面倒なとき，シンプルなワンライナーの処理を書くのにとても便利である．

```javascript
let sum = (a, b) => a + b;
let double = n => n * 2;
let sayHi = () => alert("Hello!");
let sum = (a, b) => {
  let result = a + b;
  return result;
};
```
