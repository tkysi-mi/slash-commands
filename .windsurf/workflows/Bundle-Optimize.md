---
description: Webpackバンドルサイズを分析し、コード分割・遅延ロード・Tree Shakingで最適化するワークフロー
auto_execution_mode: 1
---

# /x-OptimizeBundle

## 目的

- バンドルサイズを分析し、ページロード時間を改善する。
- 不要な依存関係を特定し、削除または軽量な代替に置き換える。
- コード分割と遅延ロードを実装し、初回ロード時間を短縮する。
- Tree Shaking を活用し、未使用コードを削減する。
- Core Web Vitals（LCP, FID, CLS）を改善する。

## 前提

- Webpack, Vite, または Rollup でバンドルを生成している。
- package.json に依存関係が記載されている。
- 本番ビルドが可能である（`npm run build`）。
- Chrome DevTools や Lighthouse が利用可能である。

## 手順

### 1. 現在のバンドルサイズの測定

**質問1: ビルドツールの確認**
- 「使用しているビルドツールは何ですか？」
  - Webpack
  - Vite
  - Rollup
  - Next.js
  - Create React App (CRA)

**本番ビルドを実行**:
```bash
npm run build
```

**ビルド出力を確認**:
```
File sizes after gzip:

  123.45 KB  build/static/js/main.abc123.js
  23.45 KB   build/static/css/main.abc123.css
  5.67 KB    build/static/js/runtime-main.abc123.js
```

**質問2: バンドルサイズの確認**
- 「メインバンドルのサイズはいくつですか？」
- 目標サイズ：
  - **<100 KB**: 優秀
  - **100-200 KB**: 良好
  - **200-500 KB**: 改善の余地あり
  - **>500 KB**: 最適化が必要

### 2. バンドルアナライザーの実行

#### 2.1. バンドルアナライザーのインストール

**Webpack**:
```bash
npm install --save-dev webpack-bundle-analyzer
```

**Vite**:
```bash
npm install --save-dev rollup-plugin-visualizer
```

#### 2.2. 設定ファイルの編集

**Webpack** (`webpack.config.js`):
```javascript
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  // ...
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html',
    }),
  ],
};
```

**Vite** (`vite.config.ts`):
```typescript
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    visualizer({
      filename: './dist/bundle-report.html',
      open: true,
    }),
  ],
});
```

**Next.js** (`next.config.js`):
```javascript
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // Next.js config
});
```

#### 2.3. バンドルアナライザーの実行

**Webpack/CRA**:
```bash
npm run build
# bundle-report.html が生成される
```

**Next.js**:
```bash
ANALYZE=true npm run build
```

**ブラウザでレポートを開く**:
```bash
open dist/bundle-report.html
# または Windows: start dist/bundle-report.html
```

### 3. 問題のあるパッケージの特定

**質問3: 大きいパッケージの確認**
- 「バンドルアナライザーで最も大きいパッケージは何ですか？」

**よくある問題パッケージ**:
- **moment.js** (500KB+) → date-fns または day.js に置き換え
- **lodash** (全体インポート) → 個別関数のみインポート
- **core-js** (古いポリフィル) → 必要なもののみインポート
- **重複パッケージ** → バージョン統一

#### 3.1. パッケージサイズの確認

```bash
npx bundlephobia <package-name>
# または
npm install -g bundle-phobia-cli
bundle-phobia <package-name>
```

### 4. 依存関係の最適化

#### 4.1. 大きいパッケージの置き換え

**moment.js → date-fns**:
```bash
npm uninstall moment
npm install date-fns
```

```typescript
// Before
import moment from 'moment';
const formatted = moment().format('YYYY-MM-DD');

// After
import { format } from 'date-fns';
const formatted = format(new Date(), 'yyyy-MM-dd');
```

**lodash 全体 → 個別関数**:
```typescript
// Before (全体インポート)
import _ from 'lodash';
const result = _.debounce(fn, 300);

// After (個別インポート)
import debounce from 'lodash/debounce';
const result = debounce(fn, 300);

// または lodash-es を使用
import { debounce } from 'lodash-es';
```

#### 4.2. 未使用パッケージの削除

**未使用パッケージを検出**:
```bash
npx depcheck
```

**削除**:
```bash
npm uninstall <unused-package>
```

#### 4.3. 重複パッケージの統一

**重複パッケージを検出**:
```bash
npm ls <package-name>
```

**package.json で解決** (npm 8.3+):
```json
{
  "overrides": {
    "package-name": "^1.2.3"
  }
}
```

