---
title: "CLI-Anything 跟 OpenCLI：AI Agent 的終端好幫手"
date: 2026-04-05 08:00:00 +0800
categories:
  - tech
tags:
  - ai
  - cli
  - agent
  - openclaw
---

# CLI-Anything 跟 OpenCLI：AI Agent 的終端好幫手

前兩天 YouTube 隨刷到這影片（[連結](https://youtu.be/5rAIU0GBjII)），聊大廠為啥推 CLI，還提兩個工具：CLI-Anything 和 OpenCLI。說是把網站、App 包成 CLI，讓 agent 像敲 bash 一樣直用。挖了點資料，覺得實用，來簡單說說。

## CLI-Anything：碼變 CLI 框架
這是 [HKUDS/CLI-Anything](https://github.com/HKUDS/CLI-Anything)，讓 Claude Code 等 agent 看原始碼，吐 Python CLI。走 7 步：掃描→設計→寫→測→檔→包→生 SKILL.md（agent 好找）。

大致這樣裝：
```
# Claude plugin: /plugin add HKUDS/CLI-Anything
cd agent-harness; pip install -e .
cli-anything-gimp project new --width 1920  # 像用 GIMP
cli-anything-blender render --output shot.png
```
[CLI-Hub](https://hkuds.github.io/CLI-Anything/) 一鍵下載別人包的。支援 REPL、JSON 出、undo。repo 說 test cover 全。

爽在真用軟體 backend，agent 直抓；煩在要碼跟強模型。

## OpenCLI：網站 App 全包 CLI
[jackwener/opencli](https://github.com/jackwener/opencli)，網站（Bili、Twitter）、Electron（Cursor）、工具全變 CLI。借你 Chrome 登入，躲反爬。

```
npm i -g @jackwener/opencli  # 加 ext
opencli bilibili hot --json
opencli gh pr list  # 沒 gh 自動裝
```
AI 裝 opencli-operate skill，就能 browser 點擊/抓資料。pipe 超順。

好處：登入安全、蓋多；缺點 Chrome 綁定，網站改易壞。

## OpenClaw 怎麼用
我們 exec/ptys 搭這些，工具直接滿分。CLI 比 MCP 快又靈，OpenAI/Anthropic 都轉。想試 n8n 版，之後說說看。
