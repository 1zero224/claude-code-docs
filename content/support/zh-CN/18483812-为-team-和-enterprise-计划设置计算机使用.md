# 为 Team 和 Enterprise 计划设置计算机使用

计算机使用让 Claude 能够查看用户的屏幕并在用户允许的桌面应用中工作。Claude 在 Claude Desktop 上的点击和输入方式与您相同。本文介绍管理员如何为您的组织启用计算机使用、将其限制为特定用户、限制为特定应用，以及通过设备管理 (MDM) 进行管理。

计算机使用在 Team 和 Enterprise 计划中处于测试阶段，默认关闭。这正在逐步向各个组织推出。

## 可用性

计算机使用仅在新的 Claude 体验中有效，该体验在启用了**聊天和协作统一**功能的地方可用。用户可以从以下位置使用它：

- **Claude Desktop**，在 macOS 14 或更高版本以及 Windows 上。

- **Claude 网页版和 Claude Mobile**，如果您的组织的**计算机访问**设置已启用，且计算机上打开了 Claude Desktop。

它在使用单独的“协作”选项启动的任务中或在 Claude Code 中不起作用。所有者可以在适用于您的计划的文章中了解更多信息：

- **[为您的 Team 组织设置和管理新的统一 Claude 体验](https://support.claude.com/en/articles/18342806-set-up-and-manage-the-new-unified-claude-experience-for-your-team-organization)**

- **[为您的 Enterprise 组织设置和管理新的统一 Claude 体验](https://support.claude.com/en/articles/17444462-set-up-and-manage-the-new-unified-claude-experience-for-your-enterprise-organization)**

无法启用计算机使用的组织包括具有以下配置的组织：

- HIPAA 就绪配置

- Claude Code 零数据保留 (ZDR)

- 客户管理的加密密钥 (CMEK)

- FERPA 或 K-12 设置

**注意：****计算机使用**和**计算机访问**是不同的设置。计算机使用让 Claude 能够查看屏幕并在应用中工作。计算机访问让用户可以从计划的任务、Claude 网页版或 Claude Mobile 访问他们的计算机。

## 启用计算机使用前

查看**[安全使用 Claude 协作](https://support.claude.com/en/articles/13364135-use-claude-cowork-safely)**并记住以下几点：

- 计算机使用在 Claude 和用户的应用程序之间没有沙箱。屏幕上的内容（如网页或电子邮件）可能导致 Claude 采取意外操作。

- 计算机使用不限于用户已连接的文件夹。通过用户批准的应用，Claude 可以打开、更改或删除这些文件夹外的文件。

- 如果您**[限制用户可以连接的文件夹](https://support.claude.com/en/articles/12622667-enterprise-configuration-for-claude-desktop)**，该限制不适用于计算机使用。将文件夹标记为“只读”也不适用于计算机使用。

- 如果您使用 Enterprise 计划并想防止某些用户使用计算机使用，请在启用它之前阅读下面的**将计算机使用限制为特定用户**。

## 屏幕截图和您的数据

Claude 会截取屏幕截图以查看和导航屏幕，这些截图会与聊天一起保存。它们可能包含个人或机密信息。审查 Claude 操作的自动检查也可以看到最新的屏幕截图，Anthropic 可以在其检查记录中保留一份副本。您的数据根据我们的**[商业条款](https://www.anthropic.com/legal/commercial-terms)**和**[数据处理附录](https://www.anthropic.com/legal/data-processing-addendum)**进行处理。

## 启用或禁用计算机使用

要为您的组织启用或关闭计算机使用：

1. 以所有者或主要所有者身份登录 Claude。

2. 导航到**[组织设置 > 功能](https://claude.ai/admin-settings/capabilities)**。

3. 确保**聊天和协作统一**已启用。没有它，计算机使用无法工作。

4. 在**Claude Desktop**下，找到**计算机使用**并将其启用或禁用。

5. 在 Enterprise 计划上，如果打开角色列表，清除不应该拥有计算机使用的每个角色，然后单击“启用。”

只有所有者或主要所有者可以更改此设置。如果该设置不存在或无法启用，则它尚未到达您的组织，或您的组织无法拥有它。请参阅上面的**可用性**。

当您关闭计算机使用时，更改可能需要大约一小时（有时更长）才能到达每个用户的 Claude Desktop。在此之前，Claude 仍然可以查看他们的屏幕并使用他们允许的应用。

要在选定的计算机上保持计算机使用关闭，请参阅下面的**使用设备管理 (MDM) 管理计算机使用**。

## 将计算机使用限制为特定用户（Enterprise 计划）

计算机使用是自定义角色中的一项功能，与其他功能一样。Team 计划没有自定义角色，因此该设置涵盖每个用户。在**[在 Enterprise 计划上管理自定义角色](https://support.claude.com/en/articles/13930452-manage-custom-roles-on-enterprise-plans)**中了解更多信息。

### 具有内置角色的用户

角色为用户、管理员、所有者或主要所有者的用户在您启用计算机使用后立即获得计算机使用。要防止用户使用，请将其角色设置为“自定义，”然后等待 15 分钟后再启用。

### 具有自定义角色的用户

要使用计算机使用，具有自定义角色的用户需要一个包含**聊天和协作统一**功能的角色。

当您启用计算机使用时，您的自定义角色会被列出，通常已被选中。清除不应该拥有它的每个角色。

您无法清除列表中设置为“所有功能”的角色。要防止该角色使用计算机使用：

1. 在启用计算机使用之前，将角色的**功能访问**设置为“仅选定的。”

2. 关闭角色上的**计算机使用**。

3. 等待 15 分钟。

4. 为您的组织启用计算机使用，并清除列表中的该角色。

### 稍后更改角色访问

启用计算机使用后，使用设置旁边的“编辑角色访问”或每个角色的**功能**选项卡更改哪些自定义角色拥有它。撤销访问可能需要大约一小时（有时更长）才能到达每个用户的 Claude Desktop。

## 将计算机使用限制为特定应用

**应用访问**设置有两个应用列表。只有所有者或主要所有者可以更改它。

- **阻止这些应用：**阻止列表中的应用（如密码管理器）将被拒绝。

- **仅允许这些应用：**除了允许列表中的应用外，所有应用都将被拒绝。用户仍需逐个批准每个应用。

Claude 无法通过计算机使用访问被拒绝的应用，除非在**输入应用的 ID** 和**阻止和允许列表未涵盖的内容**中另有说明。在下拉菜单中选择一个列表会将其启用，另一个列表保持保存状态。当您添加允许列表中的第一个应用时，空的允许列表会启用。您列出的应用会发送到用户的计算机。Claude Desktop 不会向用户显示该列表，但有人可能会找到它，因此请勿包含任何机密信息。

启用允许列表时，未更新到最新 Claude Desktop 的用户将无法使用计算机使用功能，直到他们更新。

### 添加或移除应用

添加应用：

1. 导航至**[组织设置 > 功能](https://claude.ai/admin-settings/capabilities)**。

2. 在**Claude Desktop** 下，找到**应用访问**，位于**计算机使用**下方。

3. 选择“阻止这些应用”或“仅允许这些应用”。

4. 点击“添加应用”。

5. 在**应用**下，搜索常见应用列表并选择要添加的每个应用。如果应用不在列表中，请在下方**应用 ID，每行一个**的框中输入其 ID。

6. 点击“完成”。

如果您在 Claude Desktop 中打开**组织设置**，搜索还会列出计算机上的大多数应用。这些应用仅在具有相同操作系统的计算机上被拒绝或允许。

要移除应用，请点击其行末尾的垃圾桶图标。如果您从正在使用的允许列表中移除最后一个应用，Claude 将无法使用任何应用。

### 检查应用

检查应用：

1. 更改后等待一分钟。

2. 在有该应用的计算机上退出并重新打开 Claude Desktop。

3. 在新聊天中，要求 Claude 使用应用的确切名称。请参阅下方的**对用户的影响**。

应用的新版本可能会改变其 ID。

### 输入应用的 ID

- **Mac：**输入捆绑 ID，如`com.microsoft.Excel`。

- **Windows：**输入程序的文件名，如`excel.exe`，或 Microsoft Store 应用的应用用户模型 ID (AUMID)。

没有任何检查可验证输入的 ID 是否对应真实应用。

在 Windows 上，某些应用从一个程序启动，但从另一个程序显示窗口。例如，LibreOffice 的窗口属于 soffice.bin。将窗口的文件名放在阻止列表中。否则，即使通过了上述检查，Claude 仍可在用户批准后使用该应用。

### 对用户的影响

如果用户请求被拒绝的应用，Claude 会说明其组织已阻止该应用，或该应用不在其组织的列表中。该应用之前的批准将不再计数。Claude 无法在应用中点击或输入，也无法在屏幕截图中看到该应用，除非在**输入应用的 ID** 和**阻止和允许列表未涵盖的内容**中另有说明。

添加或移除应用，或切换列表，可能需要约一小时（有时更长）才能到达每个用户的 Claude Desktop。在此期间，Claude 仍可使用您刚刚拒绝的应用。

### 阻止和允许列表未涵盖的内容

- **Claude 的其他工具：**包括 Chrome 中的 Claude、内置浏览器、Claude Code 和桌面扩展。拒绝 Google Chrome 仅会停止 Chrome 中的计算机使用。

- **其他账户：**如果用户将 Claude Desktop 切换到个人账户或另一个组织，您的阻止和允许列表在那里不适用。

- **网站和浏览器扩展：**阻止和允许列表不涵盖这些内容，如用户批准的浏览器中的密码管理器扩展。

- **远程桌面和虚拟机：**阻止和允许列表不涵盖用户批准的远程桌面或虚拟机窗口内的应用。

- **对被拒绝应用的间接影响：**Claude 可通过在用户批准的应用中的操作影响被拒绝的应用。

- **从被拒绝应用复制的文本：**Claude 可以读取它，例如通过将其粘贴到用户批准的应用中。

- **使用“记录技能”进行的录制：**录制包括屏幕上的每个应用，包括被拒绝的应用。

- **Windows 通知：**被拒绝应用的 Windows 通知在屏幕截图中保持可见，“开始”菜单、搜索和文件浏览器也是如此。

## 用户需要的内容

用户需要最新的 Claude Desktop，在 Claude 将使用的计算机上打开。每个用户在每台计算机和每个账户上打开**启用计算机使用**（如果尚未打开）。

### Claude 网页版和 Claude Mobile

从 Claude 网页版或 Claude Mobile 使用计算机使用还需要您的组织的**计算机访问**设置处于打开状态。聊天还必须连接到 Claude 将使用的计算机。用户在被询问时选择计算机，或在 Claude 网页版中，在“+” > “设备”下选择。

### macOS 权限

在 macOS 上，计算机使用需要辅助功能和屏幕录制权限。在**系统设置 > 隐私与安全**中，它们显示为：

- **辅助功能**（macOS 27 上为**设备控制和数据访问**）

- **屏幕和系统音频录制**

在其 Mac 上不是管理员的用户可能需要管理员的名称和密码才能打开这些权限。要使用 MDM 工具准备两者，请参阅下方的**准备 macOS 权限**。

## 使用设备管理 (MDM) 管理计算机使用

如果您使用 Jamf 或 Intune 等 MDM 工具管理用户的计算机，可以在选定的计算机上保持计算机使用关闭。您还可以使其在 Mac 上更容易设置。在 Windows 上，计算机使用不需要操作系统权限，因此无需准备任何内容。

### 在计算机上保持计算机使用关闭

要在计算机上保持计算机使用关闭（无论登录的是哪个账户），请将 Claude Desktop 的策略密钥 computerUseEnabled 设置为 false。要了解如何部署策略密钥，请参阅**[Claude Desktop 企业配置](https://support.claude.com/en/articles/12622667)**。

**部署密钥前**

密钥需要 Claude Desktop 5.1231.0 或更高版本。首先在所有计算机上更新 Claude Desktop。旧版本不识别该密钥，因此不会关闭计算机使用功能。

**警告：**在 Windows 上，如果 `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Claude` 中不包含旧版本认识的设置（例如，如果 `computerUseEnabled` 是唯一的值），旧版本会拒绝整个策略。在更新应用或删除密钥之前，它会忽略保存在该计算机上的应用设置，例如第三方推理设置。

**重要提示：**在计算机级别部署密钥，而不是用户级别。用户级别的设置仅适用于该计算机的一个用户。

**macOS**

在偏好设置域 `com.anthropic.claudefordesktop` 的配置文件中将密钥设置为布尔值。例如：

`<key>computerUseEnabled</key><false/>`

如果已安装该域的计算机级别配置文件，Claude Desktop 会忽略该域的用户级别配置文件，或在更新后会忽略。如果您的其他 Claude 设置在用户级别配置文件中，也请将它们放在计算机级别配置文件中。

**Windows**

在 `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Claude` 下将密钥设置为 `DWORD` 值 `0`。例如，在以管理员身份运行的 PowerShell 中，运行以下命令：

```
New-Item -Path "HKLM:\SOFTWARE\Policies\Claude" -ErrorAction SilentlyContinue
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Claude" -Name "computerUseEnabled" -Value 0 -Type DWord
```

当 `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Claude` 包含任何值时，不会读取用户级别的 Claude 设置。如果您的其他 Claude 设置在 `HKEY_CURRENT_USER` 下，也请将它们移到 `HKEY_LOCAL_MACHINE`。

**检查密钥是否有效**

Claude Desktop 仅在启动时读取密钥，因此密钥在 Claude Desktop 退出并重新打开后才会生效。仅关闭其窗口是不够的。

要检查计算机，请在该计算机上的 Claude Desktop 中打开**设置**。**启用计算机使用**切换呈灰显状态，当您将鼠标悬停在其上时，会显示：“您的组织已在此计算机上关闭计算机使用功能。请联系您的 IT 管理员以获取访问权限。”

如果 Claude Desktop 在计算机上找到 Claude 策略但无法读取，它也会关闭计算机使用功能。用户会看到：“无法检查此计算机上是否允许计算机使用功能。退出并重新打开 Claude 重试。如果仍然关闭，请联系您的 IT 管理员。”

**密钥不涵盖的内容**

密钥不涵盖“记录技能”、Chrome 中的 Claude、内置浏览器、终端中的 Claude Code、本地 MCP 服务器或桌面扩展。该密钥改变 Claude Desktop 的行为方式。它不是针对能够更改应用或计算机设置的人的锁定。

与任何 Claude Desktop 策略密钥一样，`computerUseEnabled` 使 Claude Desktop 将计算机视为受管理的，无论其值为真还是假。在受管理的计算机上，用户无法选择发布渠道，Claude Desktop 停止读取用户自己的其他应用行为设置值。在受管理的 Windows 计算机上，除非 `HKEY_LOCAL_MACHINE` 下的 `disableWslSessions` 设置为 false，否则 WSL 中的代码选项卡会话会被拒绝。

### 准备 macOS 权限

在 macOS 上，计算机使用需要的两个权限“辅助功能”和“屏幕录制”被授予给 Claude Desktop 本身，而不是 Claude 账户。Claude Desktop 的快速输入功能也使用它们。请参阅上面的**您的用户需要什么**。

您可以使用隐私偏好策略控制 (PPPC) 有效负载准备两者。它需要用户批准的 MDM 注册，无法手动安装。为 Claude Desktop 添加一个条目。应用内的辅助工具使用其权限。

- **标识符：**`com.anthropic.claudefordesktop`

- **标识符类型：**`bundleID`

- **代码要求：**`identifier "com.anthropic.claudefordesktop" and anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] /* exists */ and certificate leaf[field.1.2.840.113635.100.6.1.13] /* exists */ and certificate leaf[subject.OU] = Q6L2SF6YDW`

要检查代码要求，请在安装了 Claude Desktop 的 Mac 上运行 `codesign -d -r - /Applications/Claude.app`。`designated =>` 后面的文本应与上面的代码要求匹配。

| **权限**    | **PPPC 服务**   | **要设置的内容**                                                                     | **其作用**                                           |
| --------- | ------------- | ------------------------------------------------------------------------------ | ------------------------------------------------- |
| 辅助功能      | 辅助功能          | `Authorization` 设置为 `Allow`                                                    | 在 macOS 14 到 26 上，不会询问用户，用户也无法更改。                 |
| 屏幕和系统音频录制 | ScreenCapture | `Authorization` 设置为 `AllowStandardUserToSetSystemService`（如果您的用户不是其 Mac 上的管理员） | 配置文件无法启用此权限。此设置允许不是管理员的用户在**系统设置 > 隐私和安全**中自行启用它。 |

请记住以下几点：

- 如果您的用户不是其 Mac 上的管理员，请在配置文件中允许辅助功能。`AllowStandardUserToSetSystemService` 不适用于辅助功能，因此没有配置文件的情况下，这些用户可能需要管理员的用户名和密码才能启用它。

- 如果两个配置文件为同一权限给 Claude Desktop 提供不同的设置，macOS 会使用限制性更强的设置，因此拒绝优先。

- 如果用户设置计算机使用时已授予权限，Claude Desktop 应将其显示为已授予，用户可以跳过该步骤。

- 如果用户在 Claude Desktop 打开时启用屏幕录制，他们可能需要退出并重新打开 Claude Desktop。

- 如果您已部署的配置文件拒绝 Claude Desktop 的辅助功能或屏幕录制权限，这些 Mac 上的计算机使用会失败。Claude Desktop 无法说明原因。它仅将权限显示为未授予。

拒绝也会从 Claude Desktop 的快速输入功能中移除权限，因此这不是关闭计算机使用的好方法。改为将 `computerUseEnabled` 设置为 false。

准备这些权限不会启用计算机使用功能。

### MDM 无法做的事

- **启用计算机使用：**您的组织的**计算机使用**设置、用户在企业计划中的角色以及每个用户自己的开关决定了这一点。将 `computerUseEnabled` 设置为 true 与不设置它对计算机使用的效果相同。

- **启用用户自己的开关：**每个用户在 Claude Desktop 中启用**启用计算机使用**，在每台计算机和每个账户上，如果尚未启用的话。

- **批准 Claude 的应用：**每个用户自己批准应用。

- **启用屏幕和系统音频录制：**macOS 将其留给 Mac 前的人员。

## 您的用户看到的内容

能够使用计算机的用户会在 Claude Desktop 的**设置**中的**计算机使用**标题下看到此内容：“对于您的组织，计算机使用在新的 Claude 体验中有效。它在 Cowork 任务或 Claude Code 中不可用。”

对于无法启用它的用户，切换开关呈灰显状态，当他们将鼠标悬停在其上时会显示原因。有关每条消息和要执行的操作，请参阅**[让 Claude 在 Cowork 中使用您的计算机](https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork)**。