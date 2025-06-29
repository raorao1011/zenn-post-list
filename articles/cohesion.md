---
title: "凝集度からアプローチするディレクトリ設計とコンポーネント設計"
emoji: "💪" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech"
topics: []
published: false
---

## はじめに

なぜ凝集度の観点からディレクトリ設計とコンポーネント設計を考えるのか、その重要性と本記事の目的について説明します。

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
    <Card>
      <Avatar src={user.avatar} />
      <Text>{user.name}</Text>
      <Text>{user.email}</Text>
    </Card>
  );
};

const NotificationCenter = ({ userId }) => {
  // 通知機能のみに集中
  const { notifications, markAsRead, clearAll } = useNotifications(userId);
  
  return (
    <VStack>
      {notifications.map(notification => (
        <NotificationItem 
          key={notification.id} 
          notification={notification} 
          onRead={markAsRead} 
        />
      ))}
    </VStack>
  );
};
```

## ディレクトリ設計における凝集度

### 従来のディレクトリ構成の問題点
- 技術的分類による構成（components/, hooks/, utils/）
- スケールしない構造の課題
- 機能追加時の複雑性

### 機能的凝集度に基づくディレクトリ設計
- Feature-basedなディレクトリ構成
- ドメイン駆動設計の適用
- 具体的な構成例とベストプラクティス

### 実践例：ECサイトのディレクトリ構成
```
src/
├── features/
│   ├── auth/
│   ├── product/
│   ├── cart/
│   └── order/
├── shared/
└── app/
```

## Reactコンポーネント設計における凝集度

### コンポーネントの責任分離
- 単一責任の原則
- プレゼンテーションとロジックの分離
- カスタムフックの活用

### 高凝集度なコンポーネントの特徴
- 明確な責任範囲
- 適切な抽象化レベル
- 再利用可能性と保守性

## 実践的な設計パターン

### Atomic Designと凝集度の関係
- Atoms, Molecules, Organisms, Templates, Pages
- 各レベルでの凝集度の考え方
- 適切な粒度の見極め方

### カスタムフックによる状態管理の凝集化
- useProductDetail, useCartなどの実装例
- ビジネスロジックの集約
- テスタビリティの向上

### 共通コンポーネントの設計指針
- 汎用性と専用性のバランス
- プロパティ設計の考え方
- 拡張性を考慮した実装

## まとめ

### 凝集度を意識した設計のメリット
- 保守性の向上
- 開発効率の改善
- バグの減少

## 参考文献

https://qiita.com/uesho/items/59845b4891e12dfb3def#:~:text=%E5%87%9D%E9%9B%86%E5%BA%A6%E3%81%AE%E7%A8%AE%E9%A1%9E,%E8%A1%A8%E3%81%99%E3%81%93%E3%81%A8%E3%81%8C%E5%87%BA%E6%9D%A5%E3%81%BE%E3%81%99%E3%80%82&text=1%E3%81%8C%E6%9C%80%E3%82%82%E5%87%9D%E9%9B%86%E5%BA%A6,%E3%81%AA%E3%82%8A%E3%80%81%E4%BD%8E%E5%87%9D%E9%9B%86%E3%81%A8%E3%81%AA%E3%82%8B%E3%80%82

https://speakerdeck.com/noritakaikeda/ji-neng-de-ning-ji-nogai-nian-woyong-ite-fu-shu-roru-lei-si-noji-neng-woduo-kuhan-musisutemuno-hurontoendonokonponentowoshi-qie-nifen-ge-suru
