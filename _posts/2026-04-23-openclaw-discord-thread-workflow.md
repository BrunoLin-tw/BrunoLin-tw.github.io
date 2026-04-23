---
title: "從 Telegram 到 Discord，我才發現它更適合 AI Agent 協作"
date: 2026-04-23 22:59:00 +0800
categories:
  - ai
  - tools
tags:
  - openclaw
  - discord
  - telegram
  - ai-agent
  - hermes
---

這兩個星期因為開始玩 Hermes agent，我也順便重新認識了 Discord。

在這之前，我的 OpenClaw 一直都是用 Telegram 當作主要通訊方式。Telegram 對我來說一直都很好用，開啟快、互動直接、通知即時，拿來和 agent 溝通也很順，所以我原本並沒有特別想換別的工具。

但真的開始用 Discord 跟 Hermes agent 互動之後，我才發現，Discord 的 channel 和 thread 設計，跟 AI agent 的工作模式幾乎是天作之合。這次不只是多接了一個 message channel 而已，而是讓我重新理解，什麼樣的通訊介面才更適合拿來和 AI agent 長期協作。

## 我一直以為 Telegram 已經夠用了

過去我使用 OpenClaw，基本上就是把 Telegram 當成和 agent 溝通的主要入口。

這樣的方式其實沒有什麼問題。像是平常收通知、快速問答、臨時交代事情，Telegram 都很方便。它的優勢就是夠直覺，想到什麼就傳什麼，不需要切換太多操作模式。

只是當 AI agent 的使用開始變得更頻繁，問題開始變得更多、更細，而且不同主題會在同一段時間內交錯出現時，Telegram 這種比較線性的聊天模式，限制就慢慢浮現出來了。

最常見的情況就是，原本正在和 agent 討論某個問題，聊到一半又想到另一件完全不同的事。這時候如果直接接著問，對話脈絡很容易混在一起。雖然也不是不能處理，但會變得不夠自然。

以前我其實沒有特別強烈感受到這點，因為一直都這樣用，也就習慣了。直到這次開始實際使用 Discord，我才發現原來這件事可以被處理得更好。

## 直到我開始用 Hermes agent 跑在 Discord 上

這兩個星期我開始玩 Hermes agent，而 Hermes agent 主要就是用 Discord 當作通訊工具。

坦白說，在這之前我對 Discord 一直都很陌生。知道它很流行，也知道很多社群都在用，但我從來沒有真的把它放進自己的工作流裡。這次算是因為 Hermes agent，才第一次比較長時間、比較認真地去使用 Discord。

也因為這樣，我很快就發現 Discord 有一個特性，真的非常適合 AI agent，那就是它的 channel 和 thread 機制。

這種感受不是看文件就能完全理解的，而是你真的開始拿它來跟 agent 協作，才會突然覺得，原來這種設計比我原本想像中合理太多。

## 為什麼 Discord 比我想像中更適合 AI Agent

我自己最有感的地方，是 Discord 的 thread 機制。

在 Discord 上，只要在某個頻道裡 tag Bot 問問題，它就可以自動開出一個 thread 來處理。這個 thread 幾乎就可以看成一個獨立的 session。

也就是說，一個問題對應一個討論串，一個討論串對應一段完整上下文。整個使用邏輯非常直觀。

你可以在那個 thread 裡，把同一個主題一路談完，不怕中途被別的問題打斷。如果聊到一半，突然想到另一件完全不同的事，也不用硬把新問題塞進原本那串對話裡，只要回到主頻道再 tag Bot 一次，它又會自動開一個新的 thread，讓新問題有自己的空間。

這種感覺非常接近在平行管理多個任務。

每個 thread 都有自己的主題、自己的上下文、自己的 session。你不需要自己在腦中一直切換，也不用刻意用規則維持秩序，因為 Discord 的介面結構本身就已經幫你把這件事處理好了。

這和 Telegram 的體驗差很多。

Telegram 當然也能和 agent 對話，但它更像是一條持續往下延伸的聊天流。當你同時有兩三個主題想處理時，很容易出現上下文混在一起的問題。即使你知道可以手動切 session，整體操作感還是沒有 Discord 的 thread 那麼自然。

也是到這時候我才真正體會到，Discord 不只是另一種聊天工具而已。在 AI agent 的使用情境下，它其實更像是一個很適合承載多主題、多 session 協作的工作介面。

## 真正花時間的，是 OpenClaw 怎麼接出理想的 Discord 工作流

當我感受到 Discord 這種互動方式的優勢之後，自然就會想把 OpenClaw 也接到 Discord 上，希望它能提供類似的體驗。

結果，這一段比我原本想像中花了更多時間。

OpenClaw Docs 不是完全沒寫 Discord Bot 的設定，但如果你期待的是照著文件設定完，就能很自然地做到 tag Bot、自動開 thread、讓 thread 對應 session 的這種工作流，那實際上並沒有那麼順。

至少以我這次的經驗來看，官方文件提供的資訊不算完整，有些關鍵設定和互動行為並沒有講得很清楚。也因為這樣，我一開始照著做，並沒有成功得到自己想要的效果。

後來只好自己繼續上網找資料，看別人的實戰分享、對照不同設定方式，再自己反覆測試。中間確實花了不少時間，不過最後也總算把方向摸出來。

如果你也剛好在研究 OpenClaw 搭配 Discord，我這次覺得比較有幫助的參考資料有這兩篇：

- [OpenClaw Discord workflow](https://hanamizuki.tw/openclaw-discord-workflow/)
- [Insight OpenClaw Discord](https://www.meta-intelligence.tech/insight-openclaw-discord)

這兩篇不一定能直接解決所有問題，但至少在觀念和實作方向上，對我來說是有幫助的。

## Telegram 和 Discord，不是二選一

用到現在，我自己的結論其實很明確。

Telegram 還是很好用，而且我也不打算放掉。它很適合日常通知、快速互動、即時問答，這種想到就講、講完就走的情境，Telegram 依然非常有效率。

但如果是要和 AI agent 做比較深入、比較長，而且同時間可能有多個主題並行的互動，那 Discord 的 channel 和 thread 機制，真的會讓整體體驗好很多。

所以對我來說，Discord 並不是要取代 Telegram，而是補上 Telegram 在多主題協作上的缺口。

簡單講，Telegram 比較像是隨身入口，Discord 比較像是任務工作台。

兩者不是互斥，而是各自適合不同的使用場景。這次也算是因為 Hermes agent，才讓我真正理解 Discord 在 AI agent 協作上的價值。以前我對 Discord 其實很陌生，現在實際用過之後，反而會覺得，如果你是認真想把 AI agent 放進日常工作流裡，那 Discord 很值得花時間理解。

至少對我來說，這次不是單純多加了一個 message channel，而是重新理解了，什麼樣的通訊介面，才真的適合拿來和 AI agent 長期協作。
