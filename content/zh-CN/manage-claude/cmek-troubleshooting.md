---
title: CMEK 故障排除
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting
description: 查找 CMEK 错误消息、验证失败或意外日志条目的原因和修复方法。
---

Claude Console 中的**验证**、claude.ai 上的**添加**以及 Admin API 的 validate 调用都会运行相同的检查。当检查失败时（在 Claude Console 中显示**连接失败**），请参阅 [AWS KMS](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#aws-kms)、[Google Cloud KMS](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#google-cloud-kms) 或 [Azure Key Vault](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#azure-key-vault) 的原因。

如果某项功能停止工作，它可能是客户管理的加密密钥（CMEK）会关闭的功能之一。请查看 [Claude Platform 列表](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform#turned-off-or-changed)或 [Claude Enterprise 列表](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise#turned-off-or-changed)。

如果这里的内容都不适用，请联系您的 Anthropic 客户团队。

## 设置后请求失败

### `cmek_key_disabled`

```text wrap
Customer-managed encryption key is disabled or revoked. Re-enable the key to restore access.
```

此 400 错误表示您的密钥管理服务拒绝了 Anthropic 的调用。请按顺序检查以下原因：

1. 密钥已被禁用。
2. Anthropic 对该密钥的访问权限已被移除。[您的设置页面](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#set-it-up)列出了 Anthropic 所需的访问权限。
3. 在 AWS KMS 上，密钥策略[列出了工作区](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms-workspaces)，但未列出此工作区。
4. 在 AWS KMS 上，某个 IP 条件阻止了 [Anthropic 的出站 IP 地址](https://platform.claude.com/docs/zh-CN/api/ip-addresses#outbound-ip-addresses)。

修复该原因。访问权限恢复时，[有两项限制适用](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#restore-access)。

### `cmek_key_network_blocked`

```text wrap
The request to your key management service was blocked by its network access controls (for example, a KMS firewall or IP allowlist). Verify that your KMS network rules allow requests from Anthropic's published IP addresses, or contact Anthropic support if your configuration looks correct.
```

这是一个 400 错误。密钥本身可能是正常的。请在您的密钥管理服务的网络规则中允许 [Anthropic 公布的 IP 地址](https://platform.claude.com/docs/zh-CN/api/ip-addresses)。

### `cmek_key_unreachable`

```text wrap
The workspace's customer-managed encryption key could not be reached. Try again shortly. If this continues, check that the key is available in your key management service, or contact Anthropic support.
```

此 503 错误表示 [Anthropic 无法访问您的密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#unreachable-key)。请重试该请求。

## 验证失败：AWS KMS

| 原因                  | 修复方法                                                                                                                                        |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| 组织 ID 错误            | `org_uuid` 条件必须列出[您的 Anthropic 组织 ID](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms#prerequisites)，即不带 `org_` 前缀的 UUID。 |
| 策略中缺少某个工作区          | `compartment_uuid` 条件必须[列出该密钥所附加到的每个工作区的分区 ID](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms-workspaces)。               |
| 通过 IAM 而非密钥策略授予访问权限 | Anthropic 的角色需要在[密钥策略本身](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms#key-policy)中获得访问权限。                              |

要打印密钥策略，请运行：

```bash
aws kms get-key-policy \
  --region <REGION> \
  --key-id <KEY_ARN> \
  --policy-name default
```

### 已在保护数据的密钥

如果您要[替换某个密钥上的策略](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms-workspaces#replace-the-key-policy)，而该密钥已在保护数据，请添加此策略语句，以便 Anthropic 仍能解密这些数据。在其条件中，列出该密钥当前或曾经附加到的每个工作区的分区 ID。

```json
{
  "Sid": "AllowAnthropicCMEKDecryptExistingData",
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::915198916910:role/anthropic-cmek-client-us"
  },
  "Action": "kms:Decrypt",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "kms:EncryptionContext:anthropic:compartment_uuid": ["<COMPARTMENT_UUID>"]
    }
  }
}
```

## 验证失败：Google Cloud KMS

| 原因           | 修复方法                                                                                                                                     |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 组织标记错误       | 密钥需要带有[组织标记](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms#organization-label)，其中包含小写形式的您的 Anthropic 组织 ID。 |
| 网域限制共享       | 此组织策略会阻止向 Anthropic 的服务账号授权。[添加例外](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms#prerequisites)。            |
| 已禁用或已销毁的密钥版本 | 密钥的主版本必须处于启用状态。                                                                                                                          |

要检查授权，请运行：

```bash
gcloud kms keys get-iam-policy <your-key-name> \
  --project=<your-project-id> \
  --location=<region> \
  --keyring=<your-keyring-name>
```

## 验证失败：Azure Key Vault

| 原因                    | 修复方法                                                                                                                                     |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 组织标签错误                | 密钥的当前版本需要带有[组织标签](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-azure-key-vault#organization-tag)。                           |
| 角色分配尚未生效              | 新的角色分配可能需要几分钟才能生效。请稍候，然后重试。                                                                                                              |
| 保管库的网络规则阻止了 Anthropic | 根据 [Anthropic 所需的保管库设置](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-azure-key-vault#check-your-vault-configuration)检查这些规则。 |

## 在 Claude Platform on AWS 上附加失败

Claude Platform on AWS 没有**验证**步骤，因此密钥的问题会在您附加它时显现。

| 原因      | 修复方法                                                                                                                                                                                                                                           |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 密钥策略不匹配 | 使用 [`get-key-policy` 命令](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#aws-kms)打印密钥策略，并将其与[设置页面上的策略](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws#prepare-the-key-policy)进行比较。 |
| 条件不匹配   | 任何 `EncryptionContext` 或 `aws:SourceArn` 条件都必须包含此工作区。                                                                                                                                                                                          |
| 密钥状态或位置 | 密钥必须处于启用状态、为单区域密钥，并且与工作区位于同一 AWS 账户和区域中。                                                                                                                                                                                                       |
| 缺少权限    | 您登录所用的主体需要对该密钥拥有 `kms:DescribeKey`、`kms:Encrypt` 和 `kms:Decrypt` 权限。                                                                                                                                                                           |

如果附加仍然失败，请在密钥所在账户的 CloudTrail 中查找被拒绝的 `kms:` 事件。该事件会显示调用主体，对于加密调用，还会显示加密上下文。修复该原因，然后再次点击**保存**。

## 附加或添加密钥被拒绝

### 工作区不保留数据

```text wrap
CMEK keys can only be attached to workspaces in this organization that have inference data retention enabled.
```

[您的组织使用零数据保留（ZDR）](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#zdr-organizations)，而密钥需要一个保留数据的工作区。请先将该工作区切换为**标准数据保留**。

### 工作区没有加密密钥设置

只有[保留数据的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#data-retention)才有此设置，因此请先为该工作区选择**标准数据保留**。ZDR 组织的默认工作区无法切换，因此请使用其他工作区。

### 没有加密密钥部分

在 Claude Console 中，从数据保留页面顶部的下拉菜单中选择**组织**。如果该部分仍然缺失，则说明您的角色不包含该部分，或者您的组织尚未开启 CMEK。CMEK 由您的 Anthropic 客户团队开启。

### 工作区已有密钥

```text wrap
This workspace already has a different external key attached.
```

工作区的密钥无法替换，因此如需使用其他密钥，请[创建新的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#change-keys)。

### 在 claude.ai 上添加密钥失败

```text wrap
The encryption key couldn’t be added.
```

对您的密钥的检查失败。请查找 [AWS KMS](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#aws-kms)、[Google Cloud KMS](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#google-cloud-kms) 或 [Azure Key Vault](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#azure-key-vault) 的原因。要重试，请再次点击**添加密钥**。对话框打开时会带有您之前填写的详细信息。选中复选框，点击**继续**，然后点击**添加**。

## CloudTrail 中的预期错误

在 Claude Platform 和 Claude Enterprise 上，每当您验证密钥、将其附加到工作区或在 claude.ai 上查看它时，Anthropic 都会检查该密钥。每次检查都会在 CloudTrail 中添加四个预期的访问被拒绝错误。

要将它们过滤掉，请同时匹配以下三个值：

* `requestParameters.encryptionContext.associatedData`：`Y21lay12YWxpZGF0aW9u`
* `userIdentity.accountId`：`915198916910`
* `resources.ARN`：`arn:aws:kms:<REGION>:<AWS_ACCOUNT_ID>:key/<KEY_ID>`

仅匹配第一个值是不够的，因为任何调用方都可以设置它。
