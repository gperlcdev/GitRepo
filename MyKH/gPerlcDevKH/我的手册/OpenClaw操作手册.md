# 🦞 OpenClaw 操作手册

> 版本：基于 2026.1.4+ | 最后更新：2026-04-27

---

## 目录

1. [概述](#1-概述)
2. [安装与部署](#2-安装与部署)
3. [CLI 命令参考](#3-cli-命令参考)
4. [配置文件详解](#4-配置文件详解)
5. [渠道连接](#5-渠道连接)
6. [Gateway 网关管理](#6-gateway-网关管理)
7. [多智能体路由](#7-多智能体路由)
8. [会话与记忆管理](#8-会话与记忆管理)
9. [Skills 技能系统](#9-skills-技能系统)
10. [定时任务与自动化](#10-定时任务与自动化)
11. [安全与权限](#11-安全与权限)
12. [节点设备管理](#12-节点设备管理)
13. [模型与提供商配置](#13-模型与提供商配置)
14. [浏览器工具](#14-浏览器工具)
15. [故障排除](#15-故障排除)

---

## 1. 概述

**OpenClaw** 是一个开源的 AI 智能体网关（Gateway），可在任何操作系统上运行。它通过单一网关进程连接聊天应用与 AI 助手，支持多渠道消息收发、多智能体路由和丰富的工具生态系统。

### 核心架构

```
聊天应用 + 插件 ──→ Gateway ──→ Pi 智能体
                    │
                    ├── CLI 命令行
                    ├── Web 控制界面
                    ├── macOS 应用
                    └── iOS/Android 节点
```

### 核心功能

| 功能 | 说明 |
|------|------|
| 多渠道网关 | 通过单进程连接 WhatsApp、Telegram、Discord、iMessage 等 |
| 插件渠道 | 通过扩展包添加 Mattermost 等更多渠道 |
| 多智能体路由 | 按智能体、工作区或发送者隔离会话 |
| 媒体支持 | 发送和接收图片、音频和文档 |
| Web 控制界面 | 浏览器仪表板管理聊天、配置和节点 |
| 移动节点 | 配对 iOS/Android 节点支持 Canvas |
| 定时任务 | Cron 定时调度和自动化 |
| Docker 沙箱 | 为非主会话提供隔离运行环境 |

---

## 2. 安装与部署

### 2.1 前置条件

- **Node.js >= 22**
- `pnpm`（可选，推荐用于源码构建）
- **推荐：** Brave Search API 密钥（用于网络搜索，`openclaw configure --section web` 设置）

### 2.2 安装 CLI

**Linux/macOS（推荐）：**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows（PowerShell）：**
```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

**npm 全局安装：**
```bash
npm install -g openclaw@latest
```

**pnpm 全局安装：**
```bash
pnpm add -g openclaw@latest
```

### 2.3 新手引导向导

```bash
openclaw onboard --install-daemon
```

向导会引导你完成：
- 选择 **本地/远程 Gateway** 模式
- 配置 **认证方式**（OpenAI Codex OAuth、API 密钥等）
- 设置 **渠道连接**（WhatsApp/Telegram/Discord）
- 安装 **后台服务**（launchd/systemd）
- 生成 **Gateway 令牌**

### 2.4 启动 Gateway

```bash
openclaw gateway --port 18789 --verbose
```

验证运行状态：
```bash
openclaw status
openclaw health
openclaw security audit --deep
```

### 2.5 网络控制界面

Gateway 启动后打开浏览器：
- **本地默认地址：** `http://127.0.0.1:18789/`
- 使用 `openclaw dashboard` 快速打开

---

## 3. CLI 命令参考

### 3.1 全局标志

| 标志 | 说明 |
|------|------|
| `--dev` | 隔离状态到 `~/.openclaw-dev` |
| `--profile <name>` | 隔离状态到 `~/.openclaw-<name>` |
| `--no-color` | 禁用 ANSI 颜色（支持 `NO_COLOR=1`） |
| `--json` | JSON 格式输出 |
| `-V, --version` | 显示版本 |

### 3.2 命令树

```
openclaw [--dev] [--profile <name>] <command>
  onboard        交互式新手引导向导
  configure      配置向导
  config         get/set/unset 配置值
  doctor         健康检查和修复
  gateway        启动/管理 Gateway 网关
  status         显示运行状态
  health         获取健康状态
  logs           跟踪日志
  message        发送消息和执行渠道操作
  agent          运行智能体轮次
  agents         管理多智能体
  channels       管理渠道账户
  pairing        管理私信配对
  skills         管理技能
  plugins        管理插件
  sessions       列出会话
  cron           管理定时任务
  models         管理模型配置
  memory         管理记忆存储
  nodes/node     管理移动节点
  browser        控制浏览器
  security       安全审计
  system         系统事件管理
  reset          重置配置/状态
  update         更新 OpenClaw
  uninstall      卸载
  tui            终端 UI
  hooks          管理钩子
  webhooks       管理 webhooks
  dns            DNS 工具
  docs           搜索文档
```

### 3.3 常用命令速查

```bash
# 状态检查
openclaw status                     # 概览状态
openclaw status --all               # 完整诊断报告
openclaw status --deep              # 探测渠道健康
openclaw models status              # 模型/认证状态
openclaw health                     # 健康探测
openclaw channels status --probe    # 渠道健康探测

# 配置管理
openclaw config get gateway.port    # 读取配置
openclaw config set agents.defaults.model.primary "anthropic/claude-sonnet-4-5"
openclaw config unset channels.whatsapp.allowFrom

# 渠道操作
openclaw channels list              # 列出渠道
openclaw channels login             # WhatsApp 扫码登录
openclaw channels logout            # 登出渠道

# 配对管理
openclaw pairing list whatsapp      # 列出待审批的配对
openclaw pairing approve whatsapp <code>  # 批准配对

# 消息发送
openclaw message send --target +15555550123 --message "你好"
openclaw message send --channel telegram --target tg:123456789 --message "你好"

# 定时任务
openclaw cron list                  # 列出所有任务
openclaw cron add --name "daily-summary" --every 24h --system-event "生成日报"
openclaw cron run <id>              # 立即执行任务

# 智能体管理
openclaw agents list                # 列出智能体
openclaw agents add "work" --workspace "~/.openclaw/workspace-work"

# 安全审计
openclaw security audit             # 安全审计
openclaw security audit --deep      # 深度审计（含实时探测）
openclaw security audit --fix       # 自动修复安全配置
```

---

## 4. 配置文件详解

配置文件位于 `~/.openclaw/openclaw.json`，使用 JSON5 格式（支持注释和尾逗号）。

### 4.1 最小配置（推荐起点）

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

### 4.2 常用配置段

#### `agents` — 智能体设置

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5.2"],
      },
      imageModel: {
        primary: "openai/gpt-5.2",
      },
      timeoutSeconds: 600,
      maxConcurrent: 3,
    },
  },
}
```

#### `gateway` — 网关设置

```json5
{
  gateway: {
    mode: "local",
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token",
      token: "your-token",
    },
  },
}
```

- `mode`：`local` | `remote`
- `bind`：`loopback`（仅本地）| `lan`（局域网）| `tailnet`（Tailscale）
- `auth.mode`：`token` | `password`

#### `channels` — 渠道设置

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
    telegram: {
      enabled: true,
      botToken: "123456:ABC...",
      dmPolicy: "pairing",
    },
    discord: {
      enabled: true,
      token: "your-bot-token",
    },
  },
}
```

#### `messages` — 消息行为

```json5
{
  messages: {
    responsePrefix: "🦞",
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
    },
    inbound: {
      debounceMs: 2000,
    },
  },
}
```

#### `commands` — 聊天命令

```json5
{
  commands: {
    native: "auto",
    text: true,
    bash: false,
    config: false,
    debug: false,
    restart: false,
  },
}
```

#### `session` — 会话管理

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60,
    },
  },
}
```

#### `logging` — 日志设置

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
  },
}
```

### 4.3 配置包含（`$include`）

支持将配置拆分为多个文件：

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
}
```

支持嵌套包含（最多 10 层深度）、相对路径和绝对路径。

### 4.4 环境变量替换

配置中支持 `${VAR_NAME}` 替换环境变量：

```json5
{
  models: {
    providers: {
      "custom": {
        apiKey: "${MY_API_KEY}",
        baseUrl: "${API_BASE_URL}/v1",
      },
    },
  },
}
```

---

## 5. 渠道连接

### 5.1 WhatsApp

```bash
openclaw channels login
```
然后通过 WhatsApp → 设置 → 链接设备扫描二维码。

**配置：**
```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",      // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
      sendReadReceipts: true,    // 是否显示已读（蓝色双勾）
      textChunkLimit: 4000,      // 出站分块大小
      mediaMaxMb: 50,            // 入站媒体上限
    },
  },
}
```

**多账号：**
```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {},
      },
    },
  },
}
```

### 5.2 Telegram

**配置：**
```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123456:ABC...",
      dmPolicy: "pairing",
      groups: {
        "*": { requireMention: true },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
      ],
      historyLimit: 50,
      replyToMode: "first",     // off | first | all
      streamMode: "partial",    // off | partial | block
      mediaMaxMb: 5,
    },
  },
}
```

创建 Telegram 机器人：通过 [@BotFather](https://t.me/BotFather) 创建并获取 token。

**多账号：**
```json5
{
  channels: {
    telegram: {
      accounts: {
        default: { botToken: "123:ABC" },
        alerts: { botToken: "456:DEF", name: "Alerts Bot" },
      },
    },
  },
}
```

### 5.3 Discord

**配置：**
```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      dm: {
        policy: "pairing",
        allowFrom: ["1234567890"],
      },
      guilds: {
        "123456789012345678": {
          requireMention: false,
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
      historyLimit: 20,
      mediaMaxMb: 8,
    },
  },
}
```

### 5.4 iMessage（macOS）

需要本地 imsg CLI 工具和消息数据库权限。

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123"],
      historyLimit: 50,
    },
  },
}
```

