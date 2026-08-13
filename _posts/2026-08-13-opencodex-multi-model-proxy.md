---
title: "OpenCodex：保留 Codex 工作流，自由切換各家模型"
date: 2026-08-13 23:30:48 +0800
categories:
  - tech
tags:
  - codex
  - opencodex
  - opencode-go
  - coding-agent
  - ai-agents
  - llm
---

我平常會用 OpenAI Codex 處理程式修改、除錯和 repo 分析。Codex 的 Agent 工作流做得完整，但模型選擇主要集中在 OpenAI 生態系。OpenCodex 提供另一條路：保留 Codex 的操作介面、sandbox、工具呼叫與專案規則，只替換背後執行推理的模型。

![OpenCodex 位於 Codex 與各模型服務之間，負責代理、協定轉換與模型路由](/assets/images/posts/opencodex-proxy-architecture.png)

## OpenCodex 的角色

幾個相近名稱很容易混淆：

- Codex：OpenAI 的 coding agent
- OpenCode：Anomaly 開發的開源 coding agent
- OpenCode Go：OpenCode 推出的低價多模型訂閱
- OpenCodex：讓 Codex、Claude Code 等工具連接不同模型的本機代理

OpenCodex 預設在 `localhost:10100` 啟動 Proxy。它接收 Codex 的 Responses API 請求，再轉換成各家 Provider 使用的協定，包括 OpenAI Chat Completions、Responses API 與 Anthropic Messages API。

```text
Codex CLI / App
      ↓
OpenCodex Proxy
      ↓
Claude / Gemini / DeepSeek / Kimi
OpenRouter / OpenCode Go / Ollama
```

Codex 繼續管理 Agent loop、專案檔案、shell command、sandbox 和 MCP 工具；OpenCodex 負責連線、協定轉換與模型路由。這個分工讓模型成為可替換元件。

## 支援哪些 Provider？

OpenCodex 內建數十種 Provider，常見選項包括 OpenAI、Anthropic Claude、Google Gemini、xAI Grok、DeepSeek、Kimi、GLM、Qwen、OpenRouter、Mistral、NVIDIA NIM、Azure OpenAI 與 OpenCode Go。

本機推論服務也能接入：

| 服務 | 預設 Endpoint |
|---|---|
| Ollama | `http://localhost:11434/v1` |
| vLLM | `http://localhost:8000/v1` |
| LM Studio | `http://localhost:1234/v1` |

其他服務只要提供 OpenAI-compatible Chat Completions API，通常可以用 Custom Provider 連接。

## 安裝與初始設定

環境需要 Node.js 18 以上，並先安裝 Codex：

```bash
node --version
codex --version
npm install -g @bitkyc08/opencodex
```

啟動 Proxy 與 Dashboard：

```bash
ocx start
ocx gui
```

Dashboard 預設位址是：

```text
http://localhost:10100
```

第一次使用再執行：

```bash
ocx init
```

它會建立 `~/.opencodex/config.json`，並調整 `~/.codex/config.toml` 和 Codex model catalog。Provider 可以直接在 Dashboard 新增，填入 API key 或完成 OAuth 後，再同步模型：

```bash
ocx sync
```

停止 OpenCodex 時，它會還原自己管理的 Codex 設定：

```bash
ocx stop
```

完整移除則執行：

```bash
ocx uninstall
npm uninstall -g @bitkyc08/opencodex
```

## 從 Codex 選擇模型

OpenCodex 使用 `provider/model` 指定路由：

```bash
codex -m "anthropic/claude-opus-5"
codex -m "google/gemini-3-pro"
codex -m "ollama/qwen3-coder"
codex -m "openrouter/<model-id>"
```

實際 model ID 應以 Dashboard 或模型清單為準。我會優先寫出完整的 Provider 名稱，避免多個 Provider 同時存在時把 request 送錯地方。

## 搭配 OpenCode Go

OpenCode Go 很適合用來測試多模型工作流。它採月費制，首月 US$5，之後每月 US$10，提供一組 API key 與一批針對 coding agent 測試過的模型，目前包括：

- GPT 5.6 Luna、Grok 4.5
- GLM-5.2、Kimi K2.7 Code
- DeepSeek V4 Pro／Flash
- MiniMax M3、Qwen3 系列與 MiMo V2.5

