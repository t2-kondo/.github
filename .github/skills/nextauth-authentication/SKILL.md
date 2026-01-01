---
name: nextauth-authentication
description: Guide for implementing authentication using NextAuth.js (Auth.js v5). Use this when asked to implement login, logout, session management, or protected routes.
---

# NextAuth.js 認証ガイド

このプロジェクトでは NextAuth.js (Auth.js v5) を使用して認証を実装します。

## 現在の設定

認証設定は `src/auth.ts` にあります。

### デモ用ログイン情報

- **Email**: `demo@example.com`
- **Password**: `password`

## 基本的な使い方

### サーバーサイドでセッション取得

```tsx
// Server Component
import { auth } from "@/auth";

export default async function Page() {
  const session = await auth();

  if (!session) {
    return <p>ログインしてください</p>;
  }

  return <p>こんにちは、{session.user?.name}さん</p>;
}
```

### クライアントサイドでセッション取得

```tsx
"use client";

import { useSession } from "next-auth/react";

export function UserInfo() {
  const { data: session, status } = useSession();

  if (status === "loading") return <p>読み込み中...</p>;
  if (status === "unauthenticated") return <p>未ログイン</p>;

  return <p>こんにちは、{session?.user?.name}さん</p>;
}
```

### SessionProvider の設定

```tsx
// src/providers/session-provider.tsx
"use client";

import { SessionProvider } from "next-auth/react";

export function AuthProvider({ children }: { children: React.ReactNode }) {
  return <SessionProvider>{children}</SessionProvider>;
}

// src/app/layout.tsx
import { AuthProvider } from "@/providers/session-provider";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <AuthProvider>{children}</AuthProvider>
      </body>
    </html>
  );
}
```

## ログイン・ログアウト

### サインインボタン

```tsx
"use client";

import { signIn, signOut, useSession } from "next-auth/react";
import { Button } from "@/components/ui/button";

export function AuthButton() {
  const { data: session } = useSession();

  if (session) {
    return (
      <div className="flex items-center gap-4">
        <span>{session.user?.name}</span>
        <Button onClick={() => signOut()}>ログアウト</Button>
      </div>
    );
  }

  return <Button onClick={() => signIn()}>ログイン</Button>;
}
```

### カスタムログインページ

```tsx
// src/app/auth/signin/page.tsx
"use client";

import { signIn } from "next-auth/react";
import { useState } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";

export default function SignInPage() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const result = await signIn("credentials", {
      email,
      password,
      redirect: false,
    });

    if (result?.error) {
      setError("ログインに失敗しました");
    } else {
      window.location.href = "/";
    }
  };

  return (
    <div className="flex min-h-screen items-center justify-center">
      <Card className="w-96">
        <CardHeader>
          <CardTitle>ログイン</CardTitle>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            <div>
              <Label htmlFor="email">メールアドレス</Label>
              <Input
                id="email"
                type="email"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                required
              />
            </div>
            <div>
              <Label htmlFor="password">パスワード</Label>
              <Input
                id="password"
                type="password"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                required
              />
            </div>
            {error && <p className="text-red-500 text-sm">{error}</p>}
            <Button type="submit" className="w-full">
              ログイン
            </Button>
          </form>
        </CardContent>
      </Card>
    </div>
  );
}
```

## 保護されたルート

### ミドルウェアで保護

```typescript
// src/middleware.ts
import { auth } from "@/auth";
import { NextResponse } from "next/server";

export default auth((req) => {
  const isLoggedIn = !!req.auth;
  const isProtectedRoute = req.nextUrl.pathname.startsWith("/dashboard");

  if (isProtectedRoute && !isLoggedIn) {
    return NextResponse.redirect(new URL("/auth/signin", req.url));
  }
});

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

### Server Componentで保護

```tsx
import { auth } from "@/auth";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const session = await auth();

  if (!session) {
    redirect("/auth/signin");
  }

  return <div>ダッシュボード</div>;
}
```

## ソーシャルログイン追加

```typescript
// src/auth.ts
import NextAuth from "next-auth";
import GitHub from "next-auth/providers/github";
import Google from "next-auth/providers/google";

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [
    GitHub({
      clientId: process.env.GITHUB_ID,
      clientSecret: process.env.GITHUB_SECRET,
    }),
    Google({
      clientId: process.env.GOOGLE_ID,
      clientSecret: process.env.GOOGLE_SECRET,
    }),
  ],
});
```

## 重要なルール

1. **Session取得**: サーバー側は `auth()`、クライアント側は `useSession()`
2. **保護**: ミドルウェアまたはServer Componentでリダイレクト
3. **環境変数**: `AUTH_SECRET` は必須（自動生成済み）
4. **Provider**: ソーシャルログインは環境変数でクライアントID/シークレットを設定

## ファイル配置

```
src/
├── auth.ts                         # NextAuth設定
├── middleware.ts                   # ルート保護
├── app/
│   ├── api/auth/[...nextauth]/
│   │   └── route.ts               # APIルート
│   └── auth/
│       └── signin/
│           └── page.tsx           # ログインページ
└── providers/
    └── session-provider.tsx       # SessionProvider
```
