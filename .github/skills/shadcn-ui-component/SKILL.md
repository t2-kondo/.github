---
name: shadcn-ui-component
description: Guide for creating UI components using shadcn/ui. Use this when asked to create buttons, cards, forms, dialogs, or any UI components.
---

# shadcn/ui コンポーネント作成ガイド

このプロジェクトでは shadcn/ui を使用してUIコンポーネントを作成します。

## コンポーネントの追加方法

新しいコンポーネントを追加する場合は、CLIを使用します：

```bash
npx shadcn@latest add <component-name>
```

### 利用可能な主要コンポーネント

| コンポーネント | コマンド | 用途 |
|--------------|---------|------|
| Button | `npx shadcn@latest add button` | ボタン |
| Card | `npx shadcn@latest add card` | カード |
| Input | `npx shadcn@latest add input` | 入力フィールド |
| Label | `npx shadcn@latest add label` | ラベル |
| Dialog | `npx shadcn@latest add dialog` | モーダル |
| Form | `npx shadcn@latest add form` | フォーム |
| Table | `npx shadcn@latest add table` | テーブル |
| Select | `npx shadcn@latest add select` | セレクトボックス |
| Tabs | `npx shadcn@latest add tabs` | タブ |
| Toast | `npx shadcn@latest add toast` | 通知 |

## コンポーネントの使用例

### Button

```tsx
import { Button } from "@/components/ui/button";

export function MyComponent() {
  return (
    <div className="space-x-2">
      <Button>デフォルト</Button>
      <Button variant="secondary">セカンダリ</Button>
      <Button variant="destructive">削除</Button>
      <Button variant="outline">アウトライン</Button>
      <Button variant="ghost">ゴースト</Button>
      <Button variant="link">リンク</Button>
    </div>
  );
}
```

### Card

```tsx
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";

export function MyCard() {
  return (
    <Card>
      <CardHeader>
        <CardTitle>カードタイトル</CardTitle>
        <CardDescription>カードの説明文</CardDescription>
      </CardHeader>
      <CardContent>
        <p>カードの内容</p>
      </CardContent>
      <CardFooter>
        <Button>アクション</Button>
      </CardFooter>
    </Card>
  );
}
```

### Input + Label

```tsx
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

export function MyForm() {
  return (
    <div className="space-y-2">
      <Label htmlFor="email">メールアドレス</Label>
      <Input id="email" type="email" placeholder="example@example.com" />
    </div>
  );
}
```

## 重要なルール

1. **インポートパス**: `@/components/ui/` からインポートする
2. **スタイリング**: Tailwind CSSのクラスを使用してカスタマイズ
3. **バリアント**: `variant` プロップで見た目を変更
4. **サイズ**: `size` プロップでサイズを変更（sm, default, lg）
5. **アクセシビリティ**: Radix UIベースで自動的にアクセシブル

## ファイル配置

- コンポーネント: `src/components/ui/`
- ユーティリティ: `src/lib/utils.ts`（`cn` 関数）

## cn関数の使い方

```tsx
import { cn } from "@/lib/utils";

// 条件付きクラスの適用
<div className={cn("base-class", isActive && "active-class")} />
```
