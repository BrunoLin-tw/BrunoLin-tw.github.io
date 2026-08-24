---
title: "自架 Immich 取代 Google Photos：Docker 安裝與五萬張照片匯入記錄"
date: 2026-08-24 11:05:00 +0800
categories:
  - tech
tags:
  - immich
  - self-hosted
  - docker
  - google-photos
---

家裡的照片從 2008 年累積到現在，全放在 Google Photos。最近認真評估搬到自架，測了一輪之後把過程記下來：Immich 是目前最接近 Google Photos 使用體驗的開源方案，遷移成本比我想像中低。

![有了 Immich，就把照片搬回自己家](/assets/images/posts/2026-08-24-immich-self-hosted-google-photos.jpg)

這篇記錄在 Ubuntu 24.04 上用 Docker Compose 安裝 Immich、把五萬多張照片和影片從 Google Takeout 匯入的完整過程，包含最後踩到的坑。

## Immich 是什麼

手機自動備份、網頁時間軸、人臉與場景搜尋、地圖、共用相簿、分享連結，Google Photos 常用的功能大致都有對應。GitHub 上已經超過 100K stars，目前最新版是 v3.1（2026 年 7 月底），更新節奏很快。

先講結論：以「取代 Google Photos」這個目標來說，Immich 是我第一個測完就不想再看其他方案的專案。

## 測試環境

| 項目 | 內容 |
|---|---|
| OS | Ubuntu 24.04 LTS |
| CPU / RAM | 8 核 / 15 GB |
| GPU | Intel HD 530（Quick Sync） |
| 安裝方式 | Docker Compose |
| 匯入規模 | 約 5.4 萬張照片與影片，原始檔 76 GB |

機器是一台跑了好幾年的桌機，沒有獨立顯示卡。Immich 的機器學習功能（人臉辨識、智慧搜尋）在這種環境只能吃 CPU，這點後面會提到。

## Docker Compose 安裝

官方需要的檔案就三個，從 release 頁面直接抓：

```bash
mkdir immich && cd immich
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O hwaccel.transcoding.yml https://github.com/immich-app/immich/releases/latest/download/hwaccel.transcoding.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

`.env` 要改的重點：

```text
UPLOAD_LOCATION=/path/to/library      # 照片實際存放位置
DB_DATA_LOCATION=./postgres           # 資料庫位置，不可放網路磁碟
TZ=Asia/Taipei
```

`DB_PASSWORD` 官方建議只用英數字元，避開特殊字元在各種 shell 展開的麻煩。

### 啟用 Quick Sync 硬體轉碼

HD 530 支援 Quick Sync，影片轉碼可以不占 CPU。兩個步驟：

1. 主機裝驅動並把使用者加入 video 群組：

```bash
sudo apt install -y intel-media-va-driver-non-free vainfo
```

2. `docker-compose.yml` 的 `immich-server` service 加上：

```yaml
extends:
  file: hwaccel.transcoding.yml
  service: quicksync
```

這裡有個小坑：主機端直接跑 `vainfo` 會段錯誤，一度以為驅動有問題。實際上是 GNOME 桌面環境的干擾，容器內的 iHD 驅動載入完全正常，H264 編碼和 HEVC 解碼都可用。驗證硬體加速要在容器內做，別被主機端的結果騙了。

啟動：

```bash
docker compose up -d
docker compose ps   # 四個容器都要 healthy
```

四個容器分別是 server、postgres、machine-learning、redis（現在用的是 Valkey）。全部 healthy 之後打開 `http://IP:2283` 建立管理員帳號。

另外 postgres 用的是 Immich 特製版，內建 vectorchord 向量索引，智慧搜尋不需要自己再裝 extension。

## 用 immich-go 匯入 Google Takeout

批量匯入社群的主流做法是用 immich-go，一個獨立 CLI，對 Google Takeout 格式的處理比 Immich 內建上傳完整許多。

流程：

