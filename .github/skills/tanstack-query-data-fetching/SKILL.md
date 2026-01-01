---
name: tanstack-query-data-fetching
description: Guide for fetching and caching server data using TanStack Query. Use this when asked to fetch data from APIs, handle loading states, or implement data caching.
---

# TanStack Query データフェッチングガイド

このプロジェクトでは TanStack Query を使用してサーバーデータの取得・キャッシュを行います。

## セットアップ

### QueryClientProvider の設定

```tsx
// src/providers/query-provider.tsx
"use client";

import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { useState } from "react";

export function QueryProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1分間はキャッシュを新鮮とみなす
            refetchOnWindowFocus: false,
          },
        },
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

### layout.tsx での設定

```tsx
// src/app/layout.tsx
import { QueryProvider } from "@/providers/query-provider";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <QueryProvider>{children}</QueryProvider>
      </body>
    </html>
  );
}
```

## 基本的な使い方

### データの取得（useQuery）

```tsx
"use client";

import { useQuery } from "@tanstack/react-query";

interface User {
  id: string;
  name: string;
  email: string;
}

async function fetchUsers(): Promise<User[]> {
  const res = await fetch("/api/users");
  if (!res.ok) throw new Error("ユーザーの取得に失敗しました");
  return res.json();
}

export function UserList() {
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers,
  });

  if (isLoading) return <p>読み込み中...</p>;
  if (isError) return <p>エラー: {error.message}</p>;

  return (
    <ul>
      {data?.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### データの変更（useMutation）

```tsx
"use client";

import { useMutation, useQueryClient } from "@tanstack/react-query";

interface CreateUserInput {
  name: string;
  email: string;
}

async function createUser(input: CreateUserInput) {
  const res = await fetch("/api/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(input),
  });
  if (!res.ok) throw new Error("ユーザーの作成に失敗しました");
  return res.json();
}

export function CreateUserForm() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: createUser,
    onSuccess: () => {
      // キャッシュを無効化して再取得
      queryClient.invalidateQueries({ queryKey: ["users"] });
    },
  });

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    mutation.mutate({
      name: formData.get("name") as string,
      email: formData.get("email") as string,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" placeholder="名前" required />
      <input name="email" type="email" placeholder="メール" required />
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? "作成中..." : "ユーザーを作成"}
      </button>
      {mutation.isError && <p>エラー: {mutation.error.message}</p>}
    </form>
  );
}
```

## 高度なパターン

### パラメータ付きクエリ

```tsx
const { data } = useQuery({
  queryKey: ["user", userId], // ユニークなキー
  queryFn: () => fetchUser(userId),
  enabled: !!userId, // userIdがある時のみ実行
});
```

### 楽観的更新

```tsx
const mutation = useMutation({
  mutationFn: updateUser,
  onMutate: async (newUser) => {
    await queryClient.cancelQueries({ queryKey: ["users"] });
    const previousUsers = queryClient.getQueryData(["users"]);
    queryClient.setQueryData(["users"], (old) => [...old, newUser]);
    return { previousUsers };
  },
  onError: (err, newUser, context) => {
    queryClient.setQueryData(["users"], context.previousUsers);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ["users"] });
  },
});
```

## 重要なルール

1. **queryKey**: 配列形式で一意のキーを設定
2. **Client Component**: `"use client"` が必要
3. **エラーハンドリング**: `isError` と `error` で処理
4. **キャッシュ無効化**: `invalidateQueries` で更新後に再取得
5. **staleTime**: 適切なキャッシュ時間を設定

## ファイル配置

```
src/
├── providers/
│   └── query-provider.tsx
└── hooks/
    └── use-users.ts  # カスタムフック
```
