---
description: 要件を分析して推奨技術スタックを提案し、ユーザーとの詳細なインタビューを通じて最適な技術選定を行うワークフロー
auto_execution_mode: 1
---

# /a-007-DefineTechStack

## 目的

- 既存の要件ドキュメント（システム概要、非機能要件、ドメインモデル）を分析し、適合する技術スタックを推奨する。
- 推奨案を提示した上で、ユーザーと詳細なインタビューを行い、すべての技術選定を明確化する。
- 技術選定の理由、バージョン、選定タイミング（初期/中期/後期/随時）を明確に記録する。
- チームのスキルセット、コスト、長期的な保守性を考慮した現実的な技術スタックを定義する。

## 前提

- `docs/project/01-requirements/` 配下のドキュメントが作成されていること（特に`01-system-overview.md`、`04-non-functional-requirements.md`）
- `docs/project/03-domain/01-domain-model.md` が作成されていること（推奨）
- `docs/project/04-design/` ディレクトリが存在すること
- `.windsurf/templates/project/04-design/01-tech-stack.md` テンプレートが最新状態であること

## 手順

### 1. ドキュメントと前提条件の確認

- `docs/project/04-design/` ディレクトリの存在を確認
- 存在しない場合：`Call /a-001-SetupDocStructure`

- 要件ドキュメントを読み込む：
  - `@docs/project/01-requirements/01-system-overview.md`
  - `@docs/project/01-requirements/03-features-planned.md`
  - `@docs/project/01-requirements/04-non-functional-requirements.md`
  - `@docs/project/03-domain/01-domain-model.md` （存在する場合）

- ドキュメントが不足している場合：
  - 「要件定義が不完全です。最低限、システム概要と非機能要件が必要です。」
  - 利用可能なドキュメントの範囲で推奨を行うことをユーザーに確認

### 2. 要件分析と推奨技術スタックの生成

#### 2.1. システム特性の分析

以下の観点で要件を分析：

**アプリケーションタイプ**:
- Webアプリケーション（SPA、MPA、SSR）
- モバイルアプリ（ネイティブ、ハイブリッド）
- APIサーバー
- バッチ処理
- リアルタイム通信

**非機能要件**:
- パフォーマンス目標（応答時間、スループット）
- スケーラビリティ（同時ユーザー数、成長予測）
- 可用性（SLA、ダウンタイム許容度）
- セキュリティ（認証方式、コンプライアンス）
- データ量（レコード数、ストレージ）

**ドメイン複雑度**:
- Bounded Contextの数
- 外部システム連携の有無
- データ整合性の要件

#### 2.2. 推奨技術スタックの作成

分析結果に基づき、以下のレイヤーごとに推奨を作成：

1. **フロントエンド**
2. **バックエンド**
3. **データベース**
4. **キャッシュ**（必要に応じて）
5. **インフラ**
6. **CI/CD**
7. **監視**
8. **テスト**
9. **セキュリティ**

各推奨には以下を含める：
- 技術名
- 推奨バージョン
- 選定理由（要件とのマッピング）
- 代替案（2-3個）
- 選定タイミング（初期/中期/後期）

#### 2.3. 推奨案の提示

ユーザーに推奨を提示：

```markdown
## 推奨技術スタック

要件分析の結果、以下の技術スタックを推奨します：

### フロントエンド
**推奨**: Next.js 14.x + React 18.x + TypeScript
**理由**:
- SSR/SSGによるパフォーマンス最適化（初回ロード3秒以内の要件に対応）
- TypeScriptによる型安全性（開発効率・保守性向上）
- 豊富なエコシステム（UI libraries, 状態管理）

**代替案**:
1. Remix + React（SSRに特化、データローディング最適化）
2. SvelteKit（軽量、学習曲線が緩やか）
3. Vue + Nuxt（学習容易性、小規模チーム向け）

### バックエンド
**推奨**: Node.js 20.x LTS + Fastify
**理由**:
- フロントエンドと言語統一（TypeScript共通）
- 高パフォーマンス（Fastifyは Express より高速）
- 非同期I/O（高並行性の要件に対応）

**代替案**:
1. Express（シンプル、実績豊富）
2. NestJS（エンタープライズ向け、TypeScript標準）
3. Go + Gin/Echo（高パフォーマンス、低メモリ消費）

### データベース
**推奨**: PostgreSQL 15.x
**理由**:
- ACID準拠（データ整合性の要件）
- JSONサポート（柔軟なスキーマ対応）
- 拡張性（PostGIS、Full-text search）
- オープンソース（コスト効率）

**代替案**:
1. MySQL（シンプル、学習容易性）
2. MongoDB（スキーマレス、水平スケール容易）
3. CockroachDB（分散SQL、高可用性）

[...]
```

