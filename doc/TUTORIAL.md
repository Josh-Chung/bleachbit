# BleachBit 使用教程

BleachBit 是一款开源的系统清理与隐私保护工具，支持 Windows、Linux 和 macOS。它可以释放磁盘空间、清除浏览记录、删除临时文件，并安全擦除敏感数据。

---

## 目录

1. [安装](#1-安装)
2. [GUI 图形界面使用](#2-gui-图形界面使用)
3. [CLI 命令行使用](#3-cli-命令行使用)
4. [常用清理场景](#4-常用清理场景)
5. [安全擦除（Shred）](#5-安全擦除shred)
6. [自定义清理规则（CleanerML）](#6-自定义清理规则cleanerml)
7. [最佳实践与注意事项](#7-最佳实践与注意事项)

---

## 1. 安装

### Windows

从官网下载安装包：https://www.bleachbit.org/download/windows

有两个版本可选：
- **安装版**（`.exe`）— 推荐大多数用户使用
- **便携版**（portable）— 无需安装，可放在 U 盘中使用

### Linux

```bash
# Ubuntu / Debian
sudo apt install bleachbit

# Fedora
sudo dnf install bleachbit

# Arch Linux
sudo pacman -S bleachbit
```

### 从源码运行（Windows + MSYS2）

BleachBit 的 GUI 依赖 GTK3，在 Windows 上最简单的运行方式是通过 MSYS2。

#### 第一步：安装 MSYS2

从 https://www.msys2.org 下载安装程序（约 70 MB），安装到默认路径 `C:\msys64`。

#### 第二步：安装 Python + GTK3 + 依赖

打开 **MSYS2 MINGW64** 终端（注意选 MINGW64，不是普通 MSYS2），运行：

```bash
# 安装 Python、GTK3、PyGObject 和核心依赖（约 400 MB）
pacman -S --noconfirm \
    mingw-w64-x86_64-python \
    mingw-w64-x86_64-python-gobject \
    mingw-w64-x86_64-gtk3 \
    mingw-w64-x86_64-python-pip \
    mingw-w64-x86_64-python-psutil \
    mingw-w64-x86_64-python-requests \
    mingw-w64-x86_64-python-chardet \
    mingw-w64-x86_64-python-pywin32

# 安装 defusedxml（pacman 中没有预编译包）
pip install --break-system-packages defusedxml
```

#### 第三步：启动 BleachBit

```bash
# 在 MSYS2 MINGW64 终端中
cd /c/path/to/bleachbit
python bleachbit.py          # GUI 模式
python bleachbit.py --help   # CLI 模式
```

#### 快捷启动（从 PowerShell 或 CMD）

如果不想每次都打开 MSYS2 终端，可以直接从 PowerShell 启动：

```powershell
C:\msys64\usr\bin\bash.exe -lc "export MSYSTEM=MINGW64 && source /etc/profile && export APPDATA='C:/Users/<username>/AppData/Roaming' && export LOCALAPPDATA='C:/Users/<username>/AppData/Local' && cd '<bleachbit-source-path>' && python bleachbit.py"
```

> 将 `<username>` 替换为你的 Windows 用户名，`<bleachbit-source-path>` 替换为 BleachBit 源码路径。
> `APPDATA` 和 `LOCALAPPDATA` 需要手动设置，因为 MSYS2 环境不会自动继承这些 Windows 环境变量。

也可以保存为 `.bat` 文件方便双击启动：

```batch
@echo off
C:\msys64\usr\bin\bash.exe -lc "export MSYSTEM=MINGW64 && source /etc/profile && export APPDATA='C:/Users/<username>/AppData/Roaming' && export LOCALAPPDATA='C:/Users/<username>/AppData/Local' && cd '<bleachbit-source-path>' && python bleachbit.py"
```

#### 注意事项

- 启动时出现 `intl-8.dll` 警告是正常的（国际化库缺失，不影响功能）
- 提示 `Missing optional Python packages: plyer` 也是正常的（plyer 仅用于桌面通知）
- 修改源码后无需重新构建，直接重启即可生效

### 从源码运行（Linux）

```bash
# Ubuntu / Debian
sudo apt install python3-gi gir1.2-gtk-3.0 python3-pip

# 克隆并运行
git clone https://github.com/bleachbit/bleachbit.git
cd bleachbit
pip install -r requirements.txt
python3 bleachbit.py          # GUI 模式
python3 bleachbit.py --help   # CLI 模式
```

---

## 2. GUI 图形界面使用

### 界面布局

```
┌──────────────────────────────────────────────────┐
│  菜单栏：File / Edit / Help                       │
├────────────────┬─────────────────────────────────┤
│                │                                 │
│  左侧面板       │  右侧面板                        │
│  (清理器列表)    │  (操作日志 / 预览结果)             │
│                │                                 │
│  ☑ Firefox     │  Preview:                       │
│    ☑ Cache     │  Delete 150MB ~/.cache/firefox/  │
│    ☑ Cookies   │  Delete 2MB ~/.mozilla/cookies   │
│    ☐ Passwords │  ...                            │
│                │                                 │
│  ☑ System      │                                 │
│    ☑ Cache     │                                 │
│    ☑ Logs      │                                 │
│    ☑ Tmp       │                                 │
│                │                                 │
├────────────────┴─────────────────────────────────┤
│  工具栏：[Preview 预览]  [Clean 清理]  [Abort 中止] │
│  状态栏：Disk space recovered: 152MB               │
└──────────────────────────────────────────────────┘
```

### 基本操作流程

**第一步：选择清理项目**

在左侧面板勾选你要清理的内容。清理器按软件分类：
- **浏览器类**：Firefox、Chrome、Edge、Brave 等的缓存、Cookies、历史
- **系统类**：临时文件、日志、回收站、剪贴板
- **应用类**：Office、VLC、Adobe Reader 等的 MRU（最近使用记录）

**第二步：预览（Preview）**

点击工具栏的 **Preview** 按钮。BleachBit 会扫描但 **不删除任何文件**，在右侧显示将被清理的文件列表和预估释放空间。

> 💡 **始终先预览再清理！** 确认没有误选重要文件。

**第三步：清理（Clean）**

确认预览结果后，点击 **Clean** 按钮执行实际清理。

**第四步：查看结果**

清理完成后，底部状态栏显示：
- `Disk space recovered: XXX MB` — 释放的磁盘空间
- `Files deleted: XXX` — 删除的文件数

### 偏好设置（Preferences）

通过菜单 `Edit → Preferences` 打开：

| 设置项 | 说明 | 建议 |
|--------|------|------|
| Overwrite files | 安全覆写文件内容后再删除 | 普通清理关闭，隐私敏感时开启 |
| Check for updates | 自动检查更新 | 建议开启 |
| Dark mode | 深色主题 | 个人偏好 |
| Units (IEC) | 使用 KiB/MiB 而非 kB/MB | 个人偏好 |

---

## 3. CLI 命令行使用

CLI 模式不需要 GTK，可在无图形界面的服务器上使用。

### 基本语法

```
python bleachbit.py [选项] 清理器.选项 [清理器.选项 ...]
```

### 核心命令

#### 查看帮助

```bash
python bleachbit.py --help
```

#### 列出所有可用清理器

```bash
python bleachbit.py --list-cleaners
```

输出示例：
```
firefox.cache
firefox.cookies
firefox.crash_reports
firefox.history
google_chrome.cache
google_chrome.cookies
system.cache
system.logs
system.tmp
...
```

> 共约 260 个清理选项，覆盖 60+ 种软件。

#### 预览（不实际删除）

```bash
# 预览 Firefox 缓存清理
python bleachbit.py --preview firefox.cache

# 预览多个清理项
python bleachbit.py --preview firefox.cache system.tmp google_chrome.cache

# 预览某软件的所有选项（使用通配符 *）
python bleachbit.py --preview firefox.*
```

#### 执行清理

```bash
# 清理 Firefox 缓存和系统临时文件
python bleachbit.py --clean firefox.cache system.tmp

# 清理某软件的全部选项
python bleachbit.py --clean firefox.*

# 使用 GUI 中保存的预设
python bleachbit.py --clean --preset

# 清理所有无警告的选项
python bleachbit.py --clean --all-but-warning

# 清理所有但排除特定项
python bleachbit.py --clean --all-but-warning --except system.empty_space
```

#### 安全粉碎文件

```bash
# 安全粉碎指定文件（覆写内容 + 重命名 + 删除）
python bleachbit.py --shred secret.txt passwords.db

# 粉碎整个文件夹
python bleachbit.py --shred C:\Users\me\old-secrets\
```

#### 擦除磁盘空闲空间

```bash
# 用零覆写磁盘空闲空间，防止已删除文件被恢复
python bleachbit.py --wipe-empty-space C:\

# Linux
python bleachbit.py --wipe-empty-space /home
```

> ⚠️ **此操作耗时很长**（取决于空闲空间大小），且会大量写入磁盘。SSD 用户谨慎使用。

#### 安全覆写模式

```bash
# 清理时覆写文件内容（而非简单删除）
python bleachbit.py --clean --overwrite firefox.cache system.tmp
```

#### 其他命令

```bash
# 查看版本
python bleachbit.py --version

# 查看系统信息
python bleachbit.py --sysinfo

# 开启调试日志
python bleachbit.py --debug --clean system.tmp

# 将调试日志写入文件
python bleachbit.py --debug-log debug.txt --clean system.tmp
```

### Windows 专用命令

```bash
# 更新 winapp2.ini（社区清理规则）
python bleachbit.py --update-winapp2

# 不弹出 UAC 管理员权限提示
python bleachbit.py --no-uac --clean system.tmp
```

---

## 4. 常用清理场景

### 场景 1：快速释放磁盘空间

目标：清理所有浏览器缓存和系统临时文件。

```bash
python bleachbit.py --clean ^
    firefox.cache ^
    google_chrome.cache ^
    microsoft_edge.cache ^
    brave.cache ^
    system.cache ^
    system.tmp ^
    system.logs
```

### 场景 2：清除浏览痕迹（隐私保护）

目标：删除所有浏览器的历史记录、Cookies、表单数据。

```bash
python bleachbit.py --clean --overwrite ^
    firefox.cache firefox.cookies firefox.history firefox.form_history ^
    google_chrome.cache google_chrome.cookies google_chrome.history google_chrome.form_history ^
    microsoft_edge.cache microsoft_edge.cookies microsoft_edge.history microsoft_edge.form_history
```

### 场景 3：清理开发环境

目标：清除 Python 缓存、Node.js 模块、编辑器临时文件。

```bash
python bleachbit.py --clean ^
    deepscan.pycache ^
    deepscan.node_modules ^
    deepscan.tmp ^
    deepscan.venv ^
    deepscan.thumbs_db
```

> ⚠️ `deepscan` 清理器会深度扫描目录树，可能耗时较长。

### 场景 4：安全销毁文件

```bash
# 安全粉碎（覆写 + 重命名 + 删除）
python bleachbit.py --shred "C:\Users\me\Documents\tax-2024.xlsx"
```

### 场景 5：定期自动清理（Windows 任务计划）

创建批处理文件 `daily_clean.bat`：

```batch
@echo off
python bleachbit.py --clean --preset --no-uac
```

然后在 Windows 任务计划程序中添加定时任务即可。

---

## 5. 安全擦除（Shred）

BleachBit 的安全擦除分三个层次：

| 层次 | 命令 | 说明 |
|------|------|------|
| 普通删除 | `--clean` | 仅删除文件（可被恢复工具还原） |
| 覆写删除 | `--clean --overwrite` | 用零覆写文件内容后删除 |
| 安全粉碎 | `--shred` | 覆写内容 → 随机重命名文件 → 删除 |

### 擦除原理

1. **内容覆写**（`wipe_contents`）：用全零 (`\x00`) 覆写文件全部内容
2. **文件名擦除**（`wipe_name`）：将文件名随机重命名多次，消除文件名痕迹
3. **空闲空间擦除**（`wipe-empty-space`）：用零填满磁盘空闲空间，覆盖已删除文件的残留数据

> 📝 根据 NIST SP 800-88 标准，现代存储介质只需一次覆写即可有效清除数据。

---

## 6. 自定义清理规则（CleanerML）

BleachBit 支持通过 XML 文件定义自定义清理规则。

### 规则文件位置

- **Windows**: `%APPDATA%\BleachBit\cleaners\`
- **Linux**: `~/.config/bleachbit/cleaners/`

### 示例：清理自定义应用的缓存

创建文件 `my_app.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<cleaner id="my_app">
  <label>My Application</label>
  <description>Clean cache and logs for My Application</description>

  <option id="cache">
    <label>Cache</label>
    <description>Delete cached data</description>
    <action command="delete"
            search="walk.all"
            path="%LOCALAPPDATA%\MyApp\Cache\" />
  </option>

  <option id="logs">
    <label>Logs</label>
    <description>Delete log files</description>
    <action command="delete"
            search="glob"
            path="%LOCALAPPDATA%\MyApp\Logs\*.log" />
  </option>
</cleaner>
```

保存后重启 BleachBit，新的清理器会出现在列表中。

### CleanerML 常用动作

| 动作 | 说明 |
|------|------|
| `delete` | 删除匹配的文件 |
| `shred` | 安全粉碎匹配的文件 |
| `truncate` | 截断文件为 0 字节（保留文件） |
| `clean.ini` | 清理 INI 文件中的指定节 |
| `clean.json` | 清理 JSON 文件中的指定键 |
| `sqlite.vacuum` | 压缩 SQLite 数据库 |

---

## 7. 最佳实践与注意事项

### ✅ 推荐做法

1. **先预览，再清理** — 使用 `--preview` 确认将删除的内容
2. **关闭目标程序** — 清理浏览器数据前关闭浏览器，否则可能失败
3. **备份重要数据** — 首次使用前备份，防止误删
4. **按需使用 overwrite** — 普通清理不需要覆写，隐私敏感数据才需要
5. **定期更新 winapp2.ini** — Windows 用户可获取社区贡献的更多清理规则

### ⚠️ 注意事项

1. **不要勾选不理解的选项** — 尤其是 `Passwords`、`Session` 等会影响登录状态
2. **SSD 不需要擦除空闲空间** — SSD 有 TRIM 机制，`--wipe-empty-space` 会缩短 SSD 寿命
3. **Cookies 清理会注销网站** — 清理 Cookies 后需要重新登录所有网站
4. **`system.empty_space` 耗时长** — 此操作需要填满整个磁盘空闲空间，谨慎使用
5. **管理员权限** — 某些系统清理项需要管理员权限才能访问

### 支持的清理器分类

| 分类 | 包含软件 |
|------|---------|
| 浏览器 | Firefox, Chrome, Edge, Brave, Opera, Vivaldi, Safari, Waterfox 等 |
| 通讯 | Slack, Discord, Skype, Thunderbird, Pidgin 等 |
| 办公 | LibreOffice, Microsoft Office, Adobe Reader 等 |
| 媒体 | VLC, WinAmp, Zoom 等 |
| 开发 | DeepScan (Python cache, node_modules, .venv 等) |
| 系统 | 临时文件, 日志, 回收站, 剪贴板, MRU, 缩略图缓存 等 |

---

*本教程基于 BleachBit v6.0.1。更多信息请访问 https://www.bleachbit.org/documentation*
