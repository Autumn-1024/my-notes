# OpenClaw 安装教程

> 从零开始，在 Windows 上安装和配置 OpenClaw AI 助手平台

---

## 一、安装 Node.js

OpenClaw 基于 Node.js 运行，所以第一步是安装 Node.js。

### 1.1 使用 winget 安装（推荐）

打开 **PowerShell**，执行：

```powershell
winget install OpenJS.NodeJS.LTS
```

这会自动下载并安装 Node.js LTS 版本（当前 v24.x）。

> 💡 winget 是 Windows 11/10 自带的包管理器，无需额外安装。如果你的系统没有 winget，可以用方式二。

### 1.2 方式二：手动下载安装

如果 winget 不可用，可以手动安装：

1. 官网：https://nodejs.org/
2. 下载 **LTS 版本** 的 **Windows Installer (.msi)** 64-bit
3. 双击 `.msi` 文件，一路 Next 保持默认
4. 点击 Install 完成

### 1.3 验证安装

打开 **PowerShell**，输入：

```powershell
node --version
# 应该输出类似 v24.15.0

npm --version
# 应该输出类似 10.x.x
```

如果两个命令都有版本号输出，说明 Node.js 安装成功。

---

## 二、安装 Git

OpenClaw 的 npm 依赖需要 Git 来下载，所以必须先安装 Git。

### 2.1 使用 winget 安装（推荐）

```powershell
winget install Git.Git
```

### 2.2 方式二：手动下载安装

1. 官网：https://git-scm.com/download/win
2. 下载 64-bit Git for Windows Setup
3. 双击安装，一路 Next 保持默认即可

### 2.3 验证安装

**⚠️ 安装 Git 后需要重新打开一个 PowerShell 窗口**，否则命令不会生效。

```powershell
git --version
# 应该输出类似 git version 2.53.0
```

---

## 三、安装 OpenClaw CLI

### 3.1 全局安装

确认 Git 已安装后，在终端中执行：

```powershell
npm install -g openclaw
```

> 💡 如果遇到权限问题，可以以管理员身份运行 PowerShell。

### 3.2 验证安装

```powershell
openclaw --version
# 应该输出类似 OpenClaw 2026.5.28
```

输出版本号即安装成功。

---

## 四、初始化配置（交互式向导）

OpenClaw 提供了交互式配置向导，推荐新手使用。

### 4.1 运行配置向导

```powershell
openclaw onboard
```

向导会引导你完成以下配置：

1. **选择认证方式** — API Key、GitHub Copilot、Claude CLI 等
2. **配置模型** — 选择主要模型和备用模型
3. **配置 Gateway** — 设置端口、认证方式
4. **创建工作空间** — 默认 `~/.openclaw/workspace`
5. **配置频道** — QQ、Discord、Telegram 等
6. **安装技能** — 预装常用 skills

### 4.2 一键配置（非交互式）

如果你已经知道自己要什么，可以用命令行参数一步到位：

```powershell
openclaw onboard --non-interactive --accept-risk --install-daemon --xiaomi-api-key "tp-你的API密钥"
```

常用参数：

| 参数 | 说明 |
|------|------|
| `--non-interactive` | 无交互模式，跳过所有提示 |
| `--accept-risk` | 确认了解 AI 助手的风险 |
| `--install-daemon` | 安装 Gateway 后台服务 |
| `--xiaomi-api-key` | 小米 MiMo API Key |
| `--openai-api-key` | OpenAI API Key |
| `--anthropic-api-key` | Anthropic API Key |
| `--workspace <dir>` | 指定工作空间目录 |
| `--skip-channels` | 跳过频道配置 |
| `--skip-skills` | 跳过技能安装 |

---

## 五、手动配置（进阶）

如果你想手动编辑配置文件，或者需要更精细的控制。

### 5.1 配置文件位置

- **配置文件**：`~/.openclaw/openclaw.json`（即 `C:\Users\你的用户名\.openclaw\openclaw.json`）
- **工作空间**：`~/.openclaw/workspace/`

### 5.2 最小配置示例

```json5
// ~/.openclaw/openclaw.json
{
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      model: {
        primary: "xiaomi-coding/mimo-v2.5-pro"
      }
    }
  },
  providers: {
    "xiaomi-coding": {
      baseUrl: "https://token-plan-cn.xiaomimimo.com/v1",
      apiKey: "tp-你的API密钥",
      models: ["mimo-v2.5-pro"]
    }
  }
}
```

### 5.3 使用 CLI 命令修改配置

```powershell
# 查看当前配置
openclaw config get

# 查看某个配置项
openclaw config get agents.defaults.model

# 设置配置项
openclaw config set agents.defaults.model.primary "xiaomi-coding/mimo-v2.5-pro"

# 删除配置项
openclaw config unset plugins.entries.brave.config.webSearch.apiKey
```

### 5.4 验证配置

```powershell
openclaw doctor
```

如果配置有问题，`doctor` 会告诉你具体错误和修复建议。

```powershell
openclaw doctor --fix
```

加 `--fix` 参数可以自动修复常见问题。

---

## 六、启动 Gateway

Gateway 是 OpenClaw 的核心服务，负责处理消息、调用模型、管理会话。

### 6.1 启动 Gateway

```powershell
openclaw gateway start
```

### 6.2 查看 Gateway 状态

```powershell
openclaw gateway status
```

### 6.3 停止 Gateway

```powershell
openclaw gateway stop
```

### 6.4 重启 Gateway

```powershell
openclaw gateway restart
```

### 6.5 安装为系统服务（推荐）

如果在 `onboard` 时没有加 `--install-daemon`，可以手动安装：

```powershell
openclaw gateway install
```

