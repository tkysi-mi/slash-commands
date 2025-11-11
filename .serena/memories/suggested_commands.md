# slash-commands 推奨コマンド

## 基本コマンド
- **git status**: リポジトリの状態確認
- **git add .**: 変更ファイルをステージング
- **git commit -m "message"**: コミット作成
- **git push**: リモートへプッシュ
- **git pull**: リモートからプル
- **git log --oneline**: コミット履歴確認

## ファイル操作
- **ls -la**: ディレクトリ内容詳細表示
- **find . -name "*.md"**: Markdownファイル検索
- **grep -r "pattern" .**: パターン検索
- **cat file.md**: ファイル内容表示
- **head -n 20 file.md**: ファイル先頭20行表示
- **tail -n 20 file.md**: ファイル末尾20行表示

## Windsurfワークフロー実行
- **/create-workflow**: 新規ワークフロー作成
- **/create-system-doc**: システム概要ドキュメント作成
- **/dev-1-plan-intake**: 依頼内容収集と関係者合意
- **/dev-2-planning**: PRD作成とアーキテクチャ設計
- **/dev-3-system-design**: 詳細システム設計
- **/dev-4-task-authoring**: Storyファイル作成とタスク化
- **/dev-5-implementation**: 実装・テスト・PR作成
- **/dev-6-commit-message**: コミットメッセージ生成
- **/dev-7-swarm-qa**: マルチエージェントQA

## プロジェクト管理
- **README.md**: プロジェクト概要と使い方確認
- **@slash-commands-best-practices.md**: ベストプラクティス参照
- **@CLAUDE.md**: Claude Codeへのガイダンス確認
- **tree .**: プロジェクト構造可視化
- **wc -l *.md**: Markdownファイル行数確認

## 品質チェック
- **markdownlint *.md**: Markdown記法チェック
- **spellcheck file.md**: スペルチェック
- **git diff --check**: 空白文字問題チェック
- **pre-commit run**: pre-commitフック実行

## Windows特有コマンド
- **dir**: ディレクトリ内容表示（lsの代替）
- **type file.md**: ファイル内容表示（catの代替）
- **findstr "pattern" file.md**: パターン検索（grepの代替）
- **copy source.md dest.md**: ファイルコピー（cpの代替）
- **del file.md**: ファイル削除（rmの代替）

## 便利なエイリアス（推奨）
- **alias gs="git status"**: Git状態確認
- **alias ga="git add"**: Gitステージング
- **alias gc="git commit"**: Gitコミット
- **alias gp="git push"**: Gitプッシュ
- **alias ll="ls -la"**: 詳細ファイル一覧