# WorkBuddy + Obsidian 实现数据云端同步（操作手册）

> 更新时间：2026-10-14
> 适用场景：两台 Windows 电脑之间同步 Obsidian 笔记库
> 说明：本文档中 GitHub 用户名、邮箱、仓库名等个人信息已脱敏为占位符

---

## 一、目标与原理

**目标**：让两台 Windows 电脑上的 Obsidian 笔记保持同步，任何一台改动了笔记，另一台能自动更新。

**原理**（三层结构）：

```
Obsidian 笔记库 (本地)
      │  Obsidian Git 插件（自动定时 pull / push）
      ▼
本地 Git 仓库（.git 目录）
      │  git push / pull
      ▼
GitHub 私有仓库（云端中转）
```

- 本地 Obsidian 库用 Git 管理版本；
- GitHub 私有仓库作为云端中转站；
- Obsidian Git 插件负责定时自动「拉取（pull）」和「提交并推送（backup）」。

---

## 二、环境信息（本次实际配置）

| 项目 | 值 |
|------|-----|
| 远程仓库地址 | `https://github.com/<GITHUB用户名>/<仓库名>.git` |
| 仓库类型 | 私有仓库 |
| GitHub 用户名 | `<GITHUB用户名>` |
| Git 提交姓名 | `<GITHUB用户名>` |
| Git 提交邮箱 | `<邮箱地址>` |
| 本机 Obsidian 库路径 | `C:\Obsidian\obsidianvault\myvault` |
| Git for Windows 安装路径 | `C:\Obsidian\Git`（**自定义路径，非默认**） |
| Obsidian Git 插件版本 | 2.41.1（作者 Vinzent / Vinzent03） |
| 默认分支 | `main` |
| 同步范围 | 仅笔记类文件（md/txt/topo/json 等），排除 PDF/压缩包/Word/exe |

---

## 三、第一台电脑：完整配置步骤

### 第 1 步：安装 Git for Windows

1. 下载安装包（官方 https://git-scm.com/download/win 或国内镜像，如清华镜像）。
2. 双击安装，全程保持默认，**确保勾选 Git Credential Manager**（用于安全托管凭据）。
3. 安装完成后验证：

```bash
git --version
where git
```

> 注：本机安装时把路径改到了 `C:\Obsidian\Git`，所以后续插件里要填 `C:\Obsidian\Git\bin\git.exe`。如果你按默认安装，路径是 `C:\Program Files\Git\bin\git.exe`。

### 第 2 步：配置 Git 全局用户信息

```bash
git config --global user.name "<GITHUB用户名>"
git config --global user.email "<邮箱地址>"
```

### 第 3 步：创建 `.gitignore`（库根目录）

忽略临时文件、易冲突文件，以及**大文件学习资料**（PDF、压缩包、Word 等不进 git）：

```
.trash/
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/workspace*.json
.obsidian/cache/
.obsidian/graph.json

# 压缩包（大文件，不纳入版本控制）
*.zip
*.7z
*.rar
*.tar
*.tar.gz
*.tgz
*.gz

# 大文件学习资料（不纳入版本控制，建议改用网盘同步）
*.pdf
*.docx
*.doc
*.xlsx
*.xls
*.pptx
*.ppt
*.exe
*.msi
```

> 💡 **同步范围约定**：本仓库只同步**笔记类文件**（`.md` / `.txt` / `.topo` / `.json` / `.canvas` 等）。学习资料（PDF、压缩包、Word、exe）体积大、不适合 Git 同步，一律留在本地，建议改用网盘同步。

### 第 4 步：创建 `.gitattributes`（库根目录）

统一换行符，避免跨平台冲突：

```
* text=auto eol=lf
*.md text eol=lf
*.canvas text eol=lf
*.json text eol=lf
*.js text eol=lf
*.css text eol=lf
*.png binary
*.jpg binary
*.jpeg binary
```

### 第 5 步：初始化仓库并设默认分支为 main

```bash
cd "C:\Obsidian\obsidianvault\myvault"
git init
git branch -M main
```

### 第 6 步：添加远程仓库

```bash
git remote add origin https://github.com/<GITHUB用户名>/<仓库名>.git
```

### 第 7 步：首次提交

```bash
git add .
git commit -m "init: first commit"
```

### 第 8 步：推送到 GitHub（认证）

```bash
git push -u origin main
```

- 首次推送会弹出 **Git Credential Manager** 登录窗口；
- 推荐直接用 GitHub 账号登录（登录一次后凭据安全保存，以后免密）；
- 或用 PAT 方式：用户名填 `<GITHUB用户名>`，密码栏粘贴 PAT。

> 🔒 **安全提醒**：不要把 PAT 写进 `git remote set-url https://用户名:PAT@...` 这种形式（会明文存进 `.git/config`）。交给凭据管理器或命令行提示符即可。

### 第 9 步：安装 Obsidian Git 插件

**方式 A：插件市场安装（正常网络下）**

设置（左下角齿轮 ⚙️）→ 第三方插件 → 关闭「安全模式」→「浏览」→ 搜索 `obsidian-git` 或 `git` → 安装 → 启用。

> ⚠️ 注意：该插件在市场上**作者显示名是「Vinzent」，插件名显示是「Git」**，搜「Vinzent03」是搜不到的。

