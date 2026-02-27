---
layout: post
title: 不依赖 Mac 的黑苹果安装实践：GitHub Actions + OpCore Simplify
subtitle: 使用 macOS-iso-builder、OpCore-Simplify、USBToolBox 与 OCAT 搭建可引导 EFI
categories: [Hackintosh, macOS]
tags: [hackintosh, opencore, macos-iso-builder, OpCore-Simplify, USBToolBox, OCAT, ProperTree, NootRX]
date: 2026-02-27 00:00:00 +0800
---

## 适用场景与整体思路

这篇笔记记录的是**在没有现成 Mac 的情况下安装黑苹果**，并且结合自己验证可行的一套流程，重点是：

- **用 GitHub Actions 在线生成官方 macOS 安装镜像**：`macOS-iso-builder`  
- **用 Rufus 写入 U 盘，得到基础安装盘**  
- **用 OpCore Simplify 生成 EFI 骨架**  
- **用 USBToolBox 做 USB 端口映射**  
- **用 OCAT 或 ProperTree 校对并维护 `config.plist`**  

整套工具链大致对应下面这些项目（都建议先 Star 收藏，方便后续更新）：

- 安装镜像构建：[macOS-iso-builder](https://github.com/LongQT-sea/macos-iso-builder)  
- USB 映射工具：[USBToolBox/tool](https://github.com/USBToolBox/tool)  
- OpenCore EFI 自动化生成：[OpCore-Simplify](https://github.com/lzhoang2801/OpCore-Simplify)  
- plist 编辑与 OC 快照：
  - [ProperTree](https://github.com/corpnewt/ProperTree)  
  - 或 [OCAuxiliaryTools（OCAT）](https://github.com/ic005k/OCAuxiliaryTools)  
- 新款 AMD 显卡补丁（RDNA 2）：[NootRX](https://github.com/ChefKissInc/NootRX)  

> **免责声明**：黑苹果涉及 Apple 许可协议和硬件兼容性风险，以下内容仅供学习研究使用，请自行评估并承担风险。

---

## 第 1 步：用 GitHub Actions 在线生成 macOS 安装镜像

这里使用的是 `macOS-iso-builder`：  
[macOS-iso-builder](https://github.com/LongQT-sea/macos-iso-builder)

### 1.1 Fork 仓库并启用 Actions

1. 打开仓库页面，点击右上角 **Fork** 到自己的账号下。  
2. 进入自己 Fork 出来的仓库，切到 **Actions** 标签页。  
3. 首次使用需要点一次绿色按钮：**"I understand my workflows, go ahead and enable them"** 启用工作流。

### 1.2 选择系统版本与镜像类型

进入 Actions 页面后，左侧会看到类似下面这样的工作流列表：

![macOS-iso-builder Actions 工作流列表](/assets/image-0c040a44-2aac-41a7-b096-e44cee285940.png)

1. 一般有两类工作流可以用：
   - **01. Build Recovery ISO image**：推荐作为小体积恢复镜像，下载和构建都更快。  
   - 各版本对应的 **Build macOS \[版本名\]**（如 *Build macOS Sonoma*、*Build macOS Tahoe* 等），相当于 README 里说的 Full Installer。  
2. 点进你想要的工作流，点击右侧的 **"Run workflow"**，在弹出的表单中：
   - 如果有 **macOS version** 选项，就选你要的版本（Sonoma、Sequoia、Tahoe 等）。  
   - **Image format**：**务必选择 `dmg`**。  
     - `iso`：只适合虚拟机挂载，不适合作为实机启动 U 盘镜像。  
     - `dmg`：适合用 Rufus/`dd` 写入 U 盘，做物理机安装盘。  
3. 再次点击绿色按钮开始运行，等待 GitHub Actions 完成。

### 1.3 下载并解压镜像

1. 工作流完成后，刷新页面，下拉到 **Artifacts** 区域。  
2. 下载类似 `macOS_Sonoma_xx.dmg.zip` 或 `macOS_Sequoia_xx.dmg.zip` 的压缩包。  
3. 在本地解压，得到的文件名通常形如：`macOS_Sonoma.dmg.img` —— 这是给 Rufus 等工具直接识别用的。

---

## 第 2 步：用 Rufus 制作基础安装 U 盘

这一步建议在 **Windows 实机** 上完成，使用 [Rufus](https://rufus.ie)。

1. 插入一个容量 ≥ 16GB 的 U 盘，确认内部数据已经备份（会被完全格式化）。  
2. 打开 Rufus，关键选项：
   - **设备**：选择你的 U 盘。  
   - **启动选择**：浏览并选中上一步解压出来的 `*.dmg.img` 文件。  
   - 其余分区类型、目标系统等一般保持默认即可。  
3. 点击 **开始**，等待写入完成。  

完成后，你得到的是一个**只有基础安装文件、还没有合适 EFI** 的启动盘，接下来需要为它准备对应硬件的 EFI，  
并把生成好的 `EFI` 文件夹**直接放进这个 Rufus 写好的 U 盘的 EFI 分区**（覆盖掉原来的 EFI 即可）。

---

## 第 3 步：用 OpCore Simplify 生成基础 EFI

OpCore Simplify 的目标是：**自动根据硬件生成一份尽量合理、可开机的 OpenCore EFI 骨架**。  
项目地址：[OpCore-Simplify](https://github.com/lzhoang2801/OpCore-Simplify)

### 3.1 启动工具

- **Windows**：运行仓库中的 `OpCore-Simplify.bat`  
- **macOS / Linux**：分别运行 `OpCore-Simplify.command` 或 `OpCore-Simplify.py`

进入主菜单后，按照你给出的实践，可以这么操作：

### 3.2 扫描硬件兼容性并选择系统版本

1. 在主菜单中选择 **`1`（一般是 Compatibility Checker/硬件兼容性检查）**。  
2. 在子菜单中选择 **`E`（Export hardware report/导出硬件报告，亦可自动扫描当前机器配置）**：  
   - 工具会读取 CPU、主板、显卡、声卡等信息，并生成报告。  
3. 根据扫描结果，工具会给出**支持的 macOS 版本列表**，从中选择你想装的版本（比如从 High Sierra 一直到 Tahoe）。  

> 建议优先选择列表中标记为"推荐"或与你硬件兼容性最好的那几个版本。

### 3.3 显卡（GPU）相关设置：6000 系列及以上优先使用 NootRX

当工具弹出显卡相关选项时，如果你使用的是**AMD RX 6000 系列及以上（RDNA 2/更新架构）独显**（如 RX 6600/6700/6800/6900 以及更新型号），  
请优先选择使用 [NootRX](https://github.com/ChefKissInc/NootRX) 方案，而**不要选择旧实现**。

- 旧实现通常兼容性较差、补丁更复杂。  
- NootRX 是专门针对 rDNA 2 dGPU 的补丁 kext，维护较新，体验更好。  

记得在后续生成 EFI 时，让工具把 NootRX 一并纳入 Kext 列表。

### 3.4 声卡（Audio）设置

后面程序会出现声卡相关菜单（如选择 Layout ID），根据你的经验：

- 一般情况下**保持默认选项即可**，不要盲目更改。  
- 真实使用中如果后续出现没有声音，可以再回来针对性调整 Layout ID。

### 3.5 生成 EFI 骨架

1. 在前面选好系统版本、显卡补丁、基础 ACPI/Kext 方案后，按照提示继续，通常菜单中会有一个类似 **`6` 的“Build OpenCore EFI”选项**。  
2. 按下对应数字后，程序会自动：
   - 下载对应版本的 OpenCorePkg 与常见 kext（Lilu、WhateverGreen 等）。  
   - 按你的硬件情况生成 ACPI 补丁与 `config.plist`。  
   - 输出一份完整的 EFI 目录（含 `EFI/OC` 子目录）。

在这一阶段结束后，你已经有了一份**可用但还未定制 USB Map 的 EFI**。  
这时可以先把 `EFI` 目录复制到 Rufus 制作的安装 U 盘的 EFI 分区里，用这一个 U 盘完成后续所有安装与引导。  
接下来要结合 USBToolBox 做 USB 端口映射。

---

## 第 4 步：使用 USBToolBox 生成 USB Map

USB 映射使用的是：[USBToolBox/tool](https://github.com/USBToolBox/tool)

### 4.1 运行 USBToolBox 并扫描端口

1. 根据你的系统下载对应版本（Windows.exe / macOS.zip 等）。  
2. 打开程序，进入端口发现（Discover Ports）界面。  
3. 按提示操作：
   - 将一个 USB 设备依次插入每一个物理 USB 口，等待列表中识别到设备后再换下一个口。  
   - 如果在 Windows 下，USBToolBox 会自动推断 USB 2/3 伴生端口类型。  

### 4.2 生成 USB 映射 kext

1. 完成所有端口插拔测试后，进入“选择端口”（Select Ports）界面。  
2. 根据实际需要保留/禁用端口，并确认每个端口类型。  
3. 按提示按键（通常是 **`K`**）生成 USB Map kext。  
4. 工具会导出一个类似 `UTBMap.kext` 或自定义命名的 kext。

### 4.3 回到 OpCore Simplify，接管默认 USB 配置

在你给出的实战过程中，OpCore Simplify 会在某一步提示：

- 使用 USBToolBox 生成 USB Map 后，**删除默认的 USB 文件（比如 `UTBDefault.kext`）**。  
- 然后按照程序提示输入命令进行确认。  
- 程序会自动打开刚刚生成好的 EFI 路径，提示你把 USBToolBox 生成的 USB Map kext 放进去。

实践建议：

1. 打开 OpCore Simplify 生成的 EFI 目录，进入 `EFI/OC/Kexts/`。  
2. 删除原本默认的 USB 相关 kext（例如 `UTBDefault.kext` 之类的占位文件）。  
3. 把 USBToolBox 导出的 USB Map kext 复制到该目录。  
4. 确认 `config.plist` 中引用的是你刚刚替换进去的 USB Map kext。

这一步完成后，你的 EFI 已经具备了**针对当前主板的 USB 正确映射**，可以极大提升稳定性（睡眠、唤醒、安装过程中键盘鼠标可用等）。

---

## 第 5 步：用 ProperTree 或 OCAT 校对与维护 config.plist

在 plist 编辑工具方面，可以二选一：

- 轻量、跨平台 GUI plist 编辑器：[ProperTree](https://github.com/corpnewt/ProperTree)  
- 功能更丰富、专门针对 OpenCore 的 GUI 工具：[OCAuxiliaryTools（OCAT）](https://github.com/ic005k/OCAuxiliaryTools)

### 5.1 使用 ProperTree 的思路（简要）

1. `git clone` 或下载 ZIP，运行 `ProperTree.command`（macOS）或 `ProperTree.bat`（Windows）。  
2. 打开你的 `EFI/OC/config.plist`。  
3. 使用菜单中的 **OC Clean Snapshot/OC Snapshot** 功能：
   - 指定 `EFI/OC` 目录后，自动同步 `ACPI/Add`、`Kernel/Add`、`Misc/Tools`、`UEFI/Drivers` 等条目。  
   - 移除无用条目，保证 kext 顺序与实际磁盘内容一致。  
4. 检查以下关键点：
   - `Misc -> Security` 中的 SecureBootModel、ScanPolicy 是否符合你的安装需求。  
   - `PlatformInfo -> Generic` 中的 SMBIOS 信息是否合理（避免与真机冲突）。  
   - `Kernel -> Add` 中是否正确引用了 USB Map kext、NootRX 等。

### 5.2 使用 OCAT 的思路（简要）

如果你更偏向图形界面与自动迁移，可以使用 OCAT：

1. 运行 OCAT，挂载 EFI 分区，自动打开 `config.plist`。  
2. 点击更新按钮，让 OCAT 帮你：
   - 升级 OpenCore 版本到最新。  
   - 同步/更新常见 kext。  
3. 根据 CPU 类型选择预设 Quirks（Intel/AMD）。  
4. 在 `Kernel`、`ACPI`、`UEFI` 等页面中，确认：
   - 仅保留当前实际需要的条目。  
   - USB Map 与 NootRX 等 kext 已勾选启用。  
5. 保存退出，OCAT 会自动按最新 schema 写回 `config.plist`。

---

## 第 6 步：BIOS 设置与首次引导安装

在真正尝试从 U 盘启动前，记得先检查 BIOS/UEFI 设置：

- **开启 AHCI**，关闭 Intel RST/RAID。  
- **关闭 CSM/Legacy Boot**，改为纯 UEFI 启动。  
- 关闭安全启动（Secure Boot）。  
- 如有需要，关闭 VT-d 或按 OpenCore 指南正确配置相关 Quirks。

然后：

1. 将前面制作好的 U 盘插入目标机器。  
2. 从 BIOS 启动菜单选择 U 盘启动，进入 **OpenCore 启动界面**。  
3. 在启动项中选择对应的 **Install macOS** 条目，进入安装向导。  
4. 正常分区、格式化（APFS/Guid），按照向导完成安装过程。  

安装完成后，建议：

1. 把 U 盘上的 EFI 目录复制到系统盘的 EFI 分区（macOS 下可以用“磁盘工具”或 `diskutil` 挂载 EFI，Windows 下**推荐使用 DiskGenius** 挂载和浏览 EFI 分区，也可以用自带磁盘管理/分区工具分配盘符）。  
2. 以后就可以直接从系统盘 EFI 启动，而不再依赖 U 盘；如果后期需要修改 EFI，同样通过**磁盘工具 / DiskGenius 等分区工具挂载 EFI 分区**后，替换或编辑其中的 `EFI` 内容即可。

---

## 检查清单（根据实战流程整理）

### 镜像与启动盘

- [ ] Fork 并启用 [macOS-iso-builder](https://github.com/LongQT-sea/macos-iso-builder) 的 GitHub Actions  
- [ ] 选择目标 macOS 版本，**镜像类型选 `dmg`（实机安装）**  
- [ ] 下载并解压 Actions 生成的 `*.dmg.zip`，得到 `*.dmg.img`  
- [ ] 在 Windows 下用 Rufus 将 `*.dmg.img` 写入 U 盘  

### EFI 生成与 USB 映射

- [ ] 使用 [OpCore-Simplify](https://github.com/lzhoang2801/OpCore-Simplify) 扫描硬件（菜单 `1` → `E`）  
- [ ] 根据兼容性结果选择合适的 macOS 版本  
- [ ] 显卡部分：如为 AMD RDNA 2，选择 [NootRX](https://github.com/ChefKissInc/NootRX) 方案  
- [ ] 声卡 Layout ID 暂时保持默认，必要时再调  
- [ ] 通过 USBToolBox 扫描所有 USB 端口并生成 USB Map kext  
- [ ] 删除 EFI 中默认 USB kext（如 `UTBDefault.kext`），换成 USBToolBox 生成的 USB Map  

### config.plist 校对与优化

- [ ] 使用 [ProperTree](https://github.com/corpnewt/ProperTree) 或 [OCAuxiliaryTools](https://github.com/ic005k/OCAuxiliaryTools) 打开 `config.plist`  
- [ ] 执行 OC Snapshot/更新功能，同步 ACPI/Kernel/UEFI 列表  
- [ ] 确认 NootRX、USB Map 等 kext 已启用且顺序合理  
- [ ] 检查 SecureBootModel、ScanPolicy、SMBIOS 等关键配置  

### 安装与收尾

- [ ] BIOS 中关闭 CSM、Secure Boot，启用 AHCI、UEFI 启动  
- [ ] 通过 OpenCore 启动安装器完成 macOS 安装  
- [ ] 将 U 盘 EFI 复制到系统盘 EFI 分区，设置为默认启动  

---

以上就是基于你目前实践出来的一整套**无 Mac 前提的黑苹果安装流程**，并按博客的形式整理到了 `_posts` 中。后续如果你有更多踩坑经验，可以在这篇文章基础上继续补充，例如：特定主板/CPU 的 ACPI 细节、不同版本 macOS 的稳定性对比等。

