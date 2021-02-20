---
layout: bootstrap
title: TypeScript Coding Guidelines
type: page
nav: pages
---

# TypeScript Guidelines

## 型と値

### プリミティブのオブジェクト化の禁止

Stringなどプリミティブな型および値のオブジェクト版の使用を禁止する。

### グローバル変数

ブラウザにおいて実行頻度の高い場所でのグローバル変数の直接使用を禁止する。
グローバル変数は参照が遅いためローカル変数として再定義して使用しなければならない。

```ts
const { Array } = globalThis;
```

### void型

void型の関数の戻り値以外への使用および`null | undefined`型としての使用を禁止する。
void型に定義された値を得た場合はその値を使用せず直ちに破棄する。
void型はコールバックを経由して任意の値を割り当て可能であるため実際には`undefined`または`null | undefined`型として扱うことはできず、未知という点でunknown型、不可触という点でnever型と同じものとして扱わなければならない。

```ts
function f(): void {
  return void g();
}
function g(): void {
}
```

### undefined値

グローバル変数としての`undefined`値の参照を禁止する。
`undefined`値はモジュールレベルでローカル変数として宣言するか`void 0`で随時生成して使用しなければならない。

### null値

nullの使用をnullの検出、Object.create(null)などnullが必須である機能の使用、既存のAPIの一貫性を保つ場合を除き禁止する。

### never型

never型の関数の戻り値以外への使用を禁止する。

### Object型

`Object`型の使用を禁止する。

### `{}`型

`{}`型の使用を禁止する。

### 文字列の引用符

文字列の生成は原則としてシングルクオート使用し、ダブルクオートおよびバッククオートは必要のない限り使用を禁止する。

### Big number

10進数で15桁（正確には2^53/2-1）を超える整数は精度が保証されないため使用を禁止とし、文字列化して管理する。

```ts
JSON.parse('{"id": 9999999999999999}'); // Object {id: 10000000000000000}
```

ビット演算可能なのは32ビット、10進数で9桁（正確には2^32/2-1）まで。
符号ビットである32ビット目または32ビット目だったビットを使用する場合は符号不維持の右シフト（>>>）を使用し符号維持の右シフト（>>）を使用してはならない。
また32ビットに達するビット演算の結果を数値としても使用する場合32ビット目の符号ビットの影響を避けるため`>>> 0`でUintに変換してから使用しなければならない。

```ts
999999999.9|0; // 999999999
~~999999999.9; // 999999999
9999999999|0; // 1410065407
~~9999999999; // 1410065407
```

### 構造的Enum

構造型のEnumは次のフォーマットで定義する。
型定義を共有する場合公称型のEnumを避けたい場合がある。

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

予約語を含めたい場合は全体をオブジェクトとして定義すれば可能。
ただし型変数に予約語は使用できない。

```ts
type Type =
  | typeof Type.put
  | typeof Type.delete
  | typeof Type.get;
const Type = {
  put: 'put',
  delete: 'delete',
  get: 'get'
} as const;
```

## 演算子

### 等価演算子

厳密等価演算子のみ使用する。
等価演算子の使用を禁止する。

```ts
null === null;
```

### `??`演算子

`||`の代わりに`??`を使用できる場合は常に`??`を使用し意図の曖昧さを排除する。

### インクリメントおよびデクリメント演算子

必要のない限り常に前置記法で記述する。

```ts
++n;
--n;
```

### 比較順序

actual > expectedの順に記述する。

```ts
if (typeof n === 'number') {
}
```

### 複数行にわたる三項演算子

各演算子の前で改行しインデントを1レベル下げる。
`return`文とともに使用する場合は常に複数行で記述する。

```ts
function f() {
  return expr
    ? true
    : false;
}
```

### instanceof演算子の回避

高速なin演算子によるプロパティチェックを低速なinstanceof演算子の代わりに可能な限り使用する。
Textの選択には`wholeText`またはCommentがない場合は`data`を、Elementの選択には`id`を使用する。

```ts
function f<T extends Node>(node: T): T;
function f(node: Node | Text): Node {
  return 'data' in node
    ? node.data
    : node.textContent;
}
```

## 式と文

### 複数行にわたるUnion/Intersection型の宣言

次のフォーマットを使用する。

```ts
type union =
  | A
  | B
  | C;
```

### 変数宣言

`const`および`let`のみ使用する。
`var`の使用はアルゴリズムまたはES5の制約で必要な場合のみに止める。

### 補助変数名

補助変数名は元の変数名の末尾に`$`を加えたものとする。
`_`は可読性が低すぎる。

```ts
function f() {
}
function f$() {
}
function f$$() {
}
class C {
  private p$() {
  }
  public p() {
  }
}
```

### 破棄変数

代入を破棄する変数をアンダースコアで宣言する。
関数に2つ以上省略できない引数がある場合は複数のアンダースコアまたはアンダースコアで始まる変数名を使用する。

