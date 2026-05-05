# Ollama 操作手册

> 面向软件工程师的本地大模型运行工具
> 适用版本：0.x (2025-2026 最新版)

---

## 目录

1. [简介](#1-简介)
2. [安装](#2-安装)
3. [CLI 命令速查](#3-cli-命令速查)
4. [模型管理](#4-模型管理)
5. [运行与推理](#5-运行与推理)
6. [REST API](#6-rest-api)
7. [Modelfile 自定义模型](#7-modelfile-自定义模型)
8. [环境变量与配置](#8-环境变量与配置)
9. [实战场景](#9-实战场景)
10. [常见问题](#10-常见问题)

---

## 1. 简介

Ollama 是一个本地大模型运行工具，让你能在自己的机器上运行各种开源 LLM（Llama、Gemma、Mistral、Qwen 等），无需 GPU/CPU 集群，一条命令搞定。

**核心特性：**

- 一键下载 & 运行开源模型
- 内置 REST API（默认 `localhost:11434`）
- 支持 GGUF 量化模型，CPU/GPU 混合推理
- 支持 OpenAI 兼容 API 格式
- 自定义 Modelfile 创建专有模型
- 支持 Function Calling / Structured Output

---

## 2. 安装

### Linux 一键安装

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### 手动安装（x86_64）

```bash
curl -fsSL https://ollama.com/download/ollama-linux-amd64.tar.zst \
    | sudo tar x -C /usr
```

### 配置 systemd 服务（推荐）

创建 `ollama` 用户：

```bash
sudo useradd -r -s /bin/false -U -m -d /usr/share/ollama ollama
sudo usermod -a -G ollama $(whoami)
```

创建 `/etc/systemd/system/ollama.service`：

```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=$PATH"

[Install]
WantedBy=multi-user.target
```

启用服务：

```bash
sudo systemctl daemon-reload
sudo systemctl enable ollama
sudo systemctl start ollama
sudo systemctl status ollama
```

### macOS

```bash
curl -fsSL https://ollama.com/install.sh | sh
# 或下载官方 .dmg
```

### Windows

```powershell
irm https://ollama.com/install.ps1 | iex
# 或下载官方 OllamaSetup.exe
```

### Docker

```bash
docker run -d --gpus all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

### 验证安装

```bash
ollama -v
# 输出 ollama version is 0.x.x
curl http://localhost:11434
# 输出 Ollama is running
```

---

## 3. CLI 命令速查

| 命令 | 说明 | 示例 |
|------|------|------|
| `ollama serve` | 启动服务（daemon） | `ollama serve` |
| `ollama run <model>` | 运行模型（交互式 chat） | `ollama run gemma3` |
| `ollama pull <model>` | 下载模型 | `ollama pull qwen3:7b` |
| `ollama ls` | 列出已下载模型 | `ollama ls` |
| `ollama ps` | 列出当前运行的模型 | `ollama ps` |
| `ollama stop <model>` | 停止运行中的模型 | `ollama stop gemma3` |
| `ollama rm <model>` | 删除模型 | `ollama rm gemma3` |
| `ollama create <name> -f <Modelfile>` | 从 Modelfile 创建自定义模型 | `ollama create mymodel -f ./Modelfile` |
| `ollama show <model>` | 显示模型信息/Modelfile | `ollama show --modelfile llama3.2` |
| `ollama cp <src> <dst>` | 复制模型 | `ollama cp gemma3 my-gemma3` |
| `ollama push <model>` | 推送模型到仓库 | `ollama push username/model` |
| `ollama signin` | 登录 ollama.com | `ollama signin` |
| `ollama signout` | 登出 | `ollama signout` |
| `ollama launch` | 启动集成应用 | `ollama launch claude` |

---

## 4. 模型管理

### 搜索可用模型

打开 https://ollama.com/library 浏览所有模型。

**常用模型推荐：**

| 模型 | 大小 | 适用场景 |
|------|------|----------|
| `gemma3` | ~2B/12B/27B | Google 通用/多模态 |
| `qwen3` | ~0.5B/1.5B/7B/14B/32B | 阿里巴巴，中文/编码优秀 |
| `llama3.2` | 1B/3B | Meta，轻量通用 |
| `llama3.1` | 8B/70B/405B | Meta，强推理 |
| `mistral` | 7B | Mistral，高效 |
| `mixtral` | 8x7B | Mistral，MoE 高能 |
| `codellama` | 7B/13B/34B | Meta，代码专用 |
| `deepseek-coder` | 1.3B/6.7B/33B | 深度求索，代码 |
| `nomic-embed-text` | 137M | 文本嵌入/向量化 |
| `llava` | 7B/13B/34B | 多模态（视觉） |

### 下载模型

```bash
# 下载默认 tag（通常是最新版）
ollama pull gemma3

# 指定参数规模
ollama pull qwen3:7b
ollama pull llama3.2:3b

# 查看模型标签
ollama pull deepseek-coder:6.7b
```

### 查看已下载

```bash
ollama ls
# NAME              ID              SIZE      MODIFIED
# gemma3:latest     9f4e3e6c...     5.5 GB    2 hours ago
# qwen3:7b          4c93c9b9...     4.2 GB    1 hour ago
```

### 删除模型

```bash
ollama rm gemma3
ollama rm qwen3:7b
```

---

## 5. 运行与推理

### 交互式聊天

```bash
# 直接进入对话模式
ollama run gemma3

# 对话界面中可用命令：
# /?       帮助
# /exit    退出
# /set     设置参数（如 /set temperature 0.7）
# /show    显示模型信息
# /clear   清除上下文
# /save <name>  保存会话
# /load <name>  加载会话

# 多行输入用 """ 包裹
>>> """解释一下
... 依赖注入是什么
... """
```

### 单次问答（非交互）

```bash
# 管道输入
echo "Hello, who are you?" | ollama run gemma3

# 直接传参
ollama run gemma3 "Explain Kubernetes in one sentence"

# 带图片的多模态
ollama run gemma3 "这张图里有什么？" /path/to/image.jpg
ollama run gemma3 "What's in this image?" /Users/me/photo.png
```

### 生成 Embedding

```bash
ollama run nomic-embed-text "Hello world"
# 输出：JSON 数组

echo "将这段文字向量化" | ollama run nomic-embed-text
```

### 运行时参数控制

```bash
# 对话中动态调节
ollama run gemma3
>>> /set temperature 0.3  # 更确定性的回答
>>> /set num_ctx 8192     # 扩大上下文窗口

# 或创建自定义模型固化参数（见 Modelfile 章节）
```

---

## 6. REST API

Ollama 启动后，默认监听 `http://localhost:11434`，提供完整的 REST API。

### 基础信息

```bash
# 检查服务状态
curl http://localhost:11434

# 列出本地模型
curl http://localhost:11434/api/tags

# 查看运行中的模型
curl http://localhost:11434/api/ps
```

### `/api/generate` — 文本生成

```bash
# 流式输出（默认）
curl http://localhost:11434/api/generate -d '{
  "model": "gemma3",
  "prompt": "Why is the sky blue?"
}'

# 非流式输出
curl http://localhost:11434/api/generate -d '{
  "model": "gemma3",
  "prompt": "Why is the sky blue?",
  "stream": false
}'

# 带参数
curl http://localhost:11434/api/generate -d '{
  "model": "gemma3",
  "prompt": "Write a Python quicksort",
  "stream": false,
  "options": {
    "temperature": 0.2,
    "top_p": 0.9,
    "seed": 42
  }
}'

# 结构化输出（JSON Schema）
curl http://localhost:11434/api/generate -d '{
  "model": "gemma3",
  "prompt": "列出美中两国人口",
  "stream": false,
  "format": {
    "type": "object",
    "properties": {
      "countries": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "country": {"type": "string"},
            "population": {"type": "integer"}
          },
          "required": ["country", "population"]
        }
      }
    },
    "required": ["countries"]
  }
}'

# 带图片
curl http://localhost:11434/api/generate -d '{
  "model": "gemma3",
  "prompt": "What is in this picture?",
  "images": ["base64_encoded_image_data"]
}'

# 只加载/预载模型（不生成）
curl http://localhost:11434/api/generate -d '{"model": "gemma3"}'

# 卸载模型（释放内存）
curl http://localhost:11434/api/generate -d '{
  "model": "gemma3",
  "keep_alive": 0
}'
```

### `/api/chat` — 对话（ChatML 格式）

```bash
# 单轮对话
curl http://localhost:11434/api/chat -d '{
  "model": "gemma3",
  "messages": [
    {"role": "user", "content": "why is the sky blue?"}
  ],
  "stream": false
}'

# 多轮对话
curl http://localhost:11434/api/chat -d '{
  "model": "gemma3",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant. Answer in Chinese."},
    {"role": "user", "content": "什么是微服务？"},
    {"role": "assistant", "content": "微服务是一种架构风格..."},
    {"role": "user", "content": "它和单体架构比有什么优缺点？"}
  ],
  "stream": false
}'
```

### Function Calling（工具调用）

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "qwen3",
  "messages": [
    {"role": "user", "content": "What is the weather today in Paris?"}
  ],
  "stream": false,
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_current_weather",
        "description": "Get the current weather for a location",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The location to get weather for"
            },
            "format": {
              "type": "string",
              "enum": ["celsius", "fahrenheit"]
            }
          },
          "required": ["location", "format"]
        }
      }
    }
  ]
}'
```

### 其他 API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/tags` | GET | 列出本地模型 |
| `/api/ps` | GET | 列出运行中模型 |
| `/api/pull` | POST | 下载模型 |
| `/api/push` | POST | 上传模型 |
| `/api/delete` | DELETE | 删除模型 |
| `/api/copy` | POST | 复制模型 |
| `/api/create` | POST | 从 Modelfile 创建 |
| `/api/show` | POST | 获取模型信息 |
| `/api/embed` | POST | 生成 Embedding |
| `/api/embeddings` | POST | 旧版 Embedding |
| `/api/blobs/:digest` | POST | 上传 blob（用于创建模型） |
| `/api/version` | GET | 版本信息 |

### 编程语言调用

**Python:**

```python
pip install ollama
```

```python
from ollama import chat

response = chat(model='gemma3', messages=[
    {'role': 'user', 'content': 'Why is the sky blue?'}
])
print(response.message.content)
```

**JavaScript/TypeScript:**

```bash
npm i ollama
```

```javascript
import ollama from 'ollama';

const response = await ollama.chat({
    model: 'gemma3',
    messages: [{ role: 'user', content: 'Why is the sky blue?' }],
});
console.log(response.message.content);
```

**OpenAI 兼容 API:**

部分客户端可以通过 OpenAI 兼容接口连接 Ollama：

```bash
# base_url 指向 Ollama
# 例如 Python openai 库：
from openai import OpenAI
client = OpenAI(base_url='http://localhost:11434/v1', api_key='ollama')
response = client.chat.completions.create(
    model='gemma3',
    messages=[{'role': 'user', 'content': 'Hello'}]
)
```

---

## 7. Modelfile 自定义模型

### Modelfile 语法

```dockerfile
# 从已有模型派生
FROM gemma3

# 参数调整
PARAMETER temperature 0.7
PARAMETER num_ctx 4096
PARAMETER top_k 40
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.1
PARAMETER seed 42
PARAMETER stop "###"           # 自定义停止词

# 系统提示词
SYSTEM """你是一个资深软件架构师，精通分布式系统设计。
请用中文回答问题，保持专业但不失通俗。"""

# 对话模板（Go template 语法）
TEMPLATE """{{ if .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end }}{{ if .Prompt }}<|im_start|>user
{{ .Prompt }}<|im_end|>
{{ end }}<|im_start|>assistant
"""

# 示例对话（Few-shot）
MESSAGE user 用 Go 写一个 HTTP 服务器
MESSAGE assistant package main\n\nimport \"net/http\"\n\nfunc main() {\n\thttp.HandleFunc(\"/\", handler)\n\thttp.ListenAndServe(\":8080\", nil)\n}
MESSAGE user 加一个健康检查端点
MESSAGE assistant 好的，增加 /health 端点...
```

### 创建自定义模型

```bash
# 写入 Modelfile
cat > ~/Modelfile << 'EOF'
FROM qwen3:7b
PARAMETER temperature 0.3
SYSTEM """你是一个精通 Go 和 Rust 的代码审查助手。
逐行审查代码，指出性能问题和安全隐患。"""
EOF

# 创建自定义模型
ollama create code-reviewer -f ~/Modelfile

# 运行
ollama run code-reviewer

# 查看已生成模型的 Modelfile
ollama show --modelfile code-reviewer
```

### 从 GGUF 文件创建

```bash
# 下载 GGUF 文件后，Modelfile 中指定路径
FROM ./path/to/model.gguf
PARAMETER temperature 0.7
```

### 参数详解

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `num_ctx` | 2048 | 上下文窗口大小（token） |
| `temperature` | 0.8 | 温度；越高越有创造力 |
| `top_k` | 40 | 保留前 K 个候选 token |
| `top_p` | 0.9 | 核采样阈值 |
| `min_p` | 0.0 | 最小概率阈值（相对最可能 token） |
| `seed` | 0 | 随机种子；0=随机 |
| `repeat_penalty` | 1.1 | 重复惩罚系数 |
| `repeat_last_n` | 64 | 检测重复的窗口（-1=整个上下文） |
| `num_predict` | -1 | 最大生成 token 数（-1=不限） |
| `stop` | — | 停止序列（可设多个） |

---

## 8. 环境变量与配置

### 常用环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| `OLLAMA_HOST` | 绑定地址和端口 | `0.0.0.0:11434` |
| `OLLAMA_MODELS` | 模型存储路径 | `/data/ollama/models` |
| `OLLAMA_KEEP_ALIVE` | 模型在内存中保留时间 | `5m`（5分钟）或 `-1`（永不过期） |
| `OLLAMA_NUM_PARALLEL` | 并行处理请求数 | `2` |
| `OLLAMA_MAX_LOADED_MODELS` | 最大同时加载模型数 | `3` |
| `OLLAMA_DEBUG` | 调试模式 | `1` |
| `OLLAMA_GPU_OVERHEAD` | GPU 预留显存（字节） | `2000000000`（2GB） |
| `CUDA_VISIBLE_DEVICES` | 指定使用哪块 GPU | `0` 或 `0,1` |

### 配置示例

```bash
# 设置允许远程访问（注意安全！）
export OLLAMA_HOST=0.0.0.0:11434
ollama serve

# 持久化到 systemd
sudo systemctl edit ollama
# 添加：
# [Service]
# Environment="OLLAMA_HOST=0.0.0.0:11434"
# Environment="OLLAMA_NUM_PARALLEL=4"
```

### 修改 systemd 配置

```bash
sudo systemctl edit ollama
```

在弹出的编辑器中添加：

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_KEEP_ALIVE=-1"
Environment="OLLAMA_NUM_PARALLEL=2"
```

保存后重启：

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

---

## 9. 实战场景

### 场景 1：本地代码助手

```bash
# 下载 DeepSeek Coder（适合代码）
ollama pull deepseek-coder:6.7b

# 直接对话
ollama run deepseek-coder:6.7b

# 在 VSCode 中使用 Continue.dev 插件连接 Ollama
# 1. 安装 Continue 插件
# 2. 配置 ~/.continue/config.json:
#    {
#      "models": [{
#        "title": "Local DeepSeek",
#        "provider": "ollama",
#        "model": "deepseek-coder:6.7b"
#      }]
#    }
```

### 场景 2：批量推理脚本

```bash
#!/bin/bash
# batch-ask.sh
MODEL="qwen3:7b"
cat prompts.txt | while IFS= read -r question; do
    curl -s http://localhost:11434/api/generate -d "{
        \"model\": \"$MODEL\",
        \"prompt\": $(echo "$question" | jq -Rs .),
        \"stream\": false
    }" | jq -r '.response'
    echo "---"
done
```

### 场景 3：RAG 本地知识库

搭配向量数据库做本地 RAG 搜索：

```bash
# 1. 下载嵌入模型
ollama pull nomic-embed-text

# 2. Python 示例
cat << 'EOF' | python3
from ollama import embed

# 将文档转成向量
response = embed(model='nomic-embed-text', input=['文档内容...'])
vectors = response['embeddings']

# 查询
query_vec = embed(model='nomic-embed-text', input=['用户问题'])
# ... 用向量数据库搜索相似文档
EOF
```

### 场景 4：结构化数据提取

```bash
# 从非结构化文本中提取结构化 JSON
curl -s http://localhost:11434/api/chat -d '{
  "model": "qwen3:7b",
  "messages": [{"role": "user", "content": "从下面文本中提取人名、日期和金额：\
昨天张三和李四签订了合同，金额50万元，生效日期2025年3月1日。"}],
  "stream": false,
  "format": {
    "type": "object",
    "properties": {
      "party_a": {"type": "string"},
      "party_b": {"type": "string"},
      "amount": {"type": "number"},
      "effective_date": {"type": "string"}
    },
    "required": ["party_a", "party_b", "amount", "effective_date"]
  }
}' | jq '.message.content | fromjson'
```

### 场景 5：GPU vs CPU 管理

```bash
# 查看 GPU
nvidia-smi

# 指定某块 GPU
export CUDA_VISIBLE_DEVICES=1
ollama run gemma3

# 只看 CPU（不加载 GPU 驱动）
export OLLAMA_GPU_OVERHEAD=99999999999  # 预留所有显存给系统
ollama run llama3.2:3b
```

---

## 10. 常见问题

### Q: 模型下载太慢？

```bash
# 可以手动下载模型文件放到模型目录
# Linux 默认：~/.ollama/models/blobs/
# 但最简单的方式还是 ollama pull，会断点续传
```

### Q: 运行后命令行被占？

```bash
# 后台运行服务
ollama serve &

# 或作为 systemd 服务
sudo systemctl start ollama
```

### Q: 怎么释放显存/内存？

```bash
# 停止特定模型
ollama stop gemma3

# 或通过 API
curl http://localhost:11434/api/generate -d '{"model": "gemma3", "keep_alive": 0}'

# 完全退出 ollama serve（仅开发环境）
# 按 Ctrl+C 或 killall ollama
```

### Q: 我的模型很大、机器内存不够？

```bash
# 选择量化版本（带 q 标签的模型）
ollama pull llama3.1:8b-q4_0   # 4-bit 量化，约 4.5GB
ollama pull llama3.1:8b-q8_0   # 8-bit 量化，约 8GB
# 通常 q4_0 是在质量和大小之间最好的平衡
```

### Q: 模型存储路径在哪？

```bash
# Linux/macOS
ls ~/.ollama/models/blobs/
ls ~/.ollama/models/manifests/

# 自定义路径
export OLLAMA_MODELS=/data/ollama/models
```

### Q: 我需要远程访问怎么办？

```bash
# 注意：仅限内网或加认证，不要暴露到公网
export OLLAMA_HOST=0.0.0.0:11434
ollama serve

# 客户端设置 Ollama 地址
export OLLAMA_HOST=http://192.168.1.100:11434
```

### Q: 查看日志

```bash
# systemd 服务
journalctl -e -u ollama

# 前台启动
ollama serve
```

### Q: 更新 Ollama

```bash
# Linux
curl -fsSL https://ollama.com/install.sh | sh

# 指定版本
curl -fsSL https://ollama.com/install.sh | OLLAMA_VERSION=0.5.7 sh
```

### Q: 完全卸载

```bash
sudo systemctl stop ollama
sudo systemctl disable ollama
sudo rm /etc/systemd/system/ollama.service
sudo rm $(which ollama | tr 'bin' 'lib')  # 或直接删 /usr/lib/ollama
sudo rm $(which ollama)
sudo userdel ollama
sudo groupdel ollama
sudo rm -r /usr/share/ollama
# 警告：上面不会删除模型文件，手动删 ~/.ollama 如果需要
```

---

## 附录

### 快速安装检查清单

```bash
# 一行检查
ollama serve &>/dev/null & \
  sleep 2 && \
  curl -s http://localhost:11434 | grep -q "running" && \
  echo "✅ Ollama is running" || echo "❌ Ollama is not running"

# 下载并测试一个小模型
ollama pull llama3.2:1b
ollama run llama3.2:1b "say hello" --nowordwrap
```

### 推荐模型大小参考

| 硬件 | 推荐模型 |
|------|----------|
| 8GB RAM / 无 GPU | `llama3.2:1b`, `qwen3:0.5b`, `gemma3:2b` |
| 16GB RAM | `qwen3:7b` (q4), `gemma3:12b` (q4) |
| 32GB RAM | `llama3.1:8b`, `deepseek-coder:6.7b`, `qwen3:14b` (q4) |
| 64GB RAM + GPU | `qwen3:32b`, `mixtral:8x7b`, `llama3.1:70b` (q4) |

---

*本文档基于 [Ollama 官方文档](https://docs.ollama.com) 编写，最后更新 2026-04-30。*
