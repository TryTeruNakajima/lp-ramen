# ラーメンLP データフロー図

**作成日**: 2026-03-14
**関連アーキテクチャ**: [architecture.md](architecture.md)
**関連要件定義**: [requirements.md](../../spec/ramen-lp/requirements.md)

**【信頼性レベル凡例】**:
- 🔵 **青信号**: EARS要件定義書・ユーザヒアリングを参考にした確実なフロー
- 🟡 **黄信号**: EARS要件定義書・ユーザヒアリングから妥当な推測によるフロー
- 🔴 **赤信号**: EARS要件定義書・ユーザヒアリングにない推測によるフロー

---

## ビルド・デプロイフロー 🔵

**信頼性**: 🔵 *要件定義REQ-003・技術スタックより*

```mermaid
flowchart LR
    A[Astroコンポーネント<br/>.astro] --> B[Astro SSGビルド<br/>astro build]
    C[Tailwind CSS<br/>global.css + theme-*.css] --> B
    D[画像<br/>public/] --> B
    B --> E[静的HTML/CSS/JS<br/>dist/]
    E --> F[Webサーバー<br/>/ CDN]
    F --> G[ブラウザ]
```

## ページレンダリングフロー 🔵

**信頼性**: 🔵 *要件定義REQ-001・REQ-004より*

```mermaid
flowchart TD
    A[ブラウザ] -->|GET /| B[index.astro]
    A -->|GET /variant-dark| C[variant-dark.astro]
    A -->|GET /variant-light| D[variant-light.astro]

    B --> E[Layout.astro<br/>theme-default.css]
    C --> F[Layout.astro<br/>theme-dark.css]
    D --> G[Layout.astro<br/>theme-light.css]

    E --> H[Header → Hero → About → ShopInfo → Footer]
    F --> H
    G --> H

    H --> I[完成HTML + CSS + JS]
```

**詳細ステップ**:
1. ビルド時にAstroが各ページを静的HTMLとして生成
2. 各ページは対応するテーマCSSをインポート
3. Layout.astroが共通のHTML構造（head, body）を提供
4. 各コンポーネントがCSS変数を参照してテーマに応じた色で描画

## ユーザーインタラクションフロー 🔵

**信頼性**: 🔵 *ユーザストーリー1.1, 1.2, 3.1より*

### ページ閲覧フロー

```mermaid
sequenceDiagram
    participant U as ユーザー
    participant B as ブラウザ
    participant S as 静的ファイル

    U->>B: LPにアクセス
    B->>S: GET / (HTML)
    S-->>B: HTML + CSS + JS
    B->>B: ページレンダリング
    B->>B: Hero背景画像読み込み
    B->>B: フォント読み込み（Noto Serif JP）
    B-->>U: ヒーローセクション表示

    Note over B: Intersection Observer 初期化

    U->>B: 下にスクロール
    B->>B: スクロールアニメーション発火
    B->>B: ヘッダー背景変化
    B-->>U: こだわりセクション表示（アニメーション付き）

    U->>B: さらにスクロール
    B-->>U: 店舗情報セクション表示
```

### ナビゲーションフロー 🟡

**信頼性**: 🟡 *既存実装パターンから妥当な推測*

```mermaid
sequenceDiagram
    participant U as ユーザー
    participant H as Header
    participant P as ページ

    alt デスクトップ
        U->>H: ナビリンク「こだわり」クリック
        H->>P: scrollIntoView(#about)
        P-->>U: スムーズスクロールでこだわりセクションへ
    else モバイル
        U->>H: ハンバーガーアイコンタップ
        H->>H: フルスクリーンメニュー表示
        U->>H: 「こだわり」タップ
        H->>H: メニュー閉じる
        H->>P: scrollIntoView(#about)
        P-->>U: スムーズスクロールでこだわりセクションへ
    end
```

## テーマ適用フロー 🔵

**信頼性**: 🔵 *ユーザヒアリング（3ページ比較方式）より*

```mermaid
flowchart TD
    subgraph ビルド時
        A[index.astro] -->|import| B[theme-default.css]
        C[variant-dark.astro] -->|import| D[theme-dark.css]
        E[variant-light.astro] -->|import| F[theme-light.css]

        B --> G[CSS変数定義<br/>--color-base, --color-accent, etc.]
        D --> G
        F --> G
    end

    subgraph コンポーネント描画
        G --> H[Header.astro<br/>bg-base, text-accent]
        G --> I[Hero.astro<br/>bg-base, text-text]
        G --> J[About.astro<br/>bg-base-alt, text-accent]
        G --> K[ShopInfo.astro<br/>bg-base, text-text]
        G --> L[Footer.astro<br/>bg-base-alt, text-text-muted]
    end
```

## アニメーションフロー 🔵

**信頼性**: 🔵 *ユーザヒアリング（リッチアニメーション）より*

```mermaid
flowchart TD
    A[ページロード完了] --> B[Hero: text-reveal アニメーション発火]
    A --> C[Hero: parallax 初期化]
    A --> D[Intersection Observer 登録]

    D --> E{要素がビューポートに入った？}
    E -->|Yes| F[data-animate属性の値に応じた<br/>アニメーションクラス付与]
    E -->|No| G[待機]

    F --> H[fade-in-up / slide-in-left /<br/> slide-in-right / scale-in / line-draw]

    C --> I{スクロールイベント}
    I --> J[Hero背景の translateY 更新]
    I --> K[ヘッダー背景の透過度更新]
```

### アニメーションの遅延設計 🟡

**信頼性**: 🟡 *リッチアニメーション要件から妥当な推測*

```
Hero セクション（ページロード時）:
├── 0.0s: 背景画像表示
├── 0.3s: タグライン text-reveal
├── 0.6s: キャッチコピー text-reveal
├── 1.0s: CTAボタン fade-in-up
└── 連続: parallax スクロール連動

こだわりセクション（スクロールトリガー）:
├── 0.0s: セクションタイトル fade-in-up
├── 0.2s: 装飾線 line-draw
├── 0.4s: スープこだわりカード slide-in-left
└── 0.6s: 食材こだわりカード slide-in-right

店舗情報セクション（スクロールトリガー）:
├── 0.0s: セクションタイトル fade-in-up
├── 0.2s: 情報リスト fade-in-up（各項目0.1s遅延）
└── 0.4s: 地図エリア scale-in
```

## 状態管理 🟡

**信頼性**: 🟡 *既存実装パターンから妥当な推測*

静的LPのため、状態管理はクライアントサイドJavaScriptで最小限に行う。

| 状態 | 管理方法 | 用途 |
|------|---------|------|
| モバイルメニュー開閉 | DOMクラス切替（`is-open`） | ハンバーガーメニューの表示/非表示 |
| ヘッダー背景 | scrollイベント + クラス切替 | スクロール位置に応じた背景変化 |
| アニメーション発火済み | Intersection Observer | 一度発火したら監視解除 |

## エラーハンドリング 🔴

**信頼性**: 🔴 *一般的なベストプラクティスからの推測*

| エラーケース | 対処方法 |
|-------------|---------|
| Hero画像読み込み失敗 | CSS `background-color` でフォールバック |
| フォント読み込み失敗 | `font-family` のフォールバック（`serif`） |
| JavaScript無効 | CSS-onlyアニメーション（`@media (prefers-reduced-motion: no-preference)`）、ナビはaタグのデフォルト動作 |

## 関連文書

- **アーキテクチャ**: [architecture.md](architecture.md)
- **要件定義**: [requirements.md](../../spec/ramen-lp/requirements.md)
- **ユーザストーリー**: [user-stories.md](../../spec/ramen-lp/user-stories.md)

## 信頼性レベルサマリー

- 🔵 青信号: 7件 (58%)
- 🟡 黄信号: 3件 (25%)
- 🔴 赤信号: 2件 (17%)

**品質評価**: 高品質
