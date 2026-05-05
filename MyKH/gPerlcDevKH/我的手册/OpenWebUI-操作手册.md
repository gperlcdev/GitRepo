# Open WebUI 操作手册

> 面向软件工程师的自托管 AI 平台
> 版本：0.x (基于 Open WebUI v0.9.x)

---

## 目录

1. [简介](#1-简介)
2. [快速安装](#2-快速安装)
3. [连接模型提供商](#3-连接模型提供商)
4. [界面导航](#4-界面导航)
5. [核心功能](#5-核心功能)
6. [RAG 知识库](#6-rag-知识库)
7. [插件系统（Tools & Functions）](#7-插件系统tools--functions)
8. [Open Terminal 代码执行](#8-open-terminal-代码执行)
9. [管理与配置](#9-管理与配置)
10. [性能调优](#10-性能调优)
11. [环境变量参考](#11-环境变量参考)
12. [常见问题](#12-常见问题)

---

## 1. 简介

**Open WebUI** 是一个功能丰富的自托管 AI 平台，为 Ollama 和 OpenAI 兼容 API 提供精美、可扩展的 Web 界面。

**核心定位：** 替代 ChatGPT 网页版，运行在你的本地机器上。

**核心特性一览：**

- 🚀 **一键 Docker 部署**，支持 GPU 加速
- 🤝 **多模型提供商** — Ollama / OpenAI / Anthropic / Google / vLLM / Groq / 任意 OpenAI 兼容 API
- 📚 **RAG 知识库** — 上传文档、向量检索、智能问答
- 🛠️ **插件体系** — Tools（工具）、Functions（函数）、Pipelines（管道）
- 🐍 **Python 代码执行** — Open Terminal 在浏览器里运行代码
- 🔐 **多用户 + RBAC** — LDAP / OIDC / SSO / SCIM 企业级认证
- 🌐 **多语言** — 国际化支持
- 📱 **PWA 移动端** — 手机也能用

---

## 2. 快速安装

### 2.1 Docker 安装（推荐）

**基本命令：**

```bash
docker run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

打开 http://localhost:3000

> 首次访问需注册管理员账号。记住这个账号——只有第一个注册的账号会成为管理员。

**变体：**

| 场景 | 命令 |
|------|------|
| **Ollama 在同一台机器** | `docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main` |
| **Ollama 在另一台服务器** | `docker run -d -p 3000:8080 -e OLLAMA_BASE_URL=https://example.com -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main` |
| **NVIDIA GPU 支持** | `docker run -d -p 3000:8080 --gpus all --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:cuda` |
| **内置 Ollama（二合一）** | `docker run -d -p 3000:8080 --gpus=all -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama` |
| **单用户模式（免登录）** | `docker run -d -p 3000:8080 -e WEBUI_AUTH=False -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main` |
| **只用 OpenAI（无需 Ollama）** | `docker run -d -p 3000:8080 -e OPENAI_API_KEY=sk-xxx -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main` |
| **更改端口** | `docker run -d -p 8080:8080 --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 --name open-webui --restart always ghcr.io/open-webui/open-webui:main` |

> **关于 `--network=host` / `host.docker.internal`:**
> - Linux 上容器不能直接 `127.0.0.1` 访问宿主机
> - 用 `--add-host=host.docker.internal:host-gateway` 让 `host.docker.internal` 指向宿主机
> - 或用 `--network=host`（端口变成 8080 而非 3000）
> - macOS/Windows Docker Desktop 上 `host.docker.internal` 自动可用

### 2.2 pip 安装

```bash
# 要求 Python 3.11
pip install open-webui

# 启动
open-webui serve
# 访问 http://localhost:8080
```

### 2.3 Docker Compose

```yaml
# docker-compose.yml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    ports:
      - "3000:8080"
    volumes:
      - open-webui:/app/backend/data
    environment:
      - OLLAMA_BASE_URL=http://host.docker.internal:11434
    extra_hosts:
      - "host.docker.internal:host-gateway"
    restart: always

volumes:
  open-webui:
```

### 2.4 更新

```bash
# 法一：Watchtower
docker run --rm --volume /var/run/docker.sock:/var/run/docker.sock \
  nickfedor/watchtower --run-once open-webui

# 法二：手动
docker rm -f open-webui
docker pull ghcr.io/open-webui/open-webui:main
docker run -d -p 3000:8080 \
  -e WEBUI_SECRET_KEY="your-secret-key" \
  -v open-webui:/app/backend/data \
  --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:main
```

> ⚠️ **重要:** 设置 `WEBUI_SECRET_KEY` 避免重启后掉登录。
> 生成密钥: `openssl rand -hex 32`

### 2.5 镜像 Tags

| Tag | 说明 |
|-----|------|
| `:main` | 标准镜像（推荐） |
| `:main-slim` | 更小，Whisper/Embedding 首次使用时下载 |
| `:cuda` | NVIDIA GPU 支持（加 `--gpus all`） |
| `:ollama` | 内置 Ollama 一体包 |
| `:dev` | 最新开发版（不稳定，不可与生产共用 volume） |
| `v0.9.0` | 固定版本 |

两种 Registry：`ghcr.io/open-webui/open-webui` 或 `openwebui/open-webui`（Docker Hub），内容一致。

---

## 3. 连接模型提供商

### 3.1 连接 Ollama（本地）

前提：Ollama 已安装并运行在 `localhost:11434`。

- **自动发现：** 如果 Ollama 和 Open WebUI 在同一台机器，模型会自动出现在下拉菜单
- **手动设置：** Admin Panel > Settings > Connections > Ollama Base URL
  - Docker 内：`http://host.docker.internal:11434`
  - 独立部署：`http://localhost:11434`

### 3.2 连接 OpenAI / 第三方 API

Admin Panel > Settings > Connections > OpenAI API：

| 提供商 | URL | API Key |
|--------|-----|---------|
| OpenAI | `https://api.openai.com/v1` | 你的 sk-xxx |
| OpenRouter | `https://openrouter.ai/api/v1` | 你的 API Key |
| Groq | `https://api.groq.com/openai/v1` | 你的 API Key |
| Anthropic | 通过 OpenAI 兼容层 | 你的 API Key |
| 自托管 vLLM | `http://localhost:8000/v1` | (空) |

### 3.3 多提供商同时连接

Open WebUI 可以同时连接多个提供商，模型下拉菜单会合并显示所有可用模型。

### 3.4 OpenAI 兼容客户端模式

```python
# Python openai 库通过 Open WebUI 连接
from openai import OpenAI
client = OpenAI(
    base_url='http://localhost:3000/api/openai/v1',
    api_key='your-open-webui-api-key'
)
response = client.chat.completions.create(
    model='gemma3',
    messages=[{'role': 'user', 'content': 'Hello'}]
)
```

---

## 4. 界面导航

注册第一个账号 → 成为管理员。界面主要分区：

### 顶部（聊天界面）

- **模型选择器** — 下拉选择当前使用的模型
- **聊天控制** — 右侧面板：调整参数、切换模式
- `#` 快捷键 — 在输入框输入 `#` 快速引用知识库 / 文档 / URL

### 左侧侧边栏

| 图标 | 名称 | 说明 |
|------|------|------|
| 💬 | Chats | 聊天列表，支持文件夹/标签/搜索 |
| 📖 | Workspace | 工作区（模型/知识库/提示词/工具/笔记） |
| ⚙️ | Admin Panel | 管理面板（管理员可见） |

### 右侧聊天控制面板

- Model Parameters — temperature, top_p, max_tokens 等
- Function Calling — Default / Native 模式切换
- Tools — 启用/禁用当前对话的工具
- Attachments — 查看和管理附件

---

## 5. 核心功能

### 5.1 聊天的基本用法

- **普通问答：** 直接输入问题
- **多模型对比：** 创建对话后 + New Response → 选择不同模型，并排显示
- **文件上传：** 拖拽文件到输入框（文档、图片、代码）
- **Web 搜索：** 输入框输入 `#` + URL 或启用 Web Search 工具
- **语音输入：** 麦克风按钮（需配置 STT Provider）
- **消息队列：** AI 回复时你可以继续打字，消息自动排队

### 5.2 工作区 — 模型

Workspace > Models 中创建模型预设（每个预设 = 基础模型 + 系统提示词 + 参数 + 绑定工具/知识库）：

```text
例如创建一个 "代码审查助手" 预设：
- 基础模型: qwen3:7b
- 系统提示词: "你是专业的代码审查员..."
- 温度: 0.3
- 绑定工具: Code Interpreter
- 绑定知识库: 公司编码规范
```

支持动态变量：`{{ USER_NAME }}`、`{{ CURRENT_DATE }}` 自动注入。

### 5.3 工作区 — 提示词

创建可复用的提示词模板，支持 `/` 斜杠命令调用：

```text
/translate -> "请将以下内容翻译成英文..."
/summarize -> "请用50字以内总结以下内容..."
```

### 5.4 工作区 — 笔记

独立的富文本笔记空间，支持：
- Markdown / Rich Text 编辑
- AI 辅助改写（选中文本 → AI Enhance）
- 笔记作为对话上下文注入（不经过向量检索，全量注入）
- 模型可自动搜索、读取、更新笔记

### 5.5 频道（Channels）

团队协作空间，人类和 AI 模型在同一对话中互动：

```text
@summarizer 总结一下这个 PR
@code-reviewer 审查这段代码
```

支持线程、反应、权限控制、AI 跨频道搜索。

### 5.6 多模态

- **图片识别：** 发送图片 → 模型分析（需多模态模型支持）
- **图片生成：** 接入 DALL-E、ComfyUI、AUTOMATIC1111
- **语音通话：** 接入 Whisper (STT) + Azure/ElevenLabs (TTS)

### 5.7 任务管理

模型可在对话中维护结构化任务列表，支持多步骤工作流追踪。

---

## 6. RAG 知识库

### 6.1 基本流程

**方式一：临时文件（一次性）**

直接在聊天输入框拖入 PDF/Word/代码文件 → 自动 chunk + embed → 问答

**方式二：知识库（可复用）**

1. Workspace > Knowledge → "Create Knowledge"
2. 上传文档（PDF, Word, Markdown, TXT, CSV 等）
3. 选择 Chunk 大小等参数
4. 在聊天中用 `#` 引用知识库名称
5. 或绑定到模型预设，该模型始终自动使用该知识库

### 6.2 Chunk 配置

Admin Panel > Settings > Documents：

| 参数 | 说明 | 建议值 |
|------|------|--------|
| Chunk Size | 每块字符数 | 1000-2000 |
| Chunk Overlap | 相邻块重叠字符数 | 100-300 |
| Chunk Min Size Target | 合并过小碎片的目标大小 | 500-1000（可减少 90% chunk 数） |

默认启用 **Markdown Header Splitting** — 按标题结构切分，保持语义。

### 6.3 向量数据库

| 数据库 | 适用场景 |
|--------|----------|
| ChromaDB（默认） | 单人/少量文档；不支持多 worker |
| PGVector | 已有 PostgreSQL；多 worker 安全 |
| Milvus / Qdrant | 生产级扩展；多用户高并发 |
| Elasticsearch / OpenSearch | 已有 ES 生态 |

### 6.4 内容提取引擎

| 引擎 | 说明 |
|------|------|
| pypdf（默认） | 单用户可用，生产环境有内存泄漏 |
| **Tika** | 推荐，Java 后台服务 |
| **Docling** | IBM 的开源文档 AI 提取 |
| Azure Document Intelligence | 云端 OCR |
| Mistral OCR | Mistral AI 文档 OCR |

> **生产环境推荐：** Tika 或 Docling 替代默认 pypdf

### 6.5 Embedding 引擎

| 引擎 | 说明 |
|------|------|
| SentenceTransformers（默认） | 本地 CPU 运行，~500MB RAM/worker |
| Ollama | 用 `nomic-embed-text` 等模型 |
| OpenAI | `text-embedding-3-small/large` |

> **多用户场景：** 切换到外部 Embedding API

### 6.6 Web Search RAG

支持 15+ 搜索引擎：SearXNG、Google PSE、Brave、Kagi、Tavily、Perplexity、DuckDuckGo、Bing、Jina 等。

配置：Admin Panel > Settings > Web Search

### 6.7 Ollama 用户特别注意

Ollama 默认 context length = 2048 tokens，这对 RAG/Web Search 严重不足。**必须手动调大：**

Admin Panel > Settings > Models > 选择模型 → Advanced Parameters → `num_ctx` 设为 8192+

---

## 7. 插件系统（Tools & Functions）

### 7.1 体系概览

```
Tool Calling Mode 理解：
├── Default Mode（旧/兼容模式）
│   └── 在 system prompt 中以文本描述工具，解析模型输出
│   └── 兼容所有模型，但有性能损耗，不支持系统内置工具
│
└── Native Mode（推荐）
    └── 通过结构化 API 传递工具定义和调用
    └── 更快、支持 KV Cache、系统内置工具必需
    └── 所有主流模型均已支持（GPT/Claude/Gemini/Llama 3.1+/Qwen 2.5+）
```

**建议：** Admin Panel > Settings > Models → 全局设置 Function Calling = Native

### 7.2 内置系统工具

| 工具 | 说明 |
|------|------|
| Web Search | 联网搜索并返回结果 |
| Code Interpreter | 运行 Python 代码 |
| Image Generation | 生成/编辑图片 |
| Memory | 跨会话记住用户信息 |
| Notes | 读取/写入笔记 |
| Knowledge | 检索知识库 |

Admin Panel 中启用即可，无需额外安装。

### 7.3 Tools（工具）

Tools = 给模型调用的能力。模型在回复过程中自主决定何时调用。

**安装社区工具：** [openwebui.com/tools](https://openwebui.com/tools) → 点击 Get → 复制代码 → Workspace > Tools → 创建

**自己编写工具：**

```python
"""
title: Simple Calculator
author: You
version: 1.0
required_open_webui_version: 0.5.0
"""
import json

class Tools:
    def __init__(self):
        pass

    def calculate(self, expression: str) -> str:
        """计算数学表达式。
        :param expression: 数学表达式，如 "2 + 3 * 4"
        :return: 计算结果
        """
        try:
            result = eval(expression, {"__builtins__": {}}, {})
            return json.dumps({"result": result})
        except Exception as e:
            return json.dumps({"error": str(e)})
```

### 7.4 Functions（函数）

三种类型：

| 类型 | 作用 | 示例 |
|------|------|------|
| **Pipes** | 新增自定义模型 | 路由模型(简单问题用小模型)、多步 Agent 循环 |
| **Filters** | 拦截/修改请求/响应 | 上下文裁剪、PII 脱敏、Token 计数、Langfuse 追踪 |
| **Actions** | 消息底部按钮 | "回复翻译"、"发送到知识库" |

### 7.5 Pipelines 插件框架

独立的 Python 服务，通过 `pip install open-webui`，支持复杂业务逻辑：

```bash
# 启动 Pipelines 服务
pip install open-webui
open-webui pipelines --port 9099
```

然后在 Admin Panel > Connections 中设置 Pipelines URL。

### 7.6 MCP（Model Context Protocol）

通过 Streamable HTTP 连接外部 MCP 服务器，自动发现可用的工具：

Admin Panel > Settings > Tools → 添加 MCP Server URL

### 7.7 社区资源

**[openwebui.com](https://openwebui.com/)** — 数千个社区插件的集中仓库。

> 想做某功能但不知道怎么做？先去社区搜一下，90% 的概率已经有人写了。

---

## 8. Open Terminal 代码执行

### 8.1 功能介绍

Open Terminal = 在聊天中给 AI 一个 **真实的计算环境**。AI 可以写代码、执行、读取输出、修复错误、反复迭代——全部自动完成。

**工作要求：**

- 安装 Docker（Open Terminal 默认在隔离容器中运行）
- 或裸机模式（不安全，仅开发环境）

### 8.2 启用

Admin Panel > Settings > Open Terminal → 启用

### 8.3 使用方式

在聊天中描述任务，模型会自动：

1. 编写代码
2. 在终端执行
3. 读取输出
4. 根据结果调试
5. 返回最终结果

支持文件浏览、上传、下载，甚至可以在浏览器里预览 Web 项目。

---

## 9. 管理与配置

### 9.1 用户与认证

| 功能 | 位置 | 说明 |
|------|------|------|
| RBAC | Admin > Users | 管理员/用户/自定义角色 |
| SSO/OIDC | Admin > Settings > Auth | Google/GitHub/Microsoft 等 OAuth |
| LDAP | Admin > Settings > Auth | 企业 LDAP/AD 集成 |
| SCIM 2.0 | Admin > Settings > Auth | 自动用户同步（Okta/Azure AD 等） |
| API Keys | Settings > Account | 个人 API Key 用于程序调用 |

### 9.2 数据库

**默认 SQLite（单用户/轻量使用）**

```bash
# 数据位置：Docker volume
docker volume inspect open-webui
# mount point = /var/lib/docker/volumes/open-webui/_data/
```

**PostgreSQL（生产/多用户）**

```bash
export DATABASE_URL=postgres://user:password@localhost:5432/webui
```

### 9.3 存储后端

```bash
# S3 兼容
export STORAGE_PROVIDER=s3
export S3_ACCESS_KEY_ID=xxx
export S3_SECRET_ACCESS_KEY=xxx
export S3_BUCKET_NAME=my-webui-files
export S3_ENDPOINT_URL=https://s3.amazonaws.com
# GCS / Azure Blob 类似
```

### 9.4 OpenTelemetry

```bash
export OTEL_SDK_DISABLED=false
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
export OTEL_SERVICE_NAME=open-webui
```

### 9.5 横向扩展（多 worker / 多节点）

```bash
# 需要 Redis + PostgreSQL + 外部向量数据库
export UVICORN_WORKERS=4
export WEBUI_SECRET_KEY=xxx
export DATABASE_URL=postgres://...
export REDIS_URL=redis://...
```

---

## 10. 性能调优

### 10.1 设置专用 Task Model（最高收益）

默认情况下，UI 后台任务（生成标题、标签、自动补全、推荐问题）使用当前聊天模型。**这是最常见的性能杀手：**

- 用 GPT-4/o1 写 "Chat about Python" 这种标题 → 浪费钱和延迟
- 本地大模型做自动补全 → 卡顿

**配置：** Admin Panel > Settings > Interface

| 字段 | 推荐值 |
|------|--------|
| Task Model (External) | `gpt-5-nano` / `gemini-2.5-flash-lite` / `llama-3.1-8b-instant` |
| Task Model (Local) | `qwen3:1b` / `gemma3:1b` / `llama3.2:3b` |

如需禁用某些后台任务：

| 任务 | 环境变量 | 管理面板开关 |
|------|----------|-------------|
| 自动补全 | `ENABLE_AUTOCOMPLETE_GENERATION=False` | Interface > Autocomplete Generation |
| 后续问题 | `ENABLE_FOLLOW_UP_GENERATION=False` | Interface > Follow-up Generation |
| 标题生成 | `ENABLE_TITLE_GENERATION=False` | Interface > Title Generation |
| 标签生成 | `ENABLE_TAGS_GENERATION=False` | Interface > Tags Generation |

### 10.2 模型缓存

```bash
# 缓存模型列表（对 OpenRouter 等大量模型提供商特别重要）
export ENABLE_BASE_MODELS_CACHE=True
export MODELS_CACHE_TTL=300
```

### 10.3 RAG 性能优化

```bash
# KV Cache 优化：RAG 上下文放入 system message 而非 user message
# 利用 Ollama/OpenAI 的 prompt caching，后续问题几乎即时
export RAG_SYSTEM_CONTEXT=True
```

### 10.4 数据库

```bash
# SQLite 低配机器注意：
export DATABASE_ENABLE_SESSION_SHARING=False  # 默认，低配保持关闭

# PostgreSQL 高并发：
export DATABASE_POOL_SIZE=15
export DATABASE_POOL_MAX_OVERFLOW=20
```

### 10.5 内容提取引擎

生产环境必改：`pypdf` 有内存泄漏。

```bash
# 切换到 Tika（需要额外部署 Tika 服务）
export CONTENT_EXTRACTION_ENGINE=tika
```

---

## 11. 环境变量参考

### 核心

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `OLlAMA_BASE_URL` | `http://host.docker.internal:11434` | Ollama 服务地址 |
| `OPENAI_API_KEY` | — | OpenAI API 密钥 |
| `WEBUI_SECRET_KEY` | — | 会话加密密钥（必设，防重启掉登录） |
| `WEBUI_AUTH` | `True` | 是否开启登录认证 |
| `DATABASE_URL` | `sqlite:///data/webui.db` | 数据库连接 |
| `STORAGE_PROVIDER` | `local` | 存储后端（local/s3/gcs/azure） |

### RAG

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `RAG_EMBEDDING_ENGINE` | `sentence-transformers` | Embedding 引擎 |
| `RAG_TOP_K` | `5` | 检索 TOP K |
| `RAG_SYSTEM_CONTEXT` | `False` | RAG 注入到 system message |
| `CONTENT_EXTRACTION_ENGINE` | — | 文档提取引擎 |

### 性能

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `UVICORN_WORKERS` | `1` | 工作进程数 |
| `ENABLE_BASE_MODELS_CACHE` | `False` | 缓存模型列表 |
| `ENABLE_AUTOCOMPLETE_GENERATION` | `True` | 启用自动补全 |
| `ENABLE_TITLE_GENERATION` | `True` | 启用自动标题 |
| `ENABLE_REALTIME_CHAT_SAVE` | `False` | 实时保存（不建议开启） |

### 认证

| 变量 | 说明 |
|------|------|
| `OPENID_PROVIDER_URL` | OIDC 提供商 URL |
| `OAUTH_CLIENT_ID` | OAuth Client ID |
| `OAUTH_CLIENT_SECRET` | OAuth Client Secret |
| `ENABLE_LDAP` | LDAP 认证开关 |
| `LDAP_SERVER_HOST` | LDAP 服务器地址 |
| `ENABLE_SCIM` | SCIM 2.0 开关 |

---

## 12. 常见问题

### Q: 首次登录注册后怎么没看到模型？

先去 Admin Panel > Settings > Connections 检查 Ollama / OpenAI 连接是否正确。Ollama 需要先 `ollama pull xxx` 下载模型。

### Q: 为什么聊天输入 `#` 没有反应？

`#` 快捷方式需要：
- 已上传文档到 Knowledge（Workspace > Knowledge）
- 或输入 `#` 后跟 URL

如果完全无响应，检查浏览器控制台是否有错误。

### Q: "The prompt is too long" 错误

模型上下文窗口满了。解决方案：

1. 安装上下文裁剪 Filter（社区有现成插件）
2. 使用更大的模型（128K+ 上下文）
3. 减少附件大小或数量

### Q: Web Search 不工作 / RAG 效果差

1. **Ollama 用户** — 把 `num_ctx` 从 2048 调到 8192+（Admin > Settings > Models）
2. **检查 Web Search API Key** — 各搜索提供商需要配置 API Key
3. **Chunk Size 太小** — 调到 1000-2000

### Q: 页面加载慢 / 模型列表不显示

```bash
# 启用模型缓存（特别对 OpenRouter 有效）
export ENABLE_BASE_MODELS_CACHE=True
```

### Q: Docker 容器连不上宿主机 Ollama

Linux 上必须：
```bash
# 方法一：--network=host（端口变为 8080）
docker run --network=host -e OLLAMA_BASE_URL=http://127.0.0.1:11434 ...

# 方法二：host.docker.internal
docker run --add-host=host.docker.internal:host-gateway -e OLLAMA_BASE_URL=http://host.docker.internal:11434 ...
```

### Q: GPU 用不了？

```bash
# 1. 安装 NVIDIA Container Toolkit
# 2. 使用 :cuda 镜像
# 3. 加 --gpus all 参数
docker run -d --gpus all ... ghcr.io/open-webui/open-webui:cuda
```

### Q: 内存占用不断增长？

1. 切换到 Tika 替代 pypdf（pypdf 有内存泄漏）
2. 减少 Uvicorn Workers 数量
3. 禁用不必要的后台任务（自动补全等）

### Q: 更新后登录失效 / 数据丢失？

- **设置 `WEBUI_SECRET_KEY`** 并在更新时保持一致
- **不要** dev 和 stable 共用同一个 volume
- 更新前备份：`docker exec open-webui tar czf /tmp/backup.tar.gz /app/backend/data`

### Q: 单用户 vs 多用户？

- 第一次注册的账号自动成为管理员
- `WEBUI_AUTH=False` → 单用户免登录模式（不可逆转）
- 默认 SQLite 适合单人；多用户用 PostgreSQL

### Q: 完全卸载

```bash
docker rm -f open-webui
docker rmi ghcr.io/open-webui/open-webui:main
docker volume rm open-webui  # 删除所有数据！
```

---

## 附录：快速安装检查

```bash
# 一行启动 + Ollama 同机
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  -e WEBUI_SECRET_KEY="$(openssl rand -hex 32)" \
  --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:main

# 检查是否启动
curl -s http://localhost:3000/health 2>/dev/null || echo "等待启动..."
docker logs open-webui --tail 20
```

---

*本文档基于 [Open WebUI 官方文档](https://docs.openwebui.com/) 编写，最后更新 2026-04-30。*
