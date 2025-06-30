---
title: "機能的凝集度からアプローチするディレクトリ設計とコンポーネント設計"
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

**凝集度（Cohesion）**とは、ソフトウェア工学における重要な概念で、モジュール内の要素がどれだけ密接に関連し合っているかを表す指標です。高凝集度な設計では、関連性の高い機能がまとまっており、低凝集度な設計では、関連性の薄い機能が混在しています。

本記事では、凝集度の概念をReactアプリケーションのディレクトリ設計とコンポーネント設計に適用し、保守性・拡張性・可読性の高いフロントエンドアプリケーションを構築する方法を解説します。

ちなみにこの記事のAI成分は95%です by 著者

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

#### フェーズ1: 最初

プロジェクト開始時は、機能が少ないため自然と機能的凝集になっています。各ファイルが単一の責任を持ち、理想的な状態です。

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

新しい機能（商品）を追加しても、まだ各ファイルの責任は明確です。技術的分類でのディレクトリ構成でも問題ありません。

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

カート機能追加時、価格に関する処理を`priceUtils.ts`としてまとめ始めます。この時点では問題無いように見えますが、論理的凝集の種が蒔かれています。

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

注文機能の追加で「価格関連だから」という理由で無関係な計算処理が`priceUtils.ts`に混入し始めます。ファイルの責任が曖昧になってきます。

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

さらなる機能追加により、表面的な共通点だけで無関係な機能が一つのファイルに集まった状態です。保守性・可読性が大幅に低下しています。

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
export const formatPrice = (price: number) => { /* 商品価格フォーマット */ };
export const calculateTax = (price: number) => { /* 税金計算 */ };
export const calculateShipping = (items: CartItem[]) => { /* 送料計算 */ };
export const applyDiscount = (price: number, rate: number) => { /* 割引適用 */ };
export const calculatePoints = (price: number) => { /* ポイント計算 */ };
export const splitPayment = (total: number, methods: PaymentMethod[]) => { /* 分割払い計算 */ };
export const calculateRefund = (order: Order) => { /* 返金計算 */ };

