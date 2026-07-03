---
date: 2026-07-03T01:22:10+08:00
title: 在 Flatpak 版 GNOME Boxes 實現 USB 直通功能
slug: gnome-boxes-usb-passthrough
tags:
  - FOSS
  - GNOME
  - Linux
cover: 02.avif
summary:
description:
draft: false
---
最近想調整 Razer 滑鼠的 DPI，但手邊已經沒有任何 Windows 裝置，所以需要透過虛擬機 USB 直通來達成。目前 Flatpak 版的 GNOME Boxes 尚未支援 USB 直通功能，但我用的是原子鏡像，不想只為了這點小事就添加分層，或切換到具備完整虛擬機功能的開發者鏡像。

幸好在 GNOME Boxes 的儲存庫找到了一個[暫時解法](https://gitlab.gnome.org/GNOME/gnome-boxes/-/work_items/236#note_1858728)，實測後的確有效，成功調整了滑鼠設定，也順便更新了幾個裝置的韌體。在這裡記錄一下做法，最後也會附上一個簡單的互動式腳本。

---

## 手動操作

1. 開啟終端機，查詢目標 USB 裝置的 Bus 號碼及 Device 號碼：

	```sh
	lsusb
	```

   > [!NOTE]
    > - 輸出範例：`Bus 003 Device 005: ID 1532:00b6 Razer USA, Ltd Razer DeathAdder V3 Pro`
    > - 記下 Bus 號碼 `003` 與 Device 號碼 `005`。

2. 修改裝置檔案存取權限：

	```sh
	sudo chown user:user /dev/bus/usb/003/005
	```

   > [!NOTE]
    > 記得將 `user` 和 `003/005` 改為你的實際使用者名稱、Bus 號碼和 Device 號碼。

3. 啟動 GNOME Boxes，進入目標虛擬機（如：`win11`）。
4. 回到終端機，進入 Flatpak 沙盒環境：

	```sh
	flatpak run --command="/bin/bash" org.gnome.Boxes
	```

5. 掛載 USB 裝置：

	```sh
	virsh attach-device win11 <(echo "<hostdev mode='subsystem' type='usb' managed='yes'><source><address bus='3' device='5'/></source><alias name='hostdev0'/></hostdev>")
	```

   > [!NOTE]
    > - 同樣別忘了修改虛擬機名稱、Bus 號碼與 Device 號碼。可使用 `virsh list --all` 查詢虛擬機名稱。
    > - 執行此步驟後，目標裝置在宿主機上將暫時無法使用。若為鍵鼠，請先準備其他裝置進行操作。

6. 成功掛載後，在 Windows 虛擬機中完成需要的操作。

---

## 自動化腳本

用 LLM 寫了一個簡單的腳本來簡化操作，邏輯和上述的手動操作相同，效果不錯。如果嫌手動操作麻煩可以用用。

```sh
#!/bin/bash
set -e

# Get and select USB device
mapfile -t usb_list < <(lsusb)
echo "Select the USB device to pass through:"
for i in "${!usb_list[@]}"; do
    echo "$((i+1))) ${usb_list[$i]}"
done
read -r -p "Enter a number and press Enter: " choice
usb_choice="${usb_list[$((choice-1))]}"

# Extract and format Bus and Device numbers
bus=$(echo "$usb_choice" | awk '{print $2}')
dev=$(echo "$usb_choice" | awk '{print $4}' | tr -d ':')
bus_xml=$((10#$bus))
dev_xml=$((10#$dev))

# Modify device permissions
sudo chown "$USER:$USER" "/dev/bus/usb/$bus/$dev"

# Get and select target VM
echo "Select the target virtual machine:"
mapfile -t vm_list < <(flatpak run --command="virsh" org.gnome.Boxes list --name | awk 'NF')
for i in "${!vm_list[@]}"; do
    echo "$((i+1))) ${vm_list[$i]}"
done
read -r -p "Enter a number and press Enter: " choice
vm_choice="${vm_list[$((choice-1))]}"

# Generate XML and attach device
xml_data="<hostdev mode='subsystem' type='usb' managed='yes'><source><address bus='$bus_xml' device='$dev_xml'/></source></hostdev>"
flatpak run --command="/bin/bash" org.gnome.Boxes -c "virsh attach-device $vm_choice <(echo \"$xml_data\")"
```

> [!NOTE]
> - 檔名取為 `boxes-usb-passthrough` 放到 `~/.local/bin`（其實都隨意啦），並允許作為程式執行。
> - 須先開啟 GNOME Boxes 並啟動好虛擬機，再呼叫這個腳本。
> - 輸入數字選擇目標 USB 裝置→輸入 sudo 密碼（手動操作也需要）→最後輸入數字選擇目標虛擬機就可以了。

![](01.avif  "哎呀，接了什麼 USB 裝置都被看光光了")

---

## 復原方法

此方法屬於臨時修改，只需簡單重新啟動電腦即可復原。

---

~~以後買周邊，一定挑能夠透過網頁調整的。~~