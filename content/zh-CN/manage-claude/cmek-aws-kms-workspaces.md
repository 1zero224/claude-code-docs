---
title: 将 AWS KMS 密钥限制为仅供特定工作区使用
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms-workspaces
description: 添加一个密钥策略条件，列出允许使用您的 AWS KMS 密钥的工作区。
---

[设置中的密钥策略](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms#key-policy)允许您组织中的每个工作区使用您的密钥。添加此可选条件后，未列出的工作区将无法使用该密钥。

在将[密钥附加](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms#register-the-key-with-anthropic)到某个工作区之前，请先将该工作区的[分区 ID](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#what-one-key-protects) 添加到策略中。对于新工作区，这意味着在创建时不附加密钥。

## 复制工作区的分区 ID

在 Claude Console 中，打开[数据保留](https://platform.claude.com/settings/privacy)页面，然后从顶部的下拉菜单中选择工作区。复制**加密密钥**下的**区间 ID**。[获取工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/retrieve)端点会将其作为 `compartment_id` 返回。

Claude Enterprise 组织只有一个分区 ID。请在 [claude.ai > 组织设置 > 数据与隐私](https://claude.ai/admin-settings/data-privacy-controls)的**加密密钥**下，从您的密钥中复制该 ID。

## 将条件添加到密钥策略

用以下策略语句替换 `AllowAnthropicCMEKCrypto` 策略语句。填入您的 Anthropic 组织 ID，以及使用该密钥的每个工作区的分区 ID。已在保护数据的密钥[还需要一条策略语句](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#existing-data)。

```json
{
  "Sid": "AllowAnthropicCMEKCrypto",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::915198916910:role/anthropic-cmek-client-us"
  },
  "Action": ["kms:Encrypt", "kms:Decrypt"],
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:anthropic:org_uuid": ["<ORGANIZATION_UUID>"]
    },
    "StringEqualsIfExists": {
      "kms:EncryptionContext:anthropic:compartment_uuid": ["<COMPARTMENT_UUID>"]
    }
  }
}
```

## 替换现有密钥上的密钥策略

1. 在 [AWS KMS 控制台](https://console.aws.amazon.com/kms)中，打开该密钥并点击 **Key policy** 选项卡。
2. 如果您看到 **Switch to policy view**，请点击它。
3. 点击 **Edit**。
4. 用您的密钥策略替换其中的 JSON。
5. 点击 **Save changes**。
