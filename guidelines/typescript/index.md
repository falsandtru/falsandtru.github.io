---
layout: bootstrap
title: TypeScript Coding Guidelines
type: page
nav: pages
---

# TypeScript Guidelines

## Types and Values

### Fuzzy types

以下の型はすべてまたはほとんどの型が代入可能となり型制約として機能しない場合があるため注意を要する。

- `void`
- `{}`
- `Object`

```ts
function f<T extends Object>(p: T, cb: (p: {}) => void) {
}
f<number>(0, (n: number) => 0);
```

### `void` type

void型に定義された値を得た場合はその値を使用せず直ちに破棄する。
void型は他の型を代入可能な経路があるため信用できない。

```ts
function f(): void {
  return void g();
}
function g(): void {
}
```

### `never` type

never型の変数および仮引数宣言への使用を禁止する。

### `undefined`

undefined変数その他一切の格納されたundefined値は未定義状態の定義および即時評価以外の使用を禁止する。

- 生成：可能
- 書込：未定義状態を定義する場合のみ可能
- 読込：評価のみ可能、外部への搬出不可

```ts
function f(n?: number): void {
  n = isNaN(n) ? undefined : n;
  n = n === undefined ? 0 : n;
  return n;
  }
}
```

### `null`

nullの使用をnullの検出とObject.create(null)などnullが必須である言語または外部機能を使用する場合を除き禁止する。

### String literals

文字列の生成は原則としてシングルクオートおよびバッククオートを使用し、ダブルクオートを使用しない。
シングルクオートをエスケープする場合はダブルクオートを使用してよい。
その他JSONやHTMLなど必要に応じて使い分ける。

### Big number

10進数で15桁(正確には2^53)を超える整数は精度が保証されないため使用禁止とし、文字列化して管理する。

```ts
JSON.parse('{"id": 9999999999999999}'); // Object {id: 10000000000000000}
```

ビット演算可能なのは10進数で9桁まで。

```ts
999999999.9|0; // 999999999
~~999999999.9; // 999999999
9999999999|0; // 1410065407
~~9999999999; // 1410065407
```

### Enums

Enumは特に必要のない限りストリングリテラルタイプで置換する。

## Operators

### Equality operators

厳密等価演算子のみ使用する。
等価演算子の使用を禁止する。

```ts
null === null;
```

### Increment and Decrement operators

必要のない限り常に前置記法で記述する。

```ts
++n;
--n;
```

### Operand order of binary operators

二項演算子のオペランドは、actual > expectedの順に記述する。

```ts
if (typeof n === 'number') {
}
```

### Conditional operators across multiple lines

複数行にわたる三項演算子は、各演算子の前で改行しインデントを1レベル下げる。
`return`文とともに使用する場合は常に複数行で記述する。

```ts
function f() {
  return expr
    ? true
    : false;
}
```

### Disallow `in` operator

自身のメンバーの有無はメンバーに直接アクセスして確認し、必要のない限り`in`演算子を使用しない。
オーバーヘッドが非常に大きい。

