---
layout: post
title: Fish Shell 终极配置指南
subtitle: 从安装到全栈开发环境的完整教程
categories: [Shell, Development]
tags: [fish, shell, terminal, development]
date: 2026-02-12 00:00:00 +0800
---

# Fish Shell 终极配置指南：从安装到全栈开发环境

Fish (Friendly Interactive Shell) 是一个极其注重开箱即用体验的 shell。本指南将带你从零开始，配置一个集成主流开发工具的高效环境。

## 1. 安装 Fish Shell

### 操作系统安装命令

- **macOS**:

```bash
brew install fish
```

- **Ubuntu / Debian**:

```bash
sudo apt update
sudo apt install fish
```

### 设置为默认 Shell

安装完成后，建议将 Fish 设置为默认 shell：

1. **查看 fish 路径**:

```bash
which fish
```

2. **将 fish 添加到可信 shell 列表**:

```bash
echo (which fish) | sudo tee -a /etc/shells
```

3. **修改默认 shell**:

```bash
chsh -s (which fish)
```

---

## 2. 安装插件管理器：Fisher

**Fisher** 是 Fish 最受欢迎的插件管理器，特点是轻量且快速。

```bash
curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher
```

---

## 3. 安装核心插件 (fzf & z)

### fzf (模糊搜索)

首先确保系统中已安装 `fzf` 二进制文件：

- **macOS**: `brew install fzf`
- **Ubuntu**: `sudo apt install fzf`

然后安装 Fish 插件以获得更好的集成：

```bash
fisher install PatrickF1/fzf.fish
```

> **功能**：使用 `Ctrl+R` 搜索历史，`Ctrl+Alt+F` 搜索文件。

### z (目录跳转)

`z` 可以让你根据访问频率快速跳转目录。

```bash
fisher install jethrokuan/z
```

> **用法**：`z your_dir_name` 即可快速直达。

---

## 4. 开发语言与工具安装

Fish 的环境变量配置与 Bash 不同，通常建议使用 `fish_add_path` 命令。

### Rust

使用官方脚本安装：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

安装后，在 `~/.config/fish/config.fish` 中添加：

```bash
fish_add_path $HOME/.cargo/bin
```

### Go

安装 Go 后（假设安装在 `/usr/local/go`），配置路径：

```bash
set -gx GOPATH $HOME/go
fish_add_path /usr/local/go/bin $GOPATH/bin
```

### uv (Python 管理器)

Astral 出品的极速 Python 包管理工具：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

`uv` 通常会自动尝试配置 Fish 路径，如果没有，请手动添加：

```bash
fish_add_path $HOME/.local/bin
```

### fnm (Node.js 版本管理)

比 nvm 快得多的 Node 管理器：

```bash
curl -fsSL https://fnm.vercel.app/install | bash
```

在 `config.fish` 中添加初始化命令：

```bash
fnm env --use-on-cd | source
```

### Bun (JavaScript Runtime)

```bash
curl -fsSL https://bun.sh/install | bash
```

配置路径：

```bash
set -gx BUN_INSTALL "$HOME/.bun"
fish_add_path $BUN_INSTALL/bin
```

---

## 5. 完整的 config.fish 示例

你的配置文件通常位于 `~/.config/fish/config.fish`。以下是一个整合了上述工具的配置模板：

```bash
if status is-interactive
    # 保持默认的问候语为空
    set -g fish_greeting ""

    # --- 环境变量与路径 ---

    # Rust
    fish_add_path $HOME/.cargo/bin

    # Go
    set -gx GOPATH $HOME/go
    fish_add_path /usr/local/go/bin $GOPATH/bin

    # Bun
    set -gx BUN_INSTALL "$HOME/.bun"
    fish_add_path $BUN_INSTALL/bin

    # Python (uv)
    fish_add_path $HOME/.local/bin

    # --- 工具初始化 ---

    # fnm (Node.js)
    fnm env --use-on-cd | source

    # --- 别名 (Aliases) ---
    alias ls='ls --color=auto'
    alias g='git'
    alias vim='nvim'
end
```

---

## 6. 进阶建议：Starship 提示符

为了让 Fish 看起来更现代，强烈建议安装 **Starship**：

1. **安装**: `curl -sS https://starship.rs/install.sh | sh`
2. **启用**: 在 `config.fish` 末尾添加：

```bash
starship init fish | source
```

现在，重启你的终端，享受飞一般的开发体验吧！

---
