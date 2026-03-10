---
date: "2026-03-04T15:54:57+08:00"
title: "將 Input Remapper 與 Kando 搭配使用"
slug: ""
tags:
  - 
description: ""
summary: ""
cover: ""
draft: true
---
我經常使用滑鼠手勢。換用 Linux 系統（Bluefin，一個非常優雅的發行版）後，我花了一些時間研究如何只用滑鼠操作 Kando，同時又不影響滑鼠原有的按鍵功能。

出乎意料的是，Input Remapper 支援簡單的腳本編寫，這使得這一切成為可能。

**輸入映射器**是一款可以將滑鼠按鍵重新映射到鍵盤快速鍵的工具。它不僅可以直接將選單綁定到滑鼠按鍵（透過鍵盤快捷鍵），還支援更高級的設置，例如同時按下兩個滑鼠按鍵時顯示選單。

Input Remapper 的 GitHub 儲存庫：https://github.com/sezanzeb/input-remapper

你可以從那裡瞭解如何安裝、使用技巧等資訊，本文將聚焦在與 Kando 搭配使用的部分。

![](./01.avif "Input Remapper 的介面")

## 配置

有三種方式可以透過 Input Remapper 觸發 Kando 選單。分別是：

*為了盡可能展示最複雜的情況，以下皆使用 `Ctrl+Shift+F1` 當作例子，你可以修改成自己習慣的快捷鍵。*

### 選項 1：直接映射快捷鍵

這是最簡單的用法，按下單一滑鼠按鈕即可觸發 Kando 選單。

```
# 任意 Mouse Button
# Target: keyboard
KEY_LEFTCTRL + KEY_LEFTSHIFT + KEY_F1
```

- 優點：反應快速無延遲，設定簡單。
- 缺點：該按鈕會失去原本的功能，只適合有多餘按鈕的滑鼠。

### 選項 2：長按滑鼠按鈕

長按滑鼠按鈕即可觸發 Kando 選單，短按則為該按鈕原本的功能。下面以長按右鍵為例：

```
# Mouse Button RIGHT
# Target: keyboard + mouse
if_tap( key( BTN_RIGHT ), modify( KEY_LEFTCTRL, modify( KEY_LEFTSHIFT, key (KEY_F1 ))).wait( 100 ).hold( KEY_LEFTCTRL ), 150 )
```
> 最後面的數字 `150` 是長按所需的時間，單位為 ms。可自行調整，但過短容易造成誤觸。

- 優點：可保留按鍵原本短按的功能。
- 缺點：將失去按鍵原本長按的功能，持續按住並拖拽會變成操作 Kando 選單。

### 選項 3：同時按下兩個滑鼠按鈕

長按一個滑鼠按鈕，並短按另一個滑鼠按鈕時觸發。下面以長按滑鼠右鍵，並短按滑鼠左鍵為例：

```
# Mouse Button RIGHT
# Target: keyboard + mouse
if_tap( key( BTN_RIGHT ), hold( KEY_LEFTCTRL ))

# Mouse Button RIGHT + Mouse Button LEFT
# Target: keyboard
KEY_LEFTCTRL + KEY_LEFTSHIFT + KEY_F1
```

- 優點：按住右鍵，並多次短按左鍵可快速在相同快捷鍵的選單間切換。
- 缺點：將失去按鍵原本長按的功能，持續按住並拖拽會操作選單。