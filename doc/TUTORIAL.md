# BleachBit Tutorial

BleachBit is an open-source system cleaner and privacy tool for Windows, Linux, and macOS. It frees disk space, clears browsing history, removes temporary files, and securely shreds sensitive data.

---

## Table of Contents

1. [Installation](#1-installation)
2. [GUI Usage](#2-gui-usage)
3. [CLI Usage](#3-cli-usage)
4. [Common Cleaning Scenarios](#4-common-cleaning-scenarios)
5. [Secure Shredding](#5-secure-shredding)
6. [Custom Cleaning Rules (CleanerML)](#6-custom-cleaning-rules-cleanerml)
7. [Best Practices and Caveats](#7-best-practices-and-caveats)

---

## 1. Installation

> **About this fork**
>
> This repository (`Josh-Chung/bleachbit`) is a personal fork of the upstream BleachBit project. It contains security and code-quality fixes — see the merged PR [#1: Fix XXE vulnerabilities and code quality issues](https://github.com/Josh-Chung/bleachbit/pull/1) — that have **not** been upstreamed.
>
> Only the "Running from source" steps below use this fork (via `git clone`). All pre-built installers and packages — the Windows `.exe`, the portable build, and the Linux distro packages — are produced and published by the original project maintainer at https://www.bleachbit.org and the [upstream repository](https://github.com/bleachbit/bleachbit). To benefit from the security fixes in this fork, you must run from source.

### Windows

Download the installer from: https://www.bleachbit.org/download/windows

Two editions are available:
- **Installer** (`.exe`) — recommended for most users
- **Portable** — no installation required, can run from a USB drive

### Linux

```bash
# Ubuntu / Debian
sudo apt install bleachbit

# Fedora
sudo dnf install bleachbit

# Arch Linux
sudo pacman -S bleachbit
```

### Running from source (Windows + MSYS2)

The BleachBit GUI depends on GTK3. On Windows, the simplest way to run from source is via MSYS2.

#### Step 1: Install MSYS2

Download the installer (~70 MB) from https://www.msys2.org and install to the default path `C:\msys64`.

#### Step 2: Install Python + GTK3 + dependencies

Open the **MSYS2 MINGW64** terminal (make sure you pick MINGW64, not plain MSYS2) and run:

```bash
# Install Python, GTK3, PyGObject, and core dependencies (~400 MB)
pacman -S --noconfirm \
    mingw-w64-x86_64-python \
    mingw-w64-x86_64-python-gobject \
    mingw-w64-x86_64-gtk3 \
    mingw-w64-x86_64-python-pip \
    mingw-w64-x86_64-python-psutil \
    mingw-w64-x86_64-python-requests \
    mingw-w64-x86_64-python-chardet \
    mingw-w64-x86_64-python-pywin32

# Install defusedxml (no pre-built pacman package available)
pip install --break-system-packages defusedxml
```

#### Step 3: Clone the repository

In the MSYS2 MINGW64 terminal, first decide **where to put the source tree**. By default the terminal opens in your MSYS2 home directory (`/home/<msys2-user>/`, which lives at `C:\msys64\home\<msys2-user>\` on the Windows filesystem) — that path is awkward to access from File Explorer or other Windows editors, so it's better to clone into a regular Windows location.

MSYS2 maps Windows drives as `/c/`, `/d/`, etc. For example, `C:\Users\<username>\projects\` becomes `/c/Users/<username>/projects/`.

```bash
# Switch to a convenient Windows directory (create it if needed)
mkdir -p /c/Users/<username>/projects
cd /c/Users/<username>/projects

# Clone the repository
git clone https://github.com/Josh-Chung/bleachbit.git
cd bleachbit
```

The repository is now at `C:\Users\<username>\projects\bleachbit\` on the Windows filesystem.

#### Step 4: Launch BleachBit

```bash
python bleachbit.py          # GUI mode
python bleachbit.py --help   # CLI mode
```

#### Quick launch (from PowerShell or CMD)

If you prefer not to open the MSYS2 terminal every time, you can launch directly from PowerShell.
Replace `<username>` with your Windows username and `C:/path/to/bleachbit` with the actual clone path (e.g. `C:/projects/bleachbit`):

```powershell
C:\msys64\usr\bin\bash.exe -lc "export MSYSTEM=MINGW64 && source /etc/profile && export APPDATA='C:/Users/<username>/AppData/Roaming' && export LOCALAPPDATA='C:/Users/<username>/AppData/Local' && cd 'C:/path/to/bleachbit' && python bleachbit.py"
```

> `APPDATA` and `LOCALAPPDATA` must be set explicitly because MSYS2 does not inherit these Windows environment variables automatically.

You can also save the command as a `.bat` file for double-click convenience:

```batch
@echo off
C:\msys64\usr\bin\bash.exe -lc "export MSYSTEM=MINGW64 && source /etc/profile && export APPDATA='C:/Users/<username>/AppData/Roaming' && export LOCALAPPDATA='C:/Users/<username>/AppData/Local' && cd 'C:/path/to/bleachbit' && python bleachbit.py"
```

#### Notes

- The `intl-8.dll` warning at startup is harmless (missing internationalization library, does not affect functionality)
- The message `Missing optional Python packages: plyer` is also harmless (plyer is only used for desktop notifications)
- After editing source code, simply restart — no rebuild needed

### Running from source (Linux)

```bash
# Ubuntu / Debian
sudo apt install python3-gi gir1.2-gtk-3.0 python3-pip

# Clone and run
git clone https://github.com/Josh-Chung/bleachbit.git
cd bleachbit
pip install -r requirements.txt
python3 bleachbit.py          # GUI mode
python3 bleachbit.py --help   # CLI mode
```

---

## 2. GUI Usage

### Interface Layout

```
┌──────────────────────────────────────────────────┐
│  Menu bar: File / Edit / Help                    │
├────────────────┬─────────────────────────────────┤
│                │                                 │
│  Left panel    │  Right panel                    │
│  (Cleaner list)│  (Action log / Preview results) │
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
│  Toolbar: [Preview]  [Clean]  [Abort]            │
│  Status bar: Disk space recovered: 152MB         │
└──────────────────────────────────────────────────┘
```

### Basic Workflow

**Step 1: Select items to clean**

Check the items you want to clean in the left panel. Cleaners are grouped by application:
- **Browsers**: Firefox, Chrome, Edge, Brave, etc. — cache, cookies, history
- **System**: temporary files, logs, recycle bin, clipboard
- **Applications**: Office, VLC, Adobe Reader, etc. — MRU (most recently used) lists

**Step 2: Preview**

Click the **Preview** button on the toolbar. BleachBit scans but **does not delete anything**. The right panel shows the files that would be cleaned and the estimated space to be freed.

> **Always preview before cleaning!** Make sure nothing important is selected by mistake.

**Step 3: Clean**

After reviewing the preview, click the **Clean** button to perform the actual cleanup.

**Step 4: Review results**

When cleaning finishes, the status bar shows:
- `Disk space recovered: XXX MB` — space freed
- `Files deleted: XXX` — number of files removed

### Preferences

Open via `Edit → Preferences`:

| Setting | Description | Recommendation |
|---------|-------------|----------------|
| Overwrite files | Securely overwrite file contents before deleting | Off for routine cleaning; on for sensitive data |
| Check for updates | Automatically check for new versions | Recommended on |
| Dark mode | Dark theme | Personal preference |
| Units (IEC) | Use KiB/MiB instead of kB/MB | Personal preference |

---

## 3. CLI Usage

CLI mode does not require GTK and works on headless servers.

### Basic Syntax

```
python bleachbit.py [options] cleaner.option [cleaner.option ...]
```

### Core Commands

#### Show help

```bash
python bleachbit.py --help
```

#### List all available cleaners

```bash
python bleachbit.py --list-cleaners
```

Example output:
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

> About 260 cleaning options covering 60+ applications.

#### Preview (dry run)

```bash
# Preview Firefox cache cleanup
python bleachbit.py --preview firefox.cache

# Preview multiple items
python bleachbit.py --preview firefox.cache system.tmp google_chrome.cache

# Preview all options for an application (wildcard)
python bleachbit.py --preview firefox.*
```

#### Clean

```bash
# Clean Firefox cache and system temp files
python bleachbit.py --clean firefox.cache system.tmp

# Clean all options for an application
python bleachbit.py --clean firefox.*

# Use saved preset from the GUI
python bleachbit.py --clean --preset

# Clean all non-warning options
python bleachbit.py --clean --all-but-warning

# Clean all except specific items
python bleachbit.py --clean --all-but-warning --except system.empty_space
```

#### Securely shred files

```bash
# Securely shred specific files (overwrite + rename + delete)
python bleachbit.py --shred secret.txt passwords.db

# Shred an entire directory
python bleachbit.py --shred C:\Users\me\old-secrets\
```

#### Wipe free disk space

```bash
# Overwrite free disk space with zeros to prevent recovery of deleted files
python bleachbit.py --wipe-empty-space C:\

# Linux
python bleachbit.py --wipe-empty-space /home
```

> **Warning:** This operation takes a long time (depends on free space) and writes heavily to disk. Use with caution on SSDs.

#### Overwrite mode

```bash
# Overwrite file contents during cleaning (instead of simple deletion)
python bleachbit.py --clean --overwrite firefox.cache system.tmp
```

#### Other commands

```bash
# Show version
python bleachbit.py --version

# Show system info
python bleachbit.py --sysinfo

# Enable debug logging
python bleachbit.py --debug --clean system.tmp

# Write debug log to file
python bleachbit.py --debug-log debug.txt --clean system.tmp
```

### Windows-specific commands

```bash
# Update winapp2.ini (community cleaning rules)
python bleachbit.py --update-winapp2

# Skip the UAC admin prompt
python bleachbit.py --no-uac --clean system.tmp
```

---

## 4. Common Cleaning Scenarios

### Scenario 1: Quick disk space recovery

Goal: Clear all browser caches and system temp files.

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

### Scenario 2: Clear browsing traces (privacy)

Goal: Remove history, cookies, and form data from all browsers.

```bash
python bleachbit.py --clean --overwrite ^
    firefox.cache firefox.cookies firefox.history firefox.form_history ^
    google_chrome.cache google_chrome.cookies google_chrome.history google_chrome.form_history ^
    microsoft_edge.cache microsoft_edge.cookies microsoft_edge.history microsoft_edge.form_history
```

### Scenario 3: Clean up development artifacts

Goal: Remove Python caches, Node.js modules, editor temp files.

```bash
python bleachbit.py --clean ^
    deepscan.pycache ^
    deepscan.node_modules ^
    deepscan.tmp ^
    deepscan.venv ^
    deepscan.thumbs_db
```

> **Note:** The `deepscan` cleaner performs a deep directory-tree scan and may take a while.

### Scenario 4: Securely destroy files

```bash
# Securely shred (overwrite + rename + delete)
python bleachbit.py --shred "C:\Users\me\Documents\tax-2024.xlsx"
```

### Scenario 5: Scheduled automatic cleaning (Windows Task Scheduler)

Create a batch file `daily_clean.bat`:

```batch
@echo off
python bleachbit.py --clean --preset --no-uac
```

Then add it as a scheduled task in Windows Task Scheduler.

---

## 5. Secure Shredding

BleachBit provides three levels of secure deletion:

| Level | Command | Description |
|-------|---------|-------------|
| Normal delete | `--clean` | Deletes files (recoverable with forensic tools) |
| Overwrite delete | `--clean --overwrite` | Overwrites contents with zeros before deleting |
| Secure shred | `--shred` | Overwrites contents, randomly renames, then deletes |

### How it works

1. **Content overwrite** (`wipe_contents`): Fills the entire file with null bytes (`\x00`)
2. **Filename wipe** (`wipe_name`): Randomly renames the file multiple times to erase the original filename from the filesystem
3. **Free space wipe** (`wipe-empty-space`): Fills free disk space with zeros to overwrite residual data from previously deleted files

> Per NIST SP 800-88, a single overwrite pass is sufficient to effectively sanitize modern storage media.

---

## 6. Custom Cleaning Rules (CleanerML)

BleachBit supports custom cleaning rules defined via XML files.

### Rule file locations

- **Windows**: `%APPDATA%\BleachBit\cleaners\`
- **Linux**: `~/.config/bleachbit/cleaners/`

### Example: Clean a custom application's cache

Create a file named `my_app.xml`:

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

Save the file, restart BleachBit, and the new cleaner appears in the list.

### Common CleanerML actions

| Action | Description |
|--------|-------------|
| `delete` | Delete matching files |
| `shred` | Securely shred matching files |
| `truncate` | Truncate files to 0 bytes (preserves the file) |
| `clean.ini` | Remove specified sections from INI files |
| `clean.json` | Remove specified keys from JSON files |
| `sqlite.vacuum` | Compact a SQLite database |

---

## 7. Best Practices and Caveats

### Recommended practices

1. **Always preview before cleaning** — Use `--preview` to verify what will be deleted
2. **Close target applications** — Close browsers before cleaning their data, otherwise it may fail
3. **Back up important data** — Make a backup before your first run, just in case
4. **Use overwrite selectively** — Routine cleaning does not need overwrite; reserve it for sensitive data
5. **Keep winapp2.ini updated** — Windows users can get additional community-contributed cleaning rules

### Caveats

1. **Do not check options you do not understand** — Especially `Passwords`, `Session`, etc., which affect login state
2. **SSDs do not need free-space wiping** — SSDs have TRIM; `--wipe-empty-space` reduces SSD lifespan with no benefit
3. **Clearing cookies logs you out** — After cleaning cookies, you must re-login to all websites
4. **`system.empty_space` takes a long time** — This operation fills the entire free disk space; use with care
5. **Admin privileges** — Some system cleaning options require administrator access

### Supported cleaner categories

| Category | Applications |
|----------|-------------|
| Browsers | Firefox, Chrome, Edge, Brave, Opera, Vivaldi, Safari, Waterfox, etc. |
| Communication | Slack, Discord, Skype, Thunderbird, Pidgin, etc. |
| Office | LibreOffice, Microsoft Office, Adobe Reader, etc. |
| Media | VLC, WinAmp, Zoom, etc. |
| Development | DeepScan (Python cache, node_modules, .venv, etc.) |
| System | Temp files, logs, recycle bin, clipboard, MRU, thumbnail cache, etc. |

---

*This tutorial is based on BleachBit v6.0.1. For more information visit https://www.bleachbit.org/documentation*
