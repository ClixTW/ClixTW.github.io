---
date: 2026-04-28T20:39:50+08:00
title: 在 Linux 上玩遊戲的新手教學
slug: linux-gaming-essentials
tags:
  - Linux
  - Gaming
  - Steam
description: ""
summary: ""
cover: 07.avif
draft: false
---
這篇文章包含了我在 Linux 上**所有**遊戲相關的設定。Linux 的使用體驗很好，我希望更多人願意嘗試，使用者多了才會受到廠商重視、投入資源，整個生態又會更好。

在 Windows 玩遊戲需要做的一些調整，在 Linux 上做法可能截然不同。每件事都不難，對長期使用者來說或許是常識，但對 Linux 完全陌生的話，一項一項弄清楚需要花費不少心力，許多事情甚至太基礎而少被提及。因此寫了這篇，希望能降低轉換的摩擦。

*本文特別適用但不限於：桌面環境是 GNOME 的 NVIDIA 顯示卡使用者。*

---

在開始之前，為了避免誤會，先簡單說明。

- 如果像下面這樣，指令區塊上方寫了 `SH`，代表需要在終端機中執行：

	```sh
	
	```

- 如果像下面這樣，指令區塊上方寫了 `PLAINTEXT`，代表需要編輯文字：

	```
	
	```

- 為了避免太過冗長，部分指令或圖形界面軟體操作，實際上需要額外確認或選擇，這類步驟會省略不列出。

另外，頁面右側有目錄，滑鼠移過去或觸控螢幕輕點即可顯示。

---
## Steam

推薦透過 Flatpak 安裝 Steam，保持系統套件乾淨，並且可以用擴充功能的方式安裝 GE-Proton，方便更新而不用另外安裝 ProtonPlus 來管理。

