---
title: "Sub-agent 搭配 Git Worktree 平行開發"
date: 2026-03-22 23:06:00 +0800
categories:
  - tech
tags:
  - git
  - openclaw
  - agent
  - coding
  - workflow
---

最近在玩 OpenClaw 的 sub-agent 功能時，突然想到一個很實用的組合：**Git Worktree + Sub-Agent**。這套玩法讓你能在單一 repo 裡同時讓多個 coding AI agent 工作，每個專注一個分支/任務，完全隔離，效率爆表。

想像一下：你要同時做 UI 新功能 + API bugfix + 文件更新，傳統方式得 stash → switch branch → 解釋上下文給 AI → 等 npm install。換用這套，**一鍵 setup worktree + spawn agent**，AI 直接在獨立環境幹活。

## 什麼是 Git Worktree？
Git Worktree 讓單 repo 同時 checkout 多分支，每個分支獨立目錄，但共享 .git：
```
git worktree add ../repo-ui feature-ui
git worktree add ../repo-api bugfix-api
```
結果：
- `~/repo` (main)
- `~/repo-ui` (feature-ui，獨立 node_modules）
- `~/repo-api` (bugfix-api)

## 什麼是 OpenClaw Sub-Agent？
用 `sessions_spawn` 生獨立 AI session：
```
sessions_spawn task="UI feature" cwd="../repo-ui" runtime="acp" agentId="codex"
```
每個 agent 有獨立 cwd/context，不會互相干擾。

## 組合威力：並行開發軍團
1. **Setup**：
   ```
   cd ~/your-repo
   skills/git-worktree-subagent/scripts/create-worktree.sh feature-ui
   ```
   自動：worktree + npm ci + .env copy + VS Code 開啟。

2. **Spawn Agents**：
   ```
   skills/git-worktree-subagent/scripts/spawn-codex.sh feature-ui bugfix-api
   ```
   生成：
   ```
   sessions_spawn runtime="acp" agentId="codex" cwd="../repo-feature-ui" task="Add login UI"
   ```

3. **監控 + Merge**：
   ```
   subagents list
   skills/git-worktree-subagent/scripts/merge-worktrees.sh
   ```

## 完整 Demo（我剛跑過）
```
# 1. Init demo repo
mkdir ~/demo && cd ~/demo && git init && ... (branches ready)

# 2. Worktrees
scripts/create-worktree.sh feature-ui bugfix-api

# 3. Agents running in parallel
Agent1 (repo-ui): "Add button to index.html → commit"
Agent2 (repo-api): "Fix error in api.js → commit"

# 4. Merge
scripts/merge-worktrees.sh  # git merge + prune
```

## 避坑指南
- **cwd 絕對路徑**：agent 只改自己 tree
- **.env/node_modules**：script 自動 copy/install
- **Codex CLI**：用 `runtime="acp" agentId="codex"`
- **Cleanup**：`git worktree prune`

## 為什麼強？
- **效率**：從串行 → 並行，任務 2-3x 快
- **隔離**：無上下文污染
- **scale**：5+ agents 同時跑（UI/DB/Test/Docs）
- **成本低**：單 repo，無重複 clone

## Skill 已做好（已 push workspace）
`~/workspace-bobo/skills/git-worktree-subagent`
- SKILL.md + references（HackMD/Medium 精華）
- scripts：setup/spawn/merge/create-worktree

裝載後，直接觸發：
> "用 git-worktree-subagent 在 repo 並行開發 UI + API"

## 參考
- [HackMD: Claude Sub-Agents](https://hackmd.io/@BASHCAT/BJgGV1zvll)
- [Medium: Multi-AI Worktrees](https://medium.com/@jatin4228/git-worktrees-run-multiple-ai-agents-copilot-claude-code-on-different-branches-at-once-ba46cd8e0ae7)

試試看？指定 repo 我幫你跑 demo！🐻
