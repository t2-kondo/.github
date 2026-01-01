---
name: vitest-testing
description: Guide for writing tests using Vitest and Testing Library. Use this when asked to write unit tests, component tests, or run the test suite.
---

# Vitest テストガイド

このプロジェクトでは Vitest + Testing Library を使用してテストを作成します。

## テストの実行

```bash
# ウォッチモードで実行
npm run test

# 1回だけ実行
npm run test -- --run

# UIモードで実行
npm run test:ui

# カバレッジレポート
npm run test:coverage
```

## 基本的なテスト

### ユニットテスト

```typescript
// src/__tests__/utils.test.ts
import { describe, it, expect } from "vitest";
import { formatDate, calculateTotal } from "@/lib/utils";

describe("formatDate", () => {
  it("日付を正しくフォーマットすること", () => {
    const date = new Date("2025-01-15");
    expect(formatDate(date)).toBe("2025年1月15日");
  });

  it("無効な日付でエラーをスローすること", () => {
    expect(() => formatDate(null)).toThrow();
  });
});

describe("calculateTotal", () => {
  it("合計を正しく計算すること", () => {
    const items = [
      { price: 100, quantity: 2 },
      { price: 200, quantity: 1 },
    ];
    expect(calculateTotal(items)).toBe(400);
  });

  it("空配列で0を返すこと", () => {
    expect(calculateTotal([])).toBe(0);
  });
});
```

### コンポーネントテスト

```tsx
// src/__tests__/components/button.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { describe, it, expect, vi } from "vitest";
import { Button } from "@/components/ui/button";

describe("Button", () => {
  it("テキストを正しく表示すること", () => {
    render(<Button>クリック</Button>);
    expect(screen.getByRole("button", { name: "クリック" })).toBeInTheDocument();
  });

  it("クリック時にハンドラが呼ばれること", () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>クリック</Button>);

    fireEvent.click(screen.getByRole("button"));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it("disabled時にクリックできないこと", () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick} disabled>クリック</Button>);

    fireEvent.click(screen.getByRole("button"));
    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

### フォームテスト

```tsx
// src/__tests__/components/user-form.test.tsx
import { render, screen, fireEvent, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, it, expect, vi } from "vitest";
import { UserForm } from "@/components/forms/user-form";

describe("UserForm", () => {
  it("バリデーションエラーを表示すること", async () => {
    render(<UserForm />);
    const user = userEvent.setup();

    // 空のまま送信
    await user.click(screen.getByRole("button", { name: "登録" }));

    await waitFor(() => {
      expect(screen.getByText("名前は必須です")).toBeInTheDocument();
    });
  });

  it("有効な入力で送信できること", async () => {
    const onSubmit = vi.fn();
    render(<UserForm onSubmit={onSubmit} />);
    const user = userEvent.setup();

    await user.type(screen.getByLabelText("名前"), "田中太郎");
    await user.type(screen.getByLabelText("メール"), "tanaka@example.com");
    await user.type(screen.getByLabelText("パスワード"), "Password123");
    await user.click(screen.getByRole("button", { name: "登録" }));

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        name: "田中太郎",
        email: "tanaka@example.com",
        password: "Password123",
      });
    });
  });
});
```

## モック

### 関数のモック

```typescript
import { vi } from "vitest";

// 関数をモック
const mockFn = vi.fn();
mockFn.mockReturnValue("返り値");
mockFn.mockResolvedValue("非同期の返り値");

// 呼び出しを検証
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith("引数");
expect(mockFn).toHaveBeenCalledTimes(1);
```

### モジュールのモック

```typescript
import { vi } from "vitest";

// fetchをモック
vi.mock("@/lib/api", () => ({
  fetchUsers: vi.fn().mockResolvedValue([
    { id: "1", name: "テストユーザー" },
  ]),
}));
```

### Next.js機能のモック

```typescript
import { vi } from "vitest";

// next/navigationをモック
vi.mock("next/navigation", () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    back: vi.fn(),
  }),
  usePathname: () => "/",
  useSearchParams: () => new URLSearchParams(),
}));
```

## Testing Libraryのクエリ

| クエリ | 用途 |
|-------|------|
| `getByRole` | アクセシブルな要素を取得（推奨） |
| `getByLabelText` | ラベルに関連付けられた要素 |
| `getByPlaceholderText` | プレースホルダーで取得 |
| `getByText` | テキスト内容で取得 |
| `getByTestId` | data-testidで取得（最後の手段） |

### 非同期クエリ

```tsx
// 要素が表示されるまで待つ
await waitFor(() => {
  expect(screen.getByText("読み込み完了")).toBeInTheDocument();
});

// findByを使用（waitFor + getByの組み合わせ）
const element = await screen.findByText("読み込み完了");
```

## セットアップとクリーンアップ

```typescript
import { beforeEach, afterEach, vi } from "vitest";

describe("テストスイート", () => {
  beforeEach(() => {
    // 各テストの前に実行
    vi.clearAllMocks();
  });

  afterEach(() => {
    // 各テストの後に実行
  });
});
```

## 重要なルール

1. **ファイル命名**: `*.test.ts` または `*.test.tsx`
2. **配置**: `src/__tests__/` またはソースファイルの隣
3. **クエリ優先順位**: `getByRole` > `getByLabelText` > `getByText` > `getByTestId`
4. **非同期処理**: `waitFor` または `findBy` を使用
5. **ユーザー操作**: `userEvent` を優先（より現実的）

## ファイル配置

```
src/
└── __tests__/
    ├── setup.ts            # テストセットアップ
    ├── sample.test.ts      # サンプルテスト
    ├── utils.test.ts       # ユーティリティテスト
    └── components/
        ├── button.test.tsx
        └── user-form.test.tsx
```
