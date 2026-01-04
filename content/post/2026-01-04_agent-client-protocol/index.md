---
title: "Agent Client Protocol を理解する"
author: ["yewton"]
date: 2026-01-04
mylastmod: 2026-01-04T21:36:51+09:00
slug: "agent-client-protocol"
categories: ["技術系"]
draft: true
image:
  caption: Background image by
---

Agent Client Protocol ( <https://agentclientprotocol.com/overview/introduction> ) について、自分なりの理解を解説する記事。

具体的な仕様は公式サイトにあるのでそこは深く語らず、実際に agent-shell.el で試してみて、実際の通信内容と共に利用例を載せたい。

また、標準成立に至る背景や、 Zed や JetBrains の関わりにも触れ、一過性の流行ではないことを語りたい。

GitHub Copilot CLI にも、ドキュメントにはまだ載っていないが実は `acp` オプションが追加されており、 agent-shell の設定をすれば動くことを示して、将来性の根拠としたい。

そして、掲載中のドラフトの内、特に Forking of existing sessions と Proxy Chains については、サブエージェントの活用や Agent Skills  ( <https://agentskills.io/home> ) への対応へと繋がりそうなので個人的に期待していることを伝えたい。
