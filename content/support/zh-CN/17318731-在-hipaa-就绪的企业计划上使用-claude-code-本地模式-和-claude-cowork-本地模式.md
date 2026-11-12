# 在 HIPAA 就绪的企业计划上使用 Claude Code（本地模式）和 Claude Cowork（本地模式）

HIPAA 配置是企业计划上的组织设置。它将本地 Claude Code 和 Claude Cowork 会话纳入您与 Anthropic 的业务关联协议 (BAA) 的保护范围。云会话（包括云中的 Claude Code 会话和云中的 Cowork）不在保护范围内。本文将它们称为 Claude Code（本地模式）和 Cowork（本地模式）。它们在以下位置之一运行：

- 终端中的 Claude Code

- Claude Desktop 的代码部分中的 Claude Code

- Claude Desktop 中的 Cowork

应用配置后，成员可以在 **[检查您的 BAA 涵盖的内容](#h_eba343b314)** 中的条件下，使用这两种产品处理受保护的健康信息 (PHI)。应用配置还会关闭整个组织中的某些功能。

在具有 Anthropic 账户团队的企业计划上可用。只有主要所有者可以应用 HIPAA 配置。

本文适用于应用配置的主要所有者，以及帮助决定是否应用、准备和确认配置的任何所有者。

HIPAA 就绪"意味着产品已配置为支持您在 BAA 下的 HIPAA 义务。合规性仍然是您组织的责任。"

**注意：**

- 如果您的组织尚未启用 HIPAA，请从 **[HIPAA 就绪企业计划](https://support.claude.com/en/articles/13296973-hipaa-ready-enterprise-plans)** 开始。启用 HIPAA 会将聊天纳入您的 BAA 保护范围。仅此步骤不会将 Claude Code（本地模式）或 Cowork（本地模式）纳入您的 BAA 保护范围。

- 如果您准备成员的计算机，请参阅 **[为 HIPAA 就绪组织设置 Claude Code（本地模式）](https://code.claude.com/docs/en/hipaa-setup)** 和 **[为 HIPAA 就绪组织设置 Cowork（本地模式）](https://claude.com/docs/cowork/hipaa-setup)**。

- 如果您想检查特定功能是否在您的 BAA 保护范围内，请参阅 **[HIPAA 实体实施指南](https://trust.anthropic.com/resources?s=l1wrssd9hsbi4gak0tp5a6&name=%5Banthropic%5D-hipaa-ready-offering-implementation-guide)** 中的合格服务列表和 **[商业客户业务关联协议 (BAA)](https://support.claude.com/en/articles/8114513-business-associate-agreements-baa-for-commercial-customers)**。

## 检查您的 BAA 涵盖的内容

应用 HIPAA 配置后，您的 BAA 涵盖表中的每个产品，条件在第二列中。应用配置会关闭 Cowork 和 Claude Desktop 的代码部分（即使它们已打开），因此所有者必须在之后逐个打开它们。

| **产品**                             | **在以下情况下在您的 BAA 保护范围内**                                               |
| ---------------------------------- | --------------------------------------------------------------------- |
| 终端中的 Claude Code                   | 成员使用其 Claude Enterprise 账户登录，Claude Code 直接连接到 Anthropic，而不是通过云提供商或网关 |
| Claude Desktop 的代码部分中的 Claude Code | 所有者已在 **组织设置 > Claude Code** 中打开"Desktop"切换                           |
| Claude Desktop 中的 Cowork           | 所有者已在 **组织设置 > Cowork** 中打开"为您的组织启用"切换                                |

您不需要零数据保留 (ZDR) 来应用配置。如果您的组织对 Claude Code 启用了 ZDR，请参阅 **[用配置替换 ZDR](#h_a6b761685a)**。

### 您的 BAA 不涵盖的产品和功能

将 PHI 保留在这些产品和功能之外。即使应用了配置，您的 BAA 也不涵盖它们：

- VS Code 和 JetBrains 的 Claude Code 扩展，应用配置后继续工作

- 任何标记为测试版或研究预览的功能

- 通过云提供商（如 Amazon Bedrock）、LLM 网关或自定义基础 URL 连接的 Claude Code 会话

- 使用 Claude Console API 密钥的 Claude Code 会话，除非 Claude Console 组织启用了 ZDR

- 连接器、MCP 服务器、插件、桌面扩展、Chrome 中的 Claude 和技能，包括所有者打开它们之后

- 云和远程功能，如 Claude Code 云会话，配置使其不可用

要求您的 IT 团队 **[部署托管设置](https://code.claude.com/docs/en/hipaa-setup#deploy-managed-settings)**，这是每台计算机上的策略文件，指导成员使用 Claude Enterprise 账户登录并拒绝云提供商和网关。有关合格服务的完整列表，请查看 **[实施指南](https://trust.anthropic.com/resources?s=l1wrssd9hsbi4gak0tp5a6&name=%5Banthropic%5D-hipaa-ready-offering-implementation-guide)** 和 **[商业客户业务关联协议 (BAA)](https://support.claude.com/en/articles/8114513-business-associate-agreements-baa-for-commercial-customers)**。

## 要求 Anthropic 打开 HIPAA 配置

您可以使用 **组织设置 > 数据和隐私** 中的"应用 HIPAA 配置"按钮来应用配置。在 Anthropic 为您的组织打开该按钮之前，该按钮处于灰显状态。请要求您的 Anthropic 账户团队打开它。目前，HIPAA 配置仅适用于具有 Anthropic 账户团队的组织。Anthropic 支持无法为您的组织打开它。

请参阅 **[应用配置前的保护范围](#h_415ff36247)** 和 **[商业客户业务关联协议 (BAA)](https://support.claude.com/en/articles/8114513-business-associate-agreements-baa-for-commercial-customers)**，了解如果您尚未应用配置，您的 BAA 涵盖的内容。

### 应用配置前的保护范围

在应用配置之前，成员仍然可以在所有者打开的地方使用 Claude Code（本地模式）和 Cowork（本地模式），但您的 BAA 可能不涵盖它们的使用。对于具有和不具有 Claude Code 零数据保留 (ZDR) 的组织，保护范围有所不同：

- **不使用 Claude Code ZDR：** 您的 BAA 不涵盖 Claude Code 或 Cowork，因此不要使用它们处理 PHI。

- **使用 Claude Code ZDR：** 您的 BAA 通过 ZDR 涵盖 Claude Code。它不涵盖 Cowork，因此不要使用 Cowork 处理 PHI。

您的账户团队可以告诉您您的组织是否对 Claude Code 启用了 ZDR。

### 用配置替换 ZDR

配置旨在替换 Claude Code（本地模式）的 ZDR。应用配置不会关闭 ZDR。如果您希望保持通过 ZDR 的现有访问权限，请不要应用配置。

如果您在 Claude for Enterprise 中启用了 ZDR，请向您的账户团队发送电子邮件以获取帮助切换到 HIPAA 配置。

ZDR 移除后，Anthropic 会按照对您组织聊天数据的处理方式，将 Claude Code 数据保留在标准保留期内。

## 准备您的组织

在应用配置之前完成这些任务。

### 查看成员的变化

该表显示成员最可能注意到的变化，以及在应用配置之前对每个变化的处理方式。**[HIPAA 实体实施指南](https://trust.anthropic.com/resources?s=l1wrssd9hsbi4gak0tp5a6&name=%5Banthropic%5D-hipaa-ready-offering-implementation-guide)** 和 **[商业客户业务关联协议 (BAA)](https://support.claude.com/en/articles/8114513-business-associate-agreements-baa-for-commercial-customers)** 列出了您的 BAA 下的合格服务。

| **变化内容**                                                           | **申请前要做的事**                                        |
| ------------------------------------------------------------------ | -------------------------------------------------- |
| Cowork、Claude Desktop 的代码部分、技能和 Chrome 中的 Claude 会关闭，即使所有者已将其打开    | 列出您想要恢复的项目以及每个项目的批准者                               |
| 启用 HIPAA 后默认关闭的所有其他设置都会再次关闭，包括聊天设置                                 | 列出自您启用 HIPAA 以来所有者已打开的设置                           |
| 每个连接器都会关闭，其工具权限更改为**已阻止**，除非所有者在添加或启用它时接受了条款                       | 记录每个连接器的工具权限，因为重新打开连接器不会恢复它们                       |
| 内置 GitHub 集成会关闭                                                    | 注意您是否想要恢复它                                         |
| 每个桌面扩展都会从您组织的允许列表中删除，除非所有者在添加它时接受了条款                               | 列出您想要再次添加的扩展                                       |
| 几分钟内，成员无法启动 Claude Code 云会话或在云中使用 Cowork。已在运行的会话会停止。成员将失去对它们的访问权限。 | 要求成员完成该工作并保存他们需要的内容                                |
| Claude Code 远程控制和 Claude Tag 不可用。Claude Design 会关闭，直到所有者将其打开       | 要求使用它们的成员完成该工作                                     |
| Claude Cowork 中的计算机访问对在云中运行的会话和计划任务不可用                             | 要求使用它们的成员完成该工作，并告诉他们 Cowork（本地模式）中的计划任务仍在他们的计算机上运行 |
| 成员给予 Cowork 中的计划任务和 Claude Desktop 代码部分的批准会过期                      | 告诉成员期望再次批准他们的计划任务                                  |

聊天使用相同的连接器、技能和 Chrome 中的 Claude 设置，因此当这些关闭时，它们在聊天中也会关闭。

### 让您的团队审查条款

当您应用配置时，您接受 BAA 并确认您已审查**[HIPAA 实体实施指南](https://trust.anthropic.com/resources?s=l1wrssd9hsbi4gak0tp5a6&name=%5Banthropic%5D-hipaa-ready-offering-implementation-guide)**，其中包括您 BAA 下的合格服务列表。在应用配置之前，请要求您的法律和合规团队审查这两份文件。

### 要求您的 IT 团队准备计算机

要求您的 IT 团队遵循**[为符合 HIPAA 的组织设置 Claude Code（本地模式）](https://code.claude.com/docs/en/hipaa-setup)**。如果成员使用 Cowork，还要求团队遵循**[为符合 HIPAA 的组织设置 Cowork（本地模式）](https://claude.com/docs/cowork/hipaa-setup)**。他们将 Claude Code 和 Claude Desktop 更新到最新版本，允许网络访问 Anthropic 的服务器，并部署托管设置和 Claude Desktop 策略。

更新很重要，因为一旦应用配置，Anthropic 的服务器会拒绝来自低于最低版本的请求

### 选择一个时间并告知成员

在您组织的安静时间应用配置，因为 Cowork、Claude Desktop 的代码部分和某些连接器会保持关闭状态，直到所有者将其打开。之后保持所有者可用，并计划时间分别打开每个连接器。

告诉成员日期和时间、哪些功能会关闭以及您计划重新打开的功能。

## 应用 HIPAA 配置

要应用配置，您必须是主所有者并拥有经过验证的电子邮件地址。其他所有者和管理员无法应用它，因为应用它包括为您的组织接受 BAA。

配置适用于您现有的 Claude Enterprise 组织，即您的成员用于聊天的同一个组织，而不是单独的帐户。它适用于 Claude Code 和 Cowork，以及每个成员。

**警告：**当您应用 HIPAA 配置时，某些功能会在您确认后立即在整个组织中开始关闭。所有者可以重新打开某些功能，但云功能（如 Claude Code 云会话）保持不可用。

要应用配置：

1. 以主所有者身份登录 Claude，然后转到**[组织设置 > 数据和隐私](https://claude.ai/admin-settings/data-privacy-controls)**。

2. 在"合规"部分中，找到 Claude Code（本地模式）和 Cowork（本地模式）的行。

3. 选择"应用 HIPAA 配置"。对话框打开。

4. 在对话框中，查看更改内容，然后下载 HIPAA 实体实施指南和 BAA（如果您的组织没有备案）。对话框中的"应用 HIPAA 配置"按钮在您下载它们之前保持灰显。选择它接受 Anthropic 的更新 BAA。

5. 选择"应用 HIPAA 配置"以确认。一条消息确认配置已应用。

如果按钮灰显，请检查您是否以主所有者身份登录以及您的电子邮件地址是否已验证。如果仍然灰显，**[请要求 Anthropic 将其打开](#h_a54ff1eb7a)**。

## 重新打开功能

应用配置后，Cowork、Claude Desktop 的代码部分、技能和 Chrome 中的 Claude 会关闭，每个没有所有者接受条款的连接器也会关闭。所有者打开您的组织批准的连接器。**[HIPAA 实体实施指南](https://trust.anthropic.com/resources?s=l1wrssd9hsbi4gak0tp5a6&name=%5Banthropic%5D-hipaa-ready-offering-implementation-guide)**和**[商业客户的业务关联协议 (BAA)](https://support.claude.com/en/articles/8114513-business-associate-agreements-baa-for-commercial-customers)**列出了您 BAA 下的合格服务。

打开功能不会改变您的 BAA 是否涵盖它。

连接器需要更多步骤，因为您需要分别打开每个连接器。转到**组织设置 > 连接器**，打开连接器，在横幅中选择"启用"，选中复选框以接受条款，然后单击"继续"。然后再次设置连接器的工具权限。

## 确认配置已应用

要求成员重新启动 Claude Code 和 Claude Desktop，以便两者立即获得配置。如果不重新启动，运行中的 Claude Code 会话会在大约一小时内获得它。Claude Desktop 可能要等到成员重新启动或重新加载应用程序才能获得它。

组织设置、Claude Code 和 Claude Desktop 在应用配置后各自显示配置：

- **组织设置：**转到**[组织设置 > 数据和隐私](https://claude.ai/admin-settings/data-privacy-controls)**。确认 Claude Code（本地模式）和 Cowork（本地模式）的行显示它们已为此组织配置 HIPAA，并显示您应用配置的日期。

- **Claude Code：**要求成员在会话中运行`/status`。确认**状态**选项卡在`组织配置`行上列出`HIPAA`。

- **Claude Desktop：**确认标题栏显示**HIPAA 已配置**标签。在 Mac 上，打开侧边栏以查看它。

告诉成员在他们自己的计算机上看到`/status`中的`HIPAA`或**HIPAA 已配置**标签之前不要处理 PHI。

如果成员的计算机上缺少其中任何一个，请要求您的 IT 团队按照**[确认计算机上的配置](https://code.claude.com/docs/en/hipaa-setup#confirm-the-configuration-on-a-computer)**进行操作，该指南按顺序列出了可能的原因。有关 Claude Desktop 标签，请参阅**[缺少 HIPAA 配置标签](https://claude.com/docs/cowork/hipaa-setup#the-hipaa-configured-label-is-missing)**。如果仍然缺少，请联系您的客户团队。

## 管理成员计算机上的数据

Claude Code（本地模式）和 Cowork（本地模式）在每个成员的计算机上存储会话数据，例如记录。保护和删除该数据是您的组织的责任。

从您的组织中删除成员不会删除其计算机上的任何内容，因此当成员离职时请擦除计算机。保持全磁盘加密处于启用状态。

**[管理本地会话数据](https://code.claude.com/docs/en/hipaa-setup#manage-local-session-data)**列出了数据的存储位置、Claude Code（本地模式）和 Claude Desktop 自动删除的内容，以及如何删除其余内容。有关 Cowork 数据，请参阅**[管理每台计算机上的 Cowork 数据](https://claude.com/docs/cowork/hipaa-setup#manage-cowork-data-on-each-computer)**。

## 监控会话

这些工具显示 Claude Code（本地模式）和 Cowork（本地模式）中发生的情况。它们在是否包含会话内容方面有所不同：

- **合规 API：**为您的组织启用**[合规 API](https://support.claude.com/en/articles/13015708)**后，Anthropic 会从两个产品中捕获会话活动和内容，以便您的组织可以通过 API 读取。会话可读取 30 天，或者如果您的组织的保留期更短，则为您的组织的保留期。虽然您的组织对 Claude Code 具有 ZDR，但 Anthropic 不会捕获 Claude Code 会话。

- **审计日志和数据导出：**审计日志和数据导出不包含会话内容。审计日志记录管理更改，例如启用 HIPAA 和应用配置。

- **OpenTelemetry：**您可以将来自两个产品的事件发送到您自己的监控工具。您导出的数据是您的组织的责任。在**组织设置 > 数据和隐私 > 监控**中设置 Cowork 的目标。您的 IT 团队在**[托管设置](https://code.claude.com/docs/en/monitoring-usage#administrator-configuration)**中为 Claude Code 设置目标。

**重要：**Cowork 的 OpenTelemetry 事件可能包含提示文本、Claude 的响应和工具输入，因此会话中的 PHI 可能会到达您的监控工具。当前版本的 Claude Desktop 仅发送元数据，除非启用了内容捕获，但某些较旧版本默认发送提示文本和工具输入等内容。没有组织设置可以关闭该内容。要求您的 IT 团队将 `otlpContentCapture` 设置为 `[]`，以便支持该策略的每个版本仅发送元数据，如**[内容捕获](https://claude.com/docs/cowork/monitoring#content-capture)**中所述。元数据仍然包括连接文件夹的路径、用户电子邮件地址和 API 错误消息，API 错误消息有时可能会重复请求的一部分。如果这些可能包含 PHI，请在您的收集器处过滤或编辑它们。

## 获取帮助

如有关于 BAA、HIPAA 实体实施指南或配置的问题，请联系您的 Anthropic 客户团队。如果您的组织没有客户团队，请联系**[我们的支持团队](https://support.claude.com/en/articles/9015913-how-to-get-support)**。请勿在支持请求中包含 PHI。