远程 iMessage：使用 SSH 包装器——`clichannels.imessage.cliPath` 指向 SSH 脚本，`remoteHost` 设置远程主机。

### 5.5 Signal

需要 [signal-cli](https://github.com/AsamK/signal-cli) 运行环境中。

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own",
      historyLimit: 50,
    },
  },
}
```

### 5.6 Slack（Socket Mode）

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: { policy: "pairing", allowFrom: ["U123"] },
      channels: {
        C123: { allow: true, requireMention: true },
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
      },
    },
  },
}
```

### 5.7 Google Chat

使用服务账号认证，需要 Google Cloud 项目配置。

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      webhookPath: "/googlechat",
    },
  },
}
```

### 5.8 Mattermost（插件）

需要先安装插件：
```bash
openclaw plugins install @openclaw/mattermost
```

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

---

## 6. Gateway 网关管理

### 6.1 启动与停止

```bash
# 前台运行
openclaw gateway --port 18789 --verbose

# 安装为系统服务
openclaw gateway install

# 服务管理
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway status
```

### 6.2 高级启动选项

```bash
# 绑定方式
--bind loopback   # 仅本地（默认）
--bind lan        # 局域网可访问
--bind tailnet    # 通过 Tailscale
--bind auto       # 自动选择

# 认证
--auth token          # 令牌认证
--auth password       # 密码认证
--token <token>
--password <password>

