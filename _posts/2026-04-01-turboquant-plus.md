---
title: "這位老兄把 TurboQuant 實現還加強了 - TurboQuant+"
date: 2026-04-01 11:00:00 +0800
categories:
  - ai
  - tech
tags:
  - TurboQuant
  - kv-cache
  - llama-cpp
  - compression
  - inference
---

之前提到[Google 發表TurboQuant](https://brunolin-tw.github.io/ai/tech/google-turboquant/) 的消息，提到了使用PolarQuant + QJL 的極端壓縮方案，理論上 3.8~6.4x 壓縮率，但論文沒有code。

這時 GitHub 上冒出這位 TheTom，老兄不只原封不動實現，還丟了一堆獨家擴展，搞出 **TurboQuant+** ([repo](https://github.com/TheTom/turboquant_plus))。Stars 3800+，社群驗證滿滿（M5 Max、RTX 4090/5090、AMD），值得一看。

## TurboQuant+ 有甚麼厲害的？
核心是 Transformer **KV cache 壓縮**：
- **turbo2** (2-bit): 6.4x 壓縮，PPL 漲 6.48%（極限記憶體用）。
- **turbo3** (3-bit): 4.6~5.12x，PPL +1.06%。
- **turbo4** (4-bit): 3.8x，PPL 僅 +0.23%（近 q8_0 無損）。

用 PolarQuant（旋轉後 Beta 分佈量化）+ Walsh-Hadamard + QJL 殘差，**prefill 追 q8_0**，**decode ~0.9x**。支援 GGUF 模型，從 1.5B 到 104B Command-R+ 都驗過，128K 上下文壓到 74GB。

## TheTom 的「+」在哪？真金不怕火煉
論文是起點，老兄加了三招，已被多硬體獨立驗證：
1. **V 壓縮免費**：Value 壓到 2-bit 沒事，只要 Key 保持 q8_0。對稱壓縮失效？**非對稱 K q8_0 + V turbo** 救回來（Qwen2.5 Q4_K_M 神救）。
2. **Boundary Layers 敏感**：前 2 + 後 2 層 q8_0，其餘 turbo2，品質回收 **37~91%**。
3. **Sparse V Dequant**：用 attention 權重閘控，跳低貢獻 V，**decode +22.8%**（32K 上下文，PPL 零變動）。

其他黑科技：block size 優化（5.12x）、turbo4 復活（PolarQuant 勝 QJL）。[docs/papers/](https://github.com/TheTom/turboquant_plus/tree/main/docs/papers) 全有實驗數據，**511+ Python test + NIAH 檢索** 硬證。

## 安裝簡單，硬體友善
Python 原型 `pip install .`，生產用 **llama.cpp fork**：
```
git clone https://github.com/TheTom/llama-cpp-turboquant
cd llama-cpp-turboquant && git checkout feature/turboquant-kv-cache
cmake -B build -DGGML_METAL=ON && cmake --build build -j  # M 系列天堂
```
CUDA/HIP 換旗標就好。跑起來：
```
# 安全預設 (非對稱，任意模型)
llama-server -m qwen-Q4_K_M.gguf -ctk q8_0 -ctv turbo4 -fa 1 -ngl 99 -c 8192

# 狂壓 (大模型)
llama-server -m llama70b.gguf -ctk turbo3 -ctv turbo3 -fa 1
```
`-fa 1` 開 Sparse V + Flash Attention。**台灣 M 系列 Mac / RTX 筆電直接上**，記憶體省一半。

## 優缺點直球對決
**Pro**：
- 壓縮極致，適合 ctx 降低記憶體使用（128K 74GB）。
- 速度不鳥（prefill 1.1x q8_0）。
- 配置指南詳細（[turboquant-recommendations.md](https://github.com/TheTom/turboquant_plus/blob/main/docs/turboquant-recommendations.md)），**Apache-2.0** 隨便 hack。

**Con**：
- 小模型 Q4_K_M 要非對稱，否則 PPL 爆。
- llama.cpp fork（上游沒 merge），Windows/AMD 邊緣。
- turbo2 品質掉多（+6.48%），元數據小開銷。

**結論**：**有興趣玩可以試試**。從 asymmetric turbo4 開始，測 PPL / tok/s。看看能省多少顯卡記憶體。

Repo：
- [TurboQuant+ (Python)](https://github.com/TheTom/turboquant_plus)
- [llama-cpp-turboquant (C/C++)](https://github.com/TheTom/llama-cpp-turboquant)

