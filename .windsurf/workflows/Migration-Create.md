---
description: データベーススキーマの変更を安全にマイグレーションファイルとして作成するワークフロー
auto_execution_mode: 1
---

# /x-CreateMigration

## 目的

- データベーススキーマの変更を安全かつ追跡可能な形でマイグレーションファイルとして作成する。
- 本番環境へのデプロイ時にスキーマ変更を自動適用できるようにする。
- ロールバック可能なマイグレーションを設計し、問題発生時に迅速に戻せるようにする。
- チーム全体でスキーマ変更を共有し、環境間の一貫性を保つ。

## 前提

- データベース ORM または マイグレーションツールが設定されている。
  - Node.js: Prisma, TypeORM, Sequelize, Knex.js
  - Python: Django ORM, SQLAlchemy + Alembic, Peewee
  - Ruby: Active Record (Rails)
- データベース接続が確立されている（`DATABASE_URL` が設定済み）。
- 既存のマイグレーションがすべて適用されている（clean state）。
- Git リポジトリが初期化されている。

## 手順

### 1. スキーマ変更の要件確認

**質問1: スキーマ変更の種類**
- 「どのような変更を行いますか？」
  - 新しいテーブルの追加
  - 既存テーブルへのカラム追加
  - カラムの削除
  - カラムのデータ型変更
  - カラム名の変更
  - インデックスの追加/削除
  - 外部キー制約の追加/削除
  - 一意制約の追加/削除
  - デフォルト値の変更
  - データマイグレーション（既存データの変換）

**質問2: 影響範囲の確認**
- 「この変更はどのテーブルに影響しますか？」
- 「既存データへの影響はありますか？」
- 「破壊的変更（データ損失の可能性）がありますか？」

### 2. 既存マイグレーションの状態確認

**現在のマイグレーション状態を確認**:

**Prisma**:
```bash
npx prisma migrate status
```

**TypeORM**:
```bash
npm run migration:show
```

**Django**:
```bash
python manage.py showmigrations
```

**Alembic (SQLAlchemy)**:
```bash
alembic current
alembic history
```

**Rails**:
```bash
rails db:migrate:status
```

**質問3: マイグレーション状態**
- 「すべてのマイグレーションが適用されていますか？」
- 未適用のマイグレーションがある場合、先に適用：
  ```bash
  npm run migration:run
  # または
  npx prisma migrate deploy
  ```

### 3. スキーマの設計

#### 3.1. スキーマファイルの編集

**Prisma** (`schema.prisma`):
```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  // 新しいフィールドを追加
  age       Int?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**TypeORM** (Entity ファイル):
```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column({ nullable: true })
  name: string;

  // 新しいカラムを追加
  @Column({ nullable: true })
  age: number;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

**Django** (`models.py`):
```python
class User(models.Model):
    email = models.EmailField(unique=True)
    name = models.CharField(max_length=100, null=True, blank=True)
    # 新しいフィールドを追加
    age = models.IntegerField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

#### 3.2. マイグレーションファイルの生成

**Prisma**:
```bash
npx prisma migrate dev --name add_age_to_users
```

**TypeORM**:
```bash
npm run migration:generate -- -n AddAgeToUsers
```

**Django**:
```bash
python manage.py makemigrations
```

**Alembic**:
```bash
alembic revision --autogenerate -m "add age to users"
```

**Rails**:
```bash
rails generate migration AddAgeToUsers age:integer
```

### 4. マイグレーションファイルの確認と編集

**質問4: マイグレーションファイルの確認**
- 「生成されたマイグレーションファイルを確認してください。」
- 確認すべきポイント：
  - SQL が正しいか
  - データ損失のリスクがないか
  - パフォーマンスへの影響（大量データへのインデックス追加など）
  - ロールバック（down/undo）が正しく実装されているか

**マイグレーションファイルの例（TypeORM）**:
```typescript
import { MigrationInterface, QueryRunner } from "typeorm";

export class AddAgeToUsers1234567890123 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE "user" ADD "age" integer NULL
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE "user" DROP COLUMN "age"
    `);
  }
}
```

### 5. データマイグレーション（必要に応じて）

**質問5: データマイグレーションの必要性**
- 「既存データの変換が必要ですか？」

**データマイグレーションの例**:

**TypeORM**:
```typescript
public async up(queryRunner: QueryRunner): Promise<void> {
  // カラム追加
  await queryRunner.query(`
    ALTER TABLE "user" ADD "full_name" varchar(200) NULL
  `);

  // 既存データの変換
  await queryRunner.query(`
    UPDATE "user" SET "full_name" = CONCAT("first_name", ' ', "last_name")
  `);

  // NOT NULL 制約を追加（データ変換後）
  await queryRunner.query(`
    ALTER TABLE "user" ALTER COLUMN "full_name" SET NOT NULL
  `);
}
```

**Django**:
```python
from django.db import migrations

def migrate_data(apps, schema_editor):
    User = apps.get_model('myapp', 'User')
    for user in User.objects.all():
        user.full_name = f"{user.first_name} {user.last_name}"
        user.save()

class Migration(migrations.Migration):
    dependencies = [
        ('myapp', '0001_add_full_name_field'),
    ]

    operations = [
        migrations.RunPython(migrate_data),
    ]
```

