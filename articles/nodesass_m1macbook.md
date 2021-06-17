---
title: "M1でnode-sassが使えなかったお話"
emoji: "🥺" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech"
topics: [”M1", "Sass"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true
---

**win 環境で開発していた Redux Toolkit のプロジェクトを M1 にクローンして`yarn start`したところ、node-sassはまだM1に対応していよと怒られました。**

調べていると、参考になったページがあったので載せておきます。
https://shimi-shin.com/programming/web/m1-mac-node-sass/

sassとsass-loaderで代用できるみたいなので、node-sassをアンインストールして

```js
yarn remove node-sass
```

sassとsass-loaderをインストールしましょう
```js
yarn install sass sass-loader
```

これでM1で開発ができます！