1. 安裝 Steam：[https://flathub.org/en/apps/com.valvesoftware.Steam/install](https://flathub.org/en/apps/com.valvesoftware.Steam/install)

   > [!tip]
   > 如果點擊連結後，沒有自動跳轉到安裝畫面，可以安裝像 [Bazaar](https://flathub.org/en/apps/io.github.kolunmi.Bazaar) 這樣的應用程式商店，能夠非常方便地從 Flathub 安裝和管理應用程式。

   ![](07.avif)

2. （可選）安裝 GE-Proton：

   ```sh
   flatpak install com.valvesoftware.Steam.CompatibilityTool.Proton-GE
   ```

   > [!tip]
   > 目前不支援從網頁跳轉安裝應用程式的擴充功能，可以將上面這段指令貼到終端機進行安裝；也可以透過 Bazaar，在 Steam 的擴充功能（Add-ons）頁面中點擊安裝，兩種操作的結果完全一致。

3. 在 Steam 設定中預設為遊戲啟用 Proton，建議選擇 Proton Experimental：

   ![](08.avif)

   > [!tip]
   > 如果遊戲有提供原生的 Linux 版本，即使這裡預設啟用了 Proton，還是會安裝 Linux 版本。需要為這類遊戲個別設定「強制使用特定 Steam Play 相容性工具」，然後選擇 Proton，才會下載 Windows 版本。許多遊戲的 Linux 版本缺乏維護，透過 Proton 運作反而表現更好。
   > 
   > 少數遊戲 Linux 版本和 Windows 版本存檔不互通，如果發現明明有支援雲端存檔，進了遊戲卻找不到以前的存檔，也可以透過強制使用 Proton 來解決。

4. （可選）關閉著色器預存快取：

   ![](09.avif)

   > [!note]
   > 啟用這個選項會導致 Steam 頻繁為遊戲下載著色器預存快取，大型 3A 遊戲甚至可能高達 1GB 以上，如果真的完全不缺網路頻寬，開著也無妨，但硬體不算太差的話，在遊玩的過程中自動編譯就足夠了。
   > 
   > 少數遊戲由於缺乏專有媒體解碼器，若透過 Proton Experimental 執行，必須啟用著色器預存快取，從 Valve 下載預先轉換好的過場影片才能正常播放。但與其為此頻繁下載快取，我選擇在遇到這種情況時切換到 GE-Proton。

### 開機自啟動

由於 Steam 啟動速度頗慢，可以設定開機自動靜默啟動，想玩遊戲的時候就不用等待漫長的啟動時間了。

在 Linux 下，會在開機時自動啟動 `~/.config/autostart` （`~` 代表家目錄）中的應用程式，所以要在這裡建立 Steam 的啟動捷徑。

Ignition 是一個方便的工具，能夠透過圖形界面輕鬆管理啟動捷徑。這裡我們使用它來完成上述操作。

1. 安裝 Ignition：[https://flathub.org/en/apps/io.github.flattool.Ignition/install](https://flathub.org/en/apps/io.github.flattool.Ignition/install)
2. 點擊「+ NEW」並找到 Steam。
3. 在「Command or Script」中尾端加上 `-silent`，表示靜默啟動：

   ```
   /usr/bin/flatpak run --branch=stable --arch=x86_64 --command=/app/bin/steam --file-forwarding com.valvesoftware.Steam @@u %U @@ -silent
   ```

4. 點擊右上角「✓Save」儲存。
   ![](10.avif)

   > [!tip]
   > 如果想在開機後自動進入 Steam Big Picture 模式，也可以改成加上 `steam://open/bigpicture`。

### 哪個 Proton 版本好？

先簡單介紹兩個主要版本：

- Proton Experimental：由 Valve 官方維護的測試版 Proton，包含最新功能和尚未進入穩定版的修復。
- GE-Proton：由社群基於 Proton Experimental 製作，包含了官方無法加入的專有媒體解碼器，和一些針對特定遊戲的修復。

如果不啟用著色器預存快取，由於可能遇到影片播放的問題，照理來說預設使用 GE-Proton 會比 Proton Experimental 更好。

但實際上我沒碰過影片問題（當然是玩得少了，並非說這個問題不存在），反而發生了幾次透過 GE-Proton 運作有問題，換成 Proton Experimental 卻一切正常的情況。因此後來便改成預設使用 Proton Experimental，遇到問題才嘗試 GE-Proton。

假如換了幾個 Proton 版本，遊戲運作還是有狀況，我會嘗試透過接下來要介紹的 Gamescope 來執行，通常都能解決問題。

---
## Gamescope

Gamescope 是 Valve 專為遊戲開發的輕量視窗管理器，可以最大程度地排除桌面環境的影響，因此可以有效解決少數遊戲出現視窗比例、大小異常的問題。非必要，但遇到問題時很好用。

本身也是滿方便的工具，可以強制遊戲全螢幕顯示、以低解析度繪製再透過 FSR / NIS 升頻到高解析度等等。

1. 安裝 Gamescope：

   ```sh
   flatpak install org.freedesktop.Platform.VulkanLayer.gamescope
   ```

2. （可選）新增環境變數讓 Steam 找得到 Gamescope：

   ```sh
   flatpak override --user --env=PATH=/app/bin:/usr/bin:/usr/lib/extensions/vulkan/gamescope/bin com.valvesoftware.Steam
   ```

   > [!note]
   > 如果不執行此步驟，Steam 內建提供的 Proton 將無法和 Gamescope 搭配使用，必須使用 GE-Proton 才能正常運作。

3. 要透過 Gamescope 遊玩時，在遊戲的 Steam 啟動選項填入：

   ```
   gamescope -W 2560 -H 1440 -f --adaptive-sync --mangoapp -- %command%
   ```

   > [!tip]
   > 上列參數包含了顯示解析度、全螢幕、自適應刷新率、啟用 MangoHud，請依個人需求修改。
   > 
   > 所有選項可參考：[https://github.com/ValveSoftware/gamescope#options](https://github.com/ValveSoftware/gamescope#options)

   ![](02.avif)

---

## 增加著色器快取大小

著色器快取會在遊戲進行中自動編譯，但預設大小有上限，超過後會被丟棄。這可能會導致大型遊戲在每次啟動遊戲後都需要重新編譯， 並造成卡頓，因此需要提高大小上限。

1. 建立環境變數設定：

   ```sh
   mkdir -p ~/.config/environment.d && nano ~/.config/environment.d/99-custom.conf
   ```

2. 根據使用的顯示卡，新增以下內容，將限制提高到 12 GB：

   - NVIDIA 顯示卡

   ```
   __GL_SHADER_DISK_CACHE_SIZE=12000000000
   ```

   - AMD 顯示卡

   ```
   MESA_SHADER_CACHE_MAX_SIZE=12G
   ```

   > [!tip]
   > 使用 nano 編輯器時，按下 `Ctrl+X`，輸入 `y` 再按下 `Enter` 可儲存並退出。
   > 
   > 如果覺得太麻煩，也可以簡單地使用一般的文字編輯器來完成。

3. 重新啟動系統。

---

## 啟用 NTSync

NTSync 是一種在核心層級模擬 Windows NT 同步機制的新技術，可以改善高 CPU 負載遊戲的效能及提高相容性。Linux 6.14 版本以上的內核都支援 NTSync，但發行版不一定有啟用該內核模組，因此可能需要手動啟用。

- 檢查是否已啟用核心模組：

```sh
ls -l /dev/ntsync
```

若有輸出內容，沒顯示錯誤，則表示已成功啟用，不需要進行以下步驟。

---

1. 建立 ntsync.conf：

   ```sh
   sudo nano /etc/modules-load.d/ntsync.conf
   ```

2. 新增以下內容：
   
   ```
   ntsync
   ```

3. 重新啟動系統。

> [!note]
> NTSync 需要和 Proton 11 以上，或 Proton Experimental、GE-Proton 搭配使用。

--- 

- 若要刪除：

```sh
sudo rm /etc/modules-load.d/ntsync.conf
```

---

## 低延遲無畫面撕裂

為了實現遊戲畫面無撕裂並保持最低延遲，在 Windows 上需要透過顯示卡廠商提供的設定面板做以下設定：

- 開啟全局垂直同步，並關閉遊戲內垂直同步。
- 限制最高幀率低於螢幕更新率。
- 開啟可變動更新率（FreeSync / G-Sync）。

> [!tip]
> 更多資訊可參考這篇經典指南：[https://blurbusters.com/gsync/gsync101-input-lag-tests-and-settings/14/](https://blurbusters.com/gsync/gsync101-input-lag-tests-and-settings/14/)

在 Linux 上，實現這幾點的操作和 Windows 不一樣，以下從垂直同步開始介紹。

---

### 垂直同步

由於 Wayland 協議的設計本來就不允許畫面撕裂，除非特地開啟「允許撕裂」。主流桌面環境 GNOME、KDE Plasma 已經預設使用 Wayland，因此不需要做額外設定。

- 確認目前的環境：

```sh
echo $XDG_SESSION_TYPE
```

如果輸出結果是 `wayland`，表示：

- 系統已具備全局垂直同步，不需要手動開啟。
- 確保關閉遊戲中的垂直同步。

### 限制最高幀率

在 Linux 上限制最高幀率的推薦做法是透過 MangoHud，延遲極低且不需要透過啟動選項為每個遊戲設定，效果好又方便。

MangoHud 除了可以限制幀率，本身主要是功能豐富的效能監視器。設定檔只是文字檔，可以直接修改，也可以搭配 Mango Juice 輕鬆透過圖形界面進行各項設定。

#### MangoHud

1. 安裝 MangoHud：

   ```sh
   flatpak install org.freedesktop.Platform.VulkanLayer.MangoHud
   ```

2. 為所有 Steam 遊戲預設啟用 MangoHud：

   ```sh
   flatpak override --user --env=MANGOHUD=1 com.valvesoftware.Steam
   ```

#### 進行設定

1. 安裝 Mango Juice：[https://flathub.org/en/apps/io.github.radiolamp.mangojuice/install](https://flathub.org/en/apps/io.github.radiolamp.mangojuice/install)
2. 「Performance」→「Limiters FPS」→「late - 140 - 100 - 60 」

   ![](01.avif)

   > [!tip]
   > 實際數值請依個人設備修改，必須低於螢幕最高更新率。在遊戲中可透過快捷鍵在多個數值間切換。

3. 設定好後按下 `Ctrl+S` 或右上角選單中點擊儲存。

除此之外，也建議做以下設定：

- 「Extras」→「Hide HUD」：開啟此選項，預設隱藏效能監視器。
- 「Visual」→「Hide HUD」：設定啟用效能監視器的快捷鍵。

### 可變動更新率

GNOME 50 正式支援 VRR，可以直接在顯示器設定中啟用。低於 50 的版本需要先啟用實驗性支援，可以透過指令或 Refine 啟用。

- 透過指令啟用 VRR 實驗性支援：

```sh
gsettings set org.gnome.mutter experimental-features "['variable-refresh-rate']"
```

- 若同時需要分數縮放實驗性支援：

```sh
gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer', 'variable-refresh-rate']"
```

也可以透過圖形介面輕鬆啟用：

1. 安裝 Refine：[https://flathub.org/en/apps/page.tesk.Refine/install](https://flathub.org/en/apps/page.tesk.Refine/install)
2. 「Shell & Compositor」→「Variable Refresh Rate Options」，把選項開啟。

   ![](06.avif)

---

然後，在顯示器設定中便可以找到「可變動更新率」的選項，把它啟用。

![](05.avif)

---

## 降壓超頻

LACT 是一款很方便的顯示卡調整工具，用於監測顯示卡各項數據、超頻、控制風扇等等。這裡用它來達成降壓超頻的效果。

- 安裝 LACT：[https://flathub.org/en/apps/io.github.ilya_zlobintsev.LACT/install](https://flathub.org/en/apps/io.github.ilya_zlobintsev.LACT/install)

有兩種方式可以實現降壓超頻，分別是透過「鎖定最高核心頻率及設定頻率偏移」間接達成，或是透過「編輯電壓頻率曲線」直接控制。

### 間接做法 - 鎖定最高頻率及設定偏移

1. 勾選「啟用 GPU 時鐘鎖定」。
2. 填入「最大 GPU 時鐘」：以我的顯示卡為例，填入 1830。
3. 填入「GPU P-State 0 時鐘偏移」：以我的顯示卡為例，填入 180。
4. 點擊左下角「應用」套用設定。

![](03.avif)

> [!tip]
> 個人偏好這種做法，設定起來很簡單，效果也不錯。

### 直接做法 - 編輯電壓頻率曲線

1. 在「OC」頁面下方找到「VF Curve Editor」按鈕，點擊後進行設定。

   ![](04.avif)

2. 結束後點擊主界面左下角「應用」套用設定。

> [!warning]
> 如曲線編輯器介面上方的警告所說，NVIDIA 尚未公開相關操作的 API，因此隨驅動更新，這種做法可能會失效或出現預期外的行為。除非真的需要非常精準的調整，否則建議用上面的間接做法。

---

## 安裝 EXE 格式的模組

許多遊戲的社群中文化或模組是提供 EXE 格式，例如 nick.exe、天邈漢化組都會用這種方式封裝，這時候可以藉助 Protontricks 來安裝。

1. 安裝 Protontricks：[https://flathub.org/en/apps/com.github.Matoking.protontricks/install](https://flathub.org/en/apps/com.github.Matoking.protontricks/install)
2. 至少啟動過一次要安裝模組的遊戲，讓 Proton 建立環境。
3. 在檔案管理器中對 EXE 格式的模組按下右鍵，選擇透過「Protontricks」啟動。
4. 在 Protontricks 的介面中選擇目標遊戲，完成安裝。

---

## 更新驅動程式

在 Linux 上，推薦做法是透過套件管理器安裝一切東西，所以不需要自行上官網下載驅動程式並更新。

- AMD、Intel 顯示卡：驅動已包含在 Linux 核心，隨系統更新會自動更新。
- NVIDIA 顯示卡：驅動透過套件管理器安裝，隨套件更新會自動更新。

優點是不必像在 Windows 上，讓 NVIDIA App 在後臺運作檢查更新，或自己手動下載驅動再安裝。缺點是比較被動，需要等發行版推送更新。因此若硬體較新，不建議使用套件庫版本落後太多的發行版。

---

## 幀率異常下降

如果使用的是 Fedroa Silverblue / Kinoite，或基於這類原子鏡像的發行版，比如 Universal Blue 的 Bluefin 或 Aurora，並且使用 NVIDIA 顯示卡，有可能在系統更新後，發生效能異常下降的情況。

由於 Flatpak 採用沙盒機制，應用程式無法直接使用系統的 NVIDIA 驅動程式，必須依賴 `org.freedesktop.Platform.GL.nvidia` 這個執行環境擴充功能，而且版本必須和系統驅動的版本嚴格保持一致才能正常運作。

當原子鏡像的更新中包含了顯示卡驅動更新，重新啟動系統套用更新後，可能發生系統 NVIDIA 驅動和 Flatpak 執行環境版本不一致的情況，此時所有透過 Flatpak 安裝的應用程式，都會無法正常使用顯示卡。

發生這個問題時，只需要讓 Flatpak 執行更新，並重新啟動系統即可解決。

```sh
flatpak update
```

> [!tip]
> 這是上游已知問題，或許未來有機會解決。
> 
> 由於 AMD 顯示卡的驅動包含在 Linux 核心中，所以不會遇到這個問題。

---

大概是這樣，如果想知道更多，各項目本身的文檔是很棒的資源，常常比網路上的隨機分享（包含這篇）更加正確和全面。

也可以看看 CachyOS 的遊戲相關指南：[https://wiki.cachyos.org/configuration/gaming/](https://wiki.cachyos.org/configuration/gaming/)

有哪裡說錯或不清楚，歡迎留言指正補充或發問，感謝！
