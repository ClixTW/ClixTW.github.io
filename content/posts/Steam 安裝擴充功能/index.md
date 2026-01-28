---
date: 2026-01-28T03:01:28+08:00
title: 既然 Steam 是瀏覽器套殼，可以裝擴充功能嗎？
slug: steam-browser-extensions
tags:
  - Steam
  - Gaming
description: ""
summary:
cover: 03.avif
draft: false
---
還真的可以，而且不用靠第三方工具。

成功裝完我的想法是：啊 ??? 這麼簡單？

其實是前幾天在 X 上看到有人提的，翻了好久實在找不到原推，容我不附連結了🙏

---

## 步驟

在商店隨意找個連結，用滑鼠中鍵開啟，得到一個獨立的瀏覽器視窗：

![](01.avif)

接著在地址欄輸入以下網址，進入擴充功能的管理頁面：

```
chrome://extensions/
```

![](02.avif)

甚至能從 Chrome 線上應用程式商店直接安裝擴充功能：

![](03.avif "安裝時可能會要求選擇 .crx 檔案的暫存位置，隨意選個目錄即可，安裝完成後會自動清除。")

回到 Steam 商店，確認擴充功能都正常運作：

![](04.avif)

是的，就這樣。

重新啟動 Steam 以後也還在，但不確定用戶端更新後會不會消失，假如遇到問題再來更新。

> [!IMPORTANT]
> - 慎裝來路不明的擴充功能，避免 Steam 帳號遭竊。
> - Augmented Steam 裝了無法正常運作，可能是內建的 Chromium 版本太舊？
> - 事後想調整 SteamDB 設定，可以隨意進入一個 SteamDB 頁面，點擊 `左上角 Menu > Browser extension > 頁面上方 Options`。我這裡直接從管理頁面跳轉到選項會當掉。
> - 可以把 `chrome://extensions/` 加入書籤，點擊加號新增分頁的時候，就能快速開啟了。
> - 其他諸如 `chrome://settings/`、`chrome://flags/` 也是通的。
> - [Yellow Taxi Goes Vroom（Steam）](https://store.steampowered.com/app/2011780/Yellow_Taxi_Goes_Vroom/)好玩到爆炸，推推。

## 結語

一直都知道 Steam 包了 Chromium 進去，但認為不可能沒擋這麼低級的做法，所以從來沒試過，沒想到就真的沒擋。算是上了一課，遇到事情別想太多，說不定敵人只是自己憑空想像出來的。

不過也好，因為 Millennium 不支援透過 Flatpak 裝的 Steam，才在想該怎麼辦呢，想念能直接在 Steam 用戶端用 SteamDB 看史低價格，現在又和以前一樣方便了。