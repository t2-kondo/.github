---
name: server-actions
description: Guide for implementing Server Actions for data mutations. Use this when asked to create server actions, handle form submissions, or perform CRUD operations from the server.
---

# Server Actions ガイド

このプロジェクトでは Server Actions を使用してデータ変更処理を行います。

## 基本的なServer Action

### ファイル配置

```
src/actions/
├── users.ts      # ユーザー関連
├── posts.ts      # 投稿関連
└── tasks.ts      # タスク関連
```

### 基本構造

```typescript
// src/actions/tasks.ts
"use server";

import { revalidatePath } from "next/cache";
import { prisma } from "@/lib/prisma";
import { z } from "zod";

// バリデーションスキーマ
const createTaskSchema = z.object({
  title: z.string().min(1, "タイトルは必須です"),
  description: z.string().optional(),
  priority: z.enum(["low", "medium", "high"]).default("medium"),
  dueDate: z.string().optional(),
});

// タスク作成
export async function createTask(formData: FormData) {
  const data = {
    title: formData.get("title") as string,
    description: formData.get("description") as string,
    priority: (formData.get("priority") as string) || "medium",
    dueDate: formData.get("dueDate") as string,
  };

  const validated = createTaskSchema.parse(data);

  await prisma.task.create({
    data: {
      title: validated.title,
      description: validated.description || null,
      priority: validated.priority,
      dueDate: validated.dueDate ? new Date(validated.dueDate) : null,
    },
  });

  revalidatePath("/examples/projects/tasks");
}
```

## CRUD操作パターン

### Create（作成）

```typescript
export async function createTask(formData: FormData) {
  const title = formData.get("title") as string;
  
  await prisma.task.create({
    data: { title },
  });
  
  revalidatePath("/tasks");
}
```

### Read（取得）

```typescript
// Server Componentから直接Prismaを使用
export default async function TasksPage() {
  const tasks = await prisma.task.findMany({
    orderBy: { createdAt: "desc" },
  });

  return <TaskList tasks={tasks} />;
}
```

### Update（更新）

```typescript
export async function toggleTaskComplete(id: string) {
  const task = await prisma.task.findUnique({ where: { id } });
  if (!task) return;

  await prisma.task.update({
    where: { id },
    data: { completed: !task.completed },
  });

  revalidatePath("/tasks");
}
```

### Delete（削除）

```typescript
export async function deleteTask(id: string) {
  await prisma.task.delete({ where: { id } });
  revalidatePath("/tasks");
}
```

## フォームとの連携

### React Hook Form + useTransition

```tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { useTransition } from "react";
import { createTask } from "@/actions/tasks";
import { createTaskSchema, type CreateTaskInput } from "@/schemas/task";

export function TaskForm() {
  const [isPending, startTransition] = useTransition();

  const { register, handleSubmit, reset, formState: { errors } } = useForm<CreateTaskInput>({
    resolver: zodResolver(createTaskSchema),
  });

  async function onSubmit(data: CreateTaskInput) {
    startTransition(async () => {
      const formData = new FormData();
      formData.set("title", data.title);
      formData.set("description", data.description || "");
      formData.set("priority", data.priority);

      await createTask(formData);
      reset();
    });
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("title")} />
      {errors.title && <p>{errors.title.message}</p>}
      <button type="submit" disabled={isPending}>
        {isPending ? "追加中..." : "追加"}
      </button>
    </form>
  );
}
```

### 直接呼び出し（ボタンクリック）

```tsx
"use client";

import { useTransition } from "react";
import { toggleTaskComplete, deleteTask } from "@/actions/tasks";

export function TaskItem({ task }: { task: Task }) {
  const [isPending, startTransition] = useTransition();

  function handleToggle() {
    startTransition(() => {
      toggleTaskComplete(task.id);
    });
  }

  function handleDelete() {
    if (confirm("削除しますか？")) {
      startTransition(() => {
        deleteTask(task.id);
      });
    }
  }

  return (
    <div className={isPending ? "opacity-50" : ""}>
      <input
        type="checkbox"
        checked={task.completed}
        onChange={handleToggle}
        disabled={isPending}
      />
      <span>{task.title}</span>
      <button onClick={handleDelete} disabled={isPending}>
        削除
      </button>
    </div>
  );
}
```

## エラーハンドリング

### try-catch パターン

```typescript
export async function createTask(formData: FormData) {
  try {
    const data = createTaskSchema.parse({
      title: formData.get("title"),
    });

    await prisma.task.create({ data });
    revalidatePath("/tasks");
    return { success: true };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, error: error.errors[0].message };
    }
    return { success: false, error: "エラーが発生しました" };
  }
}
```

## 重要なルール

1. **"use server"**: ファイルの先頭に必ず記述
2. **revalidatePath**: データ変更後にキャッシュを無効化
3. **バリデーション**: Zodでサーバーサイドでも必ず検証
4. **useTransition**: ペンディング状態をUIに反映
5. **ファイル配置**: `src/actions/` に配置

## ファイル配置

```
src/
├── actions/           # Server Actions
│   ├── tasks.ts
│   └── users.ts
├── schemas/           # Zodスキーマ（共有）
│   ├── task.ts
│   └── user.ts
└── app/
    └── examples/
        └── projects/
            └── tasks/
                ├── page.tsx       # Server Component
                └── task-form.tsx  # Client Component
```