**yarn の場合**:
```json
{
  "resolutions": {
    "package-name": "^1.2.3"
  }
}
```

### 5. コード分割の実装

#### 5.1. ルートベースのコード分割

**React Router**:
```typescript
import { lazy, Suspense } from 'react';
import { BrowserRouter, Route, Routes } from 'react-router-dom';

// Lazy load components
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**Next.js**:
```typescript
import dynamic from 'next/dynamic';

const DynamicComponent = dynamic(() => import('@/components/HeavyComponent'), {
  loading: () => <p>Loading...</p>,
  ssr: false, // クライアントサイドのみでロード
});
```

#### 5.2. コンポーネントベースのコード分割

**重いコンポーネントを遅延ロード**:
```typescript
import { lazy, Suspense } from 'react';

// 重いチャートライブラリを遅延ロード
const Chart = lazy(() => import('./Chart'));

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<div>Loading chart...</div>}>
        <Chart data={data} />
      </Suspense>
    </div>
  );
}
```

#### 5.3. ベンダーチャンクの分離

**Webpack** (`webpack.config.js`):
```javascript
module.exports = {
  optimization: {
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
      },
    },
  },
};
```

**Vite** (自動で最適化):
```typescript
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },
  },
});
```

### 6. Tree Shaking の最適化

#### 6.1. ES Modules の使用

**package.json で "module" フィールドを確認**:
```json
{
  "main": "dist/index.js",
  "module": "dist/index.esm.js"
}
```

#### 6.2. サイドエフェクトの明示

**package.json**:
```json
{
  "sideEffects": false
}
```

**または特定ファイルのみ**:
```json
{
  "sideEffects": ["*.css", "*.scss"]
}
```

### 7. 画像・アセットの最適化

**質問4: 画像最適化**
- 「大きい画像やアセットがありますか？」

**画像最適化**:
- WebP 形式に変換
- 適切なサイズにリサイズ
- 遅延ロード（Lazy Loading）
- CDN の使用

**Next.js Image コンポーネント**:
```typescript
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Hero"
  width={800}
  height={600}
  loading="lazy"
  placeholder="blur"
/>
```

### 8. パフォーマンス測定

#### 8.1. Lighthouse の実行

```bash
npx lighthouse https://your-site.com --view
```

**確認する指標**:
- **LCP (Largest Contentful Paint)**: <2.5s
- **FID (First Input Delay)**: <100ms
- **CLS (Cumulative Layout Shift)**: <0.1
- **Total Blocking Time**: <200ms

#### 8.2. Bundle Phobia でパッケージサイズ確認

```bash
npx bundle-phobia <package-name>
```

### 9. 最適化結果の確認

**再度ビルドして比較**:
```bash
npm run build
```

**質問5: 最適化結果**
- 「バンドルサイズは減少しましたか？」
- 削減目標：
  - **10-20%**: 良好
  - **20-50%**: 優秀
  - **50%+**: 大幅改善

### 10. Git コミット

```bash
git add .
git commit -m "perf: optimize bundle size

- Replace moment.js with date-fns (saved 400KB)
- Implement route-based code splitting
- Enable tree shaking for lodash
- Remove unused dependencies
- Bundle size reduced from 500KB to 200KB (-60%)

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com)"
```

## 完了条件

- バンドルサイズが測定されている
- バンドルアナライザーで問題が特定されている
- 大きいパッケージが置き換えられている
- 未使用パッケージが削除されている
- コード分割が実装されている
- Tree Shaking が有効化されている
- 最適化後のビルドが成功している
- バンドルサイズが削減されている（目標10%以上）

## エスカレーション

- **バンドルサイズが削減できない**:
  - 「以下を再確認してください：」
    - 大きいパッケージの代替案
    - 未使用コードの削除
    - Dynamic Import の使用
    - 画像・アセットの最適化

- **コード分割でエラーが発生**:
  - 「Suspense の配置を確認してください。」
  - 「Loading fallback が適切に設定されているか確認してください。」

- **Tree Shaking が機能しない**:
  - 「ES Modules を使用しているか確認してください。」
  - 「package.json の "sideEffects" を設定してください。」

## ベストプラクティス

- **継続的な監視**: Lighthouse CI で定期的に測定
- **予算設定**: バンドルサイズの上限を設定（<200KB など）
- **プログレッシブエンハンスメント**: 必要な機能から順にロード
- **CDN 活用**: 静的アセットは CDN から配信
- **圧縮**: Gzip または Brotli 圧縮を有効化
- **キャッシュ戦略**: 長期キャッシュを活用
