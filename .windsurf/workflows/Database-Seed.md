---
description: 開発・テスト用のサンプルデータをデータベースに投入するワークフロー
auto_execution_mode: 1
---

# /x-SeedDatabase

## 目的

- 開発環境で動作確認するためのサンプルデータを投入する。
- テストで使用するフィクスチャデータを準備する。
- デモ環境で実際のデータに近いサンプルを表示する。
- 新規開発者が即座に開発を開始できるようデータを用意する。

## 前提

- データベースがセットアップされ、マイグレーションが完了している。
- データベース接続が確立されている（`DATABASE_URL` が設定済み）。
- Seed スクリプトまたはフィクスチャファイルが存在する（または新規作成）。
- Git リポジトリが初期化されている。

## 手順

### 1. Seed データの要件確認

**質問1: Seed データの目的**
- 「どのような目的でデータを投入しますか？」
  - 開発環境での動作確認
  - ユニットテスト用フィクスチャ
  - E2E テスト用データ
  - デモ環境用サンプル
  - パフォーマンステスト用大量データ

**質問2: 必要なデータの種類**
- 「どのテーブルにデータを投入しますか？」
- 「どのくらいのデータ量が必要ですか？」

### 2. 既存データの確認

**質問3: 既存データのクリア**
- 「既存データをクリアしてから投入しますか？」
  - はい（Clean Seed）: すべてのデータを削除してから投入
  - いいえ（Incremental Seed）: 既存データに追加

**既存データの確認**:
```bash
# PostgreSQL
psql -d <database_name> -c "SELECT COUNT(*) FROM users;"

# MySQL
mysql -u root -p <database_name> -e "SELECT COUNT(*) FROM users;"
```

### 3. Seed ファイルの作成/確認

#### 3.1. Seed ファイルの場所

**各フレームワークの Seed ファイル**:
- **Prisma**: `prisma/seed.ts` または `prisma/seed.js`
- **TypeORM**: `src/database/seeds/` ディレクトリ
- **Django**: `<app>/fixtures/*.json` または `<app>/management/commands/seed.py`
- **Rails**: `db/seeds.rb`
- **Sequelize**: `seeders/` ディレクトリ

#### 3.2. Seed スクリプトの作成

**Prisma** (`prisma/seed.ts`):
```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // ユーザーデータを投入
  const users = await Promise.all([
    prisma.user.create({
      data: {
        email: 'alice@example.com',
        name: 'Alice',
        age: 25,
      },
    }),
    prisma.user.create({
      data: {
        email: 'bob@example.com',
        name: 'Bob',
        age: 30,
      },
    }),
  ]);

  console.log(`Created ${users.length} users`);

  // 投稿データを投入
  const posts = await Promise.all([
    prisma.post.create({
      data: {
        title: 'Hello World',
        content: 'This is my first post',
        authorId: users[0].id,
      },
    }),
    prisma.post.create({
      data: {
        title: 'Getting Started',
        content: 'Learn how to use this app',
        authorId: users[1].id,
      },
    }),
  ]);

  console.log(`Created ${posts.length} posts`);
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

**Django** (`fixtures/sample_data.json`):
```json
[
  {
    "model": "myapp.user",
    "pk": 1,
    "fields": {
      "email": "alice@example.com",
      "name": "Alice",
      "age": 25
    }
  },
  {
    "model": "myapp.user",
    "pk": 2,
    "fields": {
      "email": "bob@example.com",
      "name": "Bob",
      "age": 30
    }
  }
]
```

**Rails** (`db/seeds.rb`):
```ruby
# ユーザーデータを投入
users = User.create([
  { email: 'alice@example.com', name: 'Alice', age: 25 },
  { email: 'bob@example.com', name: 'Bob', age: 30 }
])

puts "Created #{users.length} users"

# 投稿データを投入
posts = Post.create([
  { title: 'Hello World', content: 'This is my first post', user: users.first },
  { title: 'Getting Started', content: 'Learn how to use this app', user: users.second }
])

puts "Created #{posts.length} posts"
```

### 4. データ投入戦略の選択

**質問4: データ投入戦略**
- 「どの方法でデータを投入しますか？」
  - **ハードコード**: スクリプト内に直接データを記述（少量データ向け）
  - **JSON/YAML ファイル**: 外部ファイルからデータを読み込む（中量データ向け）
  - **Faker/Factory**: ランダムデータ生成（大量データ向け）
  - **実データのエクスポート**: 本番環境からデータをエクスポート（機密情報を除外）

#### 4.1. Faker を使用したランダムデータ生成

**Node.js (Faker)**:
```typescript
import { PrismaClient } from '@prisma/client';
import { faker } from '@faker-js/faker';

const prisma = new PrismaClient();

async function main() {
  const users = [];

  // 100人のユーザーを生成
  for (let i = 0; i < 100; i++) {
    const user = await prisma.user.create({
      data: {
        email: faker.internet.email(),
        name: faker.person.fullName(),
        age: faker.number.int({ min: 18, max: 80 }),
      },
    });
    users.push(user);
  }

  console.log(`Created ${users.length} users`);
}

main();
```

**Python (Faker)**:
```python
from faker import Faker
from myapp.models import User

fake = Faker()

# 100人のユーザーを生成
for _ in range(100):
    User.objects.create(
        email=fake.email(),
        name=fake.name(),
        age=fake.random_int(min=18, max=80)
    )

