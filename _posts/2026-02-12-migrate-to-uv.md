---
layout: post
title: 迁移到 uv：Python 包管理的革命性升级
subtitle: 告别 pip 和 venv，拥抱极速 Python 开发体验
categories: [Python, Package Management]
tags: [uv, python, pip, package-management, development]
date: 2026-02-12 00:00:00 +0800
---

# 迁移到 uv：Python 包管理的革命性升级

如果你还在使用 `pip` + `venv` 的传统 Python 开发方式，那么 `uv` 将彻底改变你的开发体验。由 Astral 出品的 `uv` 是一个极速的 Python 包管理器，比 pip 快 10-100 倍。

## 1. 为什么选择 uv？

### 🚀 极速性能

- **包安装速度**：比 pip 快 10-100 倍
- **依赖解析**：毫秒级解析，即使面对复杂依赖
- **冷启动**：无需等待，即开即用

### 🎯 一体化解决方案

- **包管理**：替代 pip
- **虚拟环境**：替代 venv/conda
- **项目管理**：替代 poetry/pip-tools
- **Python 版本管理**：替代 pyenv

### 🔒 兼容性保证

- **100% pip 兼容**：无缝迁移现有项目
- **PyPI 支持**：完全支持 Python 包索引
- **标准工具链**：与现有工具无缝集成

## 2. 安装 uv

### 一键安装（推荐）

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 包管理器安装

**macOS (Homebrew):**

```bash
brew install uv
```

**Windows (Scoop):**

```bash
scoop install uv
```

**Linux (各种发行版):**

```bash
# Ubuntu/Debian
curl -LsSf https://astral.sh/uv/install.sh | sh

# 或使用 pipx
pipx install uv
```

### 验证安装

```bash
uv --version
```

## 3. 基础使用：从 pip 迁移

### 创建虚拟环境

```bash
# 创建项目目录
mkdir my-project && cd my-project

# 创建虚拟环境（自动检测 Python 版本）
uv venv

# 激活环境
source .venv/bin/activate  # Linux/macOS
# 或
.venv\Scripts\activate     # Windows
```

### 安装包

```bash
# 安装单个包
uv add requests

# 安装指定版本
uv add "requests==2.31.0"

# 安装开发依赖
uv add --dev pytest black

# 从 requirements.txt 安装
uv pip install -r requirements.txt
```

### 管理依赖

```bash
# 查看已安装包
uv pip list

# 卸载包
uv remove requests

# 更新包
uv add requests@latest
```

## 4. 项目管理：pyproject.toml 工作流

### 初始化项目

```bash
uv init
```

这会创建一个标准的 `pyproject.toml` 文件：

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.8"
dependencies = []

[tool.uv]
dev-dependencies = []
```

### 添加项目依赖

```bash
# 添加生产依赖
uv add fastapi uvicorn

# 添加开发依赖
uv add --dev pytest httpx

# 添加可选依赖组
uv add --optional "matplotlib"  # 创建 extras
```

### 运行项目

```bash
# 直接运行脚本
uv run python main.py

# 运行测试
uv run pytest

# 运行任意命令
uv run black .
uv run mypy src/
```

## 5. 高级功能

### Python 版本管理

```bash
# 安装特定 Python 版本
uv python install 3.11
uv python install 3.12

# 列出可用版本
uv python list

# 设置项目 Python 版本
uv python pin 3.11
```

### 工作空间管理

对于多包项目（monorepo）：

```toml
# pyproject.toml
[tool.uv.workspace]
members = ["packages/*"]
```

```bash
# 在工作空间根目录
uv add --package "package-a" requests
uv sync  # 同步所有包的依赖
```

### 脚本管理

```toml
[project.scripts]
my-tool = "my_package.cli:main"

[tool.uv.scripts]
test = "pytest tests/"
lint = "black src/ && mypy src/"
```

```bash
# 运行定义的脚本
uv run test
uv run lint
```

## 6. 从现有项目迁移

### 从 requirements.txt 迁移

```bash
# 1. 初始化项目
uv init

