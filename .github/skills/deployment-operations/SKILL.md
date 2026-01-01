---
name: deployment-operations
description: Guide for deploying and operating Next.js applications. Use this when asked about deployment, Vercel, environment variables, monitoring, or production setup.
---

# デプロイと運用ガイド

このプロジェクトのデプロイと本番運用に関するガイドです。

## 本番環境への準備

### 環境変数ファイル

```
プロジェクトルート/
├── .env                  # デフォルト（全環境共通）
├── .env.local            # ローカル上書き（Gitに含めない）
├── .env.development      # 開発環境
├── .env.production       # 本番環境
└── .env.test             # テスト環境
```

### 環境変数の種類

```bash
# サーバー専用（NEXT_PUBLIC_ なし）
DATABASE_URL="postgresql://..."
AUTH_SECRET="super-secret-key"

# クライアント公開（NEXT_PUBLIC_ あり）
NEXT_PUBLIC_API_URL="https://api.example.com"
```

### 型安全な環境変数

```typescript
// src/lib/env.ts
import { z } from "zod";

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  AUTH_SECRET: z.string().min(32),
});

export const env = envSchema.parse({
  DATABASE_URL: process.env.DATABASE_URL,
  AUTH_SECRET: process.env.AUTH_SECRET,
});
```

## ビルド設定

### package.json スクリプト

```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "prisma generate && next build",
    "start": "next start",
    "lint": "next lint",
    "test": "vitest",
    "typecheck": "tsc --noEmit",
    "check": "npm run lint && npm run typecheck && npm run test -- --run"
  }
}
```

### next.config.ts 本番設定

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // 画像最適化
  images: {
    remotePatterns: [
      { protocol: "https", hostname: "example.com" },
    ],
    formats: ["image/avif", "image/webp"],
  },

  // 圧縮
  compress: true,

  // ソースマップ（本番では無効化）
  productionBrowserSourceMaps: false,
};

export default nextConfig;
```

## Vercelへのデプロイ

### デプロイ手順

1. GitHubリポジトリにプッシュ
2. [vercel.com](https://vercel.com) でプロジェクトをインポート
3. 環境変数を設定
4. "Deploy" をクリック

### 環境変数の設定

```
Project Settings → Environment Variables
├── DATABASE_URL = "postgresql://..."
├── AUTH_SECRET = "your-secret-key"
└── NEXT_PUBLIC_API_URL = "https://api.example.com"
```

### 自動デプロイ

- **main ブランチ**: 本番環境に自動デプロイ
- **その他のブランチ/PR**: プレビュー環境に自動デプロイ

## 監視とログ

### Vercel Analytics

```tsx
// src/app/layout.tsx
import { Analytics } from "@vercel/analytics/react";
import { SpeedInsights } from "@vercel/speed-insights/next";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}
```

### Core Web Vitals 目標

| 指標 | 説明 | 目標値 |
|------|------|--------|
| LCP | 最大コンテンツの描画 | < 2.5秒 |
| FID | 初回入力遅延 | < 100ms |
| CLS | レイアウトシフト | < 0.1 |
| INP | インタラクション遅延 | < 200ms |

### 構造化ログ

```typescript
// src/lib/logger.ts
type LogLevel = "debug" | "info" | "warn" | "error";

class Logger {
  private log(level: LogLevel, message: string, data?: Record<string, unknown>) {
    const entry = {
      level,
      message,
      timestamp: new Date().toISOString(),
      data,
    };

    if (process.env.NODE_ENV === "production") {
      console.log(JSON.stringify(entry));
    } else {
      console.log(`[${level.toUpperCase()}] ${message}`, data || "");
    }
  }

  info(message: string, data?: Record<string, unknown>) {
    this.log("info", message, data);
  }

  error(message: string, data?: Record<string, unknown>) {
    this.log("error", message, data);
  }
}

export const logger = new Logger();
```

## セキュリティ

### セキュリティヘッダー

```typescript
// next.config.ts
const nextConfig: NextConfig = {
  async headers() {
    return [
      {
        source: "/:path*",
        headers: [
          { key: "X-DNS-Prefetch-Control", value: "on" },
          { key: "Strict-Transport-Security", value: "max-age=63072000; includeSubDomains" },
          { key: "X-Content-Type-Options", value: "nosniff" },
          { key: "X-Frame-Options", value: "DENY" },
          { key: "X-XSS-Protection", value: "1; mode=block" },
        ],
      },
    ];
  },
};
```

## 本番チェックリスト

```
□ 環境変数が本番用に設定されている
□ npm run build が成功する
□ TypeScript/ESLintエラーがない
□ テストが全て通る
□ セキュリティヘッダーが設定されている
□ 画像が最適化されている
□ データベースマイグレーションが適用されている
```

## Vercel CLI

```bash
# インストール
npm install -g vercel

# ログイン
vercel login

# 手動デプロイ（プレビュー）
vercel

# 本番デプロイ
vercel --prod

# ログ確認
vercel logs
```