print("Created 100 users")
```

### 5. データの整合性確保

**質問5: 外部キー関係**
- 「データ間に外部キー関係がありますか？」
- 投入順序に注意：
  1. 親テーブル（外部キー制約なし）
  2. 子テーブル（外部キーを持つ）

**例**:
```typescript
// 正しい順序
// 1. ユーザーを作成
const user = await prisma.user.create({ ... });

// 2. ユーザーに紐づく投稿を作成
const post = await prisma.post.create({
  data: {
    title: 'Hello',
    authorId: user.id,  // 外部キー
  },
});
```

### 6. Seed の実行

#### 6.1. Seed コマンドの実行

**Prisma**:
```bash
npx prisma db seed
```

**TypeORM**:
```bash
npm run seed
# または
ts-node src/database/seeds/index.ts
```

**Django**:
```bash
# JSON フィクスチャから投入
python manage.py loaddata fixtures/sample_data.json

# カスタムコマンド
python manage.py seed
```

**Rails**:
```bash
rails db:seed
```

**Sequelize**:
```bash
npx sequelize-cli db:seed:all
```

**質問6: Seed 実行結果**
- 「データが正常に投入されましたか？」
- エラーが発生した場合、原因を特定：
  - 外部キー制約違反
  - 一意制約違反（既に同じデータが存在）
  - データ型の不一致
  - NOT NULL 制約違反

### 7. データの確認

**データベースに直接接続して確認**:
```bash
# PostgreSQL
psql -d <database_name>
SELECT * FROM users LIMIT 10;

# MySQL
mysql -u root -p <database_name>
SELECT * FROM users LIMIT 10;
```

**アプリケーションで確認**:
```bash
# 開発サーバーを起動
npm run dev

# ブラウザでアクセス
# http://localhost:3000
```

**質問7: データ確認**
- 「期待通りのデータが投入されましたか？」
- 確認すべき項目：
  - データ件数が正しい
  - リレーションが正しく設定されている
  - 画面で正しく表示される

### 8. データのリセット（必要に応じて）

**質問8: データのリセット**
- 「データをリセットして再投入したいですか？」

**データベースをリセット**:

**Prisma**:
```bash
npx prisma migrate reset
# 自動的に seed も実行される
```

**Rails**:
```bash
rails db:reset
# db:drop + db:create + db:migrate + db:seed
```

**Django**:
```bash
python manage.py flush
python manage.py loaddata fixtures/sample_data.json
```

**手動でリセット**:
```bash
# すべてのテーブルをトランケート
psql -d <database_name> -c "TRUNCATE TABLE users, posts CASCADE;"
```

### 9. Seed スクリプトの登録

**package.json に Seed コマンドを追加**:
```json
{
  "scripts": {
    "seed": "ts-node prisma/seed.ts",
    "db:reset": "npx prisma migrate reset && npm run seed"
  },
  "prisma": {
    "seed": "ts-node prisma/seed.ts"
  }
}
```

### 10. ドキュメント化

**README.md に Seed 手順を追加**:
```markdown
## Development Setup

### Seed Database

To populate the database with sample data:

\`\`\`bash
npm run seed
\`\`\`

To reset the database and seed:

\`\`\`bash
npm run db:reset
\`\`\`

### Sample Accounts

- Admin: admin@example.com / password123
- User: user@example.com / password123
```

### 11. Git コミット

**Seed ファイルをコミット**:
```bash
git add prisma/seed.ts
# または
git add db/seeds.rb
# または
git add fixtures/

git commit -m "feat: add database seed script

- Add sample users and posts
- Support random data generation with Faker
- Document seed command in README

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

## 完了条件

- Seed スクリプトが作成されている
- データが正常に投入されている
- データの整合性が保たれている（外部キー、制約など）
- アプリケーションでデータが正しく表示される
- Seed コマンドが `package.json` または README に記載されている
- Seed ファイルが Git にコミットされている

## エスカレーション

- **一意制約違反エラー**:
  - 「同じデータが既に存在します。以下を検討してください：」
    - データをリセットしてから投入
    - `upsert` を使用（存在すれば更新、なければ作成）
    - ユニークなデータを生成（UUID、Faker など）

- **外部キー制約違反**:
  - 「外部キーが存在しません。データの投入順序を確認してください：」
    - 親テーブルを先に投入
    - トランザクションで一括投入
    - 外部キー制約を一時的に無効化（非推奨）

- **パフォーマンス問題（大量データ）**:
  - 「大量データの投入に時間がかかっています。以下を検討してください：」
    - バルクインサート（一括挿入）を使用
    - トランザクションでまとめて投入
    - インデックスを一時的に無効化

- **本番環境でのシード禁止**:
  - 「本番環境では seed を実行しないでください。」
  - 環境変数で制御：
    ```typescript
    if (process.env.NODE_ENV === 'production') {
      throw new Error('Cannot seed production database');
    }
    ```

## ベストプラクティス

- **環境による切り替え**: 本番環境では seed を実行しない
- **冪等性**: 何度実行しても同じ結果になるようにする（upsert 使用）
- **リアルなデータ**: 実際の使用に近いデータを用意
- **パスワードハッシュ**: ユーザーのパスワードは必ずハッシュ化
- **機密情報の除外**: 本番データをエクスポートする場合、個人情報を除外
- **バージョン管理**: Seed ファイルは Git で管理
- **ドキュメント化**: README に seed コマンドとサンプルアカウント情報を記載
- **自動化**: CI/CD でテスト前に seed を実行
- **大量データはオプション**: 基本的なデータのみデフォルトで投入、大量データは別コマンド
