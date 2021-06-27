---
title: "React x Chakra UIでポートフォリオ作ったお話"
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech"
topics: ["react", "chakraui", "typescript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true
---

:::message
**2021/6/27 追記**
デプロイ先を Vercel に変更し、Google Domain で独自ドメインを取得しました。
:::

## はじめに

私事ですが、フロントエンドの学習を始めはや数ヶ月が過ぎ、最近では React、TypeScript の学習に突入しています。
しかし、Udemy や Youtube などにある教材を写して満足して、まったくアウトプットする習慣が身についていなく、 **これでは成長できない！** と感じていました。
今回作成したアプリケーションは、アウトプットする習慣を付けるために作成した最初のアプリケーションとなります。まだまだ改善の余地はありますが、ひとまず形にはなったので公開いたします。

## アプリケーションの概要

https://www.yuki-tagawa.com/

**関連技術、機能**

- React
- TypeScript
- Chakra UI
- React Hook Form
- Vercel
- Figma
- Slack Webhook(フォームの送信内容を Slack へ通知)
  ![portfolio](https://user-images.githubusercontent.com/68511759/122046136-7d805a00-ce19-11eb-8244-c70ab49c0d51.png)

## アプリケーション作成の上で重要となった概念

:::details コンポーネント指向
コンポーネント指向とは、**ソフトウェア開発において部品ごとに小さく分けて開発する考え方**のことです。React はコンポーネント指向のライブラリなのでこの考え方が非常に重要になってきます。メリットとして**パーツの再利用がしやすく、拡張しやすい事**が挙げられます。
:::

:::details Atomic Design
Atomic Design とはアトミックデザインはアメリカの Web デザイナー Brad Frost 氏が考案・提唱したデザインシステムです。**画面要素を 5 段階に分けて、これらの要素を組み合わせることによって、最終的に画面の UI が作られます**。

- 基本的な要素である原子(**Atom**)
- 原子の組み合わせたグループである分子(**Molecule**)
- 原子と分子の組み合わせて構成されるやや複雑な有機体(**Organism**)
- UI の骨組となる原子・分子・有機体のグループであるテンプレート(**Templates**)
- 実際の内容をテンプレートで組み込んだものであるページ(**Pages**)

---

#### Atomic Design のメリット

デザイン・コーディングがもっと効率的に出来る、デザインパーツとコンテンツ内容を分離させることが出来る、デザインの変更に強い等
:::

## タスク管理

タスク管理は Notion を使用しました。普段から使用していて慣れているのと、**おしゃれ** だからです(一番大事)。
Not Started, Next Up, In Progress, Completed とタグ分けしていました。このように開始前の粒度を挙げることにより**タスクの優先順位を可視化できた**のがとても良かったです。
![Untitled](https://user-images.githubusercontent.com/68511759/122193744-b7139c80-cecf-11eb-86cc-36559e2c9322.png)

## 苦労した点

React Hook Form の導入が一番苦戦しました。React, TypeScript, Chakra UI などは Udemy の教材で触ったことがあったので良かったのですが、React Hook Form に関しては全くの初心者で最初は使い方すら分からない状態でした。

## 苦労をどのように克服したか

今まで避けていた公式のドキュメント、GitHub の issue などを読み自分でインプットをすることで壁を乗り越えました。英語だから読むのが面倒くさいと敬遠していたことを今では後悔するほど自分でインプットする良さや楽しさを感じています。

## さいごに

今回初めてデザインからデプロイまでを一人で行いました。さまざまな困難にも出会いましたが、自分で調べ解決し、最終的に一つの形にできたことはとても大きな自信になりました。今回を通してアウトプットする重要性を改めて身に染みたので、今後も継続的に続けていきたいと思います。そしてポートフォリオ作成に際して関わってくださった方に大きく感謝致します。

## 参考サイト

**Design**
https://www.i3design.jp/in-pocket/8854
https://qiita.com/nabepon/items/723dc53ad584b3457ea7
https://www.weblab.co.jp/staff/creator/7352.html
https://ferret-plus.com/13195

**Chakra UI**
https://chakra-ui.com/

**React Hook Form**
https://react-hook-form.com/

**Amplify**
https://dev.classmethod.jp/articles/amplify-react-cicd-tutorial/
