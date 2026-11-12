---
title: CMEK、数据保留和 ZDR
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention
description: 为什么带有密钥的工作区会保留数据，这对 ZDR 意味着什么，以及 Anthropic 可以在您的密钥之外保存哪些内容。
---

在 Claude Platform 上，客户管理的加密密钥（CMEK）和[零数据保留（ZDR）](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)不能共用一个工作区。在 Claude Enterprise 上，组织可以同时使用 CMEK 和 ZDR，但之后[无法启用 artifacts](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise#turned-off-or-changed)。

## 密钥需要一个保留数据的工作区

CMEK 对存储的数据进行加密，而在 Claude Platform 上的 ZDR 下，Anthropic 不会存储您的提示或响应。因此，您只能将密钥附加到采用**标准数据保留**（30 天）的工作区。

附加密钥后，您无法为该工作区关闭数据保留。要恢复 ZDR，请创建一个新工作区并将您的流量迁移到该工作区。

## 在 ZDR 组织中

Claude Platform 上的 ZDR 组织可以使用 CMEK。您将一个工作区切换到**标准数据保留**，接受数据保留条款，并将您的密钥附加到该工作区。您切换的工作区不能是默认工作区，因为默认工作区的保留设置遵循组织的设置。您的其他工作区保持 ZDR。

[选择您的设置页面](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#set-it-up)，并按照该页面的附加步骤操作。

## 您的密钥之外的有限保存

在三种有限的情况下，Anthropic 可能会在 Anthropic 管理的加密下保存特定记录：

* Anthropic 依法须保留记录的情况（例如，根据 18 U.S.C. § 2258A 向 NCMEC 报告的材料）
* 严重伤害的紧迫风险（例如，CBRNE 武器开发、攻击性网络攻击或迫在眉睫的暴力威胁）
* 违反 Anthropic [商业服务条款](https://www.anthropic.com/legal/commercial-terms)第 D.4 节或客户与 Anthropic 之间其他适用协议中同等条款的情况

除 [CSAM 筛查](https://support.claude.com/en/articles/9020328-csam-detection-and-reporting)外，保存需要人工审核员的明确决定，并遵循 Anthropic 的[商业数据保留政策](https://privacy.claude.com/en/articles/10023548-how-long-do-you-store-my-data)。对于每一次保存，都会生成一个相应的 [Compliance API 活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)事件，其中包含说明保存目的的原因代码。有关详细信息，请参阅 [CMEK 内容保存](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#cmek-content-preservation)。安全筛查元数据在 Anthropic 管理的加密下保留，并在密钥撤销后仍可读取。这些是源自 Anthropic 自动安全扫描的记录，例如模式标识符和匹配指示符，而非对话内容。