### 6. ローカル環境でのテスト

#### 6.1. マイグレーションの適用

**Prisma**:
```bash
npx prisma migrate dev
```

**TypeORM**:
```bash
npm run migration:run
```

**Django**:
```bash
python manage.py migrate
```

**Alembic**:
```bash
alembic upgrade head
```

**Rails**:
```bash
rails db:migrate
```

**質問6: マイグレーション適用結果**
- 「マイグレーションが正常に適用されましたか？」
- エラーが発生した場合、原因を特定して修正

#### 6.2. スキーマの確認

**データベースに直接接続して確認**:
```bash
# PostgreSQL
psql -d <database_name>
\d users

# MySQL
mysql -u root -p <database_name>
DESCRIBE users;
```

#### 6.3. アプリケーションの動作確認

**開発サーバーを起動**:
```bash
npm run dev
```

**質問7: アプリケーション動作確認**
- 「アプリケーションが正常に動作しますか？」
- 確認すべき項目：
  - CRUD 操作が正常に動作する
  - 新しいカラムが使用できる
  - 既存機能に影響がない

### 7. ロールバックのテスト

**質問8: ロールバックのテスト**
- 「ロールバックが正常に動作するかテストしますか？」

**マイグレーションをロールバック**:

**Prisma**:
```bash
npx prisma migrate resolve --rolled-back <migration_name>
```

**TypeORM**:
```bash
npm run migration:revert
```

**Django**:
```bash
python manage.py migrate <app_name> <previous_migration_number>
```

**Alembic**:
```bash
alembic downgrade -1
```

**Rails**:
```bash
rails db:rollback
```

**ロールバック後の確認**:
- スキーマが元に戻っているか確認
- アプリケーションが正常に動作するか確認

**再度マイグレーションを適用**:
```bash
npm run migration:run
```

### 8. テストの実行

**すべてのテストを実行**:
```bash
npm test
```

**質問9: テスト結果**
- 「すべてのテストが通りましたか？」
- 失敗した場合、マイグレーションの影響を確認

### 9. マイグレーションファイルのコミット

**Git でマイグレーションファイルをコミット**:
```bash
git add migrations/
# または
git add prisma/migrations/
# または
git add <migration_file>

git commit -m "feat: add age column to users table

- Add nullable age column to users table
- Migration file: <migration_file>
- Supports rollback

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

### 10. ドキュメントの更新

**質問10: ドキュメント更新**
- 「スキーマ変更に伴うドキュメント更新が必要ですか？」
- 更新すべきドキュメント：
  - データモデルドキュメント
  - API ドキュメント（新しいフィールドを追加した場合）
  - ERD（Entity Relationship Diagram）

## 完了条件

- マイグレーションファイルが生成されている
- マイグレーションがローカル環境で正常に適用される
- ロールバックが正常に動作する
- アプリケーションが正常に動作する
- すべてのテストが通る
- マイグレーションファイルが Git にコミットされている
- 必要なドキュメントが更新されている

## エスカレーション

- **マイグレーションの適用が失敗する場合**:
  - 「マイグレーションの適用に失敗しました。以下を確認してください：」
    - SQL 構文エラー
    - データ型の不一致
    - 外部キー制約違反
    - 一意制約違反
    - NOT NULL 制約違反（既存データに NULL が存在）

- **データ損失のリスクがある場合**:
  - 「この変更はデータ損失のリスクがあります。以下を検討してください：」
    - バックアップを取る
    - 段階的にマイグレーションを分割
    - データ変換スクリプトを別途作成
    - カラム削除前に deprecated フラグを設定

- **本番環境でのパフォーマンス影響**:
  - 「大量データへのインデックス追加など、パフォーマンスに影響する変更の場合：」
    - メンテナンスウィンドウでの実行を検討
    - `CONCURRENTLY` オプションを使用（PostgreSQL）
    - オンラインスキーマ変更ツールを使用（gh-ost, pt-online-schema-change）

- **競合するマイグレーションがある場合**:
  - 「他の開発者のマイグレーションと競合しています。以下を実施してください：」
    - `git pull` で最新の変更を取得
    - マイグレーションの順序を調整
    - マイグレーションをマージまたは統合

## ベストプラクティス

- **小さく分割**: 大きなスキーマ変更は複数のマイグレーションに分割
- **後方互換性**: 可能な限り後方互換性のある変更を優先
- **段階的な削除**: カラム削除は2段階で実施
  1. アプリケーションコードから削除
  2. 次のリリースでカラムを削除
- **デフォルト値**: NOT NULL カラムを追加する場合はデフォルト値を設定
- **インデックス**: パフォーマンスに影響するカラムにはインデックスを追加
- **外部キー**: データ整合性のため、適切な外部キー制約を設定
- **テスト**: マイグレーション適用前にステージング環境でテスト
- **バックアップ**: 本番環境でのマイグレーション前に必ずバックアップ
- **ロールバック計画**: 問題発生時のロールバック手順を事前に準備
- **命名規則**: マイグレーションファイル名は変更内容が分かりやすいものに
