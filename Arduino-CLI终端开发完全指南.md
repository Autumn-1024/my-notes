# Arduino CLI 终端开发完全指南

> 告别 Arduino IDE，用终端高效开发 ESP32/Arduino

## 一、Arduino CLI 是什么

Arduino CLI 是 Arduino 官方的命令行工具，不需要打开 IDE，直接在终端里编译、烧录、监控串口。

**优势：**
- 编译速度快（比 IDE 快 3-5 倍）
- 占用资源少
- 可以配合任何编辑器（VS Code、Vim、Notepad++）
- 支持脚本自动化
- 适合批量操作

---

## 二、安装 Arduino CLI

### 2.1 Windows 安装

#### 方法 1：手动下载（推荐）

1. 打开下载页面：https://github.com/arduino/arduino-cli/releases
2. 下载 Windows 64 位版本：`arduino-cli_latest_Windows_64bit.zip`
3. 解压到一个目录，比如 `C:\ArduinoCLI\`
4. 添加到系统 PATH：

```powershell
# 用命令添加
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\ArduinoCLI", "User")
```

5. 重启终端，验证安装：

```powershell
arduino-cli version
```

看到版本号就成功了：
```
arduino-cli  Version: 1.5.0 Commit: dd407d42d Date: 2026-05-19T14:07:59Z
```

#### 方法 2：一键脚本安装

保存以下内容为 `install-arduino-cli.ps1`，以管理员身份运行：

```powershell
# Arduino CLI 一键安装脚本
Write-Host "=== Arduino CLI 安装脚本 ===" -ForegroundColor Green

# 1. 下载 Arduino CLI
Write-Host "下载 Arduino CLI..." -ForegroundColor Yellow
$version = "1.5.0"
$url = "https://github.com/arduino/arduino-cli/releases/download/v${version}/arduino-cli_${version}_Windows_64bit.zip"
$zip = "$env:TEMP\arduino-cli.zip"
$installDir = "C:\ArduinoCLI"

Invoke-WebRequest -Uri $url -OutFile $zip
Expand-Archive -Path $zip -DestinationPath $installDir -Force

# 2. 添加到 PATH
Write-Host "添加到系统 PATH..." -ForegroundColor Yellow
$currentPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($currentPath -notlike "*$installDir*") {
    [Environment]::SetEnvironmentVariable("Path", "$currentPath;$installDir", "User")
}

# 3. 初始化配置
Write-Host "初始化配置..." -ForegroundColor Yellow
& "$installDir\arduino-cli.exe" config init
& "$installDir\arduino-cli.exe" config add board_manager.additional_urls https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

# 4. 更新索引
Write-Host "更新开发板索引..." -ForegroundColor Yellow
& "$installDir\arduino-cli.exe" core update-index

# 5. 安装 ESP32 核心
Write-Host "安装 ESP32 核心（需要几分钟）..." -ForegroundColor Yellow
& "$installDir\arduino-cli.exe" core install esp32:esp32

# 6. 验证
Write-Host "`n=== 安装完成 ===" -ForegroundColor Green
Write-Host "版本："
& "$installDir\arduino-cli.exe" version
Write-Host "`n已安装的核心："
& "$installDir\arduino-cli.exe" core list
Write-Host "`n请重启终端后使用 arduino-cli 命令" -ForegroundColor Cyan
```

运行：
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
.\install-arduino-cli.ps1
```

### 2.2 Linux 安装

```bash
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh
```

### 2.3 macOS 安装

```bash
brew install arduino-cli
```

---

## 三、初始化配置

### 3.1 初始化配置文件

```powershell
arduino-cli config init
```

会创建配置文件：`C:\Users\你的用户名\AppData\Local\Arduino15\arduino-cli.yaml`

### 3.2 添加 ESP32 开发板索引

```powershell
arduino-cli config add board_manager.additional_urls https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

### 3.3 更新索引

```powershell
arduino-cli core update-index
```

这会下载所有可用的开发板信息，可能需要几分钟。

### 3.4 安装 ESP32 核心

```powershell
arduino-cli core install esp32:esp32
```

安装过程会下载工具链（编译器、烧录工具等），可能需要 5-10 分钟。

### 3.5 验证安装

```powershell
# 查看已安装的核心
arduino-cli core list

# 查看 ESP32 开发板
arduino-cli board listall esp32
```

---

## 四、安装驱动

ESP32 开发板通常用 USB 转串口芯片，需要安装对应驱动：

| 芯片 | 驱动下载 |
|------|----------|
| **CH340/CH341** | https://www.wch.cn/downloads/CH341SER_EXE.html |
| **CP2102/CP2104** | https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers |
| **FTDI FT232** | https://ftdichip.com/drivers/vcp-drivers/ |

安装驱动后，插上 ESP32，执行：
```powershell
arduino-cli board list
```

应该能看到 COM 口和开发板信息。

---

## 五、安装常用库

```powershell
# 常用库
arduino-cli lib install "Servo"              # 舵机控制
arduino-cli lib install "WiFi"               # WiFi 功能
arduino-cli lib install "PubSubClient"       # MQTT 客户端
arduino-cli lib install "ArduinoJson"        # JSON 解析
arduino-cli lib install "FastLED"            # LED 控制
arduino-cli lib install "Adafruit_NeoPixel"  # WS2812 灯带
arduino-cli lib install "DHT sensor library" # 温湿度传感器
arduino-cli lib install "HX711"              # 称重传感器
arduino-cli lib install "ESP32Servo"         # ESP32 舵机库
```

---

## 六、基本命令速查

### 6.1 核心管理

```powershell
# 查看已安装的核心
arduino-cli core list

