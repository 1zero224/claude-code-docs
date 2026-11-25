---
title: Google Cloud's Agent Platform 上的 Enterprise Frontier Safeguards
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/setup-google-cloud-agent-platform
description: 在 Google Cloud's Agent Platform 上获取 Enterprise Frontier Safeguards 的访问权限。
---

借助 [Enterprise Frontier Safeguards](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/overview)（EFS），您的 Google Cloud 项目中的 Cloud Storage 存储桶会保存 EFS 所涵盖的提示和响应的加密副本。EFS 目前处于 beta 阶段。有关支持的模型和注册步骤，请参阅 [Google 的 EFS 文档](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/claude/enterprise-frontier-safeguards)。

Access Transparency 不涵盖您的 Agent Platform 请求。

要设置 EFS，请联系您的 Anthropic 客户团队以获取访问权限和设置指南。如果您项目中的任何 Claude 模型开启了数据保留或数据共享，请先告知他们。EFS 仅涵盖设置指南中列出的端点和模型，并且不涵盖批量预测。

另外，也请向您的客户团队咨询存储、密钥、对话记录和 Safety Cases 相关事宜，因为其他 EFS 页面描述的是 Claude Console。如果您没有客户团队，请填写 [Enterprise Frontier Safeguards 申请表](https://claude.com/form/enterprise-frontier-safeguards)。
