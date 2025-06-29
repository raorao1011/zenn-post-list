---
title: "機能的凝集度からアプローチするディレクトリ設計とコンポーネント設計"
emoji: "💪" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech"
topics: ["react", "typescript", "frontend"]
published: false
---

## はじめに

フロントエンド開発において、プロジェクトが大規模化するにつれて、コードの保守性や可読性が課題となることがあります。特に以下のような問題に直面したことはありませんか？

- 「この機能を修正したいが、関連するファイルがプロジェクト全体に散らばっている」
- 「新しい機能を追加する際、どこに配置すべきか迷う」
- 「コンポーネントが複雑になりすぎて、テストやデバッグが困難」
- 「チーム開発で、他の人が書いたコードの意図を理解するのに時間がかかる」

これらの問題の根本的な原因の一つは、**凝集度**の低い設計にあります。

**凝集度（Cohesion）**とは、ソフトウェア工学における重要な概念で、モジュール内の要素がどれだけ密接に関連し合っているかを表す指標です。高凝集度な設計では、関連性の高い機能がまとまっており、低凝集度な設計では、関連性の薄い機能が混在しています。

本記事では、凝集度の概念をReactアプリケーションのディレクトリ設計とコンポーネント設計に適用し、保守性・拡張性・可読性の高いフロントエンドアプリケーションを構築する方法を解説します。

### 本記事で学べること

- 凝集度の基本概念と種類
- フロントエンド開発における凝集度の具体的な適用例
- 機能的凝集度に基づくディレクトリ構成
- 単一責任を持つReactコンポーネントの設計手法
- TypeScriptを活用した型安全で高凝集な設計パターン
- チーム開発での実践的な導入戦略

## 凝集度とは何か

### ソフトウェア工学における凝集度の定義
- 凝集度の基本概念
- 高凝集度と低凝集度の違い
- メンテナンス性・可読性への影響

### 凝集度の種類と特徴
- 偶然的凝集：関連性のない要素が偶然まとまっている状態
- 論理的凝集：似たような処理をまとめているが、実際の関係は薄い状態
- 時間的凝集：同じタイミングで実行される処理をまとめた状態
- 手順的凝集：処理の順序に基づいてまとめられた状態
- 連絡的凝集：同じデータを扱う処理がまとまっている状態
- 情報的凝集：同じデータ構造を操作する処理がまとまっている状態
- 機能的凝集：単一の明確な目的を持つ処理がまとまっている状態（理想的）

## 凝集度の具体例とフロントエンド開発への適用

### 低凝集度の例：偶然的凝集・論理的凝集
```typescript
// 悪い例：utils.ts（偶然的凝集）
export const formatDate = (date: Date) => { /* ... */ };
export const validateEmail = (email: string) => { /* ... */ };
export const calculateTax = (price: number) => { /* ... */ };
export const sortArray = (arr: any[]) => { /* ... */ };

// 悪い例：共通コンポーネント（論理的凝集）
const CommonModal = ({ type, data, onSubmit }) => {
  if (type === 'user') return <UserForm data={data} onSubmit={onSubmit} />;
  if (type === 'product') return <ProductForm data={data} onSubmit={onSubmit} />;
  if (type === 'order') return <OrderForm data={data} onSubmit={onSubmit} />;
  return null;
};
```

### 中程度の凝集度：時間的凝集・手順的凝集
```typescript
// 時間的凝集の例：初期化処理
const useAppInitialization = () => {
  useEffect(() => {
    // 同時に実行されるが、関連性は薄い
    loadUserSettings();
    initializeAnalytics();
    setupErrorHandling();
    preloadImages();
  }, []);
};

// 手順的凝集の例：フォーム処理
const useFormSubmission = () => {
  const handleSubmit = async (data) => {
    validateForm(data);        // ステップ1
    sanitizeInput(data);       // ステップ2
    submitToAPI(data);         // ステップ3
    showSuccessMessage();      // ステップ4
    redirectToNextPage();      // ステップ5
  };
};
```

### 高凝集度の例：情報的凝集・機能的凝集
```typescript
// 情報的凝集の例：ユーザー情報の操作
const useUserProfile = () => {
  const [user, setUser] = useState(null);
  
  const updateProfile = (profileData) => { /* ... */ };
  const changePassword = (passwordData) => { /* ... */ };
  const updateAvatar = (avatarFile) => { /* ... */ };
  const deleteAccount = () => { /* ... */ };
  
  return { user, updateProfile, changePassword, updateAvatar, deleteAccount };
};

// 機能的凝集の例：商品カート操作
const useShoppingCart = () => {
  const [items, setItems] = useState([]);
  
  const addItem = (product) => { /* 商品追加の単一責任 */ };
  const removeItem = (productId) => { /* 商品削除の単一責任 */ };
  const updateQuantity = (productId, quantity) => { /* 数量更新の単一責任 */ };
  const clearCart = () => { /* カートクリアの単一責任 */ };
  const getTotalPrice = () => { /* 合計計算の単一責任 */ };
  
  return { items, addItem, removeItem, updateQuantity, clearCart, getTotalPrice };
};
```

