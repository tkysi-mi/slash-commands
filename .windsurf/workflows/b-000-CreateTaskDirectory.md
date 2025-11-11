---
description: 新規タスク用のディレクトリを作成し、テンプレートファイルをコピーして初期化するワークフロー
auto_execution_mode: 1
---

# /b-000-CreateTaskDirectory

## 目的

- 新しいタスクのための専用ディレクトリを作成する。
- タスクIDの採番ルールを統一し、管理しやすくする。
- テンプレートファイルを自動的にコピーして、作業を開始しやすくする。
- タスクディレクトリ構造を標準化し、プロジェクト全体で一貫性を保つ。

## 前提

- `docs/tasks/` ディレクトリが存在すること（存在しない場合は `/a-001-SetupDocStructure` を実行）
- `.windsurf/templates/tasks/task-template/` ディレクトリにテンプレートファイルが存在すること：
  - `a-definition.md`（タスク定義テンプレート）
  - `b-research.md`（リサーチテンプレート）
  - `c-implementation.md`（実装タスクリストテンプレート）

## 手順

### 1. 既存タスクの確認と次のタスクIDの決定

#### 1.1. 既存タスクディレクトリの確認

- `docs/tasks/` ディレクトリ内の既存タスクディレクトリを確認
- タスクIDの最大番号を取得

例：
- 既存タスク: `task000001-email-verification`, `task000002-user-profile`
- 次のタスクID: `task000003`

**使用するコマンド**:
```bash
ls docs/tasks/
```

または Glob ツールで:
```
pattern: docs/tasks/task*
```

#### 1.2. 次のタスクIDの自動採番

- タスクIDは `task000001`, `task000002`, ... の形式で6桁のゼロパディング
- 既存タスクがない場合は `task000001` から開始
- 既存タスクがある場合は最大番号 + 1

例：
- 既存タスクなし → `task000001`
- 既存タスク最大 `task000005` → `task000006`

### 2. タスク名（スラッグ）の決定

#### 2.1. タスク名のヒアリング

**質問1: タスクの簡潔な説明**
- 「このタスクの簡潔な説明を教えてください。」
- 「タスク名として使用するキーワードは何ですか？」

**タスク名（スラッグ）の命名規則**:
- 英数字とハイフンのみ使用（小文字）
- スペースは使用せず、ハイフンで区切る
- 3〜5語以内で簡潔に
- 機能や変更内容が一目でわかる名前

例：
- ✅ `email-verification`（メール認証機能）
- ✅ `user-profile-edit`（ユーザープロフィール編集）
- ✅ `payment-integration`（決済機能統合）
- ❌ `new-feature`（抽象的すぎる）
- ❌ `task-for-user`（具体性がない）

#### 2.2. タスクIDとスラッグの結合

- フォーマット: `task{6桁番号}-{スラッグ}`
- 例: `task000001-email-verification`

### 3. タスクディレクトリの作成

#### 3.1. ディレクトリ作成

- `docs/tasks/{task-id}/` ディレクトリを作成

例：
```bash
mkdir -p docs/tasks/task000001-email-verification
```

#### 3.2. ディレクトリ構造の確認

作成後のディレクトリ構造：
```
docs/tasks/
└── task000001-email-verification/
    ├── a-definition.md（次のステップでコピー）
    ├── b-research.md（次のステップでコピー）
    └── c-implementation.md（次のステップでコピー）
```

### 4. テンプレートディレクトリのコピー

#### 4.1. テンプレートディレクトリの確認

- `.windsurf/templates/tasks/task-template/` ディレクトリが存在することを確認
- ディレクトリ内に以下のファイルが存在することを確認：
  - `a-definition.md`
  - `b-research.md`
  - `c-implementation.md`

#### 4.2. ディレクトリ全体のコピー

テンプレートディレクトリ全体を新しいタスクディレクトリにコピー：

**方法1: cp コマンドを使用（推奨）**
```bash
cp -r .windsurf/templates/tasks/task-template docs/tasks/task000001-email-verification
```

