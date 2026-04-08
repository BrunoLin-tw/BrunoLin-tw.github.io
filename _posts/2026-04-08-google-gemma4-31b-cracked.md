---
title: Google 最新開源 Gemma 4 31B 模型已被破解
date: 2026-04-08 15:30:00 +0800
categories:
  - AI
  - tech
tags:
  - AI
  - Gemma
  - Google
  - Open Source
  - Security
---

![Gemma 4 模型的牢籠已被解開](/assets/images/posts/2026-04-08-15-30-00-gemma4-uncaged.png)

Google 在 2026 年 4 月推出了 Gemma 4 開源模型家族。旗艦版 31B 在 Arena AI 排行榜上直接衝上第三名，號稱是「本地硬體上最強大的模型」。結果呢？發布不到一週就被破解了。

## 怎麼破解的？

技術叫「Abliteration」——中文姑且叫它「拒絕向量移除」吧。研究人員發現，語言模型會在內部某個向量方向上編碼「不要回答」的行為。只要把這個向量從權重中減掉，模型就什麼都肯說了。

GitHub 上有現成的實作（TrevorS/gemma-4-abliteration），用的是所謂「Norm-preserving Biprojection」方法。測試了 686 個提示，原始模型的拒絕率是 22/686（約 3.2%），破解後只剩 3/686。

## 實際效果

社群已經釋出好幾個破解版本：

- **dealignai/Gemma-4-31B-JANG_4M-CRACK**：18GB，MMLU 74.5%，HarmBench 93.7% 合規
- **TrevorJS/gemma-4-31B-it-uncensored**：約 31GB，MMLU 約 76%

所謂「合規」的意思是：port scanner、reverse shell、exploit 程式碼——統統會生成。知識基準只掉了大約 2%，但拒絕機制基本歸零。

## 這說明了什麼？

Google 砸錢做的 RLHF 安全訓練，在數學上只是向量的加减。任何有權重的人都可以自由開關這個「安全開關」。

開源模型的宿命就是这样：企業部署時不能只靠模型內建的防護，必須自己加過濾層。

---

*（圖片由 Nano Banana Pro 生成）*