### React コンポーネントにおける凝集度の比較
```typescript
// 低凝集度：多くの責任を持つコンポーネント
const UserDashboard = () => {
  // ユーザー情報管理
  const [user, setUser] = useState(null);
  // 通知管理
  const [notifications, setNotifications] = useState([]);
  // テーマ管理
  const [theme, setTheme] = useState('light');
  // API呼び出し
  const fetchUserData = () => { /* ... */ };
  // 分析イベント送信
  const trackEvent = () => { /* ... */ };
  
  return (
    <div>
      {/* 複数の関心事が混在 */}
      <UserProfile user={user} />
      <NotificationList notifications={notifications} />
      <ThemeSelector theme={theme} onChange={setTheme} />
    </div>
  );
};

// 高凝集度：単一責任を持つコンポーネント
const UserProfile = ({ user }) => {
  // ユーザープロフィール表示のみに集中
  return (
    <div className="user-profile-card">
      <img src={user.avatar} alt={`${user.name}のアバター`} className="avatar" />
      <p className="user-name">{user.name}</p>
      <p className="user-email">{user.email}</p>
    </div>
  );
};

const NotificationCenter = ({ userId }) => {
  // 通知機能のみに集中
  const { notifications, markAsRead, clearAll } = useNotifications(userId);
  
  return (
    <div className="notification-center">
      {notifications.map(notification => (
        <NotificationItem 
          key={notification.id} 
          notification={notification} 
          onRead={markAsRead} 
        />
      ))}
    </div>
  );
};
```

## ディレクトリ設計における凝集度

### 従来のディレクトリ構成の問題点

多くのプロジェクトで見られる技術的分類によるディレクトリ構成は、プロジェクトの成長とともに問題を引き起こします：

```
src/
├── components/
│   ├── Header.tsx
│   ├── Footer.tsx
│   ├── ProductCard.tsx
│   ├── UserProfile.tsx
│   ├── LoginForm.tsx
│   ├── CartItem.tsx
│   └── OrderSummary.tsx
├── hooks/
│   ├── useAuth.ts
│   ├── useCart.ts
│   ├── useProduct.ts
│   └── useOrder.ts
├── utils/
│   ├── dateUtils.ts
│   ├── cartUtils.ts
│   ├── authUtils.ts
│   └── productUtils.ts
└── pages/
    ├── LoginPage.tsx
    ├── ProductListPage.tsx
    ├── CartPage.tsx
    └── OrderPage.tsx
```

**この構成の問題点：**

1. **機能追加時の複雑性**: 新しい機能（例：レビュー機能）を追加する際、components/, hooks/, utils/, pages/ すべてのディレクトリを横断してファイルを作成・修正する必要がある

2. **関連性の把握困難**: カート機能に関連するファイルが複数のディレクトリに散らばっており、全体像を把握するのが困難

3. **スケールしない構造**: プロジェクトが大きくなると、各ディレクトリ内のファイル数が増加し、目的のファイルを見つけるのが困難

4. **チーム開発での衝突**: 複数の開発者が同じディレクトリ内の異なるファイルを同時に編集することで、merge conflict が発生しやすい

### 機能的凝集度に基づくディレクトリ設計

機能的凝集度の原則に従い、関連する機能をまとめた Feature-based なディレクトリ構成を採用します：

```
src/
├── features/                    # 機能別のディレクトリ
│   ├── auth/                   # 認証機能
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── SignupForm.tsx
│   │   │   └── UserProfile.tsx
│   │   ├── hooks/
│   │   │   ├── useAuth.ts
│   │   │   └── useUserProfile.ts
│   │   ├── services/
│   │   │   └── authAPI.ts
│   │   ├── types/
│   │   │   └── user.ts
│   │   └── index.ts            # パブリックAPI
│   │
│   ├── product/                # 商品機能
│   │   ├── components/
│   │   │   ├── ProductCard.tsx
│   │   │   ├── ProductList.tsx
│   │   │   ├── ProductFilter.tsx
│   │   │   └── ProductDetail.tsx
│   │   ├── hooks/
│   │   │   ├── useProductList.ts
│   │   │   └── useProductDetail.ts
│   │   ├── services/
│   │   │   └── productAPI.ts
│   │   ├── types/
│   │   │   └── product.ts
│   │   └── index.ts
│   │
│   ├── cart/                   # カート機能
│   │   ├── components/
│   │   │   ├── CartItem.tsx
│   │   │   ├── CartSummary.tsx
│   │   │   └── AddToCartButton.tsx
│   │   ├── hooks/
│   │   │   └── useCart.ts
│   │   ├── services/
│   │   │   └── cartAPI.ts
│   │   ├── types/
│   │   │   └── cart.ts
│   │   └── index.ts
│   │
│   └── order/                  # 注文機能
│       ├── components/
│       │   ├── OrderForm.tsx
│       │   ├── OrderSummary.tsx
│       │   └── OrderHistory.tsx
│       ├── hooks/
│       │   ├── useOrder.ts
│       │   └── useOrderHistory.ts
│       ├── services/
│       │   └── orderAPI.ts
│       ├── types/
│       │   └── order.ts
│       └── index.ts
│
├── shared/                     # 共通機能
│   ├── components/
│   │   ├── Button.tsx
│   │   ├── Modal.tsx
│   │   └── LoadingSpinner.tsx
│   ├── hooks/
│   │   ├── useLocalStorage.ts
│   │   └── useDebounce.ts
│   ├── utils/
│   │   ├── dateUtils.ts
│   │   ├── formatUtils.ts
│   │   └── validationUtils.ts
│   └── types/
│       └── common.ts
│
├── app/                        # アプリケーション設定
│   ├── App.tsx
│   ├── router.tsx
│   └── store.ts
│
└── pages/                      # ページコンポーネント
    ├── LoginPage.tsx
    ├── ProductListPage.tsx
    ├── CartPage.tsx
    └── OrderPage.tsx
```

### Feature モジュールの設計原則

各featureディレクトリは以下の原則に従って設計します：

#### 1. 公開APIの明確化
```typescript
// features/product/index.ts
export { ProductCard, ProductList, ProductFilter } from './components';
export { useProductList, useProductDetail } from './hooks';
export type { Product, ProductFilter as ProductFilterType } from './types';

// 内部実装は非公開
// export { ProductAPI } from './services'; // ❌ 内部実装は公開しない
```

