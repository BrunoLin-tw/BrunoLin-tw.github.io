---
title: "DeepSeek V4 正式版：Flash 當主力，Pro 處理難題"
date: 2026-08-13 23:04:30 +0800
categories:
  - tech
tags:
  - deepseek
  - llm
  - ai-agents
  - coding-agent
  - open-source-ai
---

DeepSeek 在四月推出 V4 Preview，現在兩個正式版本都到齊了：7 月 31 日的 V4-Flash-0731，以及 8 月 13 日正式 GA 的 V4-Pro-0813。

我第一眼看的依然是 Agent 能力。這次更新的方向很清楚：Flash 負責大量日常工作，Pro 處理更複雜、更容易卡住的任務。

## 兩個模型怎麼分？

V4-Flash 採用 MoE 架構，總參數 284B，每次推理啟用 13B。V4-Pro 則擴大到 1.6T 總參數、49B 啟用參數。

兩者都支援：

- 1M token context
- 最大 384K token 輸出
- 非思考與思考模式
- `low`、`high`、`max` 三種 reasoning effort
- Tool Calling、JSON Output、Responses API
- OpenAI 與 Anthropic 相容介面

DeepSeek 也針對 Codex、Claude Code、OpenCode 等 coding agent 做了整合。這一點比模型多拿幾分 benchmark 更實際，因為 Agent 能不能穩定延續推理、正確呼叫工具、處理長時間任務，才會直接影響工作流。

簡單整理：

| 模型 | 規模 | 定位 | 適合場景 |
|---|---:|---|---|
| V4-Flash-0731 | 284B / 13B active | 快速、低成本、高吞吐 | 日常 coding、文件整理、搜尋、批次 Agent 任務 |
| V4-Pro-0813 | 1.6T / 49B active | 複雜推理與高難度 Agent | 大型 repo、架構分析、深度除錯、長鏈工具操作 |

## Flash 正式版進步最大的地方

V4-Flash-0731 沿用 Preview 的基礎架構，主要透過新的 post-training 提升 Agent 能力，並加入 DSpark speculative decoding。

官方公布的結果包括：

| Benchmark | Flash Preview | Flash 0731 |
|---|---:|---:|
| Terminal Bench 2.1 | 61.8 | 82.7 |
| DeepSWE | 7.3 | 54.4 |
| Toolathlon-Verified | 49.7 | 70.3 |
| DSBench-FullStack | 37.0 | 68.7 |

這個進步幅度相當大。Flash 甚至在多項測試超過四月的 V4-Pro Preview，說明模型規模只決定部分能力，後訓練、Agent harness 和推理配置同樣會大幅影響結果。

以工作流來看，Flash 已經有條件成為預設模型。日常修改程式、讀文件、整理資料或執行明確任務，沒必要每次都派出 Pro。

## Pro 0813 補上複雜任務能力

V4-Pro-0813 是正式 GA 版本，重點放在生產環境中的 Agent 表現。

官方成績如下：

| Benchmark | Pro Preview | Pro 0813 |
|---|---:|---:|
| Terminal Bench 2.1 | 72.1 | 87.9 |
| NL2Repo | 38.5 | 61.5 |
| DeepSWE | 12.8 | 62.7 |
| Cybergym | 52.7 | 83.3 |
| DSBench-Hard | 31.1 | 67.2 |

Pro 和 Flash 在簡單任務上的差距可能不明顯。任務規模變大、條件開始互相牽制，或 Agent 需要連續操作大量工具時，Pro 的優勢才會慢慢出現。

這也符合實際使用經驗。模型選型只看單次回答很容易誤判，Agent 工作要看它跑到第十次、第二十次工具呼叫時，還能不能記得目標、遵守限制並處理前面留下的狀態。

## Benchmark 要怎麼看？

這批數字很亮眼，但要保留一點距離。

DeepSeek 使用自家的 Agent harness，並以 `max` reasoning effort、`temperature=1.0` 和 `top_p=0.95` 執行部分測試。DSBench-FullStack 與 DSBench-Hard 也是內部測試集。

所以這些數字適合拿來觀察版本進步，不適合直接換算成自己的專案成功率。

真正導入工作流時，我會測這幾件事：

- 多輪工具呼叫是否穩定
- 大型 repo 裡能不能找到正確修改點
- 失敗後是否能根據測試結果修正方向
- 長 context 使用一段時間後會不會偏離目標
- 同一批任務的成功率、重試次數與總成本

這些結果通常比單一 benchmark 更接近實際價值。

## 價格即將調整

目前官方 API 價格如下，每百萬 token 計價：

| 模型 | Cache miss 輸入 | 輸出 |
|---|---:|---:|
| V4-Flash | US$0.14 | US$0.28 |
| V4-Pro | US$0.435 | US$0.87 |

Pro 大約是 Flash 的 3.1 倍。

台灣時間 8 月 17 日凌晨起，DeepSeek 將改採尖峰與離峰定價：

| 模型 | 時段 | 輸入 | 輸出 |
|---|---|---:|---:|
| V4-Flash | 離峰 | US$0.22 | US$0.66 |
| V4-Flash | 尖峰 | US$0.44 | US$1.32 |
| V4-Pro | 離峰 | US$0.66 | US$1.98 |
| V4-Pro | 尖峰 | US$1.32 | US$3.96 |

台灣尖峰時段為 09:00–12:00 與 14:00–18:00。大量批次工作如果可以排程，放到離峰執行會比較合理。

## 我的看法

DeepSeek V4 正式版最有吸引力的地方，是 Flash 和 Pro 都用相對低的成本，提供了可實際放進 Agent 工作流的能力。

Flash 適合大量日常任務，速度快、API concurrency 高，目前價格也相當低。Pro 的成本雖然較高，仍遠低於多數同級旗艦模型，適合大型 repo、複雜除錯、架構決策，以及重試代價較高的工作。

我會採用這樣的 routing：

```text
一般任務
  → V4-Flash low / high

任務失敗或複雜度提高
  → V4-Pro high

高價值、長鏈 Agent 任務
  → V4-Pro max
```

這樣的分工可以控制成本，也能在必要時提高任務成功率。Agent 一次任務可能呼叫模型幾十次，價格差異累積後會很明顯。

不過，DeepSeek 即將調高 API 價格，並導入尖峰與離峰計價。調整後，即使是離峰價格，Flash 與 Pro 的輸出費用也會超過目前兩倍；尖峰時段的漲幅更高。DeepSeek 目前最突出的低成本優勢，接下來會被削弱。

未來評估這兩個模型，不能只看單次 token 價格。任務成功率、平均重試次數、推理長度、快取命中率，以及能不能把批次工作排到離峰時段，都會影響實際帳單。

DeepSeek V4 仍然很有競爭力，但價格調整後是否繼續划算，需要用真實工作流重新測一次。對使用者來說，這會是接下來選擇 Flash、Pro 或其他模型時，必須納入的成本條件。

Reference:

- [DeepSeek API Change Log](https://api-docs.deepseek.com/updates)
- [DeepSeek V4-Pro-0813](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro-0813)
- [DeepSeek V4-Flash-0731](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731)
- [DeepSeek Models & Pricing](https://api-docs.deepseek.com/quick_start/pricing)
- [DeepSeek Thinking Mode](https://api-docs.deepseek.com/guides/thinking_mode)
