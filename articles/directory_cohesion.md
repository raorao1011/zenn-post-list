---
title: "機能的凝集度からアプローチするディレクトリ設計"
emoji: "💪" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech"
topics: ["react", "typescript", "frontend", "凝集度"]
published: false
---

## はじめに

フロントエンド開発において、プロジェクトが大規模化するにつれて、以下のようなコードの保守性や可読性が課題となることがよくあると思います。

- 「この機能を修正したいが、関連するファイルがどこにあるのか分からない」
- 「新しい機能を追加する際、ファイルをどこに配置すべきか迷う」
- 「コンポーネントが複雑になりすぎて、テストやデバッグが困難」
- 「チーム開発で、他の人が書いたコードの意図を理解するのに時間がかかる」

これらの問題の根本的な原因の一つは、**凝集度**の低い設計にあります。

**凝集度**とは、ソフトウェア工学における重要な概念で、モジュール内の要素がどれだけ密接に関連し合っているかを表す指標です。高凝集度な設計では、関連性の高い機能がまとまっており、低凝集度な設計では、関連性の薄い機能が混在しています。

本記事では、凝集度の概念をReactアプリケーションのディレクトリ設計適用し、保守性・拡張性・可読性の高いフロントエンドアプリケーションを構築する方法を解説します。

## 凝集度とは何か

### 凝集度の種類と特徴
大きく7段階に分類されます。詳細はサイバーエージェントさんの新卒研修資料[^1]が分かりやすいので詳細は省略しますが、以下のイメージで問題無いと思います。

もし凝集度について知らない場合は、上記資料を先に読んでおくと分かりやすいと思います

| レベル | 凝集度の種類 | 特徴 | 凝集度 |
|--------|------------|------|------|
| 1 | 偶然的凝集 | 関連性のない要素が偶然まとまっている状態 | ❌ 低い |
| 2 | 論理的凝集 | 似たような処理をまとめているが、実際の関係は薄い状態 | ↕️ |
| 3 | 時間的凝集 | 同じタイミングで実行される処理をまとめた状態 | ↕️ |
| 4 | 手順的凝集 | 処理の順序に基づいてまとめられた状態 | ↕️ |
| 5 | 連絡的凝集 | 同じデータを扱う処理がまとまっている状態 | ↕️ |
| 6 | 情報的凝集 | 同じデータ構造を操作する処理がまとまっている状態 | ↕️ |
| 7 | 機能的凝集 | 単一の明確な目的を持つ処理がまとまっている状態 | ✅ 高い |

Webフロントエンドにおいては、論理的凝集と機能的凝集について考える機会が多いのでこの２つをピックアップします。

## ディレクトリ設計における機能的凝集度

### よくある論理的凝集への変化パターン

機能追加と共に、論理的凝集に陥ってしまう典型的なパターンを見てみましょう。

ECサイトの機能を例にします。

以下の図は、プロジェクトの成長と共に、どのように機能的凝集から論理的凝集へ劣化していくかを示しています。

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'primaryColor':'#e8f5e9','primaryTextColor':'#2e7d32','primaryBorderColor':'#4caf50','lineColor':'#ff9800','secondaryColor':'#fff3e0','tertiaryColor':'#ffebee'}}}%%
graph LR
    A["フェーズ1: 最初<br/>✅ 機能的凝集<br/>ユーザー機能のみ"] --> B["フェーズ2: 商品追加<br/>✅ 機能的凝集<br/>まだ問題なし"]
    B --> C["フェーズ3: カート追加<br/>⚠️ 論理的凝集の種<br/>priceUtils.ts登場"]
    C --> D["フェーズ4: 注文追加<br/>⚠️ 論理的凝集化<br/>無関係な機能混入"]
    D --> E["フェーズ5: 決済・レビュー<br/>❌ 完全な論理的凝集<br/>保守性大幅低下"]

    style A fill:#e8f5e9,stroke:#4caf50,stroke-width:2px
    style B fill:#e8f5e9,stroke:#4caf50,stroke-width:2px
    style C fill:#fff3e0,stroke:#ff9800,stroke-width:2px
    style D fill:#fff3e0,stroke:#ff9800,stroke-width:2px
    style E fill:#ffebee,stroke:#f44336,stroke-width:2px