# 搜索核心
arduino-cli core search esp32

# 安装核心
arduino-cli core install esp32:esp32

# 更新核心
arduino-cli core update-index
arduino-cli core upgrade

# 卸载核心
arduino-cli core uninstall esp32:esp32
```

### 6.2 开发板管理

```powershell
# 查看所有支持的开发板
arduino-cli board listall

# 搜索特定开发板
arduino-cli board listall esp32

# 查看已连接的设备
arduino-cli board list

# 绑定默认开发板（在项目目录下执行）
arduino-cli board attach --fqbn esp32:esp32:esp32
```

### 6.3 库管理

```powershell
# 搜索库
arduino-cli lib search "Servo"

# 安装库
arduino-cli lib install "Servo"

# 查看已安装的库
arduino-cli lib list

# 更新所有库
arduino-cli lib update-index
arduino-cli lib upgrade

# 卸载库
arduino-cli lib uninstall "Servo"
```

### 6.4 编译

```powershell
# 基本编译
arduino-cli compile --fqbn esp32:esp32:esp32 项目路径

# 编译并显示详细信息
arduino-cli compile --fqbn esp32:esp32:esp32 --verbose 项目路径

# 编译后自动上传
arduino-cli compile --fqbn esp32:esp32:esp32 -u -p COM3 项目路径

# 清除编译缓存
arduino-cli compile --clean --fqbn esp32:esp32:esp32 项目路径

# 导出编译产物（hex/bin）
arduino-cli compile --fqbn esp32:esp32:esp32 --output-dir ./build 项目路径
```

### 6.5 烧录

```powershell
# 基本烧录
arduino-cli upload -p COM3 --fqbn esp32:esp32:esp32 项目路径

# 烧录并验证
arduino-cli upload -p COM3 --fqbn esp32:esp32:esp32 --verify 项目路径
```

### 6.6 串口监视器

```powershell
# 打开串口监视器
arduino-cli monitor -p COM3 -c baudrate=115200

# 指定波特率
arduino-cli monitor -p COM3 -c baudrate=9600

# 指定行结束符
arduino-cli monitor -p COM3 -c baudrate=115200 -c ending=both

# 退出：按 Ctrl+C
```

---

## 七、项目开发流程

### 7.1 创建新项目

```powershell
# 创建项目目录
mkdir my-project
cd my-project

# 创建主文件（必须和目录同名）
New-Item my-project.ino -ItemType File
```

### 7.2 绑定开发板

```powershell
# 在项目目录下执行
arduino-cli board attach --fqbn esp32:esp32:esp32
```

之后编译就不需要写 `--fqbn` 了

### 7.3 日常开发流程

```powershell
# 1. 进入项目目录
cd "D:\GitHub\my-project\具身智能大赛控制系统\DOBOT_LED\main"

# 2. 修改代码（用 VS Code 或任何编辑器）

# 3. 编译
arduino-cli compile

# 4. 插上 USB，查看串口号
arduino-cli board list

# 5. 烧录
arduino-cli upload -p COM3

# 6. 查看串口输出
arduino-cli monitor -p COM3 -c baudrate=115200
```

---

## 八、PowerShell 快捷命令

在 PowerShell 配置文件里添加别名，省去每次输入长命令：

```powershell
# 打开配置文件
notepad $PROFILE
```

添加以下内容：

```powershell
# Arduino CLI 快捷命令
function acc { arduino-cli compile }
function auc { arduino-cli upload -p COM3 }
function acu { arduino-cli compile -u -p COM3 }
function amon { arduino-cli monitor -p COM3 -c baudrate=115200 }
function alist { arduino-cli board list }
function aclean { arduino-cli compile --clean }

