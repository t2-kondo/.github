---
name: zustand-state-management
description: Guide for managing client-side state using Zustand. Use this when asked to create stores, manage global state, or implement state management.
---

# Zustand 状態管理ガイド

このプロジェクトでは Zustand を使用してクライアント側の状態管理を行います。

## ストアの作成

### 基本的なストア

```typescript
// src/stores/counter-store.ts
import { create } from "zustand";

interface CounterState {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

export const useCounterStore = create<CounterState>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));
```

### コンポーネントでの使用

```tsx
"use client";

import { useCounterStore } from "@/stores/counter-store";

export function Counter() {
  const { count, increment, decrement, reset } = useCounterStore();

  return (
    <div>
      <p>カウント: {count}</p>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
      <button onClick={reset}>リセット</button>
    </div>
  );
}
```

## 高度なパターン

### 非同期アクション

```typescript
// src/stores/user-store.ts
import { create } from "zustand";

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  user: User | null;
  isLoading: boolean;
  error: string | null;
  fetchUser: (id: string) => Promise<void>;
  clearUser: () => void;
}

export const useUserStore = create<UserState>((set) => ({
  user: null,
  isLoading: false,
  error: null,
  fetchUser: async (id: string) => {
    set({ isLoading: true, error: null });
    try {
      const res = await fetch(`/api/users/${id}`);
      const user = await res.json();
      set({ user, isLoading: false });
    } catch (error) {
      set({ error: "ユーザーの取得に失敗しました", isLoading: false });
    }
  },
  clearUser: () => set({ user: null, error: null }),
}));
```

### フィルター状態管理（実践例）

```typescript
// src/stores/task-filter-store.ts
import { create } from "zustand";

type FilterStatus = "all" | "active" | "completed";
type FilterPriority = "all" | "low" | "medium" | "high";

interface TaskFilterState {
  status: FilterStatus;
  priority: FilterPriority;
  setStatus: (status: FilterStatus) => void;
  setPriority: (priority: FilterPriority) => void;
  reset: () => void;
}

export const useTaskFilterStore = create<TaskFilterState>((set) => ({
  status: "all",
  priority: "all",
  setStatus: (status) => set({ status }),
  setPriority: (priority) => set({ priority }),
  reset: () => set({ status: "all", priority: "all" }),
}));
```

### 永続化（localStorage）

```typescript
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface ThemeState {
  theme: "light" | "dark";
  toggleTheme: () => void;
}

export const useThemeStore = create<ThemeState>()(
  persist(
    (set) => ({
      theme: "light",
      toggleTheme: () =>
        set((state) => ({
          theme: state.theme === "light" ? "dark" : "light",
        })),
    }),
    {
      name: "theme-storage", // localStorage のキー
    }
  )
);
```

### セレクターによる最適化

```tsx
// 必要な値だけを選択して再レンダリングを最適化
const count = useCounterStore((state) => state.count);
const increment = useCounterStore((state) => state.increment);
```

## ファイル配置

ストアは `src/stores/` ディレクトリに配置します：

```
src/
└── stores/
    ├── counter-store.ts
    ├── user-store.ts
    └── theme-store.ts
```

## 重要なルール

1. **命名規則**: `use[Name]Store` の形式で命名
2. **型定義**: インターフェースで状態と アクションを定義
3. **Client Component**: ストアを使用するコンポーネントには `"use client"` を追加
4. **セレクター**: 必要な値だけを選択してパフォーマンスを最適化
5. **永続化**: `persist` ミドルウェアでlocalStorageに保存可能

## Next.jsでの注意点

Server Componentsでは直接使用できません。Client Componentでラップしてください：

```tsx
// Server Component
import { ClientCounter } from "@/components/client-counter";

export default function Page() {
  return <ClientCounter />;
}

// Client Component
"use client";
export function ClientCounter() {
  const count = useCounterStore((state) => state.count);
  return <p>{count}</p>;
}
```