在 Dashboard 新增 OpenCode Go Provider、貼上 API key，再執行 `ocx sync`，便可以從 Codex 啟動：

```bash
codex -m "opencode-go/kimi-k2.7-code"
codex -m "opencode-go/deepseek-v4-pro"
codex -m "opencode-go/minimax-m3"
codex -m "opencode-go/gpt-5.6-luna"
```

這些模型使用的 API wire 不完全相同。GPT 5.6 Luna 走 Responses API，MiniMax 和部分 Qwen 模型走 Anthropic Messages API，其餘多數使用 Chat Completions。OpenCodex 內建了對應規則，直接選擇 OpenCode Go Provider 比手動建立 Custom Provider 穩定。

Go 的月費仍有額度限制：

| 週期 | 包含額度 |
|---|---:|
| 每 5 小時 | US$12 usage |
| 每週 | US$30 usage |
| 每月 | US$60 usage |

不同模型的價格差距很大，因此可用 request 數也不同。OpenCodex 2.14 之後能讀取 Go 的 quota API，在 Dashboard 顯示各週期用量。

## 日常管理

```bash
ocx status          # Proxy 狀態
ocx health          # 健康檢查
ocx doctor          # 設定與連線診斷
ocx provider list   # Provider 清單
ocx sync            # 同步模型
ocx update          # 更新 OpenCodex
```

需要常駐背景執行，可以安裝 service：

```bash
ocx service install
ocx service start
ocx service status
```

如果不想長期執行背景服務，可以安裝 shim，等 `codex` 啟動時再喚醒 Proxy：

```bash
ocx codex-shim install
```

## Credential 與資料風險

API key 可以在設定檔引用環境變數：

```json
{
  "apiKey": "${OPENROUTER_API_KEY}"
}
```

`~/.opencodex/config.json`、`~/.opencodex/auth.json` 和 `.env` 都不該進入 Git。Dashboard 的 request log 也可能包含程式片段、檔案內容與 command output。

OpenCodex 是第三方社群專案，沒有獲得 OpenAI、Anthropic 或其他 Provider 正式背書。導入公司專案前，需要確認資料會送到哪裡、保留多久，以及 Provider 是否允許透過第三方 Proxy 使用帳號。機密程式碼應採正式 API、企業帳號或內部推論服務。

遠端使用也要多一層防護。OpenCodex 預設只監聽 loopback；若改綁 `0.0.0.0`，必須設定 `OPENCODEX_API_AUTH_TOKEN`，不能直接把 Proxy 暴露到網路上。

## 我會怎麼分配模型？

我的初步做法是先依任務成本和難度分層：

```text
日常修改、文件與明確任務
  → DeepSeek V4 Flash / MiMo V2.5 / MiniMax M3

大型 repo、複雜除錯
  → Kimi K2.7 Code / GLM-5.2 / GPT 5.6 Luna

高風險或高價值任務
  → 官方 Codex 模型或實測最穩定的 Provider
```

真正要追蹤的是任務完成率、平均重試次數、Tool Calling 失敗率，以及測試失敗後能不能修正方向。便宜模型如果反覆重試，總成本未必比較低。

## 我的看法

OpenCodex 把 Codex 的 Agent 工作流和模型選擇拆成兩層。使用者可以繼續使用熟悉的 Codex 介面、sandbox、MCP 和專案規則，再依任務選擇模型。OpenCode Go 則提供一個低門檻的多模型入口，適合驗證這套 routing 工作方式。

我目前會先放在個人專案、開源 repo 和非機密工作。公司程式碼要等資料政策、服務條款與 credential 管理都確認後再導入。

模型切換很容易。切換之後，Agent 能否穩定完成整個任務，才是評估 OpenCodex 的重點。

Reference:

- [OpenCodex GitHub](https://github.com/lidge-jun/opencodex)
- [OpenCodex Documentation](https://opencodex.me/)
- [OpenCodex Installation](https://opencodex.me/getting-started/installation/)
- [OpenCodex Providers](https://opencodex.me/guides/providers/)
- [OpenCodex Model Routing](https://opencodex.me/guides/model-routing/)
- [OpenCodex Configuration](https://opencodex.me/reference/configuration/)
- [OpenCode Go](https://opencode.ai/go)
- [OpenCode Go Documentation](https://opencode.ai/docs/go)