1. 到 [Google Takeout](https://takeout.google.com/) 只勾選 Photos，匯出格式選 zip，單檔大小可以拉到 50GB
2. Immich 網頁上建立一組 API Key
3. 執行匯入：

```bash
immich-go upload from-google-photos \
  --server=http://192.168.x.x:2283 \
  --api-key=你的APIKey \
  takeout-*.zip
```

zip 不需要事先解壓縮。

實際執行時我用的完整指令（Windows 環境，`XXXX` 換成自己的 API Key）：

```powershell
.\immich-go.exe upload from-google-photos `
  --server=http://192.168.1.102:2283 `
  --api-key=XXXX `
  --admin-api-key=XXXX `
  --concurrent-tasks=2 `
  --manage-raw-jpeg=KeepJPG `
  --manage-burst=NoStack `
  --log-level=INFO `
  --pause-immich-jobs=true `
  --client-timeout=60m `
  --include-unmatched `
  --session-tag `
  --log-file=.\immich_import_info.log `
  .\takeout-*.jpg
```

幾個參數值得說明：

| 參數 | 用途 |
|---|---|
| `--pause-immich-jobs=true` | 匯入期間暫停 Immich 背景工作，頻寬和磁碟 I/O 全讓給上傳 |
| `--concurrent-tasks=2` | 同時上傳 2 個任務，保守值，避免小機器吃不消 |
| `--client-timeout=60m` | 單一請求 timeout 拉到 60 分鐘，大影片才不會傳到一半被砍 |
| `--include-unmatched` | JSON 對不起來的檔案也照樣匯入，Takeout 常有 metadata 缺漏 |
| `--session-tag` | 這次匯入的檔案打上 session 標籤，中斷重跑或事後追蹤都方便 |
| `--manage-raw-jpeg=KeepJPG` | RAW+JPEG 成對時保留 JPEG 版本 |
| `--manage-burst=NoStack` | 連拍不自動 stack，維持一張一張獨立 |

immich-go 對 Takeout 做了三件關鍵處理：

- 讀取每張照片附帶的 `.supplemental-metadata.json`。Google 匯出時會拔掉部分 EXIF，尤其經過他們壓縮儲存的照片，拍攝時間靠這些 JSON 才救得回來，不然整條時間軸會變成匯出當天的日期
- 相簿、收藏（favorite）、封存狀態原樣保留
- 連拍和 Live Photo 合併成 stack，呈現方式跟 Google Photos 一致

### 匯入結果

| 項目 | 數量 |
|---|---|
| 照片 | 51,354 |
| 影片 | 2,272 |
| 相簿 | 191 |
| Stacks | 6,752 |

Stacks 數字看起來和 immich-go 的 `--manage-burst=NoStack` 矛盾，其實不衝突：NoStack 只針對連拍，Takeout 裡 RAW+JPEG 成對、Live Photo 這類組合仍會被合併成 stack，這部分是預期行為。

匯入過程沒有卡死或斷線。真正的瓶頸在匯入之後：縮圖生成和影片轉碼由 Immich 背景 job 跑，5 萬張的規模要消化好一陣子，期間瀏覽不受影響。

磁碟空間要注意，衍生檔案比想像中大：原始檔 76 GB 之外，縮圖和轉碼後的影片各占約 21 GB。估算容量時建議在原始檔之外再多抓 50%。

## 踩到的坑

### 刪除 stack 內的照片觸發 FK 錯誤

測試刪除功能時，server log 冒出大量 `PostgresError`：刪除 asset 時違反 stack 表的 foreign key constraint（`stack_primaryAssetId_fkey`），三天累積近五千筆。

成因是 stack 的主要資產被刪掉時，引用沒有先清乾淨。實際影響偏向批次/API 刪除情境，網頁上手動刪除目前沒遇到異常。這類問題通常靠升級解決，先記錄追蹤。

### XMP sidecar 找不到的警告

metadata 掃描階段出現一批 `.jpg.xmp` 不存在的 WARN，推測是反覆測試匯入時 sidecar 被清理掉的殘留狀態。不影響瀏覽，但如果跟我一樣會重複測試，看到這類警告先確認是不是舊資料造成的。

### ML 只能吃 CPU

人臉辨識和智慧搜尋的 embedding 全部在 CPU 上算。五萬張的全庫掃描要跑很久，好在都是背景 job，不擋操作。ML 容器其實有 OpenVINO 版本可以試，HD 530 理論上有支援，這次沒測，列在下一次的待辦。

## 閒置資源用量

跑三天之後的數字：

| 容器 | 記憶體 |
|---|---|
| server | 1.6 GB |
| postgres | 0.7 GB |
| machine-learning | 37 MB |
| redis | 17 MB |

合計約 2.4 GB。15 GB RAM 的機器綽綽有餘；如果機器只有 4 GB RAM，會比較勉強。

## 還沒測的部分

- 手機 App 自動備份（下一步）
- 反向代理與 HTTPS，目前只在區網使用
- OpenVINO 版 ML 容器

## 小結

從裝機到五萬張照片完成匯入，前後跨兩天，真正動手的時間很少，大部分在等 Takeout 打包和背景 job 消化。Immich 把自架相簿的門檻降到會用 Docker 就能架，資料格式也乾淨：原始檔按年份和日期目錄存放，就算資料庫壞了，檔案本身仍然完好，這對長期保存很關鍵。

我的計畫是先在區網跑一陣子，確認手機備份穩定、ML job 收斂之後，再把 Google Photos 的雲端副本收掉。搬家不用急，Takeout 隨時可以重新匯出一次，風險很低。

Reference:

- [Immich 官方文件](https://immich.app/docs)
- [immich-go](https://github.com/simulot/immich-go)
- [Google Takeout](https://takeout.google.com/)
