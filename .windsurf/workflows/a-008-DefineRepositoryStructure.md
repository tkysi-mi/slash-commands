---
description: 技術スタックに基づいてプロジェクトのディレクトリ構造、アーキテクチャパターン、命名規則を定義するワークフロー
auto_execution_mode: 1
---

# /a-008-DefineRepositoryStructure

## 目的

- 決定された技術スタックに基づいて、最適なリポジトリ構造を定義する。
- アーキテクチャパターン（レイヤードアーキテクチャ、機能ベース、クリーンアーキテクチャなど）を選定する。
- ディレクトリの責務、命名規則、依存関係ルールを明確化する。
- チームメンバーのオンボーディングとコードの配置場所を明確にし、プロジェクトの保守性を向上させる。

## 前提

- `docs/project/04-design/01-tech-stack.md` が作成されていること（先に `/a-007-DefineTechStack` を実行）
- `docs/project/03-domain/01-domain-model.md` が作成されていること（推奨）
- `.windsurf/templates/project/04-design/02-repository-structure.md` テンプレートが最新状態であること

## 手順

### 1. ドキュメントと前提条件の確認

- 技術スタックドキュメントを読み込む：
  - `@docs/project/04-design/01-tech-stack.md`

- ドキュメントが存在しない場合：
  - 「技術スタックが定義されていません。先に `/a-007-DefineTechStack` を実行してください。今すぐ実行しますか？」
  - ユーザーが「はい」と回答した場合：`Call /a-007-DefineTechStack`

- ドメインモデルを読み込む（推奨）：
  - `@docs/project/03-domain/01-domain-model.md`

### 2. 技術スタックの分析

#### 2.1. プロジェクトタイプの特定

技術スタックドキュメントから以下を分析：

**フロントエンド**:
- フレームワーク（React、Vue、Angular、Svelteなど）
- メタフレームワーク（Next.js、Nuxt、Remixなど）
- アプリケーションタイプ（SPA、SSR、MPA）

**バックエンド**:
- 言語（Node.js、Python、Go、Javaなど）
- フレームワーク（Express、Fastify、NestJS、Django、Ginなど）
- APIスタイル（REST、GraphQL、gRPC、tRPC）

**モノレポ vs マルチレポ**:
- フロントエンドとバックエンドが同じリポジトリか、分離しているか
- 複数のアプリケーションが含まれるか

#### 2.2. 推奨アーキテクチャパターンの決定

分析結果に基づいて、以下のパターンから推奨を選定：

**A. レイヤードアーキテクチャ（Layered Architecture）**
- **適用場面**: エンタープライズアプリケーション、ドメインロジックが複雑
- **特徴**: presentation → business → data アクセス層
- **推奨技術**: NestJS、Spring Boot、Django

**B. クリーンアーキテクチャ（Clean Architecture）**
- **適用場面**: ドメイン駆動設計、テスタビリティ重視
- **特徴**: domain → application → infrastructure、依存性逆転
- **推奨技術**: NestJS、Go、Java

**C. 機能ベース（Feature-based / Vertical Slices）**
- **適用場面**: スケーラビリティ重視、マイクロサービス移行を見据える
- **特徴**: features/ 以下に機能ごとにモジュール化
- **推奨技術**: React、Next.js、Express、Fastify

**D. Atomic Design（フロントエンド）**
- **適用場面**: デザインシステム構築、UIコンポーネントの再利用性重視
- **特徴**: atoms → molecules → organisms → templates → pages
- **推奨技術**: React、Vue、Storybook

**E. Feature-Sliced Design（フロントエンド）**
- **適用場面**: 大規模フロントエンド、チームでのコード整理
- **特徴**: app/, pages/, features/, entities/, shared/
- **推奨技術**: React、Next.js、Vue

**F. モノレポアーキテクチャ**
- **適用場面**: 複数アプリケーション、コード共有が必要
- **特徴**: packages/ 配下に複数のアプリとライブラリ
- **推奨技術**: npm workspaces、yarn workspaces、pnpm、Turborepo、Nx

### 3. 推奨アーキテクチャの提示

ユーザーに推奨案を提示：

