---
date: 2026-05-14T23:25:34+08:00
title: DaVinci Resolve 安裝指南
slug: davinci-resolve-davincibox-guide
tags:
  - Linux
  - Resolve
description: ""
summary: ""
cover: cover.avif
draft: false
---
Blackmagic Design 旗下的 DaVinci Resolve 是一款功能強大的影片剪輯軟體，最棒的是也提供了 Linux 版本！可惜僅提供特定發行版支援，且無法透過發行版的套件管理器安裝，還沒提供 .rpm 或 .deb 等包裝，需要使用官方腳本進行安裝。即便成功用腳本安裝好了，也會為未來系統維護帶來困難。因此成為我在 Linux 上安裝時唯一感到頭痛的軟體。

幸好，現在有 Distrobox 這樣好用的工具，可以建立特定的發行版容器。這樣一來，就能在官方不提供支援的發行版上安裝，也可以避免安裝時往系統塞一大堆有的沒的套件。此外，有好心人維護了 Davincibox 這樣的腳本，大大簡化了手動用 Distrobox 安裝的步驟。

**但是！凡事都有個但是。** 如果你是中文使用者，當打字的時候，會驚訝地發現 Linux 版的 Resolve 不支援輸入法，所以沒辦法輸入中文。這個問題在官方支援論壇有人提出來已經好幾年了，可能中文用戶太少，BMD 一直沒有處理。還好網路上有一些解法，但都是針對直接安裝才有效，透過容器安裝需要額外步驟。如果你是為了這個問題找到本文，恭喜你！我當初找遍網路都沒找到解法，花了好幾天才搞定呢。

