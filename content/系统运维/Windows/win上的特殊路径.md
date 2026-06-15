Windows 里有一些**特殊路径变量/环境变量**，它们是系统内置的快捷方式，可以在文件资源管理器、命令行、PowerShell 等地方直接使用。

---

## 一、几种不同类型的"特殊路径"

### 1. PowerShell 变量 —— `$PROFILE`

`$PROFILE` 是 PowerShell 专属变量，指向当前用户的 **PowerShell 配置文件路径**，类似于 Linux 的 `~/.bashrc`。

```powershell
# 查看路径
$PROFILE
# 通常是：C:\Users\你的用户名\Documents\PowerShell\Microsoft.PowerShell_profile.ps1

# 打开/编辑配置文件
notepad $PROFILE
```
`$PROFILE` 本身只是一个字符串（路径），但 PowerShell 里它其实是一个**有多个属性的对象**，可以用 `.` 访问不同的配置文件路径。

#### `$PROFILE` 的四个属性

```powershell
$PROFILE.CurrentUserCurrentHost    # 默认，就是 $PROFILE 本身
$PROFILE.CurrentUserAllHosts       # 当前用户，所有 Host 都生效
$PROFILE.AllUsersCurrentHost       # 所有用户，当前 Host
$PROFILE.AllUsersAllHosts          # 所有用户，所有 Host
```

#### 具体路径

| 属性                       | 路径                                                               |
| ------------------------ | ---------------------------------------------------------------- |
| `CurrentUserCurrentHost` | `Documents\PowerShell\Microsoft.PowerShell_profile.ps1`          |
| `CurrentUserAllHosts`    | `Documents\PowerShell\profile.ps1`                               |
| `AllUsersCurrentHost`    | `C:\Program Files\PowerShell\7\Microsoft.PowerShell_profile.ps1` |
| `AllUsersAllHosts`       | `C:\Program Files\PowerShell\7\profile.ps1`                      |

---

#### "Host" 是什么意思？

Host 指的是**运行 PowerShell 的宿主程序**：

- `Microsoft.PowerShell_profile.ps1` → 只在**终端/pwsh** 里生效
- `profile.ps1` → 在**所有 Host** 生效，包括 VS Code 的集成终端、ISE 等

所以如果你在 VS Code 里用 PowerShell，`AllHosts` 的配置文件也会被加载。

---

#### 常用操作

```powershell
# 查看所有四个路径
$PROFILE | Format-List *

code $PROFILE        # 用 VS Code 打开

# 重新加载配置文件（不用重启终端）
. $PROFILE
```

---

#### 加载顺序

每次启动 PowerShell，配置文件按这个顺序加载：

```
AllUsersAllHosts          ← 最先
AllUsersCurrentHost
CurrentUserAllHosts
CurrentUserCurrentHost    ← 最后（优先级最高）
```

后加载的可以覆盖前面的设置。

---

### 2. 环境变量 —— `%变量名%`

在**命令提示符（CMD）**和**资源管理器地址栏**中使用：

| 变量                         | 指向路径                              | 说明             |
| -------------------------- | --------------------------------- | -------------- |
| `%USERPROFILE%`            | `C:\Users\用户名`                    | 当前用户主目录        |
| `%APPDATA%`                | `C:\Users\用户名\AppData\Roaming`    | 应用漫游数据         |
| `%LOCALAPPDATA%`           | `C:\Users\用户名\AppData\Local`      | 应用本地数据         |
| `%TEMP%` / `%TMP%`         | `C:\Users\用户名\AppData\Local\Temp` | 临时文件           |
| `%SYSTEMROOT%`             | `C:\Windows`                      | Windows 系统目录   |
| `%WINDIR%`                 | `C:\Windows`                      | 同上             |
| `%SYSTEMDRIVE%`            | `C:`                              | 系统所在盘符         |
| `%PROGRAMFILES%`           | `C:\Program Files`                | 64位程序安装目录      |
| `%PROGRAMFILES(X86)%`      | `C:\Program Files (x86)`          | 32位程序安装目录      |
| `%COMMONPROGRAMFILES%`     | `C:\Program Files\Common Files`   | 公共组件目录         |
| `%PUBLIC%`                 | `C:\Users\Public`                 | 所有用户共享目录       |
| `%COMPUTERNAME%`           | 你的电脑名                             | 计算机名称          |
| `%USERNAME%`               | 你的用户名                             | 当前登录用户名        |
| `%HOMEDRIVE%`              | `C:`                              | 用户主目录所在盘       |
| `%HOMEPATH%`               | `\Users\用户名`                      | 用户主目录路径部分      |
| `%PATH%`                   | 一串路径                              | 系统可执行文件搜索路径    |
| `%PATHEXT%`                | `.COM;.EXE;.BAT...`               | 可执行文件扩展名列表     |
| `%OS%`                     | `Windows_NT`                      | 操作系统类型         |
| `%PROCESSOR_ARCHITECTURE%` | `AMD64` 等                         | CPU 架构         |
| `%NUMBER_OF_PROCESSORS%`   | 数字                                | 逻辑处理器数量        |
| `%ONEDRIVE%`               | OneDrive 本地路径                     | 如果安装了 OneDrive |

---

### 3. Shell 特殊文件夹 —— 在地址栏输入

在**资源管理器地址栏**直接输入这些名称，可以快速跳转：

| 输入内容                   | 跳转到             |
| ---------------------- | --------------- |
| `shell:startup`        | 当前用户启动文件夹       |
| `shell:common startup` | 所有用户启动文件夹       |
| `shell:desktop`        | 桌面              |
| `shell:downloads`      | 下载文件夹           |
| `shell:personal`       | 我的文档            |
| `shell:my pictures`    | 图片              |
| `shell:my music`       | 音乐              |
| `shell:my video`       | 视频              |
| `shell:sendto`         | 右键"发送到"的目标文件夹   |
| `shell:fonts`          | 字体文件夹           |
| `shell:appdata`        | AppData\Roaming |
| `shell:programs`       | 开始菜单程序文件夹       |
| `shell:recent`         | 最近使用的文件         |
| `shell:cookies`        | IE/Edge Cookie  |
| `shell:history`        | 浏览历史            |

---

### 4. PowerShell 中的环境变量写法

PowerShell 里访问环境变量用 `$env:` 前缀：

```powershell
$env:USERPROFILE      # C:\Users\你的用户名
$env:APPDATA          # C:\Users\你的用户名\AppData\Roaming
$env:TEMP             # 临时目录
$env:PATH             # 系统PATH
$env:COMPUTERNAME     # 电脑名
```

---

## 二、实际使用技巧

```powershell
# 快速打开 AppData（很多软件配置在这里）
explorer %APPDATA%

# 快速打开启动项文件夹（开机自启的程序放这里）
explorer shell:startup

# 查看所有环境变量
Get-ChildItem Env:         # PowerShell
set                        # CMD

# 在 CMD 中用变量拼接路径
cd %USERPROFILE%\Desktop
```

---

## 三、总结对比

| 类型              | 格式         | 适用场合             |
| --------------- | ---------- | ---------------- |
| PowerShell 变量   | `$PROFILE` | PowerShell 脚本/终端 |
| 环境变量            | `%VAR%`    | CMD、资源管理器、批处理    |
| PowerShell 环境变量 | `$env:VAR` | PowerShell 脚本/终端 |
| Shell 特殊文件夹     | `shell:名称` | 资源管理器地址栏、Run 对话框 |
