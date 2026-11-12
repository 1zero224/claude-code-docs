# 为 Windows 部署 Claude Desktop

Team 或 Enterprise 计划的管理员可以在整个组织中自动部署 Claude Desktop，以集中管理安装和更新。我们通过 Microsoft Intune、SCCM、Group Policy 或 PowerShell 为 Windows 部署提供 MSIX 包。

## 安装要求

- 对于包含完整功能支持（包括 Claude Cowork）的个人安装，需要管理员权限。用户在安装过程中会看到 Windows UAC 提示。没有管理员访问权限的用户仍然可以安装 Claude，但桌面上的 Cowork 将不可用。从**[我们的下载页面](https://claude.com/download)**访问用户友好的安装程序。

- 对于无需用户交互的静默部署，请直接使用 MSIX 包与您的企业管理工具配合使用。

## Cowork 桌面要求

Claude Desktop for Windows 需要**[虚拟机平台](https://support.microsoft.com/en-us/windows/enable-virtualization-on-windows-c5578302-6e43-4b4b-a449-8ced115f58e1)**才能使用 Cowork。您可以通过大多数端点管理解决方案自动安装此功能，但您也可以运行以下命令手动安装：

```
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -All -NoRestart
```

该功能在机器重启后生效。`-NoRestart` 标志会禁止自动重启以进行静默部署，因此请通过您的管理工具安排重启。

**注意：**您可以在 Cowork 中运行端点检测和响应 (EDR) 传感器（测试版）。详细了解**[在 Claude Cowork 中运行 EDR 传感器](https://support.claude.com/en/articles/17828685-run-your-endpoint-detection-and-response-edr-sensor-in-claude-cowork)**。

## 下载

- **[Claude MSIX (x64)](https://claude.ai/api/desktop/win32/x64/msix/latest/redirect)**

- **[Claude MSIX (arm64)](https://claude.ai/api/desktop/win32/arm64/msix/latest/redirect)**

## 安装命令

Claude MSIX 被打包为按用户应用。`Add-AppxPackage` 仅为当前用户注册它，而 `Add-AppxProvisionedPackage` 在机器范围内暂存它，使其对设备上的每个用户（包括没有管理员权限的标准用户）都可用。

### 为单个用户安装

```
Add-AppxPackage -Path "Claude.msix"
```

有关更多详情，请参阅 Microsoft 的**[Add-AppxPackage](https://learn.microsoft.com/en-us/powershell/module/appx/add-appxpackage?view=windowsserver2022-ps)** 文档。

### 为所有用户安装（机器范围内配置）

```
Add-AppxProvisionedPackage -Online -PackagePath "Claude.msix" -SkipLicense -Regions "all"
```

有关更多详情，请参阅 Microsoft 的**[Add-AppxProvisionedPackage](https://learn.microsoft.com/en-us/powershell/module/dism/add-appxprovisionedpackage?view=windowsserver2022-ps)** 文档。

## 通过 MDM 部署

**重要：**由于 MSIX 按用户打包，将其作为业务线 (LOB) 应用上传到 Intune 会在用户上下文中安装它，而不是机器范围内。对于没有管理员权限的标准用户，安装会失败。对于机器范围内的部署，请改为针对该包运行 `Add-AppxProvisionedPackage`——要么在用户登录前在目标设备上预先暂存它，要么通过 Win32 应用或 PowerShell 脚本包装器传递命令。Intune LOB 无法代表您运行此命令。

Claude Desktop 可以通过各种企业软件分发服务进行部署。选择与您组织现有基础设施相符的方法：

- **[Microsoft Intune](https://docs.microsoft.com/en-us/windows/msix/desktop/managing-your-msix-deployment-intune)**

- **[Microsoft Endpoint Configuration Manager (SCCM)](https://learn.microsoft.com/en-us/windows/msix/desktop/managing-your-msix-deployment-mem-adminconsole)**

- **[Group Policy 软件安装](https://learn.microsoft.com/en-us/troubleshoot/windows-server/group-policy/use-group-policy-to-install-software)**

- **[部署映像服务和管理 (DISM.exe)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/preinstall-apps-using-dism?view=windows-10)**

- **[PowerShell 脚本](https://learn.microsoft.com/en-us/windows/msix/desktop/powershell-msix-cmdlets)**

### 与 MDM 一起管理自动更新

默认情况下，Claude Desktop 大约每四小时检查一次更新并自动应用——独立于您的 MDM 分配的版本。为了防止应用内更新程序和您的 MDM 冲突，请在部署前选择以下方法之一：

**选项 1：您的 MDM 管理版本。**将 `disableAutoUpdates` 策略设置为 `1`，并按您自己的计划通过 MDM 推送新的 MSIX 构建。有关策略位置，请参阅**[Claude Desktop 企业配置](https://support.claude.com/en/articles/12622667-enterprise-configuration-for-claude-desktop)**。

**选项 2：Claude Desktop 管理版本。**保持 `disableAutoUpdates` 未设置。通过 Win32 包装的 `Add-AppxProvisionedPackage` 安装一次部署 MSIX，然后使用自定义检测脚本检查 `Get-AppxPackage -Name Claude` 的版本是否大于或等于您配置的版本。这样可以在应用自我更新后保持您的 MDM 报告**已安装**。

## 配置

要配置 Claude Desktop 设置（如自动更新、扩展和 MCP 服务器），请参阅**[企业配置](https://support.claude.com/en/articles/12622667-enterprise-configuration)**。

---

## 故障排除

### MSIX 包不适用于 AppLocker？

默认情况下，打包应用可能受到 AppLocker 策略的限制。确保您的 AppLocker 规则允许 MSIX 包，或将 Claude Desktop 添加到您的允许应用列表中。在进行更改前，请咨询您组织的安全策略。

### MDM 部署后出现"参数不正确"

这通常意味着应用内更新程序和您的 MDM 都已注册了该包，在 `Claude` 包系列下留下了重复条目。使用上面**[与 MDM 一起管理自动更新](#h_1297fb34f3)**中的指导选择单个更新所有者。

### 安装失败："需要管理员访问权限才能安装具有完整功能的 Claude"，或错误 0x80073D28

Claude Desktop for Windows 包含一个运行 Cowork 虚拟机的后台服务，Windows 仅在使用管理员权限安装 Claude 时才注册该服务。

- **Claude Setup (ClaudeSetup.exe)：**如果管理员提示被拒绝或提升的安装失败，Claude Setup 会显示"需要管理员访问权限才能安装具有完整功能的 Claude"并退出。无法获得管理员批准的用户可以选择改为安装不带 Cowork 的 Claude。

- **按用户部署的 MSIX（例如，分配给用户的 Intune 业务线应用，或在没有管理员权限的情况下运行的 Add-AppxPackage）：**Windows 拒绝安装并显示错误 0x80073D28。

MSIX 包永远不会在没有其 Cowork 服务的情况下安装：要么它完全注册，要么安装失败。

要修复它，请在该设备上使用管理员批准运行 Claude Setup，或在系统上下文中使用 `Add-AppxProvisionedPackage` 机器范围内配置 Claude，如**[通过 MDM 部署](#h_124a24d394)**中所述。

### 安装失败：AddPackage 失败，HRESULT 为 0x80073CF6

此错误意味着设备上之前的 Claude 安装或更新没有完全清理，留下了部分注册的包，因此 Windows 无法注册新包。

**警告：**在这种状态的设备上，安装程序的自动修复会删除之前包的本地应用数据，包括应用设置、保存的登录和仅在 Cowork 内保存的任何文件。聊天历史和用户自己文件夹中的文件不受影响。

要解决它：

1. 重启设备。待处理的删除通常在重启时完成。

2. 以提升权限运行当前安装程序。

3. 如果安装再次失败并出现相同错误，请**[联系支持](https://support.claude.com/en/articles/9015913-how-to-get-support)**并包含用户 Temp 文件夹中的 `ClaudeSetup.log` 文件。IT 需要在系统上下文中恢复设备，支持将提供步骤。

### Cowork 报告缺少 HCS 服务或虚拟化错误

如果 Cowork 启动失败并出现"缺少 HCS 服务：HNS、vmcompute、vfpext"之类的错误，虚拟机平台服务堆栈未在机器上注册，即使 Claude 安装成功也是如此。

**诊断和修复：**

检查该功能是否已启用及其服务是否存在：

```
Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform Get-Service vmcompute, hns
```

如果该功能显示为禁用或服务缺失，请重新启用它：

```
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -All
```

使用**重启**重启机器，而不是关闭并打开电源。启用 Windows 快速启动后，关闭周期可能会使虚拟化服务未初始化。

如果机器上还安装了 VMware 或 VirtualBox，请确认 Windows 虚拟机管理程序设置为在启动时启动。从提升的提示符运行 `bcdedit`，并检查 `hypervisorlaunchtype` 是否显示 `Auto`。如果不是，请运行 `bcdedit /set hypervisorlaunchtype auto` 并重启。

**注意：**不支持没有嵌套虚拟化的虚拟机和 VDI 环境，因此 Cowork 无法在这些主机上运行。要查看具体哪个检查失败，请在 Claude Desktop 中转到**帮助 > 故障排除 > 显示日志**并查看 `supported-features-info.json`。