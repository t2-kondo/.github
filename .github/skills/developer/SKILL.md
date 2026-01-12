---
name: developer
description: 開発エージェントスキル。ビジネスアナリストの成果物を分析し、コードを開発します。要件定義書やユーザーストーリーからの実装、コード生成、テスト作成を依頼された場合に使用してください。
---

# 開発エージェントスキル

ビジネスアナリストの成果物（要件定義書、ユーザーストーリー）を分析し、実装コードを生成する支援を行います。

## 主要な責務

1. **成果物分析**: 要件定義書・ユーザーストーリーを読み解き、実装タスクを特定
2. **コード生成**: 各機能のコードを実装
3. **テスト作成**: ユニットテスト・統合テストを作成
4. **品質確保**: コードレビュー観点でのセルフチェック

---

## 開発環境

### 技術スタック

| カテゴリ | 技術 | バージョン |
|----------|------|------------|
| フレームワーク | Next.js (App Router) | 15.x |
| 言語 | TypeScript | 5.x |
| スタイリング | Tailwind CSS | 3.x |
| データベース | PostgreSQL | 16 |
| ORM | Prisma | 7.x |
| AI | Claude API (@anthropic-ai/sdk) | - |
| 状態管理 | Zustand | 5.x |
| データ取得 | TanStack Query | 5.x |
| バリデーション | Zod | 4.x |

### Docker環境

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16-alpine
    container_name: dev-command-room-db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: devuser
      POSTGRES_PASSWORD: devpassword
      POSTGRES_DB: devcommandroom
```

### 開発コマンド

```bash
# データベース操作
npm run db:up        # PostgreSQL起動 (WSL Docker)
npm run db:down      # PostgreSQL停止
npm run db:reset     # データベースリセット
npm run db:migrate   # マイグレーション実行
npm run db:generate  # Prisma Client生成
npm run db:studio    # Prisma Studio起動

# 開発
npm run dev          # 開発サーバー起動
npm run build        # ビルド
npm run lint         # Lint実行
```

### 環境変数 (.env)

```bash
# Database (PostgreSQL via Docker on WSL)
DATABASE_URL="postgresql://devuser:devpassword@<WSL_IP>:5432/devcommandroom?schema=public"
SHADOW_DATABASE_URL="postgresql://devuser:devpassword@<WSL_IP>:5432/devcommandroom_shadow?schema=public"

# Claude API
ANTHROPIC_API_KEY="your-api-key-here"
```

> **Note**: WSLのIPアドレスは `wsl -e hostname -I` で確認できます。

### Prisma設定

```
prisma/
├── schema.prisma      # スキーマ定義
└── migrations/        # マイグレーションファイル

prisma.config.ts       # Prisma 7 設定ファイル
```

---

## 開発フロー

```
┌─────────────────────────────────────────────────────────────┐
│                    開発エージェントの流れ                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. 成果物分析                                                │
│     └─→ doc/02_requirements/ と doc/03_user-stories/ を読む   │
│                                                              │
│  2. 基盤実装                                                  │
│     └─→ データモデル、DB接続、基本設定                        │
│                                                              │
│  3. API実装                                                   │
│     └─→ エンドポイント、バリデーション、エラーハンドリング     │
│                                                              │
│  4. UI実装                                                    │
│     └─→ コンポーネント、ページ、スタイリング                  │
│                                                              │
│  5. テスト                                                    │
│     └─→ ユニットテスト、動作確認                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 成果物の読み取り方法

### 要件定義書（doc/02_requirements/）から読み取る情報

```markdown
# 読み取るべき項目

## 機能要件から
- 機能ID (FR-X.X) → 実装する機能の特定
- 優先度 (Must/Should/Could) → 実装順序
- アクター → 誰が使う機能か

## スコープから
- 対象スコープ → 実装対象
- スコープ外 → 実装しない
```

### ユーザーストーリー（doc/03_user-stories/）から読み取る情報

```markdown
# 読み取るべき項目

## ストーリーから
- As a [役割] → ユーザータイプ
- I want [機能] → 実装する機能
- So that [価値] → 機能の目的

## 受け入れ条件から
- [ ] 条件1 → テストケース
- [ ] 条件2 → テストケース
```

---

## コード生成規則

