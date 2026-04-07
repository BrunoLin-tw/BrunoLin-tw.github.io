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

前兩天 YouTube 刷到這影片（[連結](https://youtu.be/5rAIU0GBjII)），提到最近CLI工具紛紛出爐，尤其是Google還開源了Google Workspace CLI，用命令列就可以操作Google雲端服務，輸出結構化JSON讓AI工具可以讀取。
影片中圍繞這個主題介紹了兩個工具：CLI-Anything 和 OpenCLI。說是把網站、App 包成 CLI，讓 agent 像敲 bash 一樣直用。挖了點資料，覺得實用，來簡單說說。

## CLI-Anything：碼變 CLI 框架
這是香港大學提出的開源專案 [HKUDS/CLI-Anything](https://github.com/HKUDS/CLI-Anything)，讓 Claude Code 等 agent 自己查看原始碼，分析理解後吐出 Python CLI，中間設計了 7 個步驟：掃描→分析→設計→實現→寫測試文件→使用文件→驗證安裝 最後生成 SKILL.md（agent 好找）。

安裝範例：
```
# Claude plugin: /plugin add HKUDS/CLI-Anything
cd agent-harness; pip install -e .
cli-anything-gimp project new --width 1920  # 像用 GIMP
cli-anything-blender render --output shot.png
```

另外，也可以這裡直接安裝已經做好的各種熱門CLI
[CLI-Hub](https://hkuds.github.io/CLI-Anything/) 一鍵下載別人包的。支援 REPL、JSON 出、undo。repo 說 test cover 全。

## OpenCLI：網站 App 全包 CLI
另一個工具是 [jackwener/opencli](https://github.com/jackwener/opencli)，這個工具則是可以把網站（Bili、Twitter）、Electron（Cursor）、工具全變 CLI，讓AI agent直接使用命令來操作網站。借你 Chrome 登入，躲反爬。

```
npm i -g @jackwener/opencli  # 加 ext
opencli bilibili hot --json
opencli gh pr list  # 沒 gh 自動裝
```
AI 裝 opencli-operate skill，就能 browser 點擊/抓資料。pipe 超順。

好處：登入安全、蓋多；缺點 Chrome 綁定，網站改版就要跟著改。

## OpenClaw 怎麼用
Openclaw 透過技能使用CLI 比 MCP 快又靈，建議大家試試看。
