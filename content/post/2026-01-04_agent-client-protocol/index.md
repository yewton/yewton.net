---
title: "Agent Client Protocol を理解する"
author: ["yewton"]
date: 2026-01-04
mylastmod: 2026-01-07T23:49:39+09:00
slug: "agent-client-protocol"
tags: ["ACP", "Agent", "LSP", "Emacs", "agent-shell"]
categories: ["技術系"]
draft: true
image:
  caption: Background image by
---

<div class="ox-hugo-toc toc">

<div class="heading">&#30446;&#27425;</div>

- [はじめに](#はじめに)
- [背景: なぜ ACP が必要なのか](#背景-なぜ-acp-が必要なのか)
- [主な特徴とコンセプト](#主な特徴とコンセプト)
- [実践: `agent-shell.el` での利用例](#実践-agent-shell-dot-el-での利用例)
    - [Initialization](#initialization)
    - [Session Setup](#session-setup)
    - [Prompt Turn](#prompt-turn)
- [将来性](#将来性)
    - [GitHub Copilot CLI でのサポート](#github-copilot-cli-でのサポート)
    - [ドラフト仕様の展望](#ドラフト仕様の展望)
- [まとめ](#まとめ)

</div>
<!--endtoc-->


## はじめに {#はじめに}

Agent Client Protocol (ACP) は、AI エージェントとクライアント（エディタなど）が対話する際の標準を定義するプロトコルです。
この記事では、ACP の概要と、その背景や具体的な利用例、そして将来性について、自身の理解を元に解説します。

公式ドキュメントは [こちら](https://agentclientprotocol.com/overview/introduction) にあります。


## 背景: なぜ ACP が必要なのか {#背景-なぜ-acp-が必要なのか}

2025年8月30日、 Zed から以下のように発表がありました:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">This week, we announced ACP, an open-source protocol that standardizes communication between clients and coding agents, but we also released Zed v0.201! Let&#39;s dive into some of the highlights!<a href="https://t.co/dHIcUWRNUs">https://t.co/dHIcUWRNUs</a></p>&mdash; Zed (@zeddotdev) <a href="https://twitter.com/zeddotdev/status/1961502810068979799?ref_src=twsrc%5Etfw">August 29, 2025</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

同年10月7日には、 JetBrains から Zed と連携して ACP に取り組むことが発表されました:

[JetBrains × Zed: Open Interoperability for AI Coding Agents in Your IDE | The JetBrains AI Blog](https://blog.jetbrains.com/ai/2025/10/jetbrains-zed-open-interoperability-for-ai-coding-agents-in-your-ide/)

同年12月5日に発表された [Bring your own AI agent to JetBrains IDEs | The JetBrains AI Blog](https://blog.jetbrains.com/ai/2025/12/bring-your-own-ai-agent-to-jetbrains-ides/#behind-the-closed-doors) では、こんなことが語られています:

> \## Behind the closed doors
>
> Before we wrap up, I want to share a bit of behind-the-scenes context on how JetBrains came to ACP. Real life is a bit different, and I think it’s always nice to pull back the curtain and show you how it works from the inside. So, here we go.
>
> This summer we started working on bringing Junie (it’s a coding agent developed by JetBrains) into the same UI that we had for the AI chat. After some debates, we built a protocol that allowed us to communicate between IDE and local and remote versions of Junie. We were about to publish that work to the internet, but almost at the same time, [Zed](https://zed.dev/) announced its own protocol. We chatted with lovely Zed people and agreed that instead of bringing two competitive standards to the market, we’d join our efforts on building a single one.

JetBrains のブログ記事では、ACP 策定の経緯について率直に語られています。
当初から「エディタ非依存の標準」というような高尚な目標があったわけではなく、JetBrains が自社エージェント統合用のプロトコルを公開しようとしたタイミングで Zed も同様の動きを見せていたため、規格の乱立を避けるべく協力することになった、というのが実情のようです。
各ベンダーが「エージェントを効率よく統合したい」という現実的な課題を持っていたタイミングが重なったことが、結果として標準化を加速させたと言えるでしょう。

LSP (Language Server Protocol) がプログラミング言語ごとの開発体験の差異を吸収したように、ACP は乱立する AI エージェントとそのクライアント間の通信を標準化することを目指しています。


## 主な特徴とコンセプト {#主な特徴とコンセプト}

ACP は、タスクの実行、進捗の報告、対話的なデータ交換など、エージェントとのやり取りに必要な一連の機能を定義しています。
具体的な仕様は公式サイトに譲りますが、ここでは特に重要と思われる以下のコンセプトについて触れます。

-   エージェントのライフサイクル管理
-   タスクの委任と実行コンテキスト
-   ストリーミングレスポンスと非同期処理

TBW シーケンス図載せる
<https://agentclientprotocol.com/protocol/overview#message-flow> 引用

Protocol 配下全部並列で分かりづらいが、
init, settup, prompt までが、 overview のメッセージフローに対応、 Schema はその通りスキーマ定義、それ以外は個々のコンセプトの説明。


## 実践: `agent-shell.el` での利用例 {#実践-agent-shell-dot-el-での利用例}

ACP の具体的な動作を理解するために、Emacs のクライアント実装である [agent-shell.el](https://github.com/simon-katz/agent-shell) を使って、実際の通信内容を見ていきます。

ここでは、簡単なタスクをエージェントに依頼し、その際の ACP メッセージ（リクエストとレスポンス）をキャプチャして、プロトコルの使われ方を具体的に示します。

`agent-shell-toggle-logging` で、ログ出力の有無を設定出来ます。


### Initialization {#initialization}

```js
{
  "jsonrpc": "2.0",
  "method": "initialize",
  "id": 1,
  "params": {
    "protocolVersion": 1,
    "clientCapabilities": {
      "fs": {
        "readTextFile": true,
        "writeTextFile": true
      }
    }
  }
}
```

```js
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": 1,
    "authMethods": [
      {
        "id": "oauth-personal",
        "name": "Log in with Google",
        "description": null
      },
      {
        "id": "gemini-api-key",
        "name": "Use Gemini API key",
        "description": "Requires setting the `GEMINI_API_KEY` environment variable"
      },
      {
        "id": "vertex-ai",
        "name": "Vertex AI",
        "description": null
      }
    ],
    "agentCapabilities": {
      "loadSession": false,
      "promptCapabilities": {
        "image": true,
        "audio": true,
        "embeddedContext": true
      }
    }
  }
}
```

```js
{
  "jsonrpc": "2.0",
  "method": "authenticate",
  "id": 2,
  "params": {
    "methodId": "oauth-personal",
    "authMethod": {
      "id": "oauth-personal",
      "name": "Log in with Google",
      "description": ""
    }
  }
}
```

```js
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": null
}
```


### Session Setup {#session-setup}

```js
{
  "jsonrpc": "2.0",
  "method": "session/new",
  "id": 3,
  "params": {
    "cwd": "/home/yewton/Projects/yewton.net/",
    "mcpServers": []
  }
}
```

```js
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "sessionId": "36702bdd-86f6-4e87-b964-c2328216f375"
  }
}
```


### Prompt Turn {#prompt-turn}

```js
{
  "jsonrpc": "2.0",
  "method": "session/prompt",
  "id": 5,
  "params": {
    "sessionId": "36702bdd-86f6-4e87-b964-c2328216f375",
    "prompt": [
      {
        "type": "text",
        "text": "以下を日本語に翻訳して:\n\ncontent-org/content-org/all-posts.org:1614-1614"
      }
    ]
  }
}
```

```js
{
  "jsonrpc": "2.0",
  "method": "session/update",
  "params": {
    "sessionId": "36702bdd-86f6-4e87-b964-c2328216f375",
    "update": {
      "sessionUpdate": "agent_thought_chunk",
      "content": {
        "type": "text",
        "text": "**Examining the File's Contents**\n\nI'm now reading the file around line 1614 to grasp the full context. This step is crucial for accurate translation and the subsequent `replace` operation. It allows me to pinpoint the text requiring translation and prepare the `old_string` for the `replace` tool.\n\n\n"
      }
    }
  }
}
```

```js
{
  "jsonrpc": "2.0",
  "method": "session/update",
  "params": {
    "sessionId": "36702bdd-86f6-4e87-b964-c2328216f375",
    "update": {
      "sessionUpdate": "agent_message_chunk",
      "content": {
        "type": "text",
        "text": "I will read the file `content-org/all-posts.org` around line 1614 to identify the text to translate."
      }
    }
  }
}
```

突然英語になったりする。

```js
{
  "jsonrpc": "2.0",
  "method": "session/update",
  "params": {
    "sessionId": "36702bdd-86f6-4e87-b964-c2328216f375",
    "update": {
      "sessionUpdate": "tool_call",
      "toolCallId": "read_file-1767793979301",
      "status": "in_progress",
      "title": "content-org/all-posts.org",
      "content": [],
      "locations": [
        {
          "path": "/home/yewton/Projects/yewton.net/content-org/all-posts.org",
          "line": 1610
        }
      ],
      "kind": "read"
    }
  }
}
```

この時点では fs/read_text_file は使われない。

```js
{
  "jsonrpc": "2.0",
  "method": "session/update",
  "params": {
    "sessionId": "36702bdd-86f6-4e87-b964-c2328216f375",
    "update": {
      "sessionUpdate": "agent_message_chunk",
      "content": {
        "type": "text",
        "text": "「こんにちは、世界！」に翻訳して、次の段落に追加します。"
      }
    }
  }
}
```

```js
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "fs/read_text_file",
  "params": {
    "path": "/home/yewton/Projects/yewton.net/content-org/all-posts.org",
    "sessionId": "36702bdd-86f6-4e87-b964-c2328216f375",
    "line": null,
    "limit": null
  }
}
```

書き込みに伴う読み込みには fs/read_text_file が使われる模様。

```js
{
  "jsonrpc": "2.0",
  "id": 5,
  "method": "session/request_permission",
  "params": {
    "sessionId": "36702bdd-86f6-4e87-b964-c2328216f375",
    "options": [
      {
        "optionId": "proceed_always",
        "name": "Allow All Edits",
        "kind": "allow_always"
      },
      {
        "optionId": "proceed_once",
        "name": "Allow",
        "kind": "allow_once"
      },
      {
        "optionId": "cancel",
        "name": "Reject",
        "kind": "reject_once"
      }
    ],
    "toolCall": {
      "toolCallId": "replace-1767793984780",
      "status": "pending",
      "title": "content-org/all-posts.org: Hello, World! => Hello, World!",
      "content": [
        {
          "type": "diff",
          "path": "all-posts.org",
          "oldText": "...",
          "newText": "..."
        }
      ],
      "locations": [
        {
          "path": "/home/yewton/Projects/yewton.net/content-org/all-posts.org"
        }
      ],
      "kind": "edit"
    }
  }
}
```

```text
:direction outgoing
:kind      response
:object
           jsonrpc 2.0
           id      5
           result
                   outcome
                           outcome  selected
                           optionId proceed_once
```

この応答は生ログが出ていなかった。

```js
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "fs/read_text_file",
  "params": {
    "path": "/home/yewton/Projects/yewton.net/content-org/all-posts.org",
    "sessionId": "36702bdd-86f6-4e87-b964-c2328216f375",
    "line": null,
    "limit": null
  }
}
```

改めて読んで、

```js
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "fs/write_text_file",
  "params": {
    "path": "/home/yewton/Projects/yewton.net/content-org/all-posts.org",
    "content": "...ファイルの中身全量...",
    "sessionId": "36702bdd-86f6-4e87-b964-c2328216f375"
  }
}
```

書き込んで終わり。

注意: 必ずファイル全量読み書きになるらしい


## 将来性 {#将来性}

ACP はまだ発展途上のプロトコルですが、その将来性は非常に大きいと考えています。


### GitHub Copilot CLI でのサポート {#github-copilot-cli-でのサポート}

2026年1月7日現在、まだ公式ドキュメントには記載されていませんが、[GitHub Copilot CLI](https://github.com/github/gh-copilot) には、ACP を利用するための \`=--acp=\` オプションが実験的に追加されています。
これにより、将来的には Copilot も ACP 準拠のエージェントとして、様々なクライアントから統一的に利用できるようになる可能性があります。

[Support for ACP (Agent Client Protocol) #222](https://github.com/github/copilot-cli/issues/222)


### ドラフト仕様の展望 {#ドラフト仕様の展望}

現在議論されているドラフト仕様の中でも、特に以下の 2 つはエージェントの能力を飛躍的に向上させる可能性を秘めています。

-   **Forking of existing sessions**: 既存のセッションを複製し、異なるコンテキストでタスクを並行実行させる機能。
-   **Proxy Chains**: 複数のエージェントを連携させ、より複雑なタスクを分業させる仕組み。

これらの機能は、[Agent Skills](https://agentskills.io/home) のような外部スキルセットとの連携や、特定のタスクに特化したサブエージェントの活用といった、より高度なエージェントアーキテクチャへの道を開くものと期待しています。


## まとめ {#まとめ}

Agent Client Protocol は、AI エージェント開発の未来を形作る重要なプロトコルです。LSP が開発環境にもたらしたような変革を、ACP が AI エージェントとの連携にもたらすことを期待しています。
本記事が、ACP への理解を深める一助となれば幸いです。
