---
title: Claude Models on Google Cloud (Google managed)
url: https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed
description: Google 将 Claude Models on Google Cloud (Google managed) 作为 Google Cloud's Agent Platform 的一部分进行运营。您可以使用 Anthropic 的客户端 SDK 调用 Claude。
---

[Claude Models on Google Cloud](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/overview) 对比了这两种 Google Cloud 产品，并说明了如何判断您使用的是哪一种。

## 进行 API 调用之前

要使用 Claude Models on Google Cloud (Google managed)，您需要：

1. 一个可以使用 Agent Platform 的 Google Cloud 项目。有关设置步骤，请参阅 Google Cloud 的 [Agent Platform 上的 Anthropic Claude 模型](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/claude)页面。
2. 已安装 [gcloud CLI](https://cloud.google.com/sdk/docs/install)，用于本地开发。
3. 对于发起调用的身份，需要一个包含 `aiplatform.endpoints.predict` 权限的 IAM 角色，例如 Agent Platform User（`roles/aiplatform.user`）。有关详细信息，请参阅 Google Cloud 的 [Agent Platform 合作伙伴模型](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/use-partner-models)页面。
4. 该身份的 Google Cloud 凭据。要让 SDK 向 Google Cloud 进行身份验证，您可能需要运行 `gcloud auth application-default login`。发往 Claude Models on Google Cloud (Google managed) 的 cURL 请求可以通过 `gcloud auth print-access-token` 获取令牌。该命令使用 gcloud CLI 自身的登录，而不是 Application Default Credentials，因此请在使用它之前先运行 `gcloud auth login` 或 `gcloud init`。

### 安装 SDK

[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 通过 `AnthropicVertex` (csharp: `AnthropicVertexClient`; go: `vertex.WithGoogleAuth()`; java: `VertexBackend`; php: `Anthropic\Vertex\Client`; ruby: `Anthropic::VertexClient`) 支持 Claude Models on Google Cloud (Google managed)。请安装提供它的软件包：

<Tabs>
  <Tab title="Python">
    ```bash
    pip install -U "anthropic[vertex]"
    ```
  </Tab>

  <Tab title="TypeScript">
    ```bash
    npm install @anthropic-ai/vertex-sdk
    ```
  </Tab>

  <Tab title="C#">
    ```bash
    dotnet add package Anthropic.Vertex
    ```
  </Tab>

  <Tab title="Go">
    ```bash
    go get github.com/anthropics/anthropic-sdk-go/v2
    go get github.com/anthropics/anthropic-sdk-go/v2/vertex
    ```
  </Tab>

  <Tab title="Java">
    <Tabs>
      <Tab title="Gradle">
        ```kotlin
        implementation("com.anthropic:anthropic-java:3.8.0")
        implementation("com.anthropic:anthropic-java-vertex:3.8.0")
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
            <artifactId>anthropic-java-vertex</artifactId>
            <version>3.8.0</version>
        </dependency>
        ```
      </Tab>
    </Tabs>
  </Tab>

  <Tab title="PHP">
    ```bash
    composer require anthropic-ai/sdk google/auth
    ```
  </Tab>

  <Tab title="Ruby">
    ```bash
    # Gemfile
    gem "anthropic"
    gem "googleauth"
    ```
  </Tab>
</Tabs>

`ant` CLI 不支持 Claude Models on Google Cloud (Google managed)。

## 发起请求

发往 Claude Models on Google Cloud (Google managed) 的请求会发送到以 `:rawPredict` 结尾的 Agent Platform 端点 URL。"Streaming"（流式传输）请求则改用 `:streamRawPredict`。

请求与 [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create) 请求几乎完全相同，主要有两点区别：

* **模型：** 端点 URL 中指定了模型。在 cURL 请求中，请勿在请求体中包含 `model`。
* **版本：** 在 cURL 请求中，请在请求体中传递 `anthropic_version`，而不是作为请求头传递。将其设置为 `vertex-2023-10-16`，或设置为 [API 版本 `2026-10-21`](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21)。

以下示例使用全局端点。有关其他端点类型，请参阅[全局、多区域和区域端点](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#global-multi-region-and-regional-endpoints)。

<CodeGroup exclude="shell:CLI">
  ```bash cURL
  curl https://aiplatform.googleapis.com/v1/projects/MY_PROJECT_ID/locations/global/publishers/anthropic/models/claude-opus-5-5:rawPredict \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json" \
    -d '{
      "anthropic_version": "2026-10-21",
      "messages": [{"role": "user", "content": "Hey Claude!"}],
      "max_tokens": 100
    }'
  ```

  <CodeGroupItem language="CLI">
    `ant` CLI 不支持 Claude Models on Google Cloud (Google managed)。
  </CodeGroupItem>

  ```python Python
  from anthropic import AnthropicVertex

  project_id = "MY_PROJECT_ID"
  region = "global"

  client = AnthropicVertex(project_id=project_id, region=region)

  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=100,
      messages=[
          {
              "role": "user",
              "content": "Hey Claude!",
          }
      ],
  )
  print(message)
  ```

  ```typescript TypeScript
  import { AnthropicVertex } from "@anthropic-ai/vertex-sdk";

  const projectId = "MY_PROJECT_ID";
  const region = "global";

  // 走标准的 `google-auth-library` 认证流程。
  const client = new AnthropicVertex({
    projectId,
    region
  });

  const result = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 100,
    messages: [
      {
        role: "user",
        content: "Hey Claude!"
      }
    ]
  });
  console.log(JSON.stringify(result, null, 2));
  ```

  ```csharp C#
  using Anthropic.Models.Messages;
  using Anthropic.Vertex;

  var projectId = "MY_PROJECT_ID";
  var region = "global";

  var client = new AnthropicVertexClient(new AnthropicVertexCredentials(region, projectId));

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 100,
      Messages = [new UserMessageParam { Content = "Hey Claude!" }]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  import (
  	"context"
  	"fmt"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  	"github.com/anthropics/anthropic-sdk-go/v2/vertex"
  )
  // ...
  	// 使用默认的 Google Cloud 凭据
  	client := anthropic.NewClient(
  		vertex.WithGoogleAuth(context.Background(), "global", "MY_PROJECT_ID"),
  	)

  	message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 100,
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Hey Claude!")),
  		},
  	})
  	if err != nil {
  		panic(err)
  	}
  	fmt.Printf("%+v\n", message)
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;
  import com.anthropic.vertex.backends.VertexBackend;
  import com.google.auth.oauth2.GoogleCredentials;

  void main() throws Exception {
      AnthropicClient client = AnthropicOkHttpClient.builder()
          .backend(
              VertexBackend.builder()
                  .googleCredentials(GoogleCredentials.getApplicationDefault())
                  .region("global")
                  .project("MY_PROJECT_ID")
                  .build()
          )
          .build();

      Message message = client
          .messages()
          .create(
              MessageCreateParams.builder()
                  .model(Model.CLAUDE_OPUS_5_5)
                  .maxTokens(100)
                  .addUserMessage("Hey Claude!")
                  .build()
          );

      IO.println(message);
  }
  ```

  ```php PHP
  use Anthropic\Messages\Model;
  use Anthropic\Messages\TextBlock;
  use Anthropic\Vertex;

  $client = Vertex\Client::fromEnvironment(
      region: 'global',
      projectId: 'MY_PROJECT_ID',
  );

  $message = $client->messages->create(
      maxTokens: 100,
      messages: [
          ['role' => 'user', 'content' => 'Hey Claude!']
      ],
      model: Model::CLAUDE_OPUS_5_5,
  );

  foreach ($message->content as $block) {
      if ($block instanceof TextBlock) {
          echo $block->text . PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::VertexClient.new(
    region: "global",
    project_id: "MY_PROJECT_ID"
  )

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 100,
    messages: [{ role: "user", content: "Hey Claude!" }]
  )

  puts message.content.find { |block| block.is_a?(Anthropic::Models::TextBlock) }.text
  ```
</CodeGroup>

### API 模型 ID

| Model                                                                                                    | Agent Platform API model ID  |
| :------------------------------------------------------------------------------------------------------- | :--------------------------- |
| Claude Fable 5.2                                                                                         | `claude-fable-5-2`           |
| Claude Mythos 5.2 ([limited availability](https://support.claude.com/en/articles/14604842))              | `claude-mythos-5-2`          |
| Claude Fable 5.1                                                                                         | `claude-fable-5-1`           |
| Claude Mythos 5.1 ([limited availability](https://support.claude.com/en/articles/14604842))              | `claude-mythos-5-1`          |
| Claude Fable 5                                                                                           | `claude-fable-5`             |
| Claude Mythos 5 ([limited availability](https://support.claude.com/en/articles/14604842))                | `claude-mythos-5`            |
| Claude Opus 5.5                                                                                          | `claude-opus-5-5`            |
| Claude Opus 5                                                                                            | `claude-opus-5`              |
| Claude Opus 4.8                                                                                          | `claude-opus-4-8`            |
| Claude Opus 4.7                                                                                          | `claude-opus-4-7`            |
| Claude Opus 4.6                                                                                          | `claude-opus-4-6`            |
| Claude Opus 4.5                                                                                          | `claude-opus-4-5@20251101`   |
| Claude Opus 4.1 ([deprecated](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))   | `claude-opus-4-1@20250805`   |
| Claude Opus 4 ([deprecated](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))     | `claude-opus-4@20250514`     |
| Claude Sonnet 5.5                                                                                        | `claude-sonnet-5-5`          |
| Claude Sonnet 5                                                                                          | `claude-sonnet-5`            |
| Claude Sonnet 4.6                                                                                        | `claude-sonnet-4-6`          |
| Claude Sonnet 4.5 ([deprecated](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)) | `claude-sonnet-4-5@20250929` |
| Claude Sonnet 4 ([deprecated](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations))   | `claude-sonnet-4@20250514`   |
| Claude Haiku 5.5                                                                                         | `claude-haiku-5-5`           |
| Claude Haiku 4.5                                                                                         | `claude-haiku-4-5@20251001`  |

Claude Models on Google Cloud (Google managed) 在不同区域提供不同的 Claude 模型。要查看某个区域提供哪些模型，请在 [Model Garden](https://cloud.google.com/model-garden) 中搜索"Claude"，或参阅 Google Cloud 的 [Agent Platform 上的 Anthropic Claude 模型](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/claude)页面。

Google 为 Claude Models on Google Cloud (Google managed) 所提供的模型设定生命周期日期。这些日期可能与 Claude API 的时间表不同。有关模型当前的停用日期，请参阅 Google Cloud 的 [Agent Platform 上的 Anthropic Claude 模型](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/claude)页面。有关"已弃用"和"已停用"的含义，请参阅[模型弃用](https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations)。

<Note>
  Claude Mythos Preview 是一项研究预览。Claude Models on Google Cloud (Google managed) 向受邀客户提供该模型。有关更多信息，请参阅 [Project Glasswing](https://anthropic.com/glasswing)。
</Note>

让 Claude Code 将您的代码迁移到更新的模型

```text
/claude-api migrate 将此项目迁移到最新的 Claude Opus 模型
```

## 全局、多区域和区域端点

Claude Models on Google Cloud (Google managed) 提供三种端点类型。您可以通过创建客户端时传入的区域值来选择其中一种。在 cURL 请求中，端点类型决定了主机以及 URL 中的 `locations` 段。

| 端点类型       | 区域值                  | 主机                                                                         |
| ---------- | -------------------- | -------------------------------------------------------------------------- |
| **全局**（推荐） | `"global"`           | `aiplatform.googleapis.com`                                                |
| **多区域**    | `"us"` 或 `"eu"`      | `aiplatform.us.rep.googleapis.com`<br />`aiplatform.eu.rep.googleapis.com` |
| **区域**     | 某个区域，例如 `"us-east5"` | `{region}-aiplatform.googleapis.com`                                       |

这些端点类型的区别在于请求的发送目的地：

* **全局：** 请求会发送到任何有可用容量的区域。当您希望获得最高可用性且对数据驻留要求灵活时，请使用此类型。
* **多区域：** 请求会发送到美国（`"us"`）或欧盟（`"eu"`）内的任何区域。当您需要在较大地理范围内实现数据驻留，并且需要比单一区域更高的可用性时，请使用此类型。
* **区域：** 请求仅发送到您指定的区域，例如 `"us-east5"` 或 `"europe-west1"`。当您需要单一区域的数据驻留或必须满足严格的合规要求时，请使用此类型。

多区域和区域端点的费用比全球端点高 10%。三种端点类型都支持按需付费流量，而预配吞吐量需要使用区域端点。

区域端点仅提供 Claude Sonnet 4.6 及更早的模型。对于更新的模型，请使用全局或多区域端点。

<Note>
  多区域和区域端点的溢价仅适用于 Claude Sonnet 4.5（已弃用）及更高版本的模型。有关保留原有定价的模型，请参阅[云平台定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#cloud-platform-pricing)。对于区域端点，溢价反映了专用区域容量的基础设施成本。
</Note>

以下示例使用 `us` 多区域端点。高亮的行用于设置区域。有关在全局端点上的完整请求，请参阅[发起请求](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#accessing-vertex-ai)。

<CodeGroup exclude="shell:CLI">
  ```bash cURL
  # 多区域标识符（"us" 或 "eu"）同时出现在主机名和路径中
  curl https://aiplatform.us.rep.googleapis.com/v1/projects/MY_PROJECT_ID/locations/us/publishers/anthropic/models/claude-opus-5-5:rawPredict \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json" \
    -d '{
      "anthropic_version": "2026-10-21",
      "messages": [{"role": "user", "content": "Hey Claude!"}],
      "max_tokens": 100
    }'
  ```

  <CodeGroupItem language="CLI">
    `ant` CLI 不支持 Claude Models on Google Cloud (Google managed)。
  </CodeGroupItem>

  ```python Python
  from anthropic import AnthropicVertex

  project_id = "MY_PROJECT_ID"
  region = "us"  # Multi-region identifier: "us" or "eu"

  client = AnthropicVertex(project_id=project_id, region=region)
  ```

  ```typescript TypeScript
  import { AnthropicVertex } from "@anthropic-ai/vertex-sdk";

  const projectId = "MY_PROJECT_ID";
  const region = "us"; // Multi-region identifier: "us" or "eu"

  const client = new AnthropicVertex({
    projectId,
    region
  });
  ```

  ```csharp C#
  using Anthropic.Vertex;

  var projectId = "MY_PROJECT_ID";
  var region = "us"; // Multi-region identifier: "us" or "eu"

  var client = new AnthropicVertexClient(new AnthropicVertexCredentials(region, projectId));
  ```

  ```go Go
  import (
  	"context"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  	"github.com/anthropics/anthropic-sdk-go/v2/vertex"
  )
  // ...
  	// 多区域标识符："us" 或 "eu"
  	client := anthropic.NewClient(
  		vertex.WithGoogleAuth(context.Background(), "us", "MY_PROJECT_ID"),
  	)
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.vertex.backends.VertexBackend;
  import com.google.auth.oauth2.GoogleCredentials;

  void main() throws Exception {
      // 多区域标识符："us" 或 "eu"
      AnthropicClient client = AnthropicOkHttpClient.builder()
          .backend(
              VertexBackend.builder()
                  .googleCredentials(GoogleCredentials.getApplicationDefault())
                  .region("us")
                  .project("MY_PROJECT_ID")
                  .build()
          )
          .build();
  }
  ```

  ```php PHP
  use Anthropic\Vertex;

  $client = Vertex\Client::fromEnvironment(
      region: 'us', // Multi-region identifier: "us" or "eu"
      projectId: 'MY_PROJECT_ID',
  );
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::VertexClient.new(
    region: "us", # Multi-region identifier: "us" or "eu"
    project_id: "MY_PROJECT_ID"
  )
  ```
</CodeGroup>

## 功能支持

Claude Models on Google Cloud (Google managed) 支持的功能包括：

* [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create)

* [“Prompt caching”（提示缓存）](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)

* [思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)

* [“Tool use”（工具使用）](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)，包括以下工具：

  * [Bash 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/bash-tool)
  * [浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)
  * [计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)
  * [记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)
  * [文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)
  * [网络搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)

* [引用](https://platform.claude.com/docs/zh-CN/build-with-claude/citations)

* [结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)

* 在[支持的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#feature-availability)上使用["Fast mode"（快速模式）](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)

[功能概览](https://platform.claude.com/docs/zh-CN/build-with-claude/overview)列出了所有功能。每个功能的页面会显示 Claude Models on Google Cloud (Google managed) 是否支持该功能。

### 不支持的功能

Claude Models on Google Cloud (Google managed) 不支持以下功能：

* **输入源：** 图像和文档的 URL 源，以及 Files API
* **服务器端工具：** 代码执行、网页获取和顾问
* **智能体基础设施：** Agent Skills、MCP 连接器和程序化工具调用
* **API 端点：** Message Batches API、Models API、Admin API、Compliance API 以及用量和成本 API
* **Claude Managed Agents**
* **服务器端回退：** [`fallbacks` 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)

请使用[客户端回退模式](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#client-side-fallback)代替服务器端回退。

### 上下文窗口

Claude Models on Google Cloud (Google managed) 为以下模型提供 [1M 令牌的"context window"（上下文窗口）](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)：

* Claude Fable 5.2
* Claude Mythos 5.2
* Claude Fable 5.1
* Claude Mythos 5.1
* Claude Fable 5
* Claude Mythos 5
* Claude Opus 5.5
* Claude Opus 5
* Claude Opus 4.8
* Claude Opus 4.7
* Claude Opus 4.6
* Claude Sonnet 5.5
* Claude Sonnet 5
* Claude Sonnet 4.6
* Claude Haiku 5.5

其他 Claude 模型的上下文窗口为 200k 令牌，其中包括 Claude Sonnet 4.5（已弃用）和 Claude Sonnet 4（已弃用）。

Agent Platform 将请求负载限制为 30 MB。当您发送大型文档或大量图像时，可能会在达到令牌限制之前先达到此限制。

## 计费、速率限制和日志记录

### 速率限制

Google Cloud 管理 Claude Models on Google Cloud (Google managed) 的"rate limit"（速率限制）和配额。有关详细信息，包括如何申请提高配额，请参阅 Google Cloud 的 [Anthropic Claude 模型的配额](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/claude/quotas)页面。

### 计费

Google Cloud 会就 Claude Models on Google Cloud (Google managed) 向您开具账单。有关价格，请参阅 Google Cloud 的[生成式 AI 定价](https://cloud.google.com/vertex-ai/generative-ai/pricing#claude-models)页面。价格还取决于端点类型。请参阅[全局、多区域和区域端点](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#global-multi-region-and-regional-endpoints)。

### 数据保留

Google Cloud 负责管理 Claude Models on Google Cloud (Google managed) 的数据处理。有关详细信息，请参阅 [Agent Platform 和零数据保留](https://cloud.google.com/vertex-ai/generative-ai/docs/data-governance)。

### 监控和日志记录

要记录 Claude Models on Google Cloud (Google managed) 的提示和补全内容，请使用 Agent Platform 的[请求-响应日志记录服务](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/request-response-logging)。

Anthropic 建议您至少保留最近 30 天的日志。这些日志有助于您了解自己的活动并调查可能的滥用行为。

<Note>
  启用此服务不会让 Google 或 Anthropic 获得对您内容的任何访问权限。
</Note>

## 从 Claude Models on Google Cloud (Anthropic managed) 迁移

有关迁移时会发生哪些变化，请参阅[两种 Google Cloud 产品的区别](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/overview#how-the-two-google-cloud-offerings-differ)。要迁移到 Claude Models on Google Cloud (Google managed)，请按照以下步骤操作：

<Steps>
  <Step title="设置访问权限">
    检查您的项目以及发起调用的身份是否满足[进行 API 调用之前](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#before-making-api-calls)中的要求。
  </Step>

  <Step title="安装 SDK 软件包">
    安装为 Claude Models on Google Cloud (Google managed) 提供客户端的软件包。请参阅[安装 SDK](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#install-an-sdk-for-accessing-vertex-ai)。
  </Step>

  <Step title="替换导入和客户端">
    修改导入和创建客户端的代码。使用 `AnthropicVertex` (csharp: `AnthropicVertexClient`; go: `vertex.WithGoogleAuth()`; java: `VertexBackend`; php: `Anthropic\Vertex\Client`; ruby: `Anthropic::VertexClient`) 代替 `AnthropicGoogleCloud` (csharp: `AnthropicGoogleCloudClient`; go: `googlecloud.NewClient()`; java: `GoogleCloudBackend`; php: `Anthropic\GoogleCloud\Client`; ruby: `Anthropic::GoogleCloudClient`)。cURL 请求需要新的 URL、新的请求头和新的请求体。以下代码创建您目前使用的 Claude Models on Google Cloud (Anthropic managed) 客户端：

    <CodeGroup>
      ```bash cURL
      curl "https://claude.googleapis.com/v1/projects/$ANTHROPIC_GOOGLE_CLOUD_PROJECT/locations/global/workspaces/$ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID/invoke/v1/messages" \
        -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
        -H "content-type: application/json" \
        -H "anthropic-version: 2026-10-21" \
        -d '{
          "model": "claude-opus-5-5",
          "max_tokens": 1024,
          "messages": [
            {"role": "user", "content": "Hello, Claude"}
          ]
        }'
      ```

      ```bash CLI
      ant messages create \
        --model claude-opus-5-5 \
        --max-tokens 1024 \
        --message '{role: user, content: "Hello, Claude"}'
      ```

      ```python Python
      from anthropic import AnthropicGoogleCloud

      client = AnthropicGoogleCloud()
      ```

      ```typescript TypeScript
      import { AnthropicGoogleCloud } from "@anthropic-ai/google-cloud-sdk";

      const client = new AnthropicGoogleCloud();
      ```

      ```csharp C#
      using Anthropic.GoogleCloud;

      var client = new AnthropicGoogleCloudClient();
      ```

      ```go Go
      import (
      	"context"

      	"github.com/anthropics/anthropic-sdk-go/v2"
      	"github.com/anthropics/anthropic-sdk-go/v2/googlecloud"
      )
      // ...
      	client, err := googlecloud.NewClient(context.Background(), googlecloud.ClientConfig{})
      	if err != nil {
      		panic(err)
      	}
      ```

      ```java Java
      import com.anthropic.client.AnthropicClient;
      import com.anthropic.client.okhttp.AnthropicOkHttpClient;
      import com.anthropic.googlecloud.backends.GoogleCloudBackend;

      void main() {
          AnthropicClient client = AnthropicOkHttpClient.builder()
              .backend(GoogleCloudBackend.fromEnv())
              .build();
      }
      ```

      ```php PHP
      use Anthropic\GoogleCloud\Client;

      $client = new Client();
      ```

      ```ruby Ruby
      require "anthropic"

      client = Anthropic::GoogleCloudClient.new
      ```
    </CodeGroup>

    以下代码创建 Claude Models on Google Cloud (Google managed) 的客户端。它使用全局端点。有关其他端点类型，请参阅[全局、多区域和区域端点](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#global-multi-region-and-regional-endpoints)。

    <CodeGroup exclude="shell:CLI">
      ```bash cURL
      curl https://aiplatform.googleapis.com/v1/projects/MY_PROJECT_ID/locations/global/publishers/anthropic/models/claude-opus-5-5:rawPredict \
        -H "Authorization: Bearer $(gcloud auth print-access-token)" \
        -H "Content-Type: application/json" \
        -d '{
          "anthropic_version": "2026-10-21",
          "messages": [{"role": "user", "content": "Hey Claude!"}],
          "max_tokens": 100
        }'
      ```

      <CodeGroupItem language="CLI">
        `ant` CLI 不支持 Claude Models on Google Cloud (Google managed)。请改用 cURL 或 SDK。
      </CodeGroupItem>

      ```python Python
      from anthropic import AnthropicVertex

      project_id = "MY_PROJECT_ID"
      region = "global"

      client = AnthropicVertex(project_id=project_id, region=region)
      ```

      ```typescript TypeScript
      import { AnthropicVertex } from "@anthropic-ai/vertex-sdk";

      const projectId = "MY_PROJECT_ID";
      const region = "global";

      const client = new AnthropicVertex({
        projectId,
        region
      });
      ```

      ```csharp C#
      using Anthropic.Vertex;

      var projectId = "MY_PROJECT_ID";
      var region = "global";

      var client = new AnthropicVertexClient(new AnthropicVertexCredentials(region, projectId));
      ```

      ```go Go
      import (
      	"context"

      	"github.com/anthropics/anthropic-sdk-go/v2"
      	"github.com/anthropics/anthropic-sdk-go/v2/vertex"
      )
      // ...
      	// 使用默认的 Google Cloud 凭据
      	client := anthropic.NewClient(
      		vertex.WithGoogleAuth(context.Background(), "global", "MY_PROJECT_ID"),
      	)
      ```

      ```java Java
      import com.anthropic.client.AnthropicClient;
      import com.anthropic.client.okhttp.AnthropicOkHttpClient;
      import com.anthropic.vertex.backends.VertexBackend;
      import com.google.auth.oauth2.GoogleCredentials;

      void main() throws Exception {
          AnthropicClient client = AnthropicOkHttpClient.builder()
              .backend(
                  VertexBackend.builder()
                      .googleCredentials(GoogleCredentials.getApplicationDefault())
                      .region("global")
                      .project("MY_PROJECT_ID")
                      .build()
              )
              .build();
      }
      ```

      ```php PHP
      use Anthropic\Vertex;

      $client = Vertex\Client::fromEnvironment(
          region: 'global',
          projectId: 'MY_PROJECT_ID',
      );
      ```

      ```ruby Ruby
      require "anthropic"

      client = Anthropic::VertexClient.new(
        region: "global",
        project_id: "MY_PROJECT_ID"
      )
      ```
    </CodeGroup>

    此代码在创建客户端时传入您的项目 ID 和区域。请将 `MY_PROJECT_ID` 替换为您的项目 ID。您使用该客户端进行的 Messages API 调用保持不变，例外情况是下一步中的模型 ID 和功能。
  </Step>

  <Step title="检查模型 ID 和功能">
    * **模型 ID：** 如果您使用的是早于 Claude 4.6 的模型，请将其 ID 中快照日期前的连字符替换为 `@`。例如，`claude-haiku-4-5-20251001` 变为 `claude-haiku-4-5@20251001`。有关模型及其 ID，请参阅 [API 模型 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#api-model-ids)。
    * **功能：** 检查您的代码所使用的功能是否未列在[不支持的功能](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#features-not-supported)中。
  </Step>

  <Step title="运行测试请求">
    运行[发起请求](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#accessing-vertex-ai)中的请求，以检查您的设置是否正常工作。然后运行您自己的代码。在迁移生产流量之前，请阅读 [Claude Models on Google Cloud (Google managed) 的速率限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed#rate-limits)。
  </Step>
</Steps>

## 后续步骤

<CardGroup cols={2}>
  <Card title="功能概览" icon="stack" href="https://platform.claude.com/docs/zh-CN/build-with-claude/overview">
    探索 Claude 的功能。
  </Card>

  <Card title="Agent Platform 上的 Anthropic Claude 模型" icon="book" href="https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/claude">
    查看每个区域提供哪些 Claude 模型及其停用日期。
  </Card>

  <Card title="Agent Platform 文档" icon="book" href="https://cloud.google.com/vertex-ai/docs">
    阅读 Google Cloud’s Agent Platform 文档。
  </Card>

  <Card title="Claude 模型的全局端点" icon="link" href="https://cloud.google.com/blog/products/ai-machine-learning/global-endpoint-for-claude-models-generally-available-on-vertex-ai">
    阅读 Google Cloud 关于 Claude 模型全局端点的博客文章。
  </Card>
</CardGroup>
