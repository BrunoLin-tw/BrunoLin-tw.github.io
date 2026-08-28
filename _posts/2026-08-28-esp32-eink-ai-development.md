---
title: "買了一塊 32 美元的電子紙板，設計開發都用 AI 完成"
date: 2026-08-28 13:39:00 +0800
categories:
  - tech
tags:
  - esp32
  - e-ink
  - ai-agent
  - ox-alpha
  - platformio
---

我最近買了一塊 ELECROW CrowPanel 5.79 吋電子紙板，價格 31.9 美元。它有 ESP32-S3、microSD、按鍵、電池接頭和 792×272 的黑白電子紙。買的理由很單純：想做一台能長時間放在桌上、只在需要時更新的資訊裝置。

![CrowPanel 上執行的天氣看板](/assets/images/posts/2026-08-28-esp32-eink-weather.jpg)

但我得先講清楚：我不懂 ESP32 開發。這個專案裡的硬體研究、設計選擇、韌體和測試設計，都是我一路問 AI 做出來的，主要是 Ox Alpha；我做的是把板子接上去、燒錄、看 serial log，然後確認畫面和行為是否真的符合預期。

這不是「叫 AI 生一份程式碼」的故事。實體裝置不會因為程式能編譯就正常工作。真正的流程是：AI 提出假設和下一個測試，我在桌上驗證，再把真實結果丟回去。幾輪下來，原本一塊陌生的板子，變成了天氣看板和 SD 相框。

## 第一件事：先搞清楚自己買了什麼

這塊板子的資料沒有想像中乾淨。商品頁寫 272×792，官方 driver 內部卻使用 800×272 framebuffer；面板由兩顆 SSD1683 串接，可見區與 controller 的記憶體排列不是同一件事。把 792×272 的 bitmap 直接交給期待 800×272 buffer 的 API，畫面會錯位，接縫也可能出問題。

官方資料還有幾個容易踩到的點：USB-C 實際是 CH340C USB-to-UART，不是 ESP32-S3 原生 USB；這個型號沒有觸控和背光；microSD 與面板走不同的 SPI 腳位，SD 供電也得先用 GPIO42 啟用。

![CrowPanel DIS08792E 的板面與介面配置](/assets/images/posts/2026-08-28-esp32-eink-hardware-layout.webp)

這些不是我靠讀 datasheet 自己推完的。AI 先把商品頁、Wiki、原廠 repo、電路圖和 GxEPD2 的實作攤開比對，整理出一份可驗證的硬體假設；我再用實機確認能否編譯、燒錄、初始化面板、讀到按鍵和掛上 SD 卡。

這個分工很實際。AI 能快速讀完大量彼此矛盾的文件，人必須處理那條 USB 線到底能不能傳資料、板子到底有沒有亮起來這種不可遠端猜測的事情。

## Bring-up：從能刷進去，到能穩定醒來

第一階段沒有急著做應用，先把最基本的路徑跑通：PlatformIO 編譯與上傳、serial monitor、電子紙全刷／局刷、deep sleep 喚醒、按鍵、Wi-Fi 掃描和 microSD。

這裡的教訓是，電子紙的 UX 跟一般螢幕完全不同。它沒有背光，畫面停在那裡幾乎不耗電；代價是完整刷新約 4.4 秒。這不適合秒級跳動的資訊，卻很適合每 15 到 30 分鐘更新一次的看板、行程表或相框。

韌體最後固定用 PlatformIO + Arduino，ESP32 平台版本、GxEPD2 和字型函式庫都釘版本。這種看起來無聊的決定很重要：開發板一換、依賴一浮動，AI 再會推理也只是在追新的環境問題。

## 第一個可用應用：天氣看板

天氣看板抓 Open-Meteo 資料，顯示四個地點；搖桿切換地點，30 分鐘後由 timer 喚醒、更新畫面再回到 deep sleep。這讓它第一次像一個完成的裝置，而不是只會跑範例的開發板。

拍照當下顯示的是板橋的天氣。畫面上有時間、目前溫度、濕度、風速和接下來幾個時段的預報。它不追求即時，更新頻率和電子紙的物理限制相符就夠了。

下一版想做股票行情也是同一個想法：盤中定時顯示幾檔股票的價格與漲跌即可。電子紙沒必要假裝成 Bloomberg 終端機。

