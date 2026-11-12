---
title: Claude Opus 5.5 迁移指南
url: https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide
description: 通过本迁移指南从早期 Opus 模型或 Claude Sonnet 5 切换到 Claude Opus 5.5。启用 Claude Opus 5.5 的指导内容包括会返回错误的请求设置、每个响应中的思考块，以及针对每个起始模型的检查清单。
---

<Note>
  本指南涵盖 [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages) 代码的迁移。如果您使用 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)，则除了更新模型名称之外无需进行任何更改。
</Note>

让 Claude Code 为您迁移代码

```text
/claude-api migrate 将此项目迁移到 claude-opus-5-5
```

本页列出了从 [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-5)、[Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-4-8)、[Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-47)、[Claude Opus 4.6 及更早的 Opus 模型](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-46)或 [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-sonnet-5) 迁移到 Claude Opus 5.5 所需的代码更改。所有读者都需要阅读[每个发往 Claude Opus 5.5 的请求必须满足的要求](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#request-requirements)和[处理每个响应中的思考](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-in-every-response)。然后前往您当前模型对应的章节：该章节的第一句话会列出适用于您的其他章节。[迁移检查清单](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migration-checklist)按起始模型列出了所有更改。

Claude Opus 5.5 的费用低于 Claude Opus 5（每百万输入 / 输出令牌 $4 / $20 美元，而 Claude Opus 5 为 $5 / $25 美元；请参阅[定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing)）。有关功能支持，请参阅 [Claude Opus 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#feature-support)。有关行为差异和特定于模型的提示模式，请参阅[提示 Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5)。

## 每个发往 Claude Opus 5.5 的请求必须满足的要求

无论您从哪个模型迁移而来，发往 `claude-opus-5-5` 的请求都必须满足以下要求。如果某项说明某个设置会被拒绝，则 API 会返回 400 错误。

* **模型 ID：** 使用 `claude-opus-5-5`，这是一个不带日期后缀的固定模型 ID。在 Amazon Bedrock、Claude Platform on AWS、Claude Models on Google Cloud (Anthropic managed)、Claude Models on Google Cloud (Google managed) 和 Microsoft Foundry 上，请使用该平台的模型 ID；请参阅[可用性](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#availability)。
* **思考：** 不发送 `thinking` 字段，或发送 `thinking: {"type": "adaptive"}`，两者等效：[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)始终处于开启状态。`thinking: {"type": "disabled"}` 和手动思考预算（`thinking: {"type": "enabled", "budget_tokens": N}`）会被拒绝。请参阅[思考的前后对比](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-cant-be-disabled)。
* **Effort：** 使用 [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)控制思考深度，这是唯一控制思考深度的请求参数。支持全部五个级别（`low`、`medium`、`high`、`xhigh`、`max`），默认值为 `medium`。请参阅 [Claude Opus 5.5 的推荐 effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-opus-5-5)。
* **工具选择：** 使用 `tool_choice` `{"type": "auto"}`（默认值）或 `{"type": "none"}`。使用 `{"type": "any"}` 或 `{"type": "tool", "name": "..."}` 强制调用工具会被拒绝。请参阅[工具选择的前后对比](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#forced-tool-use)。
* **采样参数：** 省略 `temperature`、`top_p` 和 `top_k`。请使用提示来引导模型的行为。
* **预填充：** 不要以预填充的助手轮次结束 `messages`：这会被拒绝。请改用[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)或系统提示指令。
* **计算机使用：** 将计算机使用声明为 `computer_toolset_20260801` 工具集。在 Amazon Bedrock 上，`computer_20251124` 也可以使用。请参阅[计算机使用的破坏性变更](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#computer-use-toolset)。
* **上下文窗口：** [1M 令牌上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)是默认设置。为旧模型发送的上下文窗口 beta 标头不会产生任何效果。

以下请求满足列表中的每一项：设置了 effort，且没有 `thinking` 字段。在示例打印文本的地方，它按类型查找文本块，因为 `thinking` 块排在前面。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "messages": [{
        "role": "user",
        "content": "Analyze the trade-offs between microservices and monolithic architectures"
      }],
      "output_config": {
        "effort": "medium"
      }
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 4096 \
    --output-config '{effort: medium}' \
    --message '{role: user, content: "Analyze the trade-offs between microservices and monolithic architectures"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      messages=[
          {
              "role": "user",
              "content": "Analyze the trade-offs between microservices and monolithic architectures",
          }
      ],
      output_config={"effort": "medium"},
  )

  for block in response.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: "Analyze the trade-offs between microservices and monolithic architectures"
      }
    ],
    output_config: {
      effort: "medium"
    }
  });

  const textBlock = response.content.find((block) => block.type === "text");
  console.log(textBlock?.text);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      Messages = [
          new UserMessageParam
          {
              Content = "Analyze the trade-offs between microservices and monolithic architectures"
          }
      ],
      OutputConfig = new OutputConfig
      {
          Effort = Effort.Medium
      }
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Analyze the trade-offs between microservices and monolithic architectures")),
  	},
  	OutputConfig: anthropic.OutputConfigParam{
  		Effort: anthropic.OutputConfigEffortMedium,
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  for _, block := range response.Content {
  	if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(4096L)
      .addUserMessage("Analyze the trade-offs between microservices and monolithic architectures")
      .outputConfig(OutputConfig.builder()
          .effort(OutputConfig.Effort.MEDIUM)
          .build())
      .build();

  Message response = client.messages().create(params);
  response.content().stream()
      .flatMap(block -> block.text().stream())
      .forEach(textBlock -> IO.println(textBlock.text()));
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 4096,
      messages: [
          ['role' => 'user', 'content' => 'Analyze the trade-offs between microservices and monolithic architectures']
      ],
      model: Model::CLAUDE_OPUS_5_5,
      outputConfig: ['effort' => 'medium'],
  );

  foreach ($message->content as $block) {
      if ($block instanceof \Anthropic\Messages\TextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    messages: [
      { role: "user", content: "Analyze the trade-offs between microservices and monolithic architectures" }
    ],
    output_config: {
      effort: "medium"
    }
  )

  message.content.each do |block|
    puts block.text if block.is_a?(Anthropic::Models::TextBlock)
  end
  ```
</CodeGroup>

## 处理每个响应中的思考

思考会在每个 Claude Opus 5.5 请求上运行，因此每个响应都可能以 `thinking` 块开头，并且 `max_tokens` 涵盖思考和文本。如果您的代码已经在开启思考的情况下运行，那么第 1 到 3 项很可能已经就绪：请检查第 4 和第 5 项。如果您的代码在任何早期模型上都是在不开启思考的情况下运行的，那么每一项都是一项更改。

1. **`max_tokens` 涵盖思考和文本：** 在 Claude Opus 4.8 及更早的 Opus 模型上，没有 `thinking` 字段的请求在不思考的情况下运行。Claude Opus 5 和 Claude Sonnet 5 接受 `thinking: {"type": "disabled"}`。在 Claude Opus 5.5 上，每个请求都使用[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)运行。`max_tokens` 仍然是总输出（思考加响应文本）的硬性限制，因此请为之前不开启思考运行的工作负载重新审视它。即使思考文本没有返回给您，思考令牌也会按输出令牌计费，因此此类工作负载每个请求可能会产生更多输出令牌。请参阅[成本控制](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost#cost-control)。要减少用于思考的令牌，请降低 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别。如果您以 `xhigh` 或 `max` effort 运行，请设置较大的 `max_tokens`，让模型有空间进行思考和行动；从 64k 令牌开始，并在此基础上调整。如果您的提示是针对不开启思考的运行方式调优的，请参阅[为禁用思考编写的提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#prompts-written-for-thinking-disabled)。

2. **响应以思考块开头：** 响应可能在第一个 `text` 块之前以一个或多个 `thinking` 块开头。按位置读取回复的代码（例如 `content[0].text`，或将第一个 `content_block_start` 事件视为文本的流处理程序）在这些响应上会出错。请改为按 `type` 字段选择内容块：从 `type` 为 `"text"` 的块中读取 `text`，并在处理流事件时按块类型进行分支。

3. **在工具使用循环中原样返回思考块：** 如果您运行工具使用循环，在返回工具结果时，请将每个助手响应中的 `thinking` 块完整且未经修改地传回 API，包括 `thinking` 字段为空的块。请按接收时的原样回传助手消息，而不是按类型过滤其内容块或重新构建它：API 会以 400 错误拒绝被编辑、重新排序或部分丢弃的思考块。请参阅[保留思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserving-thinking-blocks)。

4. **默认省略思考文本：** `thinking.display` 默认为 `"omitted"`，因此 `thinking` 块到达时 `thinking` 字段为空，并附带其 `signature`。请仅将 `thinking` 字段视为显示文本。要改为接收可读的摘要，请将 `thinking.display` 设置为 `"summarized"`：

   <CodeGroup exclude="shell">
     ```python Python
     thinking = {
         "type": "adaptive",
         "display": "summarized",
     }
     ```

     ```typescript TypeScript
     const thinking = {
       type: "adaptive",
       display: "summarized"
     };
     ```

     ```csharp C#
     var thinking = new ThinkingConfigAdaptive { Display = Display.Summarized };
     ```

     ```go Go
     thinking := anthropic.ThinkingConfigParamUnion{
     	OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{
     		Display: anthropic.ThinkingConfigAdaptiveDisplaySummarized,
     	},
     }
     ```

     ```java Java
     ThinkingConfigAdaptive thinking = ThinkingConfigAdaptive.builder()
         .display(ThinkingConfigAdaptive.Display.SUMMARIZED)
         .build();
     ```

     ```php PHP
     $thinking = ['type' => 'adaptive', 'display' => 'summarized'];
     ```

     ```ruby Ruby
     thinking = {
       type: "adaptive",
       display: "summarized"
     }
     ```
   </CodeGroup>

   如果您的产品会向用户流式传输推理过程，默认设置会表现为输出开始前的长时间停顿；请设置 `display: "summarized"` 以在思考期间恢复可见的进度。请参阅[控制思考显示](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#controlling-thinking-display)。

5. **工具调用之间的文本以思考块返回：** 模型在工具调用之间编写的简短说明以 `thinking` 块的形式返回，在默认显示设置下这些块为空。请参阅[工具调用之间的文本以思考块返回](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#text-between-tool-calls)。

## 按起始模型划分的迁移检查清单

从上往下逐组查看，并在列出您当前模型的那一组之后停止：到该组为止的每一项都适用于您。如果您使用的是 Claude Opus 5，第一组就是完整的清单。如果您使用的是 Claude Sonnet 5，请应用第一组和最后一组。

### 所有起始模型

* 将模型 ID 更新为 `claude-opus-5-5`。
* 移除 `thinking: {"type": "disabled"}` 和 `thinking: {"type": "enabled", ...}`；改为选择一个 effort 级别。
* 显式设置 `effort`：默认值为 `medium`，而 Claude Opus 5 的默认值为 `high`。
* 将 `tool_choice` 类型 `any` 和 `tool` 替换为 `auto` 加上严格工具使用或结构化输出。
* 如果您使用较早的计算机使用工具版本，请改为声明 `computer_toolset_20260801`，删除该版本的 beta 标头，并针对该工具集更新您的代理循环。在 Amazon Bedrock 上，`computer_20251124` 也可以使用。请参阅[计算机使用的破坏性变更](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#computer-use-toolset)。
* 如果路由器或回退机制可能将对话从 Claude Opus 5.5 转移出去，请预期新模型在运行时不会带有 Claude Opus 5.5 的思考块。Claude Fable 5.1 和 Claude Mythos 5.1 在 Claude API 上会保留这些思考块。Claude Fable 5.2 和 Claude Mythos 5.2 在 Claude API 和 Claude Models on Google Cloud (Google managed) 上会保留这些思考块。Claude Opus 5.5 本身可以读取来自 Claude Opus 5 以及来自较早的 Opus、Sonnet 和 Haiku 模型的思考。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，它还可以读取来自 Claude Sonnet 5.5 和 Claude Haiku 5.5 的思考。它不能读取来自 Claude Fable 或 Claude Mythos 模型的思考。
* 按 `type` 读取内容块，并在工具使用循环中原样传回 `thinking` 块。
* 如果您的界面会渲染工具调用之间的文本，请设置 `display: "updates"`（beta）或 `"summarized"`，并渲染非空的 `thinking` 块。
* 如果您的代码会在对话中途编辑较早的轮次、`system` 提示或 `tools`，请遵循[保留思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)。
* 处理 `stop_reason: "refusal"` 并配置回退。
* 在您选择的 effort 级别下重新建立成本和延迟基线。
* 如果您的代码曾禁用思考，请重新审视 `max_tokens`，它涵盖思考和响应文本；在 `xhigh` 或 `max` effort 下，从 64k 开始。请参阅[在每个响应中处理思考](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-in-every-response)。

### Claude Opus 4.8 或更早版本

* 审查之前在没有 `thinking` 字段的情况下运行的工作负载：在 Claude Opus 5.5 上，它们会开启思考运行，并且思考无法禁用。重新审视 `max_tokens`，它仍然是总输出（思考加响应文本）的硬性限制；在您希望减少思考的地方降低 `effort`。思考令牌按输出令牌计费，因此这些工作负载每个请求可能会产生更多输出令牌。
* 验证所有解析 `thinking` 字段的代码都仅将其视为显示文本。设置 `display: "summarized"` 以接收可读的摘要。
* 审查接近缓存最小长度的提示：512 个令牌或更多的提示可以创建缓存条目。
* 如果您的组织有 [Priority Tier](https://platform.claude.com/docs/zh-CN/api/service-tiers#supported-models) 承诺，请单独规划容量：Claude Opus 5.5 不支持 Priority Tier。
* 如果您以 `xhigh` 或 `max` effort 运行，请将 `max_tokens` 提高到至少 64k 作为起点。
* 对于智能体工作负载，请考虑使用[任务预算](https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets)和对话中途工具更改。两者均处于 beta 阶段。

### Claude Opus 4.7 或更早版本

* 在您自己的评估上重新进行 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 扫描，而不是沿用为早期模型调优的设置。
* 移除所有上下文窗口 beta 标头。
* 如果您通过重建对话历史来更新指令，请考虑改用对话中途的系统消息，以保留提示缓存命中。
* 验证您的停止原因处理逻辑在拒绝时会读取 `stop_details`。
* 如果您想使用 Claude Opus 4.7 会拒绝的快速模式，请在您的 Claude API 请求上设置 `speed: "fast"`。

### Claude Opus 4.6 或更早版本

* 从请求负载中移除 `temperature`、`top_p` 和 `top_k`。
* 将 `thinking: {"type": "enabled", "budget_tokens": N}` 替换为 `thinking: {"type": "adaptive"}` 加[工作量参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)，或完全移除 `thinking` 字段；自适应思考始终开启。
* 如果您的 UI 显示思考内容，请显式选择启用思考摘要。
* 在更新后的分词方式下重新对端到端成本和延迟进行基准测试。
* 重新调整 `max_tokens` 以适应更新后的分词方式，包括压缩触发条件。
* 重新测试所有客户端令牌计数估算。
* 如果您的应用程序发送图像，请针对[高分辨率图像支持](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#high-resolution-image-support-on-claude-opus-4-7)重新制定预算（每张全分辨率图像最多约多 3 倍的图像令牌）。如果您不需要额外的保真度，请在发送前降低采样率。
* 如果您使用模型输出的指点或边界框坐标，请移除所有缩放因子转换；在 Claude Opus 4.7 及更高版本的模型上，坐标与实际图像像素为 1:1 对应。
* 审查从 Claude Opus 4.7 开始的[行为变化](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#behavior-changes)。
* 如果您的产品从事合法的安全工作，请申请加入 [Cyber Verification Program](https://support.claude.com/en/articles/14604842-real-time-cyber-safeguards-on-claude-opus-and-sonnet)，以获得对网络安全内容更宽松的限制。

### Claude Opus 4.5 或更早版本

* 移除所有助手消息预填充；Claude Opus 4.6 已经会拒绝它们。
* 验证工具调用 JSON 解析使用标准 JSON 解析器。
* 从 `client.beta.messages.create()` (csharp: `client.Beta.Messages.Create()`; go: `client.Beta.Messages.New()`; java: `client.beta().messages().create()`; php: `$client->beta->messages->create()`; cli: `ant beta:messages create`) 迁移到 `client.messages.create()` (csharp: `client.Messages.Create()`; go: `client.Messages.New()`; java: `client.messages().create()`; php: `$client->messages->create()`; cli: `ant messages create`)：自适应思考和 effort 不需要 beta 命名空间。
* 移除 `effort-2025-11-24` beta 标头（effort 参数不需要它）。
* 移除 `fine-grained-tool-streaming-2025-05-14` beta 标头。
* 移除 `interleaved-thinking-2025-05-14` beta 标头（自适应思考会自动启用交错思考）。
* 将 `output_format` 迁移到 `output_config.format`（如适用）。

### Claude 4.1 或更早版本

* 更新工具版本（`text_editor_20250728`、`code_execution_20260521`）。
* 处理 `refusal` 停止原因。
* 处理 `model_context_window_exceeded` 停止原因。
* 验证工具字符串参数对尾随换行符的处理。
* 移除 `token-efficient-tools-2025-02-19` 和 `output-128k-2025-02-19` beta 标头。
* 按照[提示最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)审查并更新提示。

### 仅限 Claude Sonnet 5

* 如果您通过重建对话历史来更新指令，请考虑改用对话中途的系统消息，以保留提示缓存命中。
* 审查接近缓存最小长度的提示：512 个令牌或更多的提示可以创建缓存条目。

## 从 Claude Opus 5 迁移到 Claude Opus 5.5

首先完成[每个发往 Claude Opus 5.5 的请求必须满足的要求](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#request-requirements)和[处理每个响应中的思考](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-in-every-response)。每个起始模型都需要本节中的更改。它们是 Claude Opus 5.5 会拒绝的请求设置，以及随之而来的响应变化。本节的检查清单是[迁移检查清单](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migration-checklist)的第一组。

### 更新您的模型名称

```python
model = "claude-opus-5"  # Before
model = "claude-opus-5-5"  # After
```

`claude-opus-5-5` 是一个不带日期后缀的固定模型 ID，与 `claude-opus-5` 采用相同的命名方案。在 Amazon Bedrock、Claude Platform on AWS、Claude Models on Google Cloud (Anthropic managed)、Claude Models on Google Cloud (Google managed) 和 Microsoft Foundry 上，请使用该平台的模型 ID；请参阅[可用性](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#availability)。

### 破坏性变更

每项变更都在《Claude Opus 5.5 的新功能》中的[破坏性变更](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#breaking-changes)下进行了说明；本节给出每项变更对应的代码更改。

#### 思考无法禁用

`thinking: {"type": "disabled"}` 和 `thinking: {"type": "enabled", "budget_tokens": N}` 都会返回 400 错误（`"thinking.type.disabled" is not supported for this model.` 或 `"thinking.type.enabled" is not supported for this model.`）。移除 `thinking` 字段并选择一个 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别；在您之前为节省令牌而禁用思考的地方，请使用较低的级别。随后响应会以 `thinking` 块开头，因此请按 `type` 选择内容块，并将 `thinking` 块与工具结果一起原样传回。请参阅[思考无法禁用](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#thinking-cant-be-disabled)。

之前。Claude Opus 5 接受此请求，而 Claude Opus 5.5 会以 400 错误拒绝它：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5",
      "max_tokens": 16000,
      "thinking": {"type": "disabled"},
      "messages": [{"role": "user", "content": "..."}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5 \
    --max-tokens 16000 \
    --thinking '{type: disabled}' \
    --message '{role: user, content: "..."}'
  ```

  ```python Python
  client.messages.create(
      model="claude-opus-5",
      max_tokens=16000,
      thinking={"type": "disabled"},
      messages=[{"role": "user", "content": "..."}],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-opus-5",
    max_tokens: 16000,
    thinking: { type: "disabled" },
    messages: [{ role: "user", content: "..." }]
  });
  ```

  ```csharp C#
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5,
      MaxTokens = 16000,
      Thinking = new ThinkingConfigDisabled(),
      Messages = [new UserMessageParam { Content = "..." }],
  });
  ```

  ```go Go
  client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5,
  	MaxTokens: 16000,
  	Thinking: anthropic.ThinkingConfigParamUnion{
  		OfDisabled: &anthropic.ThinkingConfigDisabledParam{},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("...")),
  	},
  })
  ```

  ```java Java
  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5)
      .maxTokens(16000L)
      .thinking(ThinkingConfigDisabled.builder().build())
      .addUserMessage("...")
      .build();

  client.messages().create(params);
  ```

  ```php PHP
  $client->messages->create(
      model: Model::CLAUDE_OPUS_5,
      maxTokens: 16000,
      thinking: ThinkingConfigDisabled::with(),
      messages: [['role' => 'user', 'content' => '...']],
  );
  ```

  ```ruby Ruby
  client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5,
    max_tokens: 16000,
    thinking: Anthropic::ThinkingConfigDisabled.new,
    messages: [{ role: "user", content: "..." }]
  )
  ```
</CodeGroup>

之后：

<CodeGroup>
  ```bash cURL
  # 思考始终开启；通过 effort 进行控制
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 16000,
      "output_config": {"effort": "low"},
      "messages": [{"role": "user", "content": "..."}]
    }'
  ```

  ```bash CLI
  # 思考始终开启；通过 effort 进行控制
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 16000 \
    --output-config '{effort: low}' \
    --message '{role: user, content: "..."}'
  ```

  ```python Python
  client.messages.create(
      model="claude-opus-5-5",
      max_tokens=16000,
      output_config={"effort": "low"},  # thinking is always on; effort is the control
      messages=[{"role": "user", "content": "..."}],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 16000,
    output_config: { effort: "low" }, // thinking is always on; effort is the control
    messages: [{ role: "user", content: "..." }]
  });
  ```

  ```csharp C#
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 16000,
      OutputConfig = new() { Effort = Effort.Low }, // thinking is always on; effort is the control
      Messages = [new UserMessageParam { Content = "..." }],
  });
  ```

  ```go Go
  client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 16000,
  	OutputConfig: anthropic.OutputConfigParam{
  		Effort: anthropic.OutputConfigEffortLow, // thinking is always on; effort is the control
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("...")),
  	},
  })
  ```

  ```java Java
  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(16000L)
      // 思考始终开启；由 effort 控制
      .outputConfig(OutputConfig.builder()
          .effort(OutputConfig.Effort.LOW)
          .build())
      .addUserMessage("...")
      .build();

  client.messages().create(params);
  ```

  ```php PHP
  $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 16000,
      // 思考始终开启；通过 effort 进行控制
      outputConfig: OutputConfig::with(effort: Effort::LOW),
      messages: [['role' => 'user', 'content' => '...']],
  );
  ```

  ```ruby Ruby
  client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 16000,
    # 思考始终开启；通过 effort 进行控制
    output_config: { effort: Anthropic::OutputConfig::Effort::LOW },
    messages: [{ role: "user", content: "..." }]
  )
  ```
</CodeGroup>

#### 不支持强制工具使用

`tool_choice` 类型 `any` 和 `tool` 会返回 400 错误（`tool_choice: type "tool" and "any" are not supported for this model.`），令牌计数端点也是如此。请使用 `auto` 配合[严格工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)或[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)，并在提示中说明何时应使用该工具。严格工具使用接受 JSON Schema 的一个子集，因此在添加 `strict: true` 之前，请检查每个工具的 `input_schema`。模式中的每个对象都必须设置 `additionalProperties: false`；请参阅 [JSON Schema 限制](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#json-schema-limitations)。请参阅[不支持强制工具使用](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#forced-tool-use-is-not-supported)。

之前。Claude Opus 5 接受此请求，而 Claude Opus 5.5 会以 400 错误拒绝它：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5",
      "max_tokens": 1024,
      "tools": [{
        "name": "get_weather",
        "description": "Get the current weather in a given location",
        "input_schema": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The city and state, e.g. San Francisco, CA"
            }
          },
          "required": ["location"],
          "additionalProperties": false
        }
      }],
      "tool_choice": {"type": "tool", "name": "get_weather"},
      "messages": [{"role": "user", "content": "What is the weather in Paris?"}]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5
  max_tokens: 1024
  tools:
    - name: get_weather
      description: Get the current weather in a given location
      input_schema:
        type: object
        properties:
          location:
            type: string
            description: The city and state, e.g. San Francisco, CA
        required: [location]
        additionalProperties: false
  tool_choice:
    type: tool
    name: get_weather
  messages:
    - role: user
      content: What's the weather in Paris?
  YAML
  ```

  ```python Python
  client.messages.create(
      model="claude-opus-5",
      max_tokens=1024,
      tools=tools,
      tool_choice={"type": "tool", "name": "get_weather"},
      messages=[{"role": "user", "content": "What's the weather in Paris?"}],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-opus-5",
    max_tokens: 1024,
    tools,
    tool_choice: { type: "tool", name: "get_weather" },
    messages: [{ role: "user", content: "What's the weather in Paris?" }]
  });
  ```

  ```csharp C#
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5,
      MaxTokens = 1024,
      Tools = [.. tools],
      ToolChoice = new ToolChoiceTool { Name = "get_weather" },
      Messages = [new UserMessageParam { Content = "What's the weather in Paris?" }],
  });
  ```

  ```go Go
  client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:      anthropic.ModelClaudeOpus5,
  	MaxTokens:  1024,
  	Tools:      tools,
  	ToolChoice: anthropic.ToolChoiceParamOfTool("get_weather"),
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("What's the weather in Paris?")),
  	},
  })
  ```

  ```java Java
  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5)
      .maxTokens(1024L)
      .tools(tools)
      .toolChoice(ToolChoiceTool.of("get_weather"))
      .addUserMessage("What's the weather in Paris?")
      .build();

  client.messages().create(params);
  ```

  ```php PHP
  $client->messages->create(
      model: Model::CLAUDE_OPUS_5,
      maxTokens: 1024,
      tools: $tools,
      toolChoice: ToolChoiceTool::with(name: 'get_weather'),
      messages: [['role' => 'user', 'content' => "What's the weather in Paris?"]],
  );
  ```

  ```ruby Ruby
  client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5,
    max_tokens: 1024,
    tools: tools,
    tool_choice: Anthropic::ToolChoiceTool.new(name: "get_weather"),
    messages: [{ role: "user", content: "What's the weather in Paris?" }]
  )
  ```
</CodeGroup>

之后：

<CodeGroup>
  ```bash cURL
  # 严格工具使用：每次调用都与该工具的 input_schema 匹配
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "tools": [{
        "name": "get_weather",
        "description": "Get the current weather in a given location",
        "input_schema": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The city and state, e.g. San Francisco, CA"
            }
          },
          "required": ["location"],
          "additionalProperties": false
        },
        "strict": true
      }],
      "tool_choice": {"type": "auto"},
      "messages": [{
        "role": "user",
        "content": "What is the weather in Paris? Use the get_weather tool."
      }]
    }'
  ```

  ```bash CLI
  ant messages create <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  tools:
    - name: get_weather
      description: Get the current weather in a given location
      input_schema:
        type: object
        properties:
          location:
            type: string
            description: The city and state, e.g. San Francisco, CA
        required: [location]
        additionalProperties: false
      # 严格工具使用：每次调用都与工具的 input_schema 匹配
      strict: true
  tool_choice:
    type: auto
  messages:
    - role: user
      content: What's the weather in Paris? Use the get_weather tool.
  YAML
  ```

  ```python Python
  client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      # 严格工具使用：每次调用都与工具的 input_schema 匹配
      tools=[{**tool, "strict": True} for tool in tools],
      tool_choice={"type": "auto"},
      messages=[
          {
              "role": "user",
              "content": "What's the weather in Paris? Use the get_weather tool.",
          }
      ],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    // 严格工具使用：每次调用均符合该工具的 input_schema
    tools: tools.map((tool) => ({ ...tool, strict: true })),
    tool_choice: { type: "auto" },
    messages: [
      {
        role: "user",
        content: "What's the weather in Paris? Use the get_weather tool."
      }
    ]
  });
  ```

  ```csharp C#
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      // 严格工具使用：每次调用都符合工具的 input_schema
      Tools = [.. tools.Select(tool => tool with { Strict = true })],
      ToolChoice = new ToolChoiceAuto(),
      Messages =
      [
          new UserMessageParam { Content = "What's the weather in Paris? Use the get_weather tool." },
      ],
  });
  ```

  ```go Go
  // 严格工具使用：每次调用都符合工具的 input_schema
  var strictTools []anthropic.ToolUnionParam
  for _, tool := range tools {
  	strictTool := *tool.OfTool
  	strictTool.Strict = anthropic.Bool(true)
  	strictTools = append(strictTools, anthropic.ToolUnionParam{OfTool: &strictTool})
  }
  client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:      anthropic.ModelClaudeOpus5_5,
  	MaxTokens:  1024,
  	Tools:      strictTools,
  	ToolChoice: anthropic.ToolChoiceUnionParam{OfAuto: &anthropic.ToolChoiceAutoParam{}},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(
  			anthropic.NewTextBlock("What's the weather in Paris? Use the get_weather tool."),
  		),
  	},
  })
  ```

  ```java Java
  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024L)
      // 严格工具使用：每次调用都与工具的 input_schema 匹配
      .tools(tools.stream()
          .map(tool -> tool.tool()
              .map(customTool -> customTool.toBuilder()
                  .strict(true)
                  .build())
              .map(ToolUnion::ofTool)
              .orElse(tool))
          .toList())
      .toolChoice(ToolChoiceAuto.builder().build())
      .addUserMessage("What's the weather in Paris? Use the get_weather tool.")
      .build();

  client.messages().create(params);
  ```

  ```php PHP
  $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 1024,
      // 严格工具使用：每次调用都符合该工具的 input_schema
      tools: array_map(fn (Tool $tool) => $tool->withStrict(true), $tools),
      toolChoice: ToolChoiceAuto::with(),
      messages: [
          [
              'role' => 'user',
              'content' => "What's the weather in Paris? Use the get_weather tool.",
          ],
      ],
  );
  ```

  ```ruby Ruby
  client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    # 严格工具使用：每次调用都与工具的 input_schema 匹配
    tools: tools.map { |tool| tool.merge(strict: true) },
    tool_choice: Anthropic::ToolChoiceAuto.new,
    messages: [
      { role: "user", content: "What's the weather in Paris? Use the get_weather tool." }
    ]
  )
  ```
</CodeGroup>

#### 思考块与模型和对话绑定

Claude Fable 5.1 和 Claude Mythos 5.1 可以在 Claude API 上读取 Claude Opus 5.5 的 thinking block（思考块）。Claude Fable 5.2 和 Claude Mythos 5.2 可以在 Claude API 和 Claude Models on Google Cloud (Google managed) 上读取这些思考块。其他模型均不能。将对话从 Claude Opus 5.5 转移到任何其他模型的路由器或回退机制，在运行这些轮次时将不包含这些思考块。

反过来，Claude Opus 5.5 可以读取来自 Claude Opus 5 以及更早的 Opus、Sonnet 和 Haiku 模型的思考块。在 Claude API 和 Claude Models on Google Cloud (Google managed) 上，它还可以读取来自 Claude Sonnet 5.5 和 Claude Haiku 5.5 的思考块。它不能读取来自 Claude Fable 或 Claude Mythos 模型的思考块。

请保持对话仅追加（在对话中途不编辑 `system` 提示、`tools` 或之前的消息），以使这些思考块保持有效。Claude Code、claude.ai、Claude Managed Agents 和 Claude Agent SDK 已经这样做了。如果请求在此类编辑之后重放思考块，且未设置 `thinking.block_binding.prefix_mismatch_behavior`，则会收到 400 错误。这适用于在 2026 年 8 月 31 日 00:00 UTC 或之后创建的账户。[保留的思考的可用性](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#feature-availability)列出了适用此规则的平台。对于仅追加的集成，无需更改代码。请参阅[思考块与模型和对话绑定](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5#thinking-blocks-are-tied-to-the-model-that-produced-them)和[保留的思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)。

#### 部分计算机使用工具版本不受支持

Claude Opus 5.5 通过 `computer_toolset_20260801` 工具集支持计算机使用。在 Amazon Bedrock 上，Claude Opus 5.5 还接受带有 `computer-use-2025-11-24` beta 标头的 `computer_20251124` 工具。对于任何不接受的计算机使用工具版本（例如 Claude API 上的 `computer_20251124`），Claude Opus 5.5 会返回 400 错误。它在任何平台上都不接受 `computer_20250124`。在 Claude API 上，错误消息以 `'claude-opus-5-5' does not support tool types: computer_20251124.` 开头，随后列出该模型接受的工具类型。

使用该工具集的集成无需更改。要将 `computer_20251124` 集成迁移到该工具集，请删除 beta 标头，并发送不带 `name` 或显示尺寸的 `computer_toolset_20260801` 条目。在您的代理循环中，处理一个轮次中的每个成员 `tool_use` 块，并在每个结果中回传 `toolset_name`。从每个块的 `name` 读取操作，而不是从 `input.action` 读取。[从 `computer_20251124` 迁移](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#migrate-from-computer-20251124)列出了代理循环的相关变更。

之前。Claude Opus 5 接受此请求，而 Claude Opus 5.5 在 Claude API 上会以 400 错误拒绝它：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: computer-use-2025-11-24" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5",
      "max_tokens": 4096,
      "tools": [{
        "type": "computer_20251124",
        "name": "computer",
        "display_width_px": 1024,
        "display_height_px": 768
      }],
      "messages": [{"role": "user", "content": "Open the display settings."}]
    }'
  ```

  ```bash CLI
  ant beta:messages create --beta computer-use-2025-11-24 <<'YAML'
  model: claude-opus-5
  max_tokens: 4096
  tools:
    - type: computer_20251124
      name: computer
      display_width_px: 1024
      display_height_px: 768
  messages:
    - role: user
      content: Open the display settings.
  YAML
  ```

  ```python Python
  client.beta.messages.create(
      model="claude-opus-5",
      max_tokens=4096,
      betas=["computer-use-2025-11-24"],
      tools=[
          {
              "type": "computer_20251124",
              "name": "computer",
              "display_width_px": 1024,
              "display_height_px": 768,
          }
      ],
      messages=[{"role": "user", "content": "Open the display settings."}],
  )
  ```

  ```typescript TypeScript
  await client.beta.messages.create({
    model: "claude-opus-5",
    max_tokens: 4096,
    betas: ["computer-use-2025-11-24"],
    tools: [
      {
        type: "computer_20251124",
        name: "computer",
        display_width_px: 1024,
        display_height_px: 768
      }
    ],
    messages: [{ role: "user", content: "Open the display settings." }]
  });
  ```

  ```csharp C#
  await client.Beta.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5,
      MaxTokens = 4096,
      Betas = [AnthropicBeta.ComputerUse2025_11_24],
      Tools =
      [
          new BetaToolComputerUse20251124
          {
              DisplayWidthPx = 1024,
              DisplayHeightPx = 768,
          },
      ],
      Messages = [new BetaUserMessageParam { Content = "Open the display settings." }],
  });
  ```

  ```go Go
  client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5,
  	MaxTokens: 4096,
  	Betas:     []anthropic.AnthropicBeta{anthropic.AnthropicBetaComputerUse2025_11_24},
  	Tools: []anthropic.BetaToolUnionParam{
  		{OfComputerUseTool20251124: &anthropic.BetaToolComputerUse20251124Param{
  			DisplayWidthPx:  1024,
  			DisplayHeightPx: 768,
  		}},
  	},
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Open the display settings.")),
  	},
  })
  ```

  ```java Java
  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5)
      .maxTokens(4096L)
      .addBeta(AnthropicBeta.COMPUTER_USE_2025_11_24)
      .addTool(BetaToolComputerUse20251124.builder()
          .displayWidthPx(1024L)
          .displayHeightPx(768L)
          .build())
      .addUserMessage("Open the display settings.")
      .build();

  client.beta().messages().create(params);
  ```

  ```php PHP
  $client->beta->messages->create(
      model: Model::CLAUDE_OPUS_5,
      maxTokens: 4096,
      betas: [AnthropicBeta::COMPUTER_USE_2025_11_24],
      tools: [
          BetaToolComputerUse20251124::with(
              displayWidthPx: 1024,
              displayHeightPx: 768,
          ),
      ],
      messages: [['role' => 'user', 'content' => 'Open the display settings.']],
  );
  ```

  ```ruby Ruby
  client.beta.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5,
    max_tokens: 4096,
    betas: [Anthropic::AnthropicBeta::COMPUTER_USE_2025_11_24],
    tools: [
      Anthropic::Beta::BetaToolComputerUse20251124.new(
        name: :computer,
        display_width_px: 1024,
        display_height_px: 768
      )
    ],
    messages: [{ role: "user", content: "Open the display settings." }]
  )
  ```