```markdown
## 推奨アーキテクチャパターン

技術スタック（[フロントエンド技術] + [バックエンド技術]）に基づき、以下のアーキテクチャパターンを推奨します：

### 推奨: [パターン名]

**理由**:
- [要件や技術スタックとの適合性]
- [チームサイズや成長性への対応]
- [保守性やテスタビリティの観点]

**ディレクトリ構造の概要**:
```
project-root/
├── src/
│   ├── [パターンに応じたディレクトリ]
│   └── ...
├── tests/
├── docs/
└── ...
```

**代替案**:
1. [代替パターン1]：[簡潔な説明]
2. [代替パターン2]：[簡潔な説明]
```

- ユーザーに確認：「この推奨案を採用しますか？それとも別のパターンを検討しますか？」

### 4. 詳細インタビューフェーズ

#### 4.1. アーキテクチャパターンの最終決定

- 「採用するアーキテクチャパターンを教えてください。」
  - 推奨案をそのまま採用
  - 推奨案をカスタマイズ
  - 別のパターンを選択

- 別のパターンを選択した場合：
  - 「その理由を教えてください。」
  - 「チームの経験や将来の拡張性に関する考慮事項はありますか？」

#### 4.2. ディレクトリ構造の詳細化

**質問1: ソースコードディレクトリ**
- 「ソースコードのルートディレクトリは何にしますか？」
  - 選択肢：`src/`、`app/`、ルート直下（`lib/`など）
  - 推奨：`src/`（標準的）

**質問2: テストディレクトリの配置**
- 「テストコードはどこに配置しますか？」
  - 選択肢：
    - `tests/` ディレクトリに集約
    - 各ファイルの隣にコロケーション（`UserProfile.test.tsx`）
    - ハイブリッド（ユニットテストはコロケーション、E2Eは `tests/` に集約）

**質問3: フロントエンドの構造**（該当する場合）

- 「コンポーネントの分類方法は？」
  - 選択肢：
    - `/components`（すべて）+ `/pages`
    - `/components`（共通） + `/features`（機能別）
    - Atomic Design（`/atoms`, `/molecules`, `/organisms`, `/templates`, `/pages`）
    - Feature-Sliced Design（`/app`, `/pages`, `/features`, `/entities`, `/shared`）

- 「状態管理のコードはどこに配置しますか？」
  - `/store` または `/state`（集約）
  - 各機能内にコロケーション（`/features/user/userStore.ts`）
  - グローバル context のみ `/contexts`

- 「スタイルファイルの配置は？」
  - コンポーネントと同じ場所（`Button.tsx` + `Button.module.css`）
  - `/styles` に集約（グローバルスタイル、テーマ、変数）
  - ハイブリッド（グローバルは `/styles`、コンポーネント固有はコロケーション）

**質問4: バックエンドの構造**（該当する場合）

- 「バックエンドのレイヤー構造は？」
  - 選択肢：
    - レイヤー別（`/controllers`, `/services`, `/repositories`）
    - 機能別（`/features/user`, `/features/order`）
    - ドメイン駆動（`/domain`, `/application`, `/infrastructure`）

- 「APIルートの定義はどこに配置しますか？」
  - `/routes` ディレクトリ（集約）
  - 各機能モジュール内（`/features/user/userRoutes.ts`）
  - フレームワーク標準（例：Next.js の `/app/api`、NestJS の `/controllers`）

- 「データベーススキーマ・マイグレーションはどこに配置しますか？」
  - `/database` または `/db`
  - `/prisma`、`/drizzle`（ORMに応じて）
  - `/migrations` + `/models`

**質問5: 共通・ユーティリティコード**

- 「共通ユーティリティ関数の配置は？」
  - `/lib` または `/utils`
  - `/shared`
  - `/common`

- 「型定義の配置は？」
  - `/types`（集約）
  - 各機能内にコロケーション（`/features/user/types.ts`）
  - ハイブリッド（共通型は `/types`、機能固有型はコロケーション）

**質問6: 設定ファイル・スクリプト**

- 「設定ファイル（環境変数、設定ファイル）の配置は？」
  - `/config`
  - ルート直下（`.env`、`config.json`）
  - `/src/config`

- 「スクリプト（ビルド、デプロイ、シード）の配置は？」
  - `/scripts`
  - `/tools`
  - ルート直下（`package.json` の scripts のみ）