# 快速进入项目目录（根据你的实际路径修改）
function cdproj { cd "D:\GitHub\my-project\具身智能大赛控制系统\DOBOT_LED\main" }
```

保存后重启终端，之后：

| 命令 | 作用 |
|------|------|
| `acc` | 编译 |
| `auc` | 烧录 |
| `acu` | 编译+烧录 |
| `amon` | 打开串口监视器 |
| `alist` | 查看连接的设备 |
| `aclean` | 清除缓存重新编译 |
| `cdproj` | 进入项目目录 |

---

## 九、常用 ESP32 开发板 FQBN

| 开发板 | FQBN |
|--------|------|
| 普通 ESP32 | `esp32:esp32:esp32` |
| ESP32-S2 | `esp32:esp32:esp32s2` |
| ESP32-S3 | `esp32:esp32:esp32s3` |
| ESP32-C3 | `esp32:esp32:esp32c3` |
| ESP32-C6 | `esp32:esp32:esp32c6` |
| ESP32-H2 | `esp32:esp32:esp32h2` |
| ESP32-S3-DevKitC | `esp32:esp32:esp32s3_devkitc` |
| ESP32-C3-DevKitM | `esp32:esp32:esp32c3_devkitm` |
| ESP32-WROOM-32 | `esp32:esp32:esp32` |
| ESP32-WROVER | `esp32:esp32:esp32wrover` |

---

## 十、编译输出解读

```
Sketch uses 1081904 bytes (82%) of program storage space. Maximum is 1310720 bytes.
Global variables use 48856 bytes (14%) of dynamic memory, leaving 278824 bytes for local variables. Maximum is 327680 bytes.
```

| 项目 | 说明 |
|------|------|
| **Sketch uses** | 程序占用的 Flash 空间 |
| **program storage space** | Flash 总容量 |
| **Global variables** | 全局变量占用的 RAM |
| **dynamic memory** | RAM 总容量 |
| **leaving** | 剩余可用 RAM |

**警告阈值：**
- Flash > 90%：可能有风险
- RAM > 70%：需要注意栈溢出

---

## 十一、常见问题

### Q1: 找不到串口

```powershell
# 检查设备管理器有没有 COM 口
# 如果没有，安装 CH340 或 CP2102 驱动

# 查看所有可用串口
arduino-cli board list
```

### Q2: 编译报错找不到库

```powershell
# 安装缺失的库
arduino-cli lib install "库名"

# 查看库安装位置
arduino-cli lib list
```

### Q3: 烧录失败

```powershell
# 检查串口是否正确
arduino-cli board list

# 进入下载模式：按住 BOOT 按钮，按一下 RESET，松开 BOOT

# 重新烧录
arduino-cli upload -p COM3 --fqbn esp32:esp32:esp32 项目路径
```

### Q4: 编译太慢

```powershell
# 清除缓存重新编译
arduino-cli compile --clean
```

### Q5: 如何查看编译详细过程

```powershell
arduino-cli compile --verbose
```

---

## 十二、进阶用法

### 12.1 批量编译多个项目

```powershell
Get-ChildItem -Path "D:\projects" -Directory | ForEach-Object {
    Write-Host "编译 $($_.Name)..."
    arduino-cli compile --fqbn esp32:esp32:esp32 $_.FullName
}
```

### 12.2 自动烧录脚本

```powershell
# save as flash.ps1
param(
    [string]$Port = "COM3",
    [string]$Board = "esp32:esp32:esp32",
    [string]$Project = "."
)
arduino-cli compile --fqbn $Board $Project
arduino-cli upload -p $Port --fqbn $Board $Project
```

使用：
```powershell
.\flash.ps1 -Port COM3 -Project "D:\GitHub\my-project\main"
```

### 12.3 生成编译报告

```powershell
arduino-cli compile --fqbn esp32:esp32:esp32 --output-dir ./build --clean 项目路径
```

编译产物会保存在 `./build` 目录下

---

## 十三、Arduino CLI vs Arduino IDE

| 特性 | Arduino CLI | Arduino IDE |
|------|-------------|-------------|
| 编译速度 | 快（几秒） | 慢（30秒+） |
| 资源占用 | 几乎不占 | 500MB+ |
| 编辑器 | 任意（VS Code等） | 内置编辑器 |
| 自动补全 | 依赖编辑器插件 | 一般 |
| 调试 | 需要额外配置 | 基本支持 |
| 批量操作 | 支持脚本 | 不支持 |
| 学习曲线 | 需要会命令行 | 图形界面简单 |
| 适合场景 | 专业开发、批量编译 | 初学、快速原型 |

---

## 十四、工作目录推荐结构

```
D:\GitHub\my-project\
├── 具身智能大赛控制系统\
│   ├── README.md
│   └── DOBOT_LED\
│       └── main\
│           ├── main.ino        ← 主程序
│           ├── config.h        ← 配置文件
│           ├── motor.cpp       ← 电机驱动
│           ├── motor.h
│           ├── sensor.cpp      ← 传感器
│           ├── sensor.h
│           └── .vscode\
│               └── settings.json
```

---

## 十五、总结

| 以前（Arduino IDE） | 现在（Arduino CLI） |
|---------------------|---------------------|
| 打开 IDE → 等待加载 → 打开项目 → 编译 → 烧录 | 终端输入 `acu` |
| 每次编译 30 秒+ | 编译几秒 |
| 占内存 500MB+ | 几乎不占资源 |
| 只能用 Arduino 编辑器 | 配合任何编辑器 |

**核心命令就三个：**
- `acc` — 编译
- `auc` — 烧录
- `amon` — 看串口

---

*最后更新：2026-06-04*
*作者：子秋*
