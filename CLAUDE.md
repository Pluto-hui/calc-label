# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

卷烟差价计算器 — 纯前端 PWA 应用，用于计算卷烟批发价与市场价之间的利润差价。面向手机端使用，部署在 GitHub Pages。

- **线上地址**: https://pluto-hui.github.io/calc
- **GitHub**: https://github.com/Pluto-hui/calc （main 分支）
- **技术栈**: 纯 HTML + CSS + JS，无框架，无构建工具
- **部署方式**: GitHub Actions 自动将 `web/` 目录部署到 `gh-pages` 分支

## 常用命令

```bash
# 本地开发 — 直接用浏览器打开
start web/index.html

# 部署 — push 到 main 分支，GitHub Actions 自动部署到 Pages
git push origin main
# 部署后等待约 1-2 分钟，访问 https://pluto-hui.github.io/calc

# 注意：用户在国内，GitHub HTTPS (443) 经常被墙，需要开 VPN 才能 push
```

## 关键架构

### 数据流
```
localStorage ←→ storage.js ←→ calculator.js (Calc 对象，内存状态) ←→ app.js (UI 渲染)
                                                                    ↔ brands.js (品牌匹配引擎)
```

### 核心模块

- **`js/storage.js`** — localStorage 读写。`loadBrands()` 首次使用时自动加载 `DEFAULT_BRANDS`（41 个常见国产卷烟品牌及批发价）。`DEFAULT_BRANDS` 也定义在此文件。
- **`js/calculator.js`** — `Calc` 对象：状态管理 + 计算逻辑。管理 `brands[]`、`hiddenPresets`、`wholesaleOverride`、`marketOverride`、`qtyCache`。提供 `calcProfit(idx)` 和 `calcSummary()`。
- **`js/brands.js`** — 品牌模糊匹配引擎。`BRAND_ALIASES` 表（50+ 别称映射）、多策略匹配（别名→精确→去括号→包含→关键词）、行解析 `parseLine()`。
- **`js/app.js`** — 全部 UI 交互。包括品牌列表渲染、事件委托、汇总更新、批量导入弹窗、保存为图片（html2canvas）、分享链接、Toast。
- **`js/sw.js`** — Service Worker，缓存版本 `price-calc-v5`，更新时需要 bump 版本号。

### 页面结构（3 个 Tab）

```
🧮 计算器（#page-calc）— 品牌列表行 + 汇总卡片 + 批量导入弹窗
📦 品牌库（#page-brands）— 文本编辑区，管理品牌列表
⚙️ 设置（#page-settings）— 深色模式开关 + 版本号
💾 保存（#saveImageBtn）— 弹出菜单：保存为图片 / 分享链接
```

### 汇总卡片

固定在 `bottom: 56px`，z-index 25，位于导航栏上方。金色边框光晕。利润显示最大最亮（`#ffb74d` + text-shadow）。

### 保存功能

使用 `html2canvas`（CDN 动态加载）截取 `#page-calc` 为 2x 分辨率 PNG。保存成功 Toast 显示价格摘要（前 5 个品牌 + 总利润）。

## 用户偏好和习惯

- 用户在中国大陆，GitHub 经常需要通过 VPN 访问
- 推送代码用 HTTPS + Personal Access Token（需 `repo` + `workflow` 权限）
- 偏好简洁直接的 UI，不需要的功能果断删除
- 手机端是主要使用场景
- 汇总卡片（利润计算）是最重要的核心区域
