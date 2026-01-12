---
name: ai-chat-interface
description: AIチャットインターフェース開発スキル。LLMを活用した対話型UIの設計・実装を支援します。チャットUI、ストリーミング表示、プロンプト入力、AIレスポンス表示などの開発を依頼された場合に使用してください。
---

# AIチャットインターフェース開発スキル

AIチャットインターフェースを標準化するための包括的デザインシステムとUXエンジニアリングガイドライン。

---

## UI実装標準（必須準拠）

本プロジェクトでAIチャットUIを実装する際は、以下の標準に従ってください。
**リファレンス実装**: `src/app/products/[id]/epics/[epicId]/components/ChatPanel.tsx`

### レイアウト構造

```tsx
<aside className="w-96 border-l border-slate-200 flex flex-col overflow-hidden bg-white">
  {/* 1. ヘッダー */}
  <header className="px-4 py-3 border-b border-slate-200 shrink-0 bg-gradient-to-r from-slate-50 to-white">
    {/* AIアイコン + タイトル + コンテキスト表示 */}
  </header>

  {/* 2. メッセージエリア */}
  <div 
    ref={messagesContainerRef}
    className="flex-1 overflow-y-auto px-3 py-4 space-y-4"
  >
    {/* メッセージ一覧 */}
  </div>

  {/* 3. 入力エリア */}
  <footer className="p-3 border-t border-slate-200 shrink-0 bg-white">
    {/* テキストエリア + 送信ボタン */}
  </footer>

  {/* 4. 免責事項 */}
  <div className="px-3 pb-2 text-center">
    <p className="text-[10px] text-slate-400">⚠️ AIは誤った情報を生成する可能性があります</p>
  </div>
</aside>
```

### コンポーネント標準

#### 1. ヘッダー
```tsx
<header className="px-4 py-3 border-b border-slate-200 shrink-0 bg-gradient-to-r from-slate-50 to-white">
  <div className="flex items-center gap-2">
    <div className="w-8 h-8 rounded-lg bg-gradient-to-br from-blue-500 to-blue-600 flex items-center justify-center shadow-sm">
      <span className="text-white text-sm">AI</span>
    </div>
    <div>
      <h2 className="font-semibold text-slate-800 text-sm">アシスタント</h2>
      <p className="text-xs text-slate-400">コンテキスト情報</p>
    </div>
  </div>
</header>
```

#### 2. メッセージバブル

**ユーザーメッセージ（右寄せ）**:
```tsx
<div className="flex justify-end">
  <div className="max-w-[85%] rounded-2xl rounded-br-md bg-blue-600 text-white px-3 py-2 shadow-sm">
    <p className="text-sm whitespace-pre-wrap">{content}</p>
  </div>
</div>
```

**AIメッセージ（左寄せ）**:
```tsx
<div className="flex justify-start">
  <div className="max-w-[85%] rounded-2xl rounded-bl-md bg-slate-50 border border-slate-200 text-slate-800 px-3 py-2">
    <div className="prose prose-sm prose-slate max-w-none">
      <ReactMarkdown remarkPlugins={[remarkGfm]}>{content}</ReactMarkdown>
    </div>
  </div>
</div>
```

#### 3. ストリーミング表示
```tsx
{/* ストリーミング中 */}
<div className="flex justify-start">
  <div className="max-w-[85%] rounded-2xl rounded-bl-md bg-slate-50 border border-slate-200">
    {/* ヘッダー + 停止ボタン */}
    <div className="flex items-center justify-between px-3 pt-2 pb-1 border-b border-slate-100">
      <span className="text-xs text-slate-500">生成中...</span>
      <button
        onClick={onCancelGeneration}
        className="text-xs px-2 py-1 text-red-500 hover:text-red-600 hover:bg-red-50 rounded transition-colors flex items-center gap-1"
      >
        <span>■</span>
        <span>停止</span>
      </button>
    </div>
    {/* コンテンツ + カーソル */}
    <div className="text-sm leading-relaxed px-3 py-2 text-slate-800">
      <div className="prose prose-sm prose-slate max-w-none">
        <ReactMarkdown remarkPlugins={[remarkGfm]}>{streamingText}</ReactMarkdown>
      </div>
      {/* カーソル */}
      <span className="inline-block w-1.5 h-4 bg-blue-500 ml-0.5 animate-pulse" />
    </div>
  </div>
</div>
```