- ユーザーに確認：「この推奨案について、どう思われますか？そのまま採用しますか？それとも変更したい部分がありますか？」

### 3. 詳細インタビューフェーズ

ユーザーのフィードバックを受けて、各レイヤーについて詳細にインタビューを行います。

#### 3.1. フロントエンド技術の選定

**質問1: アプリケーションタイプ**
- 「フロントエンドのアプリケーションタイプは何ですか？」
  - SPA（シングルページアプリケーション）
  - MPA（マルチページアプリケーション）
  - SSR/SSG（サーバーサイドレンダリング/静的サイト生成）
  - ハイブリッド（上記の組み合わせ）

**質問2: UIフレームワーク**
- 推奨案を提示した上で：「React、Vue、Angular、Svelteのいずれを使用しますか？」
- ユーザーが推奨と異なる選択をした場合：「その理由を教えてください。」

**質問3: メタフレームワーク**（ReactまたはVueの場合）
- 「Next.js / Remix（React）、または Nuxt（Vue）などのメタフレームワークを使用しますか？」
- 使用しない場合：「Create React App / Vite などのどちらを使いますか？」

**質問4: TypeScript**
- 「TypeScriptを使用しますか？それとも JavaScript のみですか？」
- JavaScriptのみの場合：「型安全性をどのように確保しますか？」

**質問5: 状態管理**
- 「状態管理ライブラリは何を使いますか？」
  - 選択肢：Redux、Zustand、Jotai、Recoil、Context API のみ、不要

**質問6: スタイリング**
- 「スタイリング方法は何を使いますか？」
  - 選択肢：Tailwind CSS、CSS Modules、Styled Components、Emotion、Sass、Plain CSS

**質問7: UIコンポーネントライブラリ**
- 「UIコンポーネントライブラリを使用しますか？」
  - 選択肢：shadcn/ui、MUI、Ant Design、Chakra UI、Mantine、使用しない（独自実装）

#### 3.2. バックエンド技術の選定

**質問1: プログラミング言語**
- 推奨案を提示した上で：「バックエンドの言語は何を使用しますか？」
  - 選択肢：Node.js、Python、Go、Java、Rust、C#/.NET、Ruby、PHP

**質問2: フレームワーク**
- 選択した言語に応じて推奨を提示：
  - Node.js: Express、Fastify、NestJS、Koa
  - Python: FastAPI、Django、Flask
  - Go: Gin、Echo、Chi、Fiber
  - Java: Spring Boot
  - Rust: Actix、Axum、Rocket
  - C#: ASP.NET Core
  - Ruby: Rails、Sinatra
  - PHP: Laravel、Symfony

**質問3: APIスタイル**
- 「APIのスタイルは何を採用しますか？」
  - REST API
  - GraphQL
  - gRPC
  - tRPC（TypeScript環境の場合）
  - ハイブリッド

**質問4: 認証・認可**
- 「認証方式は何を使用しますか？」
  - JWT（JSON Web Token）
  - セッションベース（Cookie）
  - OAuth 2.0 / OpenID Connect
  - 外部認証プロバイダー（Auth0、Clerk、Supabase Auth など）

#### 3.3. データベース技術の選定

**質問1: データベースタイプ**
- 推奨案を提示した上で：「データベースのタイプは何を使用しますか？」
  - RDBMS（PostgreSQL、MySQL、SQLite）
  - NoSQL（MongoDB、DynamoDB、Firestore）
  - ハイブリッド（RDBMSメイン + NoSQL補助）

**質問2: 具体的な製品**
- タイプに応じて選択肢を提示：
  - RDBMS: PostgreSQL、MySQL、MariaDB、SQLite
  - NoSQL（ドキュメント）: MongoDB、Firestore
  - NoSQL（キーバリュー）: DynamoDB、Redis
  - NoSQL（グラフ）: Neo4j

**質問3: ORM/クエリビルダー**
- 「ORM（Object-Relational Mapping）は何を使用しますか？」
  - Node.js: Prisma、TypeORM、Sequelize、Drizzle、生SQL
  - Python: SQLAlchemy、Django ORM、Tortoise ORM
  - Go: GORM、sqlx、生SQL
  - Java: Hibernate、JPA