```ts
function f([, , x], _, __, _a, y) {
  return [x, y];
}
f([1, 2, 3], 0, [], NaN, 4);
const g = _ => 0;
g(NaN);
```

### 選択的代入

複数の候補からtruethyな値を走査して代入する場合は次のフォーマットを使用する。
`&&`を併用する場合これを行頭に使用してはならない

```ts
const a = void 0
  || f()
  || g()
  || h();
```

### 条件式の評価

条件式で評価してよい値の組み合わせは次の2つのパターンのみとする。

#### true/false

比較演算、論理演算または評価関数の結果である真理値。

```ts
if (arr.indexOf(n) > -1) {}
if (!flag) {}
if (is()) {}
```

#### object/(undefined|null)

オブジェクト（非プリミティブ値）の有無。

```ts
if (err) {}
if (elem.parent) {}
```

### 複数のブロックを持つ文の改行

複数ブロックを持つ構文はブロックの終了ごとに改行する。

```ts
if (expr) {
  f();
}
else {
  g();
}
```

### ブレース省略if文

ブレースを省略したif文を次の条件をすべて満たすガード節にのみ許可する。

- ワンライナーである。
- elseおよびelse if文を持たない。
- `return`, `continue`, `break`, `throw`といったコードフローを制御する予約語で開始する。

```ts
function f() {
  if (!expr) return;
}
```

### ブレース省略if文とループ （非推奨）

条件式が短くかつ分割した記述では複雑になる場合のみ例外的にif文のブロックの省略を許可する。

```ts
if (expr) for (;;) {
}
```

### スコープ脱出順序

スコープからの脱出は原則として上位のスコープに対するものから記述する。

```ts
while (true) {
  switch (val) {
    case 0:
      if (expr) continue;
      break;
  }
}
```

### case句のブロック化

switch文内で変数を宣言する場合これが漏出しないよう当該case句をブロック化する。

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

### switch文の分離

同一ブロックにおけるswitch文と他の文の混在を非推奨とする。
if文に置換または関数に切りだすべきものである場合が多い。
単純な事前処理のとしては許可する。
単純な事後処理のとしては非推奨とする。


### Promise

Promiseの失敗文脈では回復不能な例外すなわち異常系のみ扱い、規定の失敗および回復可能な例外は多値の成功文脈すなわち正常系で扱う。

失敗文脈は原則として回復不能な例外のみを扱う異常系として設計しなければならない。
例外と同列に処理できる失敗は例外に含めてよい。
Promiseの失敗文脈は規定の失敗と予期せぬ例外を分離できないため基本的に失敗文脈として使用できない。
失敗文脈をそれとして使用する場合は絶対に規定外の例外が混入しない信頼性が必要となるがほとんどの開発者はそのような関数を作り続けられない。
このためPromise値を返し失敗文脈を持つ組み込み関数を単に模倣して失敗文脈を持つ関数を作ることは誤りである。
よって原則として組み込み関数のような機能的にも意味的にも最小単位の関数以外で失敗文脈を使用するべきではない。

```ts
new Promise(resolve => resolve([arr.length > 0, arr.pop(), arr]));
```

### async/await

async/awaitはPromise同様失敗を多値の成功文脈で表現し、失敗文脈が例外のみ扱うように設計することでtry-catch文が不要となるよう設計する。
try-catch文によりawait式の失敗文脈を捕捉する場合、tryブロックには可能な限りawait式のみを入れ、同期的処理の例外を混入させない。
同期処理において捕捉しない例外を非同期処理に限って捕捉しなければならない理由はない。

```ts
const n = 100;
try {
  await sleep(n);
}
catch (err) {
  throw err;
}
```

### async関数

Promise値を返す可能性のあるasync関数は常にawaitを介して値を返す。

```ts
async function f(g: () => unknown) {
  return await g();
}
```

## 関数とオブジェクト

### 関数定義順序

関数は粒度（目的としての抽象度）の大きい順に定義する。

```ts
f();

function f() {
  g();
}
function g() {
}
```

### クロージャー定義位置

クロージャーは`return`文の後ろに1行空けて記述する。

```ts
function f() {
  return g();

  function g() {
  }
}
```

### アダプター関数定義

関数の役割が実体である別の関数の事前処理または事後処理、あるいはインターフェイスの隠蔽であり、実体関数が他の関数から使われない場合、自身のクロージャーとして実装する。
クロージャーの名前の重複を避ける場合は名前の末尾に`$`を追加する。
メソッドにおいても同様とする。

```ts
function f() {
  return f$(1);

  function f$(cnt: number) {
    return cnt > 10
      ? void 0
      : f$(cnt + 1);
  }
}
```

実行性能に影響がある場合は外部へ露出する。

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

### 閉じ括弧のインデント