#### 4. 思考状態表示
```tsx
{/* 思考中（ストリーミング開始前） */}
<div className="flex justify-start">
  <div className="max-w-[85%] bg-blue-50 border border-blue-200 rounded-2xl rounded-bl-md px-3 py-2.5">
    <div className="flex items-center gap-3">
      {/* 3点アニメーション */}
      <div className="flex gap-1">
        <div className="w-1.5 h-1.5 bg-blue-500 rounded-full animate-bounce" />
        <div className="w-1.5 h-1.5 bg-blue-500 rounded-full animate-bounce [animation-delay:0.1s]" />
        <div className="w-1.5 h-1.5 bg-blue-500 rounded-full animate-bounce [animation-delay:0.2s]" />
      </div>
      <span className="text-sm text-blue-700">{thinkingStatus}</span>
      <button onClick={onCancelGeneration} className="text-xs px-2 py-1 text-red-500 hover:text-red-600 hover:bg-red-50 rounded ml-auto">
        ■ 停止
      </button>
    </div>
  </div>
</div>
```

#### 4.1 ツール実行ログ表示

AIがツールを呼び出す際、ユーザーに何が起きているかを透明に表示します。

**ToolLog型定義**
```tsx
interface ToolLog {
  id: string;
  toolName: string;
  status: "calling" | "completed" | "error";
  params?: Record<string, unknown>;  // ツールに渡す引数
  result?: string;                    // 実行結果の概要
  error?: string;                     // エラー時のメッセージ
}
```

**ストリーミング中のツールログ表示**
```tsx
{currentToolLogs.length > 0 && (
  <div className="mb-2 text-xs space-y-2">
    {currentToolLogs.map((log) => (
      <div
        key={log.id}
        className={`rounded ${
          log.status === "completed"
            ? "bg-green-50 border border-green-200"
            : log.status === "error"
            ? "bg-red-50 border border-red-200"
            : "bg-blue-50 border border-blue-200"
        }`}
      >
        {/* ツール名とステータス */}
        <div className={`flex items-center gap-2 px-2 py-1 ${
          log.status === "completed"
            ? "text-green-700"
            : log.status === "error"
            ? "text-red-700"
            : "text-blue-700"
        }`}>
          {log.status === "calling" && <span className="animate-spin">⚙️</span>}
          {log.status === "completed" && <span>✅</span>}
          {log.status === "error" && <span>❌</span>}
          <span className="font-mono font-medium">{log.toolName}</span>
        </div>
        
        {/* パラメータ表示（あれば） */}
        {log.params && Object.keys(log.params).length > 0 && (
          <div className="px-2 pb-1 text-slate-600">
            <span className="text-slate-400">引数: </span>
            {Object.entries(log.params).map(([key, value]) => (
              <span key={key} className="mr-2">
                <span className="text-slate-500">{key}=</span>
                <span className="text-slate-700 font-mono">
                  {typeof value === 'string' 
                    ? (value.length > 30 ? value.substring(0, 30) + '...' : value)
                    : JSON.stringify(value)}
                </span>
              </span>
            ))}
          </div>
        )}
      </div>
    ))}
  </div>
)}
```