**質問4: マイグレーション管理**
- 「データベースマイグレーションはどのように管理しますか？」
  - ORM内蔵機能
  - 専用ツール（Flyway、Liquibase、dbmate）
  - 手動SQLスクリプト

#### 3.4. キャッシュ・ストレージの選定

**質問1: キャッシュの必要性**
- 「キャッシュ層は必要ですか？」
  - 必要（初期から導入）
  - 後で導入（パフォーマンス要件次第）
  - 不要

- 必要な場合：
  - 「キャッシュ技術は何を使用しますか？」
  - 選択肢：Redis、Memcached、アプリケーション内メモリキャッシュ

**質問2: ファイルストレージ**
- 「ファイル（画像、動画、ドキュメント）の保存先は？」
  - クラウドストレージ（AWS S3、Google Cloud Storage、Azure Blob）
  - ローカルファイルシステム
  - CDN統合ストレージ（Cloudflare R2、Bunny.net）

#### 3.5. インフラ技術の選定

**質問1: デプロイ環境**
- 「アプリケーションのデプロイ先は？」
  - クラウドプロバイダー（AWS、GCP、Azure）
  - PaaS（Vercel、Netlify、Railway、Render）
  - オンプレミス
  - ハイブリッド

**質問2: コンテナ化**
- 「Dockerを使用しますか？」
  - はい（本番・開発両方）
  - はい（本番のみ）
  - はい（開発のみ）
  - いいえ

**質問3: オーケストレーション**（Dockerを使用する場合）
- 「コンテナオーケストレーションは必要ですか？」
  - Kubernetes
  - Docker Compose（開発・小規模本番）
  - AWS ECS/Fargate
  - 不要（単一コンテナ）

**質問4: IaC（Infrastructure as Code）**
- 「インフラをコードで管理しますか？」
  - Terraform
  - AWS CDK / CloudFormation
  - Pulumi
  - 手動管理

#### 3.6. CI/CD の選定

**質問1: CI/CDプラットフォーム**
- 「CI/CDは何を使用しますか？」
  - GitHub Actions
  - GitLab CI/CD
  - CircleCI
  - Jenkins
  - その他（Azure Pipelines、Bitbucket Pipelines）

**質問2: デプロイ戦略**
- 「デプロイ戦略は？」
  - ローリングデプロイ
  - ブルーグリーンデプロイ
  - カナリアリリース
  - シンプルデプロイ（ダウンタイムあり）

#### 3.7. 監視・Observability の選定

**質問1: 選定タイミング**
- 「監視ツールはいつ導入しますか？」
  - 初期（プロジェクト開始時）
  - 中期（MVP完成後）
  - 後期（本番運用開始時）

**質問2: エラートラッキング**
- 「エラートラッキングツールは？」
  - Sentry
  - Rollbar
  - Bugsnag
  - クラウドプロバイダーのツール（CloudWatch、GCP Error Reporting）
  - 不要

**質問3: ログ管理**
- 「ログはどのように管理しますか？」
  - クラウドプロバイダーのツール（CloudWatch Logs、GCP Cloud Logging）
  - ELK Stack（Elasticsearch、Logstash、Kibana）
  - Datadog、New Relic
  - シンプルなファイルログ

**質問4: APM（Application Performance Monitoring）**
- 「パフォーマンス監視は必要ですか？」
  - 必要（Datadog、New Relic、AppDynamics）
  - 後で検討
  - 不要

#### 3.8. テストツールの選定

**質問1: ユニットテスト**
- 「ユニットテストフレームワークは？」
  - JavaScript/TypeScript: Jest、Vitest、Mocha
  - Python: pytest、unittest
  - Go: 標準testing、testify
  - Java: JUnit

**質問2: E2Eテスト**
- 「E2Eテストは必要ですか？また、いつ導入しますか？」
  - 初期から導入（Playwright、Cypress、Selenium）
  - 中期（主要機能安定後）に導入
  - 後期（本番前）に導入
  - 不要

**質問3: その他のテスト**
- 「その他、必要なテストツールはありますか？」
  - パフォーマンステスト（k6、Gatling、JMeter）
  - セキュリティスキャン（Snyk、SonarQube、OWASP ZAP）
  - ビジュアルリグレッションテスト（Percy、Chromatic）

#### 3.9. 開発ツールの選定