# Tailscale
--tailscale serve    # Tailscale Serve
--tailscale funnel   # 公网暴露

# 其他
--force              # 强制终止端口监听器
--dev                # 开发模式隔离
--reset              # 重置开发配置
```

### 6.3 远程访问

**SSH 隧道：**
```bash
ssh -L 18789:127.0.0.1:18789 user@gateway-host
```

**Tailscale：**
```json5
{
  gateway: {
    tailscale: { mode: "serve" },  // off | serve | funnel
  },
}
```

### 6.4 Gateway RPC 调用

通过 `openclaw gateway call` 可调用内部 RPC 方法：

```bash
# 获取配置
openclaw gateway call config.get --params '{}'

# 应用配置（验证+写入+重启）
openclaw gateway call config.apply --params '{
  "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
  "baseHash": "<hash>",
  "restartDelayMs": 1000
}'

# 部分更新
openclaw gateway call config.patch --params '{
  "raw": "{ channels: { telegram: { enabled: true } } }",
  "baseHash": "<hash>"
}'
```

### 6.5 日志查看

```bash
openclaw logs                   # 实时日志
openclaw logs --follow          # 跟随模式
openclaw logs --limit 200       # 最近 200 行
openclaw logs --json            # JSON 格式
openclaw channels logs          # 渠道日志
```

### 6.6 健康探测

```bash
openclaw health                 # 基本健康检查
openclaw health --verbose       # 详细信息
openclaw status --deep          # 深度探测
openclaw status --all           # 完整诊断（可粘贴）
openclaw doctor                 # 配置健康检查+修复建议
openclaw doctor --fix           # 自动修复
```

---

## 7. 多智能体路由

### 7.1 配置多个智能体

同一个 Gateway 中可以运行多个隔离的智能体，各有独立的工作区、认证和会话。

```json5
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      sandbox: { mode: "non-main", scope: "session" },
    },
    list: [
      {
        id: "home",
        default: true,
        workspace: "~/.openclaw/workspace-home",
        identity: {
          name: "Clawd",
          emoji: "🦞",
        },
      },
      {
        id: "work",
        workspace: "~/.openclaw/workspace-work",
        identity: {
          name: "WorkBot",
          emoji: "💼",
        },
      },
    ],
  },
}
```

### 7.2 配置绑定规则

```json5
{
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
    { agentId: "work", match: { channel: "telegram" } },
  ],
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      },
    },
  },
}
```

匹配优先级（从高到低）：
1. `match.peer`（精确对等方）
2. `match.guildId` / `match.teamId`
3. `match.accountId`（精确账号）
4. `match.accountId: "*"`（渠道范围）
5. 默认智能体

### 7.3 每智能体安全配置

```json5
{
  agents: {
    list: [
      // 完全访问（个人）
      {
        id: "personal",
        sandbox: { mode: "off" },
      },
      // 只读访问（家庭）
      {
        id: "family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
        tools: {
          allow: ["read", "sessions_list", "sessions_history"],
          deny: ["write", "edit", "exec", "browser"],
        },
      },
      // 仅消息（公共）
      {
        id: "public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
        tools: {
          profile: "messaging",
          deny: ["read", "write", "edit", "exec", "browser"],
        },
      },
    ],
  },
}
```

### 7.4 工具组简写

| 简写 | 包含的工具 |
|------|-----------|
| `group:runtime` | exec, process |
| `group:fs` | read, write, edit, apply_patch |
| `group:sessions` | sessions_list, sessions_history, sessions_send, sessions_spawn, session_status |
| `group:memory` | memory_search, memory_get |
| `group:web` | web_search, web_fetch |
| `group:ui` | browser, canvas |
| `group:automation` | cron, gateway |
| `group:messaging` | message |
| `group:openclaw` | 所有内置工具 |

---

## 8. 会话与记忆管理

### 8.1 会话作用域

```json5
{
  session: {
    scope: "per-sender",           // 一般作用域
    dmScope: "main",               // 私聊作用域
    // main: 所有私聊共享主会话
    // per-peer: 按发送者 ID 隔离
    // per-channel-peer: 按渠道+发送者隔离
    // per-account-channel-peer: 按账号+渠道+发送者隔离
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321"],
    },
  },
}
```

### 8.2 会话重置策略

```json5
{
  session: {
    reset: {
      mode: "daily",               // daily | idle
      atHour: 4,                   // 凌晨 4 点重置
      idleMinutes: 60,             // 60 分钟空闲重置
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      dm: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
  },
}
```

### 8.3 记忆管理

记忆文件存储在智能体工作区的 `memory/` 目录。

```bash
# CLI 命令
openclaw memory status              # 查看记忆索引状态
openclaw memory index               # 重新索引
openclaw memory search "<query>"    # 语义搜索记忆
```

**记忆文件结构：**
- `MEMORY.md` — 长期记忆（经过提炼的重要信息）
- `memory/YYYY-MM-DD.md` — 每日原始日志
- `memory/heartbeat-state.json` — 心跳检查状态

### 8.4 上下文压缩（Compaction）

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",         // default | safeguard | off
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
        },
      },
    },
  },
}
```

### 8.5 上下文裁剪（Context Pruning）

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",          // adaptive | aggressive | off
        keepLastAssistants: 3,
      },
    },
  },
}
```

---

## 9. Skills 技能系统

### 9.1 技能管理

```bash
openclaw skills list               # 列出所有技能
openclaw skills list --eligible    # 仅显示可用的
openclaw skills info <name>        # 查看技能详情
openclaw skills check              # 技能就绪状态
```

### 9.2 技能配置

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm",
    },
    entries: {
      "skill-name": {
        enabled: true,
        env: {
          API_KEY: "your-key",
        },
      },
    },
  },
}
```