**サーバーサイドのイベント送信**
```tsx
// ツール呼び出し開始
send("thinking", {
  status: "calling_tool",
  tool: block.name,
  toolId: block.id,
  message: `🔧 ${getToolDisplayName(block.name)} を実行中...`,
});

// ツール引数送信
send("thinking", {
  status: "processing_tool",
  tool: block.name,
  params: toolInput,
  message: `⚙️ ${getToolDisplayName(block.name)} の結果を処理中...`,
});

// ツール完了
send("thinking", {
  status: "tool_complete",
  tool: block.name,
  result: resultSummary,
  message: `✅ ${getToolDisplayName(block.name)} 完了`,
});
```

**クライアントサイドのイベント処理**
```tsx
if (data.status === "calling_tool") {
  const newLog: ToolLog = {
    id: `tool-${Date.now()}-${data.tool}`,
    toolName: data.tool,
    status: "calling",
  };
  toolLogs.push(newLog);
  setCurrentToolLogs([...toolLogs]);
} else if (data.status === "processing_tool") {
  // ツール引数を記録
  const logIndex = toolLogs.findIndex((l) => l.toolName === data.tool && l.status === "calling");
  if (logIndex !== -1) {
    toolLogs[logIndex] = { ...toolLogs[logIndex], params: data.params };
    setCurrentToolLogs([...toolLogs]);
  }
} else if (data.status === "tool_complete") {
  // ツール結果を記録
  const logIndex = toolLogs.findIndex((l) => l.toolName === data.tool && l.status === "calling");
  if (logIndex !== -1) {
    toolLogs[logIndex] = { ...toolLogs[logIndex], status: "completed", result: data.result };
    setCurrentToolLogs([...toolLogs]);
  }
}
```

#### 5. 入力エリア

**キーボード操作**
- **Enter**: 改行（複数行入力をサポート）
- **Shift+Enter**: 送信

```tsx
<footer className="p-3 border-t border-slate-200 shrink-0 bg-white">
  <div className="flex gap-2 items-end">
    {/* テキストエリア（オートグロース） */}
    <textarea
      value={chatInput}
      onChange={(e) => {
        setChatInput(e.target.value);
        // Auto-resize: 入力内容に応じて高さを自動調整
        e.target.style.height = "auto";
        e.target.style.height = Math.min(e.target.scrollHeight, 150) + "px";
      }}
      onKeyDown={(e) => {
        // Shift+Enterで送信（Enterは改行）
        if (e.key === "Enter" && e.shiftKey && !isSending && chatInput.trim()) {
          e.preventDefault();
          onSendMessage();
        }
      }}
      placeholder="質問を入力... (Shift+Enterで送信)"
      rows={1}
      style={{ height: "auto", minHeight: "40px" }}
      className="flex-1 bg-slate-50 border border-slate-200 rounded-xl px-3 py-2.5 text-sm text-slate-900 placeholder-slate-400 resize-none overflow-y-auto focus:outline-none focus:border-blue-400 focus:ring-1 focus:ring-blue-100 focus:bg-white transition-all"
      disabled={isSending}
    />
    
    {/* 送信/停止ボタン */}
    {isSending ? (
      <button onClick={onCancelGeneration} className="rounded-xl bg-red-500 hover:bg-red-600 px-4 py-2.5 text-white transition-colors shadow-sm flex items-center gap-1.5">
        <span className="text-sm">■</span>
        <span className="text-sm font-medium">停止</span>
      </button>
    ) : (
      <button
        type="button"
        onClick={onSendMessage}
        disabled={!chatInput.trim() || isSending}
        className={`rounded-xl px-4 py-2.5 transition-all shadow-sm self-end ${
          chatInput.trim() ? "bg-blue-600 hover:bg-blue-500 text-white" : "bg-slate-200 text-slate-400 cursor-not-allowed"
        }`}
      >
        送信
      </button>
    )}
  </div>
  
  {/* キーボードヒント */}
  <div className="mt-2 flex items-center justify-between text-xs text-slate-400">
    <span>
      <kbd className="px-1 py-0.5 bg-slate-100 rounded text-[10px]">Shift+Enter</kbd> 送信
      <span className="mx-1">·</span>
      <kbd className="px-1 py-0.5 bg-slate-100 rounded text-[10px]">Enter</kbd> 改行
    </span>
    <span className="text-slate-300">{chatInput.length > 0 && `${chatInput.length}字`}</span>
  </div>
</footer>
```

