---
date: 2026-04-21T20:41:41+08:00
title: 快速方便搜尋選取文字的方法
slug: quick-text-search
tags:
  - Linux
  - FOSS
  - Kando
description:
summary: ""
cover: 03.avif
draft: false
---
使用電腦時，無論是做個人研究，還是翻譯工作，我都經常需要搜尋看到的文字。

一般來說，必須進行一系列操作：「選取想搜尋的文字」→「按下複製」→「切換到瀏覽器」→「打開網址列」→「貼上想搜尋的文字」→「按下 Enter 送出」。

我實在太懶惰了，沒辦法忍受每次搜尋都做一遍上面的動作。過去用 Windows 時，我會用手勢軟體 [WGestures](https://www.yingdev.com/projects/wgestures2) 代勞，選取文字，按著右鍵鬼畫符就完成搜尋，但 Linux 沒有一個比較完善的手勢軟體[^1]，因此得尋找其他方式實現。

[^1]: 話雖如此，我現在認為 [Kando](https://github.com/kando-menu/kando) 搭配 [Input Remapper](https://github.com/sezanzeb/input-remapper) 的效果更好，設定上也更直觀。

後來想到的一個辦法是寫一個腳本搭配快捷鍵或 Kando，效果非常好。

## 搜尋選取文字的腳本

1. 建立一個文字檔案在 `~/.local/bin` ，我把它叫做 `search-selection`。
2. 文字檔案裡包含以下內容：

```sh
#!/bin/bash

# 自訂搜尋引擎
SEARCH_ENGINE="https://www.google.com/search?q="
# SEARCH_ENGINE="https://duckduckgo.com/?q="

QUERY=$(wl-paste --primary --no-newline 2>/dev/null)

if [ -n "$QUERY" ]; then
    wl-copy --clear --primary
else
    QUERY=$(wl-paste --no-newline 2>/dev/null)
fi

if [ -z "$QUERY" ]; then
    exit 1
fi

ENCODED_QUERY=$(python3 -c "import urllib.parse, sys; print(urllib.parse.quote(sys.stdin.read()))" <<< "$QUERY")

xdg-open "${SEARCH_ENGINE}${ENCODED_QUERY}"
```

> [!tip]
> 可以修改最上面的 `SEARCH_ENGINE` 這個變數自訂搜尋引擎。

3. 設定腳本可作為程式執行，並確認 `~/.local/bin` 這個路徑已經加到環境變數，不過大多數發行版應該已經幫你做好了。

## 綁定到快捷鍵

接下來可以透過桌面環境綁定到快捷鍵，方便使用這個腳本。

- 以 GNOME 為例，能夠在「設定」→「鍵盤」→「檢視與自訂快捷鍵」→「自訂快捷鍵」裡設定：

![](01.avif)

- 以 niri 為例，則可以寫在 `binds` 裡：

```sh
binds {
...
Super+C hotkey-overlay-title="Search" { spawn-sh "search-selection"; }    
...
}
```

我設定成 `Super+C`，選取了文字以後按下這個快捷鍵，即可自動在預設的瀏覽器裡搜尋選取的文字了。

搜尋後會清空選取文字的緩衝區，當緩衝區裡沒內容的時候，會改為搜尋剪貼簿裡的文字。這在一些情況下很好用，比如整理 calibre 書庫時，對著書名欄位按下複製就可以接著搜尋，不用進入編輯再選取。

> [!note]
> 換句話說，搜尋剪貼簿裡的內容前，需要先清空選取的緩衝區。實務上就是除了剛開機從未選取過內容，否則要先進行一次搜尋來清空。選擇這麼做而不是更簡潔的做法，是為了避免腳本需要額外的依賴項 `wtype`。

## 搭配 Kando 使用

這其實是我的主要使用方式，因為我太懶了，都用滑鼠選取文字了，還要伸手按快捷鍵？都用滑鼠解決是最好的。

Kando 是一款非常優秀的開源餅狀選單工具，可以極大地增進操作電腦的效率。我是個重度的手勢軟體使用者，發現 Kando 和 Input Remapper 搭配使用，可以完全替代手勢軟體後，我才終於無後顧之憂地投入了 Linux 的世界。

- Kando 的 GitHub 儲存庫：[https://github.com/kando-menu/kando](https://github.com/kando-menu/kando)

雖然 Kando 是跨平臺軟體，但開發者 Simon 是 GNOME 使用者，GNOME 上知名的 Burn My Windows 和 Desktop Cube 擴充功能也是由他開發的，所以完全不用擔心支援度問題。

你可以建立一個「執行命令」的項目，然後在選取文字後，開啟這個項目執行搜尋：

![](02.avif)

建議搭配 Input Remapper 來完全透過滑鼠開啟選單，項目文檔裡有關於這部分的教學，是我寫的，本來也想在部落格介紹，但一直拖著沒動筆哈哈。

- 教學頁面：[https://kando.menu/input-remapper-tutorial/](https://kando.menu/input-remapper-tutorial/)

使用起來像這樣，按下右鍵並按下左鍵開啟選單，並拖拽即可進行搜尋：

![](03.avif)

怎麼樣？看起來很方便吧！
