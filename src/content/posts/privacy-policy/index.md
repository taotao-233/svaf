---
title: 隐私政策
published: 2026-05-09T00:00:00
description: 本隐私政策适用于 AcoFork Blog（以下简称"我们"）。
image: "img/privacy-policy.png"
draft: false
hide: true
lang: ""
---

## Cookie 与本地存储

### 必要（始终加载）

以下服务在所有页面加载时运行，无法通过 Cookie 设置关闭：

| 服务 | 说明 |
|------|------|
| [Umami](https://u.2x.nz)（自托管） | 收集匿名访问数据，用于统计浏览量 |
| [Cloudflare Web Analytics](https://www.cloudflare.com/analytics/) | 收集匿名访问数据，无 Cookie、无指纹追踪 |
| CF Umami（辅助统计） | 用于全站浏览量计数，同时作为浏览量 API 端点 |

### 功能（需用户同意）

| 服务 | 说明 |
|------|------|
| [Giscus](https://giscus.app) | 基于 GitHub Discussions 的评论系统，用户交互时受 [GitHub 隐私政策](https://docs.github.com/en/site-policy/privacy-policies/github-privacy-statement)约束 |

### 分析（需用户同意）

| 服务 | 说明 |
|------|------|
| [百度统计](https://tongji.baidu.com/) | 收集站点访问情况 |
| [Google Analytics (GA4)](https://analytics.google.com) | 收集站点访问情况 |
| [Microsoft Clarity](https://clarity.microsoft.com) | 收集用户行为分析数据 |

### 本地存储

我们使用浏览器 `localStorage` 存储以下数据：

| 键名 | 用途 |
|------|------|
| `cookie-consent-preferences` | Cookie 同意偏好设置 |
| `theme` | 用户主题偏好（亮色/暗色/跟随系统） |
| 论坛相关键名 | 论坛登录凭证及环境配置 |

## 其他第三方服务

### 论坛验证

论坛页面（`/forum/*`）使用 [Cloudflare Turnstile](https://www.cloudflare.com/products/turnstile/) 进行人机验证，用于登录、注册、发帖和评论操作。Turnstile 会收集浏览器信息用于机器人检测。

### 按需加载的 CDN 资源

以下资源仅在页面包含相关内容时按需加载：

| 资源 | CDN | 用途 |
|------|-----|------|
| [Mermaid.js](https://mermaid.js.org) | jsdelivr | 渲染流程图、时序图等图表 |
| [Highlight.js](https://highlightjs.org) | cdnjs | 代码块语法高亮 |
| [dash.js](https://dashjs.org) | cdn.dashjs.org | DASH 视频播放 |
| [Iconify](https://iconify.design) | api.iconify.design | 图标资源 |

### 第三方图片源

页面中可能加载来自以下来源的图片：

- QQ 头像 API（`q2.qlogo.cn`）
- 阿里云 CDN（`img.alicdn.com`）

## 自托管服务

以下服务由我们自行托管，数据不经过第三方：

| 服务 | 地址 | 用途 |
|------|------|------|
| 博客主站 | `2x.nz` | 文章发布与展示 |
| Umami 统计 | `u.2x.nz` | 匿名访问统计 |
| 论坛后端 | `i.2x.nz` | 论坛数据存储与 API |

## 数据控制

您可以通过页面底部的「Cookie 设置」按钮随时修改 Cookie 同意偏好。清除浏览器 `localStorage` 可删除本地存储的所有数据。