#### 2. 依存関係の管理
```typescript
// features/cart/hooks/useCart.ts
import { Product } from '@/features/product'; // ✅ 他のfeatureから型のみインポート
import { cartAPI } from '../services/cartAPI';  // ✅ 同一feature内のサービス

// import { ProductAPI } from '@/features/product/services'; // ❌ 他のfeatureの内部実装に依存しない
```

#### 3. 循環依存の回避
```typescript
// 悪い例：循環依存
// features/product/components/ProductCard.tsx
import { AddToCartButton } from '@/features/cart'; // ❌

// 良い例：共通コンポーネントまたはpropsで解決
// features/product/components/ProductCard.tsx
interface ProductCardProps {
  product: Product;
  onAddToCart?: (product: Product) => void; // ✅ props経由で機能を注入
}
```

### 実装例：商品機能の完全な実装

```typescript
// features/product/types/product.ts
export interface Product {
  id: string;
  name: string;
  price: number;
  image: string;
  category: string;
  rating: number;
}

export interface ProductFilter {
  category: string;
  priceRange: [number, number];
  searchTerm: string;
}

// features/product/services/productAPI.ts
export const productAPI = {
  async getProducts(filter: ProductFilter): Promise<Product[]> {
    const params = new URLSearchParams({
      category: filter.category,
      minPrice: filter.priceRange[0].toString(),
      maxPrice: filter.priceRange[1].toString(),
      search: filter.searchTerm,
    });
    
    const response = await fetch(`/api/products?${params}`);
    return response.json();
  },

  async getProductById(id: string): Promise<Product> {
    const response = await fetch(`/api/products/${id}`);
    return response.json();
  }
};

// features/product/hooks/useProductList.ts
import { useState, useEffect } from 'react';
import { Product, ProductFilter } from '../types/product';
import { productAPI } from '../services/productAPI';

export const useProductList = () => {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [filter, setFilter] = useState<ProductFilter>({
    category: 'all',
    priceRange: [0, 1000],
    searchTerm: ''
  });

  useEffect(() => {
    const fetchProducts = async () => {
      setLoading(true);
      setError(null);
      try {
        const data = await productAPI.getProducts(filter);
        setProducts(data);
      } catch (err) {
        setError('商品の取得に失敗しました');
      } finally {
        setLoading(false);
      }
    };

    fetchProducts();
  }, [filter]);

  return {
    products,
    loading,
    error,
    filter,
    setFilter
  };
};

// features/product/components/ProductCard.tsx
import React from 'react';
import { Product } from '../types/product';

interface ProductCardProps {
  product: Product;
  onAddToCart?: (product: Product) => void;
  onToggleFavorite?: (productId: string) => void;
  isFavorite?: boolean;
}

export const ProductCard: React.FC<ProductCardProps> = ({
  product,
  onAddToCart,
  onToggleFavorite,
  isFavorite = false
}) => {
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>¥{product.price.toLocaleString()}</p>
      <div className="actions">
        {onAddToCart && (
          <button onClick={() => onAddToCart(product)}>
            カートに追加
          </button>
        )}
        {onToggleFavorite && (
          <button 
            className={`favorite ${isFavorite ? 'active' : ''}`}
            onClick={() => onToggleFavorite(product.id)}
          >
            {isFavorite ? '♥' : '♡'}
          </button>
        )}
      </div>
    </div>
  );
};
```

このFeature-basedなディレクトリ構成により、以下のメリットが得られます：

**機能的凝集度の効果:**
- 関連する機能が一箇所にまとまっている
- 新機能追加時の影響範囲が明確
- 機能単位でのテスト・デバッグが容易
- チーム開発での作業分担が明確
- コードの再利用性が向上

## Reactコンポーネント設計における凝集度

### コンポーネントの責任分離
- 単一責任の原則
- プレゼンテーションとロジックの分離
- カスタムフックの活用

### 高凝集度なコンポーネントの特徴
- 明確な責任範囲
- 適切な抽象化レベル
- 再利用可能性と保守性

### 機能的凝集度による実践的なコンポーネント分割例

