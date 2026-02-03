---
date: 2026-02-02T01:28:45+08:00
title: 將書籤欄顯示在 Zen 的側邊欄
slug: ""
tags:
  - FOSS
description: ""
summary: ""
cover: ""
draft: true
---

```css
/* ========================================================================== */

.urlbar-input,
.urlbar-input[aria-expanded="false"] {
  text-align: center !important;
}

.urlbar-input[aria-expanded="true"] {
  text-align: match-parent !important;
}


/* ========================================================================== */

#statuspanel {
  margin: 4px !important;
}

#statuspanel-label {
  border-radius: 6px !important;
  padding: 2px 10px !important;
  border: 1px solid var(--zen-colors-border) !important;
  background: var(--zen-colors-tertiary) !important;
  color: var(--lwt-text-color) !important;
}


/* ========================================================================== */

#PlacesToolbarItems {
  justify-content: center !important;
  min-height: 36px !important;
}


/* ========================================================================== */

#TabsToolbar-customization-target {
  visibility: visible !important;
  padding-top: 9px !important;
}

menuitem {
  white-space: nowrap !important;
}

.bookmark-item label {
  width: auto !important;
}

.menu-highlightable-text {
  display: none !important;
}
```