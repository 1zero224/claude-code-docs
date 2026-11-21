# Claude Desktop 企业配置

Team 或 Enterprise 计划的管理员可以通过系统策略控制 Claude Desktop。

在 Enterprise 计划中，所有者可以在组织设置（测试版）中设置其中多个控制，无需 MDM 工具。请参阅 **[从组织设置管理 Claude Desktop](#h_21135034c8)**。

**注意：**用户-计算机级别的企业策略控制将覆盖应用内 **[允许列表](https://support.claude.com/en/articles/12592343-enabling-and-using-the-desktop-extension-allowlist)**。如果要使用允许列表，请确保 `isDesktopExtensionEnabled` 和 `isDesktopExtensionDirectoryEnabled` 未设置为 “false”，以便允许列表可以填充可用的注册表。

---

## macOS 企业配置

通过 MDM 解决方案使用配置文件部署配置设置。Claude Desktop 从 `com.anthropic.claudefordesktop` 域读取首选项。使用 MDM 工具（Jamf Pro、Kandji、Microsoft Intune）将配置文件部署到目标计算机或用户组。配置文件允许您集中管理 Claude Desktop 设置，无需用户干预。

**配置文件工具：**

- 内置 MDM 配置文件编辑器（Jamf Pro、Kandji、Intune）

- **[ProfileCreator](https://github.com/profileCreator/ProfileCreator/)** - 配置文件管理

- **[iMazing Profile Editor](https://imazing.com/profile-editor)** - 配置文件

---

## Windows 企业配置

通过企业管理解决方案使用 **[组策略](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/policy/group-policy-objects)**或 Intune 策略部署配置设置。可以在计算机范围 (HKLM) 或按用户 (HKCU) 级别配置设置。当两者都配置时，计算机级别设置优先于用户级别设置。

```
# Set machine-wide settings (recommended)
New-Item -Path "HKLM:\SOFTWARE\Policies\Claude" -Force
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Claude" -Name "disableAutoUpdates" -Value 0 -Type DWord
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Claude" -Name "autoUpdaterEnforcementHours" -Value 72 -Type DWord
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Claude" -Name "isDesktopExtensionEnabled" -Value 1 -Type DWord
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Claude" -Name "isDesktopExtensionDirectoryEnabled" -Value 1 -Type DWord
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Claude" -Name "isLocalDevMcpEnabled" -Value 1 -Type DWord
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Claude" -Name "isClaudeCodeForDesktopEnabled" -Value 1 -Type DWord
```

---

## 企业策略选项

| **密钥**                               | **类型**          | **默认值** | **描述**                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------------------------------ | --------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `allowedWorkspaceFolders`            | string[] (JSON) | 无限制     | 用户可以挂载到 Cowork 的文件路径或文件路径<br>Enterprise 计划：也可在组织设置中使用。                                                                                                                                                                                                                                                                                                                                                                                                      |
| `autoUpdaterEnforcementHours`        | 整数 (1-72)       | 72      | 强制重启 Claude 以应用准备好的更新前的小时数<br>Enterprise 计划：也可在组织设置中使用。                                                                                                                                                                                                                                                                                                                                                                                                     |
| `disableAutoUpdates`                 | 布尔值             | false   | 禁用自动更新。当您的 MDM 管理 Claude Desktop 版本时设置此项；保持未设置状态以让应用在一次性预配安装后自行更新。请参阅 **[为 Windows 部署 Claude Desktop](https://support.claude.com/en/articles/12622703-deploy-claude-desktop-for-windows)**。<br>Enterprise 计划：也可在组织设置中使用。                                                                                                                                                                                                                                    |
| `effortLevel`                        | 字符串             | null    | 为 Claude Desktop 中的 Claude Code 会话设置默认工作量级别（`low`、`medium`、`high`、`xhigh` 或 `max`）。托管值在每个会话开始时新鲜应用，即使某人在之前的会话中从选择器更改了工作量级别。需要 Claude Desktop 版本 1.25927.0 或更高版本。不适用于 Claude Desktop 中的 Cowork 会话。                                                                                                                                                                                                                                                           |
| `forceLoginOrgUUID`                  | 字符串 / 数组        | null    | 要求登录属于特定组织。接受单个 UUID 字符串（在登录期间也会预选该组织），或接受 UUID 数组，其中任何列出的组织都被接受而无需预选。如果经过身份验证的帐户不属于列出的组织，登录将失败。要列出多个组织，请参阅下面的 **[为多个组织设置 forceLoginOrgUUID](#h_403de560dd)**。                                                                                                                                                                                                                                                                                              |
| `isClaudeCodeForDesktopEnabled`      | 布尔值             | true    | 在桌面上启用 Claude Code 访问                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `isDesktopExtensionEnabled`          | 布尔值             | true    | 启用/禁用扩展<br>Enterprise 计划：也可在组织设置中使用。                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `isDesktopExtensionDirectoryEnabled` | 布尔值             | true    | 启用扩展目录访问<br>Enterprise 计划：也可在组织设置中使用。                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `isLocalDevMcpEnabled`               | 布尔值             | true    | 启用本地 MCP 服务器<br>Enterprise 计划：也可在组织设置中使用。                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `secureVmFeaturesEnabled`            | 布尔值             | true    | 在桌面上启用 **[Cowork](https://support.claude.com/en/articles/13345190-getting-started-with-cowork)** 访问                                                                                                                                                                                                                                                                                                                                                         |
| `coworkEDRIntegration`               | 字符串             | 未设置     | 用户计算机上 EDR 供应商签名的传感器包的完整路径（测试版）。如果未设置，则不安装任何包。在 Windows 上，在 `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Claude` 下设置此项。                                                                                                                                                                                                                                                                                                                                            |
| `coworkEDRIntegrationVendor`         | 字符串             | 未设置     | 您的 EDR 供应商的 ID，小写。在 Windows 上，在 `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Claude` 下设置此项。详细了解 **[在 Claude Cowork 中运行 EDR 传感器](https://support.claude.com/en/articles/17828685-run-your-endpoint-detection-and-response-edr-sensor-in-claude-cowork)**。                                                                                                                                                                                                           |
| `computerUseEnabled`                 | 布尔值             | 未设置     | 设置为 false 以在此计算机上为每个帐户关闭计算机使用。True 不会打开它。需要 Claude Desktop 版本 5.1231.0 或更高版本。在 Windows 上，在 `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Claude` 下将其设置为 `DWORD` 值 `0`。当 `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Claude` 包含任何值时，Claude Desktop 不会读取 `HKEY_CURRENT_USER\SOFTWARE\Policies\Claude`。在部署之前，请参阅 **[为 Team 和 Enterprise 计划设置计算机使用](https://support.claude.com/en/articles/18483812-set-up-computer-use-for-team-and-enterprise-plans)**中的**在计算机上保持计算机使用关闭**。 |

设置任何 Claude Desktop 策略密钥会隐藏**发布渠道**设置，该设备遵循稳定版。已在最新版本上的设备会保持其版本，直到推出更新的稳定版本。详细了解 **[选择发布渠道](https://support.claude.com/en/articles/10065433-install-claude-desktop#h_ac219b3bde)**。

---

## 从组织设置管理 Claude Desktop（测试版）

在 Enterprise 计划中，所有者可以在组织设置（测试版）中设置这些 Claude Desktop 控制，无需 MDM 工具，除非您需要 MDM 密钥添加的功能。

- **在 [组织设置 > 组织和访问](https://claude.ai/admin-settings/organization)中的“安全”下打开“将 Claude Desktop 限制为允许的组织（影子 IT）”：**成员在计算机上的 Claude Desktop 中登录您的组织后，该计算机上的应用会阻止个人帐户和您未允许的其他组织，从下次打开时开始。要在任何人登录之前阻止它们，请通过 MDM 设置 `forceLoginOrgUUID`。

  - **限制：**此块不是永久的。如果成员在该计算机上 30 天内未使用您的组织，它会结束，使用该计算机的任何人都可以将其删除。如果您允许多个组织，请在每个组织中设置相同的允许组织列表。

- **在 [组织设置 > 功能](https://claude.ai/admin-settings/capabilities)中关闭“自动更新”：**Claude Desktop 停止检查和下载更新。用户仍然可以从应用菜单更新，已下载的更新仍会安装。

- **在 [组织设置 > 功能](https://claude.ai/admin-settings/capabilities)中允许工作区文件夹：**选择“仅允许列表中的文件夹，”或关闭以不允许任何文件夹。将文件夹标记为“只读”，Claude 的文件工具无法更改该处的文件。

  - **限制：**“只读”仅涵盖 Claude 的文件工具。在代码选项卡中，命令、MCP 服务器和钩子仍然可以更改这些文件夹并访问其他文件夹，并且在保存 Claude Desktop 自己数据的文件夹（如主文件夹）上被忽略。它不适用于 Windows 上的 SSH。每台计算机使用自己的操作系统列表，SSH 会话使用它们启动的计算机的列表。当文件夹受限时，WSL 和 localhost SSH 被阻止。缺失的文件夹不会被创建。

- **关闭“允许本地 MCP 服务器”（[组织设置 > 插件 > 连接器](https://claude.ai/admin-settings/connectors)）：**用户添加的本地 MCP 服务器不会启动。桌面扩展仍然可以运行本地服务器，因此也要选择“阻止所有桌面扩展”以停止这些。

- **选择“阻止所有桌面扩展”（[组织设置 > 插件 > 连接器](https://claude.ai/admin-settings/connectors)）：**已安装的扩展停止工作，无法安装任何扩展。

如果计算机上的策略设置了匹配的密钥，该策略在该计算机上获胜，即使它的限制性较低。对于自动更新，`autoUpdaterEnforcementHours` 也计算在内。这些设置遵循帐户，因此不适用于个人帐户。这些设置需要 Claude Desktop 4.29.0 或更高版本。

---

## 为多个组织设置 `forceLoginOrgUUID`

要接受多个组织，请将 `forceLoginOrgUUID` 设置为组织 UUID 的 JSON 数组，保存为单个字符串值。

**Windows**

使用字符串值 (REG_SZ)，其数据是 JSON 文本。不要使用多字符串值 (REG_MULTI_SZ)。

```
New-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Claude' -Name forceLoginOrgUUID
-PropertyType String -Value '["<uuid-1>","<uuid-2>"]' -Force
```

**macOS**

在 `com.anthropic.claudefordesktop` 域的配置文件中，将密钥设置为包含 JSON 文本的字符串：

```
<key>forceLoginOrgUUID</key>
<string>["<uuid-1>","<uuid-2>"]</string>
```

Claude Desktop 在启动时读取此设置。更改后，退出并重新打开 Claude Desktop。要检查应用读取的值，请转到**帮助 > 故障排除 > 生成诊断报告**。