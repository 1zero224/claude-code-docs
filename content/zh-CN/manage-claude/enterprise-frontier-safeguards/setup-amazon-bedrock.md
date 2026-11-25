---
title: Amazon Bedrock 上的 Enterprise Frontier Safeguards
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-amazon-bedrock
description: 查找 AWS 提供的 Amazon Bedrock 上 Enterprise Frontier Safeguards 设置指南。
---

要在 Amazon Bedrock 上使用 [Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview)（EFS），您需要通过 AWS 获取访问权限并进行设置，而不是在 Claude Console 中进行。Bedrock 上的 EFS 目前处于 beta 阶段。

<Warning>
  只有在您连接自己的 Amazon S3 存储桶之后，EFS 才会生效，即使 Bedrock 已将 EFS 显示为 `enabled`。在此之前，Bedrock 会将您的提示和响应的副本存储在由 AWS 管理的存储中，而这种设置并不属于 EFS。
</Warning>

要设置 EFS，请遵循 [AWS 的设置指南](https://docs.aws.amazon.com/bedrock/latest/userguide/anthropic-efs-enterprise-data-controls.html)。存储、密钥和 Safety Cases 也请参考该指南，因为本指南中的其他页面描述的是 Claude Console。

在 beta 期间，Bedrock 上的某些控制措施和数据保留可能与本指南所述有所不同。请向 AWS 咨询详细信息。