</CodeGroup>

之后：

<CodeGroup>
  ```bash cURL
  # 无需 beta 标头；工具集条目不接受名称或显示尺寸
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 4096,
      "tools": [{"type": "computer_toolset_20260801"}],
      "messages": [{"role": "user", "content": "Open the display settings."}]
    }'
  ```

  ```bash CLI
  # 无需 beta 标头；工具集条目不接受名称或显示尺寸
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 4096 \
    --tool '{type: computer_toolset_20260801}' \
    --message '{role: user, content: "Open the display settings."}'
  ```

  ```python Python
  client.messages.create(
      model="claude-opus-5-5",
      max_tokens=4096,
      # 无需 beta 标头；工具集条目不接受名称或显示尺寸
      tools=[{"type": "computer_toolset_20260801"}],
      messages=[{"role": "user", "content": "Open the display settings."}],
  )
  ```

  ```typescript TypeScript
  await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 4096,
    // 无需 beta 标头；工具集条目不接受名称或显示尺寸
    tools: [{ type: "computer_toolset_20260801" }],
    messages: [{ role: "user", content: "Open the display settings." }]
  });
  ```

  ```csharp C#
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 4096,
      // 无需 beta 标头；工具集条目不接受名称或显示尺寸
      Tools = [new ComputerToolset20260801()],
      Messages = [new UserMessageParam { Content = "Open the display settings." }],
  });
  ```

  ```go Go
  client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 4096,
  	// 无需 beta 标头；工具集条目不接受名称或显示尺寸
  	Tools: []anthropic.ToolUnionParam{
  		{OfComputerToolset20260801: &anthropic.ComputerToolset20260801Param{}},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Open the display settings.")),
  	},
  })
  ```

  ```java Java
  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(4096L)
      // 无需 beta 标头；工具集条目不接受名称或显示尺寸
      .addTool(ComputerToolset20260801.builder().build())
      .addUserMessage("Open the display settings.")
      .build();

  client.messages().create(params);
  ```

  ```php PHP
  $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 4096,
      // 无需 beta 标头；工具集条目不接受名称或显示尺寸
      tools: [ComputerToolset20260801::with()],
      messages: [['role' => 'user', 'content' => 'Open the display settings.']],
  );
  ```

  ```ruby Ruby
  client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 4096,
    # 无需 beta 标头；工具集条目不接受名称或显示尺寸
    tools: [Anthropic::ComputerToolset20260801.new],
    messages: [{ role: "user", content: "Open the display settings." }]
  )
  ```
