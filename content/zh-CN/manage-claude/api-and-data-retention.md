---
title: API 与数据保留
url: https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention
description: 了解 Anthropic 的 API 及相关功能如何保留数据，包括零数据保留（ZDR）和 HIPAA 就绪 API 访问的相关信息。
---

本页涵盖 Claude API（`api.anthropic.com`）、Claude Platform on AWS、[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 以及 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)，在这些平台上，Anthropic 是数据处理方。在 Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock 上，云提供商是数据处理方；有关其等效控制措施，请参阅这些平台的数据保留和合规文档。

Anthropic 为 Claude API 提供两种数据处理安排：["zero data retention"（零数据保留），即 ZDR](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#zero-data-retention-zdr-scope)，以及 [HIPAA 就绪](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#hipaa-readiness)。[功能资格表](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)列出了每种安排所涵盖的 API 功能。有关这些安排之外的 Anthropic 标准保留政策，请参阅[商业数据保留政策](https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data)和[消费者数据保留政策](https://privacy.claude.com/en/articles/10023548-how-long-do-you-store-my-data)。

## Anthropic 如何处理数据保留

不同的 API 和功能有不同的存储需求。如果某项功能不需要存储客户的提示或响应，它可能符合 ZDR 资格。如果某项功能必然需要存储，Anthropic 会在以下承诺下将保留范围设计得尽可能小：

* 未经您的明确许可，保留的数据绝不会用于模型训练。
* 只保留功能正常运行在技术上所必需的内容。默认情况下不保留对话内容（您的提示和 Claude 的输出）；例外情况是[受管控模型（Covered Models）](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)，这些模型要求 30 天保留。
* 保留的数据会在尽可能短的 “time to live”（生存时间），即 TTL 内被清除，并且 Anthropic 致力于让客户控制数据的保留时长。保留的内容以及适用特定 TTL 时的保留时长，均记录在各功能的页面上。

有几种保留模式不属于本页所述的 ZDR 和 HIPAA 安排。可通过 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 访问的数据遵循其自身的保留模式。[活动源（Activity Feed）](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)会保留数据 6 年。来自 claude.ai 的聊天、文件和项目内容遵循您的组织在 [claude.ai > 组织设置 > 数据与隐私](https://claude.ai/admin-settings/data-privacy-controls)中设置的保留政策，除非用户更早将其删除。[本地会话记录](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-local-sessions)来自用户计算机上的会话，例如 Cowork 和 Claude Code 等应用中的会话。这些记录默认存储 6 年；如果您的组织设置了有限的自定义对话保留期（同一个 claude.ai 设置），则按该保留期存储。[远程会话记录](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retrieve-remote-sessions)（云端的 Cowork，以及 Claude Tag）没有默认保留期。请参阅[保留与删除](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-sessions#retention-and-deletion)。Compliance API 不会捕获已启用 ZDR 的本地会话。对于已启用 HIPAA 就绪的组织，它仅捕获 Cowork 和 Claude Code 本地会话，并将其记录存储 30 天。

## 零数据保留（ZDR）

在 ZDR 安排下，API 响应返回后，Anthropic 不会以静态方式存储客户的提示或响应。如需为您的组织申请 ZDR，请联系 [Anthropic 销售团队](https://claude.com/contact-sales)。ZDR 按组织启用；每个新组织都需要由您的客户团队单独启用 ZDR，且启用不会自动扩展到同一账户下的其他组织。

具有 ZDR 安排的组织无法直接从浏览器调用 Claude API：API 会拒绝其跨源（CORS）请求。要从 Web 应用程序调用 API，请从您自己的后端发送请求。请参阅[基于浏览器的应用程序](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#browser-based-applications)。

### ZDR 涵盖的范围

* **Claude Messages 和令牌计数 API：** ZDR 适用于这些端点上[功能资格表](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)中列出的符合资格的功能。使用 `/v1/messages` 但在表中标记为"否"的功能（例如代码执行）不在涵盖范围内。
* **使用 API 密钥的 Claude Code：** 当 Claude Code 与来自商业组织（受 Anthropic 商业服务条款约束的组织，区别于消费者 Claude 账户）的 API 密钥一起使用时，ZDR 适用。如果在 Claude Code 中启用了指标日志记录，则使用统计等生产力数据不受 ZDR 约束，可能会被保留。Claude Enterprise 上的 Claude Code 不在此安排涵盖范围内，它有单独的 ZDR 方案；请参阅 [ZDR 不涵盖的范围](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#what-zdr-does-not-cover)。
* **Claude Platform on AWS：** [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 遵循与第一方 Claude API 相同的数据保留政策。ZDR 可按需提供；请联系您的 Anthropic 客户代表以启用。
* **Claude Models on Google Cloud (Anthropic managed)：** [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 遵循与第一方 Claude API 相同的数据保留政策。ZDR 可按需提供；请联系您的 Anthropic 客户代表以启用。

### ZDR 不涵盖的范围

* **Claude Console：** 在 Claude Console 中的任何使用，包括 playground。
* **Claude Managed Agents：** Claude Managed Agents 是有状态资源；会话记录会一直保留，直到您将其删除。
* **Claude 消费者产品：** Claude Free、Pro、Max 和 Ultra 计划，包括这些计划的客户使用 Claude 的网页版、桌面版或移动应用或 Claude Code 的情况。
* **Claude Teams 和 Claude Enterprise 产品界面：** 这些界面不符合 ZDR 资格。唯一的例外是 Claude Enterprise 上的 Claude Code，它有自己的 ZDR 方案，与本页所述的 ZDR 安排相互独立。该方案仅适用于 Claude Enterprise 上符合条件的账户，并需要 Anthropic 单独启用。请参阅 [Claude Code ZDR 文档](https://code.claude.com/docs/en/zero-data-retention)，该文档仅适用于 Claude Enterprise 组织。
* **Claude for Excel：** 目前不符合 ZDR 资格。
* **Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5 和 Claude Mythos 5：** 这些模型要求 30 天数据保留，除非获得 Anthropic 的明确授权，否则无法在 ZDR 下使用。请参阅[特定模型的数据保留要求](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)。
* **第三方集成：** 由第三方网站、工具或其他集成处理的数据不在涵盖范围内，但其中一些可能提供类似方案。请查看每项服务的数据处理做法。
* **被标记的内容和法律保全：** 请参阅[无论何种安排都适用的保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#retention-regardless-of-arrangement)。

<Note>
  有关哪些产品和功能符合 ZDR 资格的最新信息，请参阅您的合同条款或联系您的 Anthropic 客户代表。
</Note>

## HIPAA 就绪

Claude API 为处理 “protected health information”（受保护的健康信息），即 PHI 的组织提供 HIPAA 就绪的集成。签署 BAA 并拥有已启用 HIPAA 的组织后，您可以使用受支持的 API 功能来处理 PHI，同时支持您组织的 HIPAA 合规。符合条件的组织可以直接在 Claude Console 中审阅并签署 BAA 以及启用 HIPAA 就绪。与 ZDR 相比，HIPAA 就绪采用一套更广泛的隐私和安全保护措施（在 PHI 整个生命周期内保护 PHI 的加密、访问控制和审计日志记录），而不是要求立即删除。如果您的组织处理 PHI，应使用 HIPAA 就绪安排；您无需同时使用 ZDR。有关每种安排涵盖的功能，请参阅[功能资格表](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)。

<Note>
  本页介绍 Claude API 的 HIPAA 就绪。有关涵盖 Claude Enterprise 和配置要求的完整 HIPAA 实施指南，请参阅 [Anthropic 信任中心](https://trust.anthropic.com/resources)。
</Note>

### HIPAA 就绪涵盖的范围

* **Claude API：** HIPAA 就绪适用于 Claude API（`api.anthropic.com`）上[功能资格表](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)中列出的符合资格的功能。
* **Microsoft Foundry 中的 Claude：** 对于与 Anthropic 签署了 BAA 并在 Azure 订阅上启用了 HIPAA 配置的客户，[Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)（托管于 Anthropic 或托管于 Azure）是符合资格的服务。请通过您的 Anthropic 客户团队申请 HIPAA 配置。有关要求和涵盖的功能，请参阅 [HIPAA 实施指南](https://trust.anthropic.com/resources)。

### HIPAA 就绪不涵盖的范围

* **Claude 消费者产品：** Claude Free、Pro、Max 和 Ultra 计划。
* **Claude Console：** 通过 Claude Console 界面的使用（支持从 Console 设置中启用 HIPAA 就绪；通过 Console 处理 PHI 不在涵盖范围内）。
* **合作伙伴运营的平台：** Amazon Bedrock 和 Google Cloud’s Agent Platform。请参阅这些平台的合规文档。
* **Claude Platform on AWS：** Claude Platform on AWS 不提供 HIPAA 就绪。
* **第三方集成：** 由连接到您应用的外部工具或服务处理的数据。
* **Claude Code：** Claude Code 不在 HIPAA 就绪涵盖范围内。
* **Beta 功能：** 处于 beta 阶段的功能通常不在 BAA 涵盖范围内，除非在[功能资格表](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)中明确列为符合资格。
* **被标记的内容和法律保全：** 请参阅[无论何种安排都适用的保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#retention-regardless-of-arrangement)。

### PHI 处理指南

受保护的健康信息（PHI）包括任何可识别个人身份的健康信息。在 Claude API 的上下文中，PHI 通常出现在消息内容（提示和 Claude 的响应）、附加文件（图像、PDF）以及与消息内容关联的文件名或元数据中。根据 BAA，以下字段预期不应包含 PHI：工作区名称、用户信息（姓名、电子邮件、电话号码）、账单数据和支持工单。

当使用[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)或带有 `strict: true` 的工具时，API 会将 JSON schema 编译为语法，这些语法与消息内容分开缓存。这些缓存的 schema 不会获得与提示和响应相同的 PHI 保护。**请勿在 JSON schema 定义中包含 PHI。** 此限制适用于 schema 属性名称、`enum` 值、`const` 值以及 `pattern` 正则表达式。患者特定信息应仅出现在消息内容中，在那里它受到 HIPAA 保护措施的保护。

### HIPAA 错误处理

您签署的 BAA 是确定哪些功能在涵盖范围内的官方依据。API 也会自动强制执行这些限制。当已启用 HIPAA 的组织发送包含不符合资格功能的请求时，API 会返回 `400` 错误，以防止意外使用不在您 BAA 涵盖范围内的功能：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "The requested features are not available for HIPAA-regulated organizations without Zero Data Retention: code_execution."
  }
}
```

错误消息会列出在请求中检测到的不符合资格的功能；请将其移除后重试。"without Zero Data Retention"这一措辞是 API 自身的表述，不会改变解决方法。对于在[功能资格表](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)的"详细信息"列中注明不会被阻止的客户端工具，API 会接受相关请求，但这些工具仍不在 HIPAA 就绪范围内。

### HIPAA 就绪入门

设置 HIPAA 就绪的 API 访问有两种方式。大多数组织可以使用 Anthropic 的标准 BAA 直接在 Claude Console 中启用；需要协商 BAA 的组织应与其客户团队合作。

#### 在 Console 中启用（标准 BAA）

<Steps>
  <Step title="打开您组织的数据保留设置">
    在 Claude Console 中，前往[数据保留](https://platform.claude.com/settings/privacy)页面（`platform.claude.com/settings/privacy`）。拥有 HIPAA 管理权限的组织管理员会在该页面看到 **HIPAA 合规**卡片。如果您没有看到该卡片，请从页面顶部的下拉菜单中选择**组织**。如果您的组织符合条件，但您仍然看不到启用选项，请让组织管理员完成这些步骤。
  </Step>

  <Step title="审阅并签署 BAA">
    下载业务伙伴协议（Business Associate Agreement）和 HIPAA 实施指南，然后以您组织的授权法定代表人身份接受该协议。每个步骤都会在您下载前一份文档后变为可用，并且您的启用将与您下载的确切 BAA 版本绑定。
  </Step>

  <Step title="启用立即生效">
    您接受后，HIPAA 就绪控制措施会立即应用于您的组织。一旦为您的组织启用了 HIPAA 就绪，该配置将是永久性的，管理员无法将其禁用。API 会自动强制执行功能限制，对使用不符合资格功能的请求返回错误。有关该错误和客户端工具例外情况，请参阅 [HIPAA 错误处理](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#hipaa-error-handling)。
  </Step>
</Steps>

#### 联系销售（自定义 BAA）

如果您的组织需要协商或自定义的 BAA，或者您的组织无法使用自助启用，请联系 [Anthropic 销售团队](https://claude.com/contact-sales)。Anthropic 会签署 BAA 并为您的组织启用 HIPAA 就绪。

#### 使用符合资格的功能进行构建

无论您使用哪种方式，都请在[功能资格表](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)中确认哪些功能受支持，并查看 [PHI 处理指南](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#phi-handling-guidelines)，了解哪些功能限制了 PHI 可出现的位置。有关详细的配置和合规要求，请参阅 [HIPAA 实施指南](https://trust.anthropic.com/resources)。

<Warning>
  HIPAA 就绪在组织级别强制执行。如果您同时需要 HIPAA 就绪的 API 访问和通用 API 访问，请为每种用途使用单独的组织。
</Warning>

## 特定模型的数据保留要求

Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5 和 Claude Mythos 5 被指定为受管控模型（Covered Models）。请参阅[受管控模型支持文章](https://support.claude.com/en/articles/15425695)。这些模型要求 30 天数据保留。因此，除非获得 Anthropic 的明确授权，否则这些模型均无法使用 ZDR。在 Claude API 上，如果组织的数据保留配置不满足此要求，则其向 Claude Fable 5 发出的请求会返回 `400 invalid_request_error`：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "In order to access this model, your organization or workspace must have data retention enabled."
  }
}
```

30 天数据保留要求适用于提供受管控模型的所有平台。在 Claude API（包括 Claude Platform on AWS）上，由 Anthropic 处理保留的数据。在 Amazon Bedrock 和 Google Cloud’s Agent Platform 上，保留的数据会留在您的云提供商环境中；有关启用步骤，请查看各平台的文档。

### 为工作区启用 30 天保留

具有 ZDR 安排的组织可以仅为特定工作区启用 30 天保留，从而在该工作区中使用这些模型。组织中的其他工作区将继续保持零数据保留。默认工作区（Default Workspace）始终遵循组织的设置，因此请使用其他工作区。

<Steps>
  <Step title="打开工作区的数据保留设置">
    在 Claude Console 中，前往[数据保留](https://platform.claude.com/settings/privacy)页面（`platform.claude.com/settings/privacy`），然后从页面顶部的下拉菜单中选择该工作区。
  </Step>

  <Step title="开启 30 天数据保留">
    选择**标准数据保留**（30 天保留），然后点击**保存**。随后会出现一个确认对话框，要求您审阅并接受适用于该工作区数据保留的条款。只有在您接受这些条款后，该工作区才会开启数据保留。
  </Step>

  <Step title="验证">
    现在，从该工作区向受管控模型发出的请求将会成功。没有覆盖设置的工作区将继续遵循组织的默认设置。
  </Step>
</Steps>

## 功能资格

下表列出了哪些 Claude API 功能符合 ZDR 和 HIPAA 就绪安排的资格。这些表格按照[功能概览](https://platform.claude.com/docs/zh-CN/build-with-claude/overview)的方式对功能进行划分。

每个资格列使用三种值：

* **是：** 该功能在该安排下完全符合资格。对于 ZDR，"是"还假定您使用的模型不要求 30 天数据保留；无论功能资格如何，[受管控模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)都无法在 ZDR 下使用。
* **是（有条件）：** 您的提示和 Claude 的输出不会被存储，但会短暂保留一个有限的技术工件（在"详细信息"列中注明），以使该功能正常运行。有关约束这些功能的承诺，请参阅 [Anthropic 如何处理数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#how-anthropic-approaches-data-retention)。
* **否：** 该功能不符合资格。在 HIPAA 就绪下，API 会阻止包含"否"功能的请求并返回 `400` 错误，除非该功能的"详细信息"列另有说明。在 ZDR 下，API **不会**阻止这些功能；使用这类功能即表示您选择让该特定数据脱离 ZDR 安排，并适用该功能自身记录的保留政策。在 ZDR 中标记为"否"的功能通常是有状态的（它们会存储作业、文件或容器状态），这就是它们无法实现零保留的原因。

### 平台功能

本节列出与 [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages) 和 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 都相关的功能。其中一些功能仅在二者之一中可用。本表给出了每项功能在 Messages API 中的资格。在 Claude Managed Agents 中，没有任何功能符合 ZDR 或 HIPAA 就绪资格；请参阅 [Managed Agents 功能](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#managed-agents-features)下的 Claude Managed Agents 行。

| 功能                                                                                                     | 端点                                       | 具备 ZDR 资格 | 具备 HIPAA 资格 | 详细信息                                                                                                                                                                                                                                                                                                                                                    |
| :----------------------------------------------------------------------------------------------------- | :--------------------------------------- | :-------- | :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)            | `/v1/messages` 使用 `advisor` 工具           | 是         | 否           | 顾问模型的输出在 API 响应中返回；响应后服务器端不存储任何内容。                                                                                                                                                                                                                                                                                                                      |
| [Agent Skills](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview)          | `/v1/skills`, `/v1/messages` 使用 `skills` | 否         | 否           | 技能数据按标准政策保留。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview#data-retention)                                                                                                                                                                                                                                       |
| [批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)                       | `/v1/messages/batches`                   | 否         | 否           | 保留 29 天；需要异步存储。 [了解更多](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing#data-retention)                                                                                                                                                                                                                                        |
| [浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)           | `/v1/messages` 使用 `browser` 工具集          | 是         | 否           | 客户端工具。除标准 API 处理外，Anthropic 不运行浏览器操作，也不保留页面内容。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#data-retention) 不在 HIPAA 就绪范围内；包含浏览器使用工具的请求不会被阻止。                                                                                                                                                               |
| [引用](https://platform.claude.com/docs/zh-CN/build-with-claude/citations)                               | `/v1/messages`                           | 是         | 是           |                                                                                                                                                                                                                                                                                                                                                         |
| [计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)          | `/v1/messages` 使用 `computer` 工具集或工具      | 是         | 是           | 客户端工具，截图和文件在您的环境中捕获和存储，而非由 Anthropic 存储。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#data-retention)                                                                                                                                                                                                      |
| [数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)                            | `/v1/messages` 使用 `inference_geo`        | 是         | 是           |                                                                                                                                                                                                                                                                                                                                                         |
| [Effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)                              | `/v1/messages` 使用 `effort`               | 是         | 是           |                                                                                                                                                                                                                                                                                                                                                         |
| [快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)                             | `/v1/messages` 使用 `speed: "fast"`        | 是         | 是           | 使用相同的 Messages API 端点，推理速度更快。无论速度设置如何，ZDR 均适用。                                                                                                                                                                                                                                                                                                          |
| [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files)                            | `/v1/files`                              | 否         | 否           | 文件会一直保留，直到被显式删除或达到其配置的过期时间。 [了解更多](https://platform.claude.com/docs/zh-CN/build-with-claude/files#data-retention)                                                                                                                                                                                                                                       |
| [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)                       | `/v1/messages` 使用 `mcp_servers`          | 否         | 否           | 数据按标准政策保留。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#data-retention)                                                                                                                                                                                                                                                 |
| [MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)                 | `/v1/tunnels`                            | 否         | 否           | 研究预览。MCP 隧道安全页面介绍了数据流边界和子处理方详情。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/security)                                                                                                                                                                                                                                    |
| [对话中途的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages) | `/v1/messages` 使用 `role: "system"` 消息    | 是         | 是           | Messages API 的请求结构功能；对话中途的系统消息经由标准推理路径处理，响应后服务器端不存储任何内容。                                                                                                                                                                                                                                                                                                |
| [PDF 支持](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support)                         | `/v1/messages`                           | 是         | 是           | HIPAA 资格适用于通过 Messages API 内联发送的 PDF，不适用于通过 Files API 发送的 PDF。                                                                                                                                                                                                                                                                                          |
| [编程式工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)  | `/v1/messages` 使用 `code_execution` 工具    | 否         | 否           | 基于代码执行容器构建；数据最多保留 30 天。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling#data-retention)                                                                                                                                                                                                               |
| [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)                        | `/v1/messages`                           | 是         | 是           | 您的提示和 Claude 的输出不会被存储。KV 缓存表示和加密哈希在缓存 TTL 期间保存在内存中，并在过期后立即删除。 [了解更多](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#data-retention)                                                                                                                                                                                            |
| ↳ [缓存诊断](https://platform.claude.com/docs/zh-CN/build-with-claude/cache-diagnostics)                   | `/v1/messages` 使用 `diagnostics`          | 是（有条件）    | 否           | 您的提示和 Claude 的输出不会被存储。由加密哈希和令牌数估算值组成的指纹会被短暂保留，以便与下一个请求进行比较。 [了解更多](https://platform.claude.com/docs/zh-CN/build-with-claude/cache-diagnostics#data-retention)                                                                                                                                                                                           |
| [搜索结果](https://platform.claude.com/docs/zh-CN/build-with-claude/search-results)                        | `/v1/messages` 使用 `search_results` 源     | 是         | 是           |                                                                                                                                                                                                                                                                                                                                                         |
| [结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)                   | `/v1/messages`                           | 是（有条件）    | 是           | 您的提示和 Claude 的输出不会被存储。仅缓存 JSON schema，自上次使用起最长保留 24 小时。这也涵盖严格工具使用（工具上的 `strict: true`），它使用相同的语法管道。 [了解更多](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#data-retention) JSON schema 定义中不得包含 PHI；请参阅 PHI 处理指南。 [了解更多](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#phi-handling-guidelines) |
| [思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)                                | `/v1/messages` 使用 `thinking`             | 是         | 是           |                                                                                                                                                                                                                                                                                                                                                         |
| ↳ [自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost)         | `/v1/messages`                           | 是         | 是           |                                                                                                                                                                                                                                                                                                                                                         |
| [阈值压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold)                  | `/v1/messages` 使用 `context_management`   | 是         | 否           | 服务器端压缩结果通过 API 响应以无状态方式返回和往返传递。                                                                                                                                                                                                                                                                                                                         |
| [令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)                        | `/v1/messages/count_tokens`              | 是         | 是           | 在发送请求之前计算令牌数。                                                                                                                                                                                                                                                                                                                                           |
| [工具搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)            | `/v1/messages` 使用 `tool_search` 工具       | 是         | 否           | 由 Anthropic 运行的服务器端工具；每次调用时在内存中搜索请求中的工具定义，响应后不存储任何内容。                                                                                                                                                                                                                                                                                                   |
| [Web fetch 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)        | `/v1/messages` 使用 `web_fetch` 工具         | 是         | 否           | 获取的网页内容在 API 响应中返回。Web fetch 的动态过滤不具备 ZDR 或 HIPAA 资格。网站发布者可能会根据其自身政策保留请求数据（例如获取的 URL 和请求元数据）。                                                                                                                                                                                                                                                           |
| [网页搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)             | `/v1/messages` 使用 `web_search` 工具        | 是         | 是           | 实时网页搜索结果在 API 响应中返回。网页搜索的动态过滤不具备 ZDR 或 HIPAA 资格。                                                                                                                                                                                                                                                                                                        |

### Messages API 功能

| 功能                                                                                                        | 端点                                     | 具备 ZDR 资格 | 具备 HIPAA 资格 | 详细信息                                                                                                                                                                                          |
| :-------------------------------------------------------------------------------------------------------- | :------------------------------------- | :-------- | :---------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)                     | `/v1/messages` 使用 `bash` 工具            | 是         | 是           | 在您的环境中运行的客户端工具。                                                                                                                                                                               |
| [代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)            | `/v1/messages` 使用 `code_execution` 工具  | 否         | 否           | 容器数据最多保留 30 天。 [了解更多](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#data-retention)                                                                    |
| [上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)                         | `/v1/messages` 使用 `context_management` | 是         | 否           | 上下文编辑（工具使用清除和思考清除）实时应用。                                                                                                                                                                       |
| [细粒度工具流式传输](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/fine-grained-tool-streaming) | `/v1/messages`                         | 是         | 是           |                                                                                                                                                                                               |
| [记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)                      | `/v1/messages` 使用 `memory` 工具          | 是         | 是           | 客户端记忆存储，数据保留由您控制。                                                                                                                                                                             |
| [消息线程](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads)                          | `/v1/messages` 使用 `thread`             | 否         | 否           | 线程的内容（对话、系统提示和工具）保存在内存中，使用服务器管理的密钥加密，从不写入持久存储，并在线程最后一次使用后 1 小时内删除。使用此功能不会影响您使用其他具备 ZDR 资格的功能。 [了解更多](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#data-retention) |
| [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)            | `/v1/messages`                         | 是         | 是           | 用于生成 Claude 响应的标准 API 调用。                                                                                                                                                                     |
| [文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)              | `/v1/messages` 使用 `text_editor` 工具     | 是         | 是           | 在您的环境中运行的客户端工具。                                                                                                                                                                               |

### Managed Agents 功能

| 功能                                                                                      | 端点                                               | 具备 ZDR 资格 | 具备 HIPAA 资格 | 详细信息                                                          |
| :-------------------------------------------------------------------------------------- | :----------------------------------------------- | :-------- | :---------- | :------------------------------------------------------------ |
| [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) | `/v1/agents`, `/v1/sessions`, `/v1/environments` | 否         | 否           | 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。 |

### 其他功能

| 功能                                                                                     | 端点                                         | 具备 ZDR 资格 | 具备 HIPAA 资格 | 详细信息                                                                                                                                                                                                                                                                                       |
| -------------------------------------------------------------------------------------- | ------------------------------------------ | --------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows) | `/v1/messages`                             | 是         | 是           |                                                                                                                                                                                                                                                                                            |
| [OpenTelemetry 导出](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry) | `/v1/organizations/telemetry_destinations` | 是         | 否           | Beta。Span 的 ZDR 资格与生成它们的功能相同。HIPAA 组织可以导出元数据。SDK span 直接从您的环境发送到您的导出器。交付后，数据由您选择的服务持有，位于 Anthropic 之外，受该服务的保留和访问控制约束。                                                                                                                                                                      |
| [用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)           | `/v1/user_profiles`                        | 否         | 否           | 配置文件是关于您的终端客户的元数据，不包含提示或输出。您发送的配置文件数据会按照标准政策保留，直到您删除该配置文件。已启用 HIPAA 的组织可以使用用户配置文件，但用户配置文件 API 不是符合 HIPAA 资格的服务，您发送的配置文件数据中不得包含 PHI。请参阅[配置文件包含的内容以及 Anthropic 如何使用它](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#what-a-profile-contains-and-how-anthropic-uses-it)。 |

## 无论何种安排都适用的保留

即使已实施 ZDR 或 HIPAA 安排，Anthropic 仍可能在法律要求或数据被 Anthropic 的自动化信任与安全系统标记的情况下保留数据。因此，如果某个聊天或会话被标记，Anthropic 可能会将输入和输出保留最多 2 年。

## 常见问题

<AccordionGroup>
  <Accordion title="如何知道我的组织是否有 ZDR 安排？">
    请查看您的合同条款或联系您的 Anthropic 客户代表，以确认您的组织是否已实施 ZDR 安排。
  </Accordion>

  <Accordion title="我可以在 ZDR 安排下使用符合 ZDR 资格（有条件）的功能吗？">
    可以。这些功能会保留一组最少且有记录的技术数据，而不是您的提示或 Claude 的输出。有关"是（有条件）"的含义，请参阅[功能资格表](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)的说明；有关约束这些功能的承诺，请参阅 [Anthropic 如何处理数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#how-anthropic-approaches-data-retention)。
  </Accordion>

  <Accordion title="如果我在 ZDR 下使用标记为&#x22;否&#x22;的功能会怎样？">
    请求不会被阻止。在 ZDR 中标记为"否"的功能本质上是有状态的：Batch API 会存储您的作业，Files API 会存储您的文件，代码执行则在持久化容器中运行。这些功能的数据会按照该功能记录的政策进行保留。使用这些功能即表示您选择让该特定数据脱离 ZDR 安排。
  </Accordion>

  <Accordion title="我可以请求删除不符合 ZDR 资格的功能中的数据吗？">
    请联系您的 Anthropic 客户代表，讨论非 ZDR 功能的删除选项。
  </Accordion>

  <Accordion title="HIPAA 就绪与 ZDR 有何不同？">
    ZDR 可防止在 API 响应返回后以静态方式存储客户数据。HIPAA 就绪则涉及一套更广泛的隐私和安全保护措施，在 PHI 的整个生命周期内对其进行保护，包括加密、访问控制和审计日志记录。在 HIPAA 就绪下，数据可以在这些保护措施到位的情况下被保留，而不是要求立即删除。这两种安排涵盖的功能集不同；请参阅[功能资格表](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)。
  </Accordion>

  <Accordion title="如果我已有 HIPAA 就绪，还需要 ZDR 吗？">
    不需要。HIPAA 就绪的 API 访问旨在为处理 PHI 的组织提供 ZDR 的替代方案。启用 HIPAA 就绪后，您可以访问受支持的 API 功能，同时保持 HIPAA 所要求的隐私和安全保护。
  </Accordion>

  <Accordion title="如果我在 HIPAA 下使用不符合资格的功能会怎样？">
    API 会返回类型为 `invalid_request_error` 的 `400` 错误，但在[功能资格表](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)的"详细信息"列中注明不会被阻止的客户端工具除外（这些工具会被接受，但仍不在 HIPAA 就绪范围内）。错误消息会指出哪些功能不可用。请从请求中移除这些功能后重试。请参阅 [HIPAA 错误处理](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#hipaa-error-handling)。
  </Accordion>

  <Accordion title="我可以将同一个组织同时用于 HIPAA 和非 HIPAA 工作负载吗？">
    不可以。HIPAA 就绪在组织级别强制执行，并会自动阻止不符合资格的功能（表格"详细信息"列中注明的客户端工具是例外：它们不会被阻止，但仍不在 HIPAA 就绪范围内）。请为不需要 HIPAA 就绪的工作负载使用单独的组织。
  </Accordion>

  <Accordion title="如何申请 HIPAA 就绪的 API 访问？">
    符合条件的组织可以直接在 Claude Console 的[数据保留](https://platform.claude.com/settings/privacy)页面（`platform.claude.com/settings/privacy`）上，通过审阅并签署 Anthropic 的标准 BAA 来启用 HIPAA 就绪；请参阅 [HIPAA 就绪入门](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#getting-started-with-hipaa-readiness)。如果您的组织需要协商的 BAA，或者您的组织无法使用自助启用，请联系 [Anthropic 销售团队](https://claude.com/contact-sales)。
  </Accordion>

  <Accordion title="这是否适用于 Amazon Bedrock 或 Google Cloud？">
    [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 遵循与第一方 Claude API 相同的数据保留政策，且 Anthropic 是数据处理方。该平台可按需提供 ZDR。在 Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock 上，云提供商是数据处理方；有关其等效控制措施，请参阅这些平台的数据保留和合规政策。
  </Accordion>

  <Accordion title="Claude Platform on AWS 是否符合 ZDR 或 HIPAA 就绪资格？">
    Claude Platform on AWS 遵循与第一方 Claude API 相同的数据保留政策。ZDR 可按需提供；请联系您的 Anthropic 客户代表以启用。Claude Platform on AWS 不提供 HIPAA 就绪。有关详细信息，请参阅 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)。
  </Accordion>

  <Accordion title="Claude Code 是否符合 ZDR 资格？">
    Claude Code 在两种独立的方案下符合 ZDR 资格：

    * **API 密钥：** 当商业组织拥有 Claude API 的 ZDR 安排（即本页所述的安排）时，与该组织的按需付费 API 密钥一起使用的 Claude Code 在涵盖范围内。
    * **Claude Enterprise：** 当 Anthropic 为组织启用了单独的方案——Claude Enterprise 上 Claude Code 的 ZDR——时，通过 Claude Enterprise 使用的 Claude Code 在该方案涵盖范围内。

    ZDR 按组织启用。每个新组织都需要由您的客户团队单独启用 ZDR。ZDR 不会自动应用于在同一账户下创建的新组织。

    此外，如果您在 Claude Code 中启用了指标日志记录，则生产力数据（例如使用统计）不受 ZDR 约束，可能会被保留。

    有关 Claude Enterprise 上 Claude Code 的 ZDR 的完整详细信息，包括被禁用的功能以及如何申请启用，请参阅 [Claude Code ZDR 文档](https://code.claude.com/docs/en/zero-data-retention)。该文档仅适用于 Claude Enterprise 组织，并不描述 Claude API 的 ZDR 安排。
  </Accordion>

  <Accordion title="Claude for Excel 是否支持 ZDR？">
    不支持，Claude for Excel 目前不符合 ZDR 资格。
  </Accordion>

  <Accordion title="如何申请 ZDR？">
    如需申请 ZDR 安排，请联系 [Anthropic 销售团队](https://claude.com/contact-sales)。
  </Accordion>
</AccordionGroup>

## 相关资源

* [隐私政策](https://www.anthropic.com/legal/privacy)
* [结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)
* [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)
* [批量处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)
* [Files API 参考](https://platform.claude.com/docs/zh-CN/api/files/upload)
* [信任中心](https://trust.anthropic.com/resources)
