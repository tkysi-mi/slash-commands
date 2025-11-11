---
description: React/Vueコンポーネントをボイラープレート、Props型定義、Storybook、テストと共に生成するワークフロー
auto_execution_mode: 1
---

# /x-CreateComponent

## 目的

- React/Vue コンポーネントを素早く作成し、開発効率を向上させる。
- Props 型定義、Storybook ストーリー、テストファイルを一括で生成する。
- プロジェクトの命名規則とディレクトリ構造に従ったコンポーネントを作成する。
- アトミックデザインや Feature-Sliced Design などのアーキテクチャパターンに対応する。

## 前提

- React または Vue プロジェクトがセットアップされている。
- TypeScript が使用されている（推奨）。
- Storybook がセットアップされている（オプション）。
- テストフレームワーク（Jest, Vitest, Testing Library）がセットアップされている（オプション）。
- プロジェクトの命名規則とディレクトリ構造が決まっている。

## 手順

### 1. コンポーネント情報の収集

**質問1: フレームワークの確認**
- 「使用しているフレームワークは何ですか？」
  - React
  - Vue 3
  - Next.js
  - Nuxt 3

**質問2: コンポーネント名**
- 「コンポーネント名を教えてください。」
- 命名規則：
  - PascalCase（例: `UserCard`, `NavigationBar`）
  - 明確で説明的な名前
  - 単数形または複数形を適切に使い分け

**質問3: コンポーネントの種類**
- 「どの種類のコンポーネントですか？」
  - **Atoms（原子）**: 最小単位（Button, Input, Label）
  - **Molecules（分子）**: Atoms の組み合わせ（SearchBar, FormField）
  - **Organisms（有機体）**: Molecules の組み合わせ（Header, UserProfile）
  - **Templates（テンプレート）**: ページレイアウト
  - **Pages（ページ）**: 実際のページ

**質問4: ディレクトリ構造**
- 「プロジェクトのディレクトリ構造は何ですか？」
  - Atomic Design: `src/components/atoms/`, `src/components/molecules/`, etc.
  - Feature-Sliced Design: `src/features/<feature>/ui/`
  - Flat: `src/components/`
  - Layered: `src/presentation/components/`

### 2. コンポーネントファイルの作成

#### 2.1. ディレクトリ構造の決定

**コンポーネントディレクトリ**:
```
src/components/atoms/Button/
├── Button.tsx           # メインコンポーネント
├── Button.module.css    # スタイル (CSS Modules の場合)
├── Button.stories.tsx   # Storybook ストーリー
├── Button.test.tsx      # テストファイル
└── index.ts             # エクスポート
```

#### 2.2. メインコンポーネントファイルの作成

**React + TypeScript** (`Button.tsx`):
```typescript
import React from 'react';
import styles from './Button.module.css';

export interface ButtonProps {
  /**
   * ボタンのラベル
   */
  label: string;
  /**
   * ボタンのバリエーション
   */
  variant?: 'primary' | 'secondary' | 'danger';
  /**
   * ボタンのサイズ
   */
  size?: 'small' | 'medium' | 'large';
  /**
   * 無効状態
   */
  disabled?: boolean;
  /**
   * クリックイベントハンドラ
   */
  onClick?: () => void;
}

/**
 * ボタンコンポーネント
 */
export const Button: React.FC<ButtonProps> = ({
  label,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  onClick,
}) => {
  return (
    <button
      className={`${styles.button} ${styles[variant]} ${styles[size]}`}
      disabled={disabled}
      onClick={onClick}
      type="button"
    >
      {label}
    </button>
  );
};
```

**Vue 3 + TypeScript** (`Button.vue`):
```vue
<script setup lang="ts">
export interface ButtonProps {
  /** ボタンのラベル */
  label: string;
  /** ボタンのバリエーション */
  variant?: 'primary' | 'secondary' | 'danger';
  /** ボタンのサイズ */
  size?: 'small' | 'medium' | 'large';
  /** 無効状態 */
  disabled?: boolean;
}

const props = withDefaults(defineProps<ButtonProps>(), {
  variant: 'primary',
  size: 'medium',
  disabled: false,
});

const emit = defineEmits<{
  click: [];
}>();

const handleClick = () => {
  emit('click');
};
</script>

<template>
  <button
    :class="['button', variant, size]"
    :disabled="disabled"
    @click="handleClick"
  >
    {{ label }}
  </button>
</template>

<style scoped>
.button {
  /* ボタンスタイル */
}
</style>
```

#### 2.3. スタイルファイルの作成

**CSS Modules** (`Button.module.css`):
```css
.button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.button:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Variants */
.primary {
  background-color: #007bff;
  color: white;
}

.secondary {
  background-color: #6c757d;
  color: white;
}

.danger {
  background-color: #dc3545;
  color: white;
}

/* Sizes */
.small {
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
}

.medium {
  padding: 0.5rem 1rem;
  font-size: 1rem;
}

.large {
  padding: 0.75rem 1.5rem;
  font-size: 1.125rem;
}
```

