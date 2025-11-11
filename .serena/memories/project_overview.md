# slash-commands プロジェクト概要

## プロジェクトの目的
Windsurfで活用するスラッシュコマンドとワークフローの保管庫。開発ライフサイクル全般（調査・要件定義・設計・実装・テスト・Git運用）を対象としたMarkdownベースのワークフロー定義を蓄積し、他プロジェクトでも即座に呼び出せるナレッジベースを構築する。

## テックスタック
- **主要言語**: Markdown（ドキュメント中心）
- **実行環境**: Windsurf IDE + Cascade
- **バージョン管理**: Git
- **オペレーティングシステム**: Windows
- **ファイルエンコーディング**: UTF-8

## プロジェクト構造
```
slash-commands/
├── .windsurf/
│   ├── workflows/           # スラッシュコマンド定義
│   │   ├── create-workflow.md
│   │   ├── create-system-doc.md
│   │   ├── dev-1-plan-intake.md
│   │   ├── dev-2-planning.md
│   │   ├── dev-3-system-design.md
│   │   ├── dev-4-task-authoring.md
│   │   ├── dev-5-implementation.md
│   │   ├── dev-6-commit-message.md
│   │   └── dev-7-swarm-qa.md
│   └── templates/           # 再利用可能なテンプレート
├── README.md                # リポジトリの使い方と方針
├── CLAUDE.md               # Claude Codeへのガイダンス
├── slash-commands-best-practices.md  # ワークフロー設計ベストプラクティス
└── .gitignore              # Git無視ファイル
```

## ワークフロー体系
開発フェーズごとに番号付きワークフローを定義：
1. `/dev-1-plan-intake` - 依頼内容収集と関係者合意
2. `/dev-2-planning` - PRD作成とアーキテクチャ設計
3. `/dev-3-system-design` - 詳細システム設計
4. `/dev-4-task-authoring` - Storyファイル作成とタスク化
5. `/dev-5-implementation` - 実装・テスト・PR作成
6. `/dev-6-commit-message` - コミットメッセージ生成
7. `/dev-7-swarm-qa` - マルチエージェントQA

補助ワークフロー：
- `/create-workflow` - 新規ワークフロー追加の標準手順
- `/create-system-doc` - システム概要ドキュメント作成

## 設計哲学
- **Markdown中心**: 実行可能コードは含めず、Windsurfのコマンド参照に特化
- **コンテキスト最適化**: 12,000文字制限を意識し、長文は`@path/to/file.md`形式で参照
- **チェイン構築**: ワークフロー間で`Call /workflow-name`により連携
- **BMAD Method適用**: 多層エージェント計画の考え方を反映
- **Claude-Flowスキル活用**: pair-programming、agentdb-vector-search等の組み込み

## 運用方針
- **DRY原則**: 重複・冗長な情報は統合し、参照形式で再利用
- **継続改善**: 実行後のフィードバックを定期的に反映
- **セキュリティ**: Memories/Rulesで権限管理、監査ログ記録、レート制御を徹底
- **ドキュメント整備**: READMEに新規ワークフロー概要を追記し、チームへ周知

## Git運用
- ブランチ戦略は各プロジェクトのポリシーに従う（trunk-based推奨）
- コミットメッセージにはStory IDまたは課題番号を含める
- PR作成時はテンプレートを利用し、変更概要/テスト結果/影響範囲を明記