### 必須機能チェックリスト

| 項目 | 必須 | 説明 |
|------|------|------|
| **レイアウト** | ✅ | 固定幅96(w-96)、border-l、flex-col |
| **ヘッダー** | ✅ | AIアイコン + タイトル + コンテキスト表示 |
| **メッセージ非対称性** | ✅ | ユーザー右寄せ/AI左寄せ |
| **テキストエリア** | ✅ | input[text]ではなくtextarea使用 |
| **オートグロース** | ✅ | 入力に応じて高さ自動拡張(最大5-6行) |
| **ストリーミングカーソル** | ✅ | 生成中テキスト末尾に点滅カーソル |
| **停止ボタン** | ✅ | ストリーミング中・思考中に表示 |
| **思考状態表示** | ✅ | 3点アニメーション + ステータステキスト |
| **ツール実行ログ** | ✅ | 呼び出しツール名・パラメータ・結果表示 |
| **キーボードヒント** | ✅ | Shift+Enter送信/Enter改行 |
| **免責事項** | ✅ | AIの誤情報可能性を明示 |

### カラーパレット

| 用途 | クラス |
|------|--------|
| ユーザーバブル背景 | `bg-blue-600` |
| ユーザーバブルテキスト | `text-white` |
| AIバブル背景 | `bg-slate-50` |
| AIバブルボーダー | `border border-slate-200` |
| AIバブルテキスト | `text-slate-800` |
| 思考状態背景 | `bg-blue-50 border-blue-200` |
| 思考状態テキスト | `text-blue-700` |
| ツール実行中背景 | `bg-blue-50 border-blue-200` |
| ツール完了背景 | `bg-green-50 border-green-200` |
| ツールエラー背景 | `bg-red-50 border-red-200` |
| 停止ボタン | `text-red-500 hover:text-red-600` |
| 送信ボタン（有効） | `bg-blue-600 hover:bg-blue-500 text-white` |
| 送信ボタン（無効） | `bg-slate-200 text-slate-400` |

### オートグロース実装（インライン方式）

refを使わずにonChange内で直接高さを調整するシンプルな実装：

```tsx
<textarea
  value={chatInput}
  onChange={(e) => {
    setChatInput(e.target.value);
    // Auto-resize: 入力内容に応じて高さを自動調整
    e.target.style.height = "auto";
    e.target.style.height = Math.min(e.target.scrollHeight, 150) + "px"; // 最大150px
  }}
  rows={1}
  style={{ height: "auto", minHeight: "40px" }}
  className="resize-none overflow-y-auto"
/>
```

**オートグロース実装（useRef方式）**

外部関数から制御する場合：

```tsx
const textareaRef = useRef<HTMLTextAreaElement>(null);

const handleTextareaChange = useCallback((value: string) => {
  setChatInput(value);
  if (textareaRef.current) {
    textareaRef.current.style.height = "auto";
    const scrollHeight = textareaRef.current.scrollHeight;
    textareaRef.current.style.height = `${Math.min(scrollHeight, 150)}px`; // 最大150px
  }
}, [setChatInput]);
```

### スマートスクロール実装

```tsx
const messagesContainerRef = useRef<HTMLDivElement>(null);
const messagesEndRef = useRef<HTMLDivElement>(null);

useEffect(() => {
  const container = messagesContainerRef.current;
  if (!container) return;
  
  // ユーザーが上にスクロールしていない場合のみ自動スクロール
  const isNearBottom = container.scrollHeight - container.scrollTop - container.clientHeight < 100;
  if (isNearBottom) {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  }
}, [messages, streamingText]);
```

---

## 序論：決定論的UIから確率論的UIへのパラダイムシフト

