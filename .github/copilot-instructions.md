<!-- Next.js 学習サンプルプロジェクト -->

## プロジェクト概要
- **フレームワーク**: Next.js 16 (App Router)
- **言語**: TypeScript
- **スタイリング**: Tailwind CSS

## 技術スタック
- **UI**: shadcn/ui, Radix UI
- **状態管理**: Zustand
- **データ取得**: TanStack Query
- **フォーム**: React Hook Form + Zod
- **認証**: NextAuth.js (Auth.js v5)
- **ORM**: Prisma + SQLite
- **テスト**: Vitest + Testing Library

## エージェントスキル
各技術の詳細なガイドは `.github/skills/` フォルダに配置されています：
- `shadcn-ui-component` - UIコンポーネント作成
- `zustand-state-management` - 状態管理
- `tanstack-query-data-fetching` - データフェッチング
- `react-hook-form-zod-validation` - フォームバリデーション
- `nextauth-authentication` - 認証実装
- `prisma-database` - データベース操作
- `vitest-testing` - テスト作成
- `nextjs-app-router` - ページ・レイアウト作成
- `nextjs-api-routes` - APIルート作成
- `server-actions` - Server Actions実装
- `deployment-operations` - デプロイと運用

## 開発ルール
- ページは `src/app/` フォルダ内に作成
- APIは `src/app/api/` フォルダ内に作成
- UIコンポーネントは `src/components/ui/` を使用
- ストアは `src/stores/` フォルダ内に作成
- スキーマは `src/schemas/` フォルダ内に作成
- クライアントコンポーネントには `"use client"` を追加
- バリデーションにはZodスキーマを使用
- 日本語でコメントを記述
- **git commit/push は明示的に指示されるまで実行しないこと**

## ファイル配置規則
```
src/
├── app/                # ページ・APIルート
├── components/
│   └── ui/             # shadcn/uiコンポーネント
├── stores/             # Zustandストア
├── schemas/            # Zodスキーマ
├── lib/                # ユーティリティ
├── providers/          # Providerコンポーネント
├── actions/            # Server Actions
└── __tests__/          # テストファイル
```

## コマンド
- 開発: `npm run dev`
- ビルド: `npm run build`
- リント: `npm run lint`
- テスト: `npm run test`
- DB管理: `npm run db:studio`


