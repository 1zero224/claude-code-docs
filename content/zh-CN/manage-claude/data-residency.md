---
title: 数据驻留
url: https://platform.claude.com/docs/zh-CN/manage-claude/data-residency
description: 通过地理控制管理模型推理的运行位置和数据的存储位置。
featureMetadata:
  status: stable
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements).
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-fable-5-1
    - claude-mythos-5-1
    - claude-fable-5
    - claude-mythos-5
    - claude-mythos-preview
    - claude-opus-5-5
    - claude-opus-5
    - claude-sonnet-5-5
    - claude-sonnet-5
    - claude-haiku-5-5
    - claude-opus-4-8
    - claude-opus-4-7
    - claude-opus-4-6
    - claude-sonnet-4-6
  supportedPlatforms:
    Claude API: stable
    Claude Platform on AWS:
      availability: stable
      note: 工作区地域不可配置；其中的 Managed Agents 会话以工作区地域 “us” 运行。
    Claude Models on Google Cloud (Anthropic managed):
      availability: stable
      note: 工作区地域不可配置。
---

**API 支持**：用法因产品而异。

* **Messages API**：在每个请求中设置 `inference_geo`，或为工作区设置默认值。
* **Managed Agents**：在智能体的模型配置中固定 `inference_geo`；在创建会话时覆盖它。

“Data residency”（数据驻留）控制让您可以管理数据的处理和存储位置。有两个相互独立的设置对此进行管理：

