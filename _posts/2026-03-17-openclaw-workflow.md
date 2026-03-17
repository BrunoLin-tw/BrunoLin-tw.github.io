---
title: "OpenClaw 自動化工作流：我的 AI 助理如何幫我天天做事"
date: 2026-03-17 21:10:00 +0800
categories:
  - tech
tags:
  - openclaw
  - automation
  - workflow
  - ai-assistant
---

## 前言：讓 AI 幫我處理例行公事

身為一個忙碌的上班族 + 業餘跑者 + AI 愛好者，我每天都有很多「例行小事」要做：
- 查天氣
- 看財經新聞
- 追蹤 AI 產業動態
- 記錄訓練進度

以前都是手動做，久了就懶得做。後來我讓 OpenClaw（我的 AI 助理）幫我自動化這些事情，從此早上起床，手機一打開就是整理好的資訊。

**這篇文分享一下我的自動化工作流設計思路。**

---

## Cron Job 實作：讓 AI 準時報到

OpenClaw 的 `cron` 功能是核心。我把常用設定整理成模板：

```json
{
  "schedule": {
    "kind": "cron",
    "expr": "30 8 * * 1-5",
    "tz": "Asia/Taipei",
    "staggerMs": 30000
  },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "任務指示..."
  },
  "delivery": {
    "mode": "announce",
    "channel": "telegram",
    "to": "871347964"
  }
}
```

### 關鍵參數說明

| 參數 | 用途 |
|------|------|
| `schedule.kind` | `cron` 定期執行，`at` 一次性 |
| `schedule.tz` | 時區設定，台灣設 `Asia/Taipei` |
| `sessionTarget` | `isolated` 隔離執行，不影響主對話 |
| `delivery` | 執行完後自動推送到 Telegram |

---

## 實際應用案例

### 1. 財經早報（每日 8:30 平日）

自動分析美股四大指數 + 台積電 ADR + 台股盤前策略，600 字精華，直接推送到手機。

```json
{
  "name": "財經早報-美股台股分析",
  "schedule": {"expr": "30 8 * * 1-5"},
  "model": "gpt5.4"
}
```

### 2. AI Daily Newsletter（每日 8:55）

這是最複雜的自動化之一：
- 先搜尋當日 AI 新聞（Tavily + Brave）
- 檢查 news-ledger.json 避免重複（見前一篇）
- 生成 5-6 則精選新聞
- 發布到 GitHub Pages
- 推送到 Telegram

```json
{
  "name": "AI Daily Newsletter Generation",
  "schedule": {"expr": "55 8 * * *"},
  "model": "gpt5.4",
  "thinking": "medium"
}
```

### 3. 天氣報告（每日 7:10）

查詢板橋天氣，輸出固定格式：
```
今日：高 24° 低 20° 雨 30% 體感 23° 風北
明日：高 26° 低 21° 雨 20% 體感 25° 風北
提醒：帶傘
```

### 4. 訓練提醒（每天 7:00 + 晚上 20:50）

根據星期幾自動調整訓練菜單：
- 週一/五：走跑交替 + 核心
- 週二/六：腿部肌力（離心訓練）
- 週三：走跑交替
- 週四：休息/游泳
- 週日：完全休息

晚上 20:50 提醒記錄訓練筆記，格式都訂好了，直接回覆即可。

### 5. 盤後分析（每日 15:15 平日）

讀取持股組合，分析大盤和個股表現，輸出 I/II/III 三段落格式。

---

## 模型切換策略：省錢與效率的平衡

不同任務用不同模型：

| 任務 | 模型 | 理由 |
|------|------|------|
| 財經分析、Newsletter | gpt5.4 | 需要深度推理 |
| 天氣、簡單查詢 | m2.5 | 快速、便宜 |
| 盤後分析 | gpt5.4 | 需要邏輯判斷 |

OpenClaw 的模型切換只要在 payload 加一行 `"model": "gpt5.4"` 即可。

---

## 一次性任務：記得加 deleteAfterRun

如果任務只跑一次（例如盤中提醒），務必加上：

```json
{
  "deleteAfterRun": true,
  "schedule": {"kind": "at", "at": "2026-03-18T01:10:00.000Z"}
}
```

否則 cron job 會一直留在清單裡，越積越多。

---

## 工作流設計原則

1. **例行任務用 cron，一次性任務用 at + deleteAfterRun**
2. **複雜任務用 isolated session**，避免影響主對話
3. **delivery 設為 announce**，執行完自動推送到 Telegram
4. **固定格式輸出**，減少來回溝通成本
5. **模型不是越貴越好**，選對的任務用對的模型

---

## 結語

自動化不是「讓 AI 取代人類」，而是「讓 AI 幫我處理我不想記得的小事」。

把例行資訊蒐集交給 OpenClaw，我把時間留給真正需要人類判斷的事情——例如訓練怎麼調整、下週出差要帶什麼衣服。

這才是 AI 助手正確的使用方式。

---

*延伸閱讀：[AI Newsletter 去重機制優化](/blog/2026/03/17/ai-newsletter-dedup)*
