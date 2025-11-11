# slash-commands

## リポジトリ概要

Windsurfで活用するスラッシュコマンドとワークフローの保管庫。調査・要件定義から実装、テスト、デプロイ、Git運用までの開発プロセスを対象としたMarkdownドキュメントを蓄積し、他プロジェクトでも即座に呼び出せるナレッジベースを目指す。

## 目的

- 反復的な開発タスクを標準化し、スラッシュコマンドとして再利用可能にする。
- 調査、仕様策定、実装支援、デバッグ、レビュー、リリース、Git管理など各フェーズのベストプラクティスを明文化する。
- 実行可能コードは置かず、Windsurfでのコマンド入力やドキュメント参照に最適化したMarkdown中心の構成を維持する。

## ディレクトリ構成

- `.windsurf/workflows/` — スラッシュコマンド定義（各種ワークフローMarkdown）
  - `a-NNN-*.md` — プロジェクト設計ワークフロー（要件定義、設計、アーキテクチャ）
  - `b-NNN-*.md` — タスク管理ワークフロー（タスク作成、リサーチ、実装計画）
  - `c-NNN-*.md` — 実装実行ワークフロー（ステップバイステップ実装）
  - `x-Object-Action.md` — 補助的なワークフロー（例: `x-Code-Refactor`, `x-CI-Setup`, `x-Repository-PushToGithub`）
  - `z-*.md` — メタワークフロー（ワークフロー作成支援など）
- `.windsurf/templates/` — テンプレート定義（各種テンプレートMarkdown）
  - `project/` — プロジェクトレベルのテンプレート
  - `tasks/task-template/` — タスクレベルのテンプレート
    - `a-definition.md` — タスク定義テンプレート
    - `b-research.md` — リサーチテンプレート
    - `c-implementation.md` — 実装タスクリストテンプレート
- `docs/tasks/` — 実プロジェクトでのタスク管理ディレクトリ（各プロジェクトで作成）
  - `task000001-{スラッグ}/` — 個別タスクディレクトリ
    - `a-definition.md` — タスク定義（目的、ユーザーストーリー、受け入れ基準）
    - `b-research.md` — リサーチ結果（ベストプラクティス、既存コード調査）
    - `c-implementation.md` — 実装タスクリスト（フェーズ、ステップ、受け入れ基準）
- `README.md` — リポジトリの使い方と方針
- `CLAUDE.md` — Claude Code向けプロジェクト説明

## 使い方

### 基本的な使い方

1. Windsurfで必要なワークフローを参照したいタイミングで該当Markdownを開く。
2. コマンド化したい手順はスラッシュコマンドとしてコピーし、自身のプロジェクトで呼び出して利用する。
3. 新規ワークフローを追加する際は`.windsurf/workflows/`配下にMarkdownを作成し、ファイル名は用途が想起できるスラッグ形式にする（例: `jira-ticket.md`）。

### タスク管理の使い方

タスクの計画から実装まで、以下のワークフローを順次実行します：

1. **タスクディレクトリ作成**: `/b-000-CreateTaskDirectory`
   - タスクID自動採番（task000001, task000002, ...）
   - ディレクトリ作成: `docs/tasks/task000001-{スラッグ}/`
   - テンプレートファイル（a-definition.md, b-research.md, c-implementation.md）を自動コピー

2. **タスク定義**: `/b-001-CreateTaskDefinition`
   - 目的の明確化（解決する問題、提供する価値）
   - ユーザーストーリーの定義
   - 変更内容の詳細化（画面、データモデル、API、ビジネスロジック）
   - 受け入れ基準の定義（正常系、異常系、テスト、セキュリティ）

3. **リサーチ**: `/b-002-CreateTaskResearch`
   - ベストプラクティスの調査
   - 既存コードの調査と再利用可能性の確認
   - 技術選定（ライブラリ、フレームワーク）
   - 技術的リスクの洗い出しと軽減策

4. **実装計画**: `/b-003-CreateTaskImplementation`
   - フェーズ分割（1-3日粒度）
   - ステップ定義（1-3時間粒度）
   - 成果物の明確化（具体的なファイル名）
   - フェーズごとの受け入れ基準

5. **タスクレビュー**: `/b-004-ReviewTask`
   - タスク定義 ↔ 実装計画の一貫性チェック
   - リサーチ ↔ 実装計画の整合性検証
   - 変更内容・ユーザーストーリーのカバレッジ確認
   - 技術選定の反映、リスク軽減策の確認
   - 実装開始可否の判定

6. **実装実行**: `/c-001-ImplementTask`
   - 各ステップを順次実行（speckit.implementスタイル）
   - 依存関係を尊重、並列実行を最適化（`[P]`マーク）
   - ステップごとにテスト実行とチェックボックス更新
   - フェーズごとに受け入れ基準を確認
   - 最終テスト、コミット、PR作成

7. **ドキュメント更新**: `/c-002-UpdateDocumentation`
   - 実装完了後にドキュメントを実装内容に合わせて更新
   - タスクドキュメント更新（a-definition.md, b-research.md, c-implementation.md）
   - プロジェクトドキュメント更新（features-implemented, domain-model, api-spec, data-model）
   - 計画と実装の差異を記録
   - README, CHANGELOG, .env.example の更新
   - ドキュメント間の整合性確認

### タスクディレクトリ構造の例

```
docs/
└── tasks/
    ├── task000001-email-verification/
    │   ├── a-definition.md      # タスク定義（目的、ユーザーストーリー、受け入れ基準）
    │   ├── b-research.md         # リサーチ（ベストプラクティス、既存コード調査）
    │   └── c-implementation.md   # 実装タスクリスト（フェーズ、ステップ、受け入れ基準）
    ├── task000002-user-profile/
    │   ├── a-definition.md
    │   ├── b-research.md
    │   └── c-implementation.md
    └── task000003-payment-integration/
        ├── a-definition.md
        ├── b-research.md
        └── c-implementation.md
```

各タスクディレクトリには3つのドキュメントが含まれ、タスクの計画から実装まで一貫して管理できます。

## 更新ポリシー

- 各ドキュメントは最新の開発手法に合わせて継続的に改善する。
- 重複や冗長な情報は統合し、DRY原則を徹底する。
- 追加の開発フェーズや管理領域が生じた場合は、該当するスラッシュコマンドを新設してREADMEにも追記する。