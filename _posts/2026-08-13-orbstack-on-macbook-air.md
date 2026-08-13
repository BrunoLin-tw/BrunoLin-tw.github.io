---
title: "MacBook Air 上的 Docker 開發環境，我選 OrbStack"
date: 2026-08-13 22:51:41 +0800
categories:
  - tech
tags:
  - macos
  - orbstack
  - docker
  - apple-silicon
---

最近換了 13 吋 MacBook Air M5，重新整理開發環境時，Docker Desktop 和 OrbStack 該選哪一個，很快就成了第一個問題。

Docker Desktop 功能完整，也是最多人熟悉的方案。不過我更在意的是背景資源、電力消耗，以及 Apple Silicon 上執行 ARM 與 x86 container 的體驗。查完 OrbStack 的架構和功能後，我決定先用 OrbStack。

## OrbStack 在做什麼？

OrbStack 是 macOS 專用的 container 與 Linux 執行環境。它內建 Docker Engine、Docker Compose、Buildx、Kubernetes，以及 Linux machine 管理功能。原本使用的 Dockerfile、Compose 專案與 VS Code Dev Containers，大多可以沿用。

安裝後，終端機裡還是使用熟悉的指令：

```bash
docker build .
docker compose up -d
docker ps
```

差別主要藏在底層。OrbStack 使用針對 macOS 與 Apple Silicon 最佳化的輕量 Linux VM，Docker Engine 和 Linux machines 共用同一個 kernel。檔案共享以 VirtioFS 為基礎，網路、DNS、VPN 和 port forwarding 也由 OrbStack 自己整合。

## MacBook Air 最需要的是資源效率

MacBook Air 沒有風扇，長時間 build image、跑資料庫或啟動多個 service 時，背景工具的 CPU 與記憶體占用會直接影響溫度和續航。

OrbStack 官方資料顯示，Apple Silicon 待機 CPU 約為 0.1%，有時會降到 0%。記憶體採動態配置，用不到的部分會還給 macOS；磁碟空間也會隨實際資料量增減。

這些數字來自官方，不能當成所有 workload 的固定結果。不過設計方向很清楚：container runtime 應該待在背景，需要時才占用資源。對 MacBook Air 這類輕薄筆電，這比多幾個管理面板更實際。

## M5 跑 ARM image，也能處理舊的 x86 環境

M5 原生架構是 ARM64，開發時應優先使用 multi-architecture 或 `linux/arm64` image。遇到只有 x86_64 版本的舊工具，OrbStack 可以透過 Rosetta 執行 `linux/amd64` image：

```bash
docker run --rm --platform linux/amd64 alpine uname -m
docker build --platform linux/amd64 -t demo .
```

這對需要測試舊版 SDK、資料庫或現成 CI image 很方便。不過 Rosetta 只能降低轉換成本，無法消除所有架構差異。碰到驅動程式、kernel module 或硬體相關 SDK，仍要逐項驗證。

## Docker 之外，還附了一套輕量 Linux 環境

OrbStack 可以直接建立 Ubuntu、Debian、Arch 等 Linux machines，並透過 `orb` 或 SSH 進入。macOS 與 Linux 之間的檔案交換、SSH agent forwarding 和 VS Code Remote SSH 都已整合。

它也內建單節點 Kubernetes。Container image 可直接給 Kubernetes 使用，不必先推到本機 registry。若需要 multi-node cluster，仍可以在 OrbStack 上使用 kind 或 k3d。

這讓它比單純的 Docker Desktop 替代品多了一層用途。我可以用 container 跑服務，也能快速開一個 Linux machine 測試工具或 script，不必另外管理一套 VM 軟體。

## 安裝與基本驗證

使用 Homebrew 安裝：

```bash
brew install orbstack
```

啟動 OrbStack 後，可以先確認環境：

```bash
docker context show
docker version
docker compose version
docker run --rm hello-world
```

Docker context 應該會顯示：

```text
orbstack
```

如果電腦原本有 Docker Desktop，也能先讓兩套環境並存，再用 Docker context 切換：

```bash
docker context use orbstack
docker context use desktop-linux
```

OrbStack 也能複製 Docker Desktop 的 images、containers 與 volumes。確認專案都能正常執行後，再移除舊環境會比較穩妥。

## 我的選擇

以個人使用來說，OrbStack 免費，授權沒有負擔。它保留 Docker CLI 和 Compose 工作流，又針對 Apple Silicon、記憶體回收、檔案共享與網路整合做了不少最佳化。

Docker Desktop 仍有完整生態與官方整合優勢，企業環境也可能早已統一管理。但在我的 M5 MacBook Air 上，我更重視安靜、低耗電，以及開機後不用特別照顧的 container 環境。

所以這次我先選 OrbStack。真正的檢驗很簡單：把平常的 Compose 專案、ARM/x86 image build 和開發工具全部跑過一輪，再看記憶體壓力、溫度與電池表現。工具最後還是要回到實際 workload 才算數。

Reference:

- [OrbStack](https://orbstack.dev/)
- [OrbStack Quick Start](https://docs.orbstack.dev/quick-start)
- [OrbStack Docker Containers](https://docs.orbstack.dev/docker)
- [OrbStack Architecture](https://docs.orbstack.dev/architecture)
- [OrbStack Efficiency](https://docs.orbstack.dev/efficiency)
- [OrbStack Kubernetes](https://docs.orbstack.dev/kubernetes)
- [Replacing Docker Desktop](https://docs.orbstack.dev/install#docker-desktop)
