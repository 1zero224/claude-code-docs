---
title: Enterprise Frontier Safeguards
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview
description: 面向使用 Claude 最强大模型的符合条件组织的零数据保留（ZDR），并将密封的安全数据保存在您自己的云账户中。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

Enterprise Frontier Safeguards（EFS）是面向使用 [Claude 最强大模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)（例如 Claude Fable 5.1）的符合条件组织的[零数据保留（Zero Data Retention，ZDR）](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)。Anthropic 不会保留您的提示和响应。密封的安全数据会存入您自己拥有的存储中。EFS 要求使用您自己的存储桶。使用 [Anthropic 管理的存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#glossary)的工作区不属于 EFS，因为 Anthropic 会存储该工作区提示和响应的加密副本。

如果您希望使用 Claude 最强大模型，但需要将密封的安全数据保存在您自己的云账户中，请使用 EFS。使用 EFS 时，由您组织中的人员（而非 Anthropic 的审查人员）审查 [Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。EFS 同时包括：

* [客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-owned-storage)：接收您的密封的安全数据（提示、响应和模型推理，以 Anthropic 的格式加密）的存储桶。即使在您自己的存储桶中，您的团队也无法读取密封的安全数据。
* [由您的团队审查 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)：Anthropic Human Review 始终处于关闭状态，因此由您自己的团队审查自动化安全监控发现的已标记案例。

<Frame>
  ![EFS、Anthropic managed 与 Legacy ZDR 的比较。在 Anthropic managed 上，Enterprise Data Controls 涵盖审查和密钥。](https://platform.claude.com/docs/zh-CN/images/efs-how-it-works-enterprise-data-controls.svg)
</Frame>

Anthropic 不对 EFS 收费，EFS 也不会改变模型行为、API 定价或速率限制。您的云服务提供商会照常就该存储向您收费，包括读取、写入和数据出站传输。

EFS 涵盖在为 EFS 配置的工作区中对 Messages API 的请求，无论使用哪个模型。它不涵盖 Claude Platform 功能为其自身用途而存储的数据，即使是在 Messages API 请求中，例如代码执行容器、Files API 上传的文件、Batch API 结果以及 Claude Managed Agents 会话。请参阅常见问题中的[数据与保留](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#data-and-retention)。

## 获取访问权限

要获取 EFS，请按照以下步骤操作：

1. 如果 Claude Console 中的[计划](https://platform.claude.com/settings/programs)页面已显示 **Enterprise Frontier Safeguards**，请直接跳至第 2 步。否则，请联系您的 Anthropic 客户团队申请访问权限。如果您没有客户团队，请填写 [Enterprise Frontier Safeguards 申请表](https://claude.com/form/enterprise-frontier-safeguards)。
2. Anthropic 确认您的组织符合资格后，由组织管理员在 Claude Console 的[计划](https://platform.claude.com/settings/programs)页面接受相关条款。
3. 要设置 EFS，请按照[适用于您平台的 EFS 设置指南](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console)进行操作。

在 Amazon Bedrock 上，您通过 AWS 获取访问权限。请参阅 [Amazon Bedrock 上的 Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-amazon-bedrock)。

## 在 Claude Platform 上，EFS 适用于整个工作区

EFS 对某个工作区生效后，即适用于该工作区中对所有模型的每一个请求，包括不要求数据保留的模型。

如果只想让一部分流量使用 EFS，请使用不同的工作区。您发送的 [API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#api-keys-and-resource-scoping)或工作区 ID 决定了请求使用哪张卡片。新工作区不会改变您现有的各个工作区及其 API 密钥。

## Enterprise Data Controls

在 Claude Platform 上，接受 [EFS 和 Enterprise Data Controls 条款](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console#get-access)后，您将同时获得两者。借助 Enterprise Data Controls，您可以在 Anthropic 管理的存储上[使用客户管理密钥](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys)或[关闭 Anthropic Human Review](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#turn-off-anthropic-human-review)。启用其中任意一项后，您的组织将开启 [Access Transparency](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/human-review-and-access-transparency#check-access-transparency)。

## 另请参阅

<CardGroup cols={2}>
  <Card title="设置客户自有存储" icon="database" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage">
    将您的密封的安全数据保存在您自己云账户中的存储中。
  </Card>

  <Card title="审查 Safety Case" icon="file" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases">
    接收、调查并回复 Anthropic 发送的 Safety Case。
  </Card>

  <Card title="使用客户管理密钥" icon="lock" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/customer-managed-keys">
    使用您保存在自己的密钥管理服务中的密钥加密您的数据，适用于 EFS 或 Anthropic 管理的存储。
  </Card>
</CardGroup>

## 后续步骤

<CardGroup cols={2}>
  <Card title="设置指南" icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-console">
    获取访问权限，并在您使用的平台上设置 EFS。
  </Card>

  <Card title="Enterprise Frontier Safeguards 常见问题" icon="book" href="https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq">
    关于 EFS 的常见问题解答。
  </Card>
</CardGroup>
