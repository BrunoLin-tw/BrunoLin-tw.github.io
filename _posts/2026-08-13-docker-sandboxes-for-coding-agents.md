---
title: "Docker Sandboxes：把 Codex 和 OpenCode 關進 microVM"
date: 2026-08-13 22:15:55 +0800
categories:
  - tech
tags:
  - docker
  - ai-agents
  - codex
  - opencode
  - sandbox
---

最近使用 Codex 和 OpenCode 的頻率越來越高，也開始遇到一個很實際的問題：coding agent 能讀檔、改程式、執行 shell，甚至啟動 Docker。能力愈完整，失控時能造成的影響也愈大。

![Docker Sandboxes 架構示意圖](/assets/images/posts/docker-sandboxes-architecture.png)

Docker Sandboxes 使用 `sbx` CLI，把 coding agent 放進獨立的 Linux microVM。Agent 可以在裡面安裝套件、執行測試、啟動 container，host 的 kernel、process 和 Docker daemon 則留在隔離邊界之外。

## 和普通 Docker container 有什麼差別？

一般 container 與 host 共用 kernel。如果為了讓 agent 操作 Docker，把 `/var/run/docker.sock` 掛進 container，agent 幾乎等同取得 host Docker daemon 的控制權，還能建立 privileged container 或掛載 host filesystem。

Docker Sandboxes 裡的每個 agent 都有自己的：

- Linux kernel、filesystem 和 network
- Docker daemon、image、container 和 volume
- 套件與執行狀態

Agent 在 VM 裡可以使用 `sudo`、`apt`、`pip`、`npm`、Docker 與 Compose。它建立的 container 不會出現在 host 的 `docker ps`；刪除 sandbox 後，VM 內的環境也會一起清除。

這個設計很適合 autonomous agent：VM 內給足權限，VM 外維持明確邊界。

## Workspace 才是最容易忽略的地方

直接執行：

```bash
sbx run codex
```

預設採用 direct mode，專案目錄會以 read-write 掛入 VM：

```text
workspace  /home/user/project (rw)
```

Codex 的修改會立即出現在 host。它也能改動 CI 設定、Git hooks、Makefile 和 `package.json` scripts。MicroVM 保護了系統，專案內容仍可能受到影響。

我會從 clone mode 開始：

```bash
sbx run --clone --no-share-skills codex
```

`--clone` 會在 VM 裡建立獨立的 Git clone，agent 的修改要等 host 明確執行 `git fetch` 才會帶回來。`--no-share-skills` 則關閉預設的 shared skills 掛載，避免某個 sandbox 修改共用 skill，影響其他 agent session。

## 安裝與兩套登入

Linux 需要 Ubuntu 24.04 以上版本與 KVM：

```bash
lsmod | grep kvm
sudo usermod -aG kvm "$USER"
newgrp kvm
```

安裝 `sbx`：

```bash
curl -fsSL https://get.docker.com |
  sudo REPO_ONLY=1 sh

sudo apt-get install docker-sbx
```

Docker Sandboxes 有兩套容易混淆的認證：

```text
sbx login                      Docker 帳號，用來下載 sandbox image
sbx secret set openai --oauth OpenAI 帳號，讓 Codex 呼叫模型
```

`docker login` 不能取代 `sbx login`。如果啟動時看到：

```text
no default account profile set
secret not found
```

通常是 `sbx` 還沒有有效的 Docker 登入紀錄。Headless Linux 可以使用 Docker Personal Access Token：

```bash
read -rsp "Docker PAT: " DOCKER_PAT
printf '%s' "$DOCKER_PAT" |
  sbx login --username YOUR_DOCKER_ID --password-stdin
unset DOCKER_PAT
```

登入資訊會保存在目前 Linux user 的 credential store，SSH 中斷或重新開機後仍可使用。環境異常時先跑：

```bash
sbx diagnose
```

## Headless 主機的 Codex OAuth 還有缺口

`sbx secret set openai --oauth` 目前使用 localhost callback。若 `sbx` 跑在遠端 Linux，瀏覽器開在另一台電腦，callback 會送到瀏覽器所在電腦的 localhost，遠端主機收不到。

現階段可用 SSH tunnel 處理。先在遠端執行 OAuth，記下 callback port，例如 `1455`，再從有瀏覽器的電腦建立 tunnel：

```bash
ssh -N \
  -L 1455:127.0.0.1:1455 \
  user@remote-host
```

接著開啟 `sbx` 顯示的 OAuth URL，登入後的 callback 就會回到遠端。

Codex CLI 已有 `codex login --device-auth`，但這份 credential 不會進入 `sbx` 的 host-side credential proxy。Docker 的 issue tracker 已有人要求替 `sbx` 加入 device-code flow，目前仍要靠 SSH tunnel 或 OpenAI API key。

API key 的設定比較直接：

```bash
sbx secret set openai
```

但 OpenAI API 與 ChatGPT Plus／Pro 分開計費，選之前要先確認使用成本。

## 我會採用的工作流

Codex：

```bash
cd ~/projects/my-project

sbx run \
  --name codex-my-project \
  --clone \
  --no-share-skills \
  codex .
```

OpenCode 的啟動方式相同：

```bash
sbx run \
  --name opencode-my-project \
  --clone \
  --no-share-skills \
  opencode .
```

我會要求 agent 建立獨立 branch、完成修改和測試，但先不 push。完成後在 host 端取回成果：

```bash
git fetch sandbox-codex-my-project
git log sandbox-codex-my-project/agent/my-task
git diff main..sandbox-codex-my-project/agent/my-task
```

確認內容後才建立本地 branch，並在可信任環境重新跑一次測試。Sandbox 能降低 agent 破壞 host 的風險，code review 和驗證仍然要做。

## 我的看法

Docker Sandboxes 提供了一個合理的 agent 權限模型：microVM 內允許高度自主，workspace、網路、credential 和 Git 成果則由外層機制管理。

我目前會從這個組合開始：

```text
microVM
+ Git clone mode
+ 關閉 shared skills
+ Balanced network policy
+ host-side credential proxy
+ agent 不直接 push
+ host 端 review 與驗證
```

多幾個步驟，換來的是 Codex 和 OpenCode 可以更自由地執行工作，而工程師仍保留最後的控制權。當 coding agent 從輔助工具走向自主執行，這類隔離層會逐漸成為標準配備。

Reference:

- [Docker Sandboxes](https://docs.docker.com/ai/sandboxes/)
- [Docker Sandboxes Architecture](https://docs.docker.com/ai/sandboxes/architecture/)
- [Docker Sandboxes Security Model](https://docs.docker.com/ai/sandboxes/security/)
- [Docker Sandboxes Workflow Patterns](https://docs.docker.com/ai/sandboxes/workflows/)
- [Codex in Docker Sandboxes](https://docs.docker.com/ai/sandboxes/agents/codex/)
- [OpenCode in Docker Sandboxes](https://docs.docker.com/ai/sandboxes/agents/opencode/)
- [Docker Sandboxes Troubleshooting](https://docs.docker.com/ai/sandboxes/troubleshooting/)
- [Headless OpenAI OAuth Feature Request](https://github.com/docker/sbx-releases/issues/208)
