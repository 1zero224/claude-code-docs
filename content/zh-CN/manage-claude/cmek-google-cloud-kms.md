---
title: 使用 Google Cloud KMS 设置 CMEK
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms
description: 创建 Google Cloud KMS 密钥，授予 Anthropic 对该密钥的访问权限，并在 Claude Platform 或 Claude Enterprise 中使用该密钥。
---

让 Claude Code 为您设置此功能

```text
/claude-api 帮我使用 Google Cloud KMS 配置客户管理的加密密钥
```

## 开始之前

在 Claude 中：

* 客户管理的加密密钥（CMEK）已[为您的组织开启](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#availability)。
* 您已获得您的 Anthropic 组织 ID，即不带 `org_` 前缀的 UUID。您可以从 Claude Console 中的[组织](https://platform.claude.com/settings/organization)复制它，也可以从 claude.ai 上的**组织设置 > 组织与访问权限**复制它。创建密钥的人需要此 ID。
* 您是 Claude Platform 上的组织管理员，或 Claude Enterprise 上的所有者或主要所有者。
* 在 Claude Platform 上，您拥有一个[保留数据](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#data-retention)的工作区，最好是新的工作区，因为[现有数据仍受 Anthropic 的密钥保护](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)。
* 在 Claude Enterprise 上，您了解[哪些功能会关闭或发生变化](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise#turned-off-or-changed)。

在 Google Cloud 中：

* 已启用 Cloud KMS API。
* 您可以创建密钥环和密钥，并在其上设置 IAM 策略（`roles/cloudkms.admin` 或同等角色）。
* 已开启 Cloud KMS [数据访问审计日志](https://docs.cloud.google.com/logging/docs/audit/configure-data-access)（Data Read 和 Data Write），以便 Cloud Logging 记录 Anthropic 对您的密钥的使用。这些日志默认处于关闭状态。
* 如果您的 Google Cloud 组织强制执行网域限制共享（`constraints/iam.allowedPolicyMemberDomains`），它会阻止向 Anthropic 的服务账号授予权限。请添加项目级例外，或允许 Anthropic 的 Cloud Identity 客户 ID。请联系 Anthropic 获取该 ID。

## 在 Google Cloud 中：创建密钥

如果**验证**或**添加**失败，请参阅 [Google Cloud KMS 的修复方法](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#google-cloud-kms)。

以下步骤会使用小写形式的 Anthropic 组织 ID（`<ORGANIZATION_UUID>`）为密钥添加标签，并向 Anthropic 的服务账号授予访问权限：

```text wrap
anthropic-cmek-client-us@gcp-anthropic-cmek-clients.iam.gserviceaccount.com
```

请仅使用此地址，切勿使用通过电子邮件或聊天发送的地址。

您可以使用任何 Cloud KMS 位置。为获得最低延迟，请使用 `us-east5` 或 `us-central1`。Anthropic 强烈建议使用 HSM 保护。

要使用您已有的密钥，请先[添加组织标签](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms#organization-label)。

<Tabs>
  <Tab title="Google Cloud Console">
    <Steps>
      <Step title="创建或选择密钥环">
        在 Google Cloud 控制台中，前往[密钥管理](https://console.cloud.google.com/security/kms)页面。打开一个密钥环，或在 `us-east5` 等位置创建一个密钥环。
      </Step>

      <Step title="配置密钥">
        点击 **Create key** 并输入名称。选择以下选项：

        * **Protection level:** HSM
        * **Key material:** Generated key
        * **Purpose:** Symmetric encrypt/decrypt
        * **Algorithm:** Google symmetric key
      </Step>

      <Step title="添加组织标签">
        点击 **Additional settings**，然后点击 **Add label**。输入组织标签：

        * **Key 1:** `anthropic-org-<ORGANIZATION_UUID>`
        * **Value 1:** `true`

        点击 **Create**。

        <Frame caption="带有组织标签的 HSM 保护的对称加密/解密密钥。">
          ![Google Cloud KMS 的 Create key 页面，设置了 HSM 保护、对称加密/解密，以及值为 true 的 anthropic-org 标签。](https://platform.claude.com/docs/images/cmek/gcp-create-key-label.png)
        </Frame>
      </Step>

      <Step title="打开密钥的权限">
        在密钥环详情页面上，选中密钥旁边的复选框。打开 **Permissions** 面板并点击 **Grant access**。
      </Step>

      <Step title="向 Anthropic 的服务账号授予对该密钥的访问权限">
        检查 **Resource** 显示的是密钥，而不是密钥环或项目。输入以下值：

        * **New principals:** `anthropic-cmek-client-us@gcp-anthropic-cmek-clients.iam.gserviceaccount.com`
        * **Role:** Cloud KMS CryptoKey Encrypter/Decrypter

        点击 **Add another role** 并选择 **Cloud KMS Viewer**。点击 **Save**。

        <Frame caption="向 Anthropic 服务账号授予这两个角色，范围限定为该密钥。">
          ![Grant access 对话框，其中 Anthropic 服务账号被分配了 Cloud KMS CryptoKey Encrypter/Decrypter 和 Viewer 角色。](https://platform.claude.com/docs/images/cmek/gcp-grant-access.png)
        </Frame>
      </Step>

      <Step title="复制完整的密钥资源名称">
        在密钥环详情页面上，打开密钥的 **Actions** 菜单并点击 **Copy resource name**。

        <Frame caption="从操作菜单中复制密钥的完整资源名称。">
          ![Google Cloud 密钥环详情，密钥操作菜单中的 Copy resource name 操作已突出显示。](https://platform.claude.com/docs/images/cmek/gcp-copy-resource-name.png)
        </Frame>
      </Step>
    </Steps>
  </Tab>

  <Tab title="gcloud CLI">
    在每个命令中，替换以下占位符：

    * **`<your-project-id>`:** 您的 Google Cloud 项目 ID
    * **`<region>`:** 密钥的位置，例如 `us-east5`
    * **`<your-keyring-name>`:** 密钥环的名称
    * **`<your-key-name>`:** 密钥的名称
    * **`<ORGANIZATION_UUID>`:** 小写形式的 Anthropic 组织 ID

    <Steps>
      <Step title="创建或选择密钥环">
        如果您已有可重复使用的密钥环，请跳过此步骤。

        ```bash
        gcloud kms keyrings create <your-keyring-name> \
          --project=<your-project-id> \
          --location=<region>
        ```
      </Step>

      <Step title="创建密钥">
        ```bash
        gcloud kms keys create <your-key-name> \
          --project=<your-project-id> \
          --location=<region> \
          --keyring=<your-keyring-name> \
          --purpose=encryption \
          --protection-level=hsm \
          --labels=anthropic-org-<ORGANIZATION_UUID>=true
        ```

        如需软件保护，请省略 `--protection-level=hsm`。
      </Step>

      <Step title="向 Anthropic 的服务账号授予对该密钥的访问权限">
        运行以下两个命令：

        ```bash
        gcloud kms keys add-iam-policy-binding <your-key-name> \
          --project=<your-project-id> \
          --location=<region> \
          --keyring=<your-keyring-name> \
          --member="serviceAccount:anthropic-cmek-client-us@gcp-anthropic-cmek-clients.iam.gserviceaccount.com" \
          --role=roles/cloudkms.cryptoKeyEncrypterDecrypter
        ```

        ```bash
        gcloud kms keys add-iam-policy-binding <your-key-name> \
          --project=<your-project-id> \
          --location=<region> \
          --keyring=<your-keyring-name> \
          --member="serviceAccount:anthropic-cmek-client-us@gcp-anthropic-cmek-clients.iam.gserviceaccount.com" \
          --role=roles/cloudkms.viewer
        ```
      </Step>

      <Step title="获取完整的密钥资源名称">
        ```bash
        gcloud kms keys describe <your-key-name> \
          --project=<your-project-id> \
          --location=<region> \
          --keyring=<your-keyring-name> \
          --format="value(name)"
        ```
      </Step>
    </Steps>
  </Tab>
</Tabs>

在 Claude 中添加密钥的人员需要该密钥的完整资源名称，格式如下：

```text wrap
projects/<your-project-id>/locations/<region>/keyRings/<your-keyring-name>/cryptoKeys/<your-key-name>
```

### 为您已有的密钥添加组织标签

除非密钥带有值为 `true` 的标签 `anthropic-org-<ORGANIZATION_UUID>`（其中使用小写形式的 Anthropic 组织 ID），否则验证会失败。要添加该标签，请运行：

```bash
gcloud kms keys update <your-key-name> \
  --project=<your-project-id> \
  --location=<region> \
  --keyring=<your-keyring-name> \
  --update-labels=anthropic-org-<ORGANIZATION_UUID>=true
```

其他值可在密钥的完整资源名称中找到，并且该命令会保留现有标签。然后跳至[授予访问权限的步骤](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms#encryption-key-setup)。

## 在 Claude 中：添加并附加密钥

要在 Claude Platform 上通过脚本完成这部分操作，请改为[使用 Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-admin-api)。

<Tabs>
  <Tab title="Claude Platform">
    <Steps>
      <Step title="打开加密密钥设置">
        在 Claude Console 中，打开数据保留页面上的[加密密钥](https://platform.claude.com/settings/privacy#encryption-keys)。如果该部分不存在，请从顶部的下拉菜单中选择**组织**。
      </Step>

      <Step title="添加密钥">
        在**加密密钥**下，点击**添加密钥**。输入**显示名称**，选择 **Google Cloud KMS**，然后点击**继续**。
      </Step>

      <Step title="输入密钥资源名称">
        检查您的密钥是否带有对话框中显示的**组织标签**。将密钥的完整资源名称粘贴到**密钥资源名称**中，然后点击**添加**。
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
        点击**添加密钥**。输入**显示名称**并选择 **Google Cloud KMS**。选中复选框以确认您已阅读 CMEK 文档，然后点击**继续**。
      </Step>

      <Step title="输入密钥资源名称">
        检查您的密钥是否带有对话框中显示的**组织标签**。将密钥的完整资源名称粘贴到**密钥资源名称**中，然后点击**添加**。
      </Step>

      <Step title="确认已添加密钥">
        检查最多可能需要 30 秒。当出现**已配置加密密钥。** 时，点击**完成**。如果改为出现**无法添加加密密钥。**，请[查找原因并重试](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#claude-ai-add-fails)。

        您的密钥保护从那时起写入的数据。请告知您的用户[哪些功能发生了变化](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise#turned-off-or-changed)。
      </Step>
    </Steps>
  </Tab>
</Tabs>
