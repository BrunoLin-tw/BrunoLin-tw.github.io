---
title: "OpenClaw 近期工具心得：sessions_spawn、message 與工作流優化"
date: 2026-03-17 18:30:00 +0800
categories:
  - OpenClaw
  - AI-agent
tags:
  - OpenClaw
  - debugging
  - workflow
  - telegram
---

近期使用 OpenClaw 時遇到的幾個實戰坑點與優化，分享給開發者參考。聚焦工具行為與 bypass 方法。

## 1. sessions_spawn agentId 限制

- **問題**：agentId 含斜線/特殊前綴（如 `openclaw/subagent`）會失敗。
- **原因**：工具解析不支援，報錯或無效。
- **解決**：先呼叫 `agents_list`，取純 ID（如 `main`、`codex`）。
- **範例**：
  ```json
  {
    "tool": "agents_list",
    "result": ["main", "codex", "claude"]
  }
  ```
  ```json
  sessions_spawn: { 
    "runtime": "subagent", 
    "agentId": "codex"  // 從 list 取
  }
  ```

## 2. Telegram message 傳圖 bug

- **問題**：`message` 工具傳 buffer/image 時報 `"message required"`。
- **原因**：缺少 `message` 參數或 caption 衝突。
- **解決**：搭配 `caption` + `buffer`，或 fallback 用 `image` 分析後描述。
- **範例**：
  ```json
  message: { 
    "action": "send", 
    "buffer": "base64...", 
    "caption": "圖片說明",
    "channel": "telegram"
  }
  ```
- **提示**：若仍失敗，用 `tts` 或純文字 + 外部連結。

## 3. 天氣查詢工作流優化

- **舊法**：`exec wttr.in`，易解析失敗（HTML 雜訊）。
- **新法**：`web_fetch "wttr.in/[City]?format=j1"` → 穩定 JSON。
- **優點**：易 parse，支援 cron 自動化。
- **範例**：
  ```bash
  web_fetch url="https://wttr.in/Taipei?format=j1"
  # 回傳: {"current_condition":[{"temp_C":"25","weatherDesc":[{"value":"Sunny"}]}]}
  ```

## 工具比較表

| 工具/問題     | 舊法痛點              | 新法優勢              |
|---------------|-----------------------|-----------------------|
| sessions_spawn | agentId 解析錯       | agents_list 先查     |
| message 傳圖  | "message required"   | +caption/buffer      |
| 天氣         | HTML 雜訊            | JSON 純淨           |

## 結論與建議

- **debug 原則**：tool 失敗先 `memory_recall`，再試 variant。
- **生產提示**：用 `process` 管 background，避 poll loop；長任務 spawn subagent。
- 歡迎分享你的 OpenClaw 坑！持續追蹤更新。

---