```

#### フェーズ1: 最初
![](https://storage.googleapis.com/zenn-user-upload/8fb7b528a457-20251219.png)
プロジェクト開始時は、機能が少ないため自然と機能的凝集になっています。各ファイルが単一の責任を持ち、理想的な状態です。

**画面イメージ：**

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'fontSize':'14px'}}}%%
graph TB
    subgraph UI["🖥️ ユーザープロフィール画面"]
        Header["ECサイト"]
        Profile["👤 ユーザープロフィール<br/>──────────────<br/>名前: 山田太郎<br/>メール: yamada@example.com<br/>登録日: 2024/01/01"]
    end

    style UI fill:#f5f5f5,stroke:#333,stroke-width:2px
    style Header fill:#2196f3,stroke:#1976d2,stroke-width:2px,color:#fff
    style Profile fill:#fff,stroke:#ddd,stroke-width:1px
```

**ディレクトリ構成：**

```
src/
├── components/
│   └── UserProfile.tsx     # ユーザープロフィール表示のみ
├── hooks/
│   └── useUser.ts          # ユーザー情報の取得・更新のみ
└── utils/
    └── userUtils.ts        # ユーザー関連のユーティリティのみ
```

#### フェーズ2: 商品機能追加
![](https://storage.googleapis.com/zenn-user-upload/7206a072f40d-20251219.png)
新しい機能（商品）を追加しても、まだ各ファイルの責任は明確です。技術的分類でのディレクトリ構成でも問題ありません。

**画面イメージ：**

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'fontSize':'14px'}}}%%
graph TB
    subgraph UI["🖥️ 商品一覧画面"]
        Header["ECサイト"]
        Products["📦 商品一覧<br/>──────────────<br/>┌─────────────┐<br/>│ 商品A │<br/>│ ¥1,000 │<br/>└─────────────┘<br/>┌─────────────┐<br/>│ 商品B │<br/>│ ¥2,500 │<br/>└─────────────┘"]
    end

    style UI fill:#f5f5f5,stroke:#333,stroke-width:2px
    style Header fill:#2196f3,stroke:#1976d2,stroke-width:2px,color:#fff
    style Products fill:#fff,stroke:#ddd,stroke-width:1px
```

**ディレクトリ構成：**

```
src/
├── components/
│   ├── UserProfile.tsx
│   └── ProductCard.tsx     # 商品表示のみ
├── hooks/
│   ├── useUser.ts
│   └── useProduct.ts       # 商品情報の取得・更新のみ
└── utils/
    ├── userUtils.ts
    └── productUtils.ts     # 商品関連のユーティリティのみ
```

#### フェーズ3: カート機能追加
![](https://storage.googleapis.com/zenn-user-upload/09f37dc20938-20251219.png)
カート機能追加時、価格に関する処理を`priceUtils.ts`としてまとめ始めます。この時点では問題無いように見えますが、論理的凝集の種が蒔かれています。

**画面イメージ：**

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'fontSize':'14px'}}}%%
graph TB
    subgraph UI["🖥️ ショッピングカート画面"]
        Header["ECサイト"]
        Cart["🛒 カート<br/>──────────────<br/>商品A × 2<br/>¥1,000 × 2 = ¥2,000<br/><br/>商品B × 1<br/>¥2,500 × 1 = ¥2,500<br/>──────────────<br/>合計: ¥4,500"]
    end

    style UI fill:#f5f5f5,stroke:#333,stroke-width:2px
    style Header fill:#2196f3,stroke:#1976d2,stroke-width:2px,color:#fff
    style Cart fill:#fff,stroke:#ddd,stroke-width:1px
```

**ディレクトリ構成：**

```
src/
├── components/
│   ├── UserProfile.tsx
│   ├── ProductCard.tsx
│   └── CartItem.tsx        # カート表示のみ
├── hooks/
│   ├── useUser.ts
│   ├── useProduct.ts
│   └── useCart.ts          # カート操作のみ
└── utils/
    ├── userUtils.ts
    ├── productUtils.ts
    └── priceUtils.ts       # ← ここから論理的凝集が始まる
```

```typescript
// utils/priceUtils.ts - 最初は商品価格のフォーマットだけ
export const formatPrice = (price: number) => {
  return new Intl.NumberFormat('ja-JP', {
    style: 'currency',
    currency: 'JPY'
  }).format(price);
};
```

