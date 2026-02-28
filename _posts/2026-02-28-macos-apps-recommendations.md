---
layout: post
title: macOS 好用软件推荐
subtitle: 系统维护、录屏与日常效率工具
categories: [技术分享, macOS, 软件推荐]
tags: [macos, onyx, kap, motrix, 录屏, 系统维护, 下载]
date: 2026-02-28 00:00:00 +0800
---

记录一批在 macOS（含黑苹果）上自己觉得好用的软件，按用途分类，随用随补。

---

## 系统维护：OnyX

[**OnyX**](https://www.titanium-software.fr/en/onyx.html) 是 Titanium Software 出的**系统多功能维护工具**，可用来：

- 检查系统文件结构  
- 执行清理与日常维护（清理缓存、删除问题文件、重建数据库与索引等）  
- 配置 Finder、Dock、Safari 及部分系统应用参数  
- 卸载应用  

很多原本需要命令行完成的操作，用 OnyX 的图形界面即可完成，适合不想记命令的用户。

**注意**：**每个大版本 macOS 对应一个 OnyX 版本**，必须下载与当前系统匹配的版本（例如 macOS Sequoia 15 用 OnyX 4.8.x，Tahoe 26 用 4.9.x）。官网提供从老版本到最新版的下载，见 [OnyX 官网](https://www.titanium-software.fr/en/onyx.html)。

---

## 录屏 / GIF：Kap

[**Kap**](https://getkap.co/) 是一款**免费开源**的 macOS 录屏工具，特点包括：

- **多格式导出**：GIF、MP4、WebM、APNG，可选带音频  
- **录制范围**：全屏、指定窗口或自选区域  
- **简单编辑**：裁剪、点击高亮等  
- **插件扩展**：可对接 Giphy、Streamable 等  
- 支持 **Apple Silicon 与 Intel**，需 macOS 12+  

适合做教程动图、演示片段。官网下载：[getkap.co](https://getkap.co/)，源码见 [GitHub](https://github.com/wulkano/kap)。

---

## 下载器：Motrix

[**Motrix**](https://motrix.app/download) 是一款**免费开源**的跨平台下载管理工具，macOS 上可替代迅雷等：

- **协议支持**：HTTP/HTTPS、FTP、BT、Magnet 磁力，BT 支持选择性下载  
- **多任务**：最多 10 个并发任务，单任务最多 64 线程  
- **限速、端口映射**：支持限速与 UPnP/NAT-PMP  
- **界面**：简洁、支持深色模式，支持 Touch Bar、菜单栏实时速度  
- **BT 追踪器**：可自动更新每日 tracker 列表  

官网下载：[motrix.app](https://motrix.app/download)，也可 `brew install --cask motrix`；源码 [GitHub](https://github.com/agalwood/Motrix)（MIT）。

---

## 后续可补充

后续会按使用情况继续往本文补充，例如：剪贴板管理、窗口管理、终端/开发相关等。若你有特别想看到的类别，可以留言。
