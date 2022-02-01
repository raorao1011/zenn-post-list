---
title: "useEffectの第二配列の中身が変わったらって具体的にどういうこと？"
emoji: "🥱" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech"
topics: ["react", "typescript", "useEffect"]
published: true
---

## はじめに

React を学習する上で useEffect は避けては通れない道だと思いますが、実際に useEffect 更新時って何をしているか気になったのでこの記事を書いています。

## 本題

useEffectの第二引数の説明でよく**依存配列の中身が変更されたら**と記述されますが、**変更されたら**ってどういう時なんでしょう？[公式 Doc](https://ja.reactjs.org/docs/hooks-reference.html#useeffect)にも同じような記述があります。

では、React の中身をのぞいてみましょう。
useEffectの中身を辿っていくと、[ここ](https://github.com/facebook/react/blob/da834083cccb6ef942f701c6b6cecc78213196a8/packages/shared/objectIs.js#L14)に辿り着きました。

#### 以下内容抜粋
基本的には`===`での比較ですが、`+0`と`-0`は区別され、`Number.NaN`と`NaN`は同値のようです。

```js
function is(x: any, y: any) {
  return (
    (x === y && (x !== 0 || 1 / x === 1 / y)) || (x !== x && y !== y) // eslint-disable-line no-self-compare
  );
}
```

このことが頭に入っていると、今後useEffectを扱う際にさらに理解が深まりそうです。