- [https://falsandtru.github.io/benchmark/suites/2/](https://falsandtru.github.io/benchmark/suites/2/)

```ts
if (obj.member === void 0) {
}
```

### Indents with closing parentheses

閉じ括弧は末尾の要素と同じ行に続ける。

```ts
f(() =>
  g(() =>
    h()));
```

## Statements and Expressions

### Union/Intersection type declarations across multiple lines

複数行にわたるUnion/Intersection型の宣言は以下のフォーマットを使用する。

```ts
type union =
  | A
  | B
  | C;
```

### Variable definition

`const`および`let`のみ使用する。
`var`の使用はアルゴリズムまたはES5の制約で必要な場合のみに止める。

### Selective assignment

複数の候補からTruethyな値を走査して代入する場合は以下のフォーマットを使用する。

```ts
const a = undefined
  || f()
  || g()
  || h();
```

### Unused variables

不要な変数をアンダースコアで宣言する。
関数に2つ以上省略できない引数がある場合は分割代入を利用して宣言を省略する。
2つ以上省略できない仮引数が生じる状況自体が基本的に不適切であるため空の分割代入はそのアノテーションとみなす。

```ts
function f([, , x], _, [], {}, y) {
  return [x, y];
}
f([1, 2, 3], 0, [], NaN, 4);
const g = _ => 0;
g(NaN);
```

### String literal types as Enums

Enumとしてのストリングリテラルタイプは以下のフォーマットで記述する。
String Enumでよい場合はそちらを使う。

```ts
type Type =
  | Type.put
  | Type.del
  | Type.get;
namespace Type {
  export type put = 'put';
  export const put: put = 'put';
  export type del = 'del';
  export const del: del = 'del';
  export type get = 'get';
  export const get: get = 'get';
}
```

予約文字を含めたい場合は全体をオブジェクトとして定義すれば可能。
ただし型空間での個別の型定義はできない。

```ts
const Type = {
  put: <'put'>'put',
  delete: <'delete'>'delete',
  get: <'get'>'get'
};
type Type =
  | typeof Type.put
  | typeof Type.delete
  | typeof Type.get;
```

### Early returns

関数的記述を心がけ、ブロックをreturnで開始させるよう努める。

### Do-like parentheses

丸括弧を利用して冗長なブロックを削除する。

```ts
[['a', 1], ['b', 2]]
  .reduce((m, [k, v]) => (m[k] = v, m), {});
```

### Evaluations of conditional expressions

条件式で評価してよい値の組み合わせは以下の2つのパターンのみとする。

#### true/false

比較演算、論理演算または評価関数の結果である真理値。

```ts
if (arr.indexOf(n) > -1) {}
if (flag === false && !flag) {}
if (is()) {}
```

#### Object/(undefined|null)

オブジェクト（非プリミティブ値）の有無。

```ts
if (err) {}
if (elem.parent) {}
```

### Line breaks of multiple block statement

複数ブロックを持つ構文はブロックの終了ごとに改行する。

```ts
if (expr) {
  f();
}
else {
  g();
}
```

### Guard statements by one-liner non-brace if statements

ガード節のためのブロックを明示しないif文を以下の条件をすべて満たす場合のみ許可する。

- ワンライナーである。
- elseおよびelse if文を持たない。
- `return`, `continue`, `break`, `throw`といったコードフローを制御する予約語で開始する。

```ts
function f() {
  if (!expr) return;
}
```

### Case clauses of switch statements

switch文のcase句の文がコードフローを制御する予約語で開始されない場合は常にブロック化する。

```ts
switch (n) {
  case 0:
    return 0;
  default: {
    const m = 1;
    return n * m;
  }
}
```

### Switch statements being with other statements

switch文の他の文との混在を非推奨とする。
if文に置換または関数に切りだすべきものである場合が多い。

### Annotation of command calls **[experimental]**

副作用のある命令的関数の呼びだしにvoid演算子を付与して副作用のある命令の発行に注釈をつける。
コード上に存在する副作用がvoid演算子によりマークアップされ可視化される。
代入操作に注釈をつけることができないのが欠点。
この項目は単純なスクリプトなど設計のないコードには適用しなくともよい。

```ts
void ++count;
void ns.push(1);
void fs.pop()();
```

### Promise

Promiseの失敗文脈では非同期の例外すなわち異常系のみ扱い、既定ないし回復可能な失敗および例外は多値の成功文脈すなわち正常系で扱う。
失敗文脈は原則として回復不能な例外のみを扱う異常系として設計しなければならない。
Promiseの失敗文脈は任意の失敗と予期せぬ例外を分離できないため基本的に失敗文脈として使用できない。
失敗文脈をそれとして使用する場合は絶対に予期せぬ例外が混入しないよう信頼性を高めなければならない。
このためPromise値を返し失敗文脈を持つ組み込み関数を単に模倣して失敗文脈を持つ関数を作ることは誤りである。
よって原則として組み込み関数のような最小単位の関数以外で失敗文脈を使用するべきではない。

多値表現にはタプルを使用し最初の要素に成否判定のフラグを入れるフォーマットを推奨する。

```ts
new Promise(resolve => resolve([arr.length > 0, arr.pop(), arr]));
```

### async/await

async/awaitはPromise同様失敗を多値の成功文脈で表現し、失敗文脈が例外のみ扱うように設計することでtry-catch文が不要となるよう設計する。
try-catch文によりawait式の失敗文脈を捕捉する場合、tryブロックにはawait式のみを入れ、同期的処理の例外を混入させない。
同期処理において捕捉しない例外を非同期処理に限って捕捉しなければならない理由はない。

```ts
await sleep(1);
var n = 100;
try {
  await sleep(n);
}
catch (err) {
  throw err;
}
```

## Functions and Objects

### Function definition order

関数は粒度（目的としての抽象度）の大きい順に定義する。

```ts
f();

function f() {
  g();
}
function g() {
}
```

### Closure definition positions

クロージャは`return`文の後ろに1行空けて記述する。

```ts
function f() {
  return g();

  function g() {
  }
}
```

ループの中のような極めて短い間隔で繰り返し実行される場所およびそこで呼ばれる関数内でクロージャを作らない。
関数定義はコストが高い。

```ts
function f() {
  return g();
}
function g() {
}
```

### Adapter functions and methods definition

関数の役割が実体である別の関数の事前処理または事後処理、あるいはインターフェイスの隠蔽であり、実体関数が他の関数から使われない場合、自身のクロージャとして実装する。
クロージャの名前が親と同じである場合は名前の先頭または末尾にアンダースコアを追加ないし削除して名前を変える。
メソッドにおいても同様とする。

```ts
function f() {
  return f_(1);

  function f_(cnt: number) {
    return cnt > 10
      ? void 0
      : f_(cnt + 1);
  }
}
```

パフォーマンスを要求される場合は逆に外部へ露出する。

```ts
function f() {
  return f_(1);
}
function f_(cnt: number) {
  return cnt > 10
    ? void 0
    : f_(cnt + 1);
}
```

### Arrow functions across multiple lines

ブロックを作らない複数行にわたるアロー関数は、アローの後で改行しインデントを1レベル下げる。
アロー関数から通常関数に変更しても末尾のreturn文の追加以外関数本体が変更されない。

```ts
dispatch(ns =>
  ns
    .filter(n => !isNaN(n))
    .map(parseInt);
);
```

```ts
dispatch(ns => {
  return ns
    .filter(n => !isNaN(n))
    .map(parseInt);
});
```

### Destructuring parameter declaration

引数のオブジェクトの一部の要素しか使わない場合、引数にとる時点で分解して使用する要素のみ宣言する。
意図しない要素に触れられる余地を作らない。

```ts
function f({textContent, innerText}: HTMLElement) {
  return textContent || innerText || '';
}
```

### Indents of function parameters

呼び出し関数と同じ行で複数行にわたる関数をパラメータとして渡す場合、以降のパラメータは改行せず同じ行に記述する。
このとき関数本体は常に何らかの括弧で囲み垂直アラインをそろえる。

```ts
[]
  .reduce(b => {
    return b;
  }, 0);
[]
  .reduce(b => [
    b
  ], 0);
[]
  .reduce(b => (
    b
  ), 0);
```

呼び出し関数とパラメータを改行して分ける場合は通常通りコロンを行末とする。

```ts
[]
  .reduce(
    b =>
      b,
    0);
[]
  .reduce(
    b => {
      return b;
    },
    0);
```

### Function return value types

関数の戻り値は原則としてプリミティブ型をはじめとするビルトインタイプおよびパラメーターの型のみとする。
メソッドの戻り値は関数で許可される型および自身の型のみとする。
void型はこの規則に則った戻り値を返せない場合のデフォルト型とする。
オブジェクトの戻り値はデータ構造やデザインパターンなど明確な規範に則って設計し、戻したいデータを積み上げただけのオブジェクトを返すことは控える。

```ts
function f(): boolean {
  return true;
}
class C {
  dispatch(): C {
    return this;
  }
}
```

### Side effect only function return types

コールバック関数等で戻り値に関心がなく型に制約が不要な場合でも戻り値の型にanyでなくvoidを使う。
誤って戻り値を受けた場合のany型の混入を防ぐ。

```ts
function f(cb: () => void): boolean {
}
```

### Command Query Segregation

コマンドの戻り値は原則として空、個別の実行結果の識別子ないし操作方法のいずれかに限る。
自身に破壊的変更を加えながら自身を返すメソッドは不適切である。
副作用を持つ操作が深いスタックを持ち垂直方向に複雑化すると副作用の管理が困難となるため副作用を持つ操作は可能な限りネストさせず浅い操作にしなければならない。
コマンドに副作用を連鎖させる戻り値を禁止する制約は副作用の複雑化を水平方向に止め垂直的複雑化を抑止する。
なお不変オブジェクトを返すメソッドチェインは一般的に副作用を持たないためこの制約対象に該当しない。

### Conditional expressions across multiple lines in return statement

複数行にわたり末尾のreturn文に条件分岐を持たせる場合は以下のフォーマットを使用する。
このフォーマットで表現できない複雑な条件式の使用は基本的に不適切である。

#### Conditional expressions

```ts
function f() {
  return expr
    ? 0
    : 1;
}
```

#### Logical operators

```ts
function f() {
  return 1
      && 2
      && 3;
}
function g() {
  return 1
      || 2
      || 3;
}
function h() {
  return 1
      || 2 && 2.1
      || 3 && 3.1;
}
```

### Order of branch on condition

条件分岐は基本状態を最初に記述し、変化形を変化率の小さい順に記述していく。
アルゴリズムの条件分岐は停止条件を最初に記述し、その他の条件を最後に記述する。

```ts
function fib(n: number): number {
  switch (n) {
    case 0:
      return 0;
    case 1:
      return 1;
    default:
      return fib(n - 1) + fib(n - 2);
  }
}
```

### Ad-hoc polymorphic functions

アドホック多相関数は単一のswitch文で関数を満たしてcase句で多相性を表現する。
共通パラメーターなどの宣言的定義はswitch文のほかに関数に含めてよい。

```ts
function f(str: string): string
function f(num: number): string
function f(p) {
  switch (typeof p) {
    case 'string':
      return p;
    case 'number':
      return p + '';
    default:
      throw new TypeError('Invalid type parameter.');
  }
  assert(false);
}
```

単純なものは最初のステートメントでreturnして三項演算子で表現する。

```ts
function f(str: string): string
function f(num: number): string
function f(p) {
  return typeof p === 'string'
    ? p
    : p + '';
}
```

### Function `call` and `apply` methods

applyメソッドはスプレッドオペレーターを利用してcallメソッドに統一する。
スプレッドオペレーターを使用できない場合のみapplyメソッドを使用する。
なおトランスパイルしないネイティブのスプレッドオペレータはv8で非常にオーバーヘッドが大きいため別途考慮が必要となる。

```ts
f.call(this, ...args);
f.apply(this, arguments);
```

### Disallow `arguments` parameter

argumentsオブジェクトでなくレストパラメーターを使用する。
オーバーヘッドが大きい。

- [https://falsandtru.github.io/benchmark/suites/6/](https://falsandtru.github.io/benchmark/suites/6/)

```ts
function f(...args) {
  g(args);
}
```

### Method chaining

メソッドチェインごとに改行しインデントを1レベル下げる。
コンテキストの変更やチェインの追加により既存チェインが変更されない。
1チェインのみであればフォーマットを整えるためにワンライナーで記述してよい。

```ts
obj.arr
  .filter(_ => true);
obj => obj.arr.filter(_ => true);
```

### Disallow immutable arrays

配列を不変オブジェクトとして扱うプログラミングスタイルを原則として禁止する。
v8でのコストが高く、計算量を管理せずに使用すると容易に重大なボトルネックとなる。
許可は厳格な管理の下で局所的かつ個別に行う。

### Disallow array methods

同じくコストが高いため計算量を管理して限定的に使用する。

- Array#concat
- Array#slice
- Array#splice

- [https://falsandtru.github.io/benchmark/suites/3/](https://falsandtru.github.io/benchmark/suites/3/)
- [https://falsandtru.github.io/benchmark/suites/8/](https://falsandtru.github.io/benchmark/suites/8/)

### Disallow the negative number index access in arrays

配列要素への負数インデクスによる不正なアクセスは事前に回避する。
オーバーヘッドが非常に大きい。

- [https://falsandtru.github.io/benchmark/suites/1/](https://falsandtru.github.io/benchmark/suites/1/)

```ts
if (0 <= index && index < arr.length) {
  arr[index];
}
```

### Object and Class member initialization across multiple lines

オブジェクトやクラスのメンバの定義が複数行にわたる場合は値の定義を改行して行う。

```ts
const o = {
  p:
    expr()
      ? 1
      : 0,
};
class C {
  public n: number =
    expr()
      ? 1
      : 0;
}
```

### Class member definition order

この順序は他に基準がない場合の目安に止めるものとする。
static member > constructor > instance member or static member only for instance の順で定義する。
各区分内は重要度の高い順に関連するメンバを隣接させて定義する。
関連するプロパティとメソッドはプロパティを先に定義する。
インスタンスメソッドからのみ使用されるプライベートなクラスメンバはインスタンスメンバと同等とする。

```ts
class C {
  private static prop_;
  public static method() {
  }

  constructor() {
  }

  private prop_;
  public method() {
  }
  private util_;
  public util() {
  }
  private static throwError_() {
  }

}
```

## Modules and Components

### Divide by functions

関連性があっても異なる独立した機能は個別のファイルに分割する。
関連をまとめて取り扱う場合は集約のみ行うモジュールファイルを用意する。

### Divide of classes

クラスの分割と抽象化は設計時および1クラスの実装が100行を大きく超えたときに検討する。
やみくもな分割と抽象化は控える。

### Namespacing

各種定義にコンテキストを与える、共通の名前空間を使用しトップレベルの名前空間の使用を減らす、名前の共通部分を省略した簡潔な命名を行うなどする場合はnamespaceを利用する。

```ts
class Entity {
  public value: Entity.Value = new Entity.Value();
}
namespace Entity {
  export class Value {
  }
}
```

### Module exports

デフォルトエクスポートの使用をパッケージモジュールなど独立した構成単位のインターフェイスのみに制限し、これ以外を禁止する。
デフォルトインポートの識別子は一括してリネームできないためリファクタリングが困難となる。

```ts
export class App {
}
```

## Code Design

### Max line length

100文字。
80文字から160文字程度。
ドキュメントおよびコメントでは制限を緩和してよい。
タイル型WMの使用を考慮する。

### Lines of code

約100行以下。
1画面に入る50行以下の行数が望ましい。

アルゴリズム、データ構造、デザインパターンなど一般化した抽象モデルの一部またはこれを拡張する場合は複雑な機能を実装しなければならないため行数が増えてもやむをえないが、
独自の機能やモデルが一般的な抽象モデルによって簡素化されず膨張する場合は抽象化に失敗している可能性がある。
一般的な抽象モデル以外の部分が100行以下で簡潔に記述されている場合は適切に抽象化されていると言える。

### Data structures and side-effects

副作用のない処理は任意のデータ構造とアルゴリズムに落とし込む。
副作用のある処理はこれに副作用をパラメータとして注入することでデータ構造を維持かつ副作用をテスト可能にする。
パラメータとする副作用はそのドメインにおけるその副作用の固有性またはテスト要件により決定する。

### Design vs Performance

設計を優先する。
パフォーマンスのために設計を歪めるとリファクタリングとモデルの成長が困難となる。
要所における関数またはメソッドの2,3個の変更で収まる最適化は行ってよい。

O(n)以下の計算量の最適化やキャッシュ化は必要時にのみ行えばよい。
O(nm)も十分に小さければ無視できる。
早まった最適化は複雑化により作りこまれたバグを特定し修正するコストによる生産性および開発速度の低下に見合わない。

## Sample products

- [securemark](https://github.com/falsandtru/securemark)
- [pjax-api](https://github.com/falsandtru/pjax-api)
- [clientchannel](https://github.com/falsandtru/clientchannel)