#### 低凝集度：商品一覧ページの悪い例
```typescript
// 悪い例：複数の責任を持つ巨大なコンポーネント
const ProductListPage = () => {
  // 商品データの管理
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(false);
  
  // フィルタリング機能
  const [category, setCategory] = useState('all');
  const [priceRange, setPriceRange] = useState([0, 1000]);
  const [searchTerm, setSearchTerm] = useState('');
  
  // ソート機能
  const [sortBy, setSortBy] = useState('name');
  const [sortOrder, setSortOrder] = useState('asc');
  
  // ページネーション
  const [currentPage, setCurrentPage] = useState(1);
  const [itemsPerPage] = useState(12);
  
  // カート機能
  const [cartItems, setCartItems] = useState([]);
  
  // お気に入り機能
  const [favorites, setFavorites] = useState([]);
  
  // API呼び出し、フィルタリング、ソート、ページング、カート操作など
  // すべての処理がここに集約されている
  const fetchProducts = async () => { /* ... */ };
  const filterProducts = () => { /* ... */ };
  const sortProducts = () => { /* ... */ };
  const addToCart = (product) => { /* ... */ };
  const toggleFavorite = (productId) => { /* ... */ };
  
  return (
    <div className="product-list-container">
      {/* フィルター UI */}
      <div className="filter-controls">
        <select value={category} onChange={(e) => setCategory(e.target.value)}>
          <option value="all">すべて</option>
          <option value="electronics">電子機器</option>
          <option value="clothing">衣類</option>
        </select>
        <input 
          type="range" 
          min={0} 
          max={1000}
          value={priceRange[1]}
          onChange={(e) => setPriceRange([0, parseInt(e.target.value)])}
        />
        <input 
          type="text"
          placeholder="商品名で検索" 
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
      </div>
      
      {/* ソート UI */}
      <div className="sort-controls">
        <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
          <option value="name">名前</option>
          <option value="price">価格</option>
          <option value="rating">評価</option>
        </select>
        <button onClick={() => setSortOrder(sortOrder === 'asc' ? 'desc' : 'asc')}>
          {sortOrder === 'asc' ? '昇順' : '降順'}
        </button>
      </div>
      
      {/* 商品グリッド */}
      <div className="product-grid">
        {products.map(product => (
          <div key={product.id} className="product-card">
            <img src={product.image} alt={product.name} />
            <p className="product-name">{product.name}</p>
            <p className="product-price">¥{product.price}</p>
            <div className="product-actions">
              <button onClick={() => addToCart(product)}>カートに追加</button>
              <button 
                className={favorites.includes(product.id) ? 'favorite active' : 'favorite'}
                onClick={() => toggleFavorite(product.id)}
              >
                {favorites.includes(product.id) ? '♥' : '♡'}
              </button>
            </div>
          </div>
        ))}
      </div>
      
      {/* ページネーション */}
      <div className="pagination">
        <button 
          onClick={() => setCurrentPage(prev => Math.max(1, prev - 1))}
          disabled={currentPage === 1}
        >
          前へ
        </button>
        <span>{currentPage}</span>
        <button onClick={() => setCurrentPage(prev => prev + 1)}>
          次へ
        </button>
      </div>
    </div>
  );
};
```

#### 高凝集度：機能別に分割した良い例
```typescript
// 良い例：単一責任を持つコンポーネントに分割

// 1. 商品フィルターコンポーネント（フィルタリング機能のみ）
const ProductFilter = ({ filters, onFiltersChange }) => {
  const handleCategoryChange = (category) => {
    onFiltersChange({ ...filters, category });
  };
  
  const handlePriceRangeChange = (maxPrice) => {
    onFiltersChange({ ...filters, priceRange: [0, maxPrice] });
  };
  
  const handleSearchChange = (searchTerm) => {
    onFiltersChange({ ...filters, searchTerm });
  };
  
  return (
    <div className="product-filter">
      <select value={filters.category} onChange={(e) => handleCategoryChange(e.target.value)}>
        <option value="all">すべて</option>
        <option value="electronics">電子機器</option>
        <option value="clothing">衣類</option>
      </select>
      <div className="price-filter">
        <label>価格: ¥0 - ¥{filters.priceRange[1]}</label>
        <input 
          type="range"
          min={0}
          max={1000}
          value={filters.priceRange[1]}
          onChange={(e) => handlePriceRangeChange(parseInt(e.target.value))}
        />
      </div>
      <input 
        type="text"
        placeholder="商品名で検索" 
        value={filters.searchTerm}
        onChange={(e) => handleSearchChange(e.target.value)}
      />
    </div>
  );
};

// 2. ソートコンポーネント（ソート機能のみ）
const ProductSort = ({ sortConfig, onSortChange }) => {
  const handleSortByChange = (sortBy) => {
    onSortChange({ ...sortConfig, sortBy });
  };
  
  const toggleSortOrder = () => {
    onSortChange({ 
      ...sortConfig, 
      sortOrder: sortConfig.sortOrder === 'asc' ? 'desc' : 'asc' 
    });
  };
  
  return (
    <div className="product-sort">
      <select value={sortConfig.sortBy} onChange={(e) => handleSortByChange(e.target.value)}>
        <option value="name">名前</option>
        <option value="price">価格</option>
        <option value="rating">評価</option>
      </select>
      <button onClick={toggleSortOrder}>
        {sortConfig.sortOrder === 'asc' ? '昇順' : '降順'}
      </button>
    </div>
  );
};

// 3. 単一商品カードコンポーネント（商品表示機能のみ）
const ProductCard = ({ product, onAddToCart, onToggleFavorite, isFavorite }) => {
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <div className="product-info">
        <h3 className="product-name">{product.name}</h3>
        <p className="product-price">¥{product.price.toLocaleString()}</p>
        <div className="product-actions">
          <button 
            className="add-to-cart-btn"
            onClick={() => onAddToCart(product)}
          >
            カートに追加
          </button>
          <button 
            className={`favorite-btn ${isFavorite ? 'active' : ''}`}
            onClick={() => onToggleFavorite(product.id)}
          >
            {isFavorite ? '♥' : '♡'}
          </button>
        </div>
      </div>
    </div>
  );
};

// 4. 商品グリッドコンポーネント（商品一覧表示機能のみ）
const ProductGrid = ({ products, onAddToCart, onToggleFavorite, favorites }) => {
  return (
    <div className="product-grid">
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={onAddToCart}
          onToggleFavorite={onToggleFavorite}
          isFavorite={favorites.includes(product.id)}
        />
      ))}
    </div>
  );
};

// 5. ページネーションコンポーネント（ページング機能のみ）
const Pagination = ({ currentPage, totalPages, onPageChange }) => {
  return (
    <div className="pagination">
      <button 
        disabled={currentPage === 1}
        onClick={() => onPageChange(currentPage - 1)}
      >
        前へ
      </button>
      <span className="page-info">{currentPage} / {totalPages}</span>
      <button 
        disabled={currentPage === totalPages}
        onClick={() => onPageChange(currentPage + 1)}
      >
        次へ
      </button>
    </div>
  );
};

// 6. 最終的な商品一覧ページ（調整・統合機能のみ）
const ProductListPage = () => {
  // カスタムフックで状態管理を分離
  const { 
    products, 
    loading, 
    filters, 
    sortConfig, 
    currentPage, 
    totalPages,
    setFilters,
    setSortConfig,
    setCurrentPage
  } = useProductList();
  
  const { addToCart } = useCart();
  const { favorites, toggleFavorite } = useFavorites();
  
  return (
    <div className="product-list-page">
      <h1>商品一覧</h1>
      
      <div className="page-layout">
        <aside className="sidebar">
          <ProductFilter filters={filters} onFiltersChange={setFilters} />
        </aside>
        
        <main className="main-content">
          <ProductSort sortConfig={sortConfig} onSortChange={setSortConfig} />
          
          {loading ? (
            <div className="loading">読み込み中...</div>
          ) : (
            <>
              <ProductGrid 
                products={products}
                onAddToCart={addToCart}
                onToggleFavorite={toggleFavorite}
                favorites={favorites}
              />
              <Pagination 
                currentPage={currentPage}
                totalPages={totalPages}
                onPageChange={setCurrentPage}
              />
            </>
          )}
        </main>
      </div>
    </div>
  );
};
```

