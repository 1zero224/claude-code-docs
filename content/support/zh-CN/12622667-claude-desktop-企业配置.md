# Claude Desktop 企业配置

Team 或 Enterprise 计划的管理员可以通过系统策略控制 Claude Desktop。

**注意：**用户-机器级别的企业策略控制将覆盖应用内的 **[允许列表](https://support.claude.com/en/articles/12592343-enabling-and-using-the-desktop-extension-allowlist)**。如果您想使用允许列表，请确保 `isDesktopExtensionEnabled` 和 `isDesktopExtensionDirectoryEnabled` 未设置为"false"，以便允许列表可以填充可用的注册表。

---

## macOS 企业配置

通过您的 MDM 解决方案使用配置文件部署配置设置。Claude Desktop 从 `com.anthropic.claudefordesktop` 域读取首选项。使用您的 MDM 工具（Jamf Pro、Kandji、Microsoft Intune）将配置文件部署到目标机器或用户组。配置文件允许您集中管理 Claude Desktop 设置，无需用户干预。

**配置文件工具：**

- 内置 MDM 配置文件编辑器（Jamf Pro、Kandji、Intune）

- **[ProfileCreator](https://github.com/profileCreator/ProfileCreator/)** - 配置文件管理

- **[iMazing Profile Editor](https://imazing.com/profile-editor)** - 配置文件

---

## Windows 企业配置

通过您的企业管理解决方案使用 **[组策略](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/policy/group-policy-objects)**或 Intune 策略部署配置设置。可以在机器范围（HKLM）或按用户（HKCU）级别配置设置。当两者都配置时，机器级别的设置优先于用户级别的设置。

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

| **密钥**                               | **类型**          | **默认值** | **描述**                                                                                                                                                                                                                                                |
| ------------------------------------ | --------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `allowedWorkspaceFolders`            | string[] (JSON) | 无限制     | 用户可以挂载到 Cowork 的文件路径或文件路径                                                                                                                                                                                                                             |
| `autoUpdaterEnforcementHours`        | 整数 (1-72)       | 72      | 强制重启 Claude 以应用准备好的更新前的小时数                                                                                                                                                                                                                            |
| `disableAutoUpdates`                 | 布尔值             | false   | 禁用自动更新。当您的 MDM 管理 Claude Desktop 版本时设置此项；保持未设置状态以让应用在一次性预配安装后自行更新。请参阅 **[为 Windows 部署 Claude Desktop](https://support.claude.com/en/articles/12622703-deploy-claude-desktop-for-windows)**。                                                           |
| `effortLevel`                        | 字符串             | null    | 为 Claude Desktop 中的 Claude Code 会话设置默认工作量级别（`low`、`medium`、`high`、`xhigh` 或 `max`）。托管值在每个会话开始时新鲜应用，即使某人在之前的会话中从选择器更改了工作量级别。需要 Claude Desktop 版本 1.25927.0 或更高版本。不适用于 Claude Desktop 中的 Cowork 会话。                                                     |
| `forceLoginOrgUUID`                  | 字符串 / 数组        | null    | 要求登录属于特定组织。接受单个 UUID 字符串（在登录期间也会预选该组织），或接受 UUID 数组，其中任何列出的组织都被接受而无需预选。如果经过身份验证的帐户不属于列出的组织，登录将失败。要列出多个组织，请参阅下面的 **[为多个组织设置 forceLoginOrgUUID](#h_403de560dd)**。                                                                                        |
| `isClaudeCodeForDesktopEnabled`      | 布尔值             | true    | 在桌面上启用 Claude 代码访问                                                                                                                                                                                                                                    |
| `isDesktopExtensionEnabled`          | 布尔值             | true    | 启用/禁用扩展                                                                                                                                                                                                                                               |
| `isDesktopExtensionDirectoryEnabled` | 布尔值             | true    | 启用扩展目录访问                                                                                                                                                                                                                                              |
| `isLocalDevMcpEnabled`               | 布尔值             | true    | 启用本地 MCP 服务器                                                                                                                                                                                                                                          |
| `secureVmFeaturesEnabled`            | 布尔值             | true    | 在桌面上启用 **[Cowork](https://support.claude.com/en/articles/13345190-getting-started-with-cowork)** 访问                                                                                                                                                   |
| `coworkEDRIntegration`               | 字符串             | 未设置     | 用户计算机上您的 EDR 供应商的已签名传感器包的完整路径（测试版）。如果未设置，则不会安装任何包。在 Windows 上，在 `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Claude` 下设置此项。                                                                                                                                  |
| `coworkEDRIntegrationVendor`         | 字符串             | 未设置     | 您的 EDR 供应商的 ID，小写。在 Windows 上，在 `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Claude` 下设置此项。了解更多关于 **[在 Claude Cowork 中运行您的 EDR 传感器](https://support.claude.com/en/articles/17828685-run-your-endpoint-detection-and-response-edr-sensor-in-claude-cowork)**。 |

---

## 为多个组织设置 `forceLoginOrgUUID`

要接受多个组织，请将 `forceLoginOrgUUID` 设置为组织 UUID 的 JSON 数组，保存为单个字符串值。

**Windows**

使用字符串值 (REG_SZ)，其数据为 JSON 文本。不要使用多字符串值 (REG_MULTI_SZ)。

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

Claude Desktop 在启动时读取此设置。更改后，退出并重新打开 Claude Desktop。要检查应用读取的值，请转到 **帮助 > 故障排除 > 生成诊断报告**。