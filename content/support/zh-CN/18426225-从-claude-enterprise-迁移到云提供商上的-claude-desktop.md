# 从 Claude Enterprise 迁移到云提供商上的 Claude Desktop

本指南说明如何从 Claude Enterprise 迁移到您自己的云提供商上的 Claude Desktop。它涵盖了迁移内容、用户和管理员的变化，以及 Chat、Claude Cowork 和 Claude Code 的迁移步骤。

本指南适用于 Claude Enterprise 计划组织的管理员。

本指南不涵盖合同、定价、采购、API 工作负载或如何解释合规声明。

使用 Claude Enterprise，Anthropic 托管模型并存储您的对话历史记录。另一种选择是第三方提供商上的 Claude Desktop，我们的文档称之为 **Claude Desktop on 3P**。桌面应用 **[通过您配置的提供商路由所有模型推理](https://claude.com/docs/third-party/claude-desktop/overview)**，因此提示和响应会通过该提供商。它可以是 Amazon Bedrock、Google Cloud 的 Agent Platform（原称 Vertex AI）、Microsoft Foundry 或您自己的 LLM 网关。

迁移分为两部分。您在 3P 上部署 Claude Desktop，每个用户将自己的历史记录复制到其中。

## 开始前

您需要：

- 您的 Claude Enterprise 组织的所有者或主要所有者。只有主要所有者才能运行 **[组织导出](https://support.claude.com/en/articles/13346720-export-your-organization-s-data)**。

- 将运行推理的云账户、项目或订阅的管理员。

- 向设备传递设置的方式：**[企业管理控制台](https://claude.com/docs/third-party/claude-desktop/admin-console)**、**[MDM 配置文件](https://claude.com/docs/third-party/claude-desktop/mdm)**或 **[引导服务器](https://claude.com/docs/third-party/claude-desktop/bootstrap)**。

- 每台设备上的最新 Claude Desktop 版本。从较早的第三方配置迁移到控制台的设备应在用户登录前使用 5.11492.0 或更高版本，因为较早版本迁移的项目较少。会话导出在两台设备上都需要 4.23118.0 或更高版本。

## 预期情况

- **每个用户迁移自己的历史记录。**每个用户在自己的计算机上运行一次性导入。聊天和项目会被导入，但上传的文件不会。请参阅 **[从 claude.ai 导入历史记录](https://claude.com/docs/third-party/claude-desktop/import)**。

- **工作存储在设备上。**对话历史记录、项目和记忆存储在用户的计算机上。提示、响应、文件和工具输出 **[仍会发送到您的推理提供商](https://claude.com/docs/third-party/claude-desktop/overview)**。没有移动应用和 claude.ai 网络应用。Claude 仅在桌面应用中运行，在 **[macOS 和 Windows 上](https://claude.com/docs/third-party/claude-desktop/installation)（Linux 处于测试版）**。用户无法彼此共享聊天或项目。请参阅 **[功能对比](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**。

- **您的云提供商向您计费。**您按使用的令牌付费，没有座位许可证。请参阅 **[功能对比](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**。

- **您的提供商决定谁处理您的数据。**在 Amazon Bedrock 和 Google Cloud 上，云提供商在您选择的区域或路由选项中处理对话数据，应用 **[不会将用户数据、提示或完成发送给 Anthropic](https://claude.com/docs/third-party/claude-desktop/overview)**。在 Microsoft Foundry 上，**[Anthropic 作为 Microsoft 的独立处理者运营 Claude 模型](https://claude.com/docs/third-party/claude-desktop/foundry)**。某些模型添加了自己的数据保留和数据共享要求。请参阅 **[比较云提供商](https://support.claude.com/en/articles/18466110)**。

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2806511265/bf14805691a94c651943902ed7ad/CleanShot+2026-11-12+at+2_43_29+PM.png?expires=1795099500&amp;signature=5c0e1fd4632dce772740490e1ddba92ac81d2593e94d5ef4fc690be0598af68b&amp;req=dignEMx%2FnINZXPMW1HO4zUslpEFH6389wA0QijyN3qq5vgI9oaAhFVpjmBpc%0Amm7RETkxGQ78zyciJfo%3D%0A)

在迁移期间，您可以并行运行 Claude Enterprise 和 Claude Desktop on 3P。Claude Desktop on 3P 将其应用数据保存在单独的目录中，**[因此两者可以在一台计算机上共存](https://claude.com/docs/third-party/claude-desktop/data-storage)**。它们共享一些文件夹，例如 `~/Claude/` 和 `~/.claude/`。如果您稍后需要返回，请参阅 **[将 Claude Desktop 从云提供商迁移到 Claude Enterprise](https://claude.com/docs/third-party/claude-desktop/move-to-claude-enterprise)**。

本指南的大部分内容适用于每个提供商。有关特定于 Amazon Bedrock、Google Cloud 或 Microsoft Foundry 的详细信息，请参阅 **[比较云提供商](https://support.claude.com/en/articles/18466110)**。

**注意：**设置和版本要求在各个版本之间会发生变化。在更改设置前，请检查链接的页面。

## 迁移内容和不迁移内容

导入会带入每个用户自己的聊天和项目，以及计算机上已有的任何 Cowork 和 Code 会话。计划在 **截止日期**前手动迁移其他所有内容，截止日期是用户可以访问 claude.ai 中工作的最后一天。您设置此日期。

导入是一次性复制。它不会与 claude.ai 保持同步，但您可以再次运行它以获取较新的聊天 **[而不会创建重复项](https://claude.com/docs/third-party/claude-desktop/import)**。

| **项目**                          | **导入会带入它吗？**                                                                                                                       | **要做什么**                                                                                                                                                                                                                                                |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 每个用户自己的聊天                       | 是                                                                                                                                  | 运行导入。在截止日期再次运行以获取较新的聊天。                                                                                                                                                                                                                                 |
| 每个用户自己的项目和项目说明                  | 是。用户 **[审查并接受](https://claude.com/docs/third-party/claude-desktop/import)**说明后才会生效。                                                | 导入后检查每个项目。如果已存在同名项目，选择 “合并到现有项目”或 “保持分离。”                                                                                                                                                                                                               |
| 项目知识文件和聊天附件                     | 否。用户的导出 **[永远不包括文件内容](https://claude.com/docs/third-party/claude-desktop/import)**。聊天保留提及文件的消息，但不保留文件本身。                           | 收集原始文件并将其添加回 Claude Desktop 中的项目。主要所有者的 **[组织导出](https://support.claude.com/en/articles/13346720-export-your-organization-s-data)**是唯一包括文件内容的导出。它不包括 Claude 创建的 Word、Excel、PowerPoint 或 PDF 文件。将其保存为存档。                                                 |
| 计算机上已有的 Cowork 和 Claude Code 会话 | 是，在 **Cowork & Code** 步骤中。会话被复制，不是移动。                                                                                              | 选择回溯多远。如果您限制导入到特定组织，此步骤不可用。                                                                                                                                                                                                                             |
| 云中的 Cowork 和 Code 会话            | 否。向导仅导入计算机上的会话。                                                                                                                    | 在截止日期前完成或下载工作。                                                                                                                                                                                                                                          |
| 其他用户的内容                         | 否。导入仅带入 **[用户自己的数据](https://claude.com/docs/third-party/claude-desktop/import)**。不包括其他用户的内容。                                       | 如果用户依赖他人创建的项目，请要求其创建者导入它。请参阅 **替换共享项目**。                                                                                                                                                                                                                |
| 记忆                              | 导出包含用户的记忆，但导入仅带入聊天和项目。                                                                                                             | 手动复制。请参阅 **手动移动记忆**。                                                                                                                                                                                                                                    |
| 个人说明                            | 计划手动移动这些。                                                                                                                          | 从 claude.ai **[设置](https://support.claude.com/en/articles/10185728-understanding-claude-s-personalization-features)**中的 **Claude 说明**（在较新的设置布局中称为说明）复制文本。                                                                                               |
| 工件                              | 部分可以。Claude 在对话中创建的工件最终版本会作为输出文件导出。早期版本不会，并且有大小限制。                                                                                 | **[导出或下载](https://support.claude.com/en/articles/17153992-what-are-artifacts-and-how-do-i-use-them)**您需要的工件。                                                                                                                                            |
| Claude 创建的文件                    | 部分可以。使用 Claude 文件创建工具编写的文件最终版本会作为输出文件导出。Word、Excel、PowerPoint 和 PDF 文件不包含在导出中。                                                     | **[从对话中下载](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude)**它们。                                                                                                                                                      |
| 用户构建的技能                         | 计划手动移动这些内容。                                                                                                                        | 从**[自定义 > 技能](https://support.claude.com/en/articles/12053672-what-happens-to-a-user-s-data-when-they-are-removed-from-a-team-or-enterprise-organization)**下载。管理员可以通过**[插件市场](https://claude.com/docs/third-party/claude-desktop/extensions)**与所有人共享技能。 |
| 连接器                             | 不会。它们是新配置的一部分。                                                                                                                     | 管理员将其设置为**[托管 MCP 服务器](https://claude.com/docs/third-party/claude-desktop/extensions)**。                                                                                                                                                                |
| 审计日志                            | 不会                                                                                                                                 | 所有者或主要所有者可以在关闭组织前**[导出过去 180 天的数据](https://support.claude.com/en/articles/9970975-access-audit-logs)**。如果您使用客户管理的加密密钥，请改用合规 API。                                                                                                                        |
| Claude Code CLI 设置、记忆和本地会话      | 无需操作。每个开发者自己的设置、记忆和本地会话**[已存储在计算机上](https://code.claude.com/docs/en/claude-directory)**。从 claude.ai 同步的组织管理设置和技能工作方式不同。            | 保持原样。请参阅**[移动 Claude Code CLI 用户](https://support.claude.com/en/articles/18466061)**。                                                                                                                                                                   |
| 自定义角色和模型访问设置                    | 不会。它们**[在您的 Claude Enterprise 组织中设置](https://support.claude.com/en/articles/15694740-manage-model-access-for-your-organization)**。 | 在您的提供商和新配置中重新设置。请参阅**[规划受限模型和访问计划](https://support.claude.com/en/articles/18466097)**。                                                                                                                                                                  |

**注意：**主要所有者**[看不到个人“导出数据”按钮](https://support.claude.com/en/articles/9450526-export-your-claude-data)**。使用您的试点来测试主要所有者自己的历史记录将如何移动。

**重要：**用户已删除的内容或您的保留设置已删除的内容**[不包含在任何导出中](https://support.claude.com/en/articles/13346720-export-your-organization-s-data)**。在设置时间表前检查您的保留期。

### 替换共享项目

在 Claude Desktop on 3P 中，**[项目存储在每个用户的计算机上，不与其他用户共享](https://claude.com/docs/cowork/guide/projects)**。为了给团队提供相同的起点，我们建议以下做法：

- **共享说明和参考资料：**将其打包为技能或插件并分发给所有人。请参阅**分发技能和插件**。

- **共享文件：**将其保存在您已使用的文件共享或存储库工具中。Claude 将工件和计划任务结果写入用户主目录中的**[`~/Claude/`](https://claude.com/docs/third-party/claude-desktop/data-storage)**。在 Cowork 会话中，Claude 在项目中创建的文件保存在项目的文件夹中。

### 手动移动记忆

在 claude.ai 中，每个用户可以**[要求 Claude 写出他们的记忆](https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude)**并复制文本。然后他们将想要保留的内容粘贴到 Claude Desktop 中的以下位置之一：

- **全局说明：**转到**[设置 > Cowork](https://claude.com/docs/third-party/claude-desktop/data-storage)**，然后单击全局说明旁的“编辑”。它们包含在每个会话中。

- **项目说明：**将文本添加到项目的**[说明](https://claude.com/docs/cowork/guide/projects)**。它们适用于该项目中的每个会话。

## 设置导入工具

Claude Desktop 包含用于此移动的工具，大多数工具处于关闭状态，直到管理员打开它们。我们建议使用最新的 Claude Desktop 版本。

### 打开导入

用户必须打开这两个设置才能从您的 Claude Enterprise 组织导入历史记录。两者默认都处于关闭状态。

1. **在 claude.ai 中：**让所有者或主要所有者转到**组织设置 > 数据和隐私**，然后打开**[允许成员导出自己的数据](https://support.claude.com/en/articles/13346720-export-your-organization-s-data)**。

2. **在 Claude Desktop 中：**打开导入向导。在 Enterprise Admin Console 中，转到**功能****> Claude.ai 数据导入**，然后打开**[启用导入](https://claude.com/docs/third-party/claude-desktop/admin-console)**。使用 MDM 或引导服务器，在**[`claudeAiImport`](https://claude.com/docs/third-party/claude-desktop/configuration#claudeaiimport)**下将`enabled`设置为`true`。

用户然后转到**设置 > 导入和导出**，然后单击“导入…”打开**[导入向导](https://claude.com/docs/third-party/claude-desktop/import)**。

## 可选设置和工具

| **工具**                                                                       | **它的作用**                                                                                                                                                                                    | **如何使用**                                                                                                                                             |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 导入提示                                                                         | 在用户开始新聊天或任务时提示他们导入**[，这样他们就不必查找设置](https://claude.com/docs/third-party/claude-desktop/configuration#claudeaiimport)**。                                                                      | 在`claudeAiImport`下将**[`bannerBehavior`](https://claude.com/docs/third-party/claude-desktop/configuration-changelog)**设置为`off`（默认值）、`detect`或`show`。  |
| 组织允许列表                                                                       | 限制导入仅限于您列出的 claude.ai 组织，仅通过登录。个人账户被拒绝。从文件或早期本地会话的导入已关闭。早期第三方会话的自动导入不受影响。                                                                                                                   | 在`claudeAiImport`下设置**[`allowedOrganizationUuids`](https://claude.com/docs/third-party/claude-desktop/configuration-changelog)**。需要 2.26454.0 或更高版本。 |
| 按组推出                                                                         | 一次为一个组打开 Claude.ai 数据导入。如果用户在多个组中，仅适用设置它的最高排名策略。                                                                                                                                            | 在控制台中使用**[权限策略](https://claude.com/docs/third-party/claude-desktop/admin-console)**。                                                                 |
| **[自动导入早期第三方会话](https://claude.com/docs/third-party/claude-desktop/import)** | 对于之前在较早配置下的第三方运行 Claude Desktop 的计算机，例如移至控制台的 MDM 试点。它每台计算机运行一次，在后台运行，并**[导入](https://claude.com/docs/third-party/claude-desktop/admin-console)**本地数据，如会话、项目和记忆。它不导入 claude.ai 历史记录，这仍需要向导。 | 在企业管理员控制台组织中默认启用，位于**功能** **> Claude.ai 数据导入 > 自动导入早期第三方会话**下。在其他地方，仅当配置了组织 ID 且管理员设置了 `automatic3pImport` 时才运行。                                     |
| 会话导出                                                                         | 将用户的 Claude Desktop 数据（如会话、项目、记忆、计划任务、技能和上传的插件）保存到未加密的 zip 文件。具有相同操作系统和组织的另一设备上的向导可以导入它，除非您限制导入到特定组织。两台设备都需要 4.23118.0 或更高版本。它不包括 Claude Code CLI 会话。                                       | 在 `claudeAiImport` 下设置**[`exportEnabled`](https://claude.com/docs/third-party/claude-desktop/import)**。除非 `enabled` 也是 `true`，否则无效。                  |
| 遥测中的导入事件                                                                     | 为每个 claude.ai 历史记录导入记录**[`desktop_claude_ai_import`](https://claude.com/docs/cowork/changelog)**。对于登录导入，它命名源组织。使用它来计算谁已导入。                                                                  | 需要**[OpenTelemetry 导出](https://claude.com/docs/third-party/claude-desktop/telemetry)**到您的收集器，`otlpDesktopLogLevel` 设置为 `info`，版本 2.26454.0 或更高。      |

**注意：**当用户登录并获取导出时，向导**[需要访问](https://claude.com/docs/third-party/claude-desktop/telemetry)** `claude.ai`、`hcaptcha.com`、`*.hcaptcha.com`、`api.anthropic.com` 和 `storage.googleapis.com`。导入已下载的导出文件不需要它们。在截止日期前保持这些主机开放。如果您使用控制台，除 `storage.googleapis.com` 外的所有主机仍需要。

使用 MDM 或引导服务器，自动更新和其他功能也可以使用 `claude.ai` 和 `api.anthropic.com`，因此在关闭它们之前请检查**[遥测和出口](https://claude.com/docs/third-party/claude-desktop/telemetry#required-egress-paths)**。

**重要：**在符合 HIPAA 的组织中，**[无法启用成员数据导出](https://claude.com/docs/third-party/claude-desktop/import#enable-member-data-export-admins)**，因此用户无法导入 claude.ai 聊天或项目。如果您不限制导入到特定组织，他们仍可以导入计算机上已有的 Cowork 和 Claude Code 会话。

## 用户检查清单

在用户迁移前与每批用户共享此检查清单。有关完整演练，请参阅**[从 claude.ai 导入历史记录](https://claude.com/docs/third-party/claude-desktop/import)**。

**迁移前，在 claude.ai 中**

- 下载 Claude 为您创建的任何 Word、Excel、PowerPoint 和 PDF 文件，并以其他格式导出您需要的任何工件。

- 收集您上传到项目和聊天的原始文件。它们不会被导入。

- 将您的记忆和**Claude 的说明**（在较新的设置布局中称为说明）复制到本地文件。

- 从**自定义 > 技能**下载您构建的技能。

- 记下您使用的连接器，以便您可以请求任何缺失的连接器。

**迁移当天，在 Claude Desktop 中**

1. 打开 Claude Desktop 并按照 IT 团队的说明登录。

2. 转到**设置 > 导入和导出**并单击“导入…”打开向导。

3. 单击“登录到 claude.ai…”，在浏览器中选择您的工作组织并单击“授权。”

4. 返回 Claude Desktop 并单击“获取导出，”然后“继续。”

5. 在**Cowork 和代码**步骤中选择包含本地会话的时间范围，或跳过该步骤。

6. 在**审查**步骤中单击“导入”。

7. 如果出现**重复项**步骤，为每个项目选择“合并到现有”或“保持分离”。

**导入后**

- 打开每个导入的项目。审查并接受其说明，然后添加回您的文件。要在聊天中使用项目的文件夹，请再次将文件夹添加到项目。

- 打开导入的聊天并发送消息。出现提示时单击“信任并继续”。

- 将保存的记忆和说明粘贴到 Claude Desktop。

- 重新连接您使用的连接器。

- 在组织的截止日期再次运行导入以获取较新的聊天。它不会创建重复项。

## 用户的变化

大多数日常工作保持不变。聊天、Cowork、项目、工件、记忆、计划任务、文件上传、技能、插件、连接器和代码都是**[第三方 Claude Desktop 的功能](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**。主要区别在于工作的存储位置和管理员的控制权。语音模式、Claude 安全和 Claude 标签不可用，Chrome 中的 Claude 仅适用于企业管理员控制台。

| **区域**    | **变化内容**                                                                                                                                                                                                                                                                    | **告诉用户的内容**                                                      |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| 用户工作的地方   | **[没有移动或 claude.ai 网络访问](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**。Claude 仅在桌面应用中运行。Cowork 需要**[macOS 14 或更高版本，或 Windows 10 版本 19041 或更高版本](https://claude.com/docs/third-party/claude-desktop/installation)**。                                  | 在您的工作计算机上使用 Claude Desktop。在截止日期后，您将无法使用 Claude 移动应用或 claude.ai。 |
| 登录        | 使用按用户登录，用户通过 AWS **[IAM Identity Center](https://claude.com/docs/third-party/claude-desktop/bedrock)**、Google 或 Workforce Identity 或 Microsoft Entra ID 在应用内登录。使用企业管理员控制台，用户**[使用工作电子邮件登录，重启，然后登录到提供商](https://claude.com/docs/third-party/claude-desktop/admin-console)**。 | 预期会看到新的登录屏幕，并预期偶尔需要再次登录。                                         |
| 历史记录      | 它存储在设备上。除了 SSH 远程会话和任何已制作的导出或副本外，它**[仅存在于该设备上，因此无法恢复已删除的历史记录](https://claude.com/docs/third-party/claude-desktop/data-storage)**。                                                                                                                                           | 您的历史记录不会跟随您到另一台计算机。在更换设备前请与 IT 部门沟通。                             |
| 共享        | **[用户之间无法共享](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**。                                                                                                                                                                                          | 共享输出文件，而不是聊天或项目的链接。                                              |
| 长期任务和计划任务 | 会话是本地的。**[设备必须保持唤醒状态](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**才能运行长期流程和计划任务。                                                                                                                                                                    | 在任务运行期间保持计算机开启和唤醒。                                               |
| 内存和搜索     | 内存存储在设备上。**[聊天历史搜索和每晚摘要在聊天中不可用](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**。                                                                                                                                                                       | 在聊天中，Claude 不会搜索您过去的聊天记录。                                        |
| 导入的聊天     | 当用户在导入的聊天中发送第一条消息时，应用会要求他们确认。**[回复随后来自您配置的提供商](https://claude.com/docs/third-party/claude-desktop/import)**，而不是来自 claude.ai。                                                                                                                                                | 点击 “信任并继续” 以继续。                                                  |
| 网络搜索和获取   | Google Cloud 和 Microsoft Foundry 提供原生搜索。Amazon Bedrock 不提供，因此您需要**[添加搜索提供商](https://claude.com/docs/third-party/claude-desktop/web-tools)**。在聊天和协作中，获取仅限于您允许的主机。                                                                                                            | 搜索可能看起来不同，Claude 可能只能打开已批准的网站。                                   |

**注意：****[统一 Claude](https://claude.com/docs/third-party/claude-desktop/unified-claude)** 将聊天和协作合并为一个体验。它在当前版本的 Claude Desktop 中默认启用，即使您之前关闭了协作。关闭它会同时关闭聊天和协作。代码不受影响。如果您之前的设置仅保留代码开启，最新版本会保持这种状态直到 2026 年 12 月 17 日。请根据统一 Claude 布局来制作您的屏幕截图和培训。

## 管理员的变化

迁移后您需要承担更多责任。您现在负责云账户、设备配置、成本控制和监控。

| **任务**             | **它在第三方 Claude Desktop 中的工作方式**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 您管理设置的位置           | 选择基于网络的**[企业管理控制台](https://claude.com/docs/third-party/claude-desktop/admin-console)**、**[MDM 配置文件](https://claude.com/docs/third-party/claude-desktop/mdm)**或**[引导服务器](https://claude.com/docs/third-party/claude-desktop/bootstrap)**。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| 身份和组               | 在控制台中，您添加用户和组、连接 SSO 和 SCIM，并为每个组设置权限策略。使用 MDM 或引导服务器，您可以通过这些工具和云提供商的身份工具管理访问权限。请参阅**[规划身份和登录](https://support.claude.com/en/articles/18466091)**。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 模型访问               | 您的 Claude Enterprise 组织中的模型设置和自定义角色保留在那里，因此您需要在新配置中重新设置模型访问。模型选择器来自**[`inferenceModels`](https://claude.com/docs/third-party/claude-desktop/models)**，**[控制台策略可以为每个组缩小范围](https://claude.com/docs/third-party/claude-desktop/admin-console)**。请参阅**[规划受限模型和访问计划](https://support.claude.com/en/articles/18466097)**。                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| 成本控制               | 没有座位，也没有 claude.ai 支出限制。在应用中，**[`inferenceMaxTokensPerWindow`](https://claude.com/docs/third-party/claude-desktop/configuration#inferencemaxtokensperwindow)** 限制每个用户在每个设备上每个窗口的令牌数。除非同时设置了 inferenceTokenWindowHours，否则无效。缓存的令牌不计入，因此不会限制您提供商的账单。预算、警报和配额来自您的云提供商。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| 按用户或团队的使用情况报告      | 按用户登录为每个用户在提供商处提供身份。共享密钥则不然。成本归因工具因提供商而异。控制台有可选的**[使用分析](https://claude.com/docs/third-party/claude-desktop/admin-console)**，默认关闭。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| 监控和审计              | **[OpenTelemetry 导出](https://claude.com/docs/third-party/claude-desktop/telemetry)**到您自己的收集器**[涵盖合规 API 和分析 API 的许多功能](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**。默认情况下仅发送元数据。元数据包括用户的身份和连接文件夹的路径。如果您启用技能扫描，扫描摘要可以引用该技能。您可以选择加入提示和工具内容，这些内容仅发送到您的收集器。使用控制台，您还可以启用合规 API 来记录管理员活动。它开始在同一父组织下的每个组织中记录，因此请先检查它捕获的内容。<br>在推出前决定要捕获的内容。LLM 网关可以看到模型请求和响应，但看不到在设备上记录的会话事件，例如权限决定。要在导出中包含 Claude 运行的 shell 命令，请**[将 `toolDetails` 添加](https://claude.com/docs/cowork/monitoring#security-and-privacy)**到 `otlpContentCapture`。长命令会被缩短。您的接收方必须**[接受 protobuf 和 JSON 格式的 OTLP over HTTP](https://claude.com/docs/third-party/claude-desktop/telemetry#collector-endpoint-and-headers)**。如果您的监控平台仅接受其自己的格式，请运行一个 OpenTelemetry Collector 将其转发到该平台。 |
| Claude Code CLI 策略 | **[服务器管理的设置在云提供商上不适用](https://code.claude.com/docs/en/server-managed-settings)**。通过文件、macOS 配置文件或 Windows 注册表传递策略。请参阅**[迁移 Claude Code CLI 用户](https://support.claude.com/en/articles/18466061)**。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

有关网络主机、更新、设备上的保留以及应用发送给 Anthropic 的内容，请参阅**[安装和设置](https://claude.com/docs/third-party/claude-desktop/installation)**、**[遥测和出口](https://claude.com/docs/third-party/claude-desktop/telemetry)**和**[用户身份和本地数据](https://claude.com/docs/third-party/claude-desktop/data-storage)**。

### 分发技能和插件

在第三方 Claude Desktop 中，您通过配置分发技能和插件。有关完整详情，请参阅**[MCP、插件、技能和钩子](https://claude.com/docs/third-party/claude-desktop/extensions)**。

**使用插件市场（推荐）：**在 git 或 HTTPS 上托管市场，并使用 `allowedPluginMarketplaces` 指向它。您可以通过更改一个配置值而不是将文件推送到每个设备来推出更新。您也可以**[在控制台中设置此项](https://claude.com/docs/third-party/claude-desktop/admin-console)**。

### 审查网络搜索和网络获取

Google Cloud 和 Microsoft Foundry 支持原生搜索。在 Google Cloud 上，组织策略默认关闭它，因此请先在那里允许它。Amazon Bedrock 不支持，因此您需要添加搜索提供商。在协作和聊天中，网络获取仅限于您在**[`coworkEgressAllowedHosts`](https://claude.com/docs/third-party/claude-desktop/configuration#coworkegressallowedhosts)** 中列出的主机。在代码中，该列表不会阻止对其他主机的获取。有关谁看到搜索查询以及如何限制、记录或关闭搜索，请参阅**[网络搜索和网络获取](https://claude.com/docs/third-party/claude-desktop/web-tools)**。

在推出前在每个操作系统和您的用户工作的每个网络（例如办公室和家庭）上测试网络搜索。如果您添加搜索提供商，其主机**[必须可通过您的防火墙和代理访问](https://claude.com/docs/third-party/claude-desktop/web-tools#web-search)**。应用**[无法提示输入代理用户名和密码](https://claude.com/docs/third-party/claude-desktop/network-proxy#default-behavior)**。如果您的代理需要登录，例如使用 NTLM，请计划额外的测试。您可能需要在每个设备上运行一个本地转发代理来登录您的代理。另一个选项是**[您托管的远程 MCP 搜索服务器](https://claude.com/docs/third-party/claude-desktop/web-tools#remote-search-mcp)**。

要了解我们如何处理提示注入，请参阅**[减轻浏览器使用中提示注入的风险](https://www.anthropic.com/research/prompt-injection-defenses)**、**[我们如何在各产品中限制 Claude](https://www.anthropic.com/engineering/how-we-contain-claude)**和 Claude Code 文档中的**[安全](https://code.claude.com/docs/en/security)**页面。有关信任中心的安全文档，请参阅**[概述](https://claude.com/docs/third-party/claude-desktop/overview)**的安全态势部分。

如果适用于您，请阅读以下内容：

- **[迁移 Claude Code CLI 用户](https://support.claude.com/en/articles/18466061)：**适用于开发人员使用 Claude Code CLI 的组织。它涵盖策略如何到达每个设备、什么会保留以及开发人员的检查清单。

- **[规划身份和登录](https://support.claude.com/en/articles/18466091)：**您的用户将看到的每次登录，以及如果您也在更改身份提供商 (IdP) 时该怎么办。

- **[规划受限模型和访问计划](https://support.claude.com/en/articles/18466097)：**适用于参与网络验证计划或其他访问计划的组织，或使用自定义角色来限制模型的组织。

- **[比较云提供商](https://support.claude.com/en/articles/18466110)：**Amazon Bedrock、Google Cloud 和 Microsoft Foundry 在数据处理、登录、网络搜索和公共部门需求方面的差异。

## 在试点前要做的决定

这些选择设置您的配置和用户体验。从这四个开始，因为它们对试点影响最大。

| **决定**            | **选项**                                                                                                                                                                                                                       | **需要考虑的事项**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. 选择哪个提供商        | Amazon Bedrock、Google Cloud 的 Agent Platform、Microsoft Foundry 或您自己的 LLM 网关                                                                                                                                                  | 您现有的云协议和身份设置、谁处理对话数据、原生网络搜索、数据驻留选项和公共部门需求。在**[比较云提供商](https://support.claude.com/en/articles/18466110)**中进行比较。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| 2. 配置如何到达设备       | **[企业管理控制台](https://claude.com/docs/third-party/claude-desktop/admin-console)**、**[MDM](https://claude.com/docs/third-party/claude-desktop/mdm)** 或**[引导服务器](https://claude.com/docs/third-party/claude-desktop/bootstrap)** | 尽早做出选择，并首先评估控制台。您无需为每次更改运行服务器或推送配置文件。<br>**控制台：**它提供 SSO、SCIM、每个组的策略和**[Chrome 中的 Claude](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**。它在每次启动时都需要 `api.anthropic.com`，用户需要 Anthropic 账户才能登录。登录还需要 `claude.ai`、`hcaptcha.com` 和 `*.hcaptcha.com`。如果设备**[在启动时无法访问 `api.anthropic.com`](https://claude.com/docs/third-party/claude-desktop/admin-console#limitations)**，应用将无法连接到您的提供商，直到它能够连接。推理连接适用于整个组织。<br>**MDM：**它不需要 `api.anthropic.com` 来获取配置，也不需要 Anthropic 账户来登录，尽管**[其他功能可以使用该主机](https://claude.com/docs/third-party/claude-desktop/telemetry#required-egress-paths)**。不同组的设置意味着需要单独的配置文件。每次更改都意味着推送更新的配置文件。如果另一个团队运行您的 MDM，请就部署速度达成一致。<br>**引导服务器：**您托管并运行它。如果在应用启动时**[无法访问](https://claude.com/docs/third-party/claude-desktop/bootstrap#availability)**，应用将没有推理提供商，直到它恢复，因此请在多个副本或区域中运行它。<br>**您自己的工具：**如果您构建工具来传递设置，您还需要维护和版本控制它。如果它写入**[本地配置文件](https://claude.com/docs/third-party/claude-desktop/configuration-format#which-configuration-source-applies-on-a-device)**，用户可以编辑这些设置。<br>**在所有情况下：**每个设备以一种方式传递您的主配置。在 Claude Desktop 2.37337.0 或更高版本上，登录到任何使用控制台的组织的用户可以**[将设备移出托管配置文件](https://claude.com/docs/third-party/claude-desktop/admin-console#keep-a-device-on-its-managed-profile)**。为了防止这种情况，在配置文件中将 `deploymentMode` 设置为 `3p`（4.16336.0 之前的设备还需要 `disableDeploymentModeChooser`）。要仅允许您的组织，请设置 `forceLoginOrgUUID`，它可以与控制台一起使用。 |
| 3. 用户如何向提供商进行身份验证 | 应用内按用户登录、凭证助手、您自己的身份验证代理或网关，或共享密钥                                                                                                                                                                                            | 按用户登录为每个用户在提供商处提供身份。共享密钥（Bearer 令牌、服务账户密钥、API 密钥）用于单个团队的概念验证。请参阅**[Bedrock](https://claude.com/docs/third-party/claude-desktop/bedrock)**、**[Google Cloud](https://claude.com/docs/third-party/claude-desktop/vertex)** 或**[Foundry](https://claude.com/docs/third-party/claude-desktop/foundry)** 的设置指南。控制台**[不提供静态密钥或 Bearer 令牌](https://claude.com/docs/third-party/claude-desktop/admin-console)**。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| 4. 用户可以导入什么       | 开放导入，或**[`allowedOrganizationUuids`](https://claude.com/docs/third-party/claude-desktop/configuration-changelog)** 设置为您的组织 ID                                                                                                | 设置列表后，用户仅通过登录导入，且仅从您列出的组织导入。个人账户被拒绝。从文件和计算机上的早期会话导入已关闭。该列表需要 Claude Desktop 2.26454.0 或更高版本，早期版本会忽略它。决定使用个人账户进行工作的用户是否应该带入该历史记录。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

然后解决其余问题：

- 每个组获得应用的哪些部分：**[统一 Claude](https://claude.com/docs/third-party/claude-desktop/unified-claude)**（结合了聊天和协作）和代码。

- 用户获得哪些权限模式。在推出前与您的安全团队达成一致。如果用户必须批准每个操作，他们可能会开始不阅读就批准，因此决定哪些操作需要提示。决定**[自动模式](https://claude.com/docs/third-party/claude-desktop/configuration#automodeenabled)**（新代码会话通常在其中启动，除非您关闭它）、**[绕过权限模式](https://claude.com/docs/third-party/claude-desktop/configuration#disablebypasspermissionsmode)**和**[哪些工具始终询问](https://claude.com/docs/third-party/claude-desktop/configuration#builtintoolpolicy)**。

- 网络搜索的工作方式。请参阅**审查网络搜索和网络获取**。

- 如何控制成本：应用内**[令牌上限](https://claude.com/docs/third-party/claude-desktop/configuration#inferencemaxtokensperwindow)**，加上您提供商的预算、警报和配额。

- 向 Anthropic 和您自己的收集器发送什么**[遥测](https://claude.com/docs/third-party/claude-desktop/telemetry)**。

- 如何处理设备上的数据：**[保留和法律保留](https://claude.com/docs/third-party/claude-desktop/data-storage)**，以及用于设备更改的会话导出。对于法律保留，将聊天、协作和代码保留期设置为 3650 天，以供保留中的用户使用。`sessionRetentionHold` 设置在 2026 年 12 月 17 日停止工作。

- 如何**配置 Claude Code CLI 用户**。

- 何时**进行任何 IdP 更改**：在第一波之前、波之间或最后一次导入之后。

- 关于**受限模型和访问计划**要做什么：为云账户打开它们、为该工作保留一些 Claude Enterprise 席位，或停止使用它们。

## 规划您的迁移

我们建议分**波次**迁移，一次一组用户，并有一个**重叠期**，在此期间 Claude Enterprise 和 Claude Desktop on 3P 都在运行。设置明确的截止日期，并为每个用户提供一份简短的检查清单，以在迁移前完成。保持 claude.ai 可用，直到最后一波已导入。

### 第 1 阶段：规划迁移

**涉及人员：**AI 项目负责人、IT 和您的 Anthropic 账户团队

- 做出**试点前要做的决定**中列出的选择。

- 为云账户、设备管理、身份、网络和通信与培训各指定一个所有者。

- 列出您的 Claude Enterprise 组织中的内容：用户、共享项目、您的组织提供的技能和正在使用的连接器。

- 列出依赖 Claude 移动应用或浏览器中 claude.ai 的用户，例如从手机或平板电脑工作的高管。Claude Desktop on 3P **[两者都没有](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**，因此决定这些用户是否保留 Claude Enterprise 席位。

- 就如何衡量成功、波次顺序和 claude.ai 的截止日期达成一致。

- 计算谁使用 Claude Code CLI。记下他们依赖的服务器管理设置、云会话和 claude.ai 连接器。

- 列出您的用户将看到的每个登录，并记下同期计划的任何 IdP 更改。请参阅**[规划身份和登录](https://support.claude.com/en/articles/18466091)**。

- 列出您今天使用的访问计划、受限模型和自定义角色。请参阅**[规划受限模型和访问计划](https://support.claude.com/en/articles/18466097)**。

### 第 2 阶段：设置新环境

**涉及人员：**云、设备管理、网络和身份管理员

- 使用您的提供商启用或部署模型、授予推理角色并为您的用户请求足够的配额。按照**[比较云提供商](https://support.claude.com/en/articles/18466110)**中链接的提供商设置指南进行操作。

- 将您的 IdP 连接到云提供商并与小组测试登录。

- 在每个硬件型号上运行**[协作就绪检查](https://claude.com/docs/third-party/claude-desktop/installation)**。协作需要硬件虚拟化，在 Windows 上需要虚拟机平台功能。

- 打包应用并与管理您的设备的团队测试自动部署。为此工作留出时间。在 Windows 上，使用 `.msix` 包，协作需要它。**[安装它需要管理员权限](https://claude.com/docs/third-party/claude-desktop/installation#install-the-app)**。

- 在控制台中构建配置，或在应用中构建并导出以供 MDM 使用。为每个设备上的最新 Claude Desktop 版本做计划。

- 决定用户不应该更改哪些设置，并在第一个用户登录前设置每一个。例如，代码提供自动模式，除非您将**[`autoModeEnabled`](https://claude.com/docs/third-party/claude-desktop/configuration#automodeenabled)** 设置为 `false`。从控制台、MDM 配置文件或引导服务器传递这些设置，因为**[用户可以编辑本地配置文件](https://claude.com/docs/third-party/claude-desktop/configuration-format#which-configuration-source-applies-on-a-device)**。仅控制台不会将设备保留在您的设置上。要强制执行它们，还要**[在机器级托管配置文件中部署 `forceLoginOrgUUID`](https://claude.com/docs/third-party/claude-desktop/admin-console#limit-claude-desktop-to-your-organization)**。该密钥限制用户可以在 Claude Desktop 中登录的组织，因此列出您的企业管理控制台组织和他们在重叠期间仍需要的所有其他组织。然后检查测试设备上用户仍然可以更改哪些设置。使用 MDM，确认应用内配置窗口**[以只读方式打开](https://claude.com/docs/third-party/claude-desktop/installation#verifying-the-deployment)**。在 Windows 上，拼写错误的值也会锁定窗口，因此请将诊断报告的 `managed-config.txt` 中的每个值与您设置的值进行比较。

- 打开**[必需的网络主机](https://claude.com/docs/third-party/claude-desktop/telemetry)**，并通过其**[签名身份](https://claude.com/docs/third-party/claude-desktop/installation)**在您的端点安全工具中允许该应用。

- 配置连接器、插件市场、网络搜索、横幅、令牌上限和 OpenTelemetry 导出。

- 在提供商处和应用的模型列表中设置模型访问规则。

- 与您的提供商设置预算警报，并在第一个用户登录前启用成本归因。

- 对于 CLI 用户，使用提供商变量、固定模型、模型限制和 `allowedProviders` 部署 Claude Code 托管设置。请参阅 **[移动 Claude Code CLI 用户](https://support.claude.com/en/articles/18466061)**。

- 如果您持有访问计划授权，请在验证门户中链接云账户并启用访问。预留充足的时间。

### 第 3 阶段：准备现有用户

**涉及人员：**所有者、主要所有者和通信负责人

- 启用成员数据导出和导入向导。请参阅 **启用导入**。

- 选择任何可选的导入设置，例如导入提示、组织允许列表和按组推出。

- 确认设备可以访问 **[登录导入所需的主机](https://claude.com/docs/third-party/claude-desktop/telemetry)**。

- 让主要所有者运行 **[组织导出](https://support.claude.com/en/articles/13346720-export-your-organization-s-data)**以供记录。下载链接在 24 小时后过期，可能只能使用一次，因此请立即下载文件。

- 向每一批用户发送 **用户检查清单**，以及您的截止日期。

- 向开发人员发送 **开发人员检查清单**，以便他们在截止日期前拉取云会话。

- 推迟对 claude.ai SSO 连接和 SCIM 目录的任何更改，直到最后一次导入，或在各批次之间安排切换。

### 第 4 阶段：运行试点，然后分批推进

**涉及人员：**设备管理、您的倡导者和您的 Anthropic 账户团队

- **[使用 MDM 或引导服务器，](https://claude.com/docs/third-party/claude-desktop/mdm)** **[先部署配置，然后部署应用](https://claude.com/docs/third-party/claude-desktop/mdm)**，以便用户直接在第三方上打开 Claude Desktop 应用。使用控制台时，**[先保存配置，然后让用户安装应用并登录](https://claude.com/docs/third-party/claude-desktop/installation)**。

- 使用 **帮助 > 故障排除 > 生成诊断报告** 检查设备。在导出的文件中，**[`deployment-mode.txt`](https://claude.com/docs/third-party/claude-desktop/installation)** 显示应用是否处于第三方模式。

- 使用混合角色进行试点。在适用的情况下，包括一名 CLI 开发人员、每个受限模型的用户和来自每个身份组的用户。

- 测试导入、每个连接器、用户工作的每个网络上的网络搜索、主要所有者的历史记录，以及用户在请求被标记时看到的内容。还要测试恢复 CLI 会话、按用户的成本报告，以及支出在您的提供商计费控制台中的显示方式。

- 修复试点发现的问题并更新您的用户指南。然后推进每一批，进行启动培训和办公时间。

- 对于每一批，跟踪有多少用户已安装应用、已登录、已导入以及在第二周仍然活跃。您的遥测中的 **[`desktop_claude_ai_import`](https://claude.com/docs/cowork/changelog)** 事件针对每个 claude.ai 历史记录导入进行记录。用户可以多次导入，因此请计算不同的用户。这是一个信息级事件，因此请将 `otlpDesktopLogLevel` 设置为 `info` 以接收它。

### 第 5 阶段：关闭 Claude Enterprise

**涉及人员：**主要所有者、身份管理员和网络管理员

- 在截止日期，让用户完成他们在 claude.ai 中的工作并最后一次运行导入。在您在网络上阻止 claude.ai 之前执行此操作，因为登录导入需要它。如果您使用控制台，**[不要阻止 `claude.ai`、`api.anthropic.com` 或 `hcaptcha.com` 主机](https://claude.com/docs/third-party/claude-desktop/admin-console#where-your-data-goes)**。改为使用租户限制将 `claude.ai` 和 `api.anthropic.com` 限制为您的组织。使用 MDM 或引导服务器时，应用还会 **[从 `claude.ai` 和 `api.anthropic.com` 读取其更新源](https://claude.com/docs/third-party/claude-desktop/telemetry#required-egress-paths)**。在阻止它们之前，将 `updateViaUpdatesHost` 设置为 `true` 并允许 `releases.claude.com`，或关闭自动更新并自己分发新版本。在代码会话中，网络获取使用 `api.anthropic.com` 检查每个域，如果无法检查则拒绝获取，因此在阻止该主机之前也要将 `skipWebFetchPreflight` 设置为 true。在截止日期后保持管理员访问权限，以便您可以运行最终导出并删除用户。

- 将用户保留在新部署中。使用 MDM 或引导服务器，在设置您的提供商或引导密钥的托管配置文件中将 **[`deploymentMode`](https://claude.com/docs/third-party/claude-desktop/mdm#restrict-a-device-to-one-deployment-mode)** 设置为 `3p`（Claude Desktop 4.16336.0 或更高版本）。在每个设备更新之前，还要在配置文件中设置 `disableDeploymentModeChooser` 为 `true`，并在引导服务器中的响应中设置。不要在使用控制台的设备上设置任何一个密钥。使用控制台时，启用 **[在 Claude Desktop 中要求此组织](https://claude.com/docs/third-party/claude-desktop/admin-console#users-in-more-than-one-claude-organization)**，以便两个组织的成员必须切换。该设置不强制登录。要强制登录，在最后一批移动后从 `forceLoginOrgUUID` 中删除您的 Claude Enterprise 组织 ID，除了保留 Claude Enterprise 的用户。

- 对于 CLI 用户，将 `allowedProviders` 缩小到新提供商，除了在用户在 Claude Desktop 中保持登录到 claude.ai 的计算机上，并停用您的服务器托管设置。

- 使用 **[限制组织创建](https://support.claude.com/en/articles/13132885-set-up-single-sign-on-sso)** 和 **[租户限制](https://support.claude.com/en/articles/13198485-enforce-network-level-access-control-with-tenant-restrictions)** 控制您的域和网络上的个人账户。如果您使用控制台，请在允许列表中包含新组织的 ID。

- 运行最终组织导出和 **[审计日志导出](https://support.claude.com/en/articles/9970975-access-audit-logs)**。被删除用户的数据 **[仍包含在主要所有者的导出中](https://support.claude.com/en/articles/12053672-what-happens-to-a-user-s-data-when-they-are-removed-from-a-team-or-enterprise-organization)**。

- 某些工作可能必须保留在 Claude Enterprise 上，例如在您的云提供商尚不支持的访问计划下，或对于需要移动应用的用户。如果是这样，请保留这些席位并缩小组织规模，而不是关闭它。

- 删除用户。使用 SCIM，您从 IdP 应用中删除的用户 **[会自动被删除](https://support.claude.com/en/articles/13133195-set-up-jit-or-scim-provisioning)**。使用 JIT，他们会保留在您的成员列表中，直到管理员删除他们。您仅在 Claude 中删除的用户会在他们下次使用 SSO 登录时再次被添加。如果您的企业管理员控制台组织共享 SCIM 目录，请先检查从 IdP 应用中删除用户如何影响它。被删除的用户 **[立即失去访问权限](https://support.claude.com/en/articles/13133750-manage-members-on-team-and-enterprise-plans)**，但删除 **[不会删除他们的账户或数据](https://support.claude.com/en/articles/12053672-what-happens-to-a-user-s-data-when-they-are-removed-from-a-team-or-enterprise-organization)**。

- 要更快地清除存储的数据，您可以缩短您的 **[自定义数据保留期](https://support.claude.com/en/articles/10440198-configure-custom-data-retention-controls-for-enterprise-plans)**。超出新期限的任何内容都会在您保存后立即安排永久删除，因此请先完成导出。最短期限是 30 天，它不涵盖您已删除用户的记忆。项目中的聊天遵循项目保留期，使用公共链接发布的工件不会被删除。

- 在关闭任何内容之前，请与您的 Anthropic 账户团队确认新部署仍然依赖哪些域和 SSO 设置。然后与他们讨论结束计划。要删除组织，主要所有者 **[向支持部门发送请求](https://support.claude.com/en/articles/12053672-what-happens-to-a-user-s-data-when-they-are-removed-from-a-team-or-enterprise-organization)**。

## 获取帮助

迁移后，请与您的云提供商联系推理服务，并与我们联系桌面应用和您的推出。

| **主题**                    | **联系人**                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 推理服务、模型访问、配额和云计费          | 您的云提供商：**[AWS 支持或您的 AWS 账户经理](https://support.claude.com/en/articles/7996921-i-use-claude-in-amazon-bedrock-who-do-i-contact-for-customer-support-inquiries)**、**[Google Cloud 支持](https://docs.cloud.google.com/gemini-enterprise-agent-platform/machine-learning/support/getting-support)**或 **[Microsoft 支持](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/claude-models-hosting-comparison)** |
| 安装或配置 Claude Desktop      | 您的 Anthropic 账户团队。**[先生成诊断报告](https://claude.com/docs/third-party/claude-desktop/installation)**并发送文件。不包括对话，但日志行可以引用文件、消息或工具输出中的文本，因此在发送前请查看文件。                                                                                                                                                                                                                                                                             |
| 设置企业管理员控制台组织              | **[您的 Anthropic 账户团队](https://claude.com/docs/third-party/claude-desktop/admin-console)**                                                                                                                                                                                                                                                                                                                                   |
| 删除被删除用户的数据或删除组织           | **[Anthropic 支持](https://support.claude.com/en/articles/12053672-what-happens-to-a-user-s-data-when-they-are-removed-from-a-team-or-enterprise-organization)**。主要所有者必须请求删除组织。                                                                                                                                                                                                                                               |
| 云提供商上的 Claude Code CLI 问题 | 您的 Anthropic 账户团队。先运行 `/feedback`。在云提供商上，它 **[写入本地存档](https://code.claude.com/docs/en/data-usage)**，在您发送该文件之前，没有任何内容离开计算机。                                                                                                                                                                                                                                                                                                  |
| IdP 更改后没有人可以登录 claude.ai  | **[Anthropic 支持](https://support.claude.com/en/articles/13132885-set-up-single-sign-on-sso)**，来自所有者或主要所有者的电子邮件地址                                                                                                                                                                                                                                                                                                            |
| 访问计划和受限模型                 | 用于应用程序和关联云账户的 **[验证门户](https://support.claude.com/en/articles/14604842-cyber-verification-program)**。用于有限可用性模型和补助的 **[Anthropic 账户团队](https://support.claude.com/en/articles/15425695-covered-models)**。                                                                                                                                                                                                                    |
| 规划您的推出                    | **[Anthropic 账户团队](https://claude.com/blog/the-full-claude-desktop-experience-on-aws-google-cloud-and-microsoft-foundry)**                                                                                                                                                                                                                                                                                                  |

## 相关资源

- **[Claude Desktop 第三方概览](https://claude.com/docs/third-party/claude-desktop/overview)**

- **[功能对比](https://claude.com/docs/third-party/claude-desktop/feature-matrix)**

- **[从 claude.ai 导入历史记录](https://claude.com/docs/third-party/claude-desktop/import)**

- **[安装和设置](https://claude.com/docs/third-party/claude-desktop/installation)**

- **[使用企业管理控制台部署](https://claude.com/docs/third-party/claude-desktop/admin-console)**

- **[配置参考](https://claude.com/docs/third-party/claude-desktop/configuration)**

- **[Claude Code 企业部署概览](https://code.claude.com/docs/en/third-party-integrations)**

- **[导出您的组织数据](https://support.claude.com/en/articles/13346720-export-your-organization-s-data)**

- **[切换到不同的身份提供商 (IdP)](https://support.claude.com/en/articles/13443687-switching-to-a-different-identity-provider-idp)**

- **[网络验证计划](https://support.claude.com/en/articles/14604842-cyber-verification-program)**

- **[支持的模型](https://support.claude.com/en/articles/15425695-covered-models)**

- **[将 Claude Desktop 从云提供商迁移到 Claude Enterprise](https://claude.com/docs/third-party/claude-desktop/move-to-claude-enterprise)**