#### フェーズ4: 注文機能追加
![](https://storage.googleapis.com/zenn-user-upload/2037948295f1-20251219.png)
注文機能の追加で「価格関連だから」という理由で無関係な計算処理が`priceUtils.ts`に混入し始めます。ファイルの責任が曖昧になってきます。

**画面イメージ：**

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'fontSize':'14px'}}}%%
graph TB
    subgraph UI["🖥️ 注文確認画面"]
        Header["ECサイト"]
        Order["📋 注文確認<br/>──────────────<br/>商品合計: ¥4,500<br/>消費税(10%): ¥450<br/>送料: ¥500<br/>割引: -¥200<br/>──────────────<br/>合計: ¥5,250"]
    end

    style UI fill:#f5f5f5,stroke:#333,stroke-width:2px
    style Header fill:#2196f3,stroke:#1976d2,stroke-width:2px,color:#fff
    style Order fill:#fff,stroke:#ddd,stroke-width:1px
```

**ディレクトリ構成：**

```
src/
├── components/
│   ├── UserProfile.tsx
│   ├── ProductCard.tsx
│   ├── CartItem.tsx
│   └── OrderSummary.tsx    # 注文サマリー表示
├── hooks/
│   ├── useUser.ts
│   ├── useProduct.ts
│   ├── useCart.ts
│   └── useOrder.ts         # 注文処理
└── utils/
    ├── userUtils.ts
    ├── productUtils.ts
    └── priceUtils.ts       # ← ここに無関係な機能が混入
```

```typescript
// utils/priceUtils.ts - 関係ない機能が追加され始める
export const formatPrice = (price: number) => { /* ... */ };

// 注文機能追加時に「価格関連だから」という理由で追加
export const calculateTax = (price: number) => price * 0.1;
export const calculateShipping = (items: CartItem[]) => { /* ... */ };

// 割引機能追加時に「価格計算だから」という理由で追加
export const applyDiscount = (price: number, discountRate: number) => { /* ... */ };
```

#### フェーズ5: 決済・レビュー機能追加
![](https://storage.googleapis.com/zenn-user-upload/1c3b9934c2b1-20251219.png)
![](https://storage.googleapis.com/zenn-user-upload/88536c98f472-20251219.png)
さらなる機能追加により、表面的な共通点だけで無関係な機能が一つのファイルに集まった状態です。保守性・可読性が大幅に低下しています。

**画面イメージ：**

```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'fontSize':'14px'}}}%%
graph LR
    subgraph Payment["💳 決済画面"]
        P1["クレジットカード番号:<br/>□□□□-□□□□-□□□□-□□□□<br/><br/>分割払い設定:<br/>□ 一括 □ 3回 □ 6回<br/><br/>合計: ¥5,250"]
    end

    subgraph Review["⭐ レビュー投稿画面"]
        R1["商品A のレビュー<br/>──────────────<br/>評価: ★★★★☆<br/><br/>コメント:<br/>┌──────────┐<br/>│ とても良い │<br/>│ 商品でした │<br/>└──────────┘"]
    end

    style Payment fill:#fff3e0,stroke:#ff9800,stroke-width:2px
    style P1 fill:#fff,stroke:#ddd,stroke-width:1px
    style Review fill:#e8f5e9,stroke:#4caf50,stroke-width:2px
    style R1 fill:#fff,stroke:#ddd,stroke-width:1px
```

**ディレクトリ構成：**

```
src/
├── components/
│   ├── UserProfile.tsx
│   ├── ProductCard.tsx
│   ├── CartItem.tsx
│   ├── OrderSummary.tsx
│   ├── PaymentForm.tsx     # 決済フォーム
│   └── ReviewForm.tsx      # レビューフォーム
├── hooks/
│   ├── useUser.ts
│   ├── useProduct.ts
│   ├── useCart.ts
│   ├── useOrder.ts
│   ├── usePayment.ts       # 決済処理
│   └── useReview.ts        # レビュー処理
└── utils/
    ├── userUtils.ts
    ├── productUtils.ts
    ├── priceUtils.ts       # ← 完全に関係ない機能の寄せ集めに
    └── validationUtils.ts  # ← 新たな論理的凝集ファイル
```

```typescript
// utils/priceUtils.ts - 「価格」「計算」という名前だけで無関係な機能が集まった状態

// 🟦 商品機能に関する処理
export const formatPrice = (price: number) => { /* 商品価格フォーマット */ };

// 🟩 注文機能に関する処理
export const calculateTax = (price: number) => { /* 税金計算 */ };
export const calculateShipping = (items: CartItem[]) => { /* 送料計算 */ };
export const applyDiscount = (price: number, rate: number) => { /* 割引適用 */ };