#### カスタムフックによる状態管理の分離
```typescript
// 商品一覧の状態管理（情報的凝集）
const useProductList = () => {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(false);
  const [filters, setFilters] = useState({
    category: 'all',
    priceRange: [0, 1000],
    searchTerm: ''
  });
  const [sortConfig, setSortConfig] = useState({
    sortBy: 'name',
    sortOrder: 'asc'
  });
  const [currentPage, setCurrentPage] = useState(1);
  const [totalPages, setTotalPages] = useState(1);
  
  // フィルタリング、ソート、ページングロジック
  useEffect(() => {
    fetchProducts();
  }, [filters, sortConfig, currentPage]);
  
  const fetchProducts = async () => {
    setLoading(true);
    try {
      const response = await productAPI.getProducts({
        ...filters,
        ...sortConfig,
        page: currentPage
      });
      setProducts(response.products);
      setTotalPages(response.totalPages);
    } catch (error) {
      console.error('商品取得エラー:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return {
    products,
    loading,
    filters,
    sortConfig,
    currentPage,
    totalPages,
    setFilters,
    setSortConfig,
    setCurrentPage
  };
};

// カート機能（機能的凝集）
const useCart = () => {
  const [items, setItems] = useState([]);
  
  const addToCart = useCallback((product) => {
    setItems(prev => {
      const existingItem = prev.find(item => item.id === product.id);
      if (existingItem) {
        return prev.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [...prev, { ...product, quantity: 1 }];
    });
  }, []);
  
  return { items, addToCart };
};

// お気に入り機能（機能的凝集）
const useFavorites = () => {
  const [favorites, setFavorites] = useState([]);
  
  const toggleFavorite = useCallback((productId) => {
    setFavorites(prev => 
      prev.includes(productId)
        ? prev.filter(id => id !== productId)
        : [...prev, productId]
    );
  }, []);
  
  return { favorites, toggleFavorite };
};
```

この分割により、以下のメリットが得られます：

**高凝集度の効果:**
- 各コンポーネントが単一の責任を持つ
- テストしやすい小さな単位
- 再利用可能なコンポーネント
- 変更時の影響範囲が限定的
- 理解しやすいコード構造

## 実践的な設計パターン

### Atomic Designと凝集度の関係

Atomic Designの各レベルにおいて、凝集度の原則を適用することで、より保守性の高いコンポーネント設計が可能になります：

#### Atoms（原子）- 機能的凝集
```typescript
// 高凝集度：単一の明確な責任を持つ
const Button: React.FC<ButtonProps> = ({ 
  children, 
  variant = 'primary', 
  disabled = false,
  onClick 
}) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
};

// 低凝集度：複数の責任を持つ（アンチパターン）
const MultiButton: React.FC = ({ type, data, onSubmit }) => {
  if (type === 'save') return <button onClick={() => onSubmit('save')}>保存</button>;
  if (type === 'delete') return <button onClick={() => onSubmit('delete')}>削除</button>;
  if (type === 'cancel') return <button onClick={() => onSubmit('cancel')}>キャンセル</button>;
  return null;
};
```

#### Molecules（分子）- 情報的凝集
```typescript
// 関連するatomsを組み合わせて単一の機能を実現
const SearchBox: React.FC<SearchBoxProps> = ({
  value,
  placeholder,
  onSearch,
  onClear
}) => {
  return (
    <div className="search-box">
      <input
        type="text"
        value={value}
        placeholder={placeholder}
        onChange={(e) => onSearch(e.target.value)}
      />
      <Button onClick={onClear} variant="secondary">
        クリア
      </Button>
    </div>
  );
};
```

#### Organisms（有機体）- 機能的凝集
```typescript
// 複数のmoleculesを組み合わせて完結した機能を提供
const ProductSearchSection: React.FC = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const [filters, setFilters] = useState<ProductFilter>({
    category: 'all',
    priceRange: [0, 1000],
    searchTerm: ''
  });

  const handleSearch = useCallback((term: string) => {
    setSearchTerm(term);
    setFilters(prev => ({ ...prev, searchTerm: term }));
  }, []);

  const handleClearSearch = useCallback(() => {
    setSearchTerm('');
    setFilters(prev => ({ ...prev, searchTerm: '' }));
  }, []);

  return (
    <section className="product-search-section">
      <SearchBox
        value={searchTerm}
        placeholder="商品を検索..."
        onSearch={handleSearch}
        onClear={handleClearSearch}
      />
      <ProductFilter
        filters={filters}
        onFiltersChange={setFilters}
      />
    </section>
  );
};
```

### カスタムフックによる状態管理の凝集化

#### ビジネスロジックの集約例

