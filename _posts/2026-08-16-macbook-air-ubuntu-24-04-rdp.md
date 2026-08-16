---
title: "MacBook Air 連線 Ubuntu 24.04：內建 RDP 設定與黑畫面修正"
date: 2026-08-16 23:48:53 +0800
categories:
  - tech
tags:
  - macos
  - ubuntu
  - rdp
  - windows-app
  - remote-desktop
---

最近換了 13 吋 MacBook Air M5，也順便重新整理幾台常用電腦的連線方式。

我有一台 Ubuntu 24.04 Server，平常主要透過 SSH 管理，偶爾還是需要進入 GNOME 桌面操作。Ubuntu 24.04 已經內建以 RDP 為基礎的 GNOME Remote Desktop，不必另外安裝 xrdp；Mac 端則可以直接使用微軟的 Windows App。

實際設定並不複雜，麻煩的是 Ubuntu 提供兩種遠端模式，加上 Windows App 連線時可能出現黑畫面。以下記錄這次確認可用的設定方式。

## 先分清楚兩種遠端模式

進入 Ubuntu 的：

```text
Settings → System → Remote Desktop
```

可以看到兩種用途不同的功能：

### Remote Login

適合 Ubuntu 放在遠端、平常沒有人使用本機桌面的情況。

它會在登入畫面接受 RDP 連線，預設使用 TCP 3389。連線前，原本登入 Ubuntu 桌面的使用者應先完整登出。

我這次需要的是這個模式。

### Desktop Sharing

用來分享目前已經登入的 GNOME 桌面，適合遠端協助或繼續操作本機正在使用的工作階段。

啟用時還要打開 Remote Control，遠端使用者才能操作滑鼠與鍵盤。若 Remote Login 與 Desktop Sharing 同時開啟，Remote Login 會使用 3389，Desktop Sharing 則可能改用 3390 等其他連接埠。

簡單區分如下：

| 使用情境 | 建議模式 |
|---|---|
| Ubuntu 停在登入畫面，需要從遠端登入 | Remote Login |
| Ubuntu 已登入，想操作同一個桌面 | Desktop Sharing |
| 沒有實體螢幕、偶爾需要 GUI | Remote Login |

## Ubuntu 24.04 啟用 Remote Login

開啟：

```text
Settings → System → Remote Desktop → Remote Login
```

接著進行以下設定：

1. 點擊 Unlock，輸入管理者密碼解除鎖定。
2. 開啟 Remote Login。
3. 記下畫面顯示的 Hostname、Port、Username 與 Password。
4. 預設連接埠通常是 3389。

這裡顯示的是 RDP 連線使用的驗證資訊，不要直接假設它與 Ubuntu 本機登入密碼相同。實際連線時應以設定畫面顯示的內容為準。

也可以從 Terminal 查詢 Ubuntu 的 IP：

```bash
hostname -I
```

假設查到的位址是：

```text
192.168.1.100
```

Mac 端之後就使用這個位址連線。

如果 Ubuntu 有啟用 UFW，還需要允許區域網路連入 RDP。以 `192.168.1.0/24` 為例：

```bash
sudo ufw allow from 192.168.1.0/24 to any port 3389 proto tcp
sudo ufw status
```

我只允許區域網路來源，不會把 3389 直接開放到 Internet。需要從外部網路連線時，應先透過 Tailscale、WireGuard 或其他 VPN 回到內部網路。

## Mac 安裝 Windows App

Microsoft Remote Desktop 已經更名為 Windows App。Mac App Store 裡看到的藍色 Windows App，就是目前微軟提供的 RDP Client。

安裝後開啟 Windows App：

1. 點擊左上角的 `+`。
2. 選擇 Add PC。
3. 在 PC name 輸入 Ubuntu 的 IP 位址。
4. User account 可以先選 Ask when required。
5. 點擊 Add 儲存。

如果 Ubuntu 顯示的 RDP Port 採用其他連接埠，需要連同 Port 一起輸入：

```text
192.168.1.100:3390
```

