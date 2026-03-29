---
title: "重要的事情要說三遍的概念對AI也有效"
date: 2026-03-27 07:50:00 +0800
categories:
  - AI
tags:
  - AI
  - LLM
  - Prompt
  - Google
  - Gemini
---

你聽過「重要的事情要說三遍」嗎？這句常聽到的話，現在連 AI 語言模型（LLM）也適用！Google Research 最新研究發現，**簡單把提示詞（prompt）重複 2-3 次**，就能大幅提升模型準確率，特別在精確檢索任務上，從 21% 飆到 97%，且不增加輸出長度或延遲。CP 值超高，值得一試。

## 為什麼 LLM 需要「說兩遍」？

主流 LLM（如 Gemini、GPT、Claude）是 **因果語言模型（Causal LLM）**，處理文字時只能「左到右」單向閱讀：讀到「問題」時，後面的「背景資料」還在「未來」，無法參考。這導致 **單向注意力（one-way attention）** 瓶頸，尤其在「問題在前，選項/上下文在後」的提示中，模型常搞錯。

**解法**：Google 提出的想法是 Prompt Repetition，把整個提示重複！
```
原始：名單：[A,B,C...] 第25個是？
重複：名單：[A,B,C...] 第25個是？名單：[A,B,C...] 第25個是？
```
第二遍時，第一遍變「歷史」，每個 token 都能 attend 到全提示，模擬 **雙向注意力**。

這個研究結果發表在：[arXiv:2512.14982](https://arxiv.org/abs/2512.14982)，作者是 Google Research，三位大牛（Yaniv Leviathan 等）。

## 實驗結果：47 勝 0 敗！

文中測試了 7 種模型（Gemini 2.0 Flash/Lite、GPT-4o、Claude 3 Haiku/Sonnet、DeepSeek V3）、70 基準（ARC、MMLU-Pro、GSM8K 等）：

| 任務 | 模型 | Baseline | ×2 重複 | ×3 重複 |
|------|------|----------|---------|---------|
| NameIndex（50 名單找第 25） | Gemini 2.0 Flash-Lite | 21% | **97%** | 更好 |
| MiddleMatch（找中間名） | 多模型 | 低 | 強增 | **最佳** |
- **非 reasoning**：全勝（p<0.1, McNemar test）。
- **CoT/reasoning**：中性（模型自重複）。


## 實作方法

1. **原始 prompt**：問題 + 背景。
2. **重複版**：完整 ×2（或 ×3 難任務）。
3. **適用**：問答、非 CoT、長文檢索。不用微調，API 直用。

範例（NameIndex）：
```
名單：Dale Lopez, Peter Sanchez, ... (50 人)
問：第 25 個？
重複：(以上全部) (以上全部)
```
Gemini Lite 的回答正確率大幅度提升：21% → 97%！

## 注意事項

這個研究也有一些限制條件，
- **不適用**：reasoning 模型（Gemini 3、o1），因為推理模型本來就會自己重複想很多次了。
- **超長 prompt**：太長的內容該忘記還是會忘記。
- **變體**：加「Let me repeat:」；Padding 無效（證明是重複效應）。

跟一般人常說的「重要的要說三遍」一樣，AI 也需要強調。下次 prompt 試試看吧！

### Reference
- [arXiv 論文 pdf](https://arxiv.org/pdf/2512.14982)