```typescript
// 商品詳細の状態管理（情報的凝集）
export const useProductDetail = (productId: string) => {
  const [product, setProduct] = useState<Product | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [reviews, setReviews] = useState<Review[]>([]);
  const [relatedProducts, setRelatedProducts] = useState<Product[]>([]);

  const fetchProductDetail = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const [productData, reviewsData, relatedData] = await Promise.all([
        productAPI.getProductById(productId),
        productAPI.getProductReviews(productId),
        productAPI.getRelatedProducts(productId)
      ]);
      
      setProduct(productData);
      setReviews(reviewsData);
      setRelatedProducts(relatedData);
    } catch (err) {
      setError('商品情報の取得に失敗しました');
    } finally {
      setLoading(false);
    }
  }, [productId]);

  useEffect(() => {
    fetchProductDetail();
  }, [fetchProductDetail]);

  return {
    product,
    reviews,
    relatedProducts,
    loading,
    error,
    refetch: fetchProductDetail
  };
};

// ショッピングカートの状態管理（機能的凝集）
export const useShoppingCart = () => {
  const [items, setItems] = useState<CartItem[]>([]);

  const addItem = useCallback((product: Product, quantity: number = 1) => {
    setItems(prev => {
      const existingItem = prev.find(item => item.product.id === product.id);
      
      if (existingItem) {
        return prev.map(item =>
          item.product.id === product.id
            ? { ...item, quantity: item.quantity + quantity }
            : item
        );
      }
      
      return [...prev, { product, quantity }];
    });
  }, []);

  const removeItem = useCallback((productId: string) => {
    setItems(prev => prev.filter(item => item.product.id !== productId));
  }, []);

  const updateQuantity = useCallback((productId: string, quantity: number) => {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }
    
    setItems(prev =>
      prev.map(item =>
        item.product.id === productId
          ? { ...item, quantity }
          : item
      )
    );
  }, [removeItem]);

  const clearCart = useCallback(() => {
    setItems([]);
  }, []);

  const getTotalPrice = useCallback(() => {
    return items.reduce((total, item) => {
      return total + (item.product.price * item.quantity);
    }, 0);
  }, [items]);

  const getTotalItems = useCallback(() => {
    return items.reduce((total, item) => total + item.quantity, 0);
  }, [items]);

  return {
    items,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    getTotalPrice,
    getTotalItems
  };
};
```

#### テスタビリティの向上

```typescript
// カスタムフックのテスト例
import { renderHook, act } from '@testing-library/react';
import { useShoppingCart } from './useShoppingCart';

describe('useShoppingCart', () => {
  const mockProduct: Product = {
    id: '1',
    name: 'テスト商品',
    price: 1000,
    image: '/test.jpg',
    category: 'test',
    rating: 4.5
  };

  it('商品をカートに追加できる', () => {
    const { result } = renderHook(() => useShoppingCart());

    act(() => {
      result.current.addItem(mockProduct, 2);
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].quantity).toBe(2);
    expect(result.current.getTotalPrice()).toBe(2000);
  });

  it('同じ商品を追加すると数量が増加する', () => {
    const { result } = renderHook(() => useShoppingCart());

    act(() => {
      result.current.addItem(mockProduct, 1);
      result.current.addItem(mockProduct, 2);
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].quantity).toBe(3);
  });
});
```

### TypeScriptによる型安全で高凝集な設計

#### Domain型による設計の明確化

```typescript
// ドメインに特化した型定義
export type ProductId = string & { __brand: 'ProductId' };
export type UserId = string & { __brand: 'UserId' };
export type OrderId = string & { __brand: 'OrderId' };

// 型ガードによる安全性の確保
export const isProductId = (value: string): value is ProductId => {
  return /^prod_[a-zA-Z0-9]+$/.test(value);
};

// 高凝集な型定義例
export interface Product {
  readonly id: ProductId;
  readonly name: string;
  readonly price: Money;
  readonly category: ProductCategory;
  readonly availability: ProductAvailability;
}

export interface Money {
  readonly amount: number;
  readonly currency: 'JPY' | 'USD' | 'EUR';
}

export interface ProductAvailability {
  readonly inStock: boolean;
  readonly quantity: number;
  readonly reservedQuantity: number;
}

// 機能的凝集度の高いサービスクラス
export class CartService {
  private constructor(private readonly items: CartItem[]) {}

  static empty(): CartService {
    return new CartService([]);
  }

  static fromItems(items: CartItem[]): CartService {
    return new CartService([...items]);
  }

  addProduct(product: Product, quantity: number): CartService {
    const existingItem = this.items.find(item => item.productId === product.id);
    
    if (existingItem) {
      const updatedItems = this.items.map(item =>
        item.productId === product.id
          ? { ...item, quantity: item.quantity + quantity }
          : item
      );
      return new CartService(updatedItems);
    }

    const newItem: CartItem = {
      productId: product.id,
      productName: product.name,
      unitPrice: product.price,
      quantity
    };

    return new CartService([...this.items, newItem]);
  }

  removeProduct(productId: ProductId): CartService {
    const updatedItems = this.items.filter(item => item.productId !== productId);
    return new CartService(updatedItems);
  }

  calculateTotal(): Money {
    const total = this.items.reduce((sum, item) => {
      return sum + (item.unitPrice.amount * item.quantity);
    }, 0);

    return {
      amount: total,
      currency: 'JPY' // 簡略化：実際は通貨統一ロジックが必要
    };
  }

  getItems(): readonly CartItem[] {
    return Object.freeze([...this.items]);
  }
}
```

### 共通コンポーネントの設計指針

#### 汎用性と専用性のバランス

