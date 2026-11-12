---
title: Claude Platform on AWS
url: https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws
description: 通过 AWS 访问 Claude 的完整平台功能，基础设施由 Anthropic 管理。
---

Claude Platform on AWS 为您提供完整的 Anthropic 平台体验，包括 Messages API、Agent Skills、代码执行和 beta 功能，均可通过您的 AWS 账户访问。与由 AWS 运营推理栈的 [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) 不同，Claude Platform on AWS 由 Anthropic 运营。AWS 提供身份验证层（SigV4 或 API 密钥）、基于 IAM 的访问控制，以及通过 AWS Marketplace 的计费集成。

<Note>
  SDK 支持 Claude Platform on AWS。
</Note>

## 平台集成的工作原理

Claude 模型运行在 Anthropic 管理的基础设施上。这是一项用于通过 AWS 进行计费和访问的商业集成。Anthropic 是推理输入和输出的数据处理方。AWS 根据 marketplace 模式处理计费和身份元数据。通过 Claude Platform on AWS 使用 Claude 的客户须遵守 Anthropic 的[数据使用条款](https://www.anthropic.com/legal)。

Anthropic 使用 AWS 基础设施来处理您的 API 请求、运行模型推理，并存储您的工作区内容（例如提示、输出、文件、Skills 和批处理）。对于在 2026 年 9 月 18 日 00:00 UTC 之前创建的工作区，Anthropic 可能会在 AWS 之外处理请求、存储数据和运行推理。无论哪种情况，子服务都可能在不另行通知的情况下发生变化。为每个请求设置 [`inference_geo`](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#data-residency) 参数，可将推理固定在特定地理区域。

Claude Platform on AWS 遵循与第一方 Claude API 相同的数据保留政策。"Zero Data Retention"（零数据保留），即 ZDR，可应要求提供。请联系您的 Anthropic 客户代表，为您的组织启用该功能。

## Claude Platform on AWS 与 Amazon Bedrock 对比

这两种产品都允许您通过 AWS 使用 Claude，但它们在架构、API 接口和功能可用性方面有所不同。

| 方面          | Claude Platform on AWS                                                                                                                                                                              | [Amazon Bedrock（`bedrock-mantle`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) | [Amazon Bedrock（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy) |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **谁运营技术栈**  | Anthropic                                                                                                                                                                                           | AWS                                                                                                                   | AWS                                                                                                                           |
| **API 接口**  | Claude API（`/v1/{endpoint}`）                                                                                                                                                                        | 位于 `/anthropic/v1/messages` 的 Messages API                                                                            | Bedrock Converse / InvokeModel                                                                                                |
| **功能可用性**   | 通常与 Claude API 同日提供（请参阅[功能限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#features-not-supported)）                                                              | 遵循 Amazon Bedrock 发布计划                                                                                                | 遵循 Amazon Bedrock 发布计划                                                                                                        |
| **Beta 功能** | 通过 `anthropic-beta` 标头传递（请参阅[功能限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#features-not-supported)）                                                         | Amazon Bedrock 支持的 beta 功能，通过 `anthropic-beta` 标头                                                                     | Amazon Bedrock 支持的 beta 功能，通过 InvokeModel 请求体的 `anthropic_beta` 字段                                                            |
| **身份验证**    | AWS IAM / SigV4 或 API 密钥                                                                                                                                                                            | AWS IAM / SigV4                                                                                                       | AWS IAM / SigV4 或 bearer 令牌                                                                                                   |
| **计费**      | AWS Marketplace                                                                                                                                                                                     | AWS（原生服务）                                                                                                             | AWS（原生服务）                                                                                                                     |
| **基础 URL**  | `aws-external-anthropic.{region}.api.aws`                                                                                                                                                           | `bedrock-mantle.{region}.api.aws`                                                                                     | `bedrock-runtime.{region}.amazonaws.com`                                                                                      |
| **SDK 客户端** | `AnthropicAWS` (typescript: `AnthropicAws`; csharp: `AnthropicAwsClient`; go: `anthropicaws.NewClient()`; java: `AwsBackend`; php: `Anthropic\Aws\Client`; ruby: `Anthropic::AWSClient`)，处于 beta 阶段 | `AnthropicBedrockMantle`                                                                                              | `AnthropicBedrock` / Bedrock SDK                                                                                              |
| **控制台**     | Claude Console（`platform.claude.com`，通过 AWS Console 访问）                                                                                                                                             | Bedrock Console                                                                                                       | Bedrock Console                                                                                                               |
| **速率限制和配额** | 由 Anthropic 管理                                                                                                                                                                                      | 由 AWS 管理                                                                                                              | 由 AWS 管理                                                                                                                      |
| **推理数据处理方** | Anthropic                                                                                                                                                                                           | AWS                                                                                                                   | AWS                                                                                                                           |

如果您需要由 AWS 运营的 Claude，请参阅 [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)。Claude Platform on AWS 使用的容量池与第一方 Claude API 和 Amazon Bedrock 均相互独立。您可以在多个平台上运行工作负载，并在它们之间进行故障转移。

支持使用 [AWS PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html) 将您的 VPC 连接到 Claude Platform on AWS 端点（请参阅[通过 AWS PrivateLink 连接](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#connect-through-aws-privatelink)）。

**何时选择 Bedrock：** 处于受监管行业、需要 FedRAMP High、IL4、IL5 或 HIPAA 就绪，或需要 AWS 作为唯一数据处理方的组织，应使用 [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)。Bedrock 完全运行在 AWS 控制的基础设施上，由 AWS 作为运营方。

**您使用的是哪种产品？** Claude 通过几种不同的产品提供：

* **Claude Platform on AWS（本页）：** 通过 AWS Marketplace 计费的 Claude API 平台。在 [Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#using-the-claude-console) 和 AWS Console 中管理。
* **[Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)：** AWS 原生服务。在 Amazon Bedrock 控制台中管理，并作为 AWS 服务用量计费。
* **通过 AWS Marketplace 采购的 Claude Enterprise：** 一种 [claude.ai](https://claude.ai) 套餐（Claude 聊天产品），而非 API 平台。在 claude.ai 中管理，其账户和迁移行为与本页所述内容不同。请参阅 [Claude 帮助中心](https://support.claude.com)。
* **直接 Anthropic 账户：** 由 Anthropic 计费的第一方 Claude API 和 claude.ai 套餐。在 Claude Console 和 claude.ai 中管理。

## 设置您的账户

设置 Claude Platform on AWS 分为四个阶段：在 AWS Console 服务页面上注册、完成 Anthropic 组织设置、记下您的工作区 ID，以及登录 Claude Console。

<Note>
  通过 AWS Console 注册会配置一个与您的 AWS 账户关联的新 Anthropic 组织。该组织与您公司在 Anthropic 已有的任何组织（包括通过 AWS Marketplace 采购的 Claude Enterprise 组织）相互独立。第一方 Anthropic 组织中的 API 密钥、工作区和 Claude Console 设置不会被沿用。

  如果您已有 Amazon Bedrock 私有报价，请在注册前联系您的 Anthropic 或 AWS 客户代表，以便您的折扣从第一个请求起即可生效。折扣无法追溯应用于接受私有报价之前产生的用量。请参阅[私有报价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#private-offers)。
</Note>

<Steps>
  <Step title="在 AWS Console 中注册">
    1. 打开 [AWS Console](https://console.aws.amazon.com/) 并导航到 **Claude Platform on AWS** 服务页面。
    2. 点击 **Sign up**。
    3. 在注册页面上，查看条款（Anthropic 的最终用户许可协议、AWS 隐私声明和 AWS 客户协议），然后勾选同意复选框。
    4. 点击 **Continue**。

    页面会显示 **Sign-up in progress** 横幅。请停留在该页面。注册需要几分钟时间，在此期间 AWS 会为您处理 AWS Marketplace 订阅，然后自动重定向。

    如果您的组织有来自 Anthropic 的私有报价，Console 会查找该报价并提示您在 AWS Marketplace 中接受它。详情请参阅[私有报价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#private-offers)。

    <Note>
      当您使用 Claude Platform on AWS 时，Anthropic 会在 AWS 基础设施上处理和存储您的内容（例如提示和输出）。对于在 2026 年 9 月 18 日 00:00 UTC 之前创建的工作区，Anthropic 可能会在 AWS 之外处理和存储这些内容。Anthropic 的[数据使用政策](https://www.anthropic.com/legal)说明了内容和元数据的处理和存储方式。
    </Note>
  </Step>

  <Step title="设置您的 Anthropic 组织">
    注册完成后，您将被重定向到 `platform.claude.com/partner-signup`。

    1. 输入您组织所有者的电子邮件地址，然后点击**开始使用**。
    2. 在该电子邮件收件箱中查找设置链接并打开它。如果您的浏览器显示**已使用其他账户登录**页面，请点击**退出登录并继续**。
    3. 填写组织详细信息表单（组织名称、实体类型、国家/地区、预期用途），然后点击**完成设置**。

    完成设置即会创建您的 Anthropic 组织，并接受 Anthropic 的商业服务条款和使用政策。AWS Console 服务页面现在会显示包含 **Home**、**API keys**、**Quickstart** 和 **Workspaces** 的侧边栏。
  </Step>

  <Step title="创建您的工作区并记下其 ID">
    完成设置后，AWS Console 会提示您创建工作区。有关区域绑定和 IAM 资源范围的详细信息，请参阅[工作区范围](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#workspace-scoping)；有关创建其他工作区，请参阅[管理工作区](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#managing-workspaces)。

    您可以在 AWS Console **Claude Platform on AWS** 服务页面的 **Workspaces** 下，或在 [Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#using-the-claude-console) 中找到工作区 ID。工作区 ID 的格式为 `wrkspc_` 后跟字母数字标识符。
  </Step>

  <Step title="登录 Claude Console">
    对 Claude Console 的访问通过 AWS IAM 进行联合：

    1. 代入具有 `aws-external-anthropic:AssumeConsole` 权限的 IAM 角色。请参阅[控制台访问](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#console-access)。
    2. 在 **Claude Platform on AWS** 服务页面中，点击 **Open Claude Console**。AWS Console 会签发 JWT 并将您重定向到 `platform.claude.com`。
    3. 首次登录时，系统会提示您输入电子邮件地址。请输入您的工作电子邮件。平台会即时配置您的 Claude Console 用户。

    当您通过 AWS Console 登录时，Claude Console 的范围限定为您的 Claude Platform on AWS 组织。Claude Console 侧边栏左下角会显示**账户管理方 AWS** 标识。
  </Step>
</Steps>

### 从现有 Anthropic 组织迁移

注册 Claude Platform on AWS 总是会配置一个与您的 AWS 账户关联的新 Anthropic 组织。不存在原地转换：现有组织（例如第一方 Claude API 组织）无法转变为 Claude Platform on AWS 组织。

请将从现有组织的迁移规划为切换到新组织：

* **先创建新组织。** 通过 AWS Console 注册（请参阅[设置您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#set-up-your-account)）。如果您的迁移涉及私有报价，请在接受报价之前完成注册：折扣从接受时起生效，不可追溯。请参阅[私有报价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#private-offers)。
* **重新创建访问权限和配置。** API 密钥、工作区和 Claude Console 设置不会从现有组织沿用。请在新组织中创建工作区，并将您的应用程序切换到 [Claude Platform on AWS 身份验证](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#authentication)。
* **更新您的集成。** Claude Platform on AWS 提供 Claude API（`/v1/{endpoint}`），因此请求和响应结构与第一方 Claude API 相同。变化的是基础 URL、身份验证方法，以及推理和资源请求上的 `anthropic-workspace-id` 标头；请参阅[发出请求](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#making-requests)。部分平台功能有所不同；请参阅[不支持的功能](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#features-not-supported)。
* **按您自己的计划进行切换。** 新组织独立于您的现有组织，两者可以并行处理流量。无需一次性硬切换：逐步迁移工作负载，直到您的所有流量都转移到新组织。

新组织运行后，差异主要集中在计费和身份验证方面，这两者均通过 AWS 处理：

* **计费**转移到 AWS Marketplace：用量以 Claude Consumption Units 计费，而非预付额度（请参阅[账单](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#billing)）。您在 Claude Console 中设置支出限额（请参阅[支出限额](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#spend-limits)）。在过渡期间，计费保持独立：现有组织将保留其当前的计费方式。
* **身份验证和访问**转移到 AWS：请求使用 AWS 凭证或在 AWS Console（而非 Claude Console）中生成的 API 密钥进行身份验证（请参阅[身份验证](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#authentication)）。您在 AWS IAM 中管理组织成员资格，而不是在 Claude Console 中管理（请参阅[可用页面](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#available-pages)）。Anthropic SDK 为 Claude Platform on AWS 提供了单独的客户端类（请参阅[安装 SDK](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#install-an-sdk)）。
* **日常 API 使用**的方式与第一方 Claude API 相同，[功能限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#features-not-supported)中注明的情况除外。在迁移生产流量之前，请检查您的速率限制：新组织被置于 Start 层级，提高限额需通过您的 Anthropic 客户代表办理（请参阅[速率限制和配额](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#rate-limits-and-quotas)）。

对于行为有所不同的 Claude Enterprise（claude.ai）组织，请参阅[产品对比](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#claude-platform-on-aws-vs-amazon-bedrock)。

### 账户设置故障排除

* **“Sign-up failed: Failed to enable OutboundWebIdentityFederation”：** 如果您在首次提交时看到此横幅，请再次点击 **Continue**。IAM 启用可能需要一点时间才能生效。
* **注册期间没有进度指示器：** 注册需要几分钟时间。在 AWS 配置您的账户期间，页面会显示静态的 **Sign-up in progress** 横幅，没有进度条。
* **打开设置链接后显示"已使用其他账户登录"：** 点击**退出登录并继续**。页面会使用您输入的电子邮件地址重新验证您的身份。
* **登录期间出现"未找到"消息：** 此消息可能会在重定向期间短暂出现。您可以忽略它。
* **首次 API 调用后用量页面不显示数据：** 用量数据可能需要几分钟才能显示在 Claude Console 中。
* **首次 API 调用时出现"Outbound web identity federation is disabled"：** 每个账户需启用一次联合。请参阅[启用出站 Web 身份联合](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#enable-outbound-web-identity-federation)。
* **出现 403 `permission_error`，其消息指明 `sts:GetWebIdentityToken` 或 `sts:TagGetWebIdentityToken`：** 向错误消息中指明的 IAM 主体授予这两个操作。请参阅 [IAM 策略](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#iam-policies)。

## 进行 API 调用之前

请确保您已具备：

1. 一个订阅了 Claude Platform on AWS 的有效 AWS 账户（请参阅[设置您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#set-up-your-account)）。
2. 已安装并配置 [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)。
3. 在您的 AWS 账户上**已启用出站 Web 身份联合**，这是一次性设置步骤（请参阅[启用出站 Web 身份联合](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#enable-outbound-web-identity-federation)）。
4. 您的工作区 ID（请参阅[获取您的工作区 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#obtain-your-workspace-id)）。
5. 调用 API 的 IAM 权限：对您工作区的 `aws-external-anthropic:CreateInference` 操作，以及 `sts:GetWebIdentityToken` 和 `sts:TagGetWebIdentityToken` 操作（请参阅 [IAM 策略](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#iam-policies)）。
6. 如果您使用 API 密钥进行身份验证，还需要 `aws-external-anthropic:CallWithBearerToken` 操作的 IAM 权限（请参阅 [API 密钥身份验证](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#api-key-authentication)）。

### 启用出站 Web 身份联合

Claude Platform on AWS 网关在服务器端调用 `sts:GetWebIdentityToken` 来生成转发给 Anthropic 的 JWT。此 STS 功能在每个 AWS 账户上**默认处于禁用状态**。请为每个账户启用一次：

```bash CLI
aws iam enable-outbound-web-identity-federation
```

如果响应为 `[ERROR] (FeatureEnabled) ... already enabled`，则表示您的账户已开启该设置，您可以继续下一步。验证并获取您账户的颁发者 URL：

```bash CLI
aws iam get-outbound-web-identity-federation-info
```

<Warning>
  如果不执行此步骤，每个请求都会返回 `"Outbound web identity federation is disabled for your account"`。这是最常见的设置错误。
</Warning>

### 获取您的工作区 ID

完成账户设置后，您可以在 AWS Console 中创建工作区（请参阅[设置您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#set-up-your-account)）。工作区绑定到单个 AWS 区域。您可以在 [Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#using-the-claude-console) 的**工作区**下或 AWS Console 服务页面的**Workspaces**部分找到工作区 ID。

设置 `ANTHROPIC_AWS_WORKSPACE_ID` 和 `AWS_REGION` 环境变量，以便 SDK 客户端自动读取它们：

```bash CLI
export ANTHROPIC_AWS_WORKSPACE_ID='wrkspc_01AbCdEf23GhIj'
export AWS_REGION='us-west-2'  # Your workspace's AWS region
```

请在创建客户端时传递 `aws_region` (typescript, php: `awsRegion`; go: `AWSRegion`; java: `.region()`; csharp: `AwsClientOptions.AwsRegion`)，或设置 `AWS_REGION`。支持所有 AWS 商业区域。有关客户端如何查找区域，请参阅[区域解析](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#region-resolution)。

## 身份验证

Claude Platform on AWS 支持两种身份验证方法：使用 Signature Version 4（SigV4）请求签名的 AWS IAM（主要方法）和 API 密钥身份验证。两者使用相同的基础 URL 和请求格式。

### SigV4 身份验证

SigV4 是企业原生路径，可与您现有的 AWS IAM 策略、角色和审计集成。使用 SDK 支持的方法配置 AWS 凭证。[AWS 默认凭证提供程序链](https://docs.aws.amazon.com/sdkref/latest/guide/standardized-credentials.html)包括：

* 环境变量（`AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY`、`AWS_SESSION_TOKEN`）
* 共享凭证文件（`~/.aws/credentials`）
* 共享配置文件（`~/.aws/config`），包括 SSO 和 `credential_process`
* 用于 IRSA 和 GitHub Actions 的 Web 身份（`AWS_WEB_IDENTITY_TOKEN_FILE` 和 `AWS_ROLE_ARN`）
* ECS 容器凭证
* EC2 实例元数据服务（IMDS）

验证您的凭证是否有效：

```bash CLI
aws sts get-caller-identity
```

### API 密钥身份验证

对于更简单的集成路径（本地开发和脚本），您可以使用 API 密钥代替 SigV4 进行身份验证。设置 `ANTHROPIC_AWS_API_KEY` 环境变量，或在创建客户端时传递 `api_key` (typescript, php: `apiKey`; go: `APIKey`; java: `.apiKey()`; csharp: `AwsClientOptions.ApiKey`)。

在 **AWS Console** 的 **Claude Platform on AWS → API keys** 下生成 API 密钥。点击 **Generate a key**，然后复制密钥值。使用 API 密钥的主体需要与使用 SigV4 对请求进行签名的主体相同的 IAM 权限，包括两个 STS 操作（请参阅 [IAM 策略](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#iam-policies)）。此外，还需将 `aws-external-anthropic:CallWithBearerToken` IAM 操作授予应被允许使用 API 密钥身份验证的主体。

<Note>
  Claude Platform on AWS 的 API 密钥在 AWS Console 中管理，而非在 Claude Console 中管理。在标准 Claude Console [platform.claude.com](https://platform.claude.com/)（用于第一方 API 访问）中创建的密钥不适用于 Claude Platform on AWS 端点。
</Note>

#### 短期 API 密钥

对于需要将凭证交给独立进程的工作负载（例如 LLM 网关、无服务器函数，或支持 bearer-token 身份验证但不支持 SigV4 的工具），请从您的 AWS 凭证生成短期 API 密钥，而不是在 AWS Console 中配置长期密钥。

AWS 发布了适用于 [JavaScript](https://github.com/aws/token-generator-for-aws-external-anthropic-js)、[Python](https://github.com/aws/token-generator-for-aws-external-anthropic-python) 和 [Java](https://github.com/aws/token-generator-for-aws-external-anthropic-java) 的令牌生成器库。每个库都通过标准提供程序链读取您的 AWS 凭证，并返回一个有时限的令牌，您以 `Authorization: Bearer <token>` 的形式发送该令牌。令牌有效期默认为 12 小时，上限为您请求的时长、您的 AWS 凭证到期时间和 12 小时三者中的最小值。有关安装和完整配置选项，请参阅链接的代码仓库 README。

将生成的令牌传递给 SDK，方式与传递 AWS Console 生成的 API 密钥相同：

<CodeGroup exclude="shell, csharp, go, php, ruby">
  ```python Python
  from token_generator_for_aws_external_anthropic import TokenGenerator
  from anthropic import AnthropicAWS

  token = TokenGenerator(region="us-west-2").get_token()

  client = AnthropicAWS(api_key=token, aws_region="us-west-2")
  ```

  ```typescript TypeScript
  import { getTokenProvider } from "@aws/token-generator-for-aws-external-anthropic";
  import AnthropicAws from "@anthropic-ai/aws-sdk";

  const tokenProvider = getTokenProvider({ region: "us-west-2" });
  const token = await tokenProvider();

  const client = new AnthropicAws({ apiKey: token, awsRegion: "us-west-2" });
  ```

  ```java Java
  import software.amazon.awsexternalanthropic.token.TokenGenerator;
  import software.amazon.awssdk.regions.Region;
  import com.anthropic.aws.backends.AwsBackend;
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;

  void main() {
      String token = TokenGenerator.builder()
          .region(Region.US_WEST_2)
          .build()
          .getToken();

      AnthropicClient client = AnthropicOkHttpClient.builder()
          .backend(AwsBackend.builder()
              .apiKey(token)
              .region(Region.US_WEST_2)
              .workspaceId(System.getenv("ANTHROPIC_AWS_WORKSPACE_ID"))
              .build())
          .build();
  }
  ```
</CodeGroup>

如果您可以在本地生成令牌，说明您的进程已经拥有 SigV4 凭证，此时 SigV4 身份验证通常是更简单的选择。当进行 API 调用的进程与持有 AWS 凭证的进程相互独立时，请使用短期密钥。

SDK 不会自动刷新短期密钥。令牌过期时，请生成新令牌并构造新的客户端。您用于生成令牌的 AWS 凭证所属的主体需要具备 [API 密钥身份验证](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#api-key-authentication)中所述的权限。

### 凭证优先级

平台专用客户端按以下顺序解析身份验证，首先是您在创建客户端时传入的内容：

1. `api_key` (typescript, php: `apiKey`; go: `APIKey`; java: `.apiKey()`; csharp: `AwsClientOptions.ApiKey`) → `Authorization: Bearer` 标头
2. `aws_access_key` (typescript, php: `awsAccessKey`; go: `AWSAccessKey`; java: `.awsAccessKey()`; csharp: `AwsClientOptions.AwsAccessKey`) + `aws_secret_key` (ruby: `aws_secret_access_key`; typescript, php: `awsSecretAccessKey`; go: `AWSSecretAccessKey`; java: `.awsSecretAccessKey()`; csharp: `AwsClientOptions.AwsSecretAccessKey`) → AWS SigV4
3. `aws_profile` (typescript, php: `awsProfile`; go: `AWSProfile`; java: `.awsProfile()`; csharp: `AwsClientOptions.AwsProfile`) → 使用命名配置文件的 AWS SigV4
4. `ANTHROPIC_AWS_API_KEY` 环境变量 → `Authorization: Bearer` 标头
5. 默认 AWS 凭证提供程序链 → AWS SigV4

### 区域解析

如果您未设置 `aws_region` (typescript, php: `awsRegion`; go: `AWSRegion`; java: `.region()`; csharp: `AwsClientOptions.AwsRegion`)，客户端会从环境中读取 `AWS_REGION`。Python、TypeScript、C#、Go、PHP 和 Ruby SDK 随后会尝试 `AWS_DEFAULT_REGION`。TypeScript SDK 随后会检查您的 AWS 配置文件。Java SDK 仅在您调用 `AwsBackend.fromEnv()` 时才读取环境，该方法使用[默认 AWS 区域提供程序链](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/region-selection.html)。该链还会检查您的 AWS 配置文件和 EC2 实例元数据。

客户端需要区域才能使用 SigV4 对请求进行签名并构造基础 URL。区域没有默认值，因此如果 `AnthropicAWS` (typescript: `AnthropicAws`; csharp: `AnthropicAwsClient`; go: `anthropicaws.NewClient()`; java: `AwsBackend`; php: `Anthropic\Aws\Client`; ruby: `Anthropic::AWSClient`) 需要区域但找不到，就会失败并报错。如果使用 API 密钥并在 `ANTHROPIC_AWS_BASE_URL` 环境变量中提供基础 URL，则它不需要区域。

## 安装 SDK

[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 支持 Claude Platform on AWS，并提供 `AnthropicAWS` (typescript: `AnthropicAws`; csharp: `AnthropicAwsClient`; go: `anthropicaws.NewClient()`; java: `AwsBackend`; php: `Anthropic\Aws\Client`; ruby: `Anthropic::AWSClient`)，用于处理 SigV4 签名、基于区域的基础 URL 构造以及 `anthropic-workspace-id` 标头。

<Tabs>
  <Tab title="Python">
    ```bash
    pip install -U "anthropic[aws]"
    ```

    <Tip>
      在使用 Homebrew Python 或其他外部管理的 Python 环境的 macOS 上，`pip install` 可能会因 PEP 668 `externally-managed-environment` 错误而失败。请先创建并激活虚拟环境：`python3 -m venv .venv && source .venv/bin/activate`。
    </Tip>
  </Tab>

  <Tab title="TypeScript">
    ```bash
    npm install @anthropic-ai/aws-sdk
    ```
  </Tab>

  <Tab title="C#">
    ```bash
    dotnet add package Anthropic.Aws
    ```
  </Tab>

  <Tab title="Go">
    ```bash
    go get github.com/anthropics/anthropic-sdk-go/v2/aws
    ```
  </Tab>

  <Tab title="Java">
    <Tabs>
      <Tab title="Gradle">
        ```kotlin
        implementation("com.anthropic:anthropic-java:3.8.0")
        implementation("com.anthropic:anthropic-java-aws:3.8.0")
        ```
      </Tab>

      <Tab title="Maven">
        ```xml
        <dependency>
            <groupId>com.anthropic</groupId>
            <artifactId>anthropic-java</artifactId>
            <version>3.8.0</version>
        </dependency>
        <dependency>
            <groupId>com.anthropic</groupId>
            <artifactId>anthropic-java-aws</artifactId>
            <version>3.8.0</version>
        </dependency>
        ```
      </Tab>
    </Tabs>
  </Tab>

  <Tab title="PHP">
    ```bash
    composer require anthropic-ai/sdk aws/aws-sdk-php
    ```
  </Tab>

  <Tab title="Ruby">
    ```bash
    gem install anthropic aws-sdk-core
    ```
  </Tab>
</Tabs>

<Note>
  Claude Platform on AWS 的 SDK 客户端处于 beta 阶段。
</Note>

## 可用模型

以下模型可在 Claude Platform on AWS 上使用：

| Model                                                                                                    | Model ID            |
| :------------------------------------------------------------------------------------------------------- | :------------------ |
| Claude Fable 5.2                                                                                         | `claude-fable-5-2`  |
| Claude Fable 5.1                                                                                         | `claude-fable-5-1`  |
| Claude Fable 5                                                                                           | `claude-fable-5`    |
| Claude Opus 5.5                                                                                          | `claude-opus-5-5`   |
| Claude Opus 5                                                                                            | `claude-opus-5`     |
| Claude Opus 4.8                                                                                          | `claude-opus-4-8`   |
| Claude Opus 4.7                                                                                          | `claude-opus-4-7`   |
| Claude Opus 4.6                                                                                          | `claude-opus-4-6`   |
| Claude Opus 4.5                                                                                          | `claude-opus-4-5`   |
| Claude Sonnet 5.5                                                                                        | `claude-sonnet-5-5` |
| Claude Sonnet 5                                                                                          | `claude-sonnet-5`   |
| Claude Sonnet 4.6                                                                                        | `claude-sonnet-4-6` |
| Claude Sonnet 4.5 ([deprecated](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | `claude-sonnet-4-5` |
| Claude Haiku 5.5                                                                                         | `claude-haiku-5-5`  |
| Claude Haiku 4.5                                                                                         | `claude-haiku-4-5`  |

模型 ID 与第一方 Claude API 完全相同。没有 Bedrock 风格的 ARN 或 `anthropic.` 前缀。

新模型通常与第一方 Claude API 同日在 Claude Platform on AWS 上推出。

让 Claude Code 将您的代码迁移到更新的模型

```text
/claude-api migrate 将此项目迁移到最新的 Claude Opus 模型
```

## 发出请求

Claude Platform on AWS 使用与第一方 Claude API 相同的 API 端点。区别在于基础 URL、身份验证方法，以及用于标识请求所针对的[工作区](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#workspaces)的 `anthropic-workspace-id` 标头。推理和资源请求（例如对 Messages API、Models API、Files API 和 Claude Managed Agents 端点的调用）需要该标头。对 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 的请求不需要它。

在运行这些示例之前，请完成[进行 API 调用之前](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#before-making-api-calls)中的步骤。

<CodeGroup>
  ```bash cURL
  # 在 URL 和 --aws-sigv4 中都将 us-west-2 替换为您的 AWS 区域
  # 如果您使用长期 IAM 用户凭证，请省略 x-amz-security-token 标头
  curl "https://aws-external-anthropic.us-west-2.api.aws/v1/messages" \
    --aws-sigv4 "aws:amz:us-west-2:aws-external-anthropic" \
    -u "$AWS_ACCESS_KEY_ID:$AWS_SECRET_ACCESS_KEY" \
    -H "x-amz-security-token: $AWS_SESSION_TOKEN" \
    -H "content-type: application/json" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-workspace-id: $ANTHROPIC_AWS_WORKSPACE_ID" \
    -d '{
      "model": "claude-sonnet-5-5",
      "max_tokens": 1024,
      "messages": [
        {"role": "user", "content": "Hello!"}
      ]
    }'
  ```

  ```bash CLI
  # 将 us-west-2 替换为您的 AWS 区域
  # ant 会读取 ANTHROPIC_API_KEY。请在 AWS 控制台中生成密钥
  # （请参阅 API 密钥身份验证）。
  export ANTHROPIC_API_KEY="YOUR_AWS_API_KEY"

  ant messages create \
    --base-url https://aws-external-anthropic.us-west-2.api.aws \
    --workspace-id "$ANTHROPIC_AWS_WORKSPACE_ID" \
    --model claude-sonnet-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello!"}'
  ```

  ```python Python
  from anthropic import AnthropicAWS

  client = AnthropicAWS()

  message = client.messages.create(
      model="claude-sonnet-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello!"}],
  )
  print(message)
  ```

  ```typescript TypeScript
  import AnthropicAws from "@anthropic-ai/aws-sdk";

  const client = new AnthropicAws();

  const message = await client.messages.create({
    model: "claude-sonnet-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello!" }]
  });
  console.log(message);
  ```

  ```csharp C#
  using Anthropic;
  using Anthropic.Aws;
  using Anthropic.Models.Messages;

  var client = new AnthropicAwsClient();

  var message = await client.Messages.Create(new()
  {
      Model = Model.ClaudeSonnet5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello!" }]
  });

  Console.WriteLine(message);
  ```

  ```go Go
  client, err := anthropicaws.NewClient(context.Background(), anthropicaws.ClientConfig{})
  if err != nil {
  	panic(err)
  }

  message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello!")),
  	},
  })
  if err != nil {
  	panic(err)
  }

  fmt.Println(message)
  ```

  ```java Java
  import com.anthropic.aws.backends.AwsBackend;
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.builder()
          .backend(AwsBackend.fromEnv())
          .build();

      Message message = client.messages().create(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_SONNET_5_5)
              .maxTokens(1024)
              .addUserMessage("Hello!")
              .build()
      );

      IO.println(message);
  }
  ```

  ```php PHP
  use Anthropic\Aws\Client;
  use Anthropic\Messages\Model;

  $client = new Client();

  $message = $client->messages->create(
      model: Model::CLAUDE_SONNET_5_5,
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello!']],
  );

  echo $message;
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::AWSClient.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello!" }]
  )

  puts message
  ```
</CodeGroup>

客户端从环境中读取 `AWS_REGION` 和 `ANTHROPIC_AWS_WORKSPACE_ID`。您可以在创建客户端时通过传递 `aws_region` (typescript, php: `awsRegion`; go: `AWSRegion`; java: `.region()`; csharp: `AwsClientOptions.AwsRegion`) 或 `workspace_id` (typescript, php: `workspaceId`; go: `WorkspaceID`; java: `.workspaceId()`; csharp: `AwsClientOptions.WorkspaceId`) 来覆盖其中任一项。如果客户端找不到工作区 ID，或者需要区域但找不到，就会失败并报错。

<Note>
  `x-amz-security-token` 标头（cURL）仅在使用临时凭证（例如 IAM 角色、SSO 或 STS）时需要。使用长期 IAM 用户凭证时请省略它。SDK 客户端会根据凭证来源自动处理此问题。
</Note>

`--aws-sigv4` 的值遵循 `aws:amz:<region>:<service>` 格式。SigV4 服务名称为 `aws-external-anthropic`，且区域必须与您端点 URL 中的区域匹配。任一项不匹配都会产生通用的签名拒绝错误，而不是具体的诊断信息。

### 上下文窗口

Claude Platform on AWS 上的 “context window”（上下文窗口）大小与第一方 Claude API 完全相同。有关每个模型的限制，请参阅[上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)。

### 通过 AWS PrivateLink 连接

要通过 AWS PrivateLink 将您的 VPC 连接到 Claude Platform on AWS 端点，请在您工作区所在的 AWS 区域中创建一个接口 VPC 端点。有关操作步骤，请参阅 AWS 文档中的[使用接口 VPC 端点访问 AWS 服务](https://docs.aws.amazon.com/vpc/latest/privatelink/create-interface-endpoint.html)。

端点服务名称遵循以下格式：

```text wrap
com.amazonaws.{region}.aws-external-anthropic
```

例如：

```text wrap
com.amazonaws.us-east-2.aws-external-anthropic
```

端点服务名称与 base URL 中的主机名 `aws-external-anthropic.{region}.api.aws` 是不同的字符串。该主机名是服务的私有 DNS 名称。在接口端点上启用私有 DNS 后，它会在您的 VPC 内解析到该端点。SDK 客户端和 cURL 请求保持使用相同的 base URL。

## 功能支持

Claude Platform on AWS 直接使用 Claude API 端点，这意味着您可以获得与第一方 Claude API 完全一致的功能（[功能限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#features-not-supported)中注明的情况除外）：

* **功能访问：** 由于 Anthropic 同时运营这两个平台，大多数新功能和 beta 标头无需单独的集成步骤即可在 Claude Platform on AWS 上使用。例外情况请参阅[功能限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#features-not-supported)。
* **Beta 功能：** 传递标准 `anthropic-beta` 标头即可访问 beta 功能，与使用 Claude API 时相同。
* **Agent Skills：** 使用与 Claude API 相同的 `container.skills` 参数来使用预构建和自定义的 [Agent Skills](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview)。所有预构建 Skills（PowerPoint、Excel、Word、PDF）均可开箱即用。
* **代码执行：** 使用[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)在 Anthropic 的托管沙箱中运行代码。
* **工具使用：** 所有 “tool use”（工具使用）[功能](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)均可用。
* **思考：** [自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)、[effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)，以及在支持它的模型上的 “extended thinking”（[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)），均使用与 Claude API 相同的参数。
* **流式传输：** 完整的 SSE “streaming”（流式传输）支持，可提供实时响应。
* **批处理：** 为高吞吐量工作负载提交批处理请求。
* **提示缓存：** 缓存工具、"system prompt"（系统提示）和消息历史记录，以降低 “latency”（延迟）和成本。所有 “prompt caching”（提示缓存）功能（5 分钟 TTL、1 小时 TTL 和自动缓存）均可用。
* **Files API：** 跨请求上传和引用文件。
* **客户管理的加密密钥（CMEK）：** [CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek) 仅支持 [AWS KMS](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-aws-kms) 密钥。无法注册 Google Cloud KMS 和 Azure Key Vault 密钥。该密钥必须是与其所附加工作区位于同一 AWS 账户和区域中的单区域 KMS 密钥，并且其密钥策略必须向 `aws-external-anthropic.amazonaws.com` 服务主体授予访问权限；请参阅[在 Claude Platform on AWS 上设置 CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws)。在 [Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#using-the-claude-console) 中注册和附加密钥；外部密钥端点也可使用，通过 [IAM 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#encryption-keys)进行授权。没有单独的验证步骤：当您将密钥附加到工作区时会隐式验证该密钥（附加调用会执行一轮加密/解密），因此密钥策略问题会在附加时而不是注册时显现。
* **Compliance API：** [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 可用。访问通过 AWS IAM [`ListComplianceActivities` 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#compliance)进行授权。

有关与 Amazon Bedrock 在功能可用性方面的差异，请参阅[对比表](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#claude-platform-on-aws-vs-amazon-bedrock)。

### Claude Managed Agents

[Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 可在 Claude Platform on AWS 上使用，包括[智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)、[环境](https://platform.claude.com/docs/zh-CN/managed-agents/environments)、[会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)、[计划部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments)、[凭证保管库](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)、[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)、[webhook](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks)、[多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)和[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)。

Claude Platform on AWS 上的会话和自托管 worker 与第一方 Claude Managed Agents 在以下三个方面有所不同：

* **自主会话重新身份验证：** 会话可以在没有任何[用户事件](https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types)的情况下自主运行最多 6 小时。6 小时后，会话需要重新进行身份验证才能继续。要重新进行身份验证，请向会话发送任意用户角色事件（请参阅[会话事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)）。第一方 Claude Managed Agents 没有自主会话运行时间限制。
* **[自托管沙箱中的记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)：** 在自托管环境上运行的会话无法附加记忆存储；包含记忆存储的会话会在创建时被拒绝。云环境上的会话可照常附加记忆存储。在第一方 Claude Managed Agents 上，云环境和自托管环境上的会话都可以附加记忆存储。
* **[自托管环境上的 worker 身份验证](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes#prerequisites)：** worker 使用 AWS IAM（SigV4）或[在 AWS Console 中生成的 API 密钥](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#api-key-authentication)进行身份验证，而不是使用环境密钥。在 Claude Console 中生成的环境密钥不适用于 Claude Platform on AWS 端点。

### 不支持的功能

以下功能目前在 Claude Platform on AWS 上不可用：

* **HIPAA 就绪：** Anthropic 的 HIPAA 就绪计划不可用。请参阅 [API 和数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)。
* **Admin API：** 工作区端点（`/v1/organizations/workspaces` 上的创建、获取、列出、更新和归档）、外部密钥端点（`/v1/organizations/external_keys` 上的注册、获取、列出、更新和删除，用于 [CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)；密钥在附加到工作区时进行验证，而不是通过验证端点）、[遥测目标端点](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#telemetry-destinations)以及[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 端点均可用。其他 Admin API 端点（组织成员、工作区成员、邀请、API 密钥、旧版[用量和成本 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-api)以及速率限制报告）不可用。AWS IAM 管理组织成员资格。
* **工作区成员管理：** 不支持在单个工作区中添加或移除用户。工作区 ARN 上的 AWS IAM 策略控制访问权限。
* **Claude Code 工作区和 Claude Code 分析 API：** 带有自动速率限制的 Claude Code 工作区不可用。Claude Code 用量显示在常规用量视图中，而不是专用界面中。
* **OAuth 身份验证：** 不支持。请使用 SigV4 或 API 密钥身份验证。
* **快速模式：** 在 Claude Platform on AWS 上不可用。
* **OpenAI 兼容 API 端点：** 在 Claude Platform on AWS 上不可用。
* **MCP 隧道：** 仅支持通过公共互联网公开的 MCP 服务器。

## 数据驻留

Claude Platform on AWS 支持以下推理地理区域：

* **US：** 推理保留在美国数据中心内。适用 1.1 倍的定价乘数。
* **全局：** 推理可以路由到 Anthropic 在全球运营的任何数据中心。适用标准定价。

<Note>
  您的工作区所绑定的 AWS 区域决定了您调用哪个网关端点，以及 AWS 端资源（IAM、CloudTrail、账单）的作用范围。它并不固定模型推理的运行位置。要将推理固定到特定地理区域，请在每个请求上设置 `inference_geo`，或配置工作区默认值。
</Note>

使用 `inference_geo` 参数为每个请求设置推理地理区域：

<Note>
  `inference_geo` 参数在 Claude 4.6 及更高版本的模型上受支持。在 Claude Opus 4.5、Claude Sonnet 4.5（已弃用）或 Claude Haiku 4.5 上使用 `inference_geo` 的请求会返回 400 错误。有关模型可用性的详细信息，请参阅[数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)。
</Note>

<CodeGroup>
  ```bash cURL
  # 将 URL 和 --aws-sigv4 中的 us-west-2 替换为您的 AWS 区域
  # 如果您使用长期 IAM 用户凭证，请省略 x-amz-security-token 标头
  curl "https://aws-external-anthropic.us-west-2.api.aws/v1/messages" \
    --aws-sigv4 "aws:amz:us-west-2:aws-external-anthropic" \
    -u "$AWS_ACCESS_KEY_ID:$AWS_SECRET_ACCESS_KEY" \
    -H "x-amz-security-token: $AWS_SESSION_TOKEN" \
    -H "content-type: application/json" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-workspace-id: $ANTHROPIC_AWS_WORKSPACE_ID" \
    -d '{
      "model": "claude-sonnet-5-5",
      "max_tokens": 1024,
      "inference_geo": "us",
      "messages": [
        {"role": "user", "content": "Hello!"}
      ]
    }'
  ```

  ```bash CLI
  # 将 us-west-2 替换为您的 AWS 区域
  # ant 会读取 ANTHROPIC_API_KEY。请在 AWS Console 中生成密钥
  # （请参阅 API 密钥身份验证）。
  export ANTHROPIC_API_KEY="YOUR_AWS_API_KEY"

  ant messages create \
    --base-url https://aws-external-anthropic.us-west-2.api.aws \
    --workspace-id "$ANTHROPIC_AWS_WORKSPACE_ID" \
    --model claude-sonnet-5-5 \
    --max-tokens 1024 \
    --inference-geo us \
    --message '{role: user, content: "Hello!"}'
  ```

  ```python Python
  from anthropic import AnthropicAWS

  client = AnthropicAWS()
  message = client.messages.create(
      model="claude-sonnet-5-5",
      max_tokens=1024,
      inference_geo="us",
      messages=[{"role": "user", "content": "Hello!"}],
  )
  print(message)
  ```

  ```typescript TypeScript
  import AnthropicAws from "@anthropic-ai/aws-sdk";
  const client = new AnthropicAws();
  const message = await client.messages.create({
    model: "claude-sonnet-5-5",
    max_tokens: 1024,
    inference_geo: "us",
    messages: [{ role: "user", content: "Hello!" }]
  });
  console.log(message);
  ```

  ```csharp C#
  using Anthropic;
  using Anthropic.Aws;
  using Anthropic.Models.Messages;

  var client = new AnthropicAwsClient();

  var message = await client.Messages.Create(new()
  {
      Model = Model.ClaudeSonnet5_5,
      MaxTokens = 1024,
      InferenceGeo = "us",
      Messages = [new UserMessageParam { Content = "Hello!" }]
  });

  Console.WriteLine(message);
  ```

  ```go Go
  client, err := anthropicaws.NewClient(context.Background(), anthropicaws.ClientConfig{})
  if err != nil {
  	panic(err)
  }

  message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  	Model:        anthropic.ModelClaudeSonnet5_5,
  	MaxTokens:    1024,
  	InferenceGeo: anthropic.String("us"),
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello!")),
  	},
  })
  if err != nil {
  	panic(err)
  }

  fmt.Println(message)
  ```

  ```java Java
  import com.anthropic.aws.backends.AwsBackend;
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.builder()
          .backend(AwsBackend.fromEnv())
          .build();

      Message message = client.messages().create(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_SONNET_5_5)
              .maxTokens(1024)
              .inferenceGeo("us")
              .addUserMessage("Hello!")
              .build()
      );

      IO.println(message);
  }
  ```

  ```php PHP
  use Anthropic\Aws\Client;
  use Anthropic\Messages\Model;

  $client = new Client();

  $message = $client->messages->create(
      model: Model::CLAUDE_SONNET_5_5,
      maxTokens: 1024,
      inferenceGeo: 'us',
      messages: [['role' => 'user', 'content' => 'Hello!']],
  );

  echo $message;
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::AWSClient.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5_5,
    max_tokens: 1024,
    inference_geo: "us",
    messages: [{ role: "user", content: "Hello!" }]
  )

  puts message
  ```
</CodeGroup>

如果您省略 `inference_geo`，请求将使用工作区的 `default_inference_geo`（如果已配置），否则使用 `global`。

工作区级别的推理地理区域控制（`allowed_inference_geos` 和 `default_inference_geo`）在 Claude Platform on AWS 上同样可用。请参阅[工作区级别限制](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency#workspace-level-restrictions)。

## 工作区

Claude Platform on AWS 上的推理和资源请求以某个工作区为目标。您需要在这些 API 调用的 `anthropic-workspace-id` 标头中传递工作区的 ID。工作区 ID 使用带标签的格式，即 `wrkspc_` 后跟字母数字标识符（例如 `wrkspc_01AbCdEf23GhIj`）。如果您还没有工作区 ID，请参阅[获取您的工作区 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#obtain-your-workspace-id)。

### 工作区作用范围

工作区绑定到单个 AWS 区域。在 `us-west-2` 中创建的工作区只能通过 `us-west-2` 端点访问。用量、配额、成本、文件、批处理和 Skills 都按工作区汇总，让您可以在 Claude Console 中查看按区域划分的明细。

工作区也是 Claude Platform on AWS 的主要 IAM 资源。您可以通过使用工作区 ARN 的 AWS IAM 策略来授予或拒绝对特定工作区的访问权限。ARN 的资源段与您在 `anthropic-workspace-id` 标头中传递的以 `wrkspc_` 为前缀的 ID 相同：

```text wrap
arn:aws:aws-external-anthropic:{region}:{account-id}:workspace/{workspace-id}
```

例如：

```text wrap
arn:aws:aws-external-anthropic:us-west-2:123456789012:workspace/wrkspc_01AbCdEf23GhIj
```

有关策略示例，请参阅 [IAM 策略](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#iam-policies)。

### 管理工作区

您可以在 AWS Console 的 **Workspaces** 页面或使用 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 工作区端点来创建其他工作区、重命名工作区或归档工作区。这些端点不需要 `anthropic-workspace-id` 标头。创建和列出操作作用于组织；获取、更新和归档操作在 URL 路径中接收工作区 ID。新工作区会绑定到您调用以创建它的端点所在的 AWS 区域（请参阅[工作区作用范围](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#workspace-scoping)）。拥有 Admin 角色时，您还可以在 Claude Console 的**工作区**页面中创建、重命名和归档工作区。

## 使用 Claude Console

Claude Platform on AWS 使用位于 [platform.claude.com](https://platform.claude.com/) 的标准 Claude Console。当您从 AWS Console 登录时，Claude Console 侧边栏左下角会显示**账户管理方 AWS** 指示器，并且 Console 的作用范围限定为您的 Claude Platform on AWS 组织。它提供用量分析、成本明细、速率限制可见性、工作区管理，以及用于管理文件、Agent Skills、批处理作业和 Claude Managed Agents 资源（代理、会话、环境、凭据保管库、记忆存储和 webhook）的页面。

### 登录

对 Claude Console 的访问通过 AWS IAM 进行联合。有关完整的首次登录流程，请参阅[设置您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#set-up-your-account)。简而言之：

1. 承担具有 `aws-external-anthropic:AssumeConsole` 权限的 IAM 角色。请参阅 [Console 访问](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#console-access)。
2. 在 [AWS Console](https://console.aws.amazon.com/) 中导航到 Claude Platform on AWS 页面。
3. 点击 **Open Claude Console**。AWS Console 会签发 JWT 并将您重定向到 `platform.claude.com`。
4. 首次登录时，系统会提示您输入电子邮件地址。请输入您的工作电子邮件。平台会即时为您预配 Claude Console 用户。

Claude Console 提供两种角色：**管理员**和**开发者**。Admin 角色可访问 Claude Platform on AWS 可用的所有 Claude Console 页面和设置。Developer 角色可读取用量、成本、速率限制和工作区信息。请联系您的 Anthropic 客户代表，为主体分配 Admin 或 Developer 角色。

### 可用页面

**通过 AWS 网关**列表示该页面是否通过 AWS 网关读取和写入数据（因此受 [Claude Platform on AWS 的 IAM 操作](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions)约束）。标记为**否**的页面直接从 Anthropic 读取组织级元数据，并绕过 IAM 操作检查。

| 页面              | 可用          | 通过 AWS 网关 | 说明                                                                                                                                                                                                                                                                                                                                                                  |
| --------------- | ----------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **用量**          | 是           | 否         | 按模型、工作区和维度查看令牌用量。请求后数据可能需要几分钟才会显示。                                                                                                                                                                                                                                                                                                                                  |
| **费用**          | 是           | 否         | 按模型和工作区查看成本明细。AWS Cost Explorer 显示汇总的 [Claude Consumption Unit (CCU)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#billing) 明细项。                                                                                                                                                                                                 |
| **速率限制**        | 是           | 否         | 查看速率限制（只读）。层级提升需通过您的 Anthropic 客户代表办理；请参阅[速率限制和配额](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#rate-limits-and-quotas)。                                                                                                                                                                                                         |
| **日志**          | 是（Admin 角色） | 否         | 查看每个请求的元数据，例如时间、模型、工作区、令牌计数和状态。日志条目不包含提示或响应。                                                                                                                                                                                                                                                                                                                        |
| **工作区**         | 是           | 是（支出限额除外） | 查看按区域划分的工作区。拥有 Admin 角色时，您还可以创建、重命名和归档工作区，并为每个工作区设置[支出限额](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#spend-limits)。                                                                                                                                                                                                            |
| **加密密钥**        | 是           | 是         | 这是 Claude Console 中[数据保留](https://platform.claude.com/settings/privacy#encryption-keys)页面（`platform.claude.com/settings/privacy`）的一个部分，在页面顶部的下拉菜单中选择**组织**时显示。按照[在 Claude Platform on AWS 上设置 CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-claude-platform-on-aws) 中的说明在此处注册 AWS KMS 密钥（Admin 角色）。要将已注册的密钥附加到工作区，请从该下拉菜单中选择工作区，然后在**加密密钥**下选择密钥。 |
| **文件**          | 是           | 是         | 查看和管理已上传的文件。                                                                                                                                                                                                                                                                                                                                                        |
| **Skills**      | 是           | 是         | 查看和管理 Agent Skills。                                                                                                                                                                                                                                                                                                                                                 |
| **批处理**         | 是           | 是         | 查看和管理批处理作业。                                                                                                                                                                                                                                                                                                                                                         |
| **智能体**         | 是           | 是         | 查看和管理代理定义。                                                                                                                                                                                                                                                                                                                                                          |
| **会话**          | 是           | 是         | 查看代理会话和事件历史记录。                                                                                                                                                                                                                                                                                                                                                      |
| **环境**          | 是           | 是         | 查看和管理会话的云沙箱配置。                                                                                                                                                                                                                                                                                                                                                      |
| **凭据保管库**       | 是           | 是         | 查看和管理用于会话身份验证的凭据保管库。                                                                                                                                                                                                                                                                                                                                                |
| **记忆存储**        | 是           | 是         | 查看和管理持久化代理记忆。                                                                                                                                                                                                                                                                                                                                                       |
| **Webhook**     | 是           | 是         | 查看和管理 webhook 端点。                                                                                                                                                                                                                                                                                                                                                   |
| **API 密钥**      | 否           | 不适用       | 在 AWS Console 中管理 API 密钥（**Claude Platform on AWS → API keys**）。请参阅 [API 密钥身份验证](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#api-key-authentication)。                                                                                                                                                                           |
| **成员**          | 否           | 不适用       | 不适用。访问权限由 AWS IAM 管理。                                                                                                                                                                                                                                                                                                                                               |
| **支出限额**        | 是           | 否         | 设置组织每月支出限额；请参阅[支出限额](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#spend-limits)。开票由 AWS Marketplace 管理。在**费用**页面查看成本明细。                                                                                                                                                                                                          |
| **Claude Code** | 否           | 不适用       | 在**用量**页面查看 Claude Code 用量。                                                                                                                                                                                                                                                                                                                                         |

### 切换组织

对于 Claude Platform on AWS，Claude Console 不支持切换组织。要访问其他组织，请退出登录，然后使用该组织 AWS 账户的 IAM 角色通过 AWS Console 重新进行身份验证。

## 速率限制和配额

Claude Platform on AWS 上的组织被置于 Start 层级。Anthropic 直接管理 “rate limit”（速率限制），而不是通过 AWS 配额系统。

随着 Claude Platform on AWS 上的组织积累已支付的 AWS Marketplace 发票记录，它们可以自动升级到更高的用量层级。Claude Console 中的自助式**申请提升等级**流程不可用：**速率限制**页面会引导您改为联系您的 Anthropic 客户代表。

要申请更高的限额，请联系您的 Anthropic 客户代表或 [Anthropic 支持](https://support.claude.com)。请在申请中包含以下内容：

* 您需要提高限额的模型
* 每个模型的峰值每分钟输入令牌数和每分钟输出令牌数（而非每日总量）
* 您的输入中属于缓存或重复上下文的大致比例（缓存读取不计入输入令牌限额；请参阅[缓存感知的 ITPM](https://platform.claude.com/docs/zh-CN/api/rate-limits#cache-aware-itpm)）

用量层级是固定的档位：每个层级都将速率限制与[每月支出上限](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#spend-limits)配对，升级到更高层级会同时提高两者。有关层级详情和每个模型的限额，请参阅[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)。

## 账单

Claude Platform on AWS 通过 [AWS Marketplace](https://aws.amazon.com/marketplace) 计费。用量以 Claude Consumption Units（CCU）计价，按小时计量，并按月在您的 AWS 账单上后付开票。CCU 不是预付额度。不存在 CCU 余额或承诺。

有关 CCU 价格、换算机制、折扣适用方式以及每个模型的令牌费率，请参阅 [Claude Platform on AWS 定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#claude-platform-on-aws-pricing)。

### 支出限额

Start、Build 和 Scale 用量层级各自带有每月支出上限；有关当前数值，请参阅[各层级支出上限](https://platform.claude.com/docs/zh-CN/api/rate-limits#spend-limits)。当您的组织在当前日历月的用量达到其层级的上限时，API 请求将失败并返回[支出上限错误](https://platform.claude.com/docs/zh-CN/api/rate-limits#reaching-your-spend-cap)，直到下个月第一天的 00:00 UTC，在此之前重试不会成功。支出上限和速率限制属于同一层级。要提高上限，或在达到上限后恢复访问，请通过您的 Anthropic 客户代表或 [Anthropic 支持](https://support.claude.com)申请层级提升（请参阅[速率限制和配额](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#rate-limits-and-quotas)）。

您还可以设置低于上限的自定义每月支出限额。首先，[打开 Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#using-the-claude-console) 并前往 [Claude Console 中的支出限额](https://platform.claude.com/settings/spend-limits)（`platform.claude.com/settings/spend-limits`）。在您组织的**邮件收件人**中添加至少一个收件人，然后设置以下任一种限额：

* **组织支出限额：** 在同一页面上，点击您组织的**每月支出限额**旁边的**设置限额**或**调整限额**。
* **工作区支出限额：** 在 Claude Console 的[工作区](https://platform.claude.com/settings/workspaces)页面（`platform.claude.com/settings/workspaces`）中选择一个工作区。在打开的面板中，点击**支出限额**。

当用量达到您设置的限额时，请求将失败并返回 HTTP 400（请参阅[支出限额错误](https://platform.claude.com/docs/zh-CN/api/rate-limits#setting-your-own-spend-limit)），直到下个月第一天的 00:00 UTC，或直到您提高或移除该限额。

支出按目录价格计算，并且可能需要约 2 小时才能反映最近的用量。因此，在请求开始失败之前，用量可能会超出上限或限额。超出部分会被计费。当层级上限或组织支出限额导致您的请求停止时，系统会向您组织的**邮件收件人**下列出的收件人发送电子邮件通知。Claude Platform on AWS 不支持基于角色的收件人（例如所有管理员）。层级上限通知还会发送到在 AWS Marketplace 注册时使用的电子邮件地址。

## 监控和日志记录

AWS CloudTrail 可以捕获对 Claude Platform on AWS 的所有请求。工作区、外部密钥、遥测目标、合规、vault 和 webhook 操作默认记录为管理事件（Management events）。推理、批处理、文件、技能、模型、用户配置文件、用量和成本报告以及 Claude Managed Agents 操作（vault 和 webhook 除外）被归类为数据事件（Data events），需要显式配置数据事件日志记录，这会产生额外的 CloudTrail 费用。有关完整的事件类型分类，请参阅 [IAM 操作参考](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#route-to-action-mapping)；有关配置详情，请参阅 [AWS CloudTrail 文档](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/)。

### 请求 ID

每个响应的响应标头中都包含两个请求 ID：

* **AWS 请求 ID（`x-amzn-requestid`）：** 主 ID，在 CloudTrail 中建立索引。通过 AWS 工具调查请求或联系 AWS 支持时请使用此 ID。
* **Anthropic 请求 ID（`request-id`）：** 次要 ID。联系 Anthropic 支持时请使用此 ID。

<CodeGroup>
  ```bash cURL
  # 请在 URL 和 --aws-sigv4 中将 us-west-2 替换为您的 AWS 区域
  # -i 会在输出中包含响应标头
  # 如果您使用长期 IAM 用户凭证，请省略 x-amz-security-token 标头
  curl -i "https://aws-external-anthropic.us-west-2.api.aws/v1/messages" \
    --aws-sigv4 "aws:amz:us-west-2:aws-external-anthropic" \
    -u "$AWS_ACCESS_KEY_ID:$AWS_SECRET_ACCESS_KEY" \
    -H "x-amz-security-token: $AWS_SESSION_TOKEN" \
    -H "content-type: application/json" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-workspace-id: $ANTHROPIC_AWS_WORKSPACE_ID" \
    -d '{
      "model": "claude-sonnet-5-5",
      "max_tokens": 1024,
      "messages": [
        {"role": "user", "content": "Hello!"}
      ]
    }'
  ```

  ```bash CLI
  # 将 us-west-2 替换为您的 AWS 区域
  # ant 会读取 ANTHROPIC_API_KEY。请在 AWS Console 中生成密钥
  # （请参阅 API 密钥身份验证）。
  export ANTHROPIC_API_KEY="YOUR_AWS_API_KEY"

  # --debug 会将 HTTP 响应（包括 X-Amzn-Requestid 和
  # Request-Id 标头）打印到 stderr
  ant --debug messages create \
    --base-url https://aws-external-anthropic.us-west-2.api.aws \
    --workspace-id "$ANTHROPIC_AWS_WORKSPACE_ID" \
    --model claude-sonnet-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello!"}'
  ```

  ```python Python
  from anthropic import AnthropicAWS

  client = AnthropicAWS()

  response = client.messages.with_raw_response.create(
      model="claude-sonnet-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello!"}],
  )

  print(response.headers.get("x-amzn-requestid"))  # AWS request ID
  print(response.headers.get("request-id"))  # Anthropic request ID

  message = response.parse()
  print(message.content)
  ```

  ```typescript TypeScript
  import AnthropicAws from "@anthropic-ai/aws-sdk";

  const client = new AnthropicAws();

  const { data: message, response } = await client.messages
    .create({
      model: "claude-sonnet-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello!" }]
    })
    .withResponse();

  console.log(response.headers.get("x-amzn-requestid")); // AWS request ID
  console.log(response.headers.get("request-id")); // Anthropic request ID
  console.log(message.content);
  ```

  ```csharp C#
  using Anthropic;
  using Anthropic.Aws;
  using Anthropic.Models.Messages;

  var client = new AnthropicAwsClient();

  using var response = await client.WithRawResponse.Messages.Create(new()
  {
      Model = Model.ClaudeSonnet5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello!" }]
  });

  Console.WriteLine(response.GetHeaderValues("x-amzn-requestid").First()); // AWS request ID
  Console.WriteLine(response.RequestID); // Anthropic request ID
  Console.WriteLine(await response.Deserialize());
  ```

  ```go Go
  client, err := anthropicaws.NewClient(context.Background(), anthropicaws.ClientConfig{})
  if err != nil {
  	panic(err)
  }

  var response *http.Response
  message, err := client.Messages.New(
  	context.Background(),
  	anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeSonnet5_5,
  		MaxTokens: 1024,
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello!")),
  		},
  	},
  	option.WithResponseInto(&response),
  )
  if err != nil {
  	panic(err)
  }

  fmt.Println(response.Header.Get("x-amzn-requestid")) // AWS request ID
  fmt.Println(response.Header.Get("request-id"))       // Anthropic request ID
  fmt.Println(message.Content)
  ```

  ```java Java
  import com.anthropic.aws.backends.AwsBackend;
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.core.http.HttpResponseFor;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.builder()
          .backend(AwsBackend.fromEnv())
          .build();

      HttpResponseFor<Message> response = client.messages().withRawResponse().create(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_SONNET_5_5)
              .maxTokens(1024)
              .addUserMessage("Hello!")
              .build()
      );

      IO.println(response.headers().values("x-amzn-requestid").get(0)); // AWS request ID
      IO.println(response.requestId().orElse(null)); // Anthropic request ID
      IO.println(response.parse().content());
  }
  ```

  ```php PHP
  use Anthropic\Aws\Client;
  use Anthropic\Messages\Model;

  $client = new Client();

  $response = $client->messages->raw->create([
      'model' => Model::CLAUDE_SONNET_5_5,
      'maxTokens' => 1024,
      'messages' => [['role' => 'user', 'content' => 'Hello!']],
  ]);

  echo $response->getHeaderLine('x-amzn-requestid') . "\n"; // AWS request ID
  echo $response->getHeaderLine('request-id') . "\n"; // Anthropic request ID
  echo $response->parse();
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::AWSClient.new

  # 在逐请求 middleware（中间件）中读取响应标头，该中间件会在
  # SDK 解析之前接收原始 HTTP 响应
  aws_request_id = nil
  anthropic_request_id = nil
  read_request_ids = lambda do |request, call_next|
    response = call_next.call(request)
    # response.headers 中的键均为小写
    aws_request_id = response.headers["x-amzn-requestid"]
    anthropic_request_id = response.headers["request-id"]
    response
  end

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_SONNET_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello!" }],
    request_options: { middleware: [read_request_ids] }
  )

  puts aws_request_id # AWS request ID
  puts anthropic_request_id # Anthropic request ID
  puts message.content
  ```
</CodeGroup>

Anthropic 建议至少以 30 天滚动周期记录您的活动，以便了解使用模式并调查问题。

<Note>
  AWS CloudTrail 在您的 AWS 账户内配置。启用日志记录不会让 AWS 或 Anthropic 访问您的内容，超出计费和服务运营所必需的范围。
</Note>

## 从 Amazon Bedrock 迁移

如果您目前在 Bedrock 上使用 Claude，迁移到 Claude Platform on AWS 需要对您的集成进行全面更改。SigV4 签名仍然受支持，但签名上下文、基础 URL、API 格式、模型 ID、SDK 客户端和软件包、"streaming"（流式传输）格式、请求标头以及区域可用性都会发生变化。Claude Platform on AWS 还会预配一个新的 Anthropic 组织。下表总结了这些差异。

### 变化内容

迁移的差异取决于您从哪种 Bedrock 集成迁移而来。下表同时展示了 [Amazon Bedrock 中的 Claude（`bedrock-mantle`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)（位于 `bedrock-mantle.{region}.api.aws` 的 Messages API）和 [Amazon Bedrock 上的 Claude（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)（InvokeModel 和 Converse API）。

| 方面               | 从 [Amazon Bedrock（`bedrock-mantle`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)                                                                                                                        | 从 [Amazon Bedrock（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)                                                                                                                       | 到 Claude Platform on AWS                                                                                                                                                                                                                                                                                                                |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **基础 URL**       | `bedrock-mantle.{region}.api.aws`                                                                                                                                                                                                              | `bedrock-runtime.{region}.amazonaws.com`                                                                                                                                                                                                              | `aws-external-anthropic.{region}.api.aws`                                                                                                                                                                                                                                                                                               |
| **API 格式**       | 位于 `/anthropic/v1/messages` 的 Messages API                                                                                                                                                                                                     | Bedrock Converse / InvokeModel                                                                                                                                                                                                                        | Claude API（`/v1/{endpoint}`）                                                                                                                                                                                                                                                                                                            |
| **模型 ID**        | anthropic.claude-haiku-4-5                                                                                                                                                                                                                     | anthropic.claude-haiku-4-5-20251001-v1:0（带有 `us.` 或 `global.` 推理配置文件前缀）                                                                                                                                                                               | claude-haiku-4-5                                                                                                                                                                                                                                                                                                                        |
| **SDK 客户端**      | `AnthropicBedrockMantle`                                                                                                                                                                                                                       | `AnthropicBedrock` / Bedrock SDK                                                                                                                                                                                                                      | 平台专用客户端（请参阅[安装 SDK](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#install-an-sdk)），处于 beta 阶段                                                                                                                                                                                                         |
| **SDK 软件包**      | `anthropic[bedrock]` (typescript: `@anthropic-ai/bedrock-sdk`; csharp: `Anthropic.Bedrock`; go: `github.com/anthropics/anthropic-sdk-go/v2/bedrock`; java: `com.anthropic:anthropic-java-bedrock`; php: `anthropic-ai/sdk`; ruby: `anthropic`) | `anthropic[bedrock]` (typescript: `@anthropic-ai/bedrock-sdk`; csharp: `Anthropic.Bedrock`; go: `github.com/anthropics/anthropic-sdk-go/bedrock`; java: `com.anthropic:anthropic-java-bedrock`; php: `anthropic-ai/sdk`; ruby: `anthropic`)，或 AWS SDK | `anthropic[aws]` (typescript: `@anthropic-ai/aws-sdk`; csharp: `Anthropic.Aws`; go: `github.com/anthropics/anthropic-sdk-go/v2/aws`; java: `com.anthropic:anthropic-java-aws`; php: `anthropic-ai/sdk`; ruby: `anthropic`)（请参阅[安装 SDK](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#install-an-sdk)） |
| **SigV4 服务名称**   | `bedrock-mantle`                                                                                                                                                                                                                               | `bedrock`                                                                                                                                                                                                                                             | `aws-external-anthropic`                                                                                                                                                                                                                                                                                                                |
| **流式传输格式**       | SSE                                                                                                                                                                                                                                            | AWS EventStream                                                                                                                                                                                                                                       | SSE（与 Claude API 相同）                                                                                                                                                                                                                                                                                                                    |
| **工作区标头**        | 不适用                                                                                                                                                                                                                                            | 不适用                                                                                                                                                                                                                                                   | `anthropic-workspace-id`，推理和资源请求必需                                                                                                                                                                                                                                                                                                      |
| **区域可用性**        | 请参阅 [Amazon Bedrock 区域](https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-regions.html)                                                                                                                                             | 请参阅 [Amazon Bedrock 区域](https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-regions.html)                                                                                                                                                    | 所有 AWS 商业区域                                                                                                                                                                                                                                                                                                                             |
| **Anthropic 组织** | 无需                                                                                                                                                                                                                                             | 无需                                                                                                                                                                                                                                                    | 注册时创建新组织。现有组织无法转换（请参阅[从现有 Anthropic 组织迁移](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#moving-from-an-existing-anthropic-organization)）                                                                                                                                                              |

如果您使用的是 Amazon Bedrock 中的 Claude，请求正文格式已经是 Messages API。需要更改的是基础 URL、SigV4 服务名称、模型 ID，以及新增的 `anthropic-workspace-id` 标头。如果您使用的是 InvokeModel 或 Converse API，您还需要将请求和响应结构重写为 Messages API 格式。有关请求结构的映射，请参阅 [Amazon Bedrock 上的 Claude（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)。

### 您将获得

* 通常可在同一天访问新模型和功能（请参阅[功能限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#features-not-supported)）
* 用于文档生成（PowerPoint、Excel、Word、PDF）的 Agent Skills
* 在 Anthropic 托管的沙箱中执行代码
* 通过 `anthropic-beta` 标头使用 Amazon Bedrock 未提供的 Claude API beta 功能（请参阅[功能限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#features-not-supported)）
* 用于配额可见性和用量分析的 Claude Console
* Anthropic 直接支持
* 作为 SigV4 替代方案的 API 密钥身份验证（请参阅 [API 密钥身份验证](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#api-key-authentication)）

### 保持不变的内容

* AWS IAM 身份验证（SigV4）
* AWS 作为开票方。计费渠道从原生 AWS 服务变为 AWS Marketplace（请参阅[商业注意事项](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#commercial-considerations)）。
* AWS 承诺消耗抵扣

### 迁移陷阱

<Warning>
  **请先启用出站 Web 身份联合。** 如果您的 AWS 账户之前未使用过 Claude Platform on AWS，则在发出请求之前，您必须为每个账户[启用出站 Web 身份联合](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#enable-outbound-web-identity-federation)一次。如果没有此步骤，所有请求都会因联合错误而失败（有关确切错误和补救措施，请参阅[启用出站 Web 身份联合](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#enable-outbound-web-identity-federation)）。Bedrock 不需要此步骤。
</Warning>

<Warning>
  **在 Claude Platform on AWS 上，零数据保留（ZDR）需选择加入。** 在 Bedrock 上，AWS 是数据处理方，Anthropic 不会保留推理输入或输出。Anthropic 的 ZDR 计划不适用于该场景。在 Claude Platform on AWS 上，Anthropic 作为独立数据处理方处理推理数据，ZDR 遵循第一方 Claude API 模式：可通过您的 Anthropic 客户代表申请。在迁移依赖数据保留保证的生产工作负载之前，请确认已加入 ZDR。
</Warning>

### 商业注意事项

* **Anthropic 服务条款：** 使用 Claude Platform on AWS 需要接受 Anthropic 的商业服务条款和使用政策。如果您的组织尚未接受这些条款（例如，您只通过 Bedrock 使用过 Claude），系统会在账户设置期间提示您接受。请参阅[设置您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#set-up-your-account)。
* **折扣和私有报价：** 协商折扣和 AWS Marketplace 私有报价不会在 Bedrock 和 Claude Platform on AWS 之间自动转移。请与您的 Anthropic 客户代表合作，为 Claude Platform on AWS 设置商业条款。

## IAM 策略

Claude Platform on AWS 与 AWS IAM 集成以进行访问控制。您可以使用标准 IAM 策略语法，授予或拒绝对特定工作区上特定 API 操作的访问权限。

SigV4 服务名称和 IAM 操作命名空间为 `aws-external-anthropic`。操作遵循 `aws-external-anthropic:<Action>` 模式（例如 `aws-external-anthropic:CreateInference`）。

每个调用 API 的主体，无论使用 SigV4 还是 API 密钥，都还需要两个 AWS STS 操作的权限：`sts:GetWebIdentityToken` 和 `sts:TagGetWebIdentityToken`。Claude Platform on AWS 网关会以该主体的身份调用 STS，以获取一个转发给 Anthropic 的 JWT（请参阅[启用出站 Web 身份联合](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#enable-outbound-web-identity-federation)）。如果缺少任一权限，即使已授予 `aws-external-anthropic` 操作，请求也会失败并返回 403 `permission_error`，其消息中会指明被拒绝的操作。

如果主体拥有 `AnthropicFullAccess`、`AnthropicLimitedAccess` 或 `AnthropicReadOnlyAccess` AWS 托管策略，或者拥有管理员访问权限，则它已具备这两个操作的权限。自定义策略必须授予这两个操作，如下例所示。与这些托管策略一样，该示例仅在网关发起调用且指定 Anthropic 的两个受众之一时才允许 `sts:GetWebIdentityToken`。

### 示例：拒绝批量推理

以下策略允许实时推理，同时阻止批处理：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "aws-external-anthropic:CreateInference",
        "aws-external-anthropic:CountTokens",
        "aws-external-anthropic:GetModel",
        "aws-external-anthropic:ListModels",
        "aws-external-anthropic:GetWorkspace"
      ],
      "Resource": "arn:aws:aws-external-anthropic:*:*:workspace/*"
    },
    {
      "Effect": "Allow",
      "Action": ["aws-external-anthropic:ListWorkspaces", "sts:TagGetWebIdentityToken"],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "sts:GetWebIdentityToken",
      "Resource": "*",
      "Condition": {
        "ForAnyValue:StringEquals": {
          "sts:IdentityTokenAudience": [
            "https://api.anthropic.com",
            "https://platform.claude.com"
          ]
        },
        "StringEquals": {
          "aws:CalledViaLast": "aws-external-anthropic.amazonaws.com"
        }
      }
    },
    {
      "Effect": "Deny",
      "Action": [
        "aws-external-anthropic:CreateBatchInference",
        "aws-external-anthropic:GetBatchInference",
        "aws-external-anthropic:ListBatchInferences"
      ],
      "Resource": "*"
    }
  ]
}
```

`GetBatchInference` 操作同时授权批处理元数据路由和批处理结果路由。拒绝该操作会阻止这两种读取。有关适用于 ZDR 敏感工作负载的仅 Deny 策略，请参阅[对 ZDR 敏感工作区进行功能锁定](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#feature-lockdown-for-a-zdr-sensitive-workspace)。

<Note>
  `ListWorkspaces` 是账户范围的操作，因此它出现在一个单独的 Allow 语句中，并带有 `"Resource": "*"`。在账户范围的操作上指定工作区 ARN 不会生效（请参阅[预配自动化](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#provisioning-automation)）。

  此策略假定使用 AWS SigV4 身份验证。如果主体使用 API 密钥进行身份验证，还需将 `aws-external-anthropic:CallWithBearerToken` 添加到包含 `ListWorkspaces` 的 `"Resource": "*"` Allow 语句中。`CallWithBearerToken` 是一个无路由的身份验证层操作，不绑定到工作区 ARN。有关授予该操作的策略，请参阅[按客户隔离工作区](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#per-customer-workspace-isolation)。
</Note>

### 托管策略

AWS 提供五个托管策略（`AnthropicFullAccess`、`AnthropicReadOnlyAccess`、`AnthropicInferenceAccess`、`AnthropicLimitedAccess` 和 `AnthropicSelfHostedEnvironmentAccess`），用于常见的访问模式。有关每个策略授予的操作，请参阅 Claude Platform on AWS 的 IAM 操作中的[托管策略](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions#managed-policies)。同一页面还提供了完整的 IAM 操作列表、路由到操作的映射以及更多策略示例。

## 后续步骤

<CardGroup cols={2}>
  <Card title="功能概览" icon="stack" href="https://platform.claude.com/docs/zh-CN/build-with-claude/overview">
    探索 Claude 的高级功能和能力。
  </Card>

  <Card title="定价" icon="chart" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing#claude-platform-on-aws-pricing">
    了解 Claude Platform on AWS 定价和 Claude Consumption Unit 费率。
  </Card>

  <Card title="模型弃用" icon="arrow-clockwise" href="https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations">
    随着更安全、更强大的模型发布，Anthropic 会定期停用较旧的模型。查看所有 API 弃用信息以及推荐的替代模型。
  </Card>
</CardGroup>

## 其他资源

<CardGroup cols={2}>
  <Card title="Claude Console" icon="browser" href="https://platform.claude.com/">
    在 Claude Console 中查看用量、成本和工作区。通过 AWS Console 登录。
  </Card>

  <Card title="Amazon Bedrock 中的 Claude（bedrock-mantle）" icon="cloud" href="https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock">
    如果您需要 AWS 作为唯一的数据处理方，请使用由 AWS 运营的 Claude。
  </Card>

  <Card title="AWS Marketplace" icon="coins" href="https://aws.amazon.com/marketplace">
    管理您的 AWS Marketplace 订阅和账单。
  </Card>
</CardGroup>
