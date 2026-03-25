# CTF Agent 使用指南

本指南将帮助您使用重构后的 CTF Agent 进行单题求解、手动录入题目以及接入自定义 LLM API。

## 1. 环境配置

在开始之前，请确保您已安装 Python 3.14+ 及项目依赖。

```bash
# 安装当前目录下的项目及其依赖
pip install .

# 或者，如果你打算修改代码进行开发（推荐），使用可编辑模式安装：
pip install -e .

# 也可用 uv 一键安装适合的 Python 版本并安装依赖：
uv sync
```

### 配置 API 密钥

在项目根目录下创建或编辑 `.env` 文件。

**通用 OpenAI 兼容接口 (新增)**
您可以接入本地模型 (如 Ollama, vLLM) 或第三方聚合 API。

```env
# 必填：API 基础地址
GENERIC_OPENAI_BASE_URL=http://localhost:11434/v1
# 必填：API Key (本地模型可随意填写)
GENERIC_OPENAI_API_KEY=ollama
```

**其他常用接口**

```env
# Anthropic
ANTHROPIC_API_KEY=sk-ant-...
# OpenAI
OPENAI_API_KEY=sk-...
```

### Docker 环境准备 (必须)

Agent 依赖 Docker 容器来执行不安全的代码和工具。首次运行前必须构建沙箱镜像。

```bash
docker build -t ctf-sandbox -f sandbox/Dockerfile.sandbox .
```

---

## 2. 核心命令

### 初始化题目 (`init`)

用于在本地创建一个标准的题目目录结构，无需依赖 CTFd 平台。

```bash
python backend/cli.py init <题目目录名> [选项]
```

### 运行求解 (`run`)

用于启动 Agent 对指定目录进行求解。

```bash
python backend/cli.py run --challenge <题目目录路径> --offline [选项]
```

---

## 3. 实战示例

### 示例 A：Web 题目 (含附件与靶机地址)

**场景**：题目名为 `easy_php`，提供了一份 PHP 源码附件，靶机地址为 `http://192.168.1.100:8080`。

**步骤 1：初始化题目**

```powershell
# 初始化题目，指定连接信息
python backend/cli.py init easy_php `
  --category web `
  --value 500 `
  --connection-info "http://192.168.1.100:8080" `
  --description "这就只是一个简单的 PHP 注入漏洞，你能找到 flag 吗？"
```

**步骤 2：添加附件**

将题目提供的源码文件 (如 `source.zip` 或 `index.php`) 放入生成的 `distfiles` 目录中。

目录结构应如下所示：
```text
easy_php/
├── metadata.yml      # 自动生成，包含题目信息
└── distfiles/        # 在此处放入附件
    └── index.php
```

**步骤 3：运行求解**

使用 `generic-openai` 模型 (配置在 .env 中) 进行离线求解。

```powershell
python backend/cli.py run `
  --challenge ./easy_php `
  --offline `
  --models generic-openai/gpt-4-turbo
```

---

### 示例 B：Misc 题目 (仅附件)

**场景**：题目名为 `hidden_pic`，仅提供了一张图片 `secret.png`，Flag 隐藏在图片中。

**步骤 1：初始化题目**

```powershell
# 初始化题目，连接信息留空 (默认为空)
python backend/cli.py init hidden_pic `
  --category misc `
  --value 100 `
  --description "这张图片看起来有点奇怪，flag 就在里面。"
```

**步骤 2：添加附件**

将 `secret.png` 放入 `distfiles` 目录。

目录结构：
```text
hidden_pic/
├── metadata.yml
└── distfiles/
    └── secret.png
```

**步骤 3：运行求解**

使用多个模型并行尝试 (例如 Claude 和 本地模型)。

```powershell
python backend/cli.py run `
  --challenge ./hidden_pic `
  --offline `
  --models claude-sdk/claude-3-5-sonnet-20241022 generic-openai/llama3
```

---

## 4. 常用参数说明

| 参数 | 说明 |
| :--- | :--- |
| `--offline` | **重要**：启用离线/手动模式。Agent 不会尝试连接 CTFd 平台，提交 Flag 时仅在本地打印。 |
| `--models` | 指定使用的模型。格式为 `provider/model-name`。支持 `claude-sdk`, `generic-openai`, `bedrock`, `azure` 等。 |
| `--no-submit` | 试运行模式。Agent 会执行所有步骤，但在最后一步找到 Flag 时不会执行提交动作 (即使在 Mock 模式下)。 |
| `--image` | 指定运行 Agent 的 Docker 镜像，默认为 `ctf-sandbox`。 |

## 5. Metadata.yml 高级配置

初始化后，您可以随时手动编辑 `metadata.yml` 来调整 Agent 的行为。

```yaml
name: easy_php
category: web
value: 500
description: 这就只是一个简单的 PHP 注入漏洞...
connection_info: http://host.docker.internal:8080  # 注意：如果是本机开启的服务，在 Docker 中请使用 host.docker.internal
hints:
  - content: "尝试关注一下 HTTP 头信息"
    cost: 0
tags:
  - php
  - injection
```
