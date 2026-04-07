---
title: "生化危機女星米拉開源 MemPalace：AI 記憶宮殿，Benchmark 史上第一個滿分！"
date: 2026-04-07 22:00:00 +0800
categories:
  - ai
  - tech
tags:
  - AI
  - 記憶系統
  - opensource
  - mempalace
---

好萊塢《生化危機》女主角米拉·喬沃維奇（Milla Jovovich）跨界AI圈，在GitHub開源 [MemPalace](https://github.com/milla-jovovich/mempalace)，一個純本地AI記憶系統。Benchmark測試LongMemEval R@5拿下100%滿分（rerank），raw也96.6%，遠勝Mem0/Zep等雲端貨。新聞：[AIbase報導](https://news.aibase.com/zh/news/26899)。

## 痛點：AI為什麼老忘事？

用Claude/ChatGPT半年，19.5M tokens的決策、debug、架構辯論全沒了。傳統記憶系統讓AI自己判斷「值得記」，丟掉原始對話。米拉痛到自己寫：全存+結構化檢索，永不遺失。

## 記憶宮殿：古希臘術 + AAAK壓縮

- **結構**：Wing（人/專案）→ Room（主題如auth）→ Closet（壓縮摘要）→ Drawer（原始檔）。Hall連室內，Tunnel跨wing，提升檢索+34%。
- **AAAK**：自創無損方言，30x壓縮（170 tokens載入關鍵事實），任何LLM懂，離線跑。
- **本地**：SQLite+ChromaDB，MIT開源，無API、無雲。

Benchmark對比：
| 系統 | R@5 | 成本 |
|------|-----|------|
| MemPalace (hybrid) | 100% | $0 |
| MemPalace (raw) | 96.6% | $0 |
| Mastra | 94.9% | API費 |
| Mem0 | ~85% | $19+/月 |

## 快速試用（5分鐘）

```bash
pip install mempalace
mempalace init ~/my-palace
mempalace mine ~/chats/ --mode convos  # 礦化對話
mempalace search "auth 決策"  # 搜
```

Claude用MCP：`claude mcp add mempalace -- python -m mempalace.mcp_server`，AI自動搜記憶。

Local LLM：`mempalace wake-up > context.txt` 丟prompt。

## 結論

結構+壓縮遠勝現有記憶插件，OpenClaw/LanceDB 直接 fork 來用。女星工程水準不輸大廠，去裝了試，舊對話全救回來。

[GitHub Repo](https://github.com/milla-jovovich/mempalace)
