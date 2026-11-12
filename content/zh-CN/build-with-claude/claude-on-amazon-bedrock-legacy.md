---
title: Amazon Bedrock 上的 Claude（bedrock-runtime）
url: https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy
description: 通过 InvokeModel 和 Converse API，在 Amazon Bedrock 的 bedrock-runtime 端点上使用 Claude 模型。
---

<Note>
  本页介绍 Amazon Bedrock 的 `bedrock-runtime` 端点：使用 AWS event-stream 编码的 `InvokeModel` 和 `Converse` API。有关 `bedrock-mantle` 端点上可用的模型，请参阅 [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)，它在 `/anthropic/v1/messages` 使用 Messages API 并采用 SSE 流式传输。如需由 Anthropic 运营、通过 AWS Marketplace 计费且通常可在同日获得新功能的替代方案，请参阅 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)。现有 Bedrock 用户可以参考[迁移指南](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#migrating-from-amazon-bedrock)。
</Note>

<Warning>
  最新版本的 SDK 默认发送 API 版本 `2026-10-21`，包括通过 Amazon Bedrock 发送时。本页模型表中版本号为 4.6 或更低的模型在 Bedrock 上不支持 `2026-10-21`，向这些模型发送该版本的请求可能会返回 400 错误。如果您通过 SDK 调用这些模型，请将 SDK 保持在您当前使用的版本，并在安装 SDK 的位置固定该版本，以免之后的安装将其升级。如果您通过 Boto3 等 AWS SDK 调用 `InvokeModel`，则需要自行在请求体中设置 `anthropic_version`。Anthropic SDK 的新默认值不适用于这些请求。

  要为这些模型安装 SDK，请使用[安装用于访问 Bedrock 的 SDK](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#install-an-sdk-for-accessing-bedrock) 中的命令，这些命令会安装 API 版本 `2026-10-21` 之前的最后一个主版本。
</Warning>

通过 Bedrock 调用 Claude 与直接通过 Claude API 调用 Claude 的方式略有不同。本指南将引导您使用 Anthropic 的某个[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 完成对 Bedrock 上 Claude 的 API 调用。

请注意，本指南假设您已经注册了 [AWS 账户](https://portal.aws.amazon.com/billing/signup)并配置了编程访问。

## 安装和配置 AWS CLI

1. [安装 AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)，版本需为 `2.13.23` 或更高。
2. 使用 AWS configure 命令配置您的 AWS 凭证（请参阅[配置 AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)），或者在 AWS 控制面板中导航到 **Command line or programmatic access**，并按照弹出窗口中的说明查找您的凭证。
3. 验证您的凭证是否有效：

```bash AWS CLI
aws sts get-caller-identity
```

## 安装用于访问 Bedrock 的 SDK

Anthropic 的[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 支持 Bedrock。您也可以直接使用 AWS SDK，例如 `boto3`。

<Tabs>
  <Tab title="Python">
    ```bash
    pip install -U "anthropic[bedrock]<2"
    ```
  </Tab>

  <Tab title="TypeScript">
    ```bash
    npm install @anthropic-ai/bedrock-sdk@0
    ```

    [列出可用模型](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#list-available-models) 中的示例还需要 AWS 的 Bedrock 客户端。使用 `npm install @aws-sdk/client-bedrock` 安装它。
  </Tab>

  <Tab title="C#">
    ```bash
    dotnet add package Anthropic.Bedrock --version 0.14.1
    ```

    如果您的项目还引用了 `Anthropic` 包，请将其保持在版本 13 以下。

    [列出可用模型](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#list-available-models) 中的示例还需要 AWS 的 Bedrock 客户端。使用 `dotnet add package AWSSDK.Bedrock` 安装它。
  </Tab>

  <Tab title="Go">
    ```bash
    go get github.com/anthropics/anthropic-sdk-go/bedrock
    ```

    [列出可用模型](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#list-available-models) 中的示例还需要 AWS 的 Bedrock 客户端。使用 `go get github.com/aws/aws-sdk-go-v2/service/bedrock` 安装它。
  </Tab>

  <Tab title="Java">
    <CodeGroup exclude="shell, python, typescript, csharp, go, php, ruby">
      ```groovy Gradle
      implementation("com.anthropic:anthropic-java:2.+")
      implementation("com.anthropic:anthropic-java-bedrock:2.+")
      ```

      ```xml Maven
      <dependency>
          <groupId>com.anthropic</groupId>
          <artifactId>anthropic-java</artifactId>
          <version>[2.0.0,3.0.0)</version>
      </dependency>
      <dependency>
          <groupId>com.anthropic</groupId>
          <artifactId>anthropic-java-bedrock</artifactId>
          <version>[2.0.0,3.0.0)</version>
      </dependency>
      ```

      ```java Java
      import com.anthropic.client.AnthropicClient;
      import com.anthropic.client.okhttp.AnthropicOkHttpClient;
      import com.anthropic.bedrock.backends.BedrockBackend;
      import com.anthropic.models.messages.MessageCreateParams;
      import com.anthropic.models.messages.Message;
      import com.anthropic.models.messages.Model;

      public class BasicMessage {
          public static void main(String[] args) {
              AnthropicClient client = AnthropicOkHttpClient.builder()
                  .backend(BedrockBackend.fromEnv())
                  .build();

              MessageCreateParams params = MessageCreateParams.builder()
                  .model(Model.CLAUDE_OPUS_4_6)
                  .maxTokens(1024L)
                  .addUserMessage("What is the capital of France?")
                  .build();

              Message response = client.messages().create(params);
              response.content().stream()
                  .flatMap(block -> block.text().stream())
                  .forEach(textBlock -> System.out.println(textBlock.text()));
          }
      }
      ```
    </CodeGroup>

    [列出可用模型](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#list-available-models) 中的示例还需要 AWS 的 Bedrock 客户端。将 `software.amazon.awssdk:bedrock` 添加到您的构建中。
  </Tab>

  <Tab title="PHP">
    ```bash
    composer require "anthropic-ai/sdk:<1.0" aws/aws-sdk-php
    ```
  </Tab>

  <Tab title="Ruby">
    ```bash
    # Gemfile
    gem "anthropic", "~> 1.0"
    gem "aws-sdk-bedrockruntime"
    ```

    [列出可用模型](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#list-available-models) 中的示例还需要 AWS 的 Bedrock 客户端。将 `gem "aws-sdk-bedrock"` 添加到您的 `Gemfile` 中。
  </Tab>

  <Tab title="Boto3 (Python)">
    ```bash
    pip install "boto3>=1.28.59"
    ```
  </Tab>
</Tabs>

## 访问 Bedrock

### 订阅 Anthropic 模型

前往 [AWS Console > Bedrock > 模型访问](https://console.aws.amazon.com/bedrock/home?region=us-west-2#/modelaccess)并申请访问 Anthropic 模型。请注意，Anthropic 模型的可用性因区域而异。有关最新信息，请参阅 [AWS 关于各区域模型可用性的文档](https://docs.aws.amazon.com/bedrock/latest/userguide/models-regions.html)。

#### API 模型 ID

<Note>
  Claude Fable 5.1、Claude Fable 5、Claude Opus 5.5、Claude Opus 5、Claude Sonnet 5.5、Claude Sonnet 5、Claude Haiku 5.5、Claude Opus 4.8 和 Claude Opus 4.7 可以通过 `bedrock-runtime` 上的 `InvokeModel` 访问。 这些请求由与 [Amazon Bedrock 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) 端点相同的基础设施提供服务。如需原生 Messages API 请求格式和完整的功能 对等，请使用该页面。这些模型未列入本页的模型 表。该表确实列出了 Claude Fable 5.2 和 Claude Mythos 5.2，因为 `bedrock-runtime` 是唯一提供这两个模型的 Amazon Bedrock 端点。
</Note>

生命周期术语（Deprecated（已弃用）、Retired（已停用））的定义请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)。合作伙伴运营平台上的生命周期日期由合作伙伴设定，可能与 Claude API 的时间表不同。有关 Amazon Bedrock 上任何模型的当前停用日期，请参阅 [Amazon Bedrock 的模型生命周期页面](https://docs.aws.amazon.com/bedrock/latest/userguide/model-lifecycle.html)。

AWS 通过[跨区域推理](https://docs.aws.amazon.com/bedrock/latest/userguide/cross-region-inference.html)而非按需吞吐量提供较新的 Claude 模型。对于这些模型，传入基础模型 ID 的请求会失败，并返回如下 HTTP 400 错误：

```text wrap
Invocation of model ID anthropic.claude-sonnet-4-5-20250929-v1:0 with on-demand throughput isn't supported. Retry your request with the ID or ARN of an inference profile that contains this model.
```

要调用这些模型，请传入推理配置文件（inference profile）而非基础模型 ID。推理配置文件 ID 是在基础模型 ID 前加上下表中标记为"是"的列所对应的前缀，例如 us.anthropic.claude-sonnet-4-5-20250929-v1:0。您也可以传入完整的推理配置文件 ARN，格式为 `arn:aws:bedrock:{region}:{account-id}:inference-profile/{inference-profile-id}`。有关 AWS 官方提供的可用推理配置文件列表，请参阅[推理配置文件支持的区域和模型](https://docs.aws.amazon.com/bedrock/latest/userguide/inference-profiles-support.html)。要了解前缀如何影响路由和定价，请参阅[全局端点与区域端点](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy#global-vs-regional-endpoints)部分。

| Model                                                                                                    | Base Bedrock model ID                       | `global` | `us` | `eu` | `jp` | `apac` |
| :------------------------------------------------------------------------------------------------------- | :------------------------------------------ | :------- | :--- | :--- | :--- | :----- |
| Claude Fable 5.2                                                                                         | `anthropic.claude-fable-5-2`                | Yes      | Yes  | No   | No   | No     |
| Claude Mythos 5.2 ([limited availability](https://support.claude.com/en/articles/14604842))              | `anthropic.claude-mythos-5-2`               | Yes      | Yes  | No   | No   | No     |
| Claude Opus 4.6                                                                                          | `anthropic.claude-opus-4-6-v1`              | Yes      | Yes  | Yes  | Yes  | Yes    |
| Claude Opus 4.5                                                                                          | `anthropic.claude-opus-4-5-20251101-v1:0`   | Yes      | Yes  | Yes  | No   | No     |
| Claude Opus 4.1 ([deprecated](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))   | `anthropic.claude-opus-4-1-20250805-v1:0`   | No       | Yes  | No   | No   | No     |
| Claude Sonnet 4.6                                                                                        | `anthropic.claude-sonnet-4-6`               | Yes      | Yes  | Yes  | Yes  | No     |
| Claude Sonnet 4.5 ([deprecated](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | `anthropic.claude-sonnet-4-5-20250929-v1:0` | Yes      | Yes  | Yes  | Yes  | No     |
| Claude Sonnet 4 ([deprecated](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))   | `anthropic.claude-sonnet-4-20250514-v1:0`   | Yes      | Yes  | Yes  | No   | Yes    |
| Claude Haiku 4.5                                                                                         | `anthropic.claude-haiku-4-5-20251001-v1:0`  | Yes      | Yes  | Yes  | No   | No     |

### 列出可用模型

以下示例展示如何打印通过 Bedrock 可用的所有 Claude 模型列表：

<CodeGroup exclude="shell">
  ```bash AWS CLI
  aws bedrock list-foundation-models --region=us-west-2 --by-provider anthropic --query "modelSummaries[*].modelId"
  ```

  ```python Boto3 (Python)
  import boto3

  bedrock = boto3.client(service_name="bedrock")
  response = bedrock.list_foundation_models(byProvider="anthropic")

  for summary in response["modelSummaries"]:
      print(summary["modelId"])
  ```

  ```typescript TypeScript
  import { BedrockClient, ListFoundationModelsCommand } from "@aws-sdk/client-bedrock";

  const client = new BedrockClient({ region: "us-west-2" });

  const command = new ListFoundationModelsCommand({ byProvider: "anthropic" });
  const response = await client.send(command);

  if (response.modelSummaries) {
    for (const summary of response.modelSummaries) {
      console.log(summary.modelId);
    }
  }
  ```

  ```csharp C#
  using Amazon;
  using Amazon.Bedrock;
  using Amazon.Bedrock.Model;

  var client = new AmazonBedrockClient(RegionEndpoint.USWest2);

  var request = new ListFoundationModelsRequest
  {
      ByProvider = "anthropic"
  };

  var response = await client.ListFoundationModelsAsync(request);

  foreach (var summary in response.ModelSummaries)
  {
      Console.WriteLine(summary.ModelId);
  }
  ```

  ```go Go
  import (
  	"context"
  	"fmt"
  	"log"

  	"github.com/aws/aws-sdk-go-v2/config"
  	"github.com/aws/aws-sdk-go-v2/service/bedrock"
  )
  // ...
  	cfg, err := config.LoadDefaultConfig(context.TODO(), config.WithRegion("us-west-2"))
  	if err != nil {
  		log.Fatal(err)
  	}

  	client := bedrock.NewFromConfig(cfg)

  	byProvider := "anthropic"
  	response, err := client.ListFoundationModels(context.TODO(), &bedrock.ListFoundationModelsInput{
  		ByProvider: &byProvider,
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	for _, summary := range response.ModelSummaries {
  		fmt.Println(*summary.ModelId)
  	}
  ```

  ```java Java
  import software.amazon.awssdk.regions.Region;
  import software.amazon.awssdk.services.bedrock.BedrockClient;
  import software.amazon.awssdk.services.bedrock.model.ListFoundationModelsRequest;
  import software.amazon.awssdk.services.bedrock.model.ListFoundationModelsResponse;
  import software.amazon.awssdk.services.bedrock.model.FoundationModelSummary;

  public class ListAnthropicModels {
      public static void main(String[] args) {
          BedrockClient client = BedrockClient.builder()
              .region(Region.US_WEST_2)
              .build();

          ListFoundationModelsRequest request = ListFoundationModelsRequest.builder()
              .byProvider("anthropic")
              .build();

          ListFoundationModelsResponse response = client.listFoundationModels(request);

          for (FoundationModelSummary summary : response.modelSummaries()) {
              System.out.println(summary.modelId());
          }

          client.close();
      }
  }
  ```

  ```php PHP
  <?php

  use Aws\Bedrock\BedrockClient;

  $client = new BedrockClient([
      'region' => 'us-west-2',
      'version' => 'latest'
  ]);

  $result = $client->listFoundationModels([
      'byProvider' => 'anthropic'
  ]);

  foreach ($result['modelSummaries'] as $summary) {
      echo $summary['modelId'] . PHP_EOL;
  }
  ```

  ```ruby Ruby
  require "aws-sdk-bedrock"

  client = Aws::Bedrock::Client.new(region: "us-west-2")

  response = client.list_foundation_models({
    by_provider: "anthropic"
  })

  response.model_summaries.each do |summary|
    puts summary.model_id
  end
  ```
</CodeGroup>

### 发送请求

以下示例展示如何在 Bedrock 上使用 Claude 生成文本：

<Tabs>
  <Tab title="cURL">
    <Note>
      使用 AWS 凭证调用 `InvokeModel` API 需要 SigV4 请求签名，其他标签页中的 SDK 示例会自动处理这一点。如需可通过独立 cURL 命令调用的 Bedrock 端点，请参阅[使用 Amazon Bedrock 中的 Claude 发送您的第一个请求](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock#making-your-first-request)。
    </Note>
  </Tab>

  <Tab title="CLI">
    <Note>
      `ant` CLI 不支持 Amazon Bedrock。请改用其中一个 SDK 示例。
    </Note>
  </Tab>

  <Tab title="Python">
    ```python
    from anthropic import AnthropicBedrock

    client = AnthropicBedrock(
        # 可通过提供以下密钥进行身份验证，或使用默认的 AWS 凭证提供程序，例如
        # 使用 ~/.aws/credentials 或 "AWS_SECRET_ACCESS_KEY" 和 "AWS_ACCESS_KEY_ID" 环境变量。
        aws_access_key="<access key>",
        aws_secret_key="<secret key>",
        # 可通过 aws_session_token 使用临时凭证。
        # 详情请参阅 https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html。
        aws_session_token="<session_token>",
        # aws_region 用于更改请求发送到的 AWS 区域。如未提供，SDK 会读取
        # AWS_REGION / AWS_DEFAULT_REGION，然后读取您的 boto3 会话或 AWS 配置文件中配置的区域
        # （包括 ~/.aws/config），如果无法解析出任何区域，则抛出 ValueError。
        aws_region="us-west-2",
    )

    message = client.messages.create(
        model="global.anthropic.claude-opus-4-6-v1",
        max_tokens=256,
        messages=[{"role": "user", "content": "Hello, world"}],
    )
    print(message.content)
    ```
  </Tab>

  <Tab title="TypeScript">
    ```typescript
    import AnthropicBedrock from "@anthropic-ai/bedrock-sdk";

    const client = new AnthropicBedrock({
      // 可通过提供以下密钥进行身份验证，或使用
      // 默认的 AWS 凭证提供程序，例如
      // ~/.aws/credentials 或 "AWS_SECRET_ACCESS_KEY" 和
      // "AWS_ACCESS_KEY_ID" 环境变量。
      awsAccessKey: "<access key>",
      awsSecretKey: "<secret key>",

      // 可通过 awsSessionToken 使用临时凭证。
      // 详情请参阅 https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html。
      awsSessionToken: "<session_token>",

      // awsRegion 用于更改请求发送到的
      // AWS 区域。默认情况下，SDK 读取 AWS_REGION，如果
      // 未设置，则默认为 us-east-1。请注意，
      // SDK 不会从 ~/.aws/config 读取区域。
      awsRegion: "us-west-2"
    });

    const message = await client.messages.create({
      model: "global.anthropic.claude-opus-4-6-v1",
      max_tokens: 256,
      messages: [{ role: "user", content: "Hello, world" }]
    });
    console.log(message);
    ```
  </Tab>

  <Tab title="C#">
    ```csharp
    using Anthropic.Bedrock;
    using Anthropic.Models.Messages;

    AnthropicBedrockClient client = new(
        await AnthropicBedrockCredentialsHelper.FromEnv()
        ?? throw new InvalidOperationException("AWS credentials not configured.")
    );

    var response = await client.Messages.Create(new MessageCreateParams
    {
        Model = "global.anthropic.claude-opus-4-6-v1",
        MaxTokens = 256,
        Messages = [new() { Role = Role.User, Content = "Hello, world" }],
    });

    Console.WriteLine(
        string.Join("", response.Content
            .Select(block => block.Value)
            .OfType<TextBlock>()
            .Select(textBlock => textBlock.Text)));
    ```
  </Tab>

  <Tab title="Go">
    ```go
    import (
    	"context"
    	"fmt"

    	"github.com/anthropics/anthropic-sdk-go"
    	"github.com/anthropics/anthropic-sdk-go/bedrock"
    )
    // ...
    	// 使用默认的 AWS 凭证提供程序链
    	client := anthropic.NewClient(
    		bedrock.WithLoadDefaultConfig(context.Background()),
    	)

    	message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
    		Model:     "global.anthropic.claude-opus-4-6-v1",
    		MaxTokens: 256,
    		Messages: []anthropic.MessageParam{
    			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, world")),
    		},
    	})
    	if err != nil {
    		panic(err)
    	}
    	fmt.Println(message.Content)
    ```
  </Tab>

  <Tab title="Java">
    ```java
    import com.anthropic.bedrock.backends.BedrockBackend;
    import com.anthropic.client.AnthropicClient;
    import com.anthropic.client.okhttp.AnthropicOkHttpClient;
    import com.anthropic.models.messages.Message;
    import com.anthropic.models.messages.MessageCreateParams;

    public class BedrockExample {

        public static void main(String[] args) {
            // 使用默认的 AWS 凭证提供程序链
            AnthropicClient client = AnthropicOkHttpClient.builder()
                .backend(BedrockBackend.fromEnv())
                .build();

            Message message = client
                .messages()
                .create(
                    MessageCreateParams.builder()
                        .model("global.anthropic.claude-opus-4-6-v1")
                        .maxTokens(256)
                        .addUserMessage("Hello, world")
                        .build()
                );

            System.out.println(message.content());
        }
    }
    ```
  </Tab>

  <Tab title="PHP">
    ```php
    <?php

    use Anthropic\Bedrock;

    $client = Bedrock\Client::withCredentials(
        accessKeyId: getenv('AWS_ACCESS_KEY_ID'),
        secretAccessKey: getenv('AWS_SECRET_ACCESS_KEY'),
        region: 'us-west-2',
        securityToken: getenv('AWS_SESSION_TOKEN'),
    );

    $message = $client->messages->create(
        maxTokens: 256,
        messages: [
            ['role' => 'user', 'content' => 'Hello, world']
        ],
        model: 'global.anthropic.claude-opus-4-6-v1',
    );
    echo $message->content[0]->text;
    ```
  </Tab>

  <Tab title="Ruby">
    ```ruby
    require "anthropic"

    client = Anthropic::BedrockClient.new

    message = client.messages.create(
      model: "global.anthropic.claude-opus-4-6-v1",
      max_tokens: 256,
      messages: [{role: "user", content: "Hello, world"}]
    )

    puts message.content.first.text
    ```
  </Tab>

  <Tab title="Boto3 (Python)">
    ```python
    import boto3
    import json

    bedrock = boto3.client(service_name="bedrock-runtime")
    body = json.dumps(
        {
            "max_tokens": 256,
            "messages": [{"role": "user", "content": "Hello, world"}],
            "anthropic_version": "bedrock-2023-05-31",
        }
    )

    response = bedrock.invoke_model(
        body=body, modelId="global.anthropic.claude-opus-4-6-v1"
    )

    response_body = json.loads(response.get("body").read())
    print(response_body.get("content"))
    ```
  </Tab>
</Tabs>

更多详情请参阅[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)以及 [Bedrock 官方文档](https://docs.aws.amazon.com/bedrock/)。

### Bearer 令牌身份验证

您可以使用 bearer token（Bearer 令牌）而非 AWS 凭证向 Bedrock 进行身份验证。这在企业环境中非常有用，因为团队需要访问 Bedrock，但无需管理 AWS 凭证、IAM 角色或账户级权限。

最简单的方法是设置 `AWS_BEARER_TOKEN_BEDROCK` 环境变量，SDK 在从环境中解析凭证时会自动检测该变量。

以编程方式提供令牌：

<Tabs>
  <Tab title="cURL">
    <Note>
      本节展示如何在 SDK 客户端中配置 Bearer 令牌，SDK 客户端也会从 `AWS_BEARER_TOKEN_BEDROCK` 环境变量中读取令牌。如需使用 Bearer 令牌直接发送 HTTP 请求，请参阅 [Amazon Bedrock 文档](https://docs.aws.amazon.com/bedrock/)。
    </Note>
  </Tab>

  <Tab title="CLI">
    <Note>
      `ant` CLI 不支持 Amazon Bedrock。请改用其中一个 SDK 示例。
    </Note>
  </Tab>

  <Tab title="Python">
    ```python
    from anthropic import AnthropicBedrock

    client = AnthropicBedrock(
        api_key="your-bearer-token",
        aws_region="us-west-2",
    )

    message = client.messages.create(
        model="us.anthropic.claude-sonnet-4-5-20250929-v1:0",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Hello!"}],
    )
    print(message.content)
    ```
  </Tab>

  <Tab title="TypeScript">
    ```typescript
    import AnthropicBedrock from "@anthropic-ai/bedrock-sdk";

    const client = new AnthropicBedrock({
      apiKey: "your-bearer-token",
      awsRegion: "us-west-2"
    });

    const message = await client.messages.create({
      model: "us.anthropic.claude-sonnet-4-5-20250929-v1:0",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello!" }]
    });
    console.log(message);
    ```
  </Tab>

  <Tab title="C#">
    ```csharp
    using Anthropic.Bedrock;
    using Anthropic.Models.Messages;

    var client = new AnthropicBedrockClient(
        new AnthropicBedrockApiTokenCredentials
        {
            BearerToken = "your-bearer-token",
            Region = "us-west-2",
        }
    );

    var response = await client.Messages.Create(new MessageCreateParams
    {
        Model = "us.anthropic.claude-sonnet-4-5-20250929-v1:0",
        MaxTokens = 1024,
        Messages = [new() { Role = Role.User, Content = "Hello!" }],
    });
    ```
  </Tab>

  <Tab title="Go">
    ```go
    import (
    	"context"
    	"fmt"

    	"github.com/anthropics/anthropic-sdk-go"
    	"github.com/anthropics/anthropic-sdk-go/bedrock"
    	"github.com/aws/aws-sdk-go-v2/aws"
    )
    // ...
    	cfg := aws.Config{
    		Region:                  "us-west-2",
    		BearerAuthTokenProvider: bedrock.NewStaticBearerTokenProvider("your-bearer-token"),
    	}
    	client := anthropic.NewClient(
    		bedrock.WithConfig(cfg),
    	)

    	message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    		Model:     "us.anthropic.claude-sonnet-4-5-20250929-v1:0",
    		MaxTokens: 1024,
    		Messages: []anthropic.MessageParam{
    			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello!")),
    		},
    	})
    	if err != nil {
    		panic(err)
    	}
    	fmt.Println(message.Content[0].Text)
    ```
  </Tab>

  <Tab title="Java">
    ```java
    import com.anthropic.bedrock.backends.BedrockBackend;
    import com.anthropic.client.AnthropicClient;
    import com.anthropic.client.okhttp.AnthropicOkHttpClient;
    import com.anthropic.models.messages.MessageCreateParams;
    import software.amazon.awssdk.regions.Region;

    // 选项 1：设置 AWS_BEARER_TOKEN_BEDROCK 环境变量并使用 fromEnv()
    AnthropicClient client = AnthropicOkHttpClient.builder()
        .backend(BedrockBackend.fromEnv())
        .build();

    // 选项 2：以编程方式提供令牌
    client = AnthropicOkHttpClient.builder()
        .backend(BedrockBackend.builder()
            .apiKey("your-bearer-token")
            .region(Region.US_WEST_2)
            .build())
        .build();

    MessageCreateParams params = MessageCreateParams.builder()
        .model("us.anthropic.claude-sonnet-4-5-20250929-v1:0")
        .maxTokens(1024)
        .addUserMessage("Hello!")
        .build();

    client.messages().create(params).content().stream()
        .flatMap(block -> block.text().stream())
        .forEach(textBlock -> System.out.println(textBlock.text()));
    ```
  </Tab>

  <Tab title="PHP">
    ```php
    <?php

    use Anthropic\Bedrock;

    $client = Bedrock\Client::withApiKey('your-bearer-token', 'us-west-2');

    $message = $client->messages->create(
        maxTokens: 1024,
        messages: [
            ['role' => 'user', 'content' => 'Hello!']
        ],
        model: 'us.anthropic.claude-sonnet-4-5-20250929-v1:0',
    );
    echo $message->content[0]->text;
    ```
  </Tab>

  <Tab title="Ruby">
    ```ruby
    require "anthropic"

    client = Anthropic::BedrockClient.new(
      api_key: "your-bearer-token",
      aws_region: "us-west-2"
    )

    message = client.messages.create(
      model: "us.anthropic.claude-sonnet-4-5-20250929-v1:0",
      max_tokens: 1024,
      messages: [{role: "user", content: "Hello!"}]
    )
    puts message.content.first.text
    ```
  </Tab>
</Tabs>

## 活动日志记录

Bedrock 提供[调用日志记录服务](https://docs.aws.amazon.com/bedrock/latest/userguide/model-invocation-logging.html)，允许您记录与您的使用相关的提示和补全内容。

Anthropic 建议您至少以 30 天滚动周期记录您的活动，以便了解您的活动并调查任何潜在的滥用行为。

<Note>
  启用此服务不会让 AWS 或 Anthropic 获得对您内容的任何访问权限。
</Note>

## 功能支持

有关包含 Amazon Bedrock 可用性的完整功能列表，请参阅[功能概览](https://platform.claude.com/docs/zh-CN/build-with-claude/overview)。

### 支持的主要功能

* [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create)
* [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)
* [思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)
* [工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)，包括 [Bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)、[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)、[记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)和[文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)
* [引用](https://platform.claude.com/docs/zh-CN/build-with-claude/citations)
* [结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)

### 不支持的功能

* 输入源（图像和文档的 URL 源、Files API）
* 服务器端工具（代码执行、网络搜索、网页获取、advisor）
* Agent 基础设施（Agent Skills、MCP 连接器、programmatic tool calling（程序化工具调用））
* API 端点（Message Batches、Models、Admin、Compliance、用量和成本）
* Claude Managed Agents
* 服务器端回退（[`fallbacks` 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)；请改用[客户端回退模式](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#client-side-fallback)）
* 自动提示缓存（[顶层 `cache_control` 字段](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#automatic-caching)；请改用[显式缓存断点](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#explicit-cache-breakpoints)）

### Bedrock 上的计算机使用和浏览器使用工具集

[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)和[浏览器使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)工具集可通过 InvokeModel API 用于以下模型：

* Claude Fable 5.2
* Claude Mythos 5.2
* Claude Fable 5.1
* Claude Fable 5
* Claude Opus 5.5
* Claude Opus 5
* Claude Opus 4.8
* Claude Sonnet 5.5
* Claude Sonnet 5
* Claude Haiku 5.5

要使用其中之一，请向 `tools` 数组添加一个类型为 `computer_toolset_20260801` 或 `browser_toolset_20260801` 的条目。这两个工具集都不需要 beta 标头。

### Bedrock 上的 PDF 支持

[PDF 支持](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support)可通过 Converse API 和 InvokeModel API 在 Bedrock 上使用。

### Bedrock 上的对话中系统消息

[对话中系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)可通过 InvokeModel API 用于 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Fable 5、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8、Claude Sonnet 5.5 和 Claude Haiku 5.5。此功能在 Claude Sonnet 5 上不可用，请改用顶层 `system` 字段。对于本页模型表中的其他模型，此功能也不可用。

在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1 和 Claude Opus 5.5 上，`role: "system"` 消息还可以设置 `output_config.effort` 以[在对话中更改 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)。此功能处于 beta 阶段：请将 `mid-conversation-output-config-2026-07-01` 添加到请求体中的 `anthropic_beta` 数组。如果未添加该值，或在 Claude Fable 5、Claude Opus 5 或 Claude Opus 4.8 上使用，请求将返回 400 错误：`messages.N.output_config: Extra inputs are not permitted`。错误中的 `N` 是该 `system` 消息在 `messages` 中的索引。

**对于 Converse API 用户：** Converse API 通过其顶层 [`system` 参数](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html)接受系统指令。如需在对话中添加系统指令，请使用 InvokeModel API。

### 上下文窗口

Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Fable 5、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8、Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 5.5、Claude Sonnet 5、Claude Sonnet 4.6 和 Claude Haiku 5.5 在 Amazon Bedrock 上拥有 [1M 令牌的上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)。其他 Claude 模型，包括 Sonnet 4.5（已弃用）和 Sonnet 4（已弃用），拥有 200k 令牌的上下文窗口。

Bedrock 将请求负载限制为 20 MB。在发送大型文档或大量图像时，您可能会在达到令牌限制之前就达到此限制。

## 全局端点与区域端点

从 **Claude Sonnet 4.5 及所有未来模型**开始，Bedrock 提供两种端点类型：

* **全局端点：** 动态路由以实现最大可用性
* **区域端点：** 保证数据通过特定地理区域路由

区域端点的价格比全局端点高出 10%。

<Note>
  这仅适用于 Claude Sonnet 4.5（已弃用）及未来模型。较旧的模型（Claude Sonnet 4（已弃用）及更早版本）保持其现有的定价结构。
</Note>

### 何时使用各选项

**全局端点（推荐）：**

* 提供最大的可用性和正常运行时间
* 将请求动态路由到有可用容量的区域
* 无价格溢价
* 最适合数据驻留要求灵活的应用程序

**区域端点（CRIS）：**

* 通过特定地理区域路由流量
* 满足数据驻留和合规要求时必须使用
* 可用于美国、欧盟、日本和亚太地区
* 有 10% 的价格溢价，反映了专用区域容量的基础设施成本

### 实现

**使用全局端点（Opus 4.6、Sonnet 4.6 和 Sonnet 4.5（已弃用）的默认设置）：**

Claude Opus 4.6、Sonnet 4.6 和 Sonnet 4.5（已弃用）的模型 ID 已包含 `global.` 前缀：

<Tabs>
  <Tab title="cURL">
    <Note>
      使用 AWS 凭证调用 `InvokeModel` API 需要 SigV4 请求签名，其他标签页中的 SDK 示例会自动处理这一点。如需可通过独立 cURL 命令调用的 Bedrock 端点，请参阅[使用 Amazon Bedrock 中的 Claude 发送您的第一个请求](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock#making-your-first-request)。
    </Note>
  </Tab>

  <Tab title="CLI">
    <Note>
      `ant` CLI 不支持 Amazon Bedrock。请改用其中一个 SDK 示例。
    </Note>
  </Tab>

  <Tab title="Python">
    ```python
    from anthropic import AnthropicBedrock

    client = AnthropicBedrock(aws_region="us-west-2")

    message = client.messages.create(
        model="global.anthropic.claude-opus-4-6-v1",
        max_tokens=256,
        messages=[{"role": "user", "content": "Hello, world"}],
    )
    ```
  </Tab>

  <Tab title="TypeScript">
    ```typescript
    import AnthropicBedrock from "@anthropic-ai/bedrock-sdk";

    const client = new AnthropicBedrock({
      awsRegion: "us-west-2"
    });

    const message = await client.messages.create({
      model: "global.anthropic.claude-opus-4-6-v1",
      max_tokens: 256,
      messages: [{ role: "user", content: "Hello, world" }]
    });
    ```
  </Tab>

  <Tab title="C#">
    ```csharp
    using Anthropic.Bedrock;
    using Anthropic.Models.Messages;

    // C# Bedrock 客户端使用带区域前缀的模型 ID 进行全局路由
    AnthropicBedrockClient client = new(
        await AnthropicBedrockCredentialsHelper.FromEnv()
        ?? throw new InvalidOperationException("AWS credentials not configured.")
    );

    var response = await client.Messages.Create(new MessageCreateParams
    {
        // 使用 "global." 前缀进行全局跨区域推理
        Model = "global.anthropic.claude-opus-4-6-v1",
        MaxTokens = 256,
        Messages = [new() { Role = Role.User, Content = "Hello, world" }],
    });
    ```
  </Tab>

  <Tab title="Go">
    ```go
    import (
    	"context"

    	"github.com/anthropics/anthropic-sdk-go"
    	"github.com/anthropics/anthropic-sdk-go/bedrock"
    )
    // ...
    	// 使用默认的 AWS 凭证提供程序链
    	client := anthropic.NewClient(
    		bedrock.WithLoadDefaultConfig(context.Background()),
    	)

    	message, _ := client.Messages.New(context.Background(), anthropic.MessageNewParams{
    		Model:     "global.anthropic.claude-opus-4-6-v1",
    		MaxTokens: 256,
    		Messages: []anthropic.MessageParam{
    			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, world")),
    		},
    	})
    ```
  </Tab>

  <Tab title="Java">
    ```java
    import com.anthropic.bedrock.backends.BedrockBackend;
    import com.anthropic.client.AnthropicClient;
    import com.anthropic.client.okhttp.AnthropicOkHttpClient;
    import com.anthropic.models.messages.MessageCreateParams;

    // 使用默认的 AWS 凭证提供程序链
    AnthropicClient client = AnthropicOkHttpClient.builder()
        .backend(BedrockBackend.fromEnv())
        .build();

    var message = client
        .messages()
        .create(
            MessageCreateParams.builder()
                .model("global.anthropic.claude-opus-4-6-v1")
                .maxTokens(256)
                .addUserMessage("Hello, world")
                .build()
        );
    ```
  </Tab>

  <Tab title="PHP">
    ```php
    <?php

    use Anthropic\Bedrock;

    $client = Bedrock\Client::fromEnvironment();

    $message = $client->messages->create(
        maxTokens: 256,
        messages: [
            ['role' => 'user', 'content' => 'Hello, world']
        ],
        model: 'global.anthropic.claude-opus-4-6-v1',
    );
    ```
  </Tab>

  <Tab title="Ruby">
    ```ruby
    require "anthropic"

    # 默认凭证从 AWS_REGION 环境变量解析区域
    client = Anthropic::BedrockClient.new

    message = client.messages.create(
      # 使用 "global." 前缀进行全局跨区域推理
      model: "global.anthropic.claude-opus-4-6-v1",
      max_tokens: 256,
      messages: [{role: "user", content: "Hello, world"}]
    )
    ```
  </Tab>
</Tabs>

**使用区域端点（CRIS）：**

要使用区域端点，请将 `global.` 前缀替换为区域前缀，例如 `us.`：

<Tabs>
  <Tab title="cURL">
    <Note>
      使用 AWS 凭证调用 `InvokeModel` API 需要 SigV4 请求签名，其他标签页中的 SDK 示例会自动处理这一点。如需可通过独立 cURL 命令调用的 Bedrock 端点，请参阅[使用 Amazon Bedrock 中的 Claude 发送您的第一个请求](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock#making-your-first-request)。
    </Note>
  </Tab>

  <Tab title="CLI">
    <Note>
      `ant` CLI 不支持 Amazon Bedrock。请改用其中一个 SDK 示例。
    </Note>
  </Tab>

  <Tab title="Python">
    ```python
    from anthropic import AnthropicBedrock

    client = AnthropicBedrock(aws_region="us-west-2")

    # 使用美国区域端点（CRIS）
    message = client.messages.create(
        model="us.anthropic.claude-opus-4-6-v1",  # Regional prefix
        max_tokens=256,
        messages=[{"role": "user", "content": "Hello, world"}],
    )
    ```
  </Tab>

  <Tab title="TypeScript">
    ```typescript
    import AnthropicBedrock from "@anthropic-ai/bedrock-sdk";

    const client = new AnthropicBedrock({
      awsRegion: "us-west-2"
    });

    // 使用美国区域端点（CRIS）
    const message = await client.messages.create({
      model: "us.anthropic.claude-opus-4-6-v1", // Regional prefix
      max_tokens: 256,
      messages: [{ role: "user", content: "Hello, world" }]
    });
    ```
  </Tab>

  <Tab title="C#">
    ```csharp
    using Anthropic.Bedrock;
    using Anthropic.Models.Messages;

    // 默认凭证从 AWS_REGION 环境变量解析区域
    AnthropicBedrockClient client = new(
        await AnthropicBedrockCredentialsHelper.FromEnv()
        ?? throw new InvalidOperationException("AWS credentials not configured.")
    );

    // 使用美国区域端点（CRIS）
    var response = await client.Messages.Create(new MessageCreateParams
    {
        Model = "us.anthropic.claude-opus-4-6-v1", // Regional prefix
        MaxTokens = 256,
        Messages = [new() { Role = Role.User, Content = "Hello, world" }],
    });
    ```
  </Tab>

  <Tab title="Go">
    ```go
    import (
    	"context"

    	"github.com/anthropics/anthropic-sdk-go"
    	"github.com/anthropics/anthropic-sdk-go/bedrock"
    )
    // ...
    	// 使用默认的 AWS 凭证提供程序链
    	client := anthropic.NewClient(
    		bedrock.WithLoadDefaultConfig(context.Background()),
    	)

    	// 使用美国区域端点（CRIS）
    	message, _ := client.Messages.New(context.Background(), anthropic.MessageNewParams{
    		Model:     "us.anthropic.claude-opus-4-6-v1", // Regional prefix
    		MaxTokens: 256,
    		Messages: []anthropic.MessageParam{
    			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, world")),
    		},
    	})
    ```
  </Tab>

  <Tab title="Java">
    ```java
    import com.anthropic.bedrock.backends.BedrockBackend;
    import com.anthropic.client.AnthropicClient;
    import com.anthropic.client.okhttp.AnthropicOkHttpClient;
    import com.anthropic.models.messages.MessageCreateParams;

    // 使用默认的 AWS 凭证提供程序链
    AnthropicClient client = AnthropicOkHttpClient.builder()
        .backend(BedrockBackend.fromEnv())
        .build();

    // 使用美国区域端点（CRIS）
    var message = client
        .messages()
        .create(
            MessageCreateParams.builder()
                .model("us.anthropic.claude-opus-4-6-v1") // Regional prefix
                .maxTokens(256)
                .addUserMessage("Hello, world")
                .build()
        );
    ```
  </Tab>

  <Tab title="PHP">
    ```php
    <?php

    use Anthropic\Bedrock;

    $client = Bedrock\Client::fromEnvironment();

    $message = $client->messages->create(
        maxTokens: 256,
        messages: [
            ['role' => 'user', 'content' => 'Hello, world']
        ],
        model: 'us.anthropic.claude-opus-4-6-v1',
    );
    ```
  </Tab>

  <Tab title="Ruby">
    ```ruby
    require "anthropic"

    # 使用美国区域端点 (CRIS)
    client = Anthropic::BedrockClient.new(aws_region: "us-west-2")

    message = client.messages.create(
      model: "us.anthropic.claude-opus-4-6-v1", # Regional prefix
      max_tokens: 256,
      messages: [{role: "user", content: "Hello, world"}]
    )
    ```
  </Tab>
</Tabs>

## 其他资源

* **Bedrock 定价：** [Amazon Bedrock 定价页面](https://aws.amazon.com/bedrock/pricing/)
* **AWS 定价文档：** [Bedrock 定价指南](https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-pricing.html)
* **AWS 博客文章：** [在 Amazon Bedrock 中推出 Claude Sonnet 4.5](https://aws.amazon.com/blogs/aws/introducing-claude-sonnet-4-5-in-amazon-bedrock-anthropics-most-intelligent-model-best-for-coding-and-complex-agents/)
* **Anthropic 定价详情：** [云平台定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#cloud-platform-pricing)