// 🟨 ユーザー機能に関する処理
export const calculatePoints = (price: number) => { /* ポイント計算 */ };

// 🟧 決済機能に関する処理
export const splitPayment = (total: number, methods: PaymentMethod[]) => { /* 分割払い計算 */ };

// 🟥 返品機能に関する処理
export const calculateRefund = (order: Order) => { /* 返金計算 */ };

// utils/validationUtils.ts - 「バリデーション」という名前だけで無関係な機能が集まった状態

// 🟦 ユーザー機能に関する処理
export const validateEmail = (email: string) => { /* メール形式チェック */ };
export const validatePassword = (password: string) => { /* パスワード強度チェック */ };

// 🟩 決済機能に関する処理
export const validateCreditCard = (number: string) => { /* クレジットカード番号チェック */ };
export const validatePostalCode = (code: string) => { /* 郵便番号チェック */ };

// 🟨 商品機能に関する処理
export const validateProductName = (name: string) => { /* 商品名チェック */ };

// 🟧 レビュー機能に関する処理
export const validateReviewText = (text: string) => { /* レビュー内容チェック */ };
```

このように、一つのファイルに複数の機能（🟦🟩🟨🟧🟥）が混在していることが分かります。これが論理的凝集の典型的な問題です。

### 論理的凝集の問題点

論理的凝集になったファイルは表面的な共通点はあるものの、実際には異なるビジネス機能に属しており、変更理由や変更タイミングが異なるため様々な問題を引き起こします。

上記の例で、最終的に `priceUtils.ts` と `validationUtils.ts` は論理的凝集になってしまいました：

**priceUtils.ts の問題：**
- 商品価格フォーマット（商品機能）
- 税金・送料計算（注文機能）
- ポイント計算（ユーザー機能）
- 分割払い計算（決済機能）
- 返金計算（返品機能）

これらは「価格計算」という表面的な共通点はあるものの、実際には異なるビジネス機能に属しており、変更理由も変更タイミングも異なります。

**実際に起こる問題：**
1. **影響範囲の拡大**: 商品の価格表示を変更したいだけなのに、決済・返金機能のテストも必要になる
2. **並行開発の阻害**: 複数チームが同じファイルを同時に変更してコンフリクトが発生
3. **理解の困難**: 新しいメンバーが「なぜこれらの機能が一つのファイルにあるのか」理解できない

#### 影響範囲の可視化

以下の図は、「商品の価格表示フォーマットを変更したい」というシナリオで、論理的凝集と機能的凝集での影響範囲の違いを示しています。

**❌ 論理的凝集の場合：影響範囲が広範囲に及ぶ**

```mermaid
graph TB
    Change["変更: formatPrice関数<br/>（商品価格フォーマット）"]

    File["priceUtils.ts<br/>⚠️ 論理的凝集ファイル"]

    Product["🟦 商品機能<br/>（変更対象）"]
    Order["🟩 注文機能<br/>（影響を受ける）"]
    User["🟨 ユーザー機能<br/>（影響を受ける）"]
    Payment["🟧 決済機能<br/>（影響を受ける）"]
    Refund["🟥 返品機能<br/>（影響を受ける）"]

    Test["😱 全機能のテストが必要<br/>- 商品表示テスト<br/>- 注文計算テスト<br/>- ポイント計算テスト<br/>- 決済処理テスト<br/>- 返金処理テスト"]

    Change --> File
    File --> Product
    File --> Order
    File --> User
    File --> Payment
    File --> Refund

    Product --> Test
    Order --> Test
    User --> Test
    Payment --> Test
    Refund --> Test

    style Change fill:#2196f3,stroke:#1976d2,stroke-width:2px,color:#fff
    style File fill:#ffebee,stroke:#f44336,stroke-width:3px
    style Product fill:#e3f2fd,stroke:#2196f3,stroke-width:2px
    style Order fill:#ffebee,stroke:#f44336,stroke-width:2px
    style User fill:#ffebee,stroke:#f44336,stroke-width:2px
    style Payment fill:#ffebee,stroke:#f44336,stroke-width:2px
    style Refund fill:#ffebee,stroke:#f44336,stroke-width:2px
    style Test fill:#ffebee,stroke:#f44336,stroke-width:3px