閉じ括弧は末尾の要素と同じ行に続ける。

```ts
f(() =>
  g(() =>
    h()));
```

### アロー関数のインデント

ブロックを作らない複数行にわたるアロー関数は、アローの後で改行しインデントを1レベル下げる。
アロー関数から通常関数に変更してもreturn文の追加以外関数本体が変更されない利点がある。

```ts
dispatch(ns =>
  ns
    .filter(n => !isNaN(n))
    .map(parseInt));
```

```ts
dispatch(ns => {
  return ns
    .filter(n => !isNaN(n))
    .map(parseInt);
});
```

### 引数関数のインデント

呼び出し関数と同じ行で複数行にわたる関数をパラメータとして渡す場合、以降のパラメータは改行せず同じ行に記述する。
このとき関数本体は常に何らかの括弧で囲み垂直アラインをそろえる。

```ts
[].reduce(b => {
  return b;
}, 0);
[].reduce(b => [
  b
], 0);
[].reduce(b => (
  b
), 0);
```

呼び出し関数とパラメータを改行して分ける場合は通常通りコロンを行末とする。

```ts
[].reduce(
  b =>
    b,
  0);
[].reduce(
  b => {
    return b;
  },
  0);
```

### 引数の型宣言

モジュール外に公開されるAPIの引数の型は初期値により推論可能な場合でも明記する。

### 戻り値の型

ファクトリー以外の関数の戻り値は原則としてプリミティブ型をはじめとするビルトインタイプおよびパラメーターの型のみとする。
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

### コールバック関数の戻り値型

コールバック関数等で戻り値を破棄する場合は戻り値の型にvoidを指定する。
戻り値を使うが型制約がない場合はunknownを指定する。

```ts
function f(cb: () => void): boolean {
}
```

### コマンドクエリ分離

コマンドの戻り値は原則として空または実行結果もしくはその識別子ないし操作方法のいずれかに限る。
自身に破壊的変更を加えながら自身を返すメソッドは禁止する。
副作用を持つ操作が深いスタックを持ち垂直方向に複雑化すると副作用の管理が困難となるため副作用を持つ操作は可能な限りネストさせず浅い操作にしなければならない。
コマンドに副作用を連鎖させる戻り値を禁止する制約は副作用の複雑化を水平方向に止め垂直的複雑化を抑止する。
不変オブジェクトを返すメソッドチェーンは一般的に副作用を持たないためこの制約対象に該当しない。

### return文における複数行にわたる条件式

複数行にわたり末尾のreturn文に条件分岐を持たせる場合は次のフォーマットを使用する。
このフォーマットで表現できない複雑な条件分岐は不適切である可能性が高い。

#### 三項演算子

```ts
function f() {
  return expr
    ? a1
    : a2;
}
```

#### 論理演算子

```ts
function f() {
  return a1
      && a2
      && a3;
}
function g() {
  return a1
      || a2
      || a3;
}
function h() {
  return a1
      || a2 && a2_1
      || a3 && a3_1;
}
```

### 条件分岐の順序

条件分岐は開始状態または終了状態（不動点）を最初に記述し、変化形を変化率の小さい順に記述していく。
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

### アドホック多相関数

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

### メソッドチェーン

メソッドチェーンごとに改行しインデントを1レベル下げる。
コンテキストの変更やチェーンの追加により既存チェーンが変更されない。
1チェーンのみであればフォーマットを整えるためにワンライナーで記述してよい。

```ts
obj.arr
  .filter(_ => true);
obj => obj.arr.filter(_ => true);
```

### 不変配列の禁止

配列を複製等の追加操作により不変オブジェクト化して扱うプログラミングスタイルを禁止する。
v8でのコストが高く、計算量を管理せずに使用すると容易に重大なボトルネックとなる。

### 配列メソッドの一部制限

次の処理はコストが高いため計算量を管理して限定的に使用する。

- Array#concat
- Array#splice
- Array#slice

