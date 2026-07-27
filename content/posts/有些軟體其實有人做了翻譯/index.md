---
date: 2026-07-20T17:47:53+08:00
title: 有些軟體其實有人做了翻譯
slug: linux-locale-fallback-fix
tags:
  - Linux
cover: ""
summary: ""
description: ""
draft: false
---
Linux 上的許多軟體可能明明有中文，卻永遠只顯示英文介面。

隨著 [Kando](https://github.com/kando-menu/kando) 和 [BasicSync](https://github.com/chenxiaolong/BasicSync) 的繁中在地化工作告一段落，我開始看還有哪些愛用軟體需要翻譯，卻發現一件奇怪的事——有些在我系統上顯示英文的軟體，其實已經有人貢獻了中文翻譯。

以功能強大的 Flatpak 軟體包管理軟體 [Warehouse](https://flathub.org/en/apps/io.github.flattool.Warehouse) 為例，翻譯託管在 Weblate 的一個伺服器上，確實已經有人貢獻了[漢語（正體中文）](https://weblate.fyralabs.com/projects/flattool/warehouse/zh_Hant/)，也早就合併進了[原始碼裡](https://github.com/flattool/warehouse/blob/main/po/zh_Hant.po)，但在我的系統卻顯示英文介面。我是透過開發者推薦的方式，從 Flathub 安裝的，也確定是最新版。那麼問題出在哪裡呢？

![](01.avif "明明有中文，卻顯示了英文介面")

經過小小的調查，發現是語系的回退機制出了問題。在終端機執行 `locale` ，我的系統輸出了 `LANG=zh_TW.UTF-8`。Warehouse 的翻譯檔則是 `zh_Hant`，軟體不知道在 `zh_TW` 的系統上，應該回退使用 `zh_Hant`，於是直接用了 `en`。

另一個例子是 Kando。前幾天測試 3.0 版本，才發現安裝完的全新設定檔，語言選項在「使用系統語言」時，顯示的會是簡體中文，而非早已包含的正體中文。原因是軟體不知道 `zh_TW` 應該使用 `zh_Hant`，而回退用了 `zh`；`zh` 的規則是 `zh: ['zh-Hans', 'en']`，於是顯示了 `zh-Hans`。我已經提交了 PR，這個問題在 3.0 版本正式上線前應該能解決。

老實說中文的情況太複雜了，有簡中、繁中，繁中還不只一種，台港澳之間各有不同的用語習慣，讓國外開發者搞懂實在強人所難。不如自己動手吧！只要加一行環境變數而已。

- 如果是使用 systemd 的發行版，在 `~/.config/environment.d` 建立一個 `locale.conf` 文件（檔名隨意，以 `.conf` 結尾就好），裡面寫上：

	```ini
	LANGUAGE=zh_TW:zh_Hant:en
	```

- 想更激進一點，寧願看到簡體字，也不想看到英文的話：

	```ini
	LANGUAGE=zh_TW:zh_Hant:zh_CN:zh_Hans:en
	```

如此一來，即便是出問題的軟體，也會由左至右依序回退語言。登出再登入，就會看到一些軟體正常顯示中文介面囉。

![](02.avif "如果中文和英文之間能加個空格又會更好🤓")