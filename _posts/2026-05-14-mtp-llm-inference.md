---
title: "MTP：讓 LLM 一次多猜幾步，推論速度就有機會上來"
date: 2026-05-14 14:16:01 +0800
categories:
  - tech
tags:
  - llm
  - inference
  - mtp
  - edge-ai
---

最近 LLM 圈常看到 MTP，完整名稱是 Multi-Token Prediction。白話講，就是讓模型在同一個 context 下，除了預測下一個 token，也順便預測後面第 2、第 3、第 4 個 token。

![NTP 和 MTP 推論流程差異示意圖](/assets/images/posts/mtp-ntp-comparison.jpg)

傳統 autoregressive decoding 規則簡單，但速度吃虧：一次 forward pass 只吐一個 token。回答越長，延遲越明顯。MTP 想處理的就是這個痛點。

| 技術 | 做法 | 解決的問題 | 工程代價 |
|---|---|---|---|
| NTP | 一次預測下一個 token | 訓練簡單、品質穩 | 推論逐 token 生成，慢 |
| MTP | 一次預測多個未來 token | 提高 sample efficiency，可當草稿模型 | 多 heads / module，部署複雜 |
| Speculative decoding | 小模型先草稿，大模型驗證 | 多 token 一次接受，加速輸出 | acceptance rate 不高就沒賺 |
| KV cache / MLA / GQA | 降低 attention 記憶體成本 | 減少 memory bandwidth 壓力 | runtime 和 kernel 要配合 |

Meta 2024 的論文把 MTP 講得很清楚：共享 transformer trunk，再用多個 prediction heads 預測未來 token。DeepSeek-V3 也把 MTP 放進架構裡，搭配 speculative decoding 來加速。Google 最近替 Gemma 4 推 MTP drafter，宣稱最高可到 3x speedup，這讓 MTP 從研究題目變成產品功能。

框架支援會決定 MTP 能不能真的落地。以 Gemma 4 為例，Google 釋出的 MTP drafter 已經放進 Hugging Face、vLLM、SGLang、MLX、Ollama 這類路線。實際部署時，vLLM 可以透過 speculative decoding 設定啟用 MTP，例如指定 `method: mtp` 和 `num_speculative_tokens`。這代表應用端不用重寫模型，只要推論框架支援，就能把 drafter 接進 serving pipeline。

但這裡也有一個現實限制：MTP 單靠模型權重不會自動變快。框架要能處理 draft、verify、accept/reject，還要把 KV cache 和 batching 做好。雲端 GPU stack 比較容易吃到紅利；到 edge NPU，就會碰到 graph 靜態化、dynamic control flow、compiler 支援這些老問題。

所以我的判斷是：MTP 會進入主流 LLM inference stack，但在嵌入式 NPU 上，要等 runtime 原生支援才會真的好用。現在可以先追，別急著把它當成萬能解法。

Reference:
- [Better & Faster Large Language Models via Multi-token Prediction](https://arxiv.org/abs/2404.19737)
- [Google: Multi-token prediction in Gemma 4](https://blog.google/innovation-and-ai/technology/developers-tools/multi-token-prediction-gemma-4/)
- [DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437)
- [Speculative Decoding](https://arxiv.org/abs/2211.17192)
- [vLLM Speculative Decoding](https://docs.vllm.ai/en/latest/features/spec_decode.html)
