---
date: 2026-02-26T17:45:34+08:00
title: 透過 davincibox 部署 DaVinci Resolve
slug: ""
tags:
  - 
description: ""
summary: ""
cover: ""
draft: true
---
#Linux #DaVinci 

## 安裝

1. 從 [Blackmagic](https://www.blackmagicdesign.com/products/davinciresolve/) 下載安裝檔。
2. 透過 [davincibox](https://github.com/zelikos/davincibox) 腳本安裝 ：

```
./setup.sh ./DaVinci_Resolve_Studio_{version}_Linux.run
```

> 路徑不可包含中文。
> 記得將安裝檔解壓縮，取出其中 `.run` 檔案。

---
*從此處開始，提到的指令皆於容器中執行。*

## 解決 Fctix 5 無法輸入

1. 安裝軟體包取得所需組件：

```
sudo dnf install fcitx5-qt5
```

2. 建立 Qt 庫連結：

```
sudo mkdir -p /opt/resolve/libs/plugins/platforminputcontexts
```

```
sudo cp /usr/lib64/qt5/plugins/platforminputcontexts/libfcitx5platforminputcontextplugin.so \
/opt/resolve/libs/plugins/platforminputcontexts/
```

參考自：[DaVinci Resolve 在 Linux 下的输入法支持](https://sh.alynx.one/posts/Input-Method-Support-for-DaVinci-Resolve-on-Linux/)

3. 修改啟動腳本：

```
sudo nano /usr/bin/run-davinci
```

4. 加入以下內容：

```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
export QT_PLUGIN_PATH=/opt/resolve/libs/plugins:/usr/lib64/qt5/plugins:$QT_PLUGIN_PATH
export QT_LOGGING_RULES="qt.qpa.input*=true"
```

## 工作室金鑰

1. 容器內執行：

```
cd /opt/resolve
sudo perl -pi -e 's/\x03\x00\x89\x45\xFC\x83\x7D\xFC\x00\x74\x11\x48\x8B\x45\xC8\x8B/\x03\x00\x89\x45\xFC\x83\x7D\xFC\x00\xEB\x11\x48\x8B\x45\xC8\x8B/' bin/resolve
sudo perl -pi -e 's/\x74\x11\x48\x8B\x45\xC8\x8B\x55\xFC\x89\x50\x58\xB8\x00\x00\x00/\xEB\x11\x48\x8B\x45\xC8\x8B\x55\xFC\x89\x50\x58\xB8\x00\x00\x00/' bin/resolve
sudo perl -pi -e 's/\x41\xb6\x01\x84\xc0\x0f\x84\xb0\x00\x00\x00\x48\x85\xdb\x74\x08\x45\x31\xf6\xe9\xa3\x00\x00\x00/\x41\xb6\x00\x84\xc0\x0f\x84\xb0\x00\x00\x00\x48\x85\xdb\x74\x08\x45\x31\xf6\xe9\xa3\x00\x00\x00/' bin/resolve
echo -e "LICENSE blackmagic davinciresolvestudio 999999 permanent uncounted\n  hostid=ANY issuer=CGP customer=CGP issued=28-dec-2023\n  akey=0000-0000-0000-0000 _ck=00 sig=\"00\"" | sudo tee .license/blackmagic.lic
```

## 修復 CUDA 問題

1. 克隆一份 davincibox，指定家目錄。

![](./01.avif)

> 指定的家目錄位置不需要和我一樣，喜歡放哪放哪。

2. 修改容器中的 .bashrc 檔案：

```
nano ~/.bashrc
```

3. 加入以下內容：

```bash
# --- Distrobox NVIDIA Library Fix (for .bashrc) ---
NEEDS_LDCONFIG_CHECK=0

fix_nvidia_symlink() {
    local link_name="$1"    # e.g., /usr/lib64/libcuda.so
    local target_name="$2"  # e.g., /usr/lib64/libcuda.so.1

    if [ ! -e "${target_name}" ]; then
        return 1 
    fi

    if { [ -L "${link_name}" ] && [ ! -e "${link_name}" ]; } || \
       { [ -f "${link_name}" ] && [ ! -s "${link_name}" ]; } || \
       [ ! -e "${link_name}" ]; then
        if [ -L "${link_name}" ] && [ ! -e "${link_name}" ]; then
            echo "NVIDIA FIX: Fixing broken ${link_name} symlink..."
        elif [ -f "${link_name}" ] && [ ! -s "${link_name}" ]; then
            echo "NVIDIA FIX: Fixing empty ${link_name}..."
        elif [ ! -e "${link_name}" ]; then
            echo "NVIDIA FIX: Creating missing ${link_name} symlink..."
        fi
        sudo rm -f "${link_name}" 
        sudo ln -sf "${target_name}" "${link_name}" && NEEDS_LDCONFIG_CHECK=1
    fi
}

fix_nvidia_symlink "/usr/lib64/libcuda.so" "/usr/lib64/libcuda.so.1"
fix_nvidia_symlink "/usr/lib64/libnvcuvid.so" "/usr/lib64/libnvcuvid.so.1"

if [ "$NEEDS_LDCONFIG_CHECK" -eq 1 ]; then
    echo "NVIDIA FIX: Running sudo ldconfig due to library fixes..."
    sudo ldconfig
fi
unset NEEDS_LDCONFIG_CHECK
# --- End Distrobox NVIDIA Library Fix ---
```

參考自：[CUDA Symlink Persistence #1764](https://github.com/89luca89/distrobox/issues/1764)

## 建立啟動捷徑

1. 建立 .desktop 檔案：

```bash
nano ~/.local/share/applications/resolve.desktop
```

2. 加入以下內容：

```
[Desktop Entry]
Version=1.0
Type=Application
Name=DaVinci Resolve
GenericName=DaVinci Resolve
Comment=Revolutionary new tools for editing, visual effects, color correction and professional audio post production, all in a single application!
Path=/var/home/ctw
Exec=distrobox-enter -n davincibox-clone -- /usr/bin/run-davinci /opt/resolve/bin/resolve %u
Terminal=false
MimeType=application/x-resolveproj;
Icon=davinci-resolve
StartupNotify=true
StartupWMClass=resolve
```

## 新增字體

1. 指定字體目錄：

```
切換至 Fusion - 頂部工具列 Fusion - 
Fusion 設置… - Path Map 路徑映射 - 
Fonts - 到：自訂字形目錄
```