**質問1: リンター・フォーマッター**
- 「コード品質ツールは？」
  - JavaScript/TypeScript: ESLint + Prettier、Biome
  - Python: Ruff、Black + Flake8、Pylint
  - Go: gofmt、golangci-lint

**質問2: 開発支援ツール**
- 「その他の開発ツールは？」
  - Storybook（UIコンポーネントカタログ）
  - Husky + lint-staged（pre-commit hooks）
  - commitlint（コミットメッセージ規約）

### 4. バージョン・ライセンス・EOLの確認

各選定した技術について：

**バージョン**:
- 「使用するバージョンは？」
- LTS（Long Term Support）の有無を確認
- 最新の安定版を推奨（ただし枯れたバージョンを選ぶことも検討）

**ライセンス**:
- 商用利用可能か確認
- GPL系ライセンスの場合は注意喚起

**EOL（End of Life）**:
- サポート終了日を確認
- EOLが近い場合は警告

### 5. 選定タイミングの分類

各技術を以下のタイミングに分類：

- **初期フェーズ**（プロジェクト開始時）
  - 言語、フレームワーク、データベース、インフラ、CI/CD、ユニットテスト

- **中期フェーズ**（主要機能実装後）
  - E2Eテスト、パフォーマンステスト、Storybook

- **後期フェーズ**（本番運用開始時）
  - 監視ツール、ログ集約、APM、キャッシュ層（要件次第）

- **随時フェーズ**（必要に応じて）
  - 特定ライブラリ、実験的ツール

### 6. 技術選定の基準の明確化

- 「技術選定で最も重視する基準は何ですか？」（複数選択可）
  - パフォーマンス
  - 開発者体験（DX）
  - コミュニティとサポート
  - コスト
  - チームのスキルセット
  - 長期的な保守性
  - セキュリティ

### 7. ドキュメント作成

- 収集した情報を基に、`docs/project/04-design/01-tech-stack.md` を作成

- テンプレートに従い、以下を記載：
  - テックスタック一覧（レイヤー、技術/ツール、バージョン、選定タイミング、メモ）
  - 技術選定の基準
  - 技術選定のタイミング戦略
  - 技術スタック図（Mermaid）
  - バージョン管理方針
  - ライセンス一覧（重要なもののみ）

- **HTMLコメントは削除せず残す**

### 8. レビューと確認

- 作成したドキュメントをユーザーに提示：
  - 「技術スタックドキュメントが完成しました。内容を確認してください。」
  - 「不足している技術や変更したい点はありますか？」

- 統計情報を表示：
  - 初期フェーズの技術数
  - 中期フェーズの技術数
  - 後期フェーズの技術数
  - 商用ライセンスの有無

### 9. 完成とコミット準備

- `docs/project/04-design/01-tech-stack.md` が保存されたことを確認

- 次のステップを提案：
  - 「技術スタックが決定しました。次はリポジトリ構造を定義しますか？（`/a-008-DefineRepositoryStructure`）」

## 完了条件

- `docs/project/04-design/01-tech-stack.md` が作成されている
- すべてのレイヤー（フロントエンド、バックエンド、データベース、インフラ、CI/CD、監視、テスト）が網羅されている
- 各技術について以下が記載されている：
  - 技術名
  - バージョン（メジャーバージョン以上）
  - 選定タイミング（初期/中期/後期/随時）
  - 選定理由（メモ欄）
- 技術選定の基準が明確に記載されている
- 技術選定のタイミング戦略が記載されている
- ユーザーが内容を確認し、承認している

## エスカレーション

- ユーザーが技術選定に迷っている場合：
  - 「どの点で迷われていますか？要件やチームの状況に基づいて、より詳細な推奨を提供できます。」
  - 代替案のメリット・デメリットを比較表で提示

- 非現実的な技術選定（過剰スペック、または不十分）が選択された場合：
  - 「この選択は要件に対して[過剰/不十分]です。以下の理由から、[代替案]を推奨します：[詳細]」

- チームのスキルセットと選定技術が大きく乖離している場合：
  - 「チームに[技術]の経験者がいない場合、学習コストが高くなります。トレーニング計画や外部支援を検討しませんか？」

- コストが高すぎる場合：
  - 「この構成はコストが高くなります（推定月額：[金額]）。初期フェーズではよりシンプルな構成から始め、必要に応じてスケールアップすることを推奨します。」

- EOLが近い技術が選択された場合：
  - 「[技術]はEOLが[日付]に予定されています。より新しいバージョンまたは代替技術を検討してください。」