```

**✅ 機能的凝集の場合：影響範囲が限定的**

```mermaid
graph TB
    Change2["変更: formatPrice関数<br/>（商品価格フォーマット）"]

    ProductFile["features/product/<br/>productUtils.ts<br/>✅ 機能的凝集"]
    OrderFile["features/order/<br/>orderUtils.ts"]
    UserFile["features/user/<br/>userUtils.ts"]
    PaymentFile["features/payment/<br/>paymentUtils.ts"]
    RefundFile["features/refund/<br/>refundUtils.ts"]

    ProductFunc["🟦 商品機能<br/>（変更対象）"]

    TestLimited["😊 商品機能のテストのみ<br/>- 商品表示テスト"]

    Change2 --> ProductFile
    ProductFile --> ProductFunc
    ProductFunc --> TestLimited

    style Change2 fill:#2196f3,stroke:#1976d2,stroke-width:2px,color:#fff
    style ProductFile fill:#e8f5e9,stroke:#4caf50,stroke-width:3px
    style OrderFile fill:#f5f5f5,stroke:#9e9e9e,stroke-width:1px,stroke-dasharray: 5 5
    style UserFile fill:#f5f5f5,stroke:#9e9e9e,stroke-width:1px,stroke-dasharray: 5 5
    style PaymentFile fill:#f5f5f5,stroke:#9e9e9e,stroke-width:1px,stroke-dasharray: 5 5
    style RefundFile fill:#f5f5f5,stroke:#9e9e9e,stroke-width:1px,stroke-dasharray: 5 5
    style ProductFunc fill:#e8f5e9,stroke:#4caf50,stroke-width:2px
    style TestLimited fill:#e8f5e9,stroke:#4caf50,stroke-width:3px
```

この図から、機能的凝集では変更の影響範囲が商品機能内に限定され、他の機能のテストが不要になることが分かります。

### 機能的凝集への改善例

論理的凝集を機能的凝集に改善するには、Feature-basedなディレクトリ構成に変更し、機能ごとに関連するファイルをまとめることが効果的です。

```
src/
├── features/
│   ├── product/
│   │   ├── components/ProductCard.tsx
│   │   ├── hooks/useProduct.ts
│   │   └── utils/productUtils.ts     # 商品価格フォーマットなど
│   ├── cart/
│   │   ├── components/CartItem.tsx
│   │   ├── hooks/useCart.ts
│   │   └── utils/cartUtils.ts        # カート計算など
│   ├── order/
│   │   ├── components/OrderSummary.tsx
│   │   ├── hooks/useOrder.ts
│   │   └── utils/orderUtils.ts       # 税金・送料計算など
│   └── payment/
│       ├── components/PaymentForm.tsx
│       ├── hooks/usePayment.ts
│       └── utils/paymentUtils.ts     # 分割払い計算など
└── shared/
    └── utils/
        └── formatUtils.ts            # 本当に共通の汎用関数のみ
```

このように機能別に分割することで、各機能の責任が明確になり、保守性が大幅に向上します。

## まとめ

### 凝集度を意識した設計のメリット

本記事で解説した凝集度に基づく設計アプローチにより、以下のメリットが得られます：

#### 1. 保守性の向上
- **影響範囲の限定**: 機能変更時の影響が関連するファイルのみに限定される
- **バグの早期発見**: 責任が明確なため、問題の発生箇所を特定しやすい
- **安全なリファクタリング**: 単一責任により、変更による副作用のリスクが低減

#### 2. 開発効率の改善
- **並行開発**: feature単位での作業により、チームメンバー間の競合が減少
- **再利用性**: 高凝集なコンポーネントは他の機能でも活用しやすい
- **学習コスト**: 新メンバーが理解すべき範囲が明確になる

#### 3. テスト品質の向上
- **単体テスト**: 責任が明確なため、テストケースの作成が容易
- **結合テスト**: feature単位でのテストにより、統合時の問題を早期発見
- **E2Eテスト**: ユーザーストーリーに沿ったテストシナリオの作成が簡単

#### 4. スケーラビリティの確保
- **段階的成長**: 新機能追加時も既存構造を維持できる
- **チーム拡大**: 明確な責任分界により、大規模チームでの開発が可能
- **技術選択**: feature単位で異なる技術スタックの採用も可能

## 参考文献

[^1]: https://note.com/cyberz_cto/n/n26f535d6c575#E0aBe

https://speakerdeck.com/noritakaikeda/ji-neng-de-ning-ji-nogai-nian-woyong-ite-fu-shu-roru-lei-si-noji-neng-woduo-kuhan-musisutemuno-hurontoendonokonponentowoshi-qie-nifen-ge-suru