### 従来のUIとAIチャットUIの本質的な違い

従来のグラフィカルユーザーインターフェース（GUI）は、**「コマンド＆コントロール」モデル**に基づいていた。ユーザーが明確なコマンドを入力し、システムが**決定論的（Deterministic）**な結果を返す。ボタンを押せば、常に同じ結果が得られる。

しかし、大規模言語モデル（LLM）の台頭により、我々は**「インテントベース」インターフェース**への移行に直面している。ユーザーの曖昧なインテント（意図）を解釈し、**確率論的（Probabilistic）**な結果を生成する。同じプロンプトでも、毎回異なる回答が返る可能性がある。

### AIチャット特有の課題

チャットインターフェースは、もはや単なるメッセージングアプリの模倣ではなく、**AIの高度な推論能力を操作するための主要なオペレーティングシステム（OS）**へと昇華しつつある。

しかし、既存のチャットUIパターンは人間同士の即時的なコミュニケーションを前提としており、生成AI特有の課題に対処するには不十分である：

| 課題 | 従来のチャット | AIチャット |
|------|---------------|-----------|
| **レイテンシー** | ほぼ即時（人間の入力速度） | 変動する（数秒〜数十秒） |
| **正確性** | 人間の責任 | ハルシネーション（事実誤認）のリスク |
| **文脈** | 人間が自然に保持 | コンテキストウィンドウの制限 |
| **予測可能性** | 高い（人間の意図は明確） | 低い（確率的な出力） |

---

## 第1章：主要デザインシステムにおける対話型インターフェースの哲学

### 1.1 Apple Human Interface Guidelines：流動性と共感

Appleのアプローチは、システムがあたかも人間のような**「共感（Empathy）」と「流動性（Fluidity）」**を持つことを重視する。

**核心的な考え方：**

- **言語の力**: インターフェース上のマイクロコピーやAIの応答は、単なる情報伝達ではなく、アプリケーションの「人格」を形成する要素である
- **段階的開示（Progressive Disclosure）**: 複雑な機能を一度に提示せず、対話の進行に合わせて情報を小出しにし、認知負荷を最小限に抑える
- **専門用語の排除**: 包括的（Inclusive）で親しみやすい言葉選びにより、ユーザーの心理的障壁を下げる

### 1.2 Google Material Design 3：適応性と感情表現

GoogleのM3は、**「アダプティブデザイン」と「表現力（Expressive Design）」**に焦点を当てる。

**核心的な考え方：**

- **Canonical Layouts**: 特に「List-Detail（リスト詳細）」レイアウトはチャットUIの標準形として有効。左ペインに会話履歴、右ペインにアクティブなチャット
- **感情を伝えるモーション**: メッセージ送信時のトランジションや、AI応答生成中のアニメーションは、待機時間を「期待感のある体験」に変換する
- **デバイス適応**: スマートフォン、タブレット、折りたたみデバイス、デスクトップへの適応

### 1.3 Microsoft Fluent UI：生産性とコパイロット

MicrosoftはAIを**「副操縦士（Copilot）」**と位置づけ、主役はあくまでユーザーである。

**核心的な考え方：**

- **非侵襲性**: AIの提案や介入は、ユーザーの作業フローを中断させない。サイドパネルやフローティングウィンドウとして実装
- **明確な意図確認**: 生成AIの出力が不確実であることを前提とし、ユーザーが容易に検証・修正・破棄できるコントロールを提供
- **過度な信頼の回避**: Over-trustによるリスクを回避する設計思想

### 1.4 統合原則：AIチャット標準化への示唆

