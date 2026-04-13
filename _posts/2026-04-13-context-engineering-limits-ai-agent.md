---
title: "真正限制 AI Agent 的，往往不是模型，而是 Context Engineering"
date: 2026-04-13 09:00:00 +0800
categories:
  - AI
  - tech
tags:
  - AI-Agent
  - Context-Engineering
  - LLM
  - Agentic-Development
  - Prompt-Engineering
---
![Context Engineering Visual](/assets/images/posts/2026-04-13-context-engineering-hero.svg)

許多人把 AI agent 的失敗歸因於模型不夠強，但實際上更常見的問題是 **context 沒設計好**——資訊太雜、重點不清、缺乏驗證機制，導致 agent 開始飄移、忘關鍵條件或一路走錯方向。

真正的 context engineering 重點不是盲目壓縮 token，而是提高 **semantic density（語意密度）**：保留高價值資訊，去除噪音。同時需要結構化的上下文（專案背景、任務目標、核心檔案、規範、驗證點），以及類似 *gather → act → verify → repeat* 的迴圈，讓 agent 能在過程中補 context、修正方向、驗證結果。

換句話說，模型是引擎，context engineering 才是導航系統、儀表板與護欄。
未來評比 AI agent，我反而更想問：**你的 context 是怎麼設計的？**

#### 關鍵參考來源
- Anthropic, *Building agents with the Claude Agent SDK* – https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk
- Anthropic Engineering – https://www.anthropic.com/engineering
- *Beyond Human-Readable: Rethinking Software Engineering Conventions for the Agentic Development Era* – https://arxiv.org/html/2604.07502v1
