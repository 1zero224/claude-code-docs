---
title: 客户管理的加密密钥
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek
description: 使用您在自己的云中持有的密钥来加密 Claude 存储的数据。
---

向 Claude Code 询问此功能

```text
/claude-api 向我介绍客户管理的加密密钥
```

借助 “customer-managed encryption keys”（客户管理的加密密钥），即 CMEK，您可以在自己的 AWS KMS、Google Cloud KMS 或 Azure Key Vault 中创建密钥。Anthropic 使用该密钥对某些静态数据进行加密。您的密钥始终由您控制，因此您可以轮换它、审计其使用情况，以及移除 Anthropic 对您密钥的访问权限。

## 选择您的设置页面

选择存放您密钥的服务。Claude Platform on AWS 同样使用 AWS KMS，它有单独的页面。

<CardGroup cols={2}>
  <Card title="AWS KMS" href="https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms">
    适用于 Claude Platform 和 Claude Enterprise。
  </Card>

  <Card title="Google Cloud KMS" href="https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms">
    适用于 Claude Platform 和 Claude Enterprise。
  </Card>

  <Card title="Azure Key Vault" href="https://platform.claude.com/docs/zh-CN/manage-claude/cmek-azure-key-vault">
    适用于 Claude Platform 和 Claude Enterprise。
  </Card>

  <Card title="Claude Platform on AWS" href="https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws">
    适用于 Claude Console 侧边栏显示**账户管理方 AWS** 的组织。
  </Card>
</CardGroup>

在 Claude Platform 上，您还可以[使用 Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-admin-api) 添加并附加您的密钥。

## 在设置之前了解 CMEK

您使用的产品决定了您的密钥加密哪些数据、哪些数据仍由 Anthropic 的密钥保护，以及哪些功能会关闭或发生变化。

<CardGroup cols={2}>
  <Card title="CMEK 的工作原理" href="https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works">
    您的密钥如何保护数据、何时生效，以及您可以看到哪些信息。
  </Card>

  <Card title="CMEK、数据保留和 ZDR" href="https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention">
    将密钥与 “zero data retention”（零数据保留），即 ZDR 一起使用，以及 Anthropic 可以在您的密钥之外保存哪些数据。
  </Card>

  <Card title="CMEK 在 Claude Platform 上涵盖的内容" href="https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform">
    查找某项功能或某类数据。
  </Card>

  <Card title="CMEK 在 Claude Enterprise 上涵盖的内容" href="https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise">
    查找某项功能，并了解您的用户会注意到哪些变化。
  </Card>
</CardGroup>

## 您需要承诺的事项

* 一旦附加密钥，您就[无法移除或替换它](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#what-each-action-does)。
* 您的密钥仅保护[在其生效之后](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)写入的数据。
* 如果您的密钥被删除，Anthropic 将无法恢复该密钥保护的数据。[在您的密钥处于禁用状态期间](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#restore-access)，Claude 无法读取这些数据。
* 在 Claude Platform 上，附加了密钥的工作区[必须保留数据](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#data-retention)。

## 谁可以使用 CMEK

CMEK 是可选功能。要为符合条件的组织启用它，请联系您的 Anthropic 客户团队。您需要具有以下角色之一才能进行设置：

* Claude Platform：组织管理员
* Claude Platform on AWS：管理员
* Claude Enterprise：所有者或主要所有者

## 设置之后

* [轮换、禁用或删除您的加密密钥时会发生什么](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle)
* [CMEK 故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting)
