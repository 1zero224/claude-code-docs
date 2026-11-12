---
title: Amazon Bedrock 中的 Claude（bedrock-mantle）
url: https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock
description: 通过 Amazon Bedrock 的 bedrock-mantle 端点上的 Messages API 调用 Claude，使用 AWS 原生的身份验证、计费和安全边界。
---

本指南将引导您完成在 Amazon Bedrock 中设置 Claude 并进行 API 调用的过程。Amazon Bedrock 中的 Claude 运行在 AWS 托管的基础设施上，具有零运营人员访问权限：Anthropic 人员无法访问推理基础设施。您可以完全在 AWS 安全边界内构建敏感应用程序，同时使用与 Claude API 相同的 Messages API 形式。

<Note>
  本页面介绍 Amazon Bedrock 中的 Claude，它通过 `bedrock-mantle` 端点上位于 `/anthropic/v1/messages` 的 Messages API 提供 Claude 服务。该端点运行在 AWS 托管的基础设施上。位于 `bedrock-runtime` 端点上的 `InvokeModel` 和 `Converse` API 的相关文档请参阅 [Amazon Bedrock 上的 Claude（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)。如需在 AWS 上使用由 Anthropic 运营、通过 AWS Marketplace 计费的替代方案，请参阅 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)。
</Note>

## 访问权限