// utils/validationUtils.ts - 「バリデーション」という名前だけで無関係な機能が集まった状態
export const validateEmail = (email: string) => { /* メール形式チェック */ };
export const validatePassword = (password: string) => { /* パスワード強度チェック */ };
export const validateCreditCard = (number: string) => { /* クレジットカード番号チェック */ };
export const validatePostalCode = (code: string) => { /* 郵便番号チェック */ };
export const validateProductName = (name: string) => { /* 商品名チェック */ };
export const validateReviewText = (text: string) => { /* レビュー内容チェック */ };
```

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

## コンポーネント設計における機能的凝集度

### よくある論理的凝集コンポーネントへの変化パターン

Reactコンポーネントでも同様に、機能追加と共に論理的凝集に陥ってしまう典型的なパターンを見てみましょう。

#### フェーズ1: 最初は機能的凝集（良い状態）

プロジェクト開始時は、各コンポーネントが単一の責任を持ち、理想的な状態です。

```typescript
// UserProfile.tsx - ユーザープロフィール表示のみ
const UserProfile = ({ user }: { user: User }) => {
  return (
    <div className="user-profile">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};
```

#### フェーズ2: 商品機能追加 - まだ機能的凝集

新しいコンポーネントを追加しても、まだ各コンポーネントの責任は明確です。

```typescript
// ProductCard.tsx - 商品表示のみ
const ProductCard = ({ product }: { product: Product }) => {
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>¥{product.price}</p>
    </div>
  );
};
```

#### フェーズ3: カート機能追加 - 論理的凝集の始まり

カート機能追加時、「フォーム関連だから」という理由で`CommonForm`コンポーネントを作り始めます。この時点では問題ないように見えますが、論理的凝集の種が蒔かれています。

```typescript
// CommonForm.tsx - 最初はログインフォームのみ
const CommonForm = ({ type, data, onSubmit }: CommonFormProps) => {
  if (type === 'login') {
    return (
      <form onSubmit={onSubmit}>
        <input type="email" placeholder="メールアドレス" />
        <input type="password" placeholder="パスワード" />
        <button type="submit">ログイン</button>
      </form>
    );
  }
  return null;
};
```

#### フェーズ4: 注文機能追加 - 論理的凝集が深刻化

注文機能の追加で「フォームだから」という理由で無関係なフォーム処理が`CommonForm`に混入し始めます。コンポーネントの責任が曖昧になってきます。

```typescript
// CommonForm.tsx - 関係ない機能が追加され始める
const CommonForm = ({ type, data, onSubmit }: CommonFormProps) => {
  if (type === 'login') {
    return <LoginForm data={data} onSubmit={onSubmit} />;
  }
  // 注文機能追加時に「フォームだから」という理由で追加
  if (type === 'order') {
    return <OrderForm data={data} onSubmit={onSubmit} />;
  }
  // 商品登録機能追加時に「フォームだから」という理由で追加
  if (type === 'product') {
    return <ProductForm data={data} onSubmit={onSubmit} />;
  }
  return null;
};
```

#### フェーズ5: 決済・レビュー機能追加 - 完全に論理的凝集

さらなる機能追加により、「フォーム」という表面的な共通点だけで無関係な機能が一つのコンポーネントに集まった状態です。保守性・可読性が大幅に低下しています。

:::details コードが長文になっているのでアコーディオン内に格納しています

```typescript
// CommonForm.tsx - 「フォーム」という名前だけで無関係な機能が集まった状態
const CommonForm = ({ type, data, onSubmit }: CommonFormProps) => {
  const [loading, setLoading] = useState(false);
  const [errors, setErrors] = useState<any>({});
  const [formData, setFormData] = useState<any>({});

  // すべてのフォームの処理が一つに混在
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    
    if (type === 'login') {
      try {
        await authAPI.login(formData);
        router.push('/dashboard');
      } catch (err) {
        setErrors({ login: 'ログインに失敗しました' });
      }
    } else if (type === 'order') {
      try {
        const orderData = {
          ...formData,
          tax: formData.price * 0.1,
          shipping: calculateShipping(formData.items)
        };
        await orderAPI.create(orderData);
        showSuccessMessage('注文が完了しました');
      } catch (err) {
        setErrors({ order: '注文に失敗しました' });
      }
    } else if (type === 'product') {
      try {
        const productData = {
          ...formData,
          slug: generateSlug(formData.name),
          createdAt: new Date().toISOString()
        };
        await productAPI.create(productData);
        invalidateProductCache();
      } catch (err) {
        setErrors({ product: '商品登録に失敗しました' });
      }
    } else if (type === 'payment') {
      try {
        const encryptedCard = encryptCardData(formData.cardData);
        await paymentAPI.process({
          ...formData,
          cardData: encryptedCard,
          amount: formData.total + formData.fee
        });
        sendReceiptEmail(formData.email);
      } catch (err) {
        setErrors({ payment: '決済に失敗しました' });
      }
    } else if (type === 'review') {
      try {
        const reviewData = {
          ...formData,
          sentiment: analyzeSentiment(formData.text),
          moderated: moderateContent(formData.text)
        };
        await reviewAPI.create(reviewData);
        updateProductRating(formData.productId);
      } catch (err) {
        setErrors({ review: 'レビュー投稿に失敗しました' });
      }
    } else if (type === 'profile') {
      try {
        const userData = {
          ...formData,
          avatar: await uploadAvatar(formData.avatarFile),
          lastUpdated: Date.now()
        };
        await userAPI.update(userData);
        refreshUserSession();
      } catch (err) {
        setErrors({ profile: 'プロフィール更新に失敗しました' });
      }
    }
    
    setLoading(false);
    onSubmit?.(formData);
  };

  // レンダリングも巨大な条件分岐
  const renderFormFields = () => {
    if (type === 'login') {
      return (
        <>
          <input 
            type="email" 
            placeholder="メールアドレス"
            onChange={(e) => setFormData(prev => ({ ...prev, email: e.target.value }))}
          />
          <input 
            type="password" 
            placeholder="パスワード"
            onChange={(e) => setFormData(prev => ({ ...prev, password: e.target.value }))}
          />
        </>
      );
    } else if (type === 'order') {
      return (
        <>
          <textarea 
            placeholder="配送先住所"
            onChange={(e) => setFormData(prev => ({ ...prev, address: e.target.value }))}
          />
          <select onChange={(e) => setFormData(prev => ({ ...prev, paymentMethod: e.target.value }))}>
            <option value="">支払い方法を選択</option>
            <option value="credit">クレジットカード</option>
            <option value="bank">銀行振込</option>
          </select>
        </>
      );
    } else if (type === 'product') {
      return (
        <>
          <input 
            type="text" 
            placeholder="商品名"
            onChange={(e) => setFormData(prev => ({ ...prev, name: e.target.value }))}
          />
          <input 
            type="number" 
            placeholder="価格"
            onChange={(e) => setFormData(prev => ({ ...prev, price: Number(e.target.value) }))}
          />
          <textarea 
            placeholder="商品説明"
            onChange={(e) => setFormData(prev => ({ ...prev, description: e.target.value }))}
          />
        </>
      );
    } else if (type === 'payment') {
      return (
        <>
          <input 
            type="text" 
            placeholder="カード番号"
            onChange={(e) => setFormData(prev => ({ ...prev, cardNumber: e.target.value }))}
          />
          <input 
            type="text" 
            placeholder="MM/YY"
            onChange={(e) => setFormData(prev => ({ ...prev, expiry: e.target.value }))}
          />
          <input 
            type="text" 
            placeholder="CVV"
            onChange={(e) => setFormData(prev => ({ ...prev, cvv: e.target.value }))}
          />
        </>
      );
    } else if (type === 'review') {
      return (
        <>
          <div className="rating">
            {[1, 2, 3, 4, 5].map(star => (
              <button 
                key={star}
                onClick={() => setFormData(prev => ({ ...prev, rating: star }))}
                className={formData.rating >= star ? 'active' : ''}
              >
                ★
              </button>
            ))}
          </div>
          <textarea 
            placeholder="レビューを書く"
            onChange={(e) => setFormData(prev => ({ ...prev, text: e.target.value }))}
          />
        </>
      );
    } else if (type === 'profile') {
      return (
        <>
          <input 
            type="text" 
            placeholder="表示名"
            value={formData.displayName || ''}
            onChange={(e) => setFormData(prev => ({ ...prev, displayName: e.target.value }))}
          />
          <textarea 
            placeholder="自己紹介"
            value={formData.bio || ''}
            onChange={(e) => setFormData(prev => ({ ...prev, bio: e.target.value }))}
          />
        </>
      );
    }
    return null;
  };

  return (
    <form onSubmit={handleSubmit} className={`common-form ${type}-form`}>
      <h2>
        {type === 'login' && 'ログイン'}
        {type === 'order' && '注文情報'}
        {type === 'product' && '商品登録'}
        {type === 'payment' && '決済情報'}
        {type === 'review' && 'レビュー投稿'}
        {type === 'profile' && 'プロフィール編集'}
      </h2>
      
      {Object.values(errors).map((error, index) => (
        <div key={index} className="error">{error}</div>
      ))}
      
      {renderFormFields()}
      
      <button type="submit" disabled={loading}>
        {loading ? '処理中...' : 
         type === 'login' ? 'ログイン' :
         type === 'order' ? '注文確定' :
         type === 'product' ? '商品登録' :
         type === 'payment' ? '決済実行' :
         type === 'review' ? 'レビュー投稿' :
         type === 'profile' ? '更新' : '送信'}
      </button>
    </form>
  );
};
```
:::

**このコンポーネントの問題点：**

1. **6つの無関係な機能が混在**: ログイン、注文、商品登録、決済、レビュー、プロフィール編集
2. **巨大な条件分岐**: type別のif-else文とswitch文が複雑に絡み合う
3. **責任の曖昧さ**: 一つのコンポーネントで認証・EC・ユーザー管理を全て担当
4. **テスト困難**: 6つの機能を1つのコンポーネントでテストする必要
5. **並行開発不可**: 複数の機能を同時に開発することが困難

### カスタムフックによる状態管理の分離

さらに機能的凝集度を高めるため、カスタムフックを使用してロジックとプレゼンテーションを分離します。

```typescript
// features/auth/hooks/useLogin.ts - 認証ロジックのみ
const useLogin = () => {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const login = async (data: LoginData) => {
    setLoading(true);
    setError(null);
    try {
      await authAPI.login(data);
      // ログイン成功処理
    } catch (err) {
      setError('ログインに失敗しました');
    } finally {
      setLoading(false);
    }
  };

  return { login, loading, error };
};

// features/auth/components/LoginForm.tsx - プレゼンテーションのみ
const LoginForm = () => {
  const { login, loading, error } = useLogin();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    login({ email, password });
  };

  return (
    <form onSubmit={handleSubmit} className="login-form">
      {error && <div className="error">{error}</div>}
      <input 
        type="email" 
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="メールアドレス" 
      />
      <input 
        type="password" 
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="パスワード" 
      />
      <button type="submit" disabled={loading}>
        {loading ? 'ログイン中...' : 'ログイン'}
      </button>
    </form>
  );
};
```

このように機能別にコンポーネントを分割し、カスタムフックでロジックを分離することで、各コンポーネントの責任が明確になり、保守性・テスタビリティ・再利用性が大幅に向上します。


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