| 観点 | Apple HIG | Material Design 3 | Fluent UI | **統合指針** |
|------|-----------|-------------------|-----------|-------------|
| **コア哲学** | 人間らしさ、流動性 | 適応性、感情表現 | 生産性、コパイロット | シームレスな操作性と感情的充足の両立 |
| **レイアウト** | 階層構造重視 | レスポンシブ、List-Detail | サイドパネル、オーバーレイ | デバイスに応じた情報密度調整 |
| **モーション** | 物理法則に基づく自然な動き | 感情を伝える表現豊かな動き | 効率的で素早い遷移 | 「思考中」の状態を感情的に演出 |
| **トーン** | 親しみやすく、敬意を払う | ユーザーの好みに適応 | プロフェッショナル、明確 | 信頼性を損なわない親しみやすさ |

---

## 第2章：チャットUIの解剖学 — コンポーネント設計の原則

### 2.1 メッセージバブル：視認性と情報階層

メッセージバブルはチャットUIの**原子（Atom）**であり、その設計品質は可読性に直結する。

#### 形状とアフォーダンスの原則

- **非対称性の導入**: ユーザーとAIの発話を明確に区別する
- **メンタルモデルの遵守**: 「右側＝自分（ユーザー）」「左側＝相手（AI）」という配置原則
- **視覚的グルーピング**: 同一話者による連続メッセージは、Gestalt原則の近接性を活用して視覚的にグループ化
- **最適な行長**: バブルの最大幅は画面幅の60〜70%（約60〜80文字）に制限し、視線移動を最適化

#### カラーシステムの考え方

- **ユーザー側**: プライマリーカラー（ブランドカラー）を使用。青系統は「信頼」「コミュニケーション」を想起させる
- **AI側**: ニュートラルなグレーまたはサーフェイスカラー。長文になる傾向があるため、目に優しい配色が必須
- **ダークモード**: 純粋な黒（#000）ではなくダークグレーを使用し、ハレーション防止と有機ELディスプレイの電力効率を考慮

#### タイポグラフィの原則

- **本文サイズ**: 16px以上を基準。14px以下は長文読解で眼精疲労の原因
- **行間**: 標準（1.2〜1.4）より広めの1.5〜1.6。生成AIの出力は情報密度が高いため
- **パディング**: テキストが窮屈に見えないよう十分な余白を確保

### 2.2 入力フィールド：コマンドセンターへの進化

AIチャットにおける入力エリアは、単なるテキストボックスから**マルチモーダルな「コマンドセンター」**へと進化している。

#### 動的拡張性の原則

- **常時アクセス可能**: 画面下部に固定（Sticky）し、チャット履歴がスクロールしても入力欄は常に見える
- **オートグロース**: 入力テキスト量に応じて高さを自動拡張（最大5〜6行）。スクロールせずに全体を見渡せることで推敲が容易に
- **アクションを提案するプレースホルダー**: 「入力してください」ではなく「『要約して』と聞いてみてください」のような具体的提案でオンボーディング支援

#### マルチモーダル入力の統合原則

- **ファイルアップロード**: アイコンだけでなくドラッグ＆ドロップ領域を明確に。送信前にサムネイルプレビューと削除ボタン
- **音声入力**: 録音中は波形アニメーションで「システムが認識している」ことを視覚的にフィードバック
- **送信制御**: 空文字送信の無効化、送信後の入力欄クリア、フォーカス維持

### 2.3 アバター：擬人化の度合いを制御

AIのアバターは、システムへの**擬人化（Anthropomorphism）の度合い**を制御する重要な要素である。

- **不気味の谷の回避**: 写真のようなリアルな人間は避け、抽象化されたロゴやイラストを使用
- **AIであることの認識**: ユーザーは相手がAIであることを認識しつつ、親しみを感じられる

---

## 第3章：時間と不確実性の管理 — 生成AI特有のインタラクション

生成AIは従来のチャットボット（シナリオ型）と異なり、応答が動的に生成され完了まで時間を要する。この**「時間的特性」と「不確実性」**を管理するUXパターンが求められる。

### 3.1 ストリーミング表示の標準化

LLMの出力はトークン単位で逐次生成されるため、全生成完了を待たずに表示開始する**ストリーミング表示**が業界標準となっている。

