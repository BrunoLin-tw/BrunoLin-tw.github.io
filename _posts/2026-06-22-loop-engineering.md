---
title: "Loop Engineering：Agent 時代真正該練的是迴圈設計"
date: 2026-06-22 17:45:00 +0800
categories:
  - tech
tags:
  - ai-agents
  - loop-engineering
  - coding-agent
  - workflow
---

最近 AI 圈開始熱講 Loop Engineering。名字聽起來像新 buzzword，但我覺得這次有實際內容。

它的重點不在提示詞技巧，而在把 agent 從「一問一答」推到「可重複、可驗證、可停止」的工作流程。Claude Code、Codex、Hermes 這類工具開始能讀檔、改 code、跑測試、看錯誤再修正後，真正拉開差距的地方，已經不只看模型本身，也看外層迴圈怎麼設計。

## 從寫 prompt 變成設計控制流程

以前我們使用 AI coding tool，多半是這樣：

```text
我：幫我修這個 bug
AI：改好了
我：跑測試
AI：測試失敗
我：看錯誤再修
AI：再改一次
```

人一直卡在中間。每一步都要手動推進。

Loop Engineering 想處理的就是這件事。你先定義目標、工具、驗證條件和停止條件，讓 agent 自己跑幾輪：

```text
目標：修到指定測試通過
流程：
  讀錯誤
  找相關程式
  修改
  跑測試
  根據錯誤再修
  測試通過就停止
  重複失敗就回報人類
```

這比較像控制系統。模型只是其中一個零件，外面還要有狀態、工具、驗證器、權限和退出機制。

## 好的 loop 一定要能停下來

我看了幾篇最近的整理，LangChain 把 loop 分成 agent loop、verification loop、event-driven loop 和 hill-climbing loop；MindStudio 和 Oracle 則把重點放在 agent 如何 act、observe、reason、repeat。說法不同，但核心很一致：loop 不能只是重複呼叫模型。

一個能用的 loop 至少要有幾個條件：

- 目標要清楚，例如「讓 `pytest tests/test_camera.py` 通過」
- agent 要能觀察真實結果，例如 compiler error、test log、benchmark output
- 驗證要盡量用 deterministic signal，例如 test、lint、type check、數值比對
- 要限制範圍，例如只能改特定 module，不能碰 deploy
- 要有停止條件，例如最多 6 輪、同一錯誤重複 3 次就停

最後那點很重要。沒有停止條件的 loop 很危險，它會把 token、CI minutes 和工程師耐心一起燒掉。

## Maker 和 Checker 最好分開

我最認同的一個設計是 maker-checker split。

Maker agent 負責做事：改 code、補測試、整理文件。Checker agent 或驗證流程負責判斷結果能不能收。

實務上 checker 不一定要是另一個 LLM。更可靠的順序應該是：

1. 測試是否通過
2. build / lint / type check 是否乾淨
3. 數值或 benchmark 是否在門檻內
4. diff 是否超出範圍
5. 最後才交給 LLM judge 看語意或需求符合度

如果只問另一個模型「你覺得完成了嗎」，那只是兩個模型互相點頭。這種驗證太鬆，不能拿來跑無人看守的工作流。

## 對嵌入式 AI 來說，這東西其實很實用

Loop Engineering 最容易被講成 coding agent 的潮流，但我覺得它對 embedded AI / NPU 工作更有價值。

例如模型轉換流程：

```text
目標：把 ONNX 轉成 RKNN，板端輸出和 PC reference cosine similarity > 0.98

loop：
  檢查 input / output shape
  跑轉換工具
  轉換失敗就分析 log
  調整 opset、preprocess、quantization config
  跑板端 inference
  比對 reference output
  達標就停止
  低於門檻就輸出可能原因
```

這種工作很適合 loop，因為它有明確輸入、明確工具、明確驗證方式，也有很清楚的失敗訊號。

Camera pipeline、NPU runtime、SDK sample build、CI regression triage 也一樣。這些任務不需要 agent 亂猜產品方向，只要它反覆處理工程細節，然後用測試結果說話。

## 真正的難點在狀態管理

長時間 agent session 常會變笨。原因很直覺：context 裡塞滿過時資訊、已經放棄的假設、舊錯誤、半成品 plan。跑越久，越容易鬼打牆。

所以好的 loop 不一定要靠一個超長對話撐到底。更務實的做法是把狀態放在外面：

```text
PLAN.md
STATUS.md
RUN_LOG.md
git diff
issue comment
trace log
```

每一輪可以重新開 agent，但讓它讀外部狀態。這樣 context 比較乾淨，也比較容易 debug。人要接手時，也知道前面到底發生什麼事。

這點跟一般 prompt 技巧差很多。Prompt 是一次性的輸入；loop 要考慮跨回合的狀態保存和錯誤恢復。

## 我會怎麼開始用

如果要在真實工作流試 Loop Engineering，我不會一開始就讓它自動 merge、deploy 或改 production config。那太衝。

比較安全的起點是：

- 修單一測試檔
- 整理 release note
- 根據 CI log 找出可能 regression
- 跑 model conversion smoke test
- 比對 inference output
- 更新文件和 sample code

這些任務都有邊界。agent 做錯了，成本可控；做對了，可以省很多重複時間。

我會用這個模板開任務：

```text
目標：
完成一件可驗證的小任務。

限制：
只改指定目錄，不 push，不 deploy，不碰 secret。

驗證：
執行指定 command，結果通過才算完成。

停止：
最多 5 輪；同一錯誤重複 3 次就停；需要高風險操作就交還人類。

輸出：
列出修改摘要、測試結果、未解問題。
```

這樣寫看起來麻煩，但這才是 agent 能不能穩定工作的關鍵。

## 我的看法

Loop Engineering 這個名字可能會紅一陣子，也可能很快被下一個名詞蓋過去。但它背後的工程方向我認為會留下來。

接下來會變重要的能力，是把工作拆成可執行、可觀察、可驗證、可停止的迴圈。這比單純會寫提示詞更接近真實工程。

模型會繼續變強，工具也會越來越多。但如果外層 loop 設計很差，強模型只會更快地把錯誤放大。

我會把 Loop Engineering 當成 agent 時代的流程控制工程。這對 coding agent 有用，對 NPU / embedded AI 這種需要反覆驗證的工作，可能更有用。

Reference:
- [LangChain: The Art of Loop Engineering](https://www.langchain.com/blog/the-art-of-loop-engineering)
- [MindStudio: What Is Loop Engineering?](https://www.mindstudio.ai/blog/what-is-loop-engineering-ai-coding-agents)
- [Oracle: The Agent Loop Decoded](https://blogs.oracle.com/developers/the-agent-loop-decoded-three-levels-every-agent-engineer-must-know)
- [Tosea.ai: What Is Loop Engineering?](https://tosea.ai/blog/loop-engineering-ai-agents-complete-guide-2026)
- [Lushbinary: Loop Engineering: Designing Systems That Prompt AI Agents](https://lushbinary.com/blog/loop-engineering-ai-coding-agents-guide)
