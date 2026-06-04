# WSL2 安装 Ubuntu 完整教程

> 适用于 Windows 10/11，手把手教你从零开始安装和配置 WSL2 Ubuntu

## 一、什么是 WSL？

**WSL（Windows Subsystem for Linux）** 是微软官方提供的 Linux 子系统，让你在 Windows 里直接运行 Linux 命令和工具，不需要虚拟机或双系统。

### WSL1 vs WSL2

| 特性 | WSL1 | WSL2 |
|------|------|------|
| 架构 | 翻译层 | 真正的 Linux 内核 |
| 性能 | 一般 | 接近原生 Linux |
| Docker 支持 | ❌ | ✅ |
| 完整系统调用 | ❌ | ✅ |
| 启动速度 | 快 | 快 |
| 推荐 | — | ✅ 推荐使用 |

---

## 二、系统要求

- **Windows 10** 版本 2004 及以上（Build 19041+）
- **Windows 11** 任意版本
- 至少 4GB 内存（推荐 8GB+）
- 至少 10GB 可用磁盘空间

### 检查 Windows 版本

按 `Win + R`，输入 `winver`，回车查看版本号。

---

## 三、启用 WSL 功能

### 方法一：命令行启用（推荐）

以**管理员身份**打开 PowerShell，执行：

```powershell
# 启用 WSL 功能
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 启用虚拟机平台
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

**执行完必须重启电脑！**

### 方法二：图形界面启用

1. 按 `Win + R`，输入 `optionalfeatures`，回车
2. 找到 **适用于 Linux 的 Windows 子系统**，勾选
3. 找到 **虚拟机平台**，勾选
4. 点确定，重启电脑

---

## 四、设置 WSL2 为默认版本

重启后，以**管理员身份**打开 PowerShell：

```powershell
# 设置 WSL2 为默认版本
wsl --set-default-version 2

# 更新 WSL 到最新版
wsl --update
```

---

## 五、安装 Ubuntu

### 方法一：命令行安装（最简单）

```powershell
wsl --install -d Ubuntu
```

等待下载和安装完成（可能需要几分钟），然后会提示你设置用户名和密码。

### 方法二：Microsoft Store 安装

1. 打开 **Microsoft Store**
2. 搜索 **Ubuntu**
3. 选择 **Ubuntu 24.04 LTS**（推荐）
4. 点击 **获取/安装**
5. 安装完成后点击 **打开**

### 方法三：手动下载安装包

如果 Microsoft Store 下载慢，可以手动下载：

1. 访问 https://aka.ms/wslubuntu2404
2. 下载 `.appx` 文件
3. 双击安装，或 PowerShell 执行：
```powershell
Add-AppxPackage .\Ubuntu_2404_xxx.appx
```

---

## 六、首次启动设置

1. 打开 Ubuntu（开始菜单搜 Ubuntu，或 PowerShell 执行 `wsl -d Ubuntu`）
2. 等待初始化完成（首次可能需要 1-2 分钟）
3. 设置**用户名**（小写字母，如 `autumn`）
4. 设置**密码**（输入时不显示，输完按回车）
5. 确认密码

设置完成后看到类似这样的提示：

```
autumn@DESKTOP-XXXX:~$
```

恭喜！Ubuntu 安装成功 🎉

---

## 七、基础配置

### 1. 更新系统

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. 安装常用工具

```bash
sudo apt install -y \
  git \
  curl \
  wget \
  vim \
  nano \
  build-essential \
  python3 \
  python3-pip \
  python3-venv \
  openssh-server
```

### 3. 设置中文语言（可选）

```bash
sudo locale-gen zh_CN.UTF-8
sudo update-locale LANG=zh_CN.UTF-8
```

需要重启 WSL 生效：

```powershell
wsl --shutdown
wsl -d Ubuntu
```

---

## 八、SSH 免密登录配置

### 1. 在 Windows 生成 SSH 密钥

PowerShell 执行：

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\vm_key"
```

连续按三次回车（密码为空）。

### 2. 查看公钥

```powershell
Get-Content "$env:USERPROFILE\.ssh\vm_key.pub"
```

输出类似：
```
ssh-ed25519 AAAAC3...xxxxxxx... user@DESKTOP
```

### 3. 在 Ubuntu 中添加公钥