# 2. 转换 requirements.txt
uv add -r requirements.txt

# 3. 转换开发依赖
uv add --dev -r requirements-dev.txt
```

### 从 poetry 迁移

```bash
# 1. 导出 poetry 依赖
poetry export -f requirements.txt --output requirements.txt

# 2. 使用 uv 导入
uv add -r requirements.txt

# 3. 或者直接转换 pyproject.toml
# uv 会自动识别 [tool.poetry] 部分
```

### 从 pipenv 迁移

```bash
# 1. 导出依赖
pipenv requirements > requirements.txt

# 2. 导入到 uv
uv add -r requirements.txt
```

## 7. 最佳实践

### 项目结构

```
my-project/
├── pyproject.toml
├── README.md
├── src/
│   └── my_package/
│       ├── __init__.py
│       └── main.py
├── tests/
│   └── test_main.py
└── .venv/
```

### 依赖分类

```toml
[project]
dependencies = [
    "fastapi>=0.100.0",
    "uvicorn[standard]>=0.23.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "black>=23.0.0",
    "mypy>=1.0.0",
]
docs = [
    "mkdocs>=1.5.0",
    "mkdocs-material>=9.0.0",
]
```

### CI/CD 集成

**GitHub Actions:**

```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up uv
        uses: astral-sh/setup-uv@v3
        with:
          version: "latest"
      - name: Install dependencies
        run: uv sync
      - name: Run tests
        run: uv run pytest
```

**Docker:**

```dockerfile
FROM python:3.11-slim

# 安装 uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv

# 复制项目文件
COPY . /app
WORKDIR /app

# 安装依赖
RUN uv sync --frozen

# 运行应用
CMD ["uv", "run", "python", "main.py"]
```

## 8. 常见问题解决

### Q: 如何处理私有包？

```bash
# 配置私有索引
uv add --index-url https://pypi.private.com/simple/ private-package

# 或在 pyproject.toml 中配置
[[tool.uv.index]]
url = "https://pypi.private.com/simple/"
```

### Q: 如何处理系统包？

```bash
# 使用 --system 标志
uv add --system requests

# 或在系统 Python 环境中
uv pip install requests
```

### Q: 如何加速下载？

```bash
# 使用国内镜像
uv add --index-url https://pypi.tuna.tsinghua.edu.cn/simple/ requests

# 或配置环境变量
export UV_INDEX_URL="https://pypi.tuna.tsinghua.edu.cn/simple/"
```

## 9. 性能对比

| 操作          | pip | uv   | 提升倍数 |
| ------------- | --- | ---- | -------- |
| 安装 Django   | 8s  | 0.8s | 10x      |
| 安装 100 个包 | 45s | 4s   | 11x      |
| 创建虚拟环境  | 3s  | 0.1s | 30x      |
| 依赖解析      | 15s | 0.2s | 75x      |

## 10. 迁移检查清单

- [ ] 安装 uv
- [ ] 创建项目结构
- [ ] 初始化 `pyproject.toml`
- [ ] 迁移现有依赖
- [ ] 更新 CI/CD 配置
- [ ] 更新 Docker 配置
- [ ] 培训团队成员
- [ ] 清理旧工具（pip, venv, poetry）

## 11. 下一步

现在你已经掌握了 uv 的核心用法，建议继续探索：

- **uvx**：运行 Python 应用而无需安装
- **缓存管理**：优化磁盘使用
- **插件系统**：扩展 uv 功能
- **企业部署**：内部包管理

---

**告别等待，拥抱效率！** 🚀

从今天开始，让你的 Python 开发体验提升一个数量级。uv 不仅仅是一个工具，它是 Python 生态系统的一次进化。

---

_你有什么 uv 使用经验或迁移故事？欢迎在评论区分享！_