这会把 Gateway 注册为 Windows 服务，开机自动启动。

---

## 七、访问 Control UI

Gateway 启动后，可以通过浏览器访问 Control UI：

### 7.1 打开 Control UI

默认地址：http://127.0.0.1:18789

在浏览器中打开即可。

### 7.2 Control UI 功能

- **Chat** — 和 AI 助手对话
- **Config** — 可视化编辑配置
- **Sessions** — 查看和管理会话
- **Status** — 查看系统状态

---

## 八、配置频道（Channels）

OpenClaw 支持多种聊天频道，以下是常用频道的配置方法。

### 8.1 QQ

需要先在 QQ 开放平台创建机器人，获取 AppID 和 Token。

```json5
{
  channels: {
    qqbot: {
      enabled: true,
      appId: "你的AppID",
      token: "你的Token",
      secret: "你的Secret"
    }
  }
}
```

### 8.2 Discord

需要在 Discord Developer Portal 创建 Bot，获取 Token。

```json5
{
  channels: {
    discord: {
      enabled: true,
      botToken: "你的Bot Token",
      dmPolicy: "allowlist",
      allowFrom: ["discord:user:你的用户ID"]
    }
  }
}
```

### 8.3 Telegram

需要在 @BotFather 创建 Bot，获取 Token。

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "你的Bot Token",
      dmPolicy: "allowlist",
      allowFrom: ["tg:你的用户ID"]
    }
  }
}
```

### 8.4 飞书

需要在飞书开放平台创建应用，获取 App ID 和 App Secret。

```json5
{
  channels: {
    feishu: {
      enabled: true,
      appId: "你的App ID",
      appSecret: "你的App Secret"
    }
  }
}
```

---

## 九、配置模型（Providers）

### 9.1 小米 MiMo

#### Token Plan（推荐）

```json5
{
  providers: {
    "xiaomi-coding": {
      baseUrl: "https://token-plan-cn.xiaomimimo.com/v1",
      apiKey: "tp-你的API密钥",
      models: ["mimo-v2.5-pro", "mimo-v2.5"]
    }
  }
}
```

#### 按量付费

```json5
{
  providers: {
    "xiaomi": {
      baseUrl: "https://api.xiaomimimo.com/v1",
      apiKey: "sk-你的API密钥",
      models: ["mimo-v2.5-pro", "mimo-v2.5"]
    }
  }
}
```

> ⚠️ **重要**：MiMo 模型必须用 OpenAI 兼容协议（`/v1`），不能用 Anthropic 协议（`/anthropic`）。
> 原因：MiMo 开了思考模式 + 多轮会话 + 工具调用时，需要回传 `reasoning_content`，Anthropic 协议不支持。

### 9.2 OpenAI

```json5
{
  providers: {
    "openai": {
      apiKey: "sk-你的API密钥",
      models: ["gpt-4o", "gpt-4o-mini"]
    }
  }
}
```

### 9.3 Anthropic

```json5
{
  providers: {
    "anthropic": {
      apiKey: "sk-ant-你的API密钥",
      models: ["claude-sonnet-4-6", "claude-3-5-sonnet-20241022"]
    }
  }
}
```

---

## 十、常用命令速查

```powershell
# 版本信息
openclaw --version

# 交互式配置向导
openclaw onboard

# 配置管理
openclaw config get              # 查看配置
openclaw config set <key> <val>  # 设置配置
openclaw config unset <key>      # 删除配置

# Gateway 管理
openclaw gateway start           # 启动
openclaw gateway stop            # 停止
openclaw gateway restart         # 重启
openclaw gateway status          # 查看状态

# 诊断
openclaw doctor                  # 检查配置问题
openclaw doctor --fix            # 自动修复
openclaw status                  # 查看运行状态
openclaw health                  # 健康检查

# 日志
openclaw logs                    # 查看日志
openclaw logs --tail 100         # 查看最近100行

# 技能管理
openclaw skills list             # 列出已安装技能
openclaw skills install <name>   # 安装技能
openclaw skills check            # 检查技能状态
```

---

## 十一、常见问题

### Q: npm install 很慢？

使用国内镜像：

```powershell
npm config set registry https://registry.npmmirror.com
```

### Q: Gateway 启动失败？

1. 检查配置文件语法：`openclaw doctor`
2. 查看错误日志：`openclaw logs`
3. 常见原因：
   - 配置文件 JSON 语法错误
   - 端口被占用（默认 18789）
   - API Key 无效

### Q: 如何更换模型？

```powershell
openclaw config set agents.defaults.model.primary "xiaomi-coding/mimo-v2.5-pro"
openclaw gateway restart
```

### Q: 如何查看当前使用的模型？

```powershell
openclaw config get agents.defaults.model.primary
```

### Q: Control UI 打不开？

1. 确认 Gateway 已启动：`openclaw gateway status`
2. 确认端口：`openclaw config get gateway.port`
3. 检查防火墙是否阻止了 18789 端口

### Q: MiMo 模型报 400 错误？

确保使用的是 OpenAI 兼容协议（`/v1`），不是 Anthropic 协议（`/anthropic`）。

---

## 十二、完整流程速查

```
1. 安装 Node.js → node --version 验证
2. 安装 Git → git --version 验证（重新打开 PowerShell）
3. npm install -g openclaw → openclaw --version 验证
4. openclaw onboard → 交互式配置（选择模型、认证、频道等）
5. openclaw gateway start → 启动 Gateway
6. 浏览器打开 http://127.0.0.1:18789 → 访问 Control UI
7. 开始对话！
```

---

*教程版本：2026-06-05 | 基于 OpenClaw 2026.5.28*
