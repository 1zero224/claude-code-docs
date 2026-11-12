---
title: 处理流式传输拒绝
url: https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/handle-streaming-refusals
description: 检测并处理流式传输响应中的拒绝停止原因，并在回退模型上重试被拒绝的请求。
featureMetadata:
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
    - claude-haiku-4-5-20251001
    - claude-opus-4-8
    - claude-opus-4-7
    - claude-opus-4-6
    - claude-sonnet-4-6
    - claude-opus-4-5-20251101
    - claude-sonnet-4-5-20250929
  supportedPlatforms:
    Claude API: stable
    Amazon Bedrock: stable
    Amazon Bedrock (bedrock-runtime): stable
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

**API 支持**：用法因产品而异。

* **Messages API**：流以 `stop_reason` `"refusal"` 结束，`stop_details`（`category` 和 `explanation`）随 `message_delta` 事件送达。
* **Managed Agents**：被拒绝的轮次会结束，会话进入空闲状态。如果 `stop_details.type` 或 `stop_reason.type` 为 `"refusal"`，则应将该轮次视为被拒绝。

从 Claude 4 模型开始，当流式传输分类器介入以处理潜在的策略违规时，Claude API 的 “streaming”（流式传输）响应会返回 **`stop_reason`: `"refusal"`**。此安全功能有助于在实时流式传输期间保持内容合规。

<Tip>
  本页介绍拒绝在流式传输响应中的呈现方式。有关每个 `stop_reason` 值及其处理方式，请参阅[停止原因与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons)。要在另一个 Claude 模型上重试被拒绝的请求，请参阅[拒绝与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)。
</Tip>

## API 响应格式

当流式传输分类器检测到违反 Anthropic 策略的内容时，API 会返回以下响应：

```json
{
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello.."
    }
  ],
  "stop_reason": "refusal",
  "stop_details": {
    "type": "refusal",
    "category": "cyber",
    "explanation": "This request was declined because it could enable cyber harm."
  }
}
```

在事件流中，`stop_details` 会与 `stop_reason` 一起出现在 `message_delta` 事件中。

<Note>
  来自流式分类器的 `refusal` 响应包含一个 `stop_details` 对象，其中含有 `category` 和一个可供您向用户展示的人类可读的 `explanation`。有关完整的响应结构和可用类别，请参阅[拒绝是什么样的](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)。

  发生拒绝时，`stop_details` 对象始终存在，但其 `category` 和 `explanation` 字段可能为 `null`，例如当该拒绝未映射到任何已命名类别时。请根据 `stop_reason` 或 `stop_details.type` 进行分支判断，而不要假定 `category` 和 `explanation` 已被填充；当它们为 `null` 时，请提供您自己的面向用户的消息。
</Note>

## 拒绝后重置上下文

当您收到 **`stop_reason`: `refusal`** 时，必须在继续之前重置对话上下文。您可以删除或改写触发拒绝的那一轮对话，也可以完全清除对话历史。若不重置而尝试继续，将导致持续的拒绝。

