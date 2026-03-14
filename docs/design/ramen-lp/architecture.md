# ラーメンLP アーキテクチャ設計

**作成日**: 2026-03-14
**関連要件定義**: [requirements.md](../../spec/ramen-lp/requirements.md)
**ヒアリング記録**: [design-interview.md](design-interview.md)

**【信頼性レベル凡例】**:
- 🔵 **青信号**: EARS要件定義書・ユーザヒアリングを参考にした確実な設計
- 🟡 **黄信号**: EARS要件定義書・ユーザヒアリングから妥当な推測による設計
- 🔴 **赤信号**: EARS要件定義書・ユーザヒアリングにない推測による設計

---

## システム概要 🔵

**信頼性**: 🔵 *要件定義REQ-001・ユーザヒアリングより*

「上州豚骨 中島屋」の濃厚豚骨ラーメン専門店LP。Astro 6による静的サイト生成（SSG）で、1ページ構成の和モダンデザインLP。カラーパレット3パターンを別ページとして作成し、クライアントが比較選択できる構成とする。

## アーキテクチャパターン 🔵

**信頼性**: 🔵 *要件定義REQ-003・REQ-401・REQ-402・ユーザヒアリングより*

- **パターン**: コンポーネントベース静的サイト生成（Astro Islands Architecture）
- **選択理由**:
  - 静的LPにはSSGが最適（サーバーサイド不要、高速表示）
  - Astroのコンポーネント分割でセクションごとに管理
  - Tailwind CSS 4でユーティリティファーストのスタイリング
  - JavaScriptは最小限（ハンバーガーメニュー・スクロールイベント・アニメーション）

## コンポーネント構成 🔵

**信頼性**: 🔵 *要件定義REQ-004・note.mdセクション構成より*

### ページ・レイアウト

| ファイル | 役割 | 関連要件 |
|---------|------|---------|
| `src/pages/index.astro` | メインLP（おまかせカラー版） | REQ-001 |
| `src/pages/variant-dark.astro` | 濃色系カラーバリエーション | REQ-002 |
| `src/pages/variant-light.astro` | 淡色系カラーバリエーション | REQ-002 |
| `src/layouts/Layout.astro` | 共通HTMLレイアウト | REQ-405 |

### コンポーネント

| ファイル | 役割 | 関連要件 |
|---------|------|---------|
| `src/components/Header.astro` | 固定ヘッダー + ナビゲーション | REQ-010, REQ-011, REQ-012 |
| `src/components/Hero.astro` | フルスクリーン背景 + キャッチコピー | REQ-020, REQ-021, REQ-022, REQ-023 |
| `src/components/About.astro` | こだわりセクション（スープ・食材） | REQ-030, REQ-031, REQ-032 |
| `src/components/ShopInfo.astro` | 店舗情報 + 地図エリア | REQ-040, REQ-041 |
| `src/components/Footer.astro` | フッター | REQ-050 |

### スタイル

| ファイル | 役割 | 信頼性 |
|---------|------|--------|
| `src/styles/global.css` | Tailwind CSS設定・カスタムテーマ・アニメーション | 🔵 |

## カラーパレット設計 🔵

**信頼性**: 🔵 *ユーザヒアリング（3パターン比較）より*

3パターンのカラーバリエーションを作成し、それぞれ別ページで確認可能にする。
カラーはTailwind CSS 4の `@theme` ディレクティブでCSS変数として定義し、各ページで異なるテーマを適用する。

### パターン A: 濃色系（`/variant-dark`）🔵

| 用途 | カラー名 | 値 | 説明 |
|------|---------|-----|------|
| 背景（基調） | `--color-base` | `#1a1612` | 漆黒に近い焦げ茶 |
| 背景（セクション交互） | `--color-base-alt` | `#231e19` | やや明るい焦げ茶 |
| アクセント（主） | `--color-accent` | `#c5a572` | 金色（和金） |
| アクセント（副） | `--color-accent-sub` | `#8b2500` | 朱赤 |
| テキスト（主） | `--color-text` | `#f5f0e8` | 生成り白 |
| テキスト（副） | `--color-text-muted` | `#a89b8c` | 灰茶 |
| ボーダー | `--color-border` | `#3a332c` | 暗い茶 |

### パターン B: 淡色系（`/variant-light`）🔵

| 用途 | カラー名 | 値 | 説明 |
|------|---------|-----|------|
| 背景（基調） | `--color-base` | `#f7f3ed` | 生成り |
| 背景（セクション交互） | `--color-base-alt` | `#ece5d9` | 薄い亜麻色 |
| アクセント（主） | `--color-accent` | `#6b4c3b` | 焦げ茶 |
| アクセント（副） | `--color-accent-sub` | `#a03020` | 朱色 |
| テキスト（主） | `--color-text` | `#2c2420` | 墨色 |
| テキスト（副） | `--color-text-muted` | `#7a6e63` | 灰茶 |
| ボーダー | `--color-border` | `#d4cbc0` | 淡い灰茶 |

### パターン C: おまかせ（`/` メイン）🟡

**信頼性**: 🟡 *「濃厚豚骨ラーメン」のイメージから妥当な推測*

| 用途 | カラー名 | 値 | 説明 |
|------|---------|-----|------|
| 背景（基調） | `--color-base` | `#f5efe6` | 豚骨スープのような温かみのあるクリーム |
| 背景（セクション交互） | `--color-base-alt` | `#2c2420` | 墨色（コントラスト用） |
| アクセント（主） | `--color-accent` | `#b8860b` | 濃い金色（ダークゴールデンロッド） |
| アクセント（副） | `--color-accent-sub` | `#8b2500` | 朱赤（ラー油の赤） |
| テキスト（主） | `--color-text` | `#2c2420` | 墨色 |
| テキスト（主・反転） | `--color-text-invert` | `#f5efe6` | クリーム白 |
| テキスト（副） | `--color-text-muted` | `#7a6e63` | 灰茶 |
| ボーダー | `--color-border` | `#d4c8b8` | 温かい灰 |

### テーマ切替の実装方式 🔵

**信頼性**: 🔵 *ユーザヒアリング（3ページ方式）より*

```
src/styles/
├── global.css          # 共通スタイル・アニメーション定義
├── theme-dark.css      # パターンA: 濃色系テーマ
├── theme-light.css     # パターンB: 淡色系テーマ
└── theme-default.css   # パターンC: おまかせテーマ
```

各テーマCSSで `@theme` ディレクティブによりCSS変数を定義。
コンポーネント側はCSS変数を参照するため、テーマ切替に対応。

## フォント設計 🔵

**信頼性**: 🔵 *ユーザヒアリング（Noto Serif JP選択）より*

| 用途 | フォント | ウェイト |
|------|---------|---------|
| 見出し（h1, h2） | Noto Serif JP | 700 (Bold), 900 (Black) |
| 本文・ナビ | Noto Serif JP | 400 (Regular), 700 (Bold) |

```css
@import url('https://fonts.googleapis.com/css2?family=Noto+Serif+JP:wght@400;700;900&display=swap');

@theme {
  --font-serif: 'Noto Serif JP', serif;
}
```

## アニメーション設計 🔵

**信頼性**: 🔵 *ユーザヒアリング（リッチなアニメーション）より*

### アニメーション一覧

| アニメーション名 | 対象 | 効果 | トリガー |
|----------------|------|------|---------|
| `fade-in-up` | セクション内テキスト・要素 | 下からフェードイン | スクロールで表示領域に入った時 |
| `slide-in-left` | こだわりセクションのカード | 左からスライドイン | スクロールで表示領域に入った時 |
| `slide-in-right` | こだわりセクションのカード | 右からスライドイン | スクロールで表示領域に入った時 |
| `parallax` | Hero背景画像 | パララックススクロール | スクロール連動 |
| `scale-in` | 画像・装飾要素 | 拡大フェードイン | スクロールで表示領域に入った時 |
| `text-reveal` | キャッチコピー | 文字が1文字ずつ現れる | ページロード時 |
| `line-draw` | 装飾線・セパレータ | 線が描画される | スクロールで表示領域に入った時 |
| `counter-up` | 数値（任意） | 数値がカウントアップ | スクロールで表示領域に入った時 |

### スクロールアニメーション実装方式 🟡

