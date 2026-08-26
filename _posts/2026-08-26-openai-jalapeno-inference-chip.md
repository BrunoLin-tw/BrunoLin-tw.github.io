---
title: "OpenAI 自研推理晶片 Jalapeño：數字漂亮，考卷也是自己出的"
date: 2026-08-26 13:50:00 +0800
categories:
  - tech
tags:
  - ai-chip
  - inference
  - asic
  - openai
  - nvidia
---

OpenAI 公布首款自研推理晶片 Jalapeño 的跑分：對上 Nvidia GB200／GB300 superchip 在 InferenceX 平台的既有最佳紀錄，每瓦多完成 1.5 到 1.9 倍的推理工作，端到端延遲低 1.7 到 3.6 倍。受測模型有三個：GPT-OSS 120B、DeepSeek R1、Kimi K2.5 1T。

## Jalapeño 是什麼

六月初次亮相，與 Broadcom 合作開發的 ASIC，專攻 AI inference——跑已經訓練好的模型。這次公布的內容就是效能數據，測試平台用 SemiAnalysis 的 InferenceX。

OpenAI 硬體副總裁 Richard Ho 的形容是「兩個世界的好處都要」：一般推理系統得在低延遲和高吞吐之間做取捨，他們宣稱 Jalapeño 同時拿下兩邊。

## 這份成績單要先打折

幾個閱讀前提：

- 受測模型由 OpenAI 挑選，其中 GPT-OSS 120B 是自家模型，軟體堆疊對它優化得最完整並不意外
- 對照組是平台上「當時的既有紀錄」，Nvidia 陣營沒有機會在同一份 benchmark 上重新調校應戰
- batch size、context 長度、量化精度這些設定細節，報導裡沒有揭露

倒也不用因此說數字造假。ASIC 在固定 workload 上贏過通用 GPU 本來就在預期內，問題只在領先幅度能否在第三方環境重現。等 InferenceX 上有獨立的重跑結果，再來認真比較。

## 換 NPU 視角看

我以前的工作就是把 AI 模型移植到嵌入式 NPU，ASIC 這條路的脾氣很清楚：固定 workload 加大規模部署，能效比可以拉得很開；代價是彈性，模型架構一翻新，編譯器和 kernel 就得跟著改一輪。把單一模型跑快不難，難的是接下來幾年每一代新模型都有效率地跑得上去。

OpenAI 剛好站在最有利的位置：自己營運推理服務，workload 自己掌控，軟硬體可以一起調。Google TPU、Amazon Trainium、Meta MTIA 走的都是同一套劇本，Anthropic 也傳出在開發自有晶片，超大戶自建晶片已經是標準配置。

## 對 agent 工作流才真的有感

推理成本下降、token 間延遲變短，最先受惠的是 agent。一次任務動輒幾十次工具呼叫、上萬 token 輸出，TBT（time between tokens）砍半，互動節奏和任務總時間都直接受益。這比任何 benchmark 分數都實際。

## 後續關注重點

- 年底先小量部署，2027 年放量，確切數量未公布
- 第二代、第三代已在開發
- 官方明講不會全面取代 GPU，Nvidia 仍是重要夥伴

規模才是後續重點。跑分領先和供貨能力是兩回事，ASIC 從設計定案到量產爬坡以年計。2027 年實際部署多少顆，比今天這組數字更值得盯。

Reference:

- [The Verge：OpenAI says its Jalapeño chip can power faster AI responses than the competition](https://www.theverge.com/ai-artificial-intelligence/984290/openai-jalapeno-ai-chip-benchmarks)
- [SemiAnalysis / InferenceX](https://semianalysis.com/about/)