<Note>
  即使响应被拒绝，响应中仍会提供用量指标。被拒绝的请求是否计费取决于拒绝发生的时间及其类别；请参阅[拒绝如何计费](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#how-refusals-are-billed)。
</Note>

<Tip>
  重置上下文并不是唯一的恢复方式。您也可以在另一个 Claude 模型上重试被拒绝的请求，[拒绝与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)页面展示了如何通过服务器端回退、SDK 中间件或手动重试来进行设置。
</Tip>

## 实现指南

以下是在您的应用程序中检测和处理流式传输拒绝的方法：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    在流中检测拒绝并重置对话需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    在流中检测拒绝并重置对话需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  client = anthropic.Anthropic()
  messages = []


  def reset_conversation():
      """Reset conversation context after refusal"""
      global messages
      messages = []
      print("Conversation reset due to refusal")


  try:
      with client.messages.stream(
          max_tokens=1024,
          messages=messages + [{"role": "user", "content": "Hello"}],
          model="claude-opus-5-5",
      ) as stream:
          for event in stream:
              # 检查 message delta 中是否存在拒绝
              if event.type == "message_delta":
                  if event.delta.stop_reason == "refusal":
                      reset_conversation()
                      break
  except Exception as e:
      print(f"Error: {e}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();
  let messages: Anthropic.MessageParam[] = [];

  function resetConversation() {
    // 拒绝后重置对话上下文
    messages = [];
    console.log("Conversation reset due to refusal");
  }

  try {
    const stream = await client.messages.stream({
      messages: [...messages, { role: "user", content: "Hello" }],
      model: "claude-opus-5-5",
      max_tokens: 1024
    });

    for await (const event of stream) {
      // 检查 message delta 中是否包含拒绝
      if (event.type === "message_delta" && event.delta.stop_reason === "refusal") {
        resetConversation();
        break;
      }
    }
  } catch (error) {
    console.error("Error:", error);
  }
  ```

  ```csharp C#
  List<Message> messages = new();
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello" }]
  };

  try
  {
      await foreach (var streamEvent in client.Messages.CreateStreaming(parameters))
      {
          if (
              streamEvent.TryPickDelta(out var deltaEvent)
              && deltaEvent.Delta.StopReason == StopReason.Refusal
          )
          {
              ResetConversation();
              break;
          }
      }
  }
  catch (Exception e)
  {
      Console.WriteLine($"Error: {e.Message}");
  }

  void ResetConversation()
  {
      messages.Clear();
      Console.WriteLine("Conversation reset due to refusal");
  }
  ```

  ```go Go
  var messages []anthropic.MessageParam

  func resetConversation() {
  	messages = []anthropic.MessageParam{}
  	fmt.Println("Conversation reset due to refusal")
  }
  // ...
  	client := anthropic.NewClient()

  	stream := client.Messages.NewStreaming(context.TODO(), anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 1024,
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello")),
  		},
  	})

  streamLoop:
  	for stream.Next() {
  		event := stream.Current()
  		switch eventVariant := event.AsAny().(type) {
  		case anthropic.MessageDeltaEvent:
  			if eventVariant.Delta.StopReason == anthropic.StopReasonRefusal {
  				resetConversation()
  				break streamLoop
  			}
  		}
  	}

  	if err := stream.Err(); err != nil {
  		log.Fatal(err)
  	}
  ```

  ```java Java
  import com.anthropic.core.http.StreamResponse;
  import com.anthropic.models.messages.RawMessageStreamEvent;
  import com.anthropic.models.messages.StopReason;
  // ...

  List<MessageParam> messages = new ArrayList<>();

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addUserMessage("Hello")
          .build();

      try (StreamResponse<RawMessageStreamEvent> stream = client.messages().createStreaming(params)) {
          stream.stream().forEach(event -> {
              event.messageDelta().ifPresent(deltaEvent -> {
                  deltaEvent.delta().stopReason().ifPresent(stopReason -> {
                      if (stopReason.equals(StopReason.REFUSAL)) {
                          resetConversation();
                      }
                  });
              });
          });
      } catch (Exception e) {
          System.err.println("Error: " + e.getMessage());
      }
  }

  void resetConversation() {
      messages.clear();
      IO.println("Conversation reset due to refusal");
  }
  ```

  ```php PHP
  $client = new Client();
  $messages = [];

  function resetConversation(&$messages)
  {
      $messages = [];
      echo "Conversation reset due to refusal\n";
  }

  try {
      $stream = $client->messages->createStream(
          maxTokens: 1024,
          messages: [
              ['role' => 'user', 'content' => 'Hello']
          ],
          model: 'claude-opus-5-5',
      );

      foreach ($stream as $event) {
          if ($event->type === 'message_delta' && $event->delta->stopReason === 'refusal') {
              resetConversation($messages);
              break;
          }
      }
  } catch (\Exception $e) {
      echo 'Error: ' . $e->getMessage() . "\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new
  messages = []

  def reset_conversation(messages)
    messages.clear
    puts "Conversation reset due to refusal"
  end

  begin
    stream = client.messages.stream(
      model: :"claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello" }]
    )

    stream.each do |event|
      if event.type == :message_delta && event.delta.stop_reason == :refusal
        reset_conversation(messages)
        break
      end
    end
  rescue => e
    puts "Error: #{e.message}"
  end
  ```
</CodeGroup>

## 当前的拒绝类型

API 目前以三种不同的方式处理拒绝：

| 拒绝类型        | 响应格式                         | 发生时机          |
| ----------- | ---------------------------- | ------------- |
| 流式传输分类器拒绝   | **`stop_reason`: `refusal`** | 流式传输期间内容违反策略时 |
| API 输入和版权验证 | 400 错误代码                     | 输入未通过验证检查时    |
| 模型生成的拒绝     | 标准文本响应                       | 模型自身拒绝时       |

## 最佳实践

* **监控拒绝：** 在您的错误处理中加入 **`stop_reason`: `refusal`** 检查。
* **自动重置：** 在检测到拒绝时实现自动上下文重置。
* **回退到另一个模型：** 配置[服务器端回退或 SDK 中间件](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)，使被拒绝的请求在另一个 Claude 模型上重试，而不是向用户呈现拒绝。Claude Haiku 5.5 没有服务器端回退，因此请在您的客户端中设置其重试。
* **在手动重试时兑换回退额度：** 如果您自行构建重试，请传递拒绝的[回退额度](https://platform.claude.com/docs/zh-CN/build-with-claude/fallback-credit)令牌，这样重试就不会重复支付提示缓存成本。Claude Haiku 5.5 的拒绝不附带回退额度。
* **提供自定义消息：** 在发生拒绝时创建用户友好的消息，以获得更好的用户体验。
* **跟踪拒绝模式：** 监控拒绝频率，以识别您的提示中的潜在问题。

## 迁移说明

如果您在此功能首次发布时就构建了拒绝处理逻辑，或者正在将其添加到现有集成中，请检查以下事项：

* **拒绝是响应，而不是错误。** 拒绝以成功的 HTTP 200 响应形式到达，并带有 `stop_reason`: `"refusal"`，因此仅基于错误率构建的监控无法发现它。请将拒绝作为独立的信号进行跟踪。
* **拒绝包含结构化详细信息。** 在所有模型上，拒绝还包含一个 `stop_details` 对象，用于标识拒绝背后的政策类别。有关完整的响应结构，请参阅[拒绝是什么样的](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)。
* **在不同的模型上重试。** 将被拒绝的请求重新发送到同一模型通常会导致再次被拒绝。与其仅重置上下文，不如通过[服务器端回退、SDK 中间件或手动重试](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)在回退模型上重试，并在自行构建重试时兑换[回退抵扣](https://platform.claude.com/docs/zh-CN/build-with-claude/fallback-credit)。Claude Haiku 5.5 没有服务器端回退，其拒绝也不附带回退抵扣，因此请在您的客户端中为其设置重试。
* **检查批处理结果中的拒绝。** [消息批次](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)中被拒绝的请求会作为成功结果返回，并带有 `stop_reason`: `"refusal"`，而不是作为出错结果返回。
* **围绕 `stop_reason` 集中处理。** API 将继续围绕 `stop_reason`: `"refusal"` 整合拒绝处理，因此请根据停止原因进行分支判断，而不是依赖特定模型的行为。

## 后续步骤

<CardGroup cols={2}>
  <Card title="拒绝与回退" icon="arrows-clockwise" href="https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback">
    在服务器端或您的客户端中，在另一个 Claude 模型上重试被拒绝的请求。
  </Card>

  <Card title="停止原因与回退" icon="code" href="https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons">
    每个 `stop_reason` 值及其处理方式。
  </Card>

  <Card title="流式传输消息" icon="lightning" href="https://platform.claude.com/docs/zh-CN/build-with-claude/streaming">
    流式传输响应，并在 `message_delta` 事件到达时从中读取 `stop_reason`。
  </Card>

  <Card title="多语言支持" icon="text-aa" href="https://platform.claude.com/docs/zh-CN/build-with-claude/multilingual-support">
    利用 Claude 的跨语言能力为不同语言的用户提供服务。
  </Card>
</CardGroup>

## 可用性

流式拒绝可用或处于 beta 阶段，具体取决于API。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：始终开启
* 用法：流以 `stop_reason` `"refusal"` 结束，`stop_details`（`category` 和 `explanation`）随 `message_delta` 事件送达。

| 状态 | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：始终开启
* 用法：被拒绝的轮次会结束，会话进入空闲状态。如果 `session.status_idle` 事件上的 `stop_details.type` 或 `stop_reason.type` 为 `"refusal"`，则应将该轮次视为被拒绝。在被拒绝的轮次中，`stop_reason.type` 可能为 `"end_turn"`。在其他任何轮次中，`stop_details` 均为 `null`，在被拒绝的轮次中也可能为 `null`。

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |
