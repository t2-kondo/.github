---
name: claude-agent-sdk
description: Guide for building AI agents using Claude Agent SDK. Use this when asked to create autonomous agents, implement agentic workflows, or integrate Claude Code capabilities into applications.
---

# Claude Agent SDK ガイド

Claude Agent SDK は、Claude Code の機能をライブラリとして使用して本番環境の AI エージェントを構築するための SDK です。ファイルの読み取り、コマンドの実行、Web 検索、コード編集などを自動的に行うエージェントを構築できます。

## インストール

### TypeScript

```bash
npm install @anthropic-ai/claude-agent-sdk
```

### Python

```bash
pip install claude-agent-sdk
```

## 前提条件

1. **Claude Code のインストール**（ランタイムとして使用）

```bash
# macOS/Linux/WSL
curl -fsSL https://claude.ai/install.sh | bash

# Homebrew
brew install claude-code

# npm
npm install -g @anthropic-ai/claude-code
```

2. **API キーの設定**

```bash
export ANTHROPIC_API_KEY=your-api-key
```

## 基本的な使用方法

### Python での基本例

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    async for message in query(
        prompt="Find and fix the bug in auth.py",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Edit", "Bash"],
            permission_mode="acceptEdits"
        )
    ):
        print(message)  # Claude がファイルを読み、バグを見つけ、編集する

asyncio.run(main())
```

### TypeScript での基本例

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const result = query({
  prompt: "Find and fix the bug in auth.py",
  options: {
    allowedTools: ["Read", "Edit", "Bash"],
    permissionMode: "acceptEdits"
  }
});

for await (const message of result) {
  console.log(message);
}
```

## 組み込みツール

| ツール | 説明 |
|--------|------|
| `Read` | ファイルを読み取る |
| `Write` | 新しいファイルを作成する |
| `Edit` | 既存ファイルを編集する |
| `Bash` | ターミナルコマンドを実行する |
| `Glob` | パターンでファイルを検索する |
| `Grep` | 正規表現でファイル内容を検索する |
| `WebSearch` | Web を検索する |
| `WebFetch` | Web ページコンテンツを取得する |
| `Task` | サブエージェントを起動する |
| `NotebookEdit` | Jupyter ノートブックを編集する |

## 権限モード

| モード | 説明 | ユースケース |
|--------|------|--------------|
| `acceptEdits` | ファイル編集を自動承認 | 信頼できる開発ワークフロー |
| `bypassPermissions` | すべての権限をバイパス | CI/CD パイプライン |
| `default` | `canUseTool` コールバックが必要 | カスタム承認フロー |
| `plan` | 実行せず計画のみ | コードレビュー |

## ClaudeAgentOptions

### Python

```python
from claude_agent_sdk import ClaudeAgentOptions

options = ClaudeAgentOptions(
    # 許可するツール
    allowed_tools=["Read", "Edit", "Bash", "Glob", "Grep"],
    
    # 権限モード
    permission_mode="acceptEdits",
    
    # システムプロンプト
    system_prompt="You are an expert Python developer",
    
    # 作業ディレクトリ
    cwd="/home/user/project",
    
    # 最大ターン数
    max_turns=10,
    
    # 使用するモデル
    model="claude-sonnet-4-20250514",
    
    # MCP サーバー設定
    mcp_servers={
        "calculator": {
            "command": "node",
            "args": ["calculator-server.js"]
        }
    },
    
    # 環境変数
    env={"NODE_ENV": "development"},
    
    # 追加アクセスディレクトリ
    add_dirs=["/shared/libs"]
)
```

### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

const result = query({
  prompt: "Analyze this codebase",
  options: {
    allowedTools: ["Read", "Edit", "Bash", "Glob", "Grep"],
    permissionMode: "acceptEdits",
    systemPrompt: "You are an expert Python developer",
    cwd: "/home/user/project",
    maxTurns: 10,
    model: "claude-sonnet-4-20250514",
    mcpServers: {
      calculator: {
        command: "node",
        args: ["calculator-server.js"]
      }
    }
  }
});
```

## 会話セッション（Python）

### ClaudeSDKClient の使用

複数のメッセージで会話を維持する場合は `ClaudeSDKClient` を使用します。

```python
import asyncio
from claude_agent_sdk import ClaudeSDKClient, AssistantMessage, TextBlock

async def main():
    async with ClaudeSDKClient() as client:
        # 最初の質問
        await client.query("What's the capital of France?")
        
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")
        
        # フォローアップ質問（コンテキストを記憶）
        await client.query("What's the population of that city?")
        
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(f"Claude: {block.text}")