## 第二個應用：把 SD 卡變成相框

現行韌體是 SD 相框。電腦端用一支 Python 工具把 JPG 或 PNG 轉成 792×272、1bpp 的 RAW，做 Floyd–Steinberg 抖動；板子從 `/raw_photos/` 掃描檔案後顯示。搖桿可以翻頁，下壓進選單設定輪播間隔，設定存進 NVS，所以重新開機不會消失。

![SD 相框顯示由照片轉成的 1bpp 抖動影像](/assets/images/posts/2026-08-28-esp32-eink-photo-frame.jpg)

效果當然不是彩色相框。電子紙的黑白抖動會吃掉細節，對比低的照片也容易一團灰。不過它有另一種味道：看起來更像報紙照片，而且待機時完全不需要亮著一盞螢幕。

這一段也讓我第一次真的理解「硬體驗證」是什麼。相框不是把一張圖成功顯示就算完成；還要測連續 sleep/wake、輪播、壞檔、讀取期間拔卡、卡鍵，還有 SD 斷電後 GPIO42 是否真的回到 low。

## AI 不是測試儀器

最有代表性的一次是故意往 SD 卡放格式錯誤的 RAW，並在讀取途中拔卡。serial log 會出現一堆 `Card Failed` 和開檔失敗訊息；真正的驗收條件卻是韌體不能 crash、要跳過壞檔、沒有可顯示照片時進入 `NO VALID PHOTOS`，最後還要確實關閉 SD 電源。

![AI 根據實機 serial log 整理 SD 壞檔與拔卡測試結果](/assets/images/posts/2026-08-28-esp32-eink-ai-verification.png)

這張圖就是我認為 agent 最有用的地方。我給它實體測試的 log，它判斷哪些錯誤是預期行為、哪些是資源沒有收乾淨，然後把下一輪要做的驗證條件寫清楚；不是憑空宣稱「測試通過」。

例如 partial refresh 在這塊雙 controller 面板上出現座標對齊問題，原本用來更新選單游標的局刷會裁掉部分內容。最後沒有硬拗局刷，改用完整刷新處理低頻選單操作。完整刷新花約 4.4 秒，對這種偶爾才開一次的設定畫面是合理交換。

AI 可以縮短查資料、列假設、改程式和設計測試的時間，不能替我看見畫面是否真的被裁掉，也不能保證 USB 線、SD 卡或這塊板子的實際行為符合文件。把它當成能快速維持上下文、一起 debug 的工程夥伴，定位才會正確。

## Ox Alpha 在這次專案裡做了什麼

前幾天我用還在免費試用期的 [Ox Alpha](/posts/ox-alpha-royal-clash/) 做了一個 Canvas 遊戲，測的是長 context 能不能讓 agent 一次掌握完整遊戲邏輯。這次的難度不同：程式之外還多了硬體狀態、插拔、刷新波形、電源路徑與不可預期的實機輸出。

它最有價值的地方，是能在多輪對話後仍記得「這塊面板是雙 SSD1683」、「partial window 有對齊問題」、「SD 的 power enable 是 GPIO42」，不必每次遇到新 log 就從頭解釋背景。這讓每一輪實測能真的往前推，而不是陷入重新交代前情提要。

我沒有因此學會 ESP32。這個專案讓我學會的是另一件事：怎麼把實體回饋整理成 AI 能用的訊號，讓它把下一個問題縮小到可以動手驗證。對沒有嵌入式背景的人來說，這比背完一套 Arduino 教學有用得多。

原始碼、硬體研究紀錄和三個階段的開發內容都放在 [BrunoLin-tw/esp32-eink](https://github.com/BrunoLin-tw/esp32-eink)。接下來會先試股票行情看板；如果能把家裡的行情 gateway 接好，這塊螢幕就會從相框再多一個日常用途。

Reference:

- [ELECROW CrowPanel ESP32 5.79-inch E-Paper HMI Display](https://www.elecrow.com/crowpanel-esp32-5-79-e-paper-hmi-display-with-272-792-resolution-black-white-color-driven-by-spi-interface.html)
- [ESP32 E-Paper 專案原始碼](https://github.com/BrunoLin-tw/esp32-eink)
- [GxEPD2](https://github.com/ZinggJM/GxEPD2)