Amazon Bedrock 为每个 Claude 模型单独设置访问条件。Claude Fable 5.1、Claude Fable 5、Claude Opus 4.8、Claude Sonnet 5、Claude Opus 4.7 和 Claude Haiku 4.5 向所有 Amazon Bedrock 客户开放。如需了解任何其他模型的访问条件，请在 AWS 控制台中查看 [Amazon Bedrock 模型访问](https://console.aws.amazon.com/bedrock/home#/modelaccess)。有关区域可用性，请参阅[区域](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock#regions)。

## 前提条件

开始之前，请确保您具备：

* 一个已为您打算使用的 Claude 模型启用 [Amazon Bedrock 模型访问](https://console.aws.amazon.com/bedrock/home#/modelaccess)的 AWS 账户
* 已安装并配置 [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)（可选，用于凭证管理）

## 身份验证

Amazon Bedrock 中的 Claude 支持三种身份验证路径。请选择最符合您安全要求的一种。

### Bedrock 服务角色（推荐）

使用带有 AWS 托管密钥的 Bedrock 服务角色，以获得最安全、长期有效的访问：

<Steps>
  <Step title="管理员：配置服务角色">
    AWS 管理员配置一个 Bedrock 服务角色，并授予开发人员对该服务角色 ARN 的 `iam:PassRole` 权限。
  </Step>

  <Step title="开发人员：传递角色">
    当您调用 API 时，Bedrock 会代表您承担该服务角色。有关如何将角色与请求关联，请参阅 [Amazon Bedrock 文档](https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-mantle.html)。
  </Step>
</Steps>

### IAM 代入角色

用于身份联合访问，会话最长 12 小时：

<Steps>
  <Step title="管理员：配置 IAM 角色">
    创建一个作用范围限定于您的 Claude 模型的 IAM 角色。信任策略指定您的身份提供商（SAML、OIDC 或 AWS Identity Center）。权限策略仅对允许的模型 ARN 授予 `bedrock-mantle:CreateInference`。
  </Step>

  <Step title="开发人员：进行身份验证并代入角色">
    通过您的企业身份提供商进行身份验证，然后代入该 IAM 角色。AWS STS 会颁发临时凭证，SDK 或 CLI 使用这些凭证对请求进行签名。
  </Step>
</Steps>

### Bearer 令牌

用于无需 IAM 角色的短期访问（最长 12 小时，最不推荐）：

<Steps>
  <Step title="管理员：限制令牌类型">
    通过附加一个策略来阻止长期密钥：除非 `bedrock:BearerTokenType` 条件匹配短期令牌，否则拒绝 `bedrock:CallWithBearerToken`。
  </Step>

  <Step title="开发人员：生成令牌">
    使用 `aws-bedrock-token-generator` CLI 生成一个 bearer 令牌。在每个请求的 `x-api-key` 标头中传递该令牌。
  </Step>
</Steps>

## 安装 SDK

[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 通过 Bedrock 专用的包或模块支持 Amazon Bedrock 中的 Claude。

<Tabs>
  <Tab title="Python">
    ```bash
    pip install -U "anthropic[bedrock]"
    ```
  </Tab>

  <Tab title="TypeScript">
    ```bash
    npm install @anthropic-ai/bedrock-sdk
    ```
  </Tab>

  <Tab title="C#">
    ```bash
    dotnet add package Anthropic.Bedrock
    ```
  </Tab>

  <Tab title="Go">
    ```bash
    go get github.com/anthropics/anthropic-sdk-go/v2/bedrock
    ```
  </Tab>

  <Tab title="Java">
    <Tabs>
      <Tab title="Gradle">
        ```kotlin
        implementation("com.anthropic:anthropic-java:3.8.0")
        implementation("com.anthropic:anthropic-java-bedrock:3.8.0")
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
            <artifactId>anthropic-java-bedrock</artifactId>
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
    # Gemfile
    gem "anthropic"
    gem "aws-sdk-core"
    ```
  </Tab>
</Tabs>

## 发出您的第一个请求

端点遵循 `https://bedrock-mantle.{region}.api.aws/anthropic/v1/messages` 模式。此端点使用标准 SSE “streaming”（流式传输），以及与 Claude API 相同的请求体结构。

请在创建客户端时传递区域，或设置 `AWS_REGION` 环境变量。如果两者都未设置，Python、TypeScript、C#、Go、PHP 和 Ruby SDK 会尝试使用 `AWS_DEFAULT_REGION`。Java SDK 仅在您调用 `BedrockMantleBackend.fromEnv()` 时才读取环境，该方法使用[默认 AWS 区域提供程序链](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/region-selection.html)。该链还会检查您的 AWS 配置文件和 EC2 实例元数据。

对于凭证，客户端使用您在创建时传递的凭证。否则，`AWS_BEARER_TOKEN_BEDROCK` 环境变量优先于客户端的 AWS 凭证链，该凭证链包括 `AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY` 和 `AWS_SESSION_TOKEN` 环境变量。

<Tabs>
  <Tab title="cURL">
    ```bash
    curl https://bedrock-mantle.us-east-1.api.aws/anthropic/v1/messages \
      --aws-sigv4 "aws:amz:us-east-1:bedrock-mantle" \
      -u "$AWS_ACCESS_KEY_ID:$AWS_SECRET_ACCESS_KEY" \
      -H "x-amz-security-token: $AWS_SESSION_TOKEN" \
      -H "content-type: application/json" \
      -H "anthropic-version: 2026-10-21" \
      -d '{
        "model": "anthropic.claude-opus-5-5",
        "max_tokens": 1024,
        "messages": [
          {"role": "user", "content": "Hello, Claude"}
        ]
      }'
    ```
  </Tab>

  <Tab title="CLI">
    `ant` CLI 不支持 Amazon Bedrock。请使用 cURL 或 SDK。
  </Tab>

  <Tab title="Python">
    ```python
    from anthropic import AnthropicBedrockMantle

    client = AnthropicBedrockMantle(aws_region="us-east-1")

    message = client.messages.create(
        model="anthropic.claude-opus-5-5",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Hello, Claude"}],
    )

    print(next(block.text for block in message.content if block.type == "text"))
    ```
  </Tab>

  <Tab title="TypeScript">
    ```typescript
    import { AnthropicBedrockMantle } from "@anthropic-ai/bedrock-sdk";

    const client = new AnthropicBedrockMantle({
      awsRegion: "us-east-1"
    });

    const message = await client.messages.create({
      model: "anthropic.claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello, Claude" }]
    });

    const textBlock = message.content.find((block) => block.type === "text");
    if (textBlock) {
      console.log(textBlock.text);
    }
    ```
  </Tab>

  <Tab title="C#">
    ```csharp
    using Anthropic.Bedrock;
    using Anthropic.Models.Messages;

    var client = new AnthropicBedrockMantleClient(new() { AwsRegion = "us-east-1" });

    var message = await client.Messages.Create(new()
    {
        Model = "anthropic.claude-opus-5-5",
        MaxTokens = 1024,
        Messages = [new UserMessageParam { Content = "Hello, Claude" }],
    });

    foreach (var item in message.Content)
    {
        if (item.Value is TextBlock block)
        {
            Console.WriteLine(block.Text);
            break;
        }
    }
    ```
  </Tab>

  <Tab title="Go">
    ```go
    client, err := bedrock.NewMantleClient(context.Background(), bedrock.MantleClientConfig{
    	AWSRegion: "us-east-1",
    })
    if err != nil {
    	panic(err)
    }

    message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
    	Model:     "anthropic.claude-opus-5-5",
    	MaxTokens: 1024,
    	Messages: []anthropic.MessageParam{
    		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
    	},
    })
    if err != nil {
    	panic(err)
    }

    for _, block := range message.Content {
    	if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
    		fmt.Println(textBlock.Text)
    		break
    	}
    }
    ```
  </Tab>

  <Tab title="Java">
    ```java
    import com.anthropic.bedrock.backends.BedrockMantleBackend;
    import com.anthropic.client.AnthropicClient;
    import com.anthropic.client.okhttp.AnthropicOkHttpClient;
    import com.anthropic.models.messages.ContentBlock;
    import com.anthropic.models.messages.Message;
    import com.anthropic.models.messages.MessageCreateParams;

    void main() {
        AnthropicClient client = AnthropicOkHttpClient.builder()
            .backend(BedrockMantleBackend.fromEnv())
            .build();

        Message message = client.messages().create(
            MessageCreateParams.builder()
                .model("anthropic.claude-opus-5-5")
                .maxTokens(1024)
                .addUserMessage("Hello, Claude")
                .build()
        );

        message.content().stream()
            .filter(ContentBlock::isText)
            .findFirst()
            .ifPresent(block -> IO.println(block.asText().text()));
    }
    ```
  </Tab>

  <Tab title="PHP">
    ```php
    use Anthropic\Bedrock\MantleClient;

    $client = new MantleClient(awsRegion: 'us-east-1');

    $message = $client->messages->create(
        model: 'anthropic.claude-opus-5-5',
        maxTokens: 1024,
        messages: [
            ['role' => 'user', 'content' => 'Hello, Claude'],
        ],
    );

    echo array_find($message->content, fn ($block) => $block->type === 'text')->text;
    ```
  </Tab>

  <Tab title="Ruby">
    ```ruby
    require "anthropic"

    client = Anthropic::BedrockMantleClient.new(aws_region: "us-east-1")

    message = client.messages.create(
      model: "anthropic.claude-opus-5-5",
      max_tokens: 1024,
      messages: [{role: "user", content: "Hello, Claude"}]
    )

    puts message.content.find { it.type == :text }.text
    ```
  </Tab>
</Tabs>

<Tip>
  您也可以使用 `Anthropic` (go: `anthropic.NewClient()`; java: `AnthropicOkHttpClient.builder()`; csharp: `AnthropicClient`; php: `Anthropic\Client`; ruby: `Anthropic::Client`) 创建标准客户端：将 `base_url` (typescript: `baseURL`; go: `option.WithBaseURL()`; java: `.baseUrl()`; csharp: `BaseUrl`; php: `baseUrl`) 设置为 `https://bedrock-mantle.{region}.api.aws/anthropic`，并将您的 bearer 令牌作为 `api_key` (typescript, php: `apiKey`; go: `option.WithAPIKey()`; java: `.apiKey()`; csharp: `ApiKey`) 传递。此方式仅支持 bearer 令牌身份验证。SigV4 签名需要使用 `AnthropicBedrockMantle` (csharp: `AnthropicBedrockMantleClient`; go: `bedrock.NewMantleClient()`; java: `BedrockMantleBackend`; php: `MantleClient`; ruby: `Anthropic::BedrockMantleClient`)。
</Tip>

## 支持的模型

Amazon Bedrock 中的 Claude 的模型 ID 带有 `anthropic.` 提供商前缀。有关模型的功能和行为，请参阅[模型概览](https://platform.claude.com/docs/zh-CN/models/overview)。Claude Fable 5.2 和 Claude Mythos 5.2 在 Amazon Bedrock 中的 Claude 中不可用。Amazon Bedrock 仅在 `bedrock-runtime` 端点上提供这两个模型。有关它们的模型 ID，请参阅 [API 模型 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#api-model-ids)。如需了解每个模型由哪个端点提供，请参阅 AWS 文档中的[按模型划分的端点可用性](https://docs.aws.amazon.com/bedrock/latest/userguide/models-endpoint-availability.html)。

| Model                                                                                       | Model ID                      | Access                                                                                                 |
| :------------------------------------------------------------------------------------------ | :---------------------------- | :----------------------------------------------------------------------------------------------------- |
| Claude Fable 5.1                                                                            | `anthropic.claude-fable-5-1`  | Open                                                                                                   |
| Claude Mythos 5.1 ([limited availability](https://support.claude.com/en/articles/14604842)) | `anthropic.claude-mythos-5-1` | Active (verification required)                                                                         |
| Claude Fable 5                                                                              | `anthropic.claude-fable-5`    | Open                                                                                                   |
| Claude Mythos 5 ([limited availability](https://support.claude.com/en/articles/14604842))   | `anthropic.claude-mythos-5`   | Active (verification required)                                                                         |
| Claude Opus 5.5                                                                             | `anthropic.claude-opus-5-5`   | [See Access](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock#access) |
| Claude Opus 5                                                                               | `anthropic.claude-opus-5`     | [See Access](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock#access) |
| Claude Opus 4.8                                                                             | `anthropic.claude-opus-4-8`   | Open                                                                                                   |
| Claude Opus 4.7                                                                             | `anthropic.claude-opus-4-7`   | Open                                                                                                   |
| Claude Sonnet 5.5                                                                           | `anthropic.claude-sonnet-5-5` | [See Access](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock#access) |
| Claude Sonnet 5                                                                             | `anthropic.claude-sonnet-5`   | Open                                                                                                   |
| Claude Haiku 5.5                                                                            | `anthropic.claude-haiku-5-5`  | [See Access](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock#access) |
| Claude Haiku 4.5                                                                            | `anthropic.claude-haiku-4-5`  | Open                                                                                                   |

与 Amazon Bedrock 中的 Claude 配合使用时，请使用最新版本的 Claude Code；运行 `claude update` 进行升级。

让 Claude Code 将您的代码迁移到更新的模型

```text
/claude-api migrate 将此项目迁移到最新的 Claude Opus 模型
```

## 功能支持

有关包含 Amazon Bedrock 可用性的完整功能列表，请参阅[功能概览](https://platform.claude.com/docs/zh-CN/build-with-claude/overview)。

### 支持的功能亮点

* [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create)（`/anthropic/v1/messages`）
* [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)
* [思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)
* [工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)，包括 [Bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)、[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)、[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)、[记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)和[文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)
* [引用](https://platform.claude.com/docs/zh-CN/build-with-claude/citations)
* [结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)

### 不支持的功能

* 输入源（图像和文档的 URL 源、Files API）
* 服务器端工具（代码执行、网络搜索、网页获取、advisor）
* 智能体基础设施（Agent Skills、MCP 连接器、程序化工具调用）
* API 端点（Message Batches、Models、Admin、Compliance、用量和成本）
* Claude Managed Agents
* 服务器端回退（[`fallbacks` 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)；请改用[客户端回退模式](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#client-side-fallback)）

## 区域

Amazon Bedrock 中的 Claude 在以下 AWS 区域可用。Amazon Bedrock 提供两种端点类型：

* **全球：** 在所有可用区域之间动态路由，以实现最高可用性。无价格溢价。
* **区域：** 端点解析到您指定的单个 AWS 区域，以满足数据驻留要求。区域端点的价格比全球端点高 10%。如需在某一地理范围内的多个区域之间路由，请使用 “inference profile”（[推理配置文件](https://docs.aws.amazon.com/bedrock/latest/userguide/cross-region-inference.html)）（US、EU、JP 或 AU）。表中标记为**仅区域内**的区域支持无需推理配置文件的直接单区域路由。

全球端点适用于 Claude Fable 5.1、Claude Fable 5、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8、Claude Opus 4.7、Claude Sonnet 5.5、Claude Sonnet 5、Claude Haiku 5.5 和 Claude Haiku 4.5。

| AWS 区域           | 位置             | 端点类型       |
| ---------------- | -------------- | ---------- |
| `af-south-1`     | 非洲（开普敦）        | 全球         |
| `ap-northeast-1` | 亚太地区（东京）       | 全球、JP、仅区域内 |
| `ap-northeast-2` | 亚太地区（首尔）       | 全球         |
| `ap-northeast-3` | 亚太地区（大阪）       | 全球、JP      |
| `ap-south-1`     | 亚太地区（孟买）       | 全球         |
| `ap-south-2`     | 亚太地区（海得拉巴）     | 全球         |
| `ap-southeast-1` | 亚太地区（新加坡）      | 全球         |
| `ap-southeast-2` | 亚太地区（悉尼）       | 全球、AU      |
| `ap-southeast-3` | 亚太地区（雅加达）      | 全球         |
| `ap-southeast-4` | 亚太地区（墨尔本）      | 全球、AU、仅区域内 |
| `ca-central-1`   | 加拿大（中部）        | 全球、US      |
| `ca-west-1`      | 加拿大西部（卡尔加里）    | 全球         |
| `eu-central-1`   | 欧洲（法兰克福）       | 全球、EU      |
| `eu-central-2`   | 欧洲（苏黎世）        | 全球、EU      |
| `eu-north-1`     | 欧洲（斯德哥尔摩）      | 全球、EU、仅区域内 |
| `eu-south-1`     | 欧洲（米兰）         | 全球、EU      |
| `eu-south-2`     | 欧洲（西班牙）        | 全球、EU      |
| `eu-west-1`      | 欧洲（爱尔兰）        | 全球、EU、仅区域内 |
| `eu-west-2`      | 欧洲（伦敦）         | 全球、EU      |
| `eu-west-3`      | 欧洲（巴黎）         | 全球、EU      |
| `il-central-1`   | 以色列（特拉维夫）      | 全球         |
| `me-central-1`   | 中东（阿联酋）        | 全球         |
| `sa-east-1`      | 南美洲（圣保罗）       | 全球         |
| `us-east-1`      | 美国东部（弗吉尼亚州北部）  | 全球、US、仅区域内 |
| `us-east-2`      | 美国东部（俄亥俄州）     | 全球、US、仅区域内 |
| `us-west-1`      | 美国西部（加利福尼亚州北部） | 全球、US      |
| `us-west-2`      | 美国西部（俄勒冈州）     | 全球、US、仅区域内 |

## 配额

默认配额为每分钟 200 万输入令牌（TPM）。您可以申请最高 500 万输入 TPM 和 50 万输出 TPM，无需 Anthropic 额外批准。AWS 在 Bedrock 端强制执行每分钟请求数（RPM）限制；如需调整 RPM，请联系 AWS 支持。

## 数据保留

此产品的数据处理由 Amazon Bedrock 管理。有关详细信息，请参阅 [Amazon Bedrock 中的数据保护](https://docs.aws.amazon.com/bedrock/latest/userguide/data-protection.html)。

## 监控和日志记录

Amazon Bedrock 中的 Claude 会向 CloudWatch 和 CloudTrail 发送日志。Anthropic 建议至少以 30 天滚动方式保留活动日志，以便了解使用模式并调查潜在问题。

## 支持

如需支持，请联系 AWS 支持或您的 AWS 客户团队。请附上您的 AWS 账户 ID 以及任何失败的 API 响应中的 `request-id`。
