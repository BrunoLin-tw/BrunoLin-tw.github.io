---
title: "AI Harness Engineering：給 AI 代理套上韁繩的工程學"
date: 2026-03-30 00:00:00 +0800
categories:
  - AI
  - tech
tags:
  - AI-Agent
  - Harness-Engineering
  - OpenAI
---

最近看到「Harness Engineering」（韁繩工程）這個新詞，覺得很有趣。簡單說，這是 AI 代理開發裡的新概念，不是教 AI 怎麼想，而是建一個「安全框架」，讓 AI 在長任務中不亂跑、不忘記目標、不出大錯。就像給賽馬手準備韁繩、安全帶和維修站，讓他專心比賽。

## 為什麼需要它？

傳統 AI 開發：
- **Prompt Engineering**：給 AI 好提示，讓它「知道怎麼想」。
- **Context Engineering**：餵對資料，讓它「看到什麼」。

但 AI 代理做複雜事（如寫完整程式、分析專案）常失敗：忘記前步、違規操作、卡死迴圈。這時 Harness Engineering 上場：設計周圍環境，確保 AI 可靠執行。

2026 年 OpenAI/Anthropic 等大廠推廣，用在 Claude/Codex 等代理，成功率從 30% 跳到 90%。

## 四大核心元素（四象限）

我整理成表格，簡單明瞭：

| 象限       | 內容                     | 實際例子                  |
|------------|--------------------------|---------------------------|
| **工具整合** | 提供 AI 專用工具/CLI   | 檔案讀寫、命令執行，但限權限 |
| **狀態管理** | 追蹤進度、存中間結果   | JSONL 記錄「已完成步驟」，防重工 |
| **約束驗證** | 防亂來（測試、檢查）   | 每步跑 linter、權限檢查 |
| **環境架構** | 模擬真實環境            | Docker 沙箱 + 持續回饋 |

## 我的看法

對嵌入式 AI 工程師如我，這很像 NPU 部署：AI 模型是核心，但周邊韁繩（驅動、記憶體管理、安全機制）決定成敗。未來建 AI 工具時，我會先想「韁繩怎麼套」。

## 參考資源
1. [OpenAI 原文章](https://openai.com/index/harness-engineering/)
2. [Martin Fowler 解釋](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html)
3. [Louis Bouchard 實戰](https://www.louisbouchard.ai/harness-engineering/)

（資料來源：Tavily 搜尋，2026/3/29）

有興趣試試？留言討論！