第一次連線可能會看到憑證提示。可以將 Windows App 顯示的 fingerprint 與 Ubuntu Remote Desktop 設定裡的 Verify Encryption 資訊比對，確認目標電腦正確後再繼續。

## 黑畫面或只看到白色游標

使用 Mac 版 Windows App 連線 Ubuntu 24.04 時，可能通過驗證後只看到黑畫面與白色游標，接著斷線或顯示錯誤。

這是新版 RDP Client 與 GNOME Remote Desktop 之間的相容性問題。社群目前常用的處理方式，是修改匯出的 `.rdp` 設定檔。

先在 Windows App 主畫面對 Ubuntu 連線按右鍵，選擇 Export，將設定匯出成 `.rdp` 檔案。

使用 VS Code 或其他純文字編輯器開啟檔案。如果使用 macOS 內建的 TextEdit，先執行：

```text
Format → Make Plain Text
```

找到以下設定：

```text
use redirection server name:i:0
```

將最後的 `0` 改為 `1`：

```text
use redirection server name:i:1
```

儲存後回到 Windows App，從選單執行：

```text
File → Import
```

匯入修改過的 `.rdp` 檔，之後改用這個連線項目。

這項修改屬於目前的相容性 workaround，並非 Ubuntu 或 Microsoft 提供的正式修正。未來更新 GNOME Remote Desktop 或 Windows App 後，如果原始設定已能正常使用，就不必繼續保留。

## 顯示帳號已登入或連線後立刻中斷

使用 Remote Login 時，Ubuntu 同一個使用者帳號可能無法同時維持本機與遠端圖形工作階段。

連線前應在 Ubuntu 本機執行完整登出：

```text
System Menu → Power Off / Log Out → Log Out
```

要注意，鎖定螢幕與登出不同。只按 Lock 仍然保留原本的 GNOME Session，遠端連線可能繼續黑畫面或被拒絕。

如果目的是操作目前已經登入的桌面，應改用 Desktop Sharing，避免選到 Remote Login。

## 一直顯示帳號或密碼錯誤

先回到 Ubuntu：

```text
Settings → System → Remote Desktop
```

重新確認目前使用的是 Remote Login 或 Desktop Sharing，並照著該頁面 Login Details 顯示的 Username 與 Password 輸入。

兩種模式的認證資料與 Port 可能不同。若同時啟用，更要確認 Windows App 連到的是 3389 或另一個 Desktop Sharing Port。

## 最後整理

我的設定方式是：

```text
Ubuntu 24.04
  → 啟用 Remote Login
  → 使用 TCP 3389
  → 本機使用者先登出

MacBook Air
  → 安裝 Windows App
  → 使用 Ubuntu IP 建立連線
  → 黑畫面時修改 .rdp 的 redirection 設定
```

Ubuntu 24.04 的內建 RDP 已足以應付偶爾進入桌面的需求，也少了一套 xrdp 服務需要維護。真正容易踩雷的地方，是把 Remote Login 與 Desktop Sharing 當成同一件事，以及 Windows App 目前仍有一項需要手動處理的相容性問題。

對這類平常以 SSH 為主、偶爾才需要 GUI 的 Ubuntu 主機，我會保留內建 Remote Login，再搭配 VPN 限制連線範圍。設定簡單，後續維護也比較乾淨。

Reference:

- [Ubuntu Documentation：Remote Login](https://help.ubuntu.com/stable/ubuntu-help/remote-login.html.en)
- [Ubuntu Documentation：Desktop Sharing](https://help.ubuntu.com/stable/ubuntu-help/sharing-desktop.html.en)
- [Microsoft：Remote Desktop client for macOS](https://learn.microsoft.com/en-us/previous-versions/remote-desktop-client/remote-desktop-macos)
- [RDP error code 0x207 on Mac for Ubuntu 24.04](https://dev.to/emile1636/rdp-error-code-0x207-on-mac-for-ubuntu-24-d6d)
- [GNOME Discourse：Ubuntu 24.04 RDP black screen](https://discourse.gnome.org/t/gnome-rdp-black-screen-ubuntu-24-04-1/23502)
