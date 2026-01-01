---
name: prisma-database
description: Guide for database operations using Prisma ORM. Use this when asked to create models, run migrations, or perform CRUD operations on the database.
---

# Prisma データベースガイド

このプロジェクトでは Prisma + SQLite を使用してデータベース操作を行います。

## 現在のスキーマ

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated/prisma"  // Prisma 7: 出力先を明示指定
}

datasource db {
  provider = "sqlite"
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  password  String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  posts     Post[]
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
}

model Task {
  id          String    @id @default(cuid())
  title       String
  description String?
  completed   Boolean   @default(false)
  priority    String    @default("medium") // low, medium, high
  dueDate     DateTime?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
}
```

## Prismaクライアント

```typescript
// src/lib/prisma.ts
import { PrismaClient } from "@/generated/prisma";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

## 型のインポート

```typescript
// モデルの型をインポート
import { Task, User, Post } from "@/generated/prisma";

// コンポーネントで使用
interface TaskListProps {
  tasks: Task[];
}
```

## CRUD操作

### Create（作成）

```typescript
// ユーザー作成
const user = await prisma.user.create({
  data: {
    email: "user@example.com",
    name: "田中太郎",
  },
});

// 投稿作成（リレーション付き）
const post = await prisma.post.create({
  data: {
    title: "最初の投稿",
    content: "こんにちは！",
    author: {
      connect: { id: userId },
    },
  },
});
```

### Read（取得）

```typescript
// 全件取得
const users = await prisma.user.findMany();

// 条件付き取得
const publishedPosts = await prisma.post.findMany({
  where: { published: true },
  orderBy: { createdAt: "desc" },
});

// 1件取得
const user = await prisma.user.findUnique({
  where: { email: "user@example.com" },
});

// リレーション込みで取得
const userWithPosts = await prisma.user.findUnique({
  where: { id: userId },
  include: { posts: true },
});

// 特定フィールドのみ取得
const userNames = await prisma.user.findMany({
  select: { id: true, name: true },
});
```

### Update（更新）

```typescript
// 1件更新
const updatedUser = await prisma.user.update({
  where: { id: userId },
  data: { name: "新しい名前" },
});

// 複数件更新
await prisma.post.updateMany({
  where: { authorId: userId },
  data: { published: true },
});
```

### Delete（削除）

```typescript
// 1件削除
await prisma.user.delete({
  where: { id: userId },
});

// 複数件削除
await prisma.post.deleteMany({
  where: { published: false },
});
```

## APIルートでの使用例

```typescript
// src/app/api/posts/route.ts
import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

// GET: 投稿一覧
export async function GET() {
  const posts = await prisma.post.findMany({
    include: {
      author: {
        select: { id: true, name: true },
      },
    },
    orderBy: { createdAt: "desc" },
  });

  return NextResponse.json(posts);
}

// POST: 投稿作成
export async function POST(request: Request) {
  const body = await request.json();

  const post = await prisma.post.create({
    data: {
      title: body.title,
      content: body.content,
      authorId: body.authorId,
    },
  });

  return NextResponse.json(post, { status: 201 });
}
```

## Server Actionsでの使用例

```typescript
// src/actions/post.ts
"use server";

import { prisma } from "@/lib/prisma";
import { revalidatePath } from "next/cache";

export async function createPost(formData: FormData) {
  const title = formData.get("title") as string;
  const content = formData.get("content") as string;
  const authorId = formData.get("authorId") as string;

  await prisma.post.create({
    data: { title, content, authorId },
  });

  revalidatePath("/posts");
}

export async function deletePost(postId: string) {
  await prisma.post.delete({
    where: { id: postId },
  });

  revalidatePath("/posts");
}
```

## マイグレーション

### 新しいモデル追加時

1. `prisma/schema.prisma` を編集
2. マイグレーション実行：

```bash
npm run db:migrate
# または
npx prisma migrate dev --name add_new_model
```

### スキーマを直接反映（開発用）

```bash
npm run db:push
```

### Prisma Studio（GUI）

```bash
npm run db:studio
```

## 重要なルール

1. **クライアント**: `src/lib/prisma.ts` からインポート
2. **型安全**: Prismaが自動生成する型を活用
3. **N+1対策**: `include` または `select` でリレーションを取得
4. **トランザクション**: `prisma.$transaction()` で複数操作を実行
5. **マイグレーション**: スキーマ変更後は必ず `migrate dev` を実行

## ファイル配置

```
prisma/
├── schema.prisma          # スキーマ定義
├── migrations/            # マイグレーション履歴
└── dev.db                 # SQLiteデータベース

src/
├── lib/
│   └── prisma.ts          # Prismaクライアント
├── generated/
│   └── prisma/            # 生成された型
└── actions/
    └── post.ts            # Server Actions
```
