---
title: "Ironwood 每美元效能贏 B200 50%？先別急著把 GPU 換掉"
date: 2026-09-11 00:30:00 +0800
categories:
  - tech
tags:
  - google-cloud
  - tpu
  - ironwood
  - nvidia
  - b200
  - llm-inference
  - ai-infrastructure
---

SemiAnalysis 在九月初公布 TPUv7 Ironwood 對 B200／B300 的 InferenceX 推論測試，最容易被轉傳的結論是：Google TPU 每美元效能最高領先 B200 約 50%，對 B300 更高。

![推論晶片競賽：NVIDIA 領跑，Google TPU 快速追近](/assets/images/posts/ironwood-inference-runners.jpg)

這個標題很吸睛，但「Ironwood 比 B200 便宜 50%」是錯誤解讀。

測到的是一條 Pareto 曲線上的幾個工作點：Qwen3.5 397B、FP8、8k input／1k output、聚合式 serving，以及特定併發與延遲目標。把這些條件拿掉，數字就沒那麼神奇了。

這篇報告真正值得看的，是 Google 開始把晶片、互連、記憶體管理和 PyTorch 生態接成一條可對外賣的推論路徑。

## 50% 是一個點，不是平均成績

| 測試條件 | Ironwood | B200 | B300 | 可讀出的結論 |
|---|---:|---:|---:|---|
| 每位使用者 20 token/s，總吞吐（token/s/chip） | 9,364 | 8,903 | 8,925 | Ironwood 吞吐約高 5% |
| 同上，外部 TCO 的每美元效能 | 基準 | -33.5% | -49.0% | Ironwood 每美元 token 吞吐比 B200 高 50.4%，比 B300 高 96.0% |
| 每位使用者 100 token/s，每百萬總 token 成本 | $0.181 | $0.222 | $0.276 | Ironwood 對 B200 約低 19%，對 B300 約低 34% |
| 併發 256，平均 TTFT | 5.41 秒 | 3.75 秒 | 2.40 秒 | Ironwood 的最高每美元效能工作點，同時有最慢首 token |
| 中位回應時間 20 秒，每百萬總 token 成本 | $0.098 | $0.106 | $0.132 | Ironwood 對 B200 約低 8%，對 B300 約低 25% |
| 約 30 秒中位回應時間 | — | — | — | B200 在曲線的一小段反超 Ironwood |

若產品是離線批次處理、文件萃取、長時間背景生成，使用者未必在意多等一兩秒，這個交換很合理。若是客服、程式助手或即時對話，首 token 多等 1.7 秒，產品端很可能寧可多付一點硬體費。

還有一層成本口徑必須講清楚。76.7% 對 B200、130.2% 對 B300 的那組數字，使用的是 Google 內部 TCO，每晶片小時 1.03 美元；外部客戶 TCO 下，對應數字是 50.4% 與 96.0%。兩套成本都來自 SemiAnalysis 的 BoM／TCO 模型，並非 Google Cloud 或 NVIDIA 的公開帳單價格。

模型這麼大，成本模型不透明，卻把單一最高數字拿來當通用結論，這種比較方式很容易把採購會議帶歪。

## Ironwood 的優勢，來自一整串協同設計

Ironwood 當然有硬體底子。單顆晶片包含兩個計算裸晶、192GB HBM、7.37TB/s HBM 頻寬，並原生支援 FP8。256×256 的 MXU 單周期可做 65,536 次乘加，矩陣吞吐比舊一代 128×128 設計高得多。

問題也正出在這裡：MXU 變大後，模型形狀不合就會浪費。

Qwen3.5 397B 對 Ironwood 很友善；報告也特別提到，Llama 3 8B 的 attention head dimension 是 128，映射到 256 維 MXU 時，利用率上限可能只剩 50%。因此，Ironwood 的優勢不能直接外推到每一個模型。選模型、改 layout、設平行化策略，都是成本的一部分。

MoE 模型更能看出 TPU 系統設計的價值。專家路由會帶來大量 All-to-All 通訊，若資料必須繞過主機 CPU 或跨多層網路，token 成本會被通訊延遲吃掉。Ironwood 用 ICI 直接連接晶片，64 顆形成 4×4×4 基本單元，再用 OCS 擴展到 9,216 顆的 Superpod。

軟體端也做了很實際的事：把 expert ID 和 routing weight 合併成一次 All-Gather。報告稱 DeepSeek-V3 每層可少約 80 微秒，58 層累積約少 4.64 毫秒。ReduceScatter 則交給 SparseCore，搭配雙緩衝把傳輸與計算重疊；在 8k1k、併發 64 到 512 的測試中，吞吐提升 4.1% 到 14.2%。

