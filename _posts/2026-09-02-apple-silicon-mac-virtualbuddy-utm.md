---
title: "Apple Silicon Mac 上的開源虛擬化軟體怎麼選：VirtualBuddy 與 UTM"
date: 2026-09-02 00:30:00 +0800
categories:
  - tech
tags:
  - apple-silicon
  - macos
  - virtualization
  - virtualbuddy
  - utm
  - linux
---

Apple Silicon Mac 的效能很夠，想開一台隔離環境測軟體、跑 agent，或架一個小型 Linux server，第一個問題通常是該選哪個前端。

VirtualBuddy 和 UTM 都免費、開源，也都能在 Apple Silicon 上建立虛擬機。兩者的設計目標差得很遠：VirtualBuddy 把重心放在 macOS；UTM 則把 QEMU 包成一個能應付各種系統的工具。選錯不會完全不能用，只是會每天跟不適合的設定搏鬥。

## 先看結論

- 要測 macOS、跑需要 macOS 環境的工具，選 **VirtualBuddy**。
- 要架 Ubuntu、Windows ARM 或其他系統，選 **UTM**。
- 要跑 x86 Linux／Windows，UTM 做得到，但那是 CPU 模擬，不要期待原生速度。

## 兩者的定位不一樣

VirtualBuddy 基於 Apple 的 Virtualization.framework，目標很明確：在 Apple Silicon Mac 上建立與管理 macOS 虛擬機。它會列出 Apple 提供的 macOS restore image，也能匯入 IPSW；針對多版本與 beta 測試，流程幾乎沒有多餘設定。

它也能啟動部分 ARM Linux 發行版，不過這是附加能力。它的價值在於把 macOS VM 該有的事情處理好：復原模式、剪貼簿、共享資料夾、儲存與還原 VM state。若想留一台乾淨的 macOS 做測試母片，利用 APFS clone 複製 VM 幾乎不立即吃掉第二份完整磁碟空間，這比每次重裝方便得多。

UTM 的方向更廣。它是 QEMU 的圖形化前端，可在 macOS 上執行 Windows、Linux、BSD 等系統；同時提供原生虛擬化與跨架構模擬兩條路。前者適合 ARM guest，效能合理；後者讓 Apple Silicon 能開 x86 作業系統，交換條件就是明顯較慢。

## 一張表先選掉大半情境

| 面向 | VirtualBuddy | UTM |
|---|---|---|
| 核心定位 | Apple Silicon 上的 macOS VM | 通用型 VM 與系統模擬器 |
| 最適合的 guest | macOS | Linux、Windows ARM、BSD 與其他 ISO |
| macOS 安裝 | 可直接選 Apple restore image 或使用 IPSW，流程集中 | 能做，但不是產品的主軸 |
| Linux server | 能跑部分 ARM Linux，選項較少 | 適合；網路、磁碟與硬體選項可細調 |
| Windows | 不適合當主要選擇 | 可跑 Windows ARM；也可模擬 x86 Windows |
| x86 guest | 不適合 | 可透過 QEMU 模擬，但效能較差 |
| 操作風格 | 設定少，快速建立 macOS 測試環境 | 選項多，需理解 CPU 架構與 VM 設定 |
| macOS 整合 | 共享資料夾、剪貼簿、快照／state 回復走得順 | 功能視 guest 與設定而定 |
| 適合的人 | Apple 平台開發、beta 相容性測試 | Homelab、Linux 服務、跨系統實驗 |

## 跑 OpenClaw，要先決定它需不需要 macOS

如果 OpenClaw 的目標是隔離環境、又需要 macOS 的 app、權限或 Apple 生態工具，VirtualBuddy 很合適。開一台乾淨的 macOS VM，將 agent 和它能碰到的檔案、帳號、權限關在那台 guest 裡，主機不會被每次實驗留下的設定污染。

但如果 OpenClaw 只是 Node.js、Python、Docker 或瀏覽器服務，硬塞進 macOS VM 沒有額外好處。用 UTM 建一台 Ubuntu ARM server 通常更輕、更接近之後真正部署到 VPS 或家中伺服器的環境。把 Linux 工作負載丟進 macOS guest，只是多繞一層。

## 效能別被「虛擬化」三個字騙了

在 Apple Silicon 上，ARM guest 使用硬體虛擬化，UTM 和 VirtualBuddy 的差別通常不會是 CPU 跑分。實際感受更多來自 guest 系統、RAM、磁碟 I/O，以及圖形與網路設定。

真正會拉開差距的是架構。M 系列是 ARM；開 ARM Linux 或 Windows ARM 屬於合理路徑。想跑 x86_64 Linux、老版 Windows 或只提供 x86 映像的 appliance，UTM 能以 QEMU 模擬救場，但那不是免費午餐。若服務需要長時間跑、要編譯程式或處理大量資料，直接找 ARM64 映像或改用別的硬體，通常比硬扛模擬更有效。

## 我的選法

我的判斷很簡單：

1. **工作負載明確需要 macOS：VirtualBuddy。** 例如測不同 macOS 版本、測 Safari／Xcode，或需要隔離的 macOS agent 環境。
2. **工作負載本質是 server：UTM。** Ubuntu ARM、Docker、資料庫、automation service 都屬於這一類。
3. **工作負載只支援 x86：先評估替代方案。** UTM 可以跑，但長期使用時，模擬效能與相容性成本要算進去。

VirtualBuddy 和 UTM 沒有誰全面贏。前者把 macOS VM 這一件事做得很俐落；後者的廣度更大，也要求使用者多理解一些虛擬化細節。把 VM 當成 macOS 測試機，就選 VirtualBuddy；把它當成一台可自行配置的小伺服器，UTM 才是正解。

Reference:

- [VirtualBuddy GitHub README](https://github.com/insidegui/VirtualBuddy)
- [UTM Documentation](https://docs.getutm.app/)