### ディレクトリ構成

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API Routes
│   │   └── [resource]/
│   │       └── route.ts
│   ├── (routes)/          # ページルート
│   │   └── [page]/
│   │       └── page.tsx
│   ├── layout.tsx
│   └── page.tsx
│
├── ai/                     # AIアシスタント関連
│   ├── prompts/           # システムプロンプト定義
│   │   └── [name].ts      # 各種プロンプト
│   ├── agents/            # AIエージェント
│   │   └── [name].ts      # 各種エージェント
│   ├── tools/             # AIツール定義
│   │   └── [name].ts      # 各種ツール
│   └── client.ts          # Claude APIクライアント
│
├── components/            # Reactコンポーネント
│   ├── ui/               # 基本UIコンポーネント (shadcn/ui)
│   ├── features/         # 機能別コンポーネント
│   └── layouts/          # レイアウトコンポーネント
│
├── lib/                   # ユーティリティ
│   ├── db.ts             # Prisma クライアント
│   ├── utils.ts          # 共通ユーティリティ
│   └── validations/      # Zodスキーマ
│
├── hooks/                 # カスタムフック
│
├── stores/                # Zustand ストア
│
└── types/                 # 型定義
```

### AIフォルダの構成ルール

| フォルダ | 用途 | 例 |
|----------|------|-----|
| `ai/prompts/` | システムプロンプト定義 | `businessAnalyst.ts`, `developer.ts` |
| `ai/agents/` | AIエージェントロジック | `requirementsAgent.ts` |
| `ai/tools/` | AIが使用するツール定義 | `documentGenerator.ts` |
| `ai/client.ts` | Claude APIクライアント | - |

```typescript
// ai/prompts/businessAnalyst.ts の例
export const BUSINESS_ANALYST_PROMPT = `
あなたは要件定義を支援するビジネスアナリストです。
...
`;

// ai/agents/requirementsAgent.ts の例
import { chat } from "../client";
import { BUSINESS_ANALYST_PROMPT } from "../prompts/businessAnalyst";

export async function analyzeRequirements(input: string) {
  return await chat([{ role: "user", content: input }], BUSINESS_ANALYST_PROMPT);
}
```

### コーディング規約

```typescript
// ファイル命名
// - コンポーネント: PascalCase (ChatMessage.tsx)
// - ユーティリティ: camelCase (formatDate.ts)
// - 型定義: PascalCase (Message.ts)

// コンポーネント
// - 関数コンポーネント + TypeScript
// - Props は interface で定義
// - 'use client' は必要な場合のみ

// API Routes
// - app/api/[resource]/route.ts
// - GET, POST, PUT, DELETE をエクスポート
// - Zod でバリデーション

// エラーハンドリング
// - try-catch で適切にラップ
// - エラーメッセージは日本語
```

---

## 実装チェックリスト

### 機能実装時

- [ ] ユーザーストーリーの受け入れ条件を確認
- [ ] データモデルは定義済みか
- [ ] APIエンドポイントは実装済みか
- [ ] 必要なコンポーネントは作成済みか
- [ ] エラーハンドリングは適切か
- [ ] TypeScript の型は正しいか

### コミット前

- [ ] `npm run lint` が通る
- [ ] `npm run type-check` が通る
- [ ] `npm run test` が通る
- [ ] 動作確認済み

---

## ユーザーストーリーからタスクへの変換例

### 入力: ユーザーストーリー

```markdown
### US-1.1: AIアシスタントとの対話
**As a** ビジネスアナリスト  
**I want** チャット形式でAIアシスタントと対話したい  
**So that** 自然な会話を通じて要件を整理できる

**受け入れ条件:**
- [ ] テキスト入力欄からメッセージを送信できる
- [ ] AIからの応答がチャット形式で表示される
- [ ] メッセージは時系列順に表示される
```

### 出力: 開発タスク

```markdown
## タスク分割

### 基盤
- [ ] Message モデル定義 (Prisma)
- [ ] Session モデル定義 (Prisma)
- [ ] DB マイグレーション

### API
- [ ] POST /api/messages - メッセージ送信
- [ ] GET /api/messages - メッセージ取得
- [ ] Claude API 連携

### UI
- [ ] ChatInput コンポーネント
- [ ] ChatMessage コンポーネント
- [ ] ChatContainer コンポーネント
- [ ] チャットページ

### テスト
- [ ] API テスト
- [ ] コンポーネントテスト
```

---

## 改訂履歴

| バージョン | 日付 | 作成者 | 変更内容 |
|------------|------|--------|----------|
| 1.0 | 2026-01-01 | - | 初版作成 |
