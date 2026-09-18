# Windows 文件链接

Windows 中常见的“文件链接”有硬链接、符号链接和 Junction（目录联接）。

## 类型概览

| 类型 | Windows 命令 | 适用对象 | 是否可跨磁盘或卷 |
| --- | --- | --- | --- |
| 硬链接（Hard Link） | `mklink /H` | 文件 | 不可以 |
| 符号链接（Symbolic Link） | `mklink` 或 `mklink /D` | 文件或目录 | 可以 |
| Junction（目录联接） | `mklink /J` | 目录 | 通常不可以跨卷 |

## 硬链接

硬链接是多个文件名指向同一份文件数据。它只能用于文件，不能用于目录，也不能跨磁盘分区或卷。

```powershell
cmd /c mklink /H link.txt target.txt
```

特点：

- 链接和原文件地位相同，没有严格意义上的“原文件”和“链接文件”。
- 删除其中一个文件名不会删除实际数据，直到所有硬链接都被删除。
- 只能在同一个文件系统卷内创建。
- 目标文件必须已经存在。

## 符号链接

符号链接类似 Unix/Linux 中的 symlink，可以指向文件或目录，也可以使用相对路径。

```powershell
# 创建文件符号链接
cmd /c mklink link.txt target.txt

# 创建目录符号链接
cmd /c mklink /D LinkDir TargetDir
```

特点：

- 可以链接文件，也可以链接目录。
- 可以跨磁盘、跨卷，甚至指向网络路径。
- 目标不存在时也可以创建，但访问时会失效。
- 删除符号链接通常不会删除目标内容。
- 某些情况下需要管理员权限，或者需要开启 Windows 开发者模式。

## （目录联接）

Junction 是 Windows 特有的目录链接机制，只能用于目录，常用于把某个目录迁移到其他位置，同时保持原路径可用。

```powershell
cmd /c mklink /J LinkDir TargetDir
```

特点：

- 只能链接目录，不能链接单个文件。
- 兼容性通常比目录符号链接更好。
- 常用于缓存目录、用户数据目录或程序目录迁移。
- 一般不需要管理员权限。
- 通常只能指向同一磁盘卷中的目录。

## PowerShell 写法

PowerShell 可以使用 `New-Item` 创建链接：

```powershell
# 文件符号链接
New-Item -ItemType SymbolicLink -Path link.txt -Target target.txt

# 目录符号链接
New-Item -ItemType SymbolicLink -Path LinkDir -Target TargetDir

# Junction
New-Item -ItemType Junction -Path LinkDir -Target TargetDir

# 文件硬链接
New-Item -ItemType HardLink -Path link.txt -Target target.txt
```

## 删除链接

删除链接时，只删除链接本身，不要把目标路径当作普通目录递归删除。

```powershell
# 删除文件链接或硬链接
Remove-Item .\link.txt

# 删除目录符号链接或 Junction
Remove-Item .\LinkDir
```

使用资源管理器删除链接目录时也要确认目标类型，避免误操作目标内容。

## 如何选择

- 需要链接单个文件，并且目标与链接必须在同一个卷：选择**硬链接**。
- 需要跨卷、跨磁盘或链接网络路径：选择**符号链接**。
- 需要把一个目录映射到另一个目录，并重视旧程序兼容性：选择**Junction**。
## 常用检查命令

可以使用 `dir` 查看链接类型：

```powershell
cmd /c dir
```

也可以使用 PowerShell 查看项目类型和目标：

```powershell
Get-Item .\LinkDir | Select-Object FullName, LinkType, Target
```

## 注意事项

1. 创建链接前确认目标路径，尤其是在系统目录和程序目录下操作时。
2. 相对符号链接依赖当前目录结构，移动链接或目标目录后可能失效。
3. 不要把快捷方式、符号链接和 Junction 混为一谈，它们的程序兼容性不同。
4. 对目录链接执行复制、删除或备份操作时，先确认工具是否会跟随链接访问目标目录。
