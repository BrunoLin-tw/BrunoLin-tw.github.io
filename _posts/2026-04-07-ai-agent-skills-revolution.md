---
title: \"AI Coding Agents 的生產力神器：addyosmani/agent-skills 技能包推薦\"
date: 2026-04-07 10:00:00 +0800
categories:
  - tech
tags:
  - ai
  - coding-agent
  - claude-code
  - agent-skills
  - productivity
---

最近發現一個超實用的 GitHub repo：**[addyosmani/agent-skills](https://github.com/addyosmani/agent-skills)**，由 Google 前資深工程師 Addy Osmani 出品，專為 AI coding agents (Claude Code、Cursor、Gemini CLI 等) 設計的 **19 個 production-grade 技能包**。這不是泛泛提示詞，而是資深工程師的結構化 workflow，能讓 AI 從「原型亂寫」升級到「生產級可靠程式碼」，強制走 Define → Plan → Build → Verify → Review → Ship 全生命週期。

## 背景：AI Coding 的痛點與解決方案

AI agents 寫 code 超快，但常忽略 spec、test、review、security 等環節，導致產出不穩、難維護。agent-skills 用 **slash commands** (/spec /plan /build 等) 自動觸發技能，內建 Google 工程文化 (Hyrum's Law、test pyramid、trunk-based dev、OWASP 等)，加上 **反懶惰設計**：每個 SKILL.md 有 \"Rationalizations\" 表，擋住「tests later」這種藉口，並要求 **verifiable evidence** (e.g. tests pass、build output) 才過關。

![agent-skills-toolkit]({{ site.url }}/assets/images/posts/2026-04-07-agent-skills-toolkit.png)

*(Nano Banana Pro 生成的視覺化：AI agent 從混亂碼堆變身結構化生產線，凸顯技能包對 agentic coding 的革命性實用。)*

## 核心技能特點

19 個技能分 6 大階段，每個有 **step-by-step process + red flags + verification**：

### Define (定義)
- **idea-refine**：模糊概念 → 具體提案 (divergent/convergent thinking)
- **spec-driven-development**：寫 PRD (objectives、API、測試規範) 前不許動 code

### Plan (規劃)
- **planning-and-task-breakdown**：spec 拆成 atomic tasks + deps + acceptance criteria

### Build (建置)
- **incremental-implementation**：thin slices + feature flags
- **test-driven-development**：Red-Green-Refactor + pyramid (80/15/5)
- **frontend-ui-engineering**：component arch + accessibility (WCAG AA)
- **api-and-interface-design**：contract-first + One-Version Rule
- **context-engineering**：最佳 context 餵 agent

### Verify (驗證)
- **browser-testing-with-devtools**：Chrome DevTools MCP 即時 debug
- **debugging-and-error-recovery**：5 步 triage (reproduce → fix → guard)

### Review (審核)
- **code-review-and-quality**：5-axis review + change sizing (~100 lines)
- **code-simplification**：Chesterton's Fence，簡化不改行為
- **security-and-hardening**：OWASP Top 10 + secrets mgmt
- **performance-optimization**：Core Web Vitals + profiling

### Ship (部署)
- **git-workflow-and-versioning**：atomic commits + trunk-based
- **ci-cd-and-automation**：Shift Left + quality gates
- **deprecation-and-migration**：code as liability
- **documentation-and-adrs**：ADRs + why
- **shipping-and-launch**：rollout + rollback

還有 3 個 specialist agents (code-reviewer、test-engineer、security-auditor) + checklists (testing/security/perf/a11y)。

**安裝超簡單**：
- Claude Code：`/plugin marketplace add addyosmani/agent-skills`
- Cursor：copy SKILL.md 到 .cursor/rules/
- OpenClaw 等：Markdown 相容，直接用。

## 結論

agent-skills 讓 AI coding 從「有趣玩具」變「可靠夥伴」，token 高效 + 品質飛躍。強推給用 Claude Code/Cursor 的你，先 clone repo 試 /spec 一個 feature。未來 AI 工程不可缺！🐻

#agentic #coding #skills