### 9.3 查找技能

访问 [ClawHub](https://clawhub.com) 或在终端搜索：
```bash
npx clawhub search <query>
npx clawhub install <skill-name>
```

---

## 10. 定时任务与自动化

### 10.1 Cron 任务

**CLI 管理：**
```bash
openclaw cron list                          # 列出任务
openclaw cron add --name "check-email" \
  --every 30m \
  --system-event "检查新邮件"
openclaw cron add --name "morning-news" \
  --at "08:00" \
  --message "今天的新闻摘要" \
  --to "telegram:123456789"
openclaw cron edit <id> --every 1h          # 修改
openclaw cron run <id>                      # 立即执行
openclaw cron rm <id>                       # 删除
openclaw cron enable|disable <id>           # 启用/禁用
```

### 10.2 心跳（Heartbeat）

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",          // 间隔（支持 ms/s/m/h）
        model: "anthropic/claude-sonnet-4-5",  // 可选覆盖模型
        target: "last",        // 投递到最近聊天的渠道
        prompt: "Read HEARTBEAT.md...",  // 覆盖心跳提示
        ackMaxChars: 300,
      },
    },
  },
}
```

心跳默认提示：读取 `HEARTBEAT.md`，无任务则回复 `HEARTBEAT_OK`。

### 10.3 Cron 与 Heartbeat 选择

| 场景 | 使用 |
|------|------|
| 固定时间执行（如每天早上 9 点） | Cron |
| 周期性检查（如每 30 分钟检查邮件） | Heartbeat 或 Cron |
| 需要精确计时 | Cron |
| 需要批量检查（减少 API 调用） | Heartbeat |
| 一次性提醒 | Cron（at 模式） |
| 需要隔离执行（独立上下文） | Cron（isolated 模式） |

### 10.4 钩子（Hooks）

```bash
openclaw hooks list              # 列出钩子
openclaw hooks install <name>    # 安装钩子
openclaw hooks enable <id>       # 启用
openclaw hooks disable <id>      # 禁用
```

### 10.5 Webhook（Gmail）

```bash
openclaw webhooks gmail setup --account user@gmail.com
openclaw webhooks gmail run
```

---

## 11. 安全与权限

### 11.1 安全审计

```bash
openclaw security audit              # 标准审计
openclaw security audit --deep       # 深度审计（含实时探测）
openclaw security audit --fix        # 自动修复安全配置
```

### 11.2 私信配对策略

每个渠道可设置私信处理策略：

| 策略 | 说明 |
|------|------|
| `pairing` | 未知发送者收到配对码，需手动批准 |
| `allowlist` | 仅白名单中的发送者可通信 |
| `open` | 允许所有入站私聊（需 `allowFrom: ["*"]`） |
| `disabled` | 忽略所有入站私聊 |

```bash
openclaw pairing list whatsapp          # 查看待审批
openclaw pairing approve whatsapp <code>   # 批准
```

### 11.3 访问控制

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

### 11.4 提升执行（Elevated）

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete"],
      },
    },
  },
}
```