* **推理地理位置（Inference geo）：** 按请求控制模型推理的运行位置。通过 `inference_geo` API 参数设置，或设置为工作区默认值。
* **工作区地理位置（Workspace geo）：** 控制静态数据的存储位置以及端点处理（例如图像转码和代码执行）的发生位置。在 [Claude Console](https://platform.claude.com/) 中按工作区级别进行配置。

<Note>
  [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 支持在智能体级别进行地理位置固定：在[智能体的模型配置](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#pin-the-inference-geo)上设置 `inference_geo`，可固定为运行该智能体的会话提供模型请求服务的地理位置，并可在创建会话时进行[按会话覆盖](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#pin-the-inference-geo-for-a-session)。未固定地理位置的智能体在每个请求上遵循工作区的默认推理地理位置。Managed Agents 还遵循在 Console 中配置的工作区地理位置；使用[自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)时，工具执行和沙箱文件系统保留在您控制的基础设施上；所附加的[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)的内容仍由 Anthropic 存储，并在会话期间复制到您的沙箱中。
</Note>

## 推理地理位置

`inference_geo` 参数控制特定 API 请求的模型推理运行位置。可将其添加到任何 `POST /v1/messages` 调用中。

| 值          | 描述                                |
| ---------- | --------------------------------- |
| `"global"` | 默认值。推理可能在任何可用的地理位置运行，以获得最佳性能和可用性。 |
| `"us"`     | 推理仅在位于美国的基础设施上运行。                 |

### API 用法

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "inference_geo": "us",
      "messages": [{
        "role": "user",
        "content": "Summarize the key points of this document."
      }]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --inference-geo us \
    --message '{role: user, content: "Summarize the key points of this document."}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      inference_geo="us",
      messages=[
          {"role": "user", "content": "Summarize the key points of this document."}
      ],
  )

  for block in response.content:
      if block.type == "text":
          print(block.text)
  # 检查推理的实际运行位置
  print(f"Inference geo: {response.usage.inference_geo}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    inference_geo: "us",
    messages: [
      {
        role: "user",
        content: "Summarize the key points of this document."
      }
    ]
  });

  const textBlock = response.content.find((block) => block.type === "text");
  console.log(textBlock?.text);
  // 检查推理实际运行的位置
  console.log(`Inference geo: ${response.usage.inference_geo}`);
  ```

  ```csharp C#
  var client = new AnthropicClient();

  var response = await client.Messages.Create(
      new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          InferenceGeo = "us",
          Messages =
          [
              new UserMessageParam { Content = "Summarize the key points of this document." },
          ],
      }
  );

  foreach (var block in response.Content)
  {
      if (block.TryPickText(out var textBlock))
      {
          Console.WriteLine(textBlock.Text);
      }
  }

  // 检查推理实际运行的位置
  Console.WriteLine($"Inference geo: {response.Usage.InferenceGeo}");
  ```

  ```go Go
  client := anthropic.NewClient()

  message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  	Model:        anthropic.ModelClaudeOpus5_5,
  	MaxTokens:    1024,
  	InferenceGeo: anthropic.String("us"),
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Summarize the key points of this document.")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  for _, block := range message.Content {
  	if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  // 检查推理实际运行的位置
  fmt.Printf("Inference geo: %s\n", message.Usage.InferenceGeo)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  Message response = client.messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .inferenceGeo("us")
          .addUserMessage("Summarize the key points of this document.")
          .build());

  response.content().stream()
      .flatMap(block -> block.text().stream())
      .forEach(textBlock -> IO.println(textBlock.text()));
  // 检查推理实际运行的位置
  IO.println("Inference geo: " + response.usage().inferenceGeo().get());
  ```

  ```php PHP
  $client = new Client();

  $response = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 1024,
      inferenceGeo: 'us',
      messages: [
          ['role' => 'user', 'content' => 'Summarize the key points of this document.'],
      ],
  );

  foreach ($response->content as $block) {
      if ($block->type === 'text') {
          echo $block->text, PHP_EOL;
      }
  }
  // 检查推理实际运行的位置
  echo "Inference geo: {$response->usage->inferenceGeo}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    inference_geo: "us",
    messages: [
      {role: "user", content: "Summarize the key points of this document."}
    ]
  )

  response.content.each do |block|
    puts block.text if block.type == :text
  end
  # 检查推理实际运行的位置
  puts "Inference geo: #{response.usage.inference_geo}"
  ```
</CodeGroup>

### 响应

响应的 `usage` 对象包含一个 `inference_geo` 字段，用于指示推理的运行位置：

```json Output
{
  "usage": {
    "input_tokens": 25,
    "output_tokens": 150,
    "inference_geo": "us"
  }
}
```

### 模型可用性

`inference_geo` 参数在 Claude 4.6 及更高版本的模型上受支持。在 Claude Opus 4.5、Claude Sonnet 4.5（已弃用）、Claude Haiku 4.5 或更早的模型上使用 `inference_geo` 的请求会返回 400 错误。

<Note>
  `inference_geo` 参数可在 Claude API（第一方）、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) 和 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) 上使用。在 Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock 上，推理区域由端点 URL 或推理配置文件决定，因此 `inference_geo` 不适用。在 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上，`inference_geo` 同样不适用：托管在 Azure 上的部署可以改用 US Data Zone Standard 部署类型，该类型将推理保持在美国境内。`inference_geo` 参数也无法通过 [OpenAI SDK 兼容性端点](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/openai-sdk)使用。
</Note>

### 工作区级别限制

工作区设置还支持限制可用的推理地理位置：

* **`allowed_inference_geos`：** 限制工作区可以使用的地理位置。如果请求指定的 `inference_geo` 不在此列表中，API 将返回错误。
* **`default_inference_geo`：** 设置请求中省略 `inference_geo` 时的回退地理位置。单个请求可以通过显式设置 `inference_geo` 来覆盖此设置。

这些设置可以通过 Console 或 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 在 `data_residency` 字段下进行配置。

<Note>
  **Claude Models on Google Cloud (Anthropic managed)：** 每个工作区都允许所有推理地理位置，并使用 `"global"` 作为其默认推理地理位置。Claude Console 将这两项设置显示为只读。如需仅在美国进行推理，请在每个请求上设置 `inference_geo: "us"`。
</Note>

## 工作区地理位置

工作区地理位置在创建工作区时设置，之后无法更改。目前，`"us"` 是唯一可用的工作区地理位置。

要设置工作区地理位置，请在 [Console](https://platform.claude.com/) 中创建一个新工作区：

1. 前往 [Claude Console 中的工作区](https://platform.claude.com/settings/workspaces)（`platform.claude.com/settings/workspaces`）。
2. 点击**创建工作区**，在 Console 中打开[创建工作区](https://platform.claude.com/settings/workspaces/new)页面（`platform.claude.com/settings/workspaces/new`）。
3. 输入名称，确认**工作区地理位置**字段中的值，然后点击**创建工作区**。

<Note>
  **Claude Platform on AWS：** 工作区地理位置不可配置。此平台上的 Claude Managed Agents 会话以 `"us"` 作为实际生效的工作区地理位置运行，这也是目前唯一可用的工作区地理位置。有关该平台特定的数据驻留注意事项，请参阅 [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)。
</Note>

<Note>
  **Claude Models on Google Cloud (Anthropic managed)：** 工作区地理位置不可配置。每个工作区都使用 `"us"` 作为其工作区地理位置，并且**工作区地理位置**字段不会出现在 Claude Console 中。
</Note>

## 定价

数据驻留定价因模型代际而异：

* **Claude 4.6 及更高版本的模型：** 仅限美国的推理（`inference_geo: "us"`）在所有令牌定价类别（输入令牌、输出令牌、缓存写入和缓存读取）中按标准费率的 1.1 倍计价。在 Claude Haiku 5.5 上，该乘数也适用于超过 100,000 个令牌的提示所适用的更高价格（请参阅[长上下文定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)）。
* **全局路由（`inference_geo: "global"`）：** 适用标准定价。
* **较旧的模型：** 不支持 `inference_geo`（请参阅[模型可用性](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency#model-availability)）。适用标准定价。包含该参数的请求会返回 400 错误。

此定价适用于 Claude API（第一方）、Claude Platform on AWS 和 Claude Models on Google Cloud (Anthropic managed)。在 Claude in Microsoft Foundry 上，相同的 1.1 倍乘数适用于托管在 Azure 上且使用 US Data Zone Standard 部署类型的部署。由合作伙伴运营的平台，即 Claude Models on Google Cloud (Google managed) 和 Bedrock，有其各自的区域定价。有关详细信息，请参阅[数据驻留定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#data-residency-pricing)。

相同的乘数也适用于 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)：当智能体的[模型配置](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)将 `inference_geo` 固定为 `"us"` 时，运行该智能体的会话中的模型请求按标准费率的 1.1 倍计价。

<Note>
  如果您有 [Priority Tier](https://platform.claude.com/docs/zh-CN/api/service-tiers) 承诺，仅限美国推理的 1.1 倍乘数也会影响令牌在您的 Priority Tier 容量中的计算方式。使用 `inference_geo: "us"` 消耗的每个令牌会从您承诺的 TPM 中扣除 1.1 个令牌，这与其他定价乘数（例如提示缓存）影响消耗速率的方式一致。
</Note>

## Batch API 支持

`inference_geo` 参数在 [Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 上受支持。批处理中的每个请求都可以指定自己的 `inference_geo` 值。

## 从旧版退出选项迁移

如果您的组织之前选择退出全局路由以将推理保留在美国，您的工作区已自动配置为 `allowed_inference_geos: ["us"]` 和 `default_inference_geo: "us"`。无需更改代码。您现有的数据驻留要求将继续通过新的地理位置控制来执行。

### 发生了哪些变化

旧版退出选项是一项组织级别的设置，它将所有请求限制在位于美国的基础设施上。新的数据驻留控制用两种机制取代了它：

* **按请求控制：** `inference_geo` 参数让您可以在每个 API 调用上指定 `"us"` 或 `"global"`，为您提供请求级别的灵活性。
* **工作区控制：** Console 中的 `default_inference_geo` 和 `allowed_inference_geos` 设置让您可以在工作区中的所有密钥上强制执行地理位置策略。

### 您的工作区发生了什么

您的工作区已自动迁移：

| 旧版设置         | 新的等效设置                                                         |
| ------------ | -------------------------------------------------------------- |
| 退出全局路由（仅限美国） | `allowed_inference_geos: ["us"]`、`default_inference_geo: "us"` |

所有使用您工作区密钥的 API 请求将继续在位于美国的基础设施上运行。无需执行任何操作即可保持您当前的行为。

### 如果您想使用全局路由

如果您的数据驻留要求已发生变化，并且您希望利用全局路由获得更好的性能和可用性，请更新工作区的推理地理位置设置，在允许的地理位置中包含 `"global"`，并将 `default_inference_geo` 设置为 `"global"`。有关详细信息，请参阅[工作区级别限制](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency#workspace-level-restrictions)。

### 定价影响

旧版模型不受此迁移影响。有关较新模型的当前定价，请参阅[定价](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency#pricing)。

## 当前限制

* **共享速率限制：** 速率限制在所有地理位置之间共享。
* **推理地理位置：** 仅 `"us"` 和 `"global"` 可用。
* **工作区地理位置：** 目前仅 `"us"` 可用。工作区地理位置在工作区创建后无法更改。

## 后续步骤

<CardGroup>
  <Card title="数据驻留定价" icon="dollar-sign" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing#data-residency-pricing">
    查看数据驻留定价详情。
  </Card>

  <Card title="工作区" icon="building" href="https://platform.claude.com/docs/zh-CN/manage-claude/workspaces">
    了解工作区配置。
  </Card>

  <Card title="用量和成本分析 API" icon="chart" href="https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api">
    按推理地理位置跟踪用量和成本。
  </Card>
</CardGroup>

## 可用性

数据驻留在部分模型上可用，并具备零数据保留和HIPAA 就绪资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用
* 用法：在每个请求中设置 `inference_geo`，或为工作区设置默认值。

| 状态             | 云平台                                                                                                                                                           | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用             | Claude API                                                                                                                                                    | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), Claude Mythos Preview, [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview) |
| 可用<sup>1</sup> | [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)                                                     | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview)                                                                                                                                                                                                                                                                                            |
| 可用<sup>1</sup> | [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview)                                                                                                                                                                                                    |

1. 工作区地域不可配置。

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用
* 用法：在智能体的模型配置中固定 `inference_geo`，并在创建会话时按会话覆盖它。未固定的智能体遵循工作区默认值。

| 状态             | 云平台                                                                                                       | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| -------------- | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用             | Claude API                                                                                                | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), Claude Mythos Preview, [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview) |
| 可用<sup>1</sup> | [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview), [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-4-7/overview), [Claude Opus 4.6](https://platform.claude.com/docs/zh-CN/models/opus-4-6/overview), [Claude Sonnet 4.6](https://platform.claude.com/docs/zh-CN/models/sonnet-4-6/overview)                                                                                                                                                                                                                                                                                            |

1. 工作区地域不可配置；其中的 Managed Agents 会话以工作区地域 “us” 运行。

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目 | 零数据保留                                                                                                                                     | HIPAA |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态 | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 具备资格  |

适用的端点：

* `/v1/messages` 使用 `inference_geo`
