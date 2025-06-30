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

  // すべてのフォームのバリデーション処理が混在
  const validateField = (field: string, value: any) => {
    if (type === 'login') {
      if (field === 'email' && !value.includes('@')) {
        setErrors(prev => ({ ...prev, email: 'メール形式が正しくありません' }));
      }
      if (field === 'password' && value.length < 8) {
        setErrors(prev => ({ ...prev, password: 'パスワードは8文字以上です' }));
      }
    } else if (type === 'product') {
      if (field === 'name' && value.length < 2) {
        setErrors(prev => ({ ...prev, name: '商品名は2文字以上です' }));
      }
      if (field === 'price' && value <= 0) {
        setErrors(prev => ({ ...prev, price: '価格は0円より大きくしてください' }));
      }
    } else if (type === 'payment') {
      if (field === 'cardNumber' && !validateCreditCard(value)) {
        setErrors(prev => ({ ...prev, cardNumber: 'カード番号が正しくありません' }));
      }
      if (field === 'cvv' && value.length !== 3) {
        setErrors(prev => ({ ...prev, cvv: 'CVVは3桁です' }));
      }
    }
    // ... 他のフォームのバリデーション処理も延々と続く
  };

  // レンダリングも巨大な条件分岐
  const renderFormFields = () => {
    if (type === 'login') {
      return (
        <>
          <input 
            type="email" 
            placeholder="メールアドレス"
            onChange={(e) => {
              setFormData(prev => ({ ...prev, email: e.target.value }));
              validateField('email', e.target.value);
            }}
          />
          <input 
            type="password" 
            placeholder="パスワード"
            onChange={(e) => {
              setFormData(prev => ({ ...prev, password: e.target.value }));
              validateField('password', e.target.value);
            }}
          />
          <div className="forgot-password">
            <a href="/forgot-password">パスワードを忘れた方</a>
          </div>
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
            <option value="cod">代金引換</option>
          </select>
          <div className="order-summary">
            <p>小計: ¥{formData.subtotal}</p>
            <p>税込: ¥{formData.subtotal * 1.1}</p>
            <p>送料: ¥{calculateShipping(formData.items)}</p>
          </div>
        </>
      );
    } else if (type === 'product') {
      return (
        <>
          <input 
            type="text" 
            placeholder="商品名"
            onChange={(e) => {
              setFormData(prev => ({ ...prev, name: e.target.value }));
              validateField('name', e.target.value);
            }}
          />
          <input 
            type="number" 
            placeholder="価格"
            onChange={(e) => {
              const price = Number(e.target.value);
              setFormData(prev => ({ ...prev, price }));
              validateField('price', price);
            }}
          />
          <textarea 
            placeholder="商品説明"
            onChange={(e) => setFormData(prev => ({ ...prev, description: e.target.value }))}
          />
          <input 
            type="file" 
            accept="image/*"
            onChange={(e) => setFormData(prev => ({ ...prev, imageFile: e.target.files?.[0] }))}
          />
          <select onChange={(e) => setFormData(prev => ({ ...prev, category: e.target.value }))}>
            <option value="">カテゴリを選択</option>
            <option value="electronics">電子機器</option>
            <option value="clothing">衣類</option>
            <option value="books">書籍</option>
          </select>
        </>
      );
    } else if (type === 'payment') {
      return (
        <>
          <input 
            type="text" 
            placeholder="カード番号"
            onChange={(e) => {
              setFormData(prev => ({ ...prev, cardNumber: e.target.value }));
              validateField('cardNumber', e.target.value);
            }}
          />
          <div className="card-details">
            <input 
              type="text" 
              placeholder="MM/YY"
              onChange={(e) => setFormData(prev => ({ ...prev, expiry: e.target.value }))}
            />
            <input 
              type="text" 
              placeholder="CVV"
              onChange={(e) => {
                setFormData(prev => ({ ...prev, cvv: e.target.value }));
                validateField('cvv', e.target.value);
              }}
            />
          </div>
          <input 
            type="text" 
            placeholder="カード名義"
            onChange={(e) => setFormData(prev => ({ ...prev, cardName: e.target.value }))}
          />
          <div className="payment-summary">
            <p>決済金額: ¥{formData.total}</p>
            <p>手数料: ¥{formData.fee}</p>
            <p>合計: ¥{formData.total + formData.fee}</p>
          </div>
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
          <label>
            <input 
              type="checkbox"
              onChange={(e) => setFormData(prev => ({ ...prev, anonymous: e.target.checked }))}
            />
            匿名で投稿
          </label>
          <input 
            type="file" 
            accept="image/*"
            multiple
            onChange={(e) => setFormData(prev => ({ ...prev, images: Array.from(e.target.files || []) }))}
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
          <input 
            type="file" 
            accept="image/*"
            onChange={(e) => setFormData(prev => ({ ...prev, avatarFile: e.target.files?.[0] }))}
          />
          <textarea 
            placeholder="自己紹介"
            value={formData.bio || ''}
            onChange={(e) => setFormData(prev => ({ ...prev, bio: e.target.value }))}
          />
          <div className="privacy-settings">
            <label>
              <input 
                type="checkbox"
                checked={formData.publicProfile || false}
                onChange={(e) => setFormData(prev => ({ ...prev, publicProfile: e.target.checked }))}
              />
              プロフィールを公開する
            </label>
            <label>
              <input 
                type="checkbox"
                checked={formData.emailNotifications || false}
                onChange={(e) => setFormData(prev => ({ ...prev, emailNotifications: e.target.checked }))}
              />
              メール通知を受け取る
            </label>
          </div>
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

// さらに酷い例：UniversalFormModal.tsx - 1500行を超える巨大コンポーネント
const UniversalFormModal = ({ 
  type, 
  data, 
  onSubmit, 
  onClose, 
  isOpen,
  config,
  theme,
  userRole,
  permissions,
  analytics,
  ...props 
}: UniversalFormModalProps) => {
  // 60個以上のuseStateが乱立
  const [loading, setLoading] = useState(false);
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState<any>({});
  const [validationRules, setValidationRules] = useState<any>({});
  const [submitCount, setSubmitCount] = useState(0);
  const [lastSubmitTime, setLastSubmitTime] = useState(0);
  const [shouldShowProgress, setShouldShowProgress] = useState(false);
  const [customStyles, setCustomStyles] = useState<any>({});
  const [dynamicFields, setDynamicFields] = useState<any[]>([]);
  const [conditionalLogic, setConditionalLogic] = useState<any>({});
  const [fileUploads, setFileUploads] = useState<File[]>([]);
  const [previewData, setPreviewData] = useState<any>(null);
  const [autoSaveData, setAutoSaveData] = useState<any>(null);
  const [draftId, setDraftId] = useState<string | null>(null);
  const [isEditing, setIsEditing] = useState(false);
  const [originalData, setOriginalData] = useState<any>(null);
  const [hasUnsavedChanges, setHasUnsavedChanges] = useState(false);
  const [showConfirmDialog, setShowConfirmDialog] = useState(false);
  const [activeTab, setActiveTab] = useState(0);
  const [expandedSections, setExpandedSections] = useState<string[]>([]);
  const [searchQuery, setSearchQuery] = useState('');
  const [selectedItems, setSelectedItems] = useState<any[]>([]);
  const [sortField, setSortField] = useState('');
  const [sortDirection, setSortDirection] = useState<'asc' | 'desc'>('asc');
  const [filterCriteria, setFilterCriteria] = useState<any>({});
  const [bulkActionMode, setBulkActionMode] = useState(false);
  const [exportFormat, setExportFormat] = useState('csv');
  const [isFullscreen, setIsFullscreen] = useState(false);
  const [zoomLevel, setZoomLevel] = useState(100);
  const [language, setLanguage] = useState('ja');
  const [accessibility, setAccessibility] = useState<any>({});
  const [shortcuts, setShortcuts] = useState<any>({});
  const [telemetryData, setTelemetryData] = useState<any>({});
  const [performanceMetrics, setPerformanceMetrics] = useState<any>({});
  const [cacheData, setCacheData] = useState<any>({});
  const [optimisticUpdates, setOptimisticUpdates] = useState<any[]>([]);
  const [retryAttempts, setRetryAttempts] = useState(0);
  const [connectionStatus, setConnectionStatus] = useState('online');
  const [syncStatus, setSyncStatus] = useState('synced');
  const [conflictResolution, setConflictResolution] = useState<any>(null);
  const [workflowState, setWorkflowState] = useState<any>({});
  const [approvalChain, setApprovalChain] = useState<any[]>([]);
  const [auditLog, setAuditLog] = useState<any[]>([]);
  const [notifications, setNotifications] = useState<any[]>([]);
  const [integrationData, setIntegrationData] = useState<any>({});
  const [customExtensions, setCustomExtensions] = useState<any[]>([]);
  const [templateData, setTemplateData] = useState<any>(null);
  const [calculatedFields, setCalculatedFields] = useState<any>({});
  const [dependencies, setDependencies] = useState<any>({});
  const [validationGroups, setValidationGroups] = useState<any[]>([]);
  const [transformRules, setTransformRules] = useState<any>({});
  const [businessRules, setBusinessRules] = useState<any[]>([]);
  const [complianceChecks, setComplianceChecks] = useState<any>({});
  const [dataSourceConfig, setDataSourceConfig] = useState<any>({});
  const [realtimeUpdates, setRealtimeUpdates] = useState<boolean>(false);
  const [collaborativeState, setCollaborativeState] = useState<any>({});
  const [versionControl, setVersionControl] = useState<any>({});
  const [backupState, setBackupState] = useState<any>(null);
  const [scheduledTasks, setScheduledTasks] = useState<any[]>([]);
  const [resourceUsage, setResourceUsage] = useState<any>({});
  // ...さらに多くのstate

  // 巨大なuseEffectの嵐（各々100行以上）
  useEffect(() => {
    // 200行以上の初期化処理
    if (type === 'user-registration') {
      initializeUserRegistration();
    } else if (type === 'product-bulk-edit') {
      initializeProductBulkEdit();
    } else if (type === 'order-workflow') {
      initializeOrderWorkflow();
    } else if (type === 'inventory-sync') {
      initializeInventorySync();
    } else if (type === 'financial-report') {
      initializeFinancialReport();
    } else if (type === 'customer-segmentation') {
      initializeCustomerSegmentation();
    } else if (type === 'marketing-campaign') {
      initializeMarketingCampaign();
    } else if (type === 'support-ticket-batch') {
      initializeSupportTicketBatch();
    } else if (type === 'compliance-audit') {
      initializeComplianceAudit();
    } else if (type === 'integration-setup') {
      initializeIntegrationSetup();
    } else if (type === 'data-migration') {
      initializeDataMigration();
    } else if (type === 'performance-analysis') {
      initializePerformanceAnalysis();
    } else if (type === 'security-assessment') {
      initializeSecurityAssessment();
    } else if (type === 'backup-restore') {
      initializeBackupRestore();
    } else if (type === 'workflow-designer') {
      initializeWorkflowDesigner();
    } else if (type === 'api-configuration') {
      initializeApiConfiguration();
    } else if (type === 'notification-center') {
      initializeNotificationCenter();
    } else if (type === 'analytics-dashboard') {
      initializeAnalyticsDashboard();
    } else if (type === 'content-management') {
      initializeContentManagement();
    } else if (type === 'user-permission-matrix') {
      initializeUserPermissionMatrix();
    }
    // ...さらに25種類以上のtype別処理
  }, [type]);

  // 巨大な関数群（各々200行以上）
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    
    try {
      // 複雑なマルチステップバリデーション
      const validationResult = await executeMultiStepValidation();
      if (!validationResult.isValid) {
        handleValidationErrors(validationResult.errors);
        return;
      }

      // type別の超複雑な処理分岐（各分岐が50-100行）
      if (type === 'user-registration') {
        await handleUserRegistration();
      } else if (type === 'product-bulk-edit') {
        await handleProductBulkEdit();
      } else if (type === 'order-workflow') {
        await handleOrderWorkflow();
      } else if (type === 'inventory-sync') {
        await handleInventorySync();
      } else if (type === 'financial-report') {
        await handleFinancialReport();
      } else if (type === 'customer-segmentation') {
        await handleCustomerSegmentation();
      } else if (type === 'marketing-campaign') {
        await handleMarketingCampaign();
      } else if (type === 'support-ticket-batch') {
        await handleSupportTicketBatch();
      } else if (type === 'compliance-audit') {
        await handleComplianceAudit();
      } else if (type === 'integration-setup') {
        await handleIntegrationSetup();
      } else if (type === 'data-migration') {
        await handleDataMigration();
      } else if (type === 'performance-analysis') {
        await handlePerformanceAnalysis();
      } else if (type === 'security-assessment') {
        await handleSecurityAssessment();
      } else if (type === 'backup-restore') {
        await handleBackupRestore();
      } else if (type === 'workflow-designer') {
        await handleWorkflowDesigner();
      } else if (type === 'api-configuration') {
        await handleApiConfiguration();
      } else if (type === 'notification-center') {
        await handleNotificationCenter();
      } else if (type === 'analytics-dashboard') {
        await handleAnalyticsDashboard();
      } else if (type === 'content-management') {
        await handleContentManagement();
      } else if (type === 'user-permission-matrix') {
        await handleUserPermissionMatrix();
      }
      // ...さらに膨大な分岐処理

    } catch (error) {
      handleSubmissionError(error);
    } finally {
      setLoading(false);
    }
  };

  // 1000行を超える巨大なレンダリング処理
  const renderFormContent = () => {
    // 各typeに対する個別のレンダリング処理（各々100-200行）
    switch (type) {
      case 'user-registration':
        return renderUserRegistrationForm();
      case 'product-bulk-edit':
        return renderProductBulkEditForm();
      case 'order-workflow':
        return renderOrderWorkflowForm();
      case 'inventory-sync':
        return renderInventorySyncForm();
      case 'financial-report':
        return renderFinancialReportForm();
      case 'customer-segmentation':
        return renderCustomerSegmentationForm();
      case 'marketing-campaign':
        return renderMarketingCampaignForm();
      case 'support-ticket-batch':
        return renderSupportTicketBatchForm();
      case 'compliance-audit':
        return renderComplianceAuditForm();
      case 'integration-setup':
        return renderIntegrationSetupForm();
      case 'data-migration':
        return renderDataMigrationForm();
      case 'performance-analysis':
        return renderPerformanceAnalysisForm();
      case 'security-assessment':
        return renderSecurityAssessmentForm();
      case 'backup-restore':
        return renderBackupRestoreForm();
      case 'workflow-designer':
        return renderWorkflowDesignerForm();
      case 'api-configuration':
        return renderApiConfigurationForm();
      case 'notification-center':
        return renderNotificationCenterForm();
      case 'analytics-dashboard':
        return renderAnalyticsDashboardForm();
      case 'content-management':
        return renderContentManagementForm();
      case 'user-permission-matrix':
        return renderUserPermissionMatrixForm();
      default:
        return renderGenericForm();
    }
  };

  // メインレンダリング（さらに300行以上）
  return (
    <div className={`universal-modal ${isOpen ? 'open' : ''} theme-${theme} zoom-${zoomLevel}`}>
      <div className="modal-backdrop" onClick={handleBackdropClick}></div>
      <div className="modal-container">
        <div className="modal-header">
          <div className="header-left">
            <h2>{getModalTitle()}</h2>
            <div className="breadcrumb">{renderBreadcrumb()}</div>
          </div>
          <div className="header-right">
            <div className="header-actions">
              {renderHeaderActions()}
            </div>
            <button className="close-button" onClick={handleClose}>×</button>
          </div>
        </div>
        
        <div className="modal-body">
          {renderProgressIndicator()}
          {renderNotifications()}
          {renderTabNavigation()}
          {renderFormContent()}
          {renderSidebar()}
        </div>
        
        <div className="modal-footer">
          {renderFooterActions()}
          {renderStatusBar()}
        </div>
        
        {renderOverlays()}
        {renderTooltips()}
        {renderContextMenus()}
        {renderConfirmDialogs()}
      </div>
    </div>
  );
};
```

**この巨大コンポーネントの深刻な問題点：**

1. **60個以上のstate管理**: 関連性のないstateが混在し、どのstateがどの機能に属するか不明
2. **1500行を超えるコード**: ファイルが巨大すぎて、全体を把握することが不可能
3. **20種類以上の機能**: ユーザー登録、在庫管理、財務レポート、セキュリティ評価など無関係な機能が混在
4. **複雑な条件分岐**: type別の処理がswitch-case地獄を作り出している
5. **テスト困難**: 一つのコンポーネントで20種類以上の機能をテストする必要
6. **並行開発不可**: 複数の機能を同時に開発することが物理的に不可能
7. **パフォーマンス問題**: 不要な再レンダリング、メモリリーク、初期化の遅延
8. **デバッグ地獄**: エラーの原因特定に数時間から数日かかる
9. **知識の属人化**: このコンポーネントを理解できる人が1-2名に限られる
10. **技術的負債の蓄積**: 新機能追加のたびに複雑さが指数関数的に増加

**実際に現場で起こる悲劇的な状況：**

- **新機能追加に3倍の工数**: 本来1日でできる機能追加が3日かかる
- **バグ修正の影響範囲拡大**: ユーザー登録のバグ修正で決済機能が壊れる
- **リリースの延期**: 一つの機能のバグで全ての機能がリリースできない
- **開発者の離職**: 新しいメンバーがこのコードを見て絶望し、転職を検討
- **技術的負債の雪だるま式増加**: 修正のたびに複雑さが増し、手に負えなくなる


### 論理的凝集コンポーネントの問題点

上記のような巨大な論理的凝集コンポーネントが生まれると、開発チーム全体に深刻な影響を与えます。これはコードリーディング（プログラム理解）の研究で明らかになっている問題と直結しています。

**研究で明らかになった事実：**
開発者は作業時間の58〜70%をコード理解に費やしています（Xia et al. 2018, Feitelson et al. 2023）。論理的凝集のような低品質なコードは、この理解時間を更に増加させ、開発効率を著しく低下させます。

**UniversalFormModalの具体的な問題：**
- **ユーザー登録フォーム**（認証機能）
- **商品一括編集フォーム**（商品管理機能）
- **注文ワークフローフォーム**（注文機能）
- **在庫同期フォーム**（在庫管理機能）
- **財務レポートフォーム**（会計機能）
- **顧客セグメンテーションフォーム**（マーケティング機能）
- **セキュリティ評価フォーム**（セキュリティ機能）

これらは「フォーム」「モーダル」という表面的な共通点はあるものの、実際には全く異なるビジネス機能に属しており、変更理由も変更タイミングも異なります。

**実際に起こる問題：**
1. **影響範囲の拡大**: ログインフォームのスタイルを変更したいだけなのに、決済・レビューフォームのテストも必要になる
2. **並行開発の阻害**: 複数チームが同じコンポーネントを同時に変更してコンフリクトが発生
3. **理解の困難**: 新しいメンバーが「なぜこれらのフォームが一つのコンポーネントにあるのか」理解できない
4. **テストの複雑化**: 一つのコンポーネントで多数の機能をテストする必要がある

### 機能的凝集コンポーネントへの改善例

論理的凝集を機能的凝集に改善するには、単一責任の原則に従い、機能ごとに独立したコンポーネントを作成することが効果的です。

```typescript
// features/auth/components/LoginForm.tsx - 認証機能のみ
const LoginForm = ({ onSubmit }: { onSubmit: (data: LoginData) => void }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit({ email, password });
  };

  return (
    <form onSubmit={handleSubmit} className="login-form">
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
      <button type="submit">ログイン</button>
    </form>
  );
};

// features/order/components/OrderForm.tsx - 注文機能のみ
const OrderForm = ({ onSubmit }: { onSubmit: (data: OrderData) => void }) => {
  const [shippingAddress, setShippingAddress] = useState('');
  const [paymentMethod, setPaymentMethod] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit({ shippingAddress, paymentMethod });
  };

  return (
    <form onSubmit={handleSubmit} className="order-form">
      <textarea 
        value={shippingAddress}
        onChange={(e) => setShippingAddress(e.target.value)}
        placeholder="配送先住所" 
      />
      <select 
        value={paymentMethod}
        onChange={(e) => setPaymentMethod(e.target.value)}
      >
        <option value="">支払い方法を選択</option>
        <option value="credit">クレジットカード</option>
        <option value="bank">銀行振込</option>
      </select>
      <button type="submit">注文確定</button>
    </form>
  );
};

// features/product/components/ProductForm.tsx - 商品管理機能のみ
const ProductForm = ({ onSubmit }: { onSubmit: (data: ProductData) => void }) => {
  const [name, setName] = useState('');
  const [price, setPrice] = useState(0);
  const [description, setDescription] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit({ name, price, description });
  };

  return (
    <form onSubmit={handleSubmit} className="product-form">
      <input 
        type="text" 
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="商品名" 
      />
      <input 
        type="number" 
        value={price}
        onChange={(e) => setPrice(Number(e.target.value))}
        placeholder="価格" 
      />
      <textarea 
        value={description}
        onChange={(e) => setDescription(e.target.value)}
        placeholder="商品説明" 
      />
      <button type="submit">商品登録</button>
    </form>
  );
};
```

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
