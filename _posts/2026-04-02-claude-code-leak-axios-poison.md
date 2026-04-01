---
title: \"近日科技圈兩大意外事件 - CC原始碼洩漏/Axios被投毒\"
date: 2026-04-02 08:00:00 +0800
categories:
  - tech
tags:
  - ai
  - security
  - opensource
---

最近刷新聞，看到科技圈兩樁意外：Anthropic 的 Claude Code 原始碼外洩，Axios npm 套件還遭人投毒。來聊聊這兩件事。

Claude Code 是 Anthropic 的 AI 寫碼助手。他們在 npm 版 v2.1.88 打包時，忘了移除 59.8MB 的 source map 檔，結果洩了 50 萬行內部碼，大概 2000 個檔案。裡頭有 Hooks 和 MCP 伺服器的邏輯細節。Anthropic 說是人為疏忽。

這麻煩大了。駭客能用這資訊做惡意 GitHub repo，誘 Claude Code 偷偷跑指令或偷資料。連競爭對手也免費學到怎麼搭 agentic AI。網上還有人猜這是故意放的煙霧彈——前一天用 Claude Code 生成的假碼，愚人節秀實力，最後說是玩笑。誰知道呢？debug 檔怎麼就溜上線？內部流程得檢討。

另一頭，Axios 更慘。這 HTTP client 每週上億下載，17.5 萬專案靠它。駭客 hack 主要維護者 npm 帳號，丟出惡意 1.14.1 和 0.30.4。裡藏假依賴，裝完就下載 RAT，跨 Win、Linux、Mac。幸好幾小時內 npm 拔包。

連 OpenClaw 也依賴 Axios。最近裝升級版的同好，趕緊 check 依賴樹，看有沒有中招。開源真脆弱。帳號 2FA、CI/CD 簽名這些得盯緊。開發者現在掃依賴、升版或用 socket audit 吧。我自己專案也趕緊 check 了。

這兩樁事讓人警覺。AI 工具方便，開源生態養我們，但隨時有坑。多留心點，別中招。