```typescript
// 汎用的すぎる例（低凝集度）
interface GenericComponentProps {
  type: 'button' | 'input' | 'select' | 'textarea';
  data: any;
  config: any;
  handlers: { [key: string]: Function };
}

// 適切な抽象化レベル（高凝集度）
interface FormFieldProps {
  label: string;
  error?: string;
  required?: boolean;
  children: React.ReactNode;
}

const FormField: React.FC<FormFieldProps> = ({
  label,
  error,
  required = false,
  children
}) => {
  return (
    <div className="form-field">
      <label className="form-label">
        {label}
        {required && <span className="required">*</span>}
      </label>
      {children}
      {error && <span className="error-message">{error}</span>}
    </div>
  );
};

// 使用例
const ProductForm = () => {
  const [name, setName] = useState('');
  const [nameError, setNameError] = useState('');

  return (
    <form>
      <FormField label="商品名" error={nameError} required>
        <input
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
      </FormField>
    </form>
  );
};
```

#### 拡張性を考慮した実装

```typescript
// 拡張可能な設計パターン
interface BaseModalProps {
  isOpen: boolean;
  onClose: () => void;
  title?: string;
  children: React.ReactNode;
}

interface ConfirmModalProps extends BaseModalProps {
  type: 'confirm';
  onConfirm: () => void;
  onCancel: () => void;
  confirmText?: string;
  cancelText?: string;
}

interface InfoModalProps extends BaseModalProps {
  type: 'info';
  onAcknowledge: () => void;
  acknowledgeText?: string;
}

type ModalProps = ConfirmModalProps | InfoModalProps;

const Modal: React.FC<ModalProps> = (props) => {
  if (!props.isOpen) return null;

  return (
    <div className="modal-overlay" onClick={props.onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        {props.title && <h2 className="modal-title">{props.title}</h2>}
        <div className="modal-body">{props.children}</div>
        <div className="modal-actions">
          {props.type === 'confirm' && (
            <>
              <button onClick={props.onCancel}>
                {props.cancelText || 'キャンセル'}
              </button>
              <button onClick={props.onConfirm} className="primary">
                {props.confirmText || '確認'}
              </button>
            </>
          )}
          {props.type === 'info' && (
            <button onClick={props.onAcknowledge} className="primary">
              {props.acknowledgeText || 'OK'}
            </button>
          )}
        </div>
      </div>
    </div>
  );
};
```

## チーム開発での実践

### コードレビューでの観点

凝集度を意識したコードレビューを実施することで、チーム全体のコード品質を向上させることができます：

#### レビューチェックリスト

**コンポーネントレベル：**
- [ ] 1つのコンポーネントが複数の責任を持っていないか？
- [ ] propsが多すぎる（8個以上）場合、責任が分散していないか？
- [ ] 条件分岐が多い場合、複数のコンポーネントに分割できないか？
- [ ] useEffect内で関連性のない処理が混在していないか？

**カスタムフックレベル：**
- [ ] 1つのフックが複数のドメインロジックを扱っていないか？
- [ ] 戻り値が多すぎる場合、複数のフックに分割できないか？
- [ ] フック名がその責任を適切に表現しているか？

**ディレクトリレベル：**
- [ ] 新しいファイルが適切なfeatureディレクトリに配置されているか？
- [ ] 他のfeatureの内部実装に直接依存していないか？
- [ ] 共通化すべき機能がsharedディレクトリに移動されているか？

#### 実際のレビューコメント例

```typescript
// レビュー前のコード
const UserDashboard = () => {
  // ユーザー情報
  const [user, setUser] = useState(null);
  // 通知
  const [notifications, setNotifications] = useState([]);
  // 統計情報
  const [stats, setStats] = useState(null);
  
  useEffect(() => {
    // 複数の責任が混在
    fetchUser();
    fetchNotifications();
    fetchStats();
    initializeAnalytics();
  }, []);
  
  // 200行のコンポーネント...
};
```

**レビューコメント：**
> このコンポーネントは複数の責任を持っているようです。以下のような分割を検討してみてください：
> 
> 1. `UserProfile` - ユーザー情報表示
> 2. `NotificationCenter` - 通知管理
> 3. `UserStats` - 統計情報表示
> 4. `UserDashboard` - これらを統合する親コンポーネント
> 
> また、`useUser`, `useNotifications`, `useUserStats` といったカスタムフックで状態管理を分離することで、テストもしやすくなります。

### 段階的な導入戦略

#### フェーズ1：新機能での実践（1-2か月）

**目標：** 新しく開発する機能で凝集度の原則を適用

**実施内容：**
1. 新機能は必ずfeature-basedなディレクトリ構成を採用
2. コンポーネント設計時に単一責任の原則を意識
3. カスタムフックによる状態管理の分離

**成功指標：**
- 新機能のコンポーネントサイズが100行以下
- 1つのコンポーネントのpropsが7個以下
- feature間の依存関係が公開APIのみ

#### フェーズ2：既存コードのリファクタリング（2-3か月）

**目標：** 既存の低凝集度なコードを段階的に改善

**優先順位付け：**
1. **高頻度で変更される機能** - ユーザー認証、商品一覧など
2. **バグが発生しやすい機能** - 決済処理、データ同期など
3. **新機能開発で影響を受ける機能** - 共通コンポーネント、ユーティリティなど

**実施手順：**
```typescript
// Step 1: 既存のコンポーネントを分析
const ProductListPage = () => {
  // 300行のコンポーネント
  // 複数の責任：データ取得、フィルタリング、ソート、表示
};

// Step 2: カスタムフックの抽出
const useProductList = () => {
  // データ取得とフィルタリングロジック
};

// Step 3: 子コンポーネントの分離
const ProductFilter = () => { /* フィルタリングUI */ };
const ProductGrid = () => { /* 商品表示 */ };
const Pagination = () => { /* ページネーション */ };

// Step 4: 統合
const ProductListPage = () => {
  const { products, filters, ... } = useProductList();
  
  return (
    <div>
      <ProductFilter filters={filters} onFiltersChange={setFilters} />
      <ProductGrid products={products} />
      <Pagination ... />
    </div>
  );
};
```

