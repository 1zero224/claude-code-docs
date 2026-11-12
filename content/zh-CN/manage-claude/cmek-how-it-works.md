---
title: CMEK 的工作原理
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works
description: 您的密钥如何保护存储的数据、一个密钥保护哪些内容、何时生效，以及您可以看到什么。
---

## 您的密钥封装用于加密您数据的密钥

Anthropic 使用数据密钥加密您存储的数据。借助客户管理的加密密钥（CMEK），您的密钥会对这些数据密钥进行加密，也就是封装。这就是信封加密。为了写入或读取数据，Anthropic 会请求您的密钥管理服务（KMS）封装或解封装数据密钥。

数据密钥在 Anthropic 和您的云之间传递。您的密钥永远不会离开您的 KMS。您的数据也永远不会发送到您的 KMS。

<Frame>
  ![您的密钥始终位于您云中的 KMS 内，并封装 Anthropic 的数据密钥，这些数据密钥用于加密您的数据。您的云可以记录每次调用。](https://platform.claude.com/docs/zh-CN/images/cmek/key-wrapping.svg)
</Frame>

Anthropic 不保留您密钥的任何副本，因此没有该密钥，Claude [就无法读取它所保护的数据](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle)。

## 一个密钥保护什么

在 Claude Platform 和 Claude Platform on AWS 上，一个密钥保护一个工作区，并且一个密钥可以服务于多个工作区。在 Claude Enterprise 上，一个密钥保护整个组织，而一个组织只能拥有一个密钥。[您的密钥保护哪些数据](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#what-cmek-protects)取决于您的产品。

在 Claude Platform 和 Claude Enterprise 上，您的密钥带有您的 Anthropic 组织 ID，该 ID 将密钥与您的组织绑定。根据您的 KMS，该 ID 位于密钥策略条件、标签或标记中。不带该 ID 的密钥无法通过验证。在多个组织之间共享的密钥带有每个组织的 ID。

AWS KMS 密钥策略可以通过分区 ID [指定工作区](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms-workspaces)。分区 ID 是 Anthropic 为每个工作区分配的用于加密的 UUID，而不是以 `wrkspc_` 开头的工作区 ID。

## 您的密钥所在位置

您的密钥位于 AWS KMS、Google Cloud KMS 或 Azure Key Vault 中。您的提供商可能会为此收费。

在 Claude Platform 和 Claude Enterprise 上，Anthropic 在美国区域处理所有加密操作。您的密钥可以位于美国境外，但会带来一定的延迟代价。[每个设置页面](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#set-it-up)都列出了延迟最低的区域。

## 您的密钥何时生效

您的密钥仅保护在其生效后写入的数据。现有数据（之前的聊天、文件和会话）仍使用 Anthropic 的密钥加密，且不会被重新加密。

在 Claude Platform 和 Claude Platform on AWS 上，如果密钥在工作区的首个请求之前附加到该工作区，则会随该请求生效。在正在使用的工作区中，您的密钥最多可能需要一天才能生效，在此期间写入的数据仍使用 Anthropic 的密钥加密。这就是 Anthropic 建议使用新工作区的原因。

## 您可以看到什么

您云中的审计日志可以记录 Anthropic 使用您的密钥执行的每项操作。如果您需要开启该日志记录，您的设置页面会将其列为前提条件。在 AWS CloudTrail 中，某些条目是[预期错误](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#cloudtrail)。

对您的 CMEK 配置所做的更改会显示在 [Compliance API 活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)中。密钥操作则不会显示。

## 延迟和标准

每次封装或解封装都是一次与您的 KMS 之间的往返，这可能会给读取和写入存储数据增加少许延迟。有关 FIPS 等加密标准，请参阅[信任中心](https://trust.anthropic.com/resources)中的 CMEK 白皮书。

接下来，请阅读[每项密钥操作的作用](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#what-each-action-does)。