```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "ssh-ed25519 AAAAC3...xxxxxxx... user@DESKTOP" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 4. 测试免密登录

PowerShell 执行：

```powershell
ssh -i "$env:USERPROFILE\.ssh\vm_key" 你的用户名@localhost
```

无需输入密码即登录成功 ✅

---

## 九、安装 Home Assistant

### 1. 安装依赖

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip libffi-dev libssl-dev
```

### 2. 创建 HA 用户

```bash
sudo useradd -rm homeassistant -G dialout
```

### 3. 创建虚拟环境并安装

```bash
sudo -u homeassistant -H -s
python3 -m venv homeassistant
source homeassistant/bin/activate
pip install --upgrade pip
pip install homeassistant
```

### 4. 启动 HA

```bash
hass
```

首次启动会下载大量文件，耐心等待。

### 5. 访问 HA

浏览器打开：http://localhost:8123

设置管理员账号和密码即可。

### 注意事项

- WSL 版本的 HA 没有 Supervisor 和 Add-ons 商店
- 关闭终端 = 关闭 HA，需要保持终端开着
- 每次开机需手动启动

---

## 十、安装 ESP-IDF 开发环境

### 1. 安装依赖

```bash
sudo apt install -y git wget flex bison gperf python3 python3-venv \
  cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
```

### 2. 克隆 ESP-IDF

```bash
mkdir -p ~/esp
cd ~/esp
git clone --recursive https://github.com/espressif/esp-idf.git
```

### 3. 安装工具链

```bash
cd ~/esp/esp-idf
./install.sh esp32
```

### 4. 激活环境

```bash
source export.sh
```

### 5. 写入 bashrc（可选，免去每次手动激活）

```bash
echo "source ~/esp/esp-idf/export.sh" >> ~/.bashrc
```

---

## 十一、常用命令速查

### WSL 管理命令（Windows PowerShell）

| 命令 | 说明 |
|------|------|
| `wsl -d Ubuntu` | 启动 Ubuntu |
| `wsl --shutdown` | 关闭所有 WSL 实例 |
| `wsl --list --verbose` | 查看已安装的发行版 |
| `wsl --unregister Ubuntu` | 卸载 Ubuntu |
| `wsl --update` | 更新 WSL |
| `wsl --set-default-version 2` | 设置默认 WSL2 |

### Ubuntu 内部操作

| 命令 | 说明 |
|------|------|
| `sudo apt update` | 更新软件源 |
| `sudo apt upgrade` | 更新已安装软件 |
| `ip addr` | 查看 IP 地址 |
| `hostname -I` | 快速查看 IP |
| `exit` | 退出 Ubuntu |

---

## 十二、常见问题

### Q1: WSL 启动卡住不动？

```powershell
# 在管理员 PowerShell 中执行
wsl --shutdown
Get-Process | Where-Object { $_.ProcessName -match "wsl|ubuntu" } | Stop-Process -Force
wsl -d Ubuntu
```

### Q2: WSL 服务无法重启？

需要**管理员权限**打开 PowerShell：

```powershell
Restart-Service wslservice
```

如果还是不行，**重启电脑**是最干净的解决方案。

### Q3: Ubuntu 无法获取 IP 地址？

```bash
sudo dhclient ens33
```

如果不行，检查 VMware 网络设置，确保是 NAT 或桥接模式。

### Q4: SSH 连接被拒绝？

```bash
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

### Q5: 忘记 Ubuntu 用户名/密码？

```powershell
# 重置密码
wsl -d Ubuntu -u root
passwd 你的用户名
```

---

## 十三、WSL vs VMware 虚拟机

| 特性 | WSL | VMware 虚拟机 |
|------|-----|--------------|
| 图形界面 | ❌ 默认无 | ✅ 完整桌面 |
| 启动速度 | 几秒 | 几分钟 |
| 资源占用 | 很少 | 多 |
| Windows 文件访问 | ✅ 直接访问 | 需共享文件夹 |
| Docker 支持 | ✅ | ✅ |
| 完整系统体验 | ❌ | ✅ |
| 适合场景 | 命令行开发、快速测试 | 完整 Linux 体验 |

**建议：** 两者可以同时使用，WSL 用于快速命令行操作，VMware 用于需要完整桌面的场景。

---

## 参考链接

- [微软官方 WSL 文档](https://learn.microsoft.com/zh-cn/windows/wsl/)
- [Ubuntu on WSL](https://ubuntu.com/wsl)
- [Home Assistant 安装文档](https://www.home-assistant.io/installation/)

---

*最后更新：2026-06-04*
*作者：子秋*