#### 心理的効果

- **体感待ち時間の短縮**: 静的なローディングスピナーより、文字が徐々に現れる様子を見る方が進行感を強く感じる
- **タイプライター効果**: AIがリアルタイムに思考・生成しているプロセスの可視化により、逆説的にシステムへの信頼が向上
- **離脱率の低下**: 「何かが起きている」という即座のフィードバックがユーザーを引き留める

#### 実装における考慮事項

- **カーソル（キャレット）の明滅**: 生成中テキスト末尾に点滅カーソル（▍）を表示し、現在進行形を視覚的に強調
- **スムージング**: トークンが塊として表示される「カクつき」を避け、短いフェードインや出現速度の補間で滑らかに
- **自動スクロール制御**: 生成に合わせて自動スクロール。ただし、ユーザーが上にスクロールしたら即座に停止（ユーザー介入検知）

### 3.2 思考状態とローディングの可視化

AIが回答を生成する前の「推論・検索・ツール使用フェーズ」にも適切なフィードバックが必要。

#### プロセスの透明化

- **具体的な処理内容の表示**: 単なるローディング表示ではなく、「Webを検索中...」「ドキュメントを解析中...」といったテキストでユーザーの不安を解消
- **フリーズしていない保証**: システムが作業を続けていることを明示

#### Chain of Thought（思考の連鎖）の開示

- **説明可能性の向上**: AIの思考プロセスをアコーディオンUIで表示可能にし、ブラックボックス化を防ぐ
- **論理ステップの可視化**: どのような論理で回答に至ったかを確認できる

### 3.3 制御権の確保：停止と再生成

生成AIは時にユーザーの意図しない方向に回答を続けたり、ループに陥ることがある。**ユーザーに制御権を戻す**機能は必須。

#### 生成停止（Stop Generating）

- **明確な配置**: ストリーミング中は送信ボタンの位置または生成中メッセージの直下に停止ボタン
- **視認性**: アイコン（■）だけでなくテキストラベル併記
- **即時遮断**: ボタン押下時は即座に生成を遮断し、そこまでの出力を保持
- **シームレスな継続**: エラーではなく「中断されました」状態を表示し、即座に次の入力を受付可能に

#### 再生成（Regenerate）とバージョン管理

- **上書きしない**: 再生成された回答は既存を消去せず、「回答 1/2」「回答 2/2」としてバージョン管理
- **比較・選択可能**: 矢印ボタンやスワイプで異なるバージョンを行き来
- **後戻りリスクの排除**: 以前の回答の方が良かった場合に戻れる

---

## 第4章：信頼性と透明性を高める情報設計

AIの出力には誤り（ハルシネーション）が含まれる可能性がある。UIを通じて**情報の信頼性を担保し、ユーザーが検証できる仕組み**を提供する。

### 4.1 出典・引用（Citations）の視覚化

RAG（検索拡張生成）を用いたAIチャットでは、回答の根拠となる情報源を明示することが標準ルール。

#### インライン引用

- **脚注番号の埋め込み**: 本文中の該当箇所にリンクとして[1][2]を埋め込み
- **ポップオーバー表示**: クリックまたはホバーで出典のタイトル、ドメイン、スニペットを表示
- **ページ遷移なしの確認**: ユーザーは離脱せずにソースの信頼性を確認可能

#### 出典リストの展開

- **視覚的ノイズの軽減**: 参照リストはデフォルトで折りたたみ、興味あるユーザーだけが展開
- **情報の階層化**: 回答本文と出典情報を明確に分離

### 4.2 フィードバックループの実装

AIの精度向上のため、ユーザーからのフィードバックを収集するUIパターンを組み込む。

#### Thumbs Up / Down

- **控えめな配置**: メッセージバブル下部に目立たないように配置
- **理由の収集**: 低評価時は「事実ではない」「有害な内容」「役に立たない」から理由選択
- **感謝の表明**: フィードバック送信後、「ご意見ありがとうございます」のトースト通知