**方法2: 個別ファイルコピー（代替）**
```bash
mkdir -p docs/tasks/task000001-email-verification
cp .windsurf/templates/tasks/task-template/a-definition.md docs/tasks/task000001-email-verification/
cp .windsurf/templates/tasks/task-template/b-research.md docs/tasks/task000001-email-verification/
cp .windsurf/templates/tasks/task-template/c-implementation.md docs/tasks/task000001-email-verification/
```

**注意**:
- テンプレート内の HTML コメントは削除せず、そのまま残す
- プレースホルダー（`{スラッグ}` など）は必要に応じて後で置換

### 5. タスクディレクトリの確認

#### 5.1. 作成されたファイルの確認

- タスクディレクトリ内にテンプレートファイルがコピーされていることを確認

```bash
ls -la docs/tasks/task000001-email-verification/
```

期待される出力：
```
a-definition.md
b-research.md
c-implementation.md
```

#### 5.2. ファイル内容の確認（オプション）

- 各ファイルが正しくコピーされ、テンプレート構造が保持されていることを確認
- HTMLコメントが削除されていないことを確認

### 6. タスクディレクトリパスの提示

- 作成したタスクディレクトリのパスをユーザーに提示：
  - 「タスクディレクトリを作成しました：`docs/tasks/task000001-email-verification/`」
  - 「以下のファイルが作成されました：」
    - `a-definition.md`（タスク定義ドキュメント）
    - `b-research.md`（リサーチドキュメント）
    - `c-implementation.md`（実装タスクリスト）

### 7. 次のステップの提案

- 「タスクディレクトリが作成されました。次はタスク定義ドキュメントを作成しますか？（`/b-001-CreateTaskDefinition`）」

## 完了条件

- `docs/tasks/task{番号}-{スラッグ}/` ディレクトリが作成されている
- タスクIDが既存タスクと重複していない
- 以下のテンプレートファイルがコピーされている：
  - `a-definition.md`
  - `b-research.md`
  - `c-implementation.md`
- テンプレートファイル内の HTML コメントが削除されずに保持されている
- ユーザーにタスクディレクトリパスが提示されている

## エスカレーション

- `docs/tasks/` ディレクトリが存在しない場合：
  - 「`docs/tasks/` ディレクトリが存在しません。先に `/a-001-SetupDocStructure` を実行してプロジェクトドキュメント構造を作成してください。」

- テンプレートディレクトリが存在しない場合：
  - 「テンプレートディレクトリが見つかりません。`.windsurf/templates/tasks/task-template/` ディレクトリに以下のファイルが存在することを確認してください：」
    - `a-definition.md`
    - `b-research.md`
    - `c-implementation.md`
  - 「テンプレートファイルが不足している場合は、このリポジトリから `.windsurf/templates/tasks/task-template/` をコピーしてください。」

- タスク名が命名規則に違反している場合：
  - 「タスク名は英数字とハイフンのみを使用してください。スペースや特殊文字は使用できません。」
  - 「例: `email-verification`, `user-profile-edit`」

- タスクディレクトリが既に存在する場合：
  - 「タスクディレクトリ `docs/tasks/task{番号}-{スラッグ}/` は既に存在します。」
  - 「既存のタスクを使用しますか？それとも別のタスクIDを採番しますか？」

## タスクID採番の例

| 既存タスク | 次のタスクID | 作成されるディレクトリ |
|-----------|-------------|----------------------|
| なし | `task000001` | `docs/tasks/task000001-email-verification/` |
| `task000001-email-verification` | `task000002` | `docs/tasks/task000002-user-profile/` |
| `task000001-email-verification`<br>`task000002-user-profile`<br>`task000003-payment` | `task000004` | `docs/tasks/task000004-notification/` |

## 参考：ディレクトリ構造の全体像

```
docs/
└── tasks/
    ├── task000001-email-verification/
    │   ├── a-definition.md
    │   ├── b-research.md
    │   └── c-implementation.md
    ├── task000002-user-profile/
    │   ├── a-definition.md
    │   ├── b-research.md
    │   └── c-implementation.md
    └── task000003-payment-integration/
        ├── a-definition.md
        ├── b-research.md
        └── c-implementation.md
```