**信頼性**: 🟡 *リッチアニメーション要件からの妥当な推測*

Intersection Observer APIを使用し、要素がビューポートに入ったタイミングでアニメーションクラスを付与する。

```javascript
// src/scripts/scroll-animation.js（client-sideスクリプト）
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('is-visible');
    }
  });
}, { threshold: 0.1 });

document.querySelectorAll('[data-animate]').forEach(el => observer.observe(el));
```

### パララックス実装方式 🟡

**信頼性**: 🟡 *リッチアニメーション要件からの妥当な推測*

CSSの `background-attachment: fixed` またはJavaScriptによる `transform: translateY()` でHero背景のパララックス効果を実現。

## ディレクトリ構造 🔵

**信頼性**: 🔵 *note.md・要件定義より*

```
src/
├── components/
│   ├── Header.astro        # 固定ヘッダー + ナビ
│   ├── Hero.astro           # ヒーローセクション
│   ├── About.astro          # こだわりセクション
│   ├── ShopInfo.astro       # 店舗情報セクション
│   └── Footer.astro         # フッター
├── layouts/
│   └── Layout.astro         # 共通HTMLレイアウト（テーマCSS受け取り）
├── pages/
│   ├── index.astro          # メインLP（おまかせテーマ）
│   ├── variant-dark.astro   # 濃色系バリエーション
│   └── variant-light.astro  # 淡色系バリエーション
└── styles/
    ├── global.css           # 共通スタイル・アニメーション
    ├── theme-default.css    # おまかせテーマ
    ├── theme-dark.css       # 濃色系テーマ
    └── theme-light.css      # 淡色系テーマ
public/
├── hero.webp                # Hero背景画像
└── favicon.svg              # ファビコン
```

## 非機能要件の実現方法

### パフォーマンス 🟡

**信頼性**: 🟡 *NFR-001, NFR-002から妥当な推測*

- **LCP最適化**: Hero画像にpreload指定、WebP形式で圧縮済み
- **CLS最適化**: 画像にwidth/height指定またはaspect-ratio設定
- **FID最適化**: JavaScriptは最小限（スクロールアニメーション・ハンバーガーメニューのみ）
- **画像最適化**: WebP形式、適切なサイズ・圧縮率

### レスポンシブデザイン 🟡

**信頼性**: 🟡 *NFR-101から妥当な推測*

| ブレークポイント | 幅 | レイアウト |
|----------------|-----|----------|
| モバイル | ~767px | 1カラム、ハンバーガーメニュー |
| タブレット | 768px~1023px | 1-2カラム、インラインナビ |
| デスクトップ | 1024px~ | max-width: 1280px センタリング |

### アクセシビリティ 🟡

**信頼性**: 🟡 *NFR-201~203から妥当な推測*

- セマンティックHTML（header, nav, main, section, footer）
- 画像にalt属性設定
- ハンバーガーメニューにaria-label, aria-expanded
- フォーカス可能な要素にフォーカスリング
- カラーコントラスト比 WCAG AA準拠

### SEO 🟡

**信頼性**: 🟡 *NFR-301から妥当な推測*

- titleタグ: 「上州豚骨 中島屋 | 濃厚豚骨ラーメン専門店」
- meta description設定
- lang="ja" 設定
- OGP設定（将来対応可）

## 技術的制約 🔵

**信頼性**: 🔵 *要件定義REQ-401~405より*

- Astro 6フレームワーク必須
- Tailwind CSS 4必須
- Node.js >= 22.12.0
- pnpmパッケージ管理
- 日本語（lang="ja"）

## 関連文書

- **データフロー**: [dataflow.md](dataflow.md)
- **要件定義**: [requirements.md](../../spec/ramen-lp/requirements.md)
- **ユーザストーリー**: [user-stories.md](../../spec/ramen-lp/user-stories.md)
- **受け入れ基準**: [acceptance-criteria.md](../../spec/ramen-lp/acceptance-criteria.md)

## 信頼性レベルサマリー

- 🔵 青信号: 14件 (64%)
- 🟡 黄信号: 7件 (32%)
- 🔴 赤信号: 1件 (4%)

**品質評価**: 高品質