### 11.5 Docker 沙箱

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",          // off | non-main | all
        scope: "session",          // session | agent | shared
        workspaceAccess: "none",   // none | ro | rw
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          network: "none",         // 网络隔离
          readOnlyRoot: true,
          pidsLimit: 256,
          memory: "1g",
          cpus: 1,
          capDrop: ["ALL"],
        },
      },
    },
  },
}
```

首次构建沙箱镜像：
```bash
scripts/sandbox-setup.sh
```

### 11.6 命令安全

```json5
{
  commands: {
    text: true,                // 斜杠命令解析
    bash: false,               // 禁止 shell 命令
    config: false,             // 禁止 /config 写入
    debug: false,              // 禁止 /debug 运行时覆盖
    restart: false,            // 禁止重启
  },
}
```

---

## 12. 节点设备管理

### 12.1 节点概述

节点是连接到 Gateway 的移动设备（iOS/Android/macOS），提供以下功能：
- **Canvas** — Web 视图界面用于展示
- **相机** — 拍照和录像
- **语音** — TTS 播放
- **位置** — 获取设备位置
- **屏幕录制** — 录屏

### 12.2 节点管理 CLI

```bash
# 节点列表与状态
openclaw nodes status                     # 所有节点状态
openclaw nodes list --connected           # 仅在线节点
openclaw nodes describe --node <id>       # 节点详情

