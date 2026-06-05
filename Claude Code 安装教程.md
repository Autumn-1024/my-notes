# Claude Code 安装教程

> 从零开始，在 Windows 上安装和配置 Claude Code（含 MiMo 模型接入）

---

## 一、安装 Node.js

Claude Code 是基于 Node.js 的 CLI 工具，所以第一步是安装 Node.js。

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

打开 **PowerShell** 或 **CMD**，输入：

```bash
node --version
# 应该输出类似 v24.15.0

npm --version
# 应该输出类似 10.x.x
```

如果两个命令都有版本号输出，说明 Node.js 安装成功。

---

## 二、安装 Claude Code CLI

### 2.1 全局安装

在终端中执行：

```bash
npm install -g @anthropic-ai/claude-code
```

> 💡 如果遇到权限问题（Windows 一般不会），可以以管理员身份运行 PowerShell。

### 2.2 验证安装

```bash
claude --version
```

输出版本号即安装成功。

---

## 三、配置 Claude Code（使用 MiMo 模型）

Claude Code 默认连接 Anthropic 官方 API。如果你要使用小米 MiMo 模型，需要修改配置。

### 3.1 了解协议差异

| 产品 | 协议 | 说明 |
|------|------|------|
| Claude Code | Anthropic 兼容 | `/anthropic` 端点 |
| OpenClaw | OpenAI 兼容 | `/v1` 端点 |

### 3.2 MiMo API 两种付费方式

#### 方式一：按量付费（推荐新手）

- **BASE_URL**: `https://api.xiaomimimo.com/anthropic`
- **API Key 格式**: `sk-xxxxx`
- 在 MiMo 控制台获取 API Key

#### 方式二：Token Plan（固定订阅）

- **BASE_URL**: `https://token-plan-cn.xiaomimimo.com/anthropic`
- **API Key 格式**: `tp-xxxxx`
- 套餐：Lite ¥39/月、Standard ¥99/月、Pro ¥329/月、Max ¥659/月

> ⚠️ **tp-xxxxx 和 sk-xxxxx 不可混用！** Token Plan 的 Key 只能用于 Token Plan 端点。

### 3.3 一键配置（推荐）

在 PowerShell 中执行以下命令，自动创建配置文件并跳过首次引导：

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude" | Out-Null; Set-Content -Path "$env:USERPROFILE\.claude\settings.json" -Value '{"env":{"ANTHROPIC_BASE_URL":"https://token-plan-cn.xiaomimimo.com/anthropic","ANTHROPIC_AUTH_TOKEN":"tp-你的API密钥","MODEL":"mimo-v2.5-pro"}}'; Set-Content -Path "$env:USERPROFILE\.claude.json" -Value '{"hasCompletedOnboarding":true}'
```

> ⚠️ 把 `tp-你的API密钥` 替换成你自己的 Token Plan API Key。
> 如果用按量付费，把 `token-plan-cn` 改成 `api`，Key 格式从 `tp-` 改成 `sk-`。

#### 1M 上下文模型（可选）

如果需要 1M 上下文窗口，把命令里的 `mimo-v2.5-pro` 改成 `mimo-v2.5-pro[1m]`。

### 3.5 清除官方环境变量（重要！）

如果你之前设置过 Anthropic 官方的环境变量，需要先清除，否则会冲突：

在 PowerShell 中执行：

```powershell
Remove-Item Env:ANTHROPIC_AUTH_TOKEN -ErrorAction SilentlyContinue
Remove-Item Env:ANTHROPIC_BASE_URL -ErrorAction SilentlyContinue
```

> 💡 这只是临时清除当前终端的环境变量。如果系统环境变量里有，需要去 **系统设置 → 环境变量** 里手动删除。

### 3.6 启动 Claude Code

```bash
claude
```

启动后即可正常对话。

---

## 四、Token Plan 支持的模型

| 模型 | 说明 |
|------|------|
| mimo-v2.5-pro | 最强推理，1T参数/42B激活/1M上下文 |
| mimo-v2.5 | 全模态（图/视频/音频/文本） |
| mimo-v2-pro | 1T参数/42B激活/1M上下文 |
| mimo-v2-omni | 多模态 |
| mimo-v2.5-tts-voiceclone | 音色克隆（限时免费） |
| mimo-v2.5-tts-voicedesign | 音色设计（限时免费） |
| mimo-v2.5-tts | TTS（限时免费） |
| mimo-v2-tts | TTS（限时免费） |

> ⚠️ **mimo-v2-flash 不在 Token Plan 支持列表中！**

---

## 五、Token Plan Credit 消耗参考

| 模型 | 缓存命中 | 未命中 | 输出 |
|------|---------|--------|------|
| V2.5-Pro | 2.5 | 300 | 600 |
| V2.5 | 2 | 100 | 200 |
| V2-Pro | 140 | 700 | 2100 |
| V2-Omni | 56 | 280 | 1400 |

- 夜间（0:00-8:00）消耗 × 0.8（更便宜）
- 额度耗尽后停止服务
- **仅限编程工具使用，禁止非 Coding 场景**

---

## 六、常见问题

### Q: 启动后报 400 错误？

**原因**：MiMo 模型开了思考模式 + 多轮会话 + 工具调用时，assistant 消息必须完整回传 `reasoning_content`。Anthropic 协议下某些工具不支持。

**解决**：确保用的是 Anthropic 协议端点（`/anthropic`），不是 OpenAI 协议（`/v1`）。

### Q: npm install 很慢？

使用国内镜像：

```bash
npm config set registry https://registry.npmmirror.com
```

### Q: VS Code 插件怎么配？

VS Code 的 Claude Code 插件会自动复用 CLI 的配置（`~/.claude/settings.json`），所以配好 CLI 后插件直接可用。

也可以在 VS Code 设置中独立配置环境变量。

### Q: 多集群选择？

Token Plan 支持多地区集群：

| 地区 | BASE_URL |
|------|----------|
| 中国（默认） | `https://token-plan-cn.xiaomimimo.com/anthropic` |
| 新加坡 | `https://token-plan-sgp.xiaomimimo.com/anthropic` |
| 欧洲 | `https://token-plan-ams.xiaomimimo.com/anthropic` |

---

## 七、完整流程速查

```
1. 安装 Node.js → node --version 验证
2. npm install -g @anthropic-ai/claude-code → claude --version 验证
3. 创建 ~/.claude/settings.json（填入 BASE_URL + API Key + MODEL）
4. 创建 ~/.claude.json → {"hasCompletedOnboarding": true}
5. 清除官方环境变量（如有）
6. 终端输入 claude 启动
```

---

*教程版本：2026-06-05 | 基于 MiMo V2.5 系列 API*