asyncio.run(main())
```

### query() vs ClaudeSDKClient の比較

| 機能 | `query()` | `ClaudeSDKClient` |
|------|-----------|-------------------|
| セッション | 毎回新規 | 再利用可能 |
| 会話 | 単一の交換 | 複数ターン |
| 割り込み | ❌ | ✅ |
| フック | ❌ | ✅ |
| カスタムツール | ❌ | ✅ |
| ユースケース | 1回限りのタスク | 継続的な会話 |

## カスタム MCP ツールの作成

### Python

```python
from claude_agent_sdk import tool, create_sdk_mcp_server, ClaudeAgentOptions, query

@tool("calculate", "Perform mathematical calculations", {"expression": str})
async def calculate(args):
    try:
        result = eval(args["expression"], {"__builtins__": {}})
        return {
            "content": [{
                "type": "text",
                "text": f"Result: {result}"
            }]
        }
    except Exception as e:
        return {
            "content": [{
                "type": "text",
                "text": f"Error: {str(e)}"
            }],
            "is_error": True
        }

@tool("get_time", "Get current time", {})
async def get_time(args):
    from datetime import datetime
    return {
        "content": [{
            "type": "text",
            "text": f"Current time: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
        }]
    }

# MCP サーバーを作成
calculator = create_sdk_mcp_server(
    name="calculator",
    version="1.0.0",
    tools=[calculate, get_time]
)

# Claude で使用
options = ClaudeAgentOptions(
    mcp_servers={"calc": calculator},
    allowed_tools=["mcp__calc__calculate", "mcp__calc__get_time"]
)

async def main():
    async for message in query(
        prompt="What's 123 * 456?",
        options=options
    ):
        print(message)
```

### TypeScript

```typescript
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const calculate = tool(
  "calculate",
  "Perform mathematical calculations",
  { expression: z.string() },
  async (args) => {
    const result = eval(args.expression);
    return {
      content: [{ type: "text", text: `Result: ${result}` }]
    };
  }
);

const calculator = createSdkMcpServer({
  name: "calculator",
  version: "1.0.0",
  tools: [calculate]
});

const result = query({
  prompt: "What's 123 * 456?",
  options: {
    mcpServers: { calc: calculator },
    allowedTools: ["mcp__calc__calculate"]
  }
});
```

## フック

ツール実行の前後にカスタムコードを実行できます。

### Python

```python
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher

async def validate_bash_command(input_data, tool_use_id, context):
    """危険なコマンドをブロック"""
    if input_data['tool_name'] == 'Bash':
        command = input_data['tool_input'].get('command', '')
        if 'rm -rf /' in command:
            return {
                'hookSpecificOutput': {
                    'hookEventName': 'PreToolUse',
                    'permissionDecision': 'deny',
                    'permissionDecisionReason': 'Dangerous command blocked'
                }
            }
    return {}

async def log_tool_use(input_data, tool_use_id, context):
    """ツール使用をログ"""
    print(f"Tool used: {input_data.get('tool_name')}")
    return {}

options = ClaudeAgentOptions(
    hooks={
        'PreToolUse': [
            HookMatcher(matcher='Bash', hooks=[validate_bash_command]),
            HookMatcher(hooks=[log_tool_use])
        ],
        'PostToolUse': [
            HookMatcher(hooks=[log_tool_use])
        ]
    }
)
```

### フックイベント

| イベント | 説明 |
|----------|------|
| `PreToolUse` | ツール実行前 |
| `PostToolUse` | ツール実行後 |
| `UserPromptSubmit` | ユーザープロンプト送信時 |
| `SessionStart` | セッション開始時 |
| `SessionEnd` | セッション終了時 |
| `Stop` | 実行停止時 |
| `SubagentStop` | サブエージェント停止時 |
| `PreCompact` | メッセージ圧縮前 |

## メッセージ型

### Python

```python
from claude_agent_sdk import (
    query,
    AssistantMessage,
    UserMessage,
    SystemMessage,
    ResultMessage,
    TextBlock,
    ToolUseBlock,
    ToolResultBlock
)

async for message in query(prompt="Hello"):
    if isinstance(message, AssistantMessage):
        for block in message.content:
            if isinstance(block, TextBlock):
                print(f"Text: {block.text}")
            elif isinstance(block, ToolUseBlock):
                print(f"Tool: {block.name}, Input: {block.input}")
    elif isinstance(message, ResultMessage):
        print(f"Done! Cost: ${message.total_cost_usd}")
        print(f"Tokens: {message.usage}")
```

### TypeScript

```typescript
import { query, SDKAssistantMessage, SDKResultMessage } from "@anthropic-ai/claude-agent-sdk";