**質問7: ドキュメント**

- 「プロジェクトドキュメントの配置は？」
  - `/docs`（推奨、既存構造）
  - `/documentation`
  - ルート直下（README.md のみ）

**質問8: モノレポ構成**（該当する場合）

- 「モノレポの場合、パッケージはどのように分割しますか？」
  - `/packages`（すべてのパッケージ）
  - `/apps`（アプリケーション） + `/packages`（ライブラリ）
  - カスタム構成

- 「各パッケージの命名規則は？」
  - スコープ付き（`@myproject/web`、`@myproject/api`）
  - プレフィックスなし（`web`、`api`）

#### 4.3. 命名規則の決定

**質問1: ファイル名のケース**
- 「ファイル名のケーススタイルは？」
  - React コンポーネント：PascalCase（`UserProfile.tsx`）
  - TypeScript/JavaScript ファイル：camelCase（`formatDate.ts`）
  - ディレクトリ：kebab-case（`user-management/`）または camelCase（`userManagement/`）

- 「テストファイルの命名は？」
  - `*.test.ts` / `*.test.tsx`
  - `*.spec.ts` / `*.spec.tsx`
  - `__tests__/` ディレクトリ

- 「スタイルファイルの命名は？」（CSS Modulesの場合）
  - `*.module.css`
  - `*.module.scss`

**質問2: コード要素の命名**
- 「変数・関数の命名規則は？」
  - camelCase（推奨）

- 「定数の命名規則は？」
  - UPPER_SNAKE_CASE（`API_BASE_URL`）
  - camelCase（`apiBaseUrl`）

- 「クラス・型の命名規則は？」
  - PascalCase（推奨）

- 「インターフェースに接頭辞 `I` を付けますか？」
  - 付けない（推奨：`interface User {}`）
  - 付ける（`interface IUser {}`）

#### 4.4. 依存関係ルールの定義

- 「モジュール間の依存関係ルールはありますか？」
  - 例：
    - ✅ 許可：`features/` → `components/`, `lib/`, `types/`
    - ❌ 禁止：`lib/` → `features/`（汎用ライブラリが機能に依存）
    - ❌ 禁止：`components/` → `features/`（汎用コンポーネントが機能に依存）

- 「依存関係を強制するツールを使用しますか？」
  - dependency-cruiser
  - ESLint の import/no-restricted-paths プラグイン
  - 使用しない

#### 4.5. コロケーション戦略の決定

- 「関連ファイル（コンポーネント、スタイル、テスト）の配置方針は？」
  - **機能別コロケーション**（推奨）：関連ファイルを同じディレクトリに
    ```
    features/user/
    ├── UserProfile.tsx
    ├── UserProfile.module.css
    ├── UserProfile.test.tsx
    └── userService.ts
    ```
  - **ファイルタイプ別**：`/components`, `/styles`, `/tests` を分離
  - **ハイブリッド**：機能内ではコロケーション、共通コンポーネントはタイプ別

### 5. ディレクトリ構造の生成

#### 5.1. 完全なディレクトリツリーの作成

収集した情報を基に、完全なディレクトリ構造を作成：

**フロントエンド（機能ベース + React + Next.js）の例**:
```
project-root/
├── src/
│   ├── app/                    # Next.js App Router（ルーティング）
│   │   ├── (auth)/            # ルートグループ
│   │   │   ├── login/
│   │   │   └── register/
│   │   ├── dashboard/
│   │   └── layout.tsx
│   ├── components/             # 共通UIコンポーネント
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.module.css
│   │   │   └── Button.test.tsx
│   │   └── Input/
│   ├── features/               # 機能モジュール
│   │   ├── user/
│   │   │   ├── UserProfile.tsx
│   │   │   ├── UserList.tsx
│   │   │   ├── userService.ts
│   │   │   ├── userStore.ts
│   │   │   └── types.ts
│   │   └── auth/
│   ├── lib/                    # ユーティリティ、APIクライアント
│   │   ├── api.ts
│   │   ├── formatters.ts
│   │   └── validation.ts
│   ├── hooks/                  # カスタムフック
│   │   ├── useAuth.ts
│   │   └── useFetch.ts
│   ├── types/                  # 共通型定義
│   │   ├── api.ts
│   │   └── models.ts
│   ├── styles/                 # グローバルスタイル
│   │   ├── globals.css
│   │   └── theme.ts
│   └── config/                 # 設定ファイル
│       └── constants.ts
├── public/                     # 静的ファイル
│   ├── images/
│   └── favicon.ico
├── docs/                       # プロジェクトドキュメント
│   ├── project/
│   └── tasks/
├── tests/                      # E2Eテスト（ユニットテストはコロケーション）
│   └── e2e/
├── scripts/                    # ビルド・デプロイスクリプト
│   └── seed.ts
├── .github/                    # GitHub Actions
│   └── workflows/
│       └── ci.yml
├── node_modules/               # 依存パッケージ（.gitignore）
├── .env.example                # 環境変数テンプレート
├── .gitignore
├── package.json
├── tsconfig.json
├── next.config.js
├── .eslintrc.json
├── .prettierrc
└── README.md
```