這些優化不會出現在產品型錄第一頁，卻很可能比多幾百 TFLOPS 更接近真實的每 token 成本。

## KV cache 會直接拉高可服務 token 的成本

大模型服務的成本，越來越不像「模型權重有多大」這麼單純。長 context、多輪對話與 agent 工作流，會讓 KV cache 持續吃掉 HBM 容量。HBM 被 cache 占滿後，同一組硬體能同時保留的 request 與 token 變少，併發能力下降，最終反映在每 token 的成本。

報告中的一個調整是把迴圈狀態從 FP32 降為 BF16，同時仍在 VMEM 維持 FP32 算術。HBM 佔用減半後，在 1k8k、併發 512 的吞吐增加 15%。

另一個更直接：最佳化 KV page layout，讓可用頁數從 5,141 增至 10,283。在 8k1k、併發 128 的測試中，吞吐提升 16.5%，中位 TTFT 降低 95%。

這些數字說明，長 context 的 token 成本很大一部分取決於 KV cache 能塞下多少內容、頁面能否有效利用，以及 cache 命中後能省掉多少 prefill。只看 accelerator 的 FP8 峰值，跟只看資料庫的 CPU 核心數一樣，會漏掉真正燒錢的地方。

## TorchTPU 的目標，是降低 TPU 的遷移摩擦

Google 過去讓 PyTorch 模型跑 TPU，多半要經過 TorchAX 轉到 JAX。這條路能跑，但 vLLM、SGLang、paged attention 和各種上游新功能，都容易卡在框架邊界。

新的 TorchTPU 走 PyTorch 的 PrivateUse1 擴充點，讓開發者可以直接使用 `.to("tpu")`。上層保留 PyTorch tensor、DDP、FSDP2、DTensor 與既有 serving engine 的架構；底下再透過 TorchDynamo、AOTAutograd、StableHLO、XLA 和 Pallas kernel 落到 TPU。

這不代表 TPU 從此不需要專用 kernel。高效能推論仍得看 tensor layout、sharding、MXU 對齊與通訊排程。差別在於 vLLM／SGLang 不必為 TPU 維護一套幾乎平行發展的 JAX 服務程式，模型支援和新功能才有機會跟上游靠近。

不過截至報告發布時，TorchTPU 仍在 private beta，預計十月才會在 PyTorch Conference 前後開源。現階段結果來自正在快速演進的 bring-up stack，還不能直接當成「隨便開一個 GCP 專案就能重現」的生產成績。

## 對台灣團隊，地理位置會改寫這筆帳

Google 官方 TPU 區域表顯示，TPU7x Ironwood 目前支援 `us-central1-ai1a` 與 `us-central1-c`；東京的 TPU 是 v6e Trillium，彰化 `asia-east1-c` 則只有 v2-8。

這讓台灣團隊的實際選擇很清楚：要在本地區域部署，就得使用較舊的 TPU，或改走 NVIDIA GPU；要用 Ironwood，工作負載得放到美國中部。

跨太平洋部署不只多了網路往返時間，也要重新檢查資料主權、客戶資料流向與合規限制。偏偏這份報告最關鍵的比較軸之一，就是 TTFT 和端到端延遲。

所以 Ironwood 的每美元效能優勢，對台灣使用者得連同地點、資料、延遲和模型形狀一起算；它是一道系統設計題。

## 我的結論

Ironwood 的訊號很明確：Google 已經不滿足於讓 TPU 只服務自家 Gemini，現在想把 TPU 變成外部推論市場的選項。

目前最合理的結論很窄：**在 Qwen3.5 397B、FP8、聚合式 serving 與特定延遲區間，Ironwood 能用較低的建模 TCO 打出非常強的每美元效能。**

它還不是「B200 的全面替代品」。

NVIDIA 在 FP4、disaggregated prefill/decode、成熟的 CUDA 工具鏈和模型 day-0 支援上仍有明顯優勢；SemiAnalysis 自己也承認 TPU 的外部 disaggregated serving 與 speculative decoding 還在補課。

真正值得追的是 TorchTPU 開源後，第三方能不能用公開程式碼重現成績、支援更多模型，並把長 context 和 agent 工作負載跑進可驗證的生產 SLA。

那時候，TPU 才算真的走出 Google 內部。

## Reference

- [SemiAnalysis: TPU Inference Externalization Full Steam Ahead - InferenceX](https://newsletter.semianalysis.com/p/tpu-inferencex-full-steam)
- [Google Cloud: TPU regions and zones](https://cloud.google.com/tpu/docs/regions-zones)