解決完輸入法問題後，如果你是 NVIDIA 顯示卡的使用者，再次恭喜！安裝完的第一次啟動，一切都運作得那麼完美。直到隔天，再次打開 Resolve，會發現怎麼這麼卡頓，甚至沒辦法正常輸出影片，檢查才發現 Resolve 找不到 CUDA 了。不對呀…昨天不是好好的嗎？原來[只有第一次啟動容器才能正確認到 CUDA，第二次就會找不到了](https://github.com/zelikos/davincibox/issues/154)。還好一樣有些解法，當初也是花了好大的功夫才搞定。

雖然看起來很麻煩，但相比於 Windows，在 Linux 透過 Distrobox 安裝 Resolve 還是有好處的。由於 Distrobox 可以為容器指定家目錄，所以能夠避免 Resolve 在文件、影片等使用者目錄裡~~拉屎~~放資料，通通放在自己希望的位置，強迫症狂喜！

OK！廢話結束，進到正題吧。

*以下內容皆以 DaVinci Resolve Studio 20.3.2 版本驗證過。*

---

## 安裝

1. 從 [Blackmagic Design](https://www.blackmagicdesign.com/products/davinciresolve/) 官方網站下載安裝檔，解壓縮取得 `DaVinci_Resolve_Studio_{version}_Linux.run` 檔案。
2. 從 [Davincibox](https://github.com/zelikos/davincibox) 取得安裝腳本，將 `setup.sh` 與 `.run` 檔案放在相同目錄下。
3. 在目錄中開啟終端機，執行以下指令進行安裝：

	```sh
	./setup.sh ./DaVinci_Resolve_Studio_{version}_Linux.run
	```

   > [!note]
    > 別忘了將 `./DaVinci_Resolve_Studio_{version}_Linux.run` 的部分，依實際安裝的檔案名稱修改。

   > [!warning]
    > 放置安裝檔和腳本的目錄，路徑絕對不可包含中日文等全形字元，否則會失敗。

4. 安裝結束前，會提示是否匯出啟動捷徑：
   - 如果不打算指定家目錄：輸入 `y` 並按下 `Enter`。
   - 如果想避免汙染，打算複製一份容器並指定家目錄：輸入 `n` 並按下 `Enter`。

---
*從此處開始，指令皆於容器中執行。*

## 解決 Fctix 5 無法輸入

根據網路上討論，由於 Resolve 在建構時沒有開啟 QT 的輸入法支援，因此沒辦法正常輸入中文。我們可以在 Distrobox 中安裝 `fcitx5-qt5`，並借用套件中的 `.so` 檔案，符號連結過去給 Resolve 使用。

1. 安裝軟體包取得所需組件：

	```sh
	sudo dnf install -y fcitx5-qt5
	```

2. 建立 Qt 庫連結：

	```sh
	sudo mkdir -p /opt/resolve/libs/plugins/platforminputcontexts && \
	sudo ln -sf /usr/lib64/qt5/plugins/platforminputcontexts/libfcitx5platforminputcontextplugin.so \
	/opt/resolve/libs/plugins/platforminputcontexts/
	```

   > 參考自：[DaVinci Resolve 在 Linux 下的输入法支持](https://sh.alynx.one/posts/Input-Method-Support-for-DaVinci-Resolve-on-Linux/)

3. 修改啟動腳本：

	```sh
	sudo nano /usr/bin/run-davinci
	```

4. 於開頭加入以下內容：

	```sh
	export XMODIFIERS="@im=fcitx"
	export GTK_IM_MODULE="fcitx"
	export QT_IM_MODULE="fcitx"
	export QT_PLUGIN_PATH="/opt/resolve/libs/plugins${QT_PLUGIN_PATH:+:$QT_PLUGIN_PATH}"
	```

> [!tip]
> 如果你是用 NVIDIA 顯示卡，並且有遇到僅第一次啟動能讀取到 CUDA 的問題，先別急著儲存並退出，下面的步驟也是修改同一個檔案。

## 修復 CUDA 消失問題

根據網路上討論，Distrobox 有一個錯誤：停止並重新啟動容器後，容器內的 NVIDIA 運行環境符號連結會失效，導致容器中的軟體只在第一次啟動時能正常使用 CUDA。我們可以在啟動腳本加上確認機制，如果連結出問題就重建一遍，來暫時規避這個問題。

1. 修改啟動腳本：

	```sh
	sudo nano /usr/bin/run-davinci
	```

2. 於最開頭加入以下內容：

	```sh
	fix_needed=0
	for lib in libcuda.so libnvcuvid.so; do
	    if [ ! -e "/usr/lib64/$lib" ] || [ ! -s "/usr/lib64/$lib" ]; then
	        sudo ln -sf "/usr/lib64/$lib.1" "/usr/lib64/$lib" && fix_needed=1
	    fi
	done
	
	[ $fix_needed -eq 1 ] && sudo ldconfig
	unset fix_needed lib
	```

> [!tip]
> 上游的 Distrobox 已經合併了[修復的 PR](https://github.com/89luca89/distrobox/pull/2034)，版本更新後應該就不會遇到這個問題了。

---

*容器操作結束，以下指令皆不在容器中執行。*

## （可選）指定家目錄

這是透過 Distrobox 安裝最棒的好處，可以為容器指定家目錄，避免 Resolve 直接在使用者目錄放置資料。當然如果你不在乎，也完全可以跳過這個步驟。雖然也可以靠指令完成，但這裡用 [DistroShelf](https://flathub.org/en/apps/com.ranfdev.DistroShelf) 來操作。

1. 停止前面設定好的 `davincibox`，點擊 Clone Container 進行容器複製。
2. 輸入容器名稱，例如 `davincibox-clone`，並指定家目錄。Resolve 產生的資料都會放在裡面，不過還是可以存取宿主機的資料。
3. 確認一切正常後刪除原本的 `davincibox`。

![](01.avif)

## 建立啟動捷徑

建立一個啟動捷徑，方便開啟 Resolve。

1. 建立 .desktop 檔案：

	```sh
	nano ~/.local/share/applications/com.blackmagicdesign.Resolve.desktop
	```

2. 加入以下內容：

	```sh
	[Desktop Entry]
	Version=1.0
	Type=Application
	Name=DaVinci Resolve
	GenericName=DaVinci Resolve
	Comment=Revolutionary new tools for editing, visual effects, color correction and professional audio post production, all in a single application!
	Exec=distrobox-enter -n davincibox-clone -- /usr/bin/run-davinci /opt/resolve/bin/resolve %u
	Terminal=false
	Icon=davinci-resolve
	StartupNotify=true
	StartupWMClass=resolve
	MimeType=application/x-resolveproj;
	Categories=AudioVideo;Video;Recorder;Graphics;
	Keywords=davinci;blackmagic;video;editor;color;fusion;fairlight;timeline;
	```

   > [!note]
    > 注意啟動命令 `Exec` 中的容器名稱，已修改為 `davincibox-clone`。如果你跳過了指定家目錄的步驟，需修改回 `davincibox`。

到此，安裝過程基本已結束，可以正常使用 Resolve 剪輯影片囉！

下面是一些和安裝無關，但通常會遇到的問題，順便寫在本文。

---

## 新增字體

Resolve 會讀取安裝到系統的字體，在 Window 都是安裝到系統，但在 Linux 通常不會這麼做，而是放到 `~/.local/share/fonts`。Resolve 不會讀取這個目錄裡的字體，因此可能很多人不知道如何新增。

- 切換至 Fusion 介面
- 頂部工具列 Fusion → Fusion 設置…
- Path Map 路徑映射 → Fonts:　SystemFonts:
- 右下方資料夾圖示 → 選擇自訂字體目錄

![](02.avif)

這種方式的好處是字體列表不會列出一大堆系統字體，挑選起來更容易。

## UI 簡轉繁

這似乎是一個 Resolve 的錯誤，會隨機選擇 `fonts` 目錄裡的字體來當 UI 字體。由於我們指定了容器家目錄，可以只在 `fonts` 目錄放一個簡轉繁的字體，來達成 UI 繁化的效果。

> [!note] 
> 版本較新的 Resolve 可能已經修復這個錯誤，所以沒變化的話不要太意外。我目前使用 20.3.2 版本是失效的。

1. 下載簡轉繁字體，比如：
   - 繁媛黑體 Fan Wun Hak：[https://github.com/ayaka14732/FanWunHak](https://github.com/ayaka14732/FanWunHak)
   - 原俠正楷 GuanKiapTsingKhai：[https://github.com/tonyhuan/GuanKiapTsingKhai](https://github.com/tonyhuan/GuanKiapTsingKhai)
   - 思源黑體簡轉繁 NotoSansCJKTC：[https://www.ptt.cc/bbs/Android/M.1496671955.A.543.html](https://www.ptt.cc/bbs/Android/M.1496671955.A.543.html)
1. 放至 `容器家目錄/.local/share/fonts` 目錄中。

## 素材沒聲音或無法播放

Linux 版的 Resolve 對格式要求非常嚴格，最常碰到的問題是：

- 不支援 AAC：許多甚至是大多數影片，音訊格式都是 AAC，但免費版和工作室版皆不支援，必須轉換成 PCM 或 FLAC。
- 不支援 H.264 / H.265：免費版完全不支援 H.264/H.265，不只硬體加速，連播放都不行，需要轉換成 ProRes、DNxHR 等格式，或購買工作室版本。

  > [!note]
  > 印象中即便是工作室版，也不支援 AMD 顯示卡的 H.264/H.265 編解碼？我不太確定，購買前請自行確認官方資訊。