**バックエンド（レイヤードアーキテクチャ + Node.js + Express）の例**:
```
project-root/
├── src/
│   ├── controllers/            # プレゼンテーション層（リクエストハンドラ）
│   │   ├── userController.ts
│   │   └── authController.ts
│   ├── services/               # ビジネスロジック層
│   │   ├── userService.ts
│   │   └── authService.ts
│   ├── repositories/           # データアクセス層
│   │   ├── userRepository.ts
│   │   └── authRepository.ts
│   ├── models/                 # データモデル（ORM/ODM）
│   │   ├── User.ts
│   │   └── Session.ts
│   ├── routes/                 # APIルート定義
│   │   ├── userRoutes.ts
│   │   └── authRoutes.ts
│   ├── middlewares/            # ミドルウェア
│   │   ├── authMiddleware.ts
│   │   └── errorHandler.ts
│   ├── lib/                    # ユーティリティ
│   │   ├── logger.ts
│   │   └── validation.ts
│   ├── types/                  # 型定義
│   │   ├── express.d.ts
│   │   └── models.ts
│   ├── config/                 # 設定
│   │   ├── database.ts
│   │   └── env.ts
│   └── index.ts                # エントリーポイント
├── database/                   # データベース関連
│   ├── migrations/
│   └── seeds/
├── tests/                      # テスト
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── scripts/                    # スクリプト
│   └── migrate.ts
├── docs/                       # ドキュメント
├── .env.example
├── .gitignore
├── package.json
├── tsconfig.json
├── .eslintrc.json
└── README.md
```

**モノレポの例（フロントエンド + バックエンド）**:
```
project-root/
├── apps/
│   ├── web/                    # Next.js フロントエンド
│   │   ├── src/
│   │   ├── public/
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── api/                    # Express バックエンド
│       ├── src/
│       ├── package.json
│       └── tsconfig.json
├── packages/
│   ├── shared/                 # 共通コード（型定義、ユーティリティ）
│   │   ├── src/
│   │   └── package.json
│   └── ui/                     # 共通UIコンポーネント
│       ├── src/
│       └── package.json
├── docs/
├── .github/
├── package.json                # ルートの package.json（workspaces定義）
├── pnpm-workspace.yaml         # pnpm workspaces設定（または lerna.json、nx.json）
└── README.md
```

#### 5.2. ディレクトリツリーのカスタマイズ

- ユーザーに提示：「上記のディレクトリ構造で問題ありませんか？追加・削除・変更したいディレクトリはありますか？」

- フィードバックに基づき、ディレクトリ構造を調整

### 6. 主要ディレクトリの責務定義

各ディレクトリについて、以下を定義：

| ディレクトリ | 役割 | 配置すべきもの | 配置すべきでないもの |
|--------------|------|----------------|---------------------|
| `/src` | ソースコード全体 | ビジネスロジック、UI、ユーティリティ | ビルド成果物、テスト結果 |
| `/src/components` | 再利用可能なUIコンポーネント | 汎用Button、Input、Modalなど | ビジネスロジック、API呼び出し |
| `/src/features` | 機能単位のモジュール | UI、ロジック、API呼び出し、状態管理 | 他機能への直接依存（循環依存注意） |
| `/src/lib` | ユーティリティ関数 | 日付フォーマット、バリデーション、APIクライアント | ビジネスロジック、UI |
| `/tests` | テストコード | E2Eテスト、統合テスト | ユニットテスト（コロケーション推奨） |
| `/docs` | プロジェクトドキュメント | 要件、設計、タスク管理 | ソースコード、設定ファイル |
| `/scripts` | ビルド・デプロイスクリプト | データシード、マイグレーション、デプロイ自動化 | ビジネスロジック |

