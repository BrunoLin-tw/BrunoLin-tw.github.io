---
title: "更難防的垃圾郵件/詐騙網站 全都用Vibe-coding"
date: 2026-03-23 21:00:00 +0800
categories:
  - tech
  - ai
tags:
  - AI
  - 程式設計
  - 資安
  - HackerNews
  - vibe-coding
---

這裡先寫前言，簡述這篇文章要解決的問題：AI 讓垃圾郵件網站進入「Vibe-coding」時代，生成低質但海量詐騙頁面，為何這讓防禦更難？

## 背景

最近在 [Hacker News](https://news.ycombinator.com/item?id=47482760) 上，一篇「They’re vibe-coding spam now」引發熱議（104 points，62 評論）。文章指出，過去垃圾郵件附的詐騙網站是手工粗製濫造，現在全用 AI 「感覺式編碼」（vibe-coding）生成：不講結構、安全，直接「感覺對了就行」。

- **Vibe-coding 是什麼？** AI 工具如 Cursor、Claude Code 讓人隨意 prompt 出網站，但忽略最佳實踐。結果：SQL 注入漏洞、明文洩密、易被黑帽駭客攻破。
- **Spam 演變**：從低質過濾笨蛋，到 AI 自動化海量生產。Gmail 等也擋不住，因為頁面看起來「正常」。

相關討論：
- [Vibe code is legacy code](https://news.ycombinator.com/item?id=44739556)：視為即時遺留碼。
- [Vibe Coding Is a Security Disaster](https://news.ycombinator.com/item?id=47479724)：預警資安災難。

## 分析：為何更難防？

1. **海量 + 變異**：AI 生成無限變體，傳統黑名單失效。
2. **偽裝正常**：不像舊 spam 爛到一眼認出，vibe-code 網站有 UI、動畫，但內部是爛攤子。
3. **資安漏洞放大**：開發者怒批「大公司也 vibe-code 洩數據」。黑帽當「義警」，專攻這些弱點。
4. **AI 雙刃劍**：加速 scam，但也讓偵測 AI 跟進（e.g., 分析 code 品質）。

## 結論

Vibe-coding 讓 spam 從「笨」變「氾濫」，資安挑戰升級。開發者：別濫用 AI，寫生產碼要嚴謹。讀者：收到可疑郵件，別急點！

下一步：試用 AI 偵測 vibe-code？歡迎討論。

---