#### 2.4. Storybook ストーリーの作成

**React** (`Button.stories.tsx`):
```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta = {
  title: 'Atoms/Button',
  component: Button,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger'],
    },
    size: {
      control: 'select',
      options: ['small', 'medium', 'large'],
    },
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    label: 'Button',
    variant: 'primary',
  },
};

export const Secondary: Story = {
  args: {
    label: 'Button',
    variant: 'secondary',
  },
};

export const Danger: Story = {
  args: {
    label: 'Button',
    variant: 'danger',
  },
};

export const Small: Story = {
  args: {
    label: 'Button',
    size: 'small',
  },
};

export const Large: Story = {
  args: {
    label: 'Button',
    size: 'large',
  },
};

export const Disabled: Story = {
  args: {
    label: 'Button',
    disabled: true,
  },
};
```

#### 2.5. テストファイルの作成

**React Testing Library** (`Button.test.tsx`):
```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('renders with label', () => {
    render(<Button label="Click me" />);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button label="Click me" onClick={handleClick} />);

    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('does not call onClick when disabled', () => {
    const handleClick = jest.fn();
    render(<Button label="Click me" onClick={handleClick} disabled />);

    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).not.toHaveBeenCalled();
  });

  it('applies variant class', () => {
    const { rerender } = render(<Button label="Button" variant="primary" />);
    expect(screen.getByText('Button')).toHaveClass('primary');

    rerender(<Button label="Button" variant="secondary" />);
    expect(screen.getByText('Button')).toHaveClass('secondary');
  });

  it('applies size class', () => {
    const { rerender } = render(<Button label="Button" size="small" />);
    expect(screen.getByText('Button')).toHaveClass('small');

    rerender(<Button label="Button" size="large" />);
    expect(screen.getByText('Button')).toHaveClass('large');
  });
});
```

#### 2.6. Index ファイルの作成

**Index** (`index.ts`):
```typescript
export { Button } from './Button';
export type { ButtonProps } from './Button';
```

### 3. コンポーネントの動作確認

**質問5: 動作確認**
- 「Storybook で動作確認しますか？」

**Storybook を起動**:
```bash
npm run storybook
```

**ブラウザでアクセス**:
- `http://localhost:6006`
- Atoms/Button を確認

### 4. テストの実行

**テストを実行**:
```bash
npm test Button
# または
npm test -- Button.test.tsx
```

**質問6: テスト結果**
- 「すべてのテストが通りましたか？」

### 5. コンポーネントの使用例

**他のコンポーネントで使用**:
```typescript
import { Button } from '@/components/atoms/Button';

function MyPage() {
  const handleClick = () => {
    console.log('Button clicked!');
  };

  return (
    <div>
      <Button label="Click me" variant="primary" onClick={handleClick} />
    </div>
  );
}
```

### 6. Git コミット

**ファイルをコミット**:
```bash
git add src/components/atoms/Button/
git commit -m "feat: add Button component

- Add Button component with variants (primary, secondary, danger)
- Add size options (small, medium, large)
- Add Storybook stories
- Add unit tests with React Testing Library

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

## 完了条件

- コンポーネントファイルが作成されている
- Props 型定義が記述されている
- スタイルファイルが作成されている
- Storybook ストーリーが作成されている
- テストファイルが作成され、テストが通る
- Index ファイルでエクスポートされている
- Storybook で正しく表示される
- 他のコンポーネントで使用できる

## エスカレーション

- **命名規則の不一致**:
  - 「コンポーネント名がプロジェクトの命名規則に従っていません。PascalCase を使用してください。」

- **ディレクトリ構造の不一致**:
  - 「コンポーネントの配置場所が不適切です。プロジェクトのアーキテクチャパターンに従ってください。」

- **Props の型定義不足**:
  - 「Props の型定義が不十分です。すべての Props に型と JSDoc コメントを追加してください。」

- **テストが失敗する**:
  - 「テストが失敗しました。以下を確認してください：」
    - テストの import パスが正しいか
    - Testing Library のマッチャーが正しいか
    - Mock が必要な依存関係があるか

- **Storybook が表示されない**:
  - 「Storybook が表示されません。以下を確認してください：」
    - Storybook の設定ファイル
    - ストーリーファイルの import パス
    - コンポーネントのエクスポート

## ベストプラクティス

- **Props の型定義**: すべての Props に型と説明を追加
- **デフォルト値**: Props にデフォルト値を設定
- **アクセシビリティ**: ARIA 属性を適切に設定
- **テストカバレッジ**: Props のすべてのバリエーションをテスト
- **Storybook**: すべてのバリエーションのストーリーを作成
- **再利用性**: 汎用的で再利用可能なコンポーネントを設計
- **ドキュメント**: JSDoc コメントで使用方法を記載
- **命名**: 説明的で一貫性のある命名
- **スタイル**: CSS Modules または styled-components を使用
- **パフォーマンス**: React.memo でメモ化（必要に応じて）
