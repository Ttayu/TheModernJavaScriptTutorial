# Code quality

## Debugging in Chrome

`debugger;`コマンドによってコードを停止することが出来る．
[開発者ツールのマニュアル](https://developers.google.com/web/tools/chrome-devtools)

## Coding Style

## Comments

コードがコメントを必要とするほど不明瞭な場合，書き直すべきである．
関数自身がコメントになるため．これを自己記述的(self-descriptive)と呼ぶ．
一般的にはコードをシンプルで自己記述的に保つよう努めるべきである．
良いコメントとは

1. アーキテクチャの説明をする．
   全体的なアーキテクチャ，高水準の概説
   コードの俯瞰図．UML を利用するべき．
1. 関数の使用方法を文書化する．
   使用方法，パラメータ，返却値
   関数の目的を理解し，コードの中を見ることなく正しい方法で利用することが出来る．
1. 重要な解決策，特にそれが一目瞭然でないとき．
   なぜこのようなアルゴリズムで解決したか．
   書かれていないことが重要ではあるが避けられないこともある．
   コードがわかりにくく，紛らわしいものがある場合にはコメントを残しておく価値がある.

## Ninja code

過去のプログラマの ninja はコード管理者泣かせのトリックを使い，コードレビューに障害をもたらす．
反面教師として覚える.

### Warning

以下は悪いコードを書き込むルールである．

- 簡潔が肝心(Brevity is the soul of wit)
  できるだけコードを短くする．より短く書くことが常により良いとする．

```javascript
i = i ? (i < 0 ? Math.max(0, len + i) : i) : 0;
```

- 1 文字の変数(One-letter variables)
  より早くコードを書くための方法として，あらゆる場所で 1 文字の変数名を使うことである．
- 略語を使用する(Use abbreviations)
  1 文字や曖昧な名前の使用を禁止している場合，それらを短縮して略語を使う．
  - list → lst
  - userAgent → ua
  - brower → brsr
- 高く舞い上がる，抽象的になる．(Soar high. Be abstract)
  名前を選ぶとき，最も抽象な言葉を使う．`obj`, `data`, `value`, `item`, `elem`, etc
  - 変数の理想の名前は`data`. どこでもそれを使う
  - 変数をその型で命名する．`str`, `num`...
    値の型はデバッグで簡単にわかる．
    外部からこのコードを理解する時に全く情報がないことがわかる．
  - `data1, item2, elem5`...
- 注意力テスト(Attention test)
  本当に気が利くプログラマだけがそのコードを理解できる．
  1 つの方法として，似た変数名を使うことである．`data`, `date`のような
  更にタイポがあるとなお良い．
- スマートな同義語(Smart synonyms)
  同じものに対して類似の名前を使うようにする．
  関数が画面上に何かを表示する時に，`displayMessage`, `showName`, `render...`, `patin...`
  実際にはそこに違いはないのに関数間で微妙な違いがあるかのようにほのめかす．
- 名前の再利用(Reuse names)
  本当に必要なときだけ新しい変数を追加する．
  代わりに既存の名前を再利用し，新しい値をそこに書き込む．
  このアプローチの高度なパターンは，値をループや関数の途中でごっそり似たものに置き換えることである．
- 楽しみのためのアンダースコア(Underscores for fun)
  変数名の前に`_`, `__`を置く．特に意味はなく単に楽しむために追加する．
  異なる場所で異なる意味をもたせるのも良い．
  1 つ目はコードがより長くなり可読性が下がる．
  2 つ目はチームの開発者はアンダースコアの意味を知ろうとするために長い時間を費やす．
- あなたの愛を示す(Show your love)
  `superElement`, `megaFrame`, `niceItem`のような名前は読者を啓発する．
  このプリフィックスにはその詳細を何も示しておらず，読み手はその隠された意味を探すために時間をさく．
- 外部の変数と重ね合わせる(Overlap outer variables)
  関数の内側と外側で同じ変数名を使う．
- いたるところで副作用(Side-effects everywhere!)
  何も変えないように見える関数がある．`isReady()`, `checkPermission()`, `findTags`
  それらは副作用なしの関数である．
  本当に美しいトリックはメインの処理に加えて役立つアクションを追加することである．
  驚かせるためのもう 1 つの方法として標準ではない結果を返すことである．
  `checkPermission`で`true/false`ではなく複雑なオブジェクトを返す，など
- 強力な関数(Powerful functions!)
  その名前にかかれていることで関数を制限しない．
  `validateEmail`は email の正しさのチェックに加えて，エラーメッセージを表示し，email を再度入力することを要求する．
  複数のアクションを 1 つに結合すると，あなたのコードを再利用から守ることが出来る．

---

すべての**アドバイス**は実際の経験豊富な開発者により書かれていることもある．

## Automated testing with mocha

多くのタスクがある場合，自動テストが使用される．
なぜテストが必要なのか
関数を書く時に，通常それが何をすべきかをイメージできる．
どのパラメータがどのような結果を与えるか
開発中，関数を実行し，その結果と期待値を比較することで確認することが出来る．
何かが間違っていたらコードを直し，再度実行して結果を確認し，期待通り動くまで行う．
しかし，手動の**再実行**はいろいろなことを見逃すので十分ではない．
自動テストは実際のコードに加えてテストが別々に書かれていることを意味する．これにより，それらのテストは簡単に実行でき，すべての主要なユースケースをチェックすることが出来る．

### BDD(Behavior-driven development)

BDD にはテスト，ドキュメント，例の 3 つがある．
eg. "pow"の開発

#### 仕様

`pow`のコードを作成する前に，関数が何をすべきかを考える．
これを仕様，もしくはスペックと呼ぶ．

```javascript
describe("pow", function() {
  it("raises to n-th power", function() {
    assert.equal(pow(2, 3), 8);
  });
});
```

上記の通り，3 つの主要な構成要素を持っている．
`describe("title", function() { ... })`
何の機能を記述しているか，"ワーカー(it)"をグループ化するために使う．

`it("title", function() { ... })`
`it`の title は特定のユースケースを人間が読めるように記述し，2 つ目の引数はそれをテストするための関数である．

`assert.equal(value1, value2)`
実装が正しければエラーなく実行される．
関数`assert.*`は`pow`が期待通り動作するかをチェックするために使われる．

#### 開発フロー

1. 最も基本的な機能のテストと一緒に，初期仕様が書かれる．
1. 初期の実装がされる．
1. それが動くかを確認するため，テストフレームワークを実行する．エラーが表示されるとすべてが動作するまで修正を行う．
1. テストと一緒に，動作する初期の実装が出来上がる．
1. まだ実装でサポートされていない可能性のある，より多くのユースケースを仕様(test)に柄する．
1. 3 に戻り，テストのエラーが無くなるまで実装を更新する．
1. 機能が完成するまで 3-6 のステップを繰り返す．

#### アクションの仕様

- Mocha - コアフレームワーク，テストを実行するメインの機能を提供している．
- Chai - 多くのアサーションをもつライブラリ
- Sinon - 関数をスパイするためのライブラリで組み込み関数とそれ以上のものをエミュレートする．

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- add mocha css, to show results -->
    <link
      rel="stylesheet"
      href="https://cdnjs.cloudflare.com/ajax/libs/mocha/3.2.0/mocha.css"
    />
    <!-- add mocha framework code -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mocha/3.2.0/mocha.js"></script>
    <script>
      mocha.setup("bdd"); // minimal setup
    </script>
    <!-- add chai -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chai/3.5.0/chai.js"></script>
    <script>
      // chai has a lot of stuff, let's make assert global
      let asset = chai.assert;
    </script>
  </head>

  <body>
    <script>
      function pow(x, n) {
        /* function code is to be written, empty now */
      }
    </script>

    <!-- the script with tests (describe, it...) -->
    <script src="test.js"></script>

    <!-- the element with id="mocha" will contain test results -->
    <div id="mocha"></div>

    <!-- run tests! -->
    <script>
      mocha.run();
    </script>
  </body>
</html>
```

1. `<head>` テストのためのサードパーティライブラリやスタイルの追加
1. テストするための関数の`<script>`，このケースでは`pow`
1. テスト，`desctibe("pow", ...)`を持つ外部スクリプト`test.js`
1. Mocha が結果を出力するために HTML 要素`<div id="mocha">`を使う
1. テストは`mocha.run()`で開始される．
   `karma`のような高度なテストランナーもある．

#### 初期の実装，仕様を改善

仕様を改善する際にユースケースの追加が必要となる．
1 つのテストは 1 つのことを確認する．
そのため，2 つの独立したチェックがその中にある場合，分割して 2 つのシンプルなテストにするほうが良い．

```javascript
describe("pow", function() {
  it("2 raises to power 3 is 8", function() {
    assert.equal(pow(2, 3), 8);
  });
  it("3 raises to power 3 is 27", function() {
    assert.equal(pow(3, 3), 27);
  });
});
```

#### 実装を改善する

より多くの値をテストする．`it`ブロックを手動で書く代わりに`for`で生成することも出来る．

```javascript
describe("pow", function() {
  function makeTest(x) {
    let expected = x * x * x;
    it(`${x} in the power 3 is ${expected}`, function() {
      assert.equal(pow(x, 3), expected);
    });
  }

  for (let x = 1; x <= 5; x++) {
    makeTest(x);
  }
});
```

#### ネストされた記述

さらに多くのテストを追加する際に，他のテストでは`makeTest`は不要であり，`for`の中でのみ必要である，
グループ化は`describe`をネストすることでできる．

```javascript
describe("pow", function() {
  describe("raises x to power n", function() {
    function makeTest(x) {
      let expected = x * x * x;
      it(`${x} in the power 3 is ${expected}`, function() {
        assert.equal(pow(x, 3), expected);
      });
    }

    for (let x = 1; x <= 5; x++) {
      makeTest(x);
    }
  });
  // ... より多くのテストが続く
});
```
##### Information
`before/after`と`beforeEach/afterEach`を設定することが出来る．
通常初期化の実行のために使われ，カウンタをゼロにしたり，テストの間で何かをする時に使う．

#### 仕様を拡張する
数学的なエラーを示す方法として`n`の不正値(`NaN`)を返す．
BDDでは最初に失敗するテストを書き，次にそれらのための実装を作る．

---
仕様があると，安全に改善，変更，スクラッチで関数の書き直しですらもでき，それらが正しく動くことを確認できる．
関数が多くの場所で使われている場合，大規模なプロジェクトではそれが特に重要になる．
よくテストされたコードは，より良いアーキテクチャを持っている．
一般的にはテストを書くことは開発を早く，より安定させる．
## Polyfills
「最近の機能をサポートしていない古いブラウザで，その機能を使えるようにするためのコードである．」
### Babel
Babelはトランスパイラであり，最新のjavascriptコードを以前の標準仕様に書き直してくれる．
