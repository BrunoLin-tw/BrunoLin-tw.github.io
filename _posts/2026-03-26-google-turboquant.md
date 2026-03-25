---
title: \"Google TurboQuant：KV Cache 壓縮新招，記憶體省 6 倍還不掉精度\"
date: 2026-03-26 23:00:00 +0800
categories:
  - ai
  - tech
tags:
  - google
  - llm
  - compression
  - kv-cache
---
今天在 [AI Daily Newsletter](https://brunolin-tw.github.io/ai-newsletter/reports/2026/03/25.html) 看到 Google Research 的 TurboQuant，覺得挺有意思的。這東西專門壓縮 LLM 裡的 KV cache（key-value 快取），用 3-bit 量化就把原本 16-32 bit 的東西縮小，記憶體用量降 6 倍以上，H100 GPU 上推理還快了 8 倍。最厲害的是零精度損失，benchmark 像 LongBench、Needle-in-Haystack 都滿分。

Google 提到的做法是，先用 PolarQuant 把向量轉成極座標（半徑 + 角度），避開傳統量化的開銷；然後 QJL 用 1-bit 修正殘差偏差。兩個算法加起來，壓縮 KV cache 時不需訓練，直接上。論文在 arXiv 上，ICLR 2026 要發表。

對產業來說，這不只是小技巧。LLM 跑長上下文時 KV cache 吃掉大半 RAM，壓縮後雲端成本能降不少，中小團隊更容易玩大模型。向量搜尋也加速，recall@K 比 PQ、RabbiQ 好，Google Search 或推薦系統會更快。開源模型像 Gemma、Mistral 整合後，聊天 bot、RAG 應用實時性 up up。

我看這會推一波邊緣部署熱潮，行動裝置上跑更大模型變可能。Google 丟這論文，感覺在逼對手跟進，Arm AGI CPU 同天新聞，AI 硬體軟體一起衝。值得追論文細節，你們怎麼看？

原文：[Google Research](https://research.google/blog/turboquant-redefining-ai-efficiency-with-extreme-compression/)
