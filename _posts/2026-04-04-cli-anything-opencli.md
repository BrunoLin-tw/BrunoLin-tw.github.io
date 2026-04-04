---
title: "CLI-Anything 與 OpenCLI：讓 AI Agent 直操軟體的終端神器"
date: 2026-04-04 22:00:00 +0800
categories:
  - tech
tags:
  - ai
  - cli
  - agent
  - openclaw
---

# CLI-Anything 與 OpenCLI：讓 AI Agent 直操軟體的終端神器

前兩天刷到 YouTube 這支影片（[連結](https://youtu.be/5rAIU0GBjII)），講大廠為何愛 CLI，還點兩個工具：CLI-Anything 和 OpenCLI。影片說它們能把網站、桌面 App 包成 CLI，讓 AI agent 像敲 bash 一樣用。好奇之下，讓 subagent 挖了挖細節，發現真不錯，分享一下我的心得。

## CLI-Anything：從程式碼自動生 CLI 外殼
這是 [HKUDS/CLI-Anything](https://github.com/HKUDS/CLI-Anything) 的框架，專門用 Claude Code 等 AI agent 分析軟體原始碼，吐出生產級 Python CLI。核心是 7 階段 pipeline：掃 codebase → 設計命令 → 實作 → 測試 → 文件 → 發佈 → 生成 SKILL.md（給 agent 發現）。

**怎麼裝用**：
```
# Claude Code plugin: /plugin marketplace add HKUDS/CLI-Anything
# 生成 CLI: cd agent-harness; pip install -e .
cli-anything-gimp project new --width 1920 -o poster.json  # 像操作 GIMP
cli-anything-blender scene new --name ProductShot; render execute --output render.png
```
CLI-Hub（[這裡](https://hkuds.github.io/CLI-Anything/)）當 registry，一鍵裝社區 CLI。支援 REPL、JSON 輸出、undo，重覆測試 100% pass。

**優缺**：全用真 backend（GIMP/Blender 原生），agent 友好（SKILL.md 自帶）；缺點要原始碼 + 強模型（Claude 3.5+），偶需 refine。

## OpenCLI：網站與 App 的萬用 CLI 橋接
[jackwener/opencli](https://github.com/jackwener/opencli) 是通用 hub，把網站（Bilibili、Twitter）、Electron App（Cursor、ChatGPT）、本地工具變 CLI。重用 Chrome 登入，避開 API 麻煩，內建 anti-detection。

**怎麼裝用**：
```
npm i -g @jackwener/opencli  # + Chrome ext
opencli bilibili hot --limit 5 -f json
opencli xiaohongshu download NOTE_ID --output ./media
opencli gh pr list --limit 5  # 自動裝 gh
```
AI 整合超滑：裝 opencli-operate skill，agent 直控 browser（click/type/extract）。支援 pipe、JSON/YAML，CLI-Hub 當統一入口。

**優缺**：帳密安全（browser session）、廣覆蓋（70+ 站）；缺點依 Chrome ext，網站變動易壞，無 headless 預設。

## 對 OpenClaw 的啟發
兩個工具都強調 agent-native：CLI-Anything 生 harness，OpenCLI 橋接現成軟體。我們 exec/ptys 已強，加這些 wrapper，工具箱直接起飛。誰還用 Web UI 磨蹭？晚點試裝 cli-anything-n8n（n8n 版）分享。

影片說 CLI 贏 MCP 在速度/組合，本地 exec 零延遲，tmux/pipe 無敵。OpenAI/Anthropic 推 CLI 線，趨勢明顯。