</CodeGroup>

### 工具调用之间的文本以思考块返回

在 Claude Opus 5 上，模型在工具调用之间编写的文本以 `text` 块的形式返回。在 Claude Opus 5.5 上，与 Claude Fable 5.1 一样，这些叙述以[进度更新 `thinking` 块](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#progress-updates)的形式返回，每次工具调用之前最多一个。在 `thinking.display` 的默认值 `"omitted"` 下，它们的 `thinking` 字段为空。不会有请求失败，但将这些文本作为进度更新流式传输给用户的应用程序在工具调用之间会变得沉默。要恢复这些更新，请从 `thinking` 块中读取它们，并设置一个会返回其文本的 `display` 值：`"updates"`（beta，需 `thinking-display-updates-2026-08-18` 标头）返回进度更新而推理保持隐藏，`"summarized"` 则将两者混合一起返回。然后，将每个非空 `thinking` 块渲染在紧随其后的 `tool_use` 块之前，并将这些块与助手轮次的其余部分一起原样传回。请参阅[面向用户的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#user-facing-progress-updates)。

### 安全分类器和回退

Claude Opus 5.5 可能会返回带有 `stop_details` 类别的 `stop_reason: "refusal"`。它的分类器覆盖的类别范围比 Claude Opus 5 更广，因此除了 `"cyber"` 和 `"reasoning_extraction"` 之外，还可能出现 `"bio"` 等 `stop_details.category` 值；请参阅[拒绝类别表](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#refusal-response)。请处理拒绝并配置[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)或您自己的重试机制（服务器端回退不会重试以 `"reasoning_extraction"` 拒绝的请求；该拒绝会返回给您）；请参阅[拒绝和回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)和[安全防护拒绝](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#safeguard-refusals)。

### 推荐的更改

1. **重新进行 effort 扫描。** Effort 是 Claude Opus 5.5 上唯一的思考控制手段，其默认值为 `medium`，而 Claude Opus 5 的默认值为 `high`，因此省略 `effort` 的请求会以 `medium` 运行。在质量能够保持的地方降低级别，对要求最高的工作则提高级别。请参阅 [Effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)。
2. **重新评估特定于模型的提示指令。** 针对 Claude Opus 5 行为调优的指令可能不再需要；请参阅[提示 Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5)。如果您之前在禁用思考的情况下运行，另请参阅[为禁用思考编写的提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#prompts-written-for-thinking-disabled)。
3. 在切换生产流量之前，**在开发环境中进行测试**。

## 从 Claude Opus 4.8 迁移到 Claude Opus 5.5

首先完成[每个发往 Claude Opus 5.5 的请求必须满足的要求](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#request-requirements)、[处理每个响应中的思考](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-in-every-response)和[从 Claude Opus 5 迁移到 Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-5)。将 `claude-opus-4-8` 作为您要替换的模型 ID。最后那一节原样适用于 Claude Opus 4.8 上的代码，因为 Claude Opus 4.8 与 Claude Opus 5 一样：

* 接受 `thinking: {"type": "disabled"}`、强制工具选择和 `computer_20251124` 工具。
* 以 `text` 块返回工具调用之间的文本。
* 默认使用 `high` effort。

本节补充了 Claude Opus 4.8 与 Claude Opus 5 之间的变化。有关检查清单，请参阅[迁移检查清单](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migration-checklist)的前两组。

### 变化内容

1. **省略思考的请求也会运行思考：** 在 Claude Opus 4.8 上，除非您请求，否则思考是关闭的。在 Claude Opus 5.5 上，没有 `thinking` 字段的请求会开启思考运行，因此[处理每个响应中的思考](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-in-every-response)中的每一项对于此类代码都是一项更改。如果您的代码从未发送过 `thinking` 字段，那么在[思考设置的前后对比](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-cant-be-disabled)下就没有需要移除的内容。

2. **更低的提示缓存最小长度：** Claude Opus 5.5 上可缓存提示的最小长度为 512 个令牌，低于 Claude Opus 4.8 上的 1,024 个令牌。在 Claude Opus 4.8 上因过短而无法缓存的提示现在可以创建缓存条目，无需更改代码。有关各模型的最小长度，请参阅[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#cache-limitations)。

3. **不支持 Priority Tier：** Claude Opus 5.5 不支持 [Priority Tier](https://platform.claude.com/docs/zh-CN/api/service-tiers#supported-models)，而 Claude Opus 4.8 保留了该支持。如果您的组织有 Priority Tier 承诺，请单独规划容量。

### 推荐的更改

这些更改是可选的，可以改善您的使用体验：

1. **考虑任务预算（beta）：** 对于智能体工作负载，[任务预算](https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets)会告诉模型在完整的智能体循环中可用的令牌数量。它们需要 `task-budgets-2026-03-13` beta 标头。

2. **考虑对话中途的工具更改（beta）：** 对话中途的工具更改允许您在对话的轮次之间添加或移除工具，而不会使早期轮次的[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)命中失效。更改 `tools` 数组本身会使缓存的前缀失效。在 Claude API、Claude Platform on AWS 和 Claude Models on Google Cloud (Anthropic managed) 上，请发送 `inline-tools-2026-09-15` beta 标头。`mid-conversation-tool-changes-2026-07-01` 标头适用于通过引用指定工具名称的更改，支持 Claude API、Amazon Bedrock、Claude Platform on AWS、Claude Models on Google Cloud (Anthropic managed) 和 Claude Models on Google Cloud (Google managed)。

## 从 Claude Opus 4.7 迁移到 Claude Opus 5.5

首先完成[每个发往 Claude Opus 5.5 的请求必须满足的要求](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#request-requirements)、[处理每个响应中的思考](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-in-every-response)、[从 Claude Opus 5 迁移到 Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-5)和[从 Claude Opus 4.8 迁移到 Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-4-8)。将 `claude-opus-4-7` 作为您要替换的模型 ID。这些章节原样适用于 Claude Opus 4.7 上的代码。与 Claude Opus 4.8 一样，它接受 `thinking: {"type": "disabled"}`、强制工具选择和 `computer_20251124` 工具。它默认使用 `high` effort，并且除非您请求，否则不开启思考运行。

本节补充了 Claude Opus 4.7 之后的变化。如果您的代码使用的是 Claude Opus 4.6 或更早版本，请在本节之后继续阅读[从 Claude Opus 4.6 及更早的 Opus 模型迁移到 Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-46)。该节补充了在 Claude Opus 4.7 中生效的破坏性变更。有关检查清单，请参阅[迁移检查清单](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migration-checklist)的前三组。

### 变化内容

以下各项都不会在前面章节的基础上新增破坏性变更；在您替换模型 ID 之后，值得对它们进行检查。

1. **重新校准的 effort 级别：** 与 Claude Opus 4.7 相比，Claude Opus 5.5 上每个 effort 级别背后的令牌分配有所变化。默认值为 `medium`，而 Claude Opus 4.7 的默认值为 `high`。请在您自己的评估上重新进行 effort 扫描，而不是沿用为 Claude Opus 4.7 调优的设置。请参阅 [Effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)。

2. **1M 上下文窗口为默认值：** Claude Opus 5.5 默认提供完整的 1M 令牌 “context window”（[上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)）。如果您的客户端为了兼容旧模型而传递了上下文窗口 beta 标头，请将其移除。

3. **对话中途的系统消息：** 在 Claude API、Amazon Bedrock、Claude Platform on AWS、Claude Models on Google Cloud (Anthropic managed) 和 Claude Models on Google Cloud (Google managed) 上，Claude Opus 5.5 接受在 `messages` 数组中紧跟在用户轮次之后的 `role: "system"` 消息（需遵守[放置规则](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#limitations)）。对于从一开始就适用的指令，请使用顶层 `system` 字段。Claude Opus 4.7 会以 400 错误拒绝 `messages` 中的 `role: "system"`。如果您维护着通过重建完整消息历史来更新指令的代码路径，您可以简化它们，并保留早期轮次的[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)命中。

4. **拒绝停止详情：** 当模型拒绝请求时，Claude Opus 5.5 会在返回 `refusal` 停止原因的同时，返回一个指明拒绝类别的 `stop_details` 对象。Claude Opus 4.7 也返回相同的对象，因此只有当您的停止原因处理逻辑尚未读取该对象时，这一点才需要关注；如果是这样，请参阅[停止原因与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons)。无法选择退出。Claude Opus 5.5 会在更多类别中拒绝请求；请参阅[安全分类器和回退](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#safety-classifiers-and-fallback)。

5. **快速模式：** Claude Opus 5.5 在 Claude API 上支持[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)。快速模式在 Claude Opus 4.7 上不可用，在该模型上带有 `speed: "fast"` 的请求会返回错误。请在您的请求中设置 `speed: "fast"`。

6. **计算机使用工具集和浏览器使用工具：** Claude Opus 5.5 支持以 `computer_toolset_20260801` 工具集形式提供的[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)，以及用于网页内任务的[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)。Claude Opus 4.7 两者均不支持。要将 `computer_20251124` 集成迁移到该工具集，请参阅[计算机使用的破坏性变更](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#computer-use-toolset)。

## 从 Claude Opus 4.6 及更早的 Opus 模型迁移到 Claude Opus 5.5

首先按页面顺序完成前面的每一个部分。它们是[每个发送到 Claude Opus 5.5 的请求必须满足的条件](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#request-requirements)、[在每个响应中处理思考内容](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-in-every-response)，以及 [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-5)、[Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-4-8) 和 [Claude Opus 4.7](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-47) 的相关部分。这些部分按原样适用于运行在 Claude Opus 4.6 上的代码。与 Claude Opus 4.7 一样，Claude Opus 4.6 接受 `thinking: {"type": "disabled"}`、强制工具选择以及 `computer_20251124` 工具。它默认使用 `high` effort，并且除非您明确要求，否则不会启用思考。Claude Opus 4.5 及更早的 Opus 模型同样接受 `thinking: {"type": "disabled"}` 和强制工具选择，并且除非您明确要求，否则不会启用思考，因此这些部分也适用于它们。

本部分补充了 Claude Opus 4.7 中的变更，您需要替换的模型 ID 为 `claude-opus-4-6`。其两个子部分补充了在此之前的变更，分别面向使用 [Claude Opus 4.5 或更早版本](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-45)和 [Claude 4.1 或更早版本](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-4-1-or-earlier)的读者。如需检查清单，请参阅[迁移检查清单](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migration-checklist)，直至列出您所用模型的那一组。

### 破坏性变更

1. **扩展思考已移除：** Claude Opus 4.7 及更高版本的模型不再支持 `thinking: {"type": "enabled", "budget_tokens": N}`，使用该配置将返回 400 错误。请改用 [adaptive thinking（自适应思考）](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)（`thinking: {"type": "adaptive"}`），并使用 [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)来控制思考深度。在 Claude Opus 5.5 上，自适应思考始终开启：`thinking: {"type": "adaptive"}` 是有效的，并且等同于完全省略 `thinking` 字段。

   之前（Claude Opus 4.6）：

   <CodeGroup>
     ```bash cURL
     curl https://api.anthropic.com/v1/messages \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "anthropic-version: 2026-10-21" \
       -H "content-type: application/json" \
       -d '{
         "model": "claude-opus-4-6",
         "max_tokens": 16000,
         "thinking": {
           "type": "enabled",
           "budget_tokens": 10000
         },
         "messages": [
           {
             "role": "user",
             "content": "..."
           }
         ]
       }'
     ```

     ```bash CLI
     ant messages create <<'YAML'
     model: claude-opus-4-6
     max_tokens: 16000
     thinking:
       type: enabled
       budget_tokens: 10000
     messages:
       - role: user
         content: "..."
     YAML
     ```

     ```python Python
     client.messages.create(
         model="claude-opus-4-6",
         max_tokens=16000,
         thinking={"type": "enabled", "budget_tokens": 10000},
         messages=[{"role": "user", "content": "..."}],
     )
     ```

     ```typescript TypeScript
     await client.messages.create({
       model: "claude-opus-4-6",
       max_tokens: 16000,
       thinking: { type: "enabled", budget_tokens: 10000 },
       messages: [{ role: "user", content: "..." }]
     });
     ```

     ```csharp C#
     AnthropicClient client = new();

     var parameters = new MessageCreateParams
     {
         Model = Model.ClaudeOpus4_6,
         MaxTokens = 16000,
         Thinking = new ThinkingConfigEnabled(budgetTokens: 10000),
         Messages = [new UserMessageParam { Content = "..." }]
     };

     var response = await client.Messages.Create(parameters);
     Console.WriteLine(response);
     ```

     ```go Go
     client := anthropic.NewClient()

     response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
     	Model:     anthropic.ModelClaudeOpus4_6,
     	MaxTokens: 16000,
     	Thinking:  anthropic.ThinkingConfigParamOfEnabled(10000),
     	Messages: []anthropic.MessageParam{
     		anthropic.NewUserMessage(anthropic.NewTextBlock("...")),
     	},
     })
     if err != nil {
     	log.Fatal(err)
     }
     fmt.Println(response)
     ```

     ```java Java
     AnthropicClient client = AnthropicOkHttpClient.fromEnv();

     MessageCreateParams params = MessageCreateParams.builder()
         .model(Model.CLAUDE_OPUS_4_6)
         .maxTokens(16000L)
         .enabledThinking(10000L)
         .addUserMessage("...")
         .build();

     Message response = client.messages().create(params);
     IO.println(response);
     ```

     ```php PHP
     $client = new Client();

     $message = $client->messages->create(
         maxTokens: 16000,
         messages: [['role' => 'user', 'content' => '...']],
         model: Model::CLAUDE_OPUS_4_6,
         thinking: ['type' => 'enabled', 'budgetTokens' => 10000],
     );
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     message = client.messages.create(
       model: Anthropic::Model::CLAUDE_OPUS_4_6,
       max_tokens: 16000,
       thinking: {
         type: "enabled",
         budget_tokens: 10000
       },
       messages: [
         { role: "user", content: "..." }
       ]
     )
     ```
   </CodeGroup>

   之后（Claude Opus 5.5），其中模型 ID、`thinking` 和 `output_config` 这几行有所不同：

   <CodeGroup>
     ```bash cURL
     curl https://api.anthropic.com/v1/messages \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "anthropic-version: 2026-10-21" \
       -H "content-type: application/json" \
       -d '{
         "model": "claude-opus-5-5",
         "max_tokens": 16000,
         "thinking": {
           "type": "adaptive"
         },
         "output_config": {
           "effort": "high"
         },
         "messages": [
           {
             "role": "user",
             "content": "..."
           }
         ]
       }'
     ```

     ```bash CLI
     ant messages create <<'YAML'
     model: claude-opus-5-5
     max_tokens: 16000
     thinking:
       type: adaptive
     output_config:
       effort: high
     messages:
       - role: user
         content: "..."
     YAML
     ```

     ```python Python
     client.messages.create(
         model="claude-opus-5-5",
         max_tokens=16000,
         thinking={"type": "adaptive"},
         output_config={"effort": "high"},  # or "max", "xhigh", "medium", "low"
         messages=[{"role": "user", "content": "..."}],
     )
     ```

     ```typescript TypeScript
     await client.messages.create({
       model: "claude-opus-5-5",
       max_tokens: 16000,
       thinking: { type: "adaptive" },
       output_config: { effort: "high" }, // or "max", "xhigh", "medium", "low"
       messages: [{ role: "user", content: "..." }]
     });
     ```

     ```csharp C#
     AnthropicClient client = new();

     var parameters = new MessageCreateParams
     {
         Model = Model.ClaudeOpus5_5,
         MaxTokens = 16000,
         Thinking = new ThinkingConfigAdaptive(),
         OutputConfig = new OutputConfig { Effort = Effort.High }, // or Max, Xhigh, Medium, Low
         Messages = [new UserMessageParam { Content = "..." }]
     };

     var response = await client.Messages.Create(parameters);
     Console.WriteLine(response);
     ```

     ```go Go
     client := anthropic.NewClient()

     response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
     	Model:     anthropic.ModelClaudeOpus5_5,
     	MaxTokens: 16000,
     	Thinking: anthropic.ThinkingConfigParamUnion{
     		OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{},
     	},
     	OutputConfig: anthropic.OutputConfigParam{
     		Effort: anthropic.OutputConfigEffortHigh, // or Max, Xhigh, Medium, Low
     	},
     	Messages: []anthropic.MessageParam{
     		anthropic.NewUserMessage(anthropic.NewTextBlock("...")),
     	},
     })
     if err != nil {
     	log.Fatal(err)
     }
     fmt.Println(response)
     ```

     ```java Java
     AnthropicClient client = AnthropicOkHttpClient.fromEnv();

     MessageCreateParams params = MessageCreateParams.builder()
         .model(Model.CLAUDE_OPUS_5_5)
         .maxTokens(16000L)
         .thinking(ThinkingConfigAdaptive.builder().build())
         .outputConfig(OutputConfig.builder()
             .effort(OutputConfig.Effort.HIGH) // or MAX, XHIGH, MEDIUM, LOW
             .build())
         .addUserMessage("...")
         .build();

     Message response = client.messages().create(params);
     IO.println(response);
     ```

     ```php PHP
     $client = new Client();

     $message = $client->messages->create(
         maxTokens: 16000,
         messages: [['role' => 'user', 'content' => '...']],
         model: Model::CLAUDE_OPUS_5_5,
         thinking: ['type' => 'adaptive'],
         outputConfig: ['effort' => 'high'], // or 'max', 'xhigh', 'medium', 'low'
     );
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     message = client.messages.create(
       model: Anthropic::Model::CLAUDE_OPUS_5_5,
       max_tokens: 16000,
       thinking: {
         type: "adaptive"
       },
       output_config: {
         effort: "high" # or "max", "xhigh", "medium", "low"
       },
       messages: [
         { role: "user", content: "..." }
       ]
     )
     ```
   </CodeGroup>

   自适应思考可以通过提示和 [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)进行引导，effort 参数取代了思考预算，成为控制模型推理量的方式。请在您自己的评估上进行 effort 扫描测试，而不是直接换算 `budget_tokens` 值。[effort 级别表](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#effort-levels)说明了何时使用各个级别，[Claude Opus 5.5 的推荐 effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#recommended-effort-levels-for-claude-opus-5-5)则介绍了此模型的情况。

2. **采样参数已移除：** 请省略 `temperature`、`top_p` 和 `top_k`。 在 Claude Opus 5.5 上，推荐通过提示来引导模型行为。如果您之前使用 `temperature = 0` 来获得确定性，请注意，它在之前的模型上也从未保证输出完全相同。

3. **默认省略思考内容：** 在 Claude Opus 4.7 及更高版本的模型上，思考块仍会出现在响应流中，但除非您明确选择启用，否则其 `thinking` 字段为空。这是相对于 Claude Opus 4.6 的一项静默变更，在 Claude Opus 4.6 中，默认会返回摘要形式的思考文本。要恢复该行为，请参阅[在每个响应中处理思考内容](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-in-every-response)的第 4 项。

4. **令牌计数已更新：** Claude Opus 4.7 引入了新的 tokenizer（分词器），之后的 Opus 模型（包括 Claude Opus 5.5）也使用该分词器。它有助于提升在各类任务上的性能，但与 Claude Opus 4.7 之前的模型相比，处理文本时可能会使用大约 1 到 1.35 倍的令牌（最多多出约 35%，因内容而异）。

   [`/v1/messages/count_tokens`](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting) 为 Claude Opus 5.5 返回的令牌数与为 Claude Opus 4.6 返回的不同。令牌效率可能因工作负载形态而异。

   请更新您的 `max_tokens` 参数以留出额外余量（包括压缩触发阈值），并重新测试任何在客户端估算令牌数或假定固定令牌/字符比例的代码路径。请使用[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)端点进行验证。提示层面的干预、[`task_budget`](https://platform.claude.com/docs/zh-CN/build-with-claude/task-budgets) 和 [`effort`](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 可以帮助控制成本；但这些控制手段可能会以牺牲模型智能为代价。

5. **预填充移除（在 Claude Opus 4.6 上已生效）：** 在 Claude Opus 4.6 及更高版本的 Opus 模型（包括 Claude Opus 5.5）上，预填充助手消息会返回 400 错误，因此只有当您从 Claude Opus 4.5 或更早版本迁移时，这才是一项变更。请改用[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)或系统提示指令。

### 行为变更

Claude Opus 4.7 引入了一些与 Claude Opus 4.6 不同的行为差异，这些差异并非 API 破坏性变更。以下三项会影响代码或脚手架：

1. **智能体轨迹中的内置进度更新：** 在较长的智能体轨迹中，Claude Opus 4.7 会向用户提供更频繁、更高质量的更新。如果您添加了用于强制输出中间状态消息的脚手架（例如"每进行 3 次工具调用后，总结进度"），请尝试将其移除。在 Claude Opus 5.5 上，这些更新出现在 `thinking` 块中，而在默认的 `thinking.display` 设置下这些块为空。要接收这些更新，请参阅[工具调用之间的文本以思考块形式返回](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#text-between-tool-calls)。要调整其长度和内容，请参阅[面向用户的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#user-facing-progress-updates)。

2. **实时网络安全防护措施：** 这些防护措施是 Claude Opus 4.7 中新增的：涉及受禁止或高风险主题的请求可能会被拒绝。对于渗透测试、漏洞研究或红队测试等合法安全工作，请申请加入 [Cyber Verification Program](https://support.claude.com/en/articles/14604842-real-time-cyber-safeguards-on-claude-opus-and-sonnet) 以请求放宽限制。申请途径取决于您访问 Claude 的方式。

3. **高分辨率图像支持：** Claude Opus 4.7 是第一个支持高分辨率图像的 Claude 模型。最大图像分辨率为长边 2,576 像素，高于之前模型的 1,568 像素。这为视觉密集型工作负载带来了提升，对于计算机使用、屏幕截图理解和文档分析尤其有价值。

   高分辨率支持是自动启用的。请提前规划两点：

   * 全分辨率图像可能比之前的模型多使用约 3 倍的图像令牌（每张图像最多 4,784 个令牌，而之前的上限约为 1,600 个令牌）。请为图像密集型工作负载重新规划 `max_tokens` 和成本预期；如果您不需要额外的保真度，也可以在发送前进行降采样。
   * 在 Claude Opus 4.7 上，模型返回的指向坐标和边界框坐标与实际图像像素为 1:1 对应，因此无需进行缩放系数换算。

   详情请参阅 [Claude Opus 4.7 上的高分辨率图像支持](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#high-resolution-image-support-on-claude-opus-4-7)。

有关提示方面的差异，请参阅[为 Claude Opus 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5)和[提示最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)。

### 从 Claude Opus 4.5 或更早版本迁移

如果您要从 Claude Opus 4.5、Claude Opus 4.1 或更早的模型直接迁移到 Claude Opus 5.5，请从头阅读本页：首先按页面顺序完成前面的每一个部分。然后完成本部分前面的[从 Claude Opus 4.6 迁移的破坏性变更](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#opus-46-breaking-changes)。接着应用以下累积变更，这些变更在 Claude Opus 4.5 到 Claude Opus 4.7 之间生效。如果您使用的是 Claude Opus 4.1 或更早版本，请在完成本子部分后继续阅读[从 Claude 4.1 或更早版本迁移](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-4-1-or-earlier)。

#### 破坏性变更

1. **预填充移除**已在[从 Claude Opus 4.6 迁移的破坏性变更](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#opus-46-breaking-changes)中介绍。

2. **工具参数引号处理：** Claude Opus 4.6 及更高版本的模型在工具调用参数中生成的 JSON 字符串转义可能略有不同（例如，对 Unicode 转义或正斜杠转义的处理方式不同）。如果您将工具调用的 `input` 作为原始字符串解析而不是使用 JSON 解析器，请检查您的解析逻辑。标准 JSON 解析器（例如 `json.loads()` 或 `JSON.parse()`）会自动处理这些差异。

#### 推荐变更

第一项在 Claude Opus 5.5 上是必需的；其余各项为推荐变更。

1. **迁移到自适应思考（必需）：** 在 Claude Opus 4.7 及更高版本的模型上，`thinking: {"type": "enabled", "budget_tokens": N}` 会返回 400 错误。迁移前后的对比见[从 Claude Opus 4.6 迁移的破坏性变更](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#opus-46-breaking-changes)的第 1 项。此次迁移还需从 `client.beta.messages.create()` (csharp: `client.Beta.Messages.Create()`; go: `client.Beta.Messages.New()`; java: `client.beta().messages().create()`; php: `$client->beta->messages->create()`; cli: `ant beta:messages create`) 改为 `client.messages.create()` (csharp: `client.Messages.Create()`; go: `client.Messages.New()`; java: `client.messages().create()`; php: `$client->messages->create()`; cli: `ant messages create`)：自适应思考和 effort 不需要 beta SDK 命名空间，也不需要任何 beta 标头。

2. **移除 effort beta 标头：** effort 参数不需要 beta 标头。请从您的请求中移除 `effort-2025-11-24` beta。

3. **移除细粒度工具流式传输 beta 标头：** 细粒度工具流式传输不需要 beta 标头。请从您的请求中移除 `fine-grained-tool-streaming-2025-05-14` beta。

4. **移除交错思考 beta 标头：** 使用自适应思考时，在每个支持自适应思考的模型上，交错思考都会自动启用。请从您的请求中移除 `interleaved-thinking-2025-05-14` beta。

5. **迁移到 output\_config.format：** 如果您使用结构化输出，请将 `output_format={...}` 更新为 `output_config={"format": {...}}`。`output_format` 参数已弃用，并将在未来移除。如果仍要使用它，请添加 `structured-outputs-2025-11-13` beta 标头。否则，API 将返回 400 错误。Python SDK（v1.0 及更高版本）在 `client.beta.messages.create()` 或 `count_tokens()` 上不接受 `output_format={...}`。`parse()` 和 `stream()` 辅助方法的 `output_format=Model` 参数保持不变。

### 从 Claude 4.1 或更早版本迁移

要从 Claude Opus 4.1 或更早的模型直接迁移到 Claude Opus 5.5：

1. 从头开始按页面顺序阅读本页，直至并包括[从 Claude Opus 4.5 或更早版本迁移](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-45)。
2. 应用本子部分中的其他变更。

#### 其他破坏性变更

1. **移除采样参数：** 详见[采样参数已移除](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#opus-46-breaking-changes)。

2. **更新工具版本**

   <Warning>
     从 Claude 3.x 模型迁移时，这是一项破坏性变更。
   </Warning>

   更新到当前的工具版本。移除任何使用 `undo_edit` 命令的代码。

   <CodeGroup exclude="shell">
     ```python Python
     # 之前
     tools = [{"type": "text_editor_20250124", "name": "str_replace_editor"}]

     # 之后
     tools = [{"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"}]
     ```

     ```typescript TypeScript
     // 之前
     const legacyTools = [{ type: "text_editor_20250124", name: "str_replace_editor" }];

     // 之后
     const tools = [{ type: "text_editor_20250728", name: "str_replace_based_edit_tool" }];
     ```

     ```csharp C#
     var parameters = new MessageCreateParams
     {
         // 之前：{"type": "text_editor_20250124", "name": "str_replace_editor"}
         // 之后：
         Tools = [new ToolTextEditor20250728()],
         // ...
     };
     ```

     ```go Go
     params := anthropic.MessageNewParams{
     	// 之前：{"type": "text_editor_20250124", "name": "str_replace_editor"}
     	// 之后：
     	Tools: []anthropic.ToolUnionParam{
     		{OfTextEditor20250728: &anthropic.ToolTextEditor20250728Param{}},
     	},
     	// ...
     }
     ```

     ```java Java
     MessageCreateParams params = MessageCreateParams.builder()
         // 之前：{"type": "text_editor_20250124", "name": "str_replace_editor"}
         // 之后：
         .addTool(ToolTextEditor20250728.builder().build())
         // ...
         .build();
     ```

     ```php PHP
     $message = $client->messages->create(
         // 之前：['type' => 'text_editor_20250124', 'name' => 'str_replace_editor']
         // 之后：
         tools: [new ToolTextEditor20250728()],
         // ...
     );
     ```

     ```ruby Ruby
     # 之前
     legacy_tools = [{ type: "text_editor_20250124", name: "str_replace_editor" }]

     # 之后
     tools = [{ type: "text_editor_20250728", name: "str_replace_based_edit_tool" }]
     ```
   </CodeGroup>

   * **文本编辑器：** 使用 `text_editor_20250728` 和 `str_replace_based_edit_tool`。详情请参阅[文本编辑器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/text-editor-tool)文档。
   * **代码执行：** 升级到 `code_execution_20260521`。迁移说明请参阅[代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool#upgrade-to-latest-tool-version)文档。
   * **计算机使用：** 使用 `computer_toolset_20260801` 工具集代替 Claude Opus 5.5 不接受的 `computer_20250124`。在 Amazon Bedrock 上，`computer_20251124` 也可以使用。请参阅[计算机使用的破坏性变更](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#computer-use-toolset)。

3. **处理 `refusal` 停止原因**

   更新您的应用程序以[处理 `refusal` 停止原因](https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/handle-streaming-refusals)：

   <CodeGroup exclude="shell">
     ```python Python
     response = client.messages.create(...)

     if response.stop_reason == "refusal":
         # 妥善处理拒绝响应
         pass
     ```

     ```typescript TypeScript
     const response = await client.messages.create(/* ... */);

     if (response.stop_reason === "refusal") {
       // 妥善处理拒绝响应
     }
     ```

     ```csharp C#
     var response = await client.Messages.Create(...);

     if (response.StopReason?.Value() == StopReason.Refusal)
     {
         // 妥善处理拒绝响应
     }
     ```

     ```go Go
     response, _ := client.Messages.New(ctx, params) // your existing request

     if response.StopReason == anthropic.StopReasonRefusal {
     	// 适当处理拒绝响应
     }
     ```

     ```java Java
     Message response = client.messages().create(...);

     StopReason reason = response.stopReason().orElse(StopReason.END_TURN);
     if (reason.equals(StopReason.REFUSAL)) {
         // 适当处理拒绝响应
     }
     ```

     ```php PHP
     $response = $client->messages->create(...);

     if ($response->stopReason === 'refusal') {
         // 妥善处理拒绝情况
     }
     ```

     ```ruby Ruby
     response = client.messages.create(...)

     if response.stop_reason == :refusal
       # 适当处理拒绝响应
     end
     ```
   </CodeGroup>

4. **处理 `model_context_window_exceeded` 停止原因**

   当生成因达到上下文窗口限制（而非请求的 `max_tokens` 限制）而停止时，Claude 4.5 及更高版本的模型会返回 `model_context_window_exceeded` 停止原因。请更新您的应用程序以处理这一新的停止原因：

   <CodeGroup exclude="shell">
     ```python Python
     response = client.messages.create(...)

     if response.stop_reason == "model_context_window_exceeded":
         # 妥善处理上下文窗口限制
         pass
     ```

     ```typescript TypeScript
     const response = await client.messages.create(/* ... */);

     if (response.stop_reason === "model_context_window_exceeded") {
       // 妥善处理 context window（上下文窗口）限制
     }
     ```

     ```csharp C#
     var response = await client.Messages.Create(...);

     if (response.StopReason?.Raw() == "model_context_window_exceeded")
     {
         // 妥善处理上下文窗口限制
     }
     ```

     ```go Go
     response, _ := client.Messages.New(ctx, params) // your existing request

     if response.StopReason == "model_context_window_exceeded" {
     	// 妥善处理上下文窗口限制
     }
     ```

     ```java Java
     Message response = client.messages().create(...);

     StopReason reason = response.stopReason().orElse(StopReason.END_TURN);
     if (reason.equals(StopReason.of("model_context_window_exceeded"))) {
         // 妥善处理上下文窗口限制
     }
     ```

     ```php PHP
     $response = $client->messages->create(...);

     if ($response->stopReason === 'model_context_window_exceeded') {
         // 妥善处理上下文窗口限制
     }
     ```

     ```ruby Ruby
     response = client.messages.create(...)

     if response.stop_reason == :model_context_window_exceeded
       # 妥善处理上下文窗口限制
     end
     ```
   </CodeGroup>

5. **验证工具参数处理（尾随换行符）**

   Claude 4.5 及更高版本的模型会保留工具调用字符串参数中的尾随换行符，而这些换行符以前会被去除。如果您的工具依赖于对工具调用参数进行精确字符串匹配，请验证您的逻辑能否正确处理尾随换行符。

6. **针对行为变化更新您的提示**

   Claude 4 及更高版本的模型具有更简洁、直接的沟通风格，并且需要明确的指示。有关优化指导，请查阅[提示最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)。

#### 其他推荐变更

* **Beta 标头：** 移除 `token-efficient-tools-2025-02-19` 和 `output-128k-2025-02-19`。所有 Claude 4 及更高版本的模型都内置了令牌高效的工具使用功能，这些标头不再起作用。

## 从 Claude Sonnet 5 迁移到 Claude Opus 5.5

请完成[每个发送到 Claude Opus 5.5 的请求必须满足的条件](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#request-requirements)、[在每个响应中处理思考内容](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#thinking-in-every-response)以及[从 Claude Opus 5 迁移到 Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/migration-guide#migrating-from-claude-opus-5)。将 `claude-sonnet-5` 作为您要替换的模型 ID。最后一个部分按原样适用于运行在 Claude Sonnet 5 上的代码，因为 Claude Sonnet 5 与 Claude Opus 5 一样：

* 默认启用思考，并接受 `thinking: {"type": "disabled"}`（对于 Claude Sonnet 5，在任何 effort 级别下均可）。
* 接受强制工具选择和 `computer_20251124` 工具。
* 将工具调用之间的文本作为 `text` 块返回。
* 默认使用 `high` effort。

手动扩展思考、采样参数以及助手预填充在这两个模型上都会返回 400 错误，因此这方面没有任何变化。Claude Opus 4.8、Claude Opus 4.7 和 Claude Opus 4.6 相关部分中的必需变更均不适用于您。

### 变更内容

1. **对话中途的系统消息：** 在 Claude API、Amazon Bedrock、Claude Platform on AWS、Claude Models on Google Cloud (Anthropic managed) 和 Claude Models on Google Cloud (Google managed) 上，Claude Opus 5.5 接受在 `messages` 数组中紧跟在用户轮次之后的 `role: "system"` 消息（需遵守[放置规则](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#limitations)）。此功能在 Claude Sonnet 5 上不可用。如果您维护着通过重建完整消息历史来更新指令的代码路径，您可以简化它们，并保留早期轮次的[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)命中。

2. **更低的提示缓存最小值：** Claude Opus 5.5 上可缓存提示的最小长度为 512 个令牌，低于 Claude Sonnet 5 上的 1,024 个令牌。在 Claude Sonnet 5 上因过短而无法缓存的提示现在可以创建缓存条目，无需任何代码更改。有关各模型的最小值，请参阅[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#cache-limitations)。