### 7. ドキュメント作成

- 収集した情報を基に、`docs/project/04-design/02-repository-structure.md` を作成

- テンプレートに従い、以下を記載：
  - ディレクトリ構造（treeコマンド形式）
  - アーキテクチャパターン（採用パターン、理由、依存方向）
  - 主要ディレクトリ（役割、例）
  - 命名規則（ファイル、コード要素）
  - モジュール境界と依存関係（依存ルール、強制ツール）
  - コロケーション戦略（採用戦略、配置例、理由）
  - 設定ファイル一覧（主要ファイルと役割）
  - メモ（補足情報、将来の計画）

- **HTMLコメントは削除せず残す**

### 8. レビューと確認

- 作成したドキュメントをユーザーに提示：
  - 「リポジトリ構造ドキュメントが完成しました。内容を確認してください。」
  - 「新規メンバーがこのドキュメントを読んで、コードの配置場所がわかりますか？」
  - 「不足している情報や変更したい点はありますか？」

- 統計情報を表示：
  - ルート直下のディレクトリ数
  - `/src` 配下の主要ディレクトリ数
  - 依存関係ルールの数

### 9. 完成とコミット準備

- `docs/project/04-design/02-repository-structure.md` が保存されたことを確認

- 次のステップを提案：
  - 「リポジトリ構造が決定しました。次は画面設計を作成しますか？（`/a-009-DefineScreenDesign`）」

## 完了条件

- `docs/project/04-design/02-repository-structure.md` が作成されている
- 以下のセクションがすべて記載されている：
  - ディレクトリ構造（treeコマンド形式）
  - アーキテクチャパターン（採用パターン、理由、依存方向）
  - 主要ディレクトリ（各ディレクトリの役割と例）
  - 命名規則（ファイル、ディレクトリ、コード要素）
  - モジュール境界と依存関係（依存ルール）
  - コロケーション戦略（採用戦略、理由）
- ディレクトリ構造が技術スタックと整合している
- アーキテクチャパターンが明確に定義されている
- 命名規則が具体例とともに記載されている
- 依存関係ルールが明確に記載されている（許可・禁止・注意）
- ユーザーが内容を確認し、承認している

## エスカレーション

- ユーザーがアーキテクチャパターンを理解していない場合：
  - 「アーキテクチャパターンについて、具体例を提示しながら説明します。[パターン名]は次のような構造です：[図解]」
  - 「チームの経験や将来の拡張性を考慮して、[推奨パターン]をお勧めします。」

- ディレクトリ構造が複雑すぎる場合：
  - 「この構造は初期段階には複雑すぎます。まずはシンプルな構成から始め、必要に応じて拡張することを推奨します。」
  - シンプルな代替案を提示

- 技術スタックとディレクトリ構造が不整合な場合：
  - 「技術スタック（[技術名]）と選択したディレクトリ構造が整合していません。[理由]のため、[代替構造]を推奨します。」

- 依存関係ルールが現実的でない場合：
  - 「このルールは厳格すぎて、実装が困難になる可能性があります。より柔軟なルールを検討しませんか？」

- モノレポとマルチレポで迷っている場合：
  - 「コード共有の必要性、チームの分割、デプロイの独立性を考慮して、以下の観点で比較します：」
  - モノレポとマルチレポのメリット・デメリット比較表を提示

- 命名規則が統一されていない場合：
  - 「プロジェクト全体で命名規則を統一することが重要です。チームで合意の上、一貫したスタイルを選択してください。」

- アーキテクチャパターンが将来のスケーラビリティに不適切な場合：
  - 「現在の選択は初期には適していますが、将来的にスケールする際に制約になる可能性があります。[理由]。移行計画を検討しませんか？」