for await (const message of result) {
  if (message.type === 'assistant') {
    for (const block of message.message.content) {
      if (block.type === 'text') {
        console.log(`Text: ${block.text}`);
      } else if (block.type === 'tool_use') {
        console.log(`Tool: ${block.name}`);
      }
    }
  } else if (message.type === 'result') {
    console.log(`Done! Cost: $${message.total_cost_usd}`);
  }
}
```

## 権限コールバック

### Python

```python
async def custom_permission_handler(tool_name, input_data, context):
    """カスタム権限ロジック"""
    
    # システムディレクトリへの書き込みをブロック
    if tool_name == "Write" and input_data.get("file_path", "").startswith("/system/"):
        return {
            "behavior": "deny",
            "message": "System directory write not allowed",
            "interrupt": True
        }
    
    # それ以外は許可
    return {
        "behavior": "allow",
        "updatedInput": input_data
    }

options = ClaudeAgentOptions(
    can_use_tool=custom_permission_handler,
    allowed_tools=["Read", "Write", "Edit"]
)
```

### TypeScript

```typescript
const result = query({
  prompt: "Modify files",
  options: {
    canUseTool: async (toolName, input) => {
      if (toolName === "Write" && input.file_path?.startsWith("/system/")) {
        return {
          behavior: "deny",
          message: "System directory write not allowed"
        };
      }
      return { behavior: "allow", updatedInput: input };
    }
  }
});
```

## サンドボックス設定

```python
from claude_agent_sdk import query, ClaudeAgentOptions

options = ClaudeAgentOptions(
    sandbox={
        "enabled": True,
        "autoAllowBashIfSandboxed": True,
        "excludedCommands": ["docker"],
        "network": {
            "allowLocalBinding": True,
            "allowUnixSockets": ["/var/run/docker.sock"]
        }
    }
)
```

## エラーハンドリング

### Python

```python
from claude_agent_sdk import (
    query,
    CLINotFoundError,
    ProcessError,
    CLIJSONDecodeError
)

try:
    async for message in query(prompt="Hello"):
        print(message)
except CLINotFoundError:
    print("Please install Claude Code: npm install -g @anthropic-ai/claude-code")
except ProcessError as e:
    print(f"Process failed with exit code: {e.exit_code}")
except CLIJSONDecodeError as e:
    print(f"Failed to parse response: {e}")
```

## サブエージェントの定義

```python
options = ClaudeAgentOptions(
    agents={
        "code_reviewer": {
            "description": "Reviews code for bugs and best practices",
            "prompt": "You are an expert code reviewer. Focus on bugs, security issues, and best practices.",
            "tools": ["Read", "Grep", "Glob"],
            "model": "sonnet"
        },
        "test_writer": {
            "description": "Writes unit tests for code",
            "prompt": "You are an expert at writing comprehensive unit tests.",
            "tools": ["Read", "Write", "Bash"],
            "model": "sonnet"
        }
    }
)
```

## 構造化出力

```python
options = ClaudeAgentOptions(
    output_format={
        "type": "json_schema",
        "schema": {
            "type": "object",
            "properties": {
                "bugs": {
                    "type": "array",
                    "items": {
                        "type": "object",
                        "properties": {
                            "file": {"type": "string"},
                            "line": {"type": "integer"},
                            "description": {"type": "string"}
                        }
                    }
                },
                "summary": {"type": "string"}
            },
            "required": ["bugs", "summary"]
        }
    }
)
```

## ベストプラクティス

1. **最小限のツール権限**: 必要なツールのみを許可する
2. **適切な権限モード**: 本番環境では `acceptEdits` を慎重に使用
3. **エラーハンドリング**: すべての例外を適切に処理
4. **タイムアウト設定**: 長時間実行タスクにはタイムアウトを設定
5. **ログ記録**: フックを使用してツール使用をログ
6. **サンドボックス**: 信頼できない入力にはサンドボックスを使用

## 参考リンク

- [Agent SDK 概要](https://platform.claude.com/docs/ja/agent-sdk/overview)
- [クイックスタート](https://platform.claude.com/docs/ja/agent-sdk/quickstart)
- [Python SDK リファレンス](https://platform.claude.com/docs/ja/agent-sdk/python)
- [TypeScript SDK リファレンス](https://platform.claude.com/docs/ja/agent-sdk/typescript)
- [フックガイド](https://platform.claude.com/docs/ja/agent-sdk/hooks)
- [権限ガイド](https://platform.claude.com/docs/ja/agent-sdk/permissions)
- [サンプルエージェント](https://github.com/anthropics/claude-agent-sdk-demos)