# 配对管理
openclaw nodes pending                    # 待审批的节点
openclaw nodes approve <requestId>        # 批准节点
openclaw nodes reject <requestId>         # 拒绝节点
openclaw nodes rename --node <id> --name "新名字"

# 节点执行
openclaw nodes run --node <id> <command>  # 在节点执行命令

# 通知
openclaw nodes notify --node <id> --title "提醒" --body "内容"
```

### 12.3 相机操作

```bash
openclaw nodes camera list --node <id>                    # 列出相机
openclaw nodes camera snap --node <id>                    # 拍照
openclaw nodes camera snap --node <id> --facing front     # 前置相机
openclaw nodes camera clip --node <id> --duration 10s     # 录像10秒
```

### 12.4 Canvas 操作

```bash
openclaw nodes canvas snapshot --node <id>                 # 截图
openclaw nodes canvas present --node <id> --target <url>   # 展示 URL
openclaw nodes canvas hide --node <id>                     # 隐藏
openclaw nodes canvas navigate <url> --node <id>           # 导航
openclaw nodes canvas eval "document.title" --node <id>    # 执行 JS
```

### 12.5 位置获取

```bash
openclaw nodes location get --node <id>                   # 获取位置
```

### 12.6 无头节点主机

在没有图形界面的服务器上运行节点服务：

```bash
openclaw node run --host <gateway-host> --port 18789      # 运行节点
openclaw node install                                      # 安装服务
openclaw node status                                       # 检查状态
```

---

## 13. 模型与提供商配置

### 13.1 模型管理命令

```bash
# 模型状态
openclaw models status                       # 模型/认证状态
openclaw models status --probe               # 实时探测
openclaw models list                         # 列出可用模型
openclaw models list --all                   # 全部列出

# 设置模型
openclaw models set "anthropic/claude-sonnet-4-5"
openclaw models set-image "openai/gpt-5.2"

# 别名管理
openclaw models aliases list
openclaw models aliases add opus "anthropic/claude-opus-4-5"

# 回退管理
openclaw models fallbacks list
openclaw models fallbacks add "openai/gpt-5.2"
openclaw models fallbacks clear

# 认证管理
openclaw models auth add                     # 交互式添加
openclaw models auth setup-token             # 设置 token (Anthropic)
openclaw models auth order get --provider anthropic   # 查看认证顺序
```

### 13.2 模型配置

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "openai/gpt-5.2": { alias: "GPT" },
        "openai/gpt-5-mini": {},
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["openai/gpt-5.2", "openai/gpt-5-mini"],
      },
      imageModel: {
        primary: "openai/gpt-5.2",
      },
    },
  },
}
```

### 13.3 自定义提供商

