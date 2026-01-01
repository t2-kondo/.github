---
name: nextjs-api-routes
description: Guide for creating API endpoints using Next.js Route Handlers. Use this when asked to create APIs, handle HTTP requests, or implement backend logic.
---

# Next.js API Routes ガイド

このプロジェクトでは Next.js の Route Handlers を使用してAPIを作成します。

## 基本的なAPIルート

### ファイル配置

```
src/app/api/
├── hello/
│   └── route.ts          → GET /api/hello
├── users/
│   ├── route.ts          → /api/users
│   └── [id]/
│       └── route.ts      → /api/users/:id
└── posts/
    └── route.ts          → /api/posts
```

### GET リクエスト

```typescript
// src/app/api/users/route.ts
import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

export async function GET() {
  const users = await prisma.user.findMany();

  return NextResponse.json({
    users,
    total: users.length,
  });
}
```

### POST リクエスト

```typescript
// src/app/api/users/route.ts
import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { z } from "zod";

const createUserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});

export async function POST(request: Request) {
  try {
    const body = await request.json();

    // バリデーション
    const validated = createUserSchema.parse(body);

    const user = await prisma.user.create({
      data: validated,
    });

    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: "バリデーションエラー", details: error.errors },
        { status: 400 }
      );
    }
    return NextResponse.json(
      { error: "サーバーエラー" },
      { status: 500 }
    );
  }
}
```

### 動的ルート（ID指定）

```typescript
// src/app/api/users/[id]/route.ts
import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

interface Props {
  params: Promise<{ id: string }>;
}

// GET /api/users/:id
export async function GET(request: Request, { params }: Props) {
  const { id } = await params;

  const user = await prisma.user.findUnique({
    where: { id },
  });

  if (!user) {
    return NextResponse.json(
      { error: "ユーザーが見つかりません" },
      { status: 404 }
    );
  }

  return NextResponse.json(user);
}

// PUT /api/users/:id
export async function PUT(request: Request, { params }: Props) {
  const { id } = await params;
  const body = await request.json();

  const user = await prisma.user.update({
    where: { id },
    data: body,
  });

  return NextResponse.json(user);
}

// DELETE /api/users/:id
export async function DELETE(request: Request, { params }: Props) {
  const { id } = await params;

  await prisma.user.delete({
    where: { id },
  });

  return new NextResponse(null, { status: 204 });
}
```

## クエリパラメータの取得

```typescript
// GET /api/users?page=1&limit=10
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get("page") || "1");
  const limit = parseInt(searchParams.get("limit") || "10");

  const users = await prisma.user.findMany({
    skip: (page - 1) * limit,
    take: limit,
  });

  return NextResponse.json({
    users,
    page,
    limit,
  });
}
```

## ヘッダーとCookie

```typescript
import { NextResponse } from "next/server";
import { cookies, headers } from "next/headers";

export async function GET() {
  // ヘッダーの取得
  const headersList = await headers();
  const authHeader = headersList.get("authorization");

  // Cookieの取得
  const cookieStore = await cookies();
  const token = cookieStore.get("token");

  // レスポンスヘッダーの設定
  return NextResponse.json(
    { message: "成功" },
    {
      headers: {
        "X-Custom-Header": "value",
      },
    }
  );
}

// Cookieの設定
export async function POST() {
  const response = NextResponse.json({ success: true });

  response.cookies.set("token", "abc123", {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "strict",
    maxAge: 60 * 60 * 24 * 7, // 1週間
  });

  return response;
}
```

## エラーハンドリング

```typescript
// src/lib/api-error.ts
export class ApiError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500
  ) {
    super(message);
  }
}

// src/app/api/users/route.ts
import { ApiError } from "@/lib/api-error";

export async function GET() {
  try {
    const users = await prisma.user.findMany();
    return NextResponse.json(users);
  } catch (error) {
    if (error instanceof ApiError) {
      return NextResponse.json(
        { error: error.message },
        { status: error.statusCode }
      );
    }
    console.error(error);
    return NextResponse.json(
      { error: "Internal Server Error" },
      { status: 500 }
    );
  }
}
```

## 認証チェック

```typescript
import { auth } from "@/auth";
import { NextResponse } from "next/server";

export async function GET() {
  const session = await auth();

  if (!session) {
    return NextResponse.json(
      { error: "認証が必要です" },
      { status: 401 }
    );
  }

  // 認証済みユーザーの処理
  return NextResponse.json({
    user: session.user,
  });
}
```

## レスポンスの種類

```typescript
// JSON
return NextResponse.json({ data: "value" });

// ステータスコード付き
return NextResponse.json({ error: "Not Found" }, { status: 404 });

// リダイレクト
return NextResponse.redirect(new URL("/login", request.url));

// 空レスポンス
return new NextResponse(null, { status: 204 });

// テキスト
return new NextResponse("Hello", {
  headers: { "Content-Type": "text/plain" },
});
```

## 重要なルール

1. **HTTPメソッド**: `GET`, `POST`, `PUT`, `PATCH`, `DELETE` を関数としてエクスポート
2. **バリデーション**: Zodでリクエストボディを検証
3. **エラーハンドリング**: try-catchで適切なステータスコードを返す
4. **認証**: 必要に応じて `auth()` でセッションを確認
5. **型安全**: パラメータとレスポンスに型を定義

## ファイル配置

```
src/app/api/
├── auth/
│   └── [...nextauth]/
│       └── route.ts      # NextAuth.js
├── users/
│   ├── route.ts          # GET/POST /api/users
│   └── [id]/
│       └── route.ts      # GET/PUT/DELETE /api/users/:id
└── posts/
    └── route.ts          # /api/posts
```
