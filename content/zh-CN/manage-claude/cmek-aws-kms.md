---
title: 使用 AWS KMS 设置 CMEK
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms
description: 创建 AWS KMS 密钥，授予 Anthropic 对该密钥的访问权限，并在 Claude Platform 或 Claude Enterprise 中使用它。
---

让 Claude Code 为您设置此功能

```text
/claude-api 帮我使用 AWS KMS 配置客户管理的加密密钥
```

如果您的 Claude Console 侧边栏显示**账户管理方 AWS**，请改为按照[在 Claude Platform on AWS 上设置 CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws) 进行操作。

## 开始之前

在 Claude 中：

* 客户管理的加密密钥（CMEK）已[为您的组织开启](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#availability)。
* 您已获得您的 Anthropic 组织 ID，即不带 `org_` 前缀的 UUID。您可以从 Claude Console 中的[组织](https://platform.claude.com/settings/organization)复制它，也可以从 claude.ai 上的**组织设置 > 组织与访问权限**复制它。创建密钥的人需要此 ID。
* 您是 Claude Platform 上的组织管理员，或 Claude Enterprise 上的所有者或主要所有者。
* 在 Claude Platform 上，您拥有一个[保留数据](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#data-retention)的工作区，最好是新的工作区，因为[现有数据仍受 Anthropic 的密钥保护](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)。
* 在 Claude Enterprise 上，您了解[哪些功能会关闭或发生变化](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise#turned-off-or-changed)。

在 AWS 中：

* 您可以创建 AWS KMS 密钥并设置其密钥策略（`kms:CreateKey` 和 `kms:PutKeyPolicy`）。
* 对 AWS KMS 访问的任何 IP 限制都允许 [Anthropic 的出站 IP 地址](https://platform.claude.com/docs/zh-CN/api/ip-addresses#outbound-ip-addresses)。

## 在 AWS 中：创建密钥

如果**验证**或**添加**失败，请参阅 [AWS KMS 的修复方法](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#aws-kms)。

### 编写密钥策略

此策略授予 Anthropic 的 IAM 角色对您的密钥的访问权限。请仅使用本页发布的角色 ARN，切勿使用通过电子邮件或聊天发送的 ARN。将策略复制到文本编辑器中：

```json key-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AccountRootAdmin",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<AWS_ACCOUNT_ID>:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
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
        }
      }
    },
    {
      "Sid": "AllowAnthropicCMEKDescribe",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::915198916910:role/anthropic-cmek-client-us"
      },
      "Action": "kms:DescribeKey",
      "Resource": "*"
    }
  ]
}
```

将 `<AWS_ACCOUNT_ID>` 替换为您的 [AWS 账户 ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/console-account-id.html)，将 `<ORGANIZATION_UUID>` 替换为您的 [Anthropic 组织 ID](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms#prerequisites)。请保留 `kms:EncryptionContext:anthropic:org_uuid` 条件，否则验证会失败。

您还可以通过列出工作区的分区 ID 来[将密钥限制为特定工作区](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms-workspaces)。如果您要替换已保护 Claude 数据的密钥上的策略，请[再添加一条策略语句](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#existing-data)。

### 创建密钥

您可以在任何 AWS 商业区域中创建密钥。为获得最低延迟，请使用 `us-east-2`。

要使用您在 AWS KMS 中已有的密钥，请[替换其密钥策略](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms-workspaces#replace-the-key-policy)。

<Tabs>
  <Tab title="AWS Console">
    <Steps>
      <Step title="打开 Create key 向导">
        打开 [AWS KMS 控制台](https://console.aws.amazon.com/kms)，并切换到该密钥所在的区域。转到 **Customer managed keys**，然后点击 **Create key**。
      </Step>

      <Step title="Configure key">
        确保已选择以下选项：

        * **Key type:** Symmetric
        * **Key usage:** Encrypt and decrypt
        * **Advanced options > Key material origin:** KMS
        * **Advanced options > Regionality:** Single-Region key

        点击 **Next**。

        <Frame caption="对称、加密和解密、单区域密钥。">
          ![AWS KMS 的 Configure key 步骤，已选择 Symmetric 密钥类型、Encrypt and decrypt 密钥用法和 Single-Region key。](https://platform.claude.com/docs/images/cmek/aws-configure-key.png)
        </Frame>
      </Step>

      <Step title="Add labels">
        为该密钥输入一个 **Alias**，例如 `anthropic-cmek`。点击 **Next**。

        <Frame caption="该密钥的别名。">
          ![AWS KMS 的 Add labels 步骤，别名为 anthropic-cmek，描述为 Anthropic CMEK。](https://platform.claude.com/docs/images/cmek/aws-add-labels.png)
        </Frame>
      </Step>

      <Step title="Define key administrative permissions">
        不要选择任何密钥管理员。您的密钥策略已授予您的 AWS 账户完全的管理控制权。点击 **Next**。

        <Frame caption="将密钥管理员留空。">
          ![AWS KMS 的 Define key administrative permissions 步骤，列出了可以管理该密钥的 IAM 角色。](https://platform.claude.com/docs/images/cmek/aws-admin-permissions.png)
        </Frame>
      </Step>

      <Step title="Define key usage permissions">
        不要选择任何密钥用户，也不要在 **Other AWS accounts** 下添加任何内容。您的密钥策略已授予所需的访问权限。点击 **Next**。
      </Step>

      <Step title="Edit key policy">
        点击 **Edit**。用您的密钥策略替换整个策略，然后点击 **Next**。
      </Step>

      <Step title="Review">
        点击 **Finish**。
      </Step>

      <Step title="复制您的密钥的 ARN">
        在 **Customer managed keys** 中，打开新密钥并复制其 ARN。在 Claude 中添加该密钥的人员需要用到它。
      </Step>
    </Steps>
  </Tab>

  <Tab title="AWS CLI">
    将您的密钥策略保存为 `key-policy.json`，然后创建密钥。将 `<REGION>` 替换为该密钥所在的区域。

    ```bash
    aws kms create-key \
      --region <REGION> \
      --description "Anthropic CMEK" \
      --key-usage ENCRYPT_DECRYPT \
      --policy file://key-policy.json
    ```

    从输出中复制 `KeyMetadata.Arn`。在 Claude 中添加该密钥的人员需要用到此 ARN。
  </Tab>
</Tabs>

在 Claude 中每次检查密钥都会向 CloudTrail 添加四个预期的拒绝访问错误。您可以[将它们过滤掉](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#cloudtrail)。

## 在 Claude 中：添加并附加密钥

要在 Claude Platform 上通过脚本完成这部分操作，请改为[使用 Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-admin-api)。

<Tabs>
  <Tab title="Claude Platform">
    <Steps>
      <Step title="打开加密密钥设置">
        在 Claude Console 中，打开数据保留页面上的[加密密钥](https://platform.claude.com/settings/privacy#encryption-keys)。如果该部分不存在，请从顶部的下拉菜单中选择**组织**。
      </Step>

      <Step title="添加密钥">
        在**加密密钥**下，点击**添加密钥**。输入**显示名称**，选择 **AWS KMS**，然后点击**继续**。
      </Step>

      <Step title="输入密钥 ARN">
        检查您的[密钥策略](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms#key-policy)是否包含对话框中显示的**密钥策略的组织 ID**。将密钥 ARN 粘贴到 **KMS 密钥 ARN** 中，然后点击**添加**。
      </Step>

      <Step title="验证密钥">
        在**加密密钥**列表中，点击密钥旁边的**验证**。检查通过时会显示**已连接**。如果显示的是**连接失败**，请参阅 [CMEK 故障排除](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting)。
      </Step>

      <Step title="将密钥附加到工作区">
        <Warning>
          附加密钥[无法撤消](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#what-each-action-does)。您无法将其从工作区中移除，也无法替换它。如果密钥被删除，Anthropic 将无法恢复其保护的数据。
        </Warning>

        1. 在数据保留页面顶部，从下拉菜单中选择工作区。
        2. 在**加密密钥**下，选择密钥并点击**保存**。
        3. 选中复选框以确认您已阅读 CMEK 文档，然后点击**确认**。

        您会看到**工作区设置已更新。** 一旦您的密钥[生效](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)，您的云服务提供商的审计日志中就会显示 Anthropic 对您的密钥的调用。

        在 [ZDR 组织](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#zdr-organizations)中，选择工作区后，请选择**标准数据保留**，以便显示**加密密钥**。点击**保存**后，在数据保留条款上点击**接受**。
      </Step>
    </Steps>
  </Tab>

  <Tab title="Claude Enterprise">
    在 Claude Enterprise 上，CMEK 适用于整个组织，并且您只能添加一个密钥。

    <Warning>
      添加密钥[无法撤消](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#what-each-action-does)。您无法从组织中移除该密钥，也无法替换它。如果该密钥被删除，Anthropic 将无法恢复其所保护的数据。
    </Warning>

    添加密钥会为每位用户[关闭或更改某些功能](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise#turned-off-or-changed)。如需先试用 CMEK，请使用单独的组织。

    <Steps>
      <Step title="打开加密密钥设置">
        前往 [claude.ai > 组织设置 > 数据与隐私](https://claude.ai/admin-settings/data-privacy-controls)，找到**加密密钥**。
      </Step>

      <Step title="添加密钥">
        点击**添加密钥**。输入**显示名称**并选择 **AWS KMS**。选中复选框以确认您已阅读 CMEK 文档，然后点击**继续**。
      </Step>

      <Step title="输入密钥 ARN">
        检查您的[密钥策略](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms#key-policy)是否包含对话框中显示的**密钥策略的组织 ID**。将密钥 ARN 粘贴到 **KMS 密钥 ARN** 中，然后点击**添加**。
      </Step>

      <Step title="确认已添加密钥">
        检查最多可能需要 30 秒。当出现**已配置加密密钥。** 时，点击**完成**。如果改为出现**无法添加加密密钥。**，请[查找原因并重试](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#claude-ai-add-fails)。

        您的密钥保护从那时起写入的数据。请告知您的用户[哪些功能发生了变化](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise#turned-off-or-changed)。
      </Step>
    </Steps>
  </Tab>
</Tabs>
