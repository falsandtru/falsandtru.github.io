---
layout: bootstrap
title: CSS Coding Guidelines
type: page
nav: pages
---

# CSS Guidelines

## Architecture

- idまたはclass1つのみで対象を指定する。
  - CSSセレクタの階層化をModuleの例外を除き禁止。
- 要素1つはCSS1つのみと対応する。
  - CSSの複数指定をDecoration, JSの例外を除き禁止。
- Base以外のCSSのオーバーライドを禁止。
- 擬似要素の使用を禁止。
- Layout以下のすべての名前付き構造のCSSはサイト全体でユニークであり包含関係により変化しない。
- 正規形の表記を非正規形と大きく変えることで混在させても区別を容易にする。

### Framework

- 使用するCSSフレームワークに関するCSSを記述。
- コーディング規約は使用するフレームワークに準じる。

```css
.col-sp-5 {
}
```

### Base

- tag1つのみで指定。
- 英数小文字表記。

```css
tag {
}
```

### Layout

- id1つのみで指定。
- アッパーキャメルケース表記。
- id名は以下のものに限る。

- Container
  - Header
  - Body
    - Wrapper
      - Primary
      - Secondary
      - Tertiary
  - Footer

```css
#Container {
}
```

### Module

- class1つのみで指定。
- アッパーキャメルケース表記。
- Framework, Base以外のCSSのオーバーライドを禁止。
- 作業量または容量負荷が大きい場合はtagによる子孫セレクタ1つに限り使用して構造化してよい。

```css
.Module {
}
```

#### Element

- Moduleにアンダースコア2つでつないで指定。
- その他はModuleに準じる。

```css
.Module__Title {
}
.Module__Content img {
}
```

#### State

- ModuleまたはElementにハイフン2つでつないで指定。
- 英小文字表記1語。
- その他はModuleに準じる。

```css
.Module--active {
}
```

### Decoration

- アンダースコア1つを接頭辞とするclass1つのみで指定。
- 英数小文字表記1語。
- インライン要素の視覚効果にのみ使用する。
- Framework, Base以外のCSSのオーバーライドを禁止。
- Decoration, JS以外との同時使用を禁止。
- Base, Moduleへの記述を優先し最小限の使用に止める。

```css
._large {
}
```

### Shame

- 既存の一般的な記法で記述し区別を容易にする。
- 英数小文字ハイフン区切り表記。

```css
.video-id {
}
```

### JS

- `js-`接頭辞をModuleに付加して指定。
- CSSの使用禁止。

## Priority

項目|優先度
----|------
全体|影響度 > 関心度 > 関連度
構造|大 > 小
テキスト|文字 > 文字列 > レイアウト
非テキスト|未定義
並列|関連度

1. 表示
  1. display
  2. visibility
  3. float
  4. overflow
2. 座標
  1. position
  2. z-index
  3. top
  4. bottom
  5. left
  6. right
3. サイズ
  1. margin
  2. padding
  3. border
  4. width
  5. height
4. テキスト
  1. font
  2. color
  3. line-height
  4. letter-spacing
  5. text-align
5. 非テキスト
  1. background
6. モーション
  1. animation
  2. transition
