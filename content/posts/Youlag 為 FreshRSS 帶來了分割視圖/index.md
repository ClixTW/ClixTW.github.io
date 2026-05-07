---
date: 2026-05-07T12:53:07+08:00
title: Youlag 為 FreshRSS 帶來了分割視圖
slug: youlag-freshrss-split-view
tags:
  - 碎碎念
  - RSS
  - FOSS
cover: 01.avif
summary: ""
description: ""
draft: false
---
今天 [Youlag](https://github.com/civilblur/youlag/releases/tag/v4.4.0) 熱騰騰的更新，為 FreshRSS 帶來了新的分割視圖功能！

![](01.avif)

原本的單欄視圖在瀏覽文章時，不需要頻繁左右移動視線；這種分割視圖，則是切換文章比較方便，也更能從右側滾動條了解當前閱讀進度。算是各有千秋吧，有點難決定要用哪種視圖。

---

順便附上個人目前用的 CSS 調整：

```css
/* 圖片圓角 */
.text img {
    border-radius: 18px !important;
}

/* 隱藏訂閱源錯誤提示 */
.category .title.error::before {
    display: none !important;
}

/* 放大 Ruby 字體 */
.text rt {
    font-size: 0.7rem !important;
}

/* 調整 Code 樣式 */
.text code {
  background-color: #383838 !important;
  color: #FFFFFF !important;
  border: none !important;
  border-radius: 6px !important;
}
```

> [!tip] 
> 在擴充功能裡找到 User CSS，點擊齒輪就能看到編輯的地方了。