**方式 B：手动安装（市场打不开 / 搜不到时）**

从 GitHub Release 下载三个文件，放到插件目录：

```
C:\Obsidian\obsidianvault\myvault\.obsidian\plugins\obsidian-git\
├── main.js
├── manifest.json
└── styles.css
```

下载地址（版本号按实际替换）：

```
https://github.com/Vinzent03/obsidian-git/releases/download/2.41.1/main.js
https://github.com/Vinzent03/obsidian-git/releases/download/2.41.1/manifest.json
https://github.com/Vinzent03/obsidian-git/releases/download/2.41.1/styles.css
```

放好后**完全重启 Obsidian**，再到「第三方插件 → 已安装插件」里启用「Git」。

### 第 10 步：配置插件

点「Git」插件旁的齿轮进入设置：

| 设置项 | 值 | 说明 |
|--------|-----|------|
| Git executable path | `C:\Obsidian\Git\bin\git.exe` | Git 可执行文件路径（按你实际安装路径填） |
| Auto backup interval (minutes) | `30` | 每 30 分钟自动提交 + 推送 |
| Auto pull interval (minutes) | `10` | 每 10 分钟自动拉取 |
| Pull updates on startup | 开 | 打开 Obsidian 时先拉最新，减少冲突 |

### 第 11 步：验证

按 `Ctrl + P` 打开命令面板：

1. 执行 `Git: Pull` —— 应提示拉取成功；
2. 执行 `Git: Backup` —— 应提示提交并推送成功。

之后去 GitHub 网页刷新 `<仓库名>` 仓库，能看到笔记已在云端。

---

## 四、第二台电脑：部署步骤

换到第二台 Windows 电脑时，重复以下三步：

1. **装 Git for Windows**（同上第 1 步），并配置全局用户信息（第 2 步）。
2. **装 Obsidian Git 插件**（第 9 步，市场装或手动装均可）。
3. **拉取仓库**：
   - 先用 Obsidian 随便打开/新建一个临时空库（比如默认的 My-Vault）；
   - 按 `Ctrl + P` 执行 `Git: Clone an existing remote repo`；
   - 在弹出的 **「Enter directory for clone」** 输入框里，填一个**全新的、不存在的路径**（要求目录为空或不存在），例如：
     ```
     C:\Obsidian\obsidianvault\myvault
     ```
   - 回车后插件会自动创建目录并克隆，完成后**确认切换到新库**即可；
   - 之前那个临时库（只有“欢迎”笔记）可以关闭删除，不影响。

   > ⚠️ **不能填当前已打开库自身的路径**——目录非空会被拒绝，报 "It needs to be empty or not exist"。

之后配置插件（Git 路径 + 自动同步时间），两台电脑就会按设定间隔自动互相同步。

> ⚠️ **注意**：Git 只同步笔记类文件。第二台电脑如果本地也有学习资料（PDF/压缩包/Word 等），这些**不会**通过 Git 同步，需要另外用网盘等方式在两台电脑间手动同步。

---

## 五、注意事项

- **PAT 安全**：PAT 全程不要明文写入任何文件（`.git/config`、脚本、配置文件都不行），只通过 Git Credential Manager 或命令提示符输入。
- **网络问题**：GitHub 直连在国内常不稳定。若 push/pull 报 `Failed to connect to github.com:443` 或超时，需开启代理工具，并配置 Git 走代理（仅对 GitHub 生效）：

```bash
git config --global http.https://github.com.proxy http://127.0.0.1:端口号
```

- **换行符**：`.gitattributes` 里的 `eol=lf` 统一了换行符，避免 Windows 与 Linux 间出现大量无意义 diff。
- **大文件不要进 Git**：GitHub 单文件上限 100MB、仓库建议 <1GB。PDF、压缩包、Word、exe 等学习资料务必保持被 `.gitignore` 忽略，改用网盘同步。若不小心把大文件提交进去了，会导致 push 一直失败（报 502/408 或超时）。
- **冲突处理**：两台电脑同时改同一篇笔记可能产生冲突。建议养成「打开先 pull、改完及时 backup」的习惯，或开启 `Pull updates on startup`。

---

## 六、常见问题排查（FAQ）

| 现象 | 原因 | 解决 |
|------|------|------|
| 插件市场搜不到插件 | 搜索词不对 / 市场未加载 | 搜 `git` 或 `obsidian-git`；或手动安装（方式 B） |
| `Git: Pull` 没反应 / 报连接失败 | 网络连不上 GitHub | 开启代理并配置 Git 代理；或等网络恢复 |
| push 报 502 / 408 / 超时 | 库里有大文件（超 GitHub 100MB 上限）或网络不稳 | 确认大文件已被 `.gitignore` 排除；排除后重试 |
| 提示需要账号密码 | 凭据未保存 | 首次 push 时用 GCM 登录一次；PAT 用提示符粘贴 |
| 插件找不到 Git | Git 路径填错 | 在插件设置里填 `where git` 查到的完整路径 |
| 大量文件被标记改动 | 换行符不一致 | 确认 `.gitattributes` 存在并重新 `git add --renormalize .` |
| Clone 时报 "It needs to be empty or not exist" | 克隆目标目录非空 | 输入框里填一个**全新的、不存在的路径**，不要填当前已打开库的路径 |