#### フェーズ3：組織全体への展開（3-6か月）

**目標：** チーム全体で凝集度を意識した設計が標準となる

**実施内容：**
1. **設計ガイドラインの策定**
   ```markdown
   ## コンポーネント設計ガイドライン
   
   ### 単一責任の原則
   - 1つのコンポーネントは1つの明確な責任のみを持つ
   - 100行を超える場合は分割を検討
   - propsが7個を超える場合は責任の分散を疑う
   
   ### ディレクトリ構成
   - 新機能は必ずfeatures/配下に配置
   - 他featureの内部実装に直接依存しない
   - 共通機能はshared/に配置
   ```

2. **自動化ツールの導入**
   ```json
   // .eslintrc.js
   {
     "rules": {
       "max-lines": ["error", 100],
       "react/prop-types": ["error"],
       "import/no-restricted-paths": [
         "error",
         {
           "zones": [
             {
               "target": "./src/features/*/!(index.ts)",
               "from": "./src/features/*/services/*"
             }
           ]
         }
       ]
     }
   }
   ```

3. **定期的な振り返り**
   - 月次でのコード品質メトリクス確認
   - 四半期でのアーキテクチャレビュー
   - 年次での設計ガイドライン見直し

### リスクを最小化する移行計画

#### 段階的移行のメリット

**技術的リスク軽減：**
- 既存機能の動作に影響を与えない
- 小さな単位でのテスト・検証が可能
- 問題発生時の影響範囲が限定的

**チームの学習曲線：**
- 新しい設計原則を段階的に習得
- 実践を通じたノウハウの蓄積
- チームメンバー間での知識共有

#### 移行時の注意点

```typescript
// 悪い例：一度に大規模なリファクタリング
// features/product/components/ProductListPage.tsx
const ProductListPage = () => {
  // 既存の300行コードを一度にすべて書き換え
  // → リスクが高く、レビューも困難
};

// 良い例：段階的なリファクタリング
// Phase 1: カスタムフックの抽出（既存UIはそのまま）
const ProductListPage = () => {
  const { products, loading, filters, setFilters } = useProductList();
  
  // 既存のUIコードはそのまま
  return (
    <div>
      {/* 既存のフィルターUI */}
      {/* 既存の商品表示UI */}
    </div>
  );
};

// Phase 2: UIコンポーネントの分離（ロジックはそのまま）
const ProductListPage = () => {
  const productListHook = useProductList();
  
  return (
    <div>
      <ProductFilter {...productListHook} />
      <ProductGrid {...productListHook} />
    </div>
  );
};
```

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

### 今後の発展と継続的改善

#### マイクロフロントエンドとの親和性
Feature-basedなアーキテクチャは、将来的なマイクロフロントエンドへの移行を見据えた設計でもあります：

```typescript
// 将来的なマイクロフロントエンド化
// product-app (独立したアプリケーション)
export const ProductApp = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/products" element={<ProductListPage />} />
        <Route path="/products/:id" element={<ProductDetailPage />} />
      </Routes>
    </BrowserRouter>
  );
};

// main-app (他のマイクロフロントエンドを統合)
const MainApp = () => {
  return (
    <div>
      <Header />
      <MicroFrontend name="auth" host="http://auth.example.com" />
      <MicroFrontend name="product" host="http://product.example.com" />
      <MicroFrontend name="cart" host="http://cart.example.com" />
    </div>
  );
};
```

#### 継続的な改善サイクル

**月次レビュー：**
- コンポーネントの行数・複雑度の計測
- feature間の依存関係の可視化
- チームメンバーからのフィードバック収集

**四半期改善：**
- 設計ガイドラインのアップデート
- 新しい設計パターンの導入検討
- ツール・プロセスの改善

**年次評価：**
- アーキテクチャ全体の見直し
- 技術トレンドに応じたアップデート
- チームスキルの成長に合わせた高度化

### 実践への第一歩

1. **小さく始める**: 次の新機能開発から凝集度を意識した設計を試す
2. **チームで議論**: 設計レビューの際に凝集度の観点を含める
3. **継続的改善**: 定期的な振り返りで設計品質を向上させる

凝集度は一朝一夕で身につくものではありませんが、意識的に実践することで確実にコード品質が向上します。本記事の内容を参考に、保守性の高いフロントエンドアプリケーションの構築にチャレンジしてみてください。

## 参考文献

https://qiita.com/uesho/items/59845b4891e12dfb3def#:~:text=%E5%87%9D%E9%9B%86%E5%BA%A6%E3%81%AE%E7%A8%AE%E9%A1%9E,%E8%A1%A8%E3%81%99%E3%81%93%E3%81%A8%E3%81%8C%E5%87%BA%E6%9D%A5%E3%81%BE%E3%81%99%E3%80%82&text=1%E3%81%8C%E6%9C%80%E3%82%82%E5%87%9D%E9%9B%86%E5%BA%A6,%E3%81%AA%E3%82%8A%E3%80%81%E4%BD%8E%E5%87%9D%E9%9B%86%E3%81%A8%E3%81%AA%E3%82%8B%E3%80%82

https://speakerdeck.com/noritakaikeda/ji-neng-de-ning-ji-nogai-nian-woyong-ite-fu-shu-roru-lei-si-noji-neng-woduo-kuhan-musisutemuno-hurontoendonokonponentowoshi-qie-nifen-ge-suru
