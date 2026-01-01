---
name: react-hook-form-zod-validation
description: Guide for creating forms with validation using React Hook Form and Zod. Use this when asked to create forms, implement validation, or handle form submissions.
---

# React Hook Form + Zod フォームバリデーションガイド

このプロジェクトでは React Hook Form と Zod を組み合わせてフォームを作成します。

## 基本的なフォーム

### Zodスキーマの定義

```typescript
// src/schemas/user.ts
import { z } from "zod";

export const userSchema = z.object({
  name: z
    .string()
    .min(1, "名前は必須です")
    .max(50, "名前は50文字以内で入力してください"),
  email: z
    .string()
    .min(1, "メールアドレスは必須です")
    .email("有効なメールアドレスを入力してください"),
  age: z
    .number()
    .min(0, "年齢は0以上で入力してください")
    .max(150, "有効な年齢を入力してください")
    .optional(),
  password: z
    .string()
    .min(8, "パスワードは8文字以上で入力してください")
    .regex(/[A-Z]/, "大文字を1文字以上含めてください")
    .regex(/[0-9]/, "数字を1文字以上含めてください"),
});

export type UserFormData = z.infer<typeof userSchema>;
```

### フォームコンポーネント

```tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { userSchema, type UserFormData } from "@/schemas/user";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

export function UserForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset,
  } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: {
      name: "",
      email: "",
      password: "",
    },
  });

  const onSubmit = async (data: UserFormData) => {
    try {
      const res = await fetch("/api/users", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data),
      });
      if (!res.ok) throw new Error("送信に失敗しました");
      reset();
      alert("登録が完了しました");
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <Label htmlFor="name">名前</Label>
        <Input id="name" {...register("name")} />
        {errors.name && (
          <p className="text-red-500 text-sm">{errors.name.message}</p>
        )}
      </div>

      <div>
        <Label htmlFor="email">メールアドレス</Label>
        <Input id="email" type="email" {...register("email")} />
        {errors.email && (
          <p className="text-red-500 text-sm">{errors.email.message}</p>
        )}
      </div>

      <div>
        <Label htmlFor="password">パスワード</Label>
        <Input id="password" type="password" {...register("password")} />
        {errors.password && (
          <p className="text-red-500 text-sm">{errors.password.message}</p>
        )}
      </div>

      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "送信中..." : "登録"}
      </Button>
    </form>
  );
}
```

## 高度なバリデーション

### パスワード確認

```typescript
const signupSchema = z
  .object({
    password: z.string().min(8, "パスワードは8文字以上"),
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "パスワードが一致しません",
    path: ["confirmPassword"],
  });
```

### 条件付きバリデーション

```typescript
const orderSchema = z.object({
  deliveryMethod: z.enum(["pickup", "delivery"]),
  address: z.string().optional(),
}).refine(
  (data) => {
    if (data.deliveryMethod === "delivery") {
      return data.address && data.address.length > 0;
    }
    return true;
  },
  {
    message: "配送の場合は住所が必要です",
    path: ["address"],
  }
);
```

### 数値フィールド

```tsx
// 数値フィールドは valueAsNumber を使用
<Input
  type="number"
  {...register("age", { valueAsNumber: true })}
/>
```

## Server Actions との連携

```tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { createUser } from "@/actions/user";

export function UserForm() {
  const form = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
  });

  const onSubmit = async (data: UserFormData) => {
    const result = await createUser(data);
    if (result.error) {
      // サーバーエラーの処理
      form.setError("root", { message: result.error });
    }
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      {/* フォームフィールド */}
      {form.formState.errors.root && (
        <p className="text-red-500">{form.formState.errors.root.message}</p>
      )}
    </form>
  );
}
```

## 重要なルール

1. **スキーマ定義**: `src/schemas/` にZodスキーマを配置
2. **型推論**: `z.infer<typeof schema>` で型を生成
3. **エラー表示**: `errors.fieldName.message` でエラーメッセージを表示
4. **バリデーション**: サーバー側でも同じスキーマで検証
5. **リセット**: `reset()` でフォームをクリア

## ファイル配置

```
src/
├── schemas/
│   ├── user.ts
│   └── post.ts
└── components/
    └── forms/
        └── user-form.tsx
```
