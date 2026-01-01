---
name: nextjs-app-router
description: Guide for creating pages and layouts using Next.js App Router. Use this when asked to create new pages, layouts, or implement routing.
---

# Next.js App Router ガイド

このプロジェクトでは Next.js 16 の App Router を使用してルーティングを実装します。

## ファイルベースルーティング

### ディレクトリ構造とURL

```
src/app/
├── page.tsx              → /
├── about/
│   └── page.tsx          → /about
├── blog/
│   ├── page.tsx          → /blog
│   └── [slug]/
│       └── page.tsx      → /blog/:slug
├── (auth)/
│   ├── login/
│   │   └── page.tsx      → /login
│   └── register/
│       └── page.tsx      → /register
└── api/
    └── users/
        └── route.ts      → /api/users
```

## 特殊ファイル

| ファイル | 用途 |
|---------|------|
| `page.tsx` | ページコンポーネント |
| `layout.tsx` | 共有レイアウト |
| `loading.tsx` | ローディングUI |
| `error.tsx` | エラーUI |
| `not-found.tsx` | 404ページ |
| `route.ts` | APIルート |

## ページの作成

### 基本的なページ

```tsx
// src/app/about/page.tsx
export default function AboutPage() {
  return (
    <main>
      <h1>About</h1>
      <p>このサイトについて</p>
    </main>
  );
}
```

### メタデータ付きページ

```tsx
// src/app/about/page.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "About | My Site",
  description: "このサイトについての説明",
};

export default function AboutPage() {
  return <main>...</main>;
}
```

### 動的ページ

```tsx
// src/app/blog/[slug]/page.tsx
interface Props {
  params: Promise<{ slug: string }>;
}

export default async function BlogPostPage({ params }: Props) {
  const { slug } = await params;

  return (
    <article>
      <h1>記事: {slug}</h1>
    </article>
  );
}

// 静的生成するパスを指定
export async function generateStaticParams() {
  const posts = await fetchPosts();
  return posts.map((post) => ({
    slug: post.slug,
  }));
}
```

## レイアウト

### ルートレイアウト

```tsx
// src/app/layout.tsx
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "My Site",
  description: "サイトの説明",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ja">
      <body>{children}</body>
    </html>
  );
}
```

### ネストされたレイアウト

```tsx
// src/app/dashboard/layout.tsx
import { Sidebar } from "@/components/sidebar";

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex">
      <Sidebar />
      <main className="flex-1">{children}</main>
    </div>
  );
}
```

## ローディングとエラー

### ローディング

```tsx
// src/app/blog/loading.tsx
export default function Loading() {
  return (
    <div className="flex justify-center items-center min-h-screen">
      <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-gray-900" />
    </div>
  );
}
```

### エラーハンドリング

```tsx
// src/app/blog/error.tsx
"use client";

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div className="text-center py-10">
      <h2>エラーが発生しました</h2>
      <p>{error.message}</p>
      <button onClick={reset}>再試行</button>
    </div>
  );
}
```

### Not Found

```tsx
// src/app/not-found.tsx
import Link from "next/link";

export default function NotFound() {
  return (
    <div className="text-center py-10">
      <h2>ページが見つかりません</h2>
      <Link href="/">ホームに戻る</Link>
    </div>
  );
}
```

## ナビゲーション

### Linkコンポーネント

```tsx
import Link from "next/link";

export function Navigation() {
  return (
    <nav>
      <Link href="/">ホーム</Link>
      <Link href="/about">About</Link>
      <Link href="/blog">ブログ</Link>
    </nav>
  );
}
```

### プログラマティックナビゲーション

```tsx
"use client";

import { useRouter } from "next/navigation";

export function LoginButton() {
  const router = useRouter();

  const handleLogin = async () => {
    // ログイン処理
    router.push("/dashboard");
    // または
    router.replace("/dashboard"); // 履歴を置き換え
  };

  return <button onClick={handleLogin}>ログイン</button>;
}
```

## ルートグループ

括弧 `()` でフォルダ名を囲むと、URLに影響しないグループを作成できます：

```
src/app/
├── (marketing)/
│   ├── layout.tsx        # マーケティング用レイアウト
│   ├── page.tsx          → /
│   └── about/
│       └── page.tsx      → /about
└── (dashboard)/
    ├── layout.tsx        # ダッシュボード用レイアウト
    └── dashboard/
        └── page.tsx      → /dashboard
```

## 重要なルール

1. **Server Components**: デフォルトでServer Component（データフェッチ可能）
2. **Client Components**: インタラクティブな機能には `"use client"` を追加
3. **メタデータ**: `metadata` オブジェクトまたは `generateMetadata` 関数で設定
4. **動的ルート**: `[param]` で動的セグメント、`[...slug]` でキャッチオール
5. **並列ルート**: `@folder` で並列レンダリング

## ファイル配置

```
src/app/
├── layout.tsx            # ルートレイアウト
├── page.tsx              # ホームページ
├── globals.css           # グローバルスタイル
├── not-found.tsx         # 404ページ
├── error.tsx             # エラーページ
├── loading.tsx           # ローディング
├── (routes)/             # ルートグループ
│   └── ...
└── api/                  # APIルート
    └── ...
```
