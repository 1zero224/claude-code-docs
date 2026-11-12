---
title: 在 Claude Platform on AWS 上设置 CMEK
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws
description: 在您的 AWS 账户中创建 AWS KMS 密钥，并将其附加到 Claude Platform on AWS 上的工作区。
---

如果您的 Claude Console 侧边栏未显示**账户管理方 AWS**，请改为按照[使用 AWS KMS 设置 CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms) 进行操作。

## 开始之前

* “Customer-managed encryption keys”（客户管理的加密密钥），即 CMEK，已[为您的组织开启](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#availability)。

* 在托管您组织的 AWS 账户中，您可以创建 AWS KMS 密钥并设置密钥策略（`kms:CreateKey` 和 `kms:PutKeyPolicy`）。

* 您在 [Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#using-the-claude-console) 中拥有**管理员**角色。

* 您用于登录的 IAM 主体具有以下权限：

  * 登录：`aws-external-anthropic:AssumeConsole`
  * 添加密钥：`aws-external-anthropic:RegisterKey`
  * 查看密钥：`aws-external-anthropic:ListKeys` 和 `aws-external-anthropic:GetKey`
  * 附加密钥：`aws-external-anthropic:UpdateWorkspace` 或 `aws-external-anthropic:CreateWorkspace`，以及对该密钥的 `kms:DescribeKey`、`kms:Encrypt` 和 `kms:Decrypt`
  * 从列表中选择密钥（可选）：`kms:ListKeys` 和 `kms:DescribeKey`

  如需限定这些权限的范围，请参阅 [IAM 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#encryption-keys)。

* 您有一个未附加任何密钥的工作区，最好是新工作区，因为[现有数据仍由 Anthropic 的密钥加密](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)。密钥策略中会列出该工作区的分区 ID。

在 Claude Console 中，打开[数据保留](https://platform.claude.com/settings/privacy)页面，然后从顶部的下拉菜单中选择工作区。复制**加密密钥**下的**区间 ID**。[获取工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/retrieve)端点会将其作为 `compartment_id` 返回。

## 在 AWS 中：创建密钥

### 编写密钥策略

此策略向 AWS 服务主体 `aws-external-anthropic.amazonaws.com` 授予对您的密钥的访问权限。请仅使用本页面上公布的名称，切勿使用通过电子邮件或聊天发送的名称。将该策略复制到文本编辑器中：

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
      "Sid": "AllowClaudePlatformOnAWSCrypto",
      "Effect": "Allow",
      "Principal": {
        "Service": "aws-external-anthropic.amazonaws.com"
      },
      "Action": ["kms:Encrypt", "kms:Decrypt", "kms:GenerateDataKey"],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:EncryptionContext:anthropic:compartment_uuid": ["<COMPARTMENT_UUID>"]
        }
      }
    },
    {
      "Sid": "AllowClaudePlatformOnAWSDescribe",
      "Effect": "Allow",
      "Principal": {
        "Service": "aws-external-anthropic.amazonaws.com"
      },
      "Action": "kms:DescribeKey",
      "Resource": "*"
    }
  ]
}
```

将 `<AWS_ACCOUNT_ID>` 替换为您的 [AWS 账户 ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/console-account-id.html)，并将 `<COMPARTMENT_UUID>` 替换为[分区 ID](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws#prerequisites)。

可选的 `EncryptionContext` 条件将该密钥限制为仅用于您列出的工作区。在将该密钥附加到另一个工作区之前，请添加该工作区的分区 ID，并[替换该密钥上的策略](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms-workspaces#replace-the-key-policy)。如果不需要该条件，请从 `AllowClaudePlatformOnAWSCrypto` 语句中删除 `Condition` 块。如果您之后再添加该条件，请列出该密钥已附加到的每个工作区。

如需进一步缩小访问范围，请使用密钥策略，而不是基于 IP 的限制。向两条服务主体语句都添加 `aws:SourceArn` 条件。以下条件允许您 AWS 账户中的每个工作区：

```text wrap
"ArnLike": {"aws:SourceArn": "arn:aws:aws-external-anthropic:*:<AWS_ACCOUNT_ID>:workspace/*"}
```

对于特定工作区，请改为列出它们完整的[工作区 ARN](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#service-details)，格式如下：

```text wrap
arn:aws:aws-external-anthropic:<REGION>:<AWS_ACCOUNT_ID>:workspace/<WORKSPACE_ID>
```

### 创建密钥

在托管您组织的 AWS 账户中、与工作区相同的区域内创建密钥。

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

## 在 Claude 中：添加并附加密钥

通过 AWS 登录 Claude Console。如果想改用脚本完成这一部分，请使用 [IAM 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#encryption-keys)中列出的端点。

<Steps>
  <Step title="打开加密密钥设置">
    在 Claude Console 中，打开数据保留页面上的[加密密钥](https://platform.claude.com/settings/privacy#encryption-keys)。如果该部分不存在，请从顶部的下拉菜单中选择**组织**。
  </Step>

  <Step title="添加密钥">
    在**加密密钥**下，点击**添加密钥**并输入**显示名称**。
  </Step>

  <Step title="输入密钥 ARN">
    在 **KMS 密钥 ARN** 中，选择您的密钥。如果未列出该密钥，请点击**手动输入 ARN** 并粘贴 ARN。Anthropic 不接受别名 ARN 和多区域密钥。点击**添加**。
  </Step>

  <Step title="将密钥附加到工作区">
    <Warning>
      附加密钥[无法撤消](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#what-each-action-does)。您无法将其从工作区中移除，也无法替换它。如果密钥被删除，Anthropic 将无法恢复其保护的数据。
    </Warning>

    1. 在数据保留页面顶部，从下拉菜单中选择工作区。
    2. 在**加密密钥**下，选择该密钥并点击**保存**。
    3. 勾选复选框以确认您已阅读 CMEK 文档，然后点击**确认**。

    您会看到**工作区设置已更新。** 密钥[生效](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)后，CloudTrail 会记录对您的密钥的调用。

    附加时会通过一次加密调用和一次解密调用来检查密钥，因此密钥策略的问题会在此处显现。如果您收到 KMS 访问错误，请参阅[原因和解决方法](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#claude-platform-on-aws)。
  </Step>
</Steps>