- [https://falsandtru.github.io/benchmark/suites/3/](https://falsandtru.github.io/benchmark/suites/3/)
- [https://falsandtru.github.io/benchmark/suites/8/](https://falsandtru.github.io/benchmark/suites/8/)

### 負数インデクスによる配列アクセスの禁止

配列要素への負数インデクスによる不正なアクセスは事前に回避する。
コストが非常に大きい。

- [https://falsandtru.github.io/benchmark/suites/1/](https://falsandtru.github.io/benchmark/suites/1/)

```ts
if (0 <= index && index < arr.length) {
  arr[index];
}
```

### クラスフィールド定義順序 （実験的）

この順序は他に基準がない場合の目安に止めるものとする。
static member > constructor > instance member or static member only for instance の順で定義する。
各区分内は重要度の高い順に関連するフィールドを隣接させて定義する。
関連するプロパティとメソッドはプロパティを先に定義する。
インスタンスメソッドからのみ使用されるプライベートなクラスフィールドはインスタンスフィールドと同等とする。

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

### キャッシュプロパティ名

キャッシュ用プロパティ名は元のプロパティ名の先頭に`$`を加えたものとする。
`_`は可読性が低すぎる。

```ts
class C {
  private $id?: string;
  private get id(): string {
    return this.$id ??= gen();
  }
}
```

### 検査メソッド/プロパティ名

状態などの検査結果を返すメソッドの名前は常にis/hasを先頭に付ける。
これをゲッターまたはプロパティとして実装する場合は常に除く。

```ts
class C {
  public alive: boolean = false;
  private available: boolean = false;
  public isAvailable: boolean {
    return this.available;
  }
}
```

## モジュールとコンポーネント

### 機能の分割

関連性があっても本質的に異なる需要を満たす、将来的に分岐する可能性のある機能は個別のファイルに分割する。
関連をまとめて取り扱う場合は集約のみ行うモジュールファイルを用意する。

### クラスの分割

クラスの分割と抽象化は設計時および1クラスの実装が100行を大きく超えたときに検討する。
やみくもな分割と抽象化は控える。

### 名前空間の分離

同種の名前を共通の名前空間に分離しトップレベルの名前空間の散乱を避ける。

```ts
class C {
  constructor(opts: C.Options) {
  }
}
namespace C {
  export interface Options {
  }
}
```

役割が異なるため個別に名前を提供したい場合は名前を分離する。

```ts
class App {
  constructor(opts: AppOptions) {
  }
}
interface AppOptions {
}

class Observer implements Publisher, Subscriber {
}
interface Publisher {
}
interface Subscriber {
}
```

### モジュールインポート

次の順でインポートを列挙する。

1. ビルトインモジュール（重要度順または定義準）
1. 内部モジュール（同上、次いで祖先から子孫順）
1. 外部モジュール（同上）

議論の余地をなくすなら単に昇順とする。

1. ビルトインモジュール（昇順）
1. 内部モジュール（祖先から子孫順、次いで昇順）
1. 外部モジュール（昇順）

### モジュールエクスポート

デフォルトエクスポートの使用をパッケージモジュールなど独立した構成単位のインターフェイスのみに制限し、これ以外を禁止する。
デフォルトインポートの識別子は一括してリネームできないためリファクタリングが困難となる。

```ts
export class App {
}
```

## ファイル

### 最大列数

100文字。
80文字から160文字程度。
ドキュメントおよびコメントでは制限を緩和してよい。
タイル型WMの使用などを考慮し画面サイズに比例して表示幅が広がると考えない。

### 推奨行数

抽象化されたコードは100行以下が8割150行以下が9割。
1画面に入る50行以下が望ましい。
抽象モデルそのものの実装は300から500行以下。

アルゴリズム、データ構造、デザインパターンなど一般化した抽象モデル自体を実装またはこれを拡張する場合はその記述のために行数が増えてもやむをえないが、独自の機能やモデルが一般的な抽象モデルによって単純化されず肥大する場合は抽象化に失敗している可能性がある。
抽象モデル以外の部分が概ね100行以下で簡潔に記述されていれば適切に抽象化されていると言える。

## 設計

### データ構造と副作用

副作用のない処理は任意のデータ構造とアルゴリズムに落とし込む。
副作用のある処理はこれに副作用をパラメータとして注入することでデータ構造を維持かつ副作用をテスト可能にする。
パラメータにする副作用はそのドメインにおけるその副作用の固有性またはテスト要件により決定する。

### 設計と性能

設計を優先する。
性能のために設計を歪めるとリファクタリングとモデルの成長が困難となる。
要所における関数またはメソッドの2,3個の変更で収まる最適化は行ってよい。

O(n)以下の計算量の最適化やキャッシュ化は必要時にのみ行えばよい。
O(nm)も十分に小さければ無視できる。
早まった最適化は複雑化により作りこまれたバグを特定し修正するコストによる生産性および開発速度の低下に見合わない。

## 開発

### 構造駆動

初めに適切なディレクトリ構造およびモジュール構造を模索して全体像を固めその後も安定するまで最適な構造を模索し続ける。
初期構造確定前のコードとテストは構造変更を妨げないよう段階的に記述する。

### テスタブル

最初からテスト可能であることを要件として最大限すべてのコードがテスト可能となるよう設計とコーディングを行う。
またテストは整然として読みやすく追加変更が容易でなければならない。
テストできないものは作れない。

### 整然性

優れたコードは整然としてリズミカルでありその同型性と反復性を利用して軽快に読み解ける。
優れたコードはこのように書かなければならない。

## サンプルプロダクト

- [securemark](https://github.com/falsandtru/securemark)
- [pjax-api](https://github.com/falsandtru/pjax-api)