---

## 第5章：コンテキストと記憶の可視化 — 高度な対話管理

AIとの対話が長引くと、「何について話していたか」というコンテキスト管理が難しくなる。**認知負荷を下げるための記憶（Memory）と文脈の可視化**が必要。

### 5.1 スレッドとブランチング（Branching）

#### 会話の分岐

- **上書きしない編集**: 過去の発言を編集した際、履歴を消すのではなく新たな分岐として保存
- **ページネーションUI**: 「< 2 / 2 >」のような形式で異なる会話展開を行き来
- **探索的タスクへの対応**: 複数のアプローチを試し、最適解を選択可能

#### トピックの自動要約

- **意味のあるタイトル**: 「新しいチャット」ではなく、会話内容に基づいたタイトルを自動生成
- **過去対話の再開容易化**: チャット履歴リストで目的の会話を素早く発見

### 5.2 メモリ管理UI

#### 記憶の明示

- **透明性の確保**: AIが何を記憶したかを確認・削除できるダッシュボード
- **通知**: 「あなたがPythonエンジニアであることを覚えました」のような明示的な通知
- **修正・削除可能**: ユーザーが記憶を制御でき、プライバシー懸念を軽減

#### コンテキストウィンドウの警告

- **上限への対処**: 会話が長くなりすぎた場合、「古い会話の記憶が薄れています」と警告
- **新しいチャットの促進**: または自動要約が行われたことを通知

---

## 第6章：実装チェックリスト

### UX/UI実装チェックリスト

| カテゴリ | 項目 | 基準と仕様 |
|---------|------|-----------|
| レイアウト | レスポンシブ対応 | List-Detailレイアウト、モバイル/デスクトップ最適化 |
| 入力 | オートグロース | 内容に応じて高さ自動拡張（最大5-6行） |
| 入力 | 送信制御 | 空文字送信無効化、送信後クリア、フォーカス維持 |
| フィードバック | ストリーミング表示 | TTFT最小化、スムージングされたテキスト表示 |
| 制御 | 停止機能 | 生成中にいつでも中断可能な停止ボタン |
| 可読性 | コントラスト比 | バブル背景とテキストは4.5:1以上（WCAG AA） |
| 信頼性 | 出典表示 | 引用元リンクをインラインまたはリスト形式で明示 |

### 推奨技術スタック

| 用途 | 推奨ライブラリ |
|------|---------------|
| ストリーミング処理・状態管理 | Vercel AI SDK |
| Markdownレンダリング | react-markdown, remark-gfm |
| アニメーション | Framer Motion |

---

## 結論：標準化の先にある「アダプティブAI」

本ガイドラインで定義した開発ルールとベストプラクティスは、AIチャットインターフェースにおける**「衛生要因」**――満たされていなければユーザーに不満や不信感を与える要素――を網羅している。

これらを遵守することで、開発者はユーザーに対して**学習コストが低く、信頼性の高い対話体験**を提供できる。

### 今後の展望

- **Generative UI（GenUI）**: チャットという枠組みを超え、AIがユーザーのニーズに合わせてGUIそのものを動的に生成
- **高度なコンテキスト認識**: 対話履歴に基づいた、よりパーソナライズされた対話体験

### 開発者・デザイナーへの指針

> **「基本に忠実でありながら、変化に柔軟であること」**
> 
> 確立されたガイドライン（HIG, Material, Fluent）を基盤としつつ、ユーザーからのフィードバック（定量的・定性的データ）を常に監視し、インターフェースを継続的に反復改善（Iterative Design）していく姿勢が求められる。

---

## 参考リソース

- Apple Human Interface Guidelines - Conversational Experiences
- Google Material Design 3 - Adaptive Layouts
- Microsoft Fluent UI - Copilot Design Patterns
- Nielsen Norman Group - AI & UX Guidelines
- Vercel AI SDK Documentation