```json5
{
  models: {
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "${MY_KEY}",
        api: "openai-completions",  // openai-completions | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

### 13.4 支持的内置提供商

| 提供商 | 配置方式 | 标识符 |
|--------|---------|--------|
| Anthropic | OAuth / API Key | `anthropic/*` |
| OpenAI | Codex OAuth / API Key | `openai/*` |
| Google Gemini | API Key | `google/*` |
| Z.AI (GLM) | API Key (`ZAI_API_KEY`) | `zai/*` |
| Moonshot (Kimi) | API Key | `moonshot/*` |
| Kimi Coding | API Key | `kimi-coding/*` |
| MiniMax | API Key | `minimax/*` |
| OpenRouter | API Key | `openrouter/*` |
| OpenCode Zen | API Key | `opencode/*` |
| Cerebras | API Key | `cerebras/*` |
| Ollama | 本地 | `ollama/*` |

---

## 14. 浏览器工具

### 14.1 浏览器管理

OpenClaw 内置浏览器控制功能，可启动隔离的 Chrome/Brave/Edge/Chromium 实例。

```bash
# 浏览器管理
openclaw browser status                 # 状态
openclaw browser start                  # 启动
openclaw browser stop                   # 停止
openclaw browser profiles               # 列出配置
openclaw browser start --browser-profile work  # 使用指定配置

# 标签页管理
openclaw browser tabs                   # 列出标签页
openclaw browser open "https://example.com"  # 打开 URL
openclaw browser close <targetId>       # 关闭标签
openclaw browser focus <targetId>       # 聚焦标签
```

### 14.2 浏览器操作

```bash
# 导航与截图
openclaw browser navigate "https://example.com"
openclaw browser screenshot              # 截图
openclaw browser screenshot --full-page  # 全页截图
openclaw browser snapshot                # 无障碍快照

# 交互操作
openclaw browser click "123"             # 点击元素
openclaw browser type "456" "hello"      # 输入文本
openclaw browser type "456" "hello" --submit  # 输入并提交
openclaw browser press "Enter"           # 按键
openclaw browser select "789" "option1"  # 选择
openclaw browser hover "123"             # 悬停
openclaw browser resize 1920 1080        # 调整窗口

# 高级操作
openclaw browser evaluate "document.title"       # 执行 JS
openclaw browser console                          # 查看控制台
openclaw browser wait --text "加载完成"            # 等待文本
openclaw browser pdf                              # 生成 PDF
```

### 14.3 浏览器配置

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
    },
  },
}
```

---

## 15. 故障排除

### 15.1 常见问题快速诊断

```bash
# 通用诊断命令
openclaw status --all                    # 完整诊断报告
openclaw doctor                          # 配置健康检查
openclaw doctor --fix                    # 自动修复
openclaw health --verbose                # 详细健康状态
openclaw logs --follow                   # 实时日志跟踪
```

### 15.2 Gateway 启动失败

**问题：** Gateway 无法启动，提示配置验证错误

**解决：**
```bash
openclaw doctor                          # 查看具体错误
openclaw doctor --fix --yes              # 自动修复
```

配置严格验证：无效配置会导致拒绝启动。验证失败时只允许诊断命令。

### 15.3 渠道连接问题

**WhatsApp 连接问题：**
```bash
openclaw channels login                  # 重新扫码登录
openclaw channels status --probe         # 检查渠道状态
openclaw logs --channel whatsapp         # 查看渠道日志
```

**Telegram 无法响应私信：**
```bash
openclaw pairing list telegram           # 查看待审批配对
openclaw pairing approve telegram <code> # 批准配对
```

### 15.4 智能体不回复

**检查顺序：**
1. `openclaw models status` — 确认认证配置正常
2. `openclaw health` — 确认 Gateway 运行正常
3. `openclaw channels status` — 确认渠道在线
4. `openclaw pairing list` — 检查是否需要批准配对
5. 查看 `openclaw logs --follow` — 实时观察日志

### 15.5 认证问题

```bash
openclaw models status --probe           # 实时探测认证
openclaw models auth add                 # 重新添加认证
```

```
# OAuth 过期
claude setup-token                       # Anthropic 重新认证
# 或通过向导重新设置
openclaw onboard --auth-choice setup-token
```

### 15.6 沙箱问题

```bash
openclaw sandbox list                    # 列出沙箱
openclaw sandbox recreate                # 重建沙箱
openclaw sandbox explain                 # 解释沙箱配置
```

### 15.7 配置迁移

```bash
openclaw doctor                          # 自动检测需要迁移的配置
openclaw doctor --fix                    # 执行迁移
```

### 15.8 重置

```bash
openclaw reset --scope config            # 仅重置配置
openclaw reset --scope config+creds+sessions  # 重置配置+凭据+会话
openclaw reset --scope full --yes        # 完全重置（非交互）
```

### 15.9 更新

```bash
openclaw update                          # 更新到最新版本
npm update -g openclaw@latest            # 通过 npm 更新
```

### 15.10 配置示例集合

**自聊天模式（同一 WhatsApp 账号对话）：**
```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

**带身份标识的智能体：**
```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Clawd",
          emoji: "🦞",
          avatar: "avatars/clawd.png",
        },
        groupChat: {
          mentionPatterns: ["@Clawd", "🦞"],
        },
      },
    ],
  },
  messages: {
    responsePrefix: "auto",    // 自动推断 [{identity.name}]
    ackReaction: "🦞",
  },
}
```

**完整的多渠道配置：**
```json5
{
  gateway: {
    port: 18789,
    bind: "loopback",
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5.2"],
      },
      timeoutSeconds: 600,
    },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
    telegram: {
      enabled: true,
      botToken: "${TELEGRAM_BOT_TOKEN}",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
  messages: {
    responsePrefix: "🦞",
    queue: { mode: "collect", debounceMs: 1000 },
  },
}
```

---

> **更多资源：**
> - 官方文档：`openclaw docs` 或访问 docs.openclaw.ai
> - 技能市场：clawhub.com
> - 源码：github.com/openclaw/openclaw
> - 社区：Discord discord.com/invite/clawd
