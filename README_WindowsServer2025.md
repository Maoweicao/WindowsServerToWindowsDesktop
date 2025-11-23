# Windows Server To Windows Desktop 脚本适配说明

## 概述
本文档说明了如何将原有的 "Windows Server To Windows Desktop" 脚本适配到 Windows Server 2025。主要修改是替换已弃用的 WMIC 命令为 PowerShell CIM 命令。

## 修改详情

### 1. 操作系统标题获取
**原命令:**
```bat
for /f "skip=1 delims=" %%t in ('wmic os get caption') do (
  if not defined caption set caption=%%t
)
```

**新命令:**
```bat
for /f "delims=" %%t in ('PowerShell /Command "&{(Get-CimInstance -ClassName Win32_OperatingSystem).Caption}"') do set caption=%%t
```

### 2. 设置用户密码永不过期
**原命令:**
```bat
wmic Path Win32_UserAccount Where Name="%currentuser%" Set PasswordExpires="FALSE"
```

**新命令:**
```bat
PowerShell /Command "&{$user = Get-CimInstance -ClassName Win32_UserAccount -Filter \"Name = '%currentuser%'\"; Invoke-CimMethod -InputObject $user -MethodName SetPasswordExpires -Arguments @{PasswordExpires=$false}}"
```

### 3. 显示用户密码过期信息
**原命令:**
```bat
wmic useraccount get Name,PasswordExpires
```

**新命令:**
```bat
PowerShell /Command "&{Get-CimInstance -ClassName Win32_UserAccount | Select-Object Name,PasswordExpires}"
```

## 为什么需要这些修改？

Windows Server 2025 不再包含 WMIC (Windows Management Instrumentation Command-line) 工具，因为 Microsoft 已经弃用了该工具并推荐使用 PowerShell 的 CIM cmdlet 作为替代方案。

PowerShell 的 `Get-CimInstance` 和相关 cmdlet 提供了与 WMIC 相同的功能，但具有更好的性能和更一致的语法。

## 兼容性

修改后的脚本应该可以在以下版本的 Windows Server 上正常运行：
- Windows Server 2025
- Windows Server 2022
- Windows Server 2019
- Windows Server 2016

## 使用说明

1. 确保 PowerShell 已启用并可执行
2. 以管理员身份运行脚本
3. 按照提示操作

## 注意事项

- 脚本需要管理员权限才能执行所有操作
- 某些操作可能需要重启计算机
- 建议在生产环境中使用前先在测试环境中验证