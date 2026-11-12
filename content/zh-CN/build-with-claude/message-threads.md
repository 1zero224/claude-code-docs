---
title: 消息线程
url: https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads
description: 通过在每次请求中仅发送新消息，为长时间运行的智能体缩短首个令牌时间，在长对话中尤其如此。
featureMetadata:
  status: beta
  betaHeader: message-threads-2026-08-12
  zdr:
    eligibility: not-eligible
    note: 线程的内容（对话、系统提示和工具）保存在内存中，使用服务器管理的密钥加密，从不写入持久存储，并在线程最后一次使用后 1 小时内删除。使用此功能不会影响您使用其他具备 ZDR 资格的功能。
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
    Claude API: beta
    Claude Platform on AWS: beta
    Claude Models on Google Cloud (Anthropic managed): beta
    Microsoft Foundry: beta
---

使用 “message thread”（消息线程）时，请求更小，并且随着对话增长，"time to first token"（首个令牌时间）保持更低、更稳定。线程在较长的智能体循环中最有帮助。没有线程时，每一轮都会重新上传整个对话，API 也会重新验证并对其全部进行分词。使用线程后，大部分工作只涉及新的一轮。

“Message thread”（消息线程）会在服务器上短时间保留一段对话。您只需发送一次完整对话，连同您的系统提示和工具。在之后的每一轮中，您只需发送新消息以及上一个响应的 `id`。API 会在调用模型之前重建对话。您只需在 [Messages API](https://platform.claude.com/docs/zh-CN/api/messages) 请求中添加一个参数 `thread`。响应就是通常的消息。

<Note>
  消息线程不同于 Claude Managed Agents 中的[会话线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)。
</Note>

## 消息线程的工作原理

* **创建。** 像现在一样发送您的第一个请求，包含完整对话、`system` 和 `tools`，并添加 `"thread": {"type": "create"}`。API 会将对话（包括 Claude 的回复）存储在响应 `id` 下。
* **继续。** 仅发送自上一个响应以来的消息，并添加 `"thread": {"type": "continue", "previous_message_id": "<previous id>"}`。请求中不要包含 `system` 和 `tools`。线程会保留创建请求中的值。每个新响应的 `id` 用于下一轮。
* **重放。** 线程是尽力而为的。线程会在其最后一次请求后 5 分钟过期，API 会在其最后一次使用后不超过 1 小时内将其删除。API 存储的每个响应都会重新开始这 5 分钟的计时。如果 API 无法再继续该线程，继续请求会返回 404 错误。请在新的创建请求中再次发送完整对话，并从该响应的 `id` 继续。SDK 线程辅助工具会为您执行此重放（请参阅[SDK 辅助工具的工作方式](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#sdk-helper-handles)）。在收到来自回退模型的响应后，请参阅[在线程中使用回退](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#fallbacks)。

您的应用程序始终是对话的事实来源。请保留每条消息的副本，并从一开始就构建[重放路径](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#recover)。无论是对创建请求还是继续请求的响应，都不会说明 API 是否存储了该轮次。您会在下一次继续请求时才得知。SDK 线程辅助工具在辅助对象存在期间将此副本保存在内存中，不会保存到其他任何地方。如果对话需要在您的进程结束后继续存在，请保留您自己的副本。

## 通过 SDK 使用线程

SDK 提供了一个线程辅助工具，可以为您构建线程请求。使用与 Messages 请求相同的字段创建它，然后一次发送一轮。该辅助工具会在内存中保留完整对话。

此示例有两轮。辅助工具在第一轮创建线程，在第二轮继续该线程，并为您添加 beta 标头。顶层 `cache_control` 是可选的。它会在每一轮请求提示缓存（请参阅[线程上的提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#prompt-caching)）。没有它时，之后的每一轮仍然只发送新消息，但除非您自己添加缓存断点，否则不会缓存任何内容。

<CodeGroup exclude="shell">
  ```python Python
  client = anthropic.Anthropic()

  # 创建辅助对象。此调用不会发送请求。
  thread = client.beta.messages.thread(
      model="claude-opus-5-5",
      max_tokens=1024,
      cache_control={"type": "ephemeral"},
      system="You are a support agent for Acme. Keep replies short.",
  )

  # 第 1 轮：辅助对象创建一个线程并发送完整对话
  thread.send({"role": "user", "content": "Hi! I have a question about an order."})

  # 第 2 轮：辅助对象继续该线程，仅发送新消息
  second = thread.send(
      {"role": "user", "content": "How long does standard shipping usually take?"}
  )

  for block in second.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  // 创建辅助对象。此调用不会发送请求。
  const thread = client.beta.messages.thread({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    cache_control: { type: "ephemeral" },
    system: "You are a support agent for Acme. Keep replies short."
  });

  // 第 1 轮：辅助对象创建一个线程并发送完整对话
  await thread.send({ role: "user", content: "Hi! I have a question about an order." });

  // 第 2 轮：辅助对象继续该线程，仅发送新消息
  const second = await thread.send({
    role: "user",
    content: "How long does standard shipping usually take?"
  });

  for (const block of second.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  using Anthropic.Helpers.Beta;

  AnthropicClient client = new();

  // 创建辅助对象。此调用不会发送任何请求。
  var thread = client.Beta.Messages.Thread(
      new ThreadCreateParams
      {
          Model = Messages::Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [],
          CacheControl = new(),
          System = "You are a support agent for Acme. Keep replies short.",
      }
  );

  // 第 1 轮：辅助对象创建一个线程并发送完整对话
  await thread.Send(
      new BetaUserMessageParam { Content = "Hi! I have a question about an order." }
  );

  // 第 2 轮：辅助对象继续该线程，仅发送新消息
  var second = await thread.Send(
      new BetaUserMessageParam { Content = "How long does standard shipping usually take?" }
  );

  foreach (var block in second.Content)
  {
      if (block.TryPickText(out var textBlock))
      {
          Console.WriteLine(textBlock.Text);
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()
  ctx := context.Background()

  // 创建辅助对象。此调用不会发送请求。
  thread := client.Beta.Messages.NewThread(anthropic.BetaMessageThreadCreateParams{
  	Model:        anthropic.ModelClaudeOpus5_5,
  	MaxTokens:    1024,
  	CacheControl: anthropic.NewBetaCacheControlEphemeralParam(),
  	System: []anthropic.BetaTextBlockParam{
  		{Text: "You are a support agent for Acme. Keep replies short."},
  	},
  })

  // 第 1 轮：辅助对象创建一个线程并发送完整对话
  _, err := thread.Send(ctx, anthropic.BetaMessageThreadSendParams{
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hi! I have a question about an order.")),
  	},
  })
  if err != nil {
  	panic(err)
  }

  // 第 2 轮：辅助对象继续该线程，仅发送新消息
  second, err := thread.Send(ctx, anthropic.BetaMessageThreadSendParams{
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("How long does standard shipping usually take?")),
  	},
  })
  if err != nil {
  	panic(err)
  }

  for _, block := range second.Content {
  	if textBlock, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  ```

  ```java Java
  var client = AnthropicOkHttpClient.fromEnv();

  // 创建辅助对象。此调用不会发送任何请求。
  var thread = client.beta().messages().thread(
      ThreadCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .cacheControl(BetaCacheControlEphemeral.builder().build())
          .system("You are a support agent for Acme. Keep replies short.")
          .build()
  );

  // 第 1 轮：辅助对象创建一个线程并发送完整对话
  thread.send("Hi! I have a question about an order.");

  // 第 2 轮：辅助对象继续该线程，仅发送新消息
  var second = thread.send("How long does standard shipping usually take?");

  second.content().stream()
      .flatMap(block -> block.text().stream())
      .forEach(textBlock -> IO.println(textBlock.text()));
  ```

  ```php PHP
  $client = new Client();

  // 创建辅助对象。此调用不会发送任何请求。
  $thread = $client->beta->messages->thread(
      maxTokens: 1024,
      model: Model::CLAUDE_OPUS_5_5,
      cacheControl: new BetaCacheControlEphemeral,
      system: 'You are a support agent for Acme. Keep replies short.',
  );

  // 第 1 轮：辅助对象创建一个线程并发送完整对话
  $thread->send(['role' => 'user', 'content' => 'Hi! I have a question about an order.']);

  // 第 2 轮：辅助对象继续该线程，仅发送新消息
  $second = $thread->send(['role' => 'user', 'content' => 'How long does standard shipping usually take?']);

  foreach ($second->content as $block) {
      if ($block instanceof \Anthropic\Beta\Messages\BetaTextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  # 创建辅助对象。此调用不会发送任何请求。
  thread = client.beta.messages.thread(
    model: :"claude-opus-5-5",
    max_tokens: 1024,
    cache_control: {type: "ephemeral"},
    system_: "You are a support agent for Acme. Keep replies short."
  )

  # 第 1 轮：辅助对象创建一个线程并发送完整对话
  thread.send_message({role: "user", content: "Hi! I have a question about an order."})

  # 第 2 轮：辅助对象继续该线程，仅发送新消息
  second = thread.send_message({role: "user", content: "How long does standard shipping usually take?"})

  second.content.each do |block|
    puts block.text if block.type == :text
  end
  ```
</CodeGroup>

## 自行发送 `thread` 字段

若要自行构建线程请求，例如使用 cURL、CLI 或 SDK 的常规 `create` 方法，请在每个请求中添加 `thread` 字段。

在线程的每个请求上发送 `message-threads-2026-08-12` beta 标头。

此示例在第一轮创建线程，在第二轮继续该线程。顶层 `cache_control` 会在每一轮请求提示缓存。一旦对话长度超过[最小可缓存长度](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)，它就会生效。

<CodeGroup>
  ```bash cURL
  # 第 1 轮：发送完整对话并创建一个 thread（线程）
  curl -sS --fail-with-body https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: message-threads-2026-08-12" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "cache_control": {"type": "ephemeral"},
      "system": "You are a support agent for Acme. Keep replies short.",
      "messages": [{"role": "user", "content": "Hi! I have a question about an order."}],
      "thread": {"type": "create"}
    }'

  # 第 2 轮：仅发送新消息。将 message_id 设置为第 1 轮响应的 id。
  # 省略 system 和 tools：线程会保留它们。
  curl -sS --fail-with-body https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: message-threads-2026-08-12" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "cache_control": {"type": "ephemeral"},
      "messages": [{"role": "user", "content": "How long does standard shipping usually take?"}],
      "thread": {"type": "continue", "previous_message_id": "'"$message_id"'"}
    }'
  ```

  ```bash CLI
  # 第 1 轮：发送完整对话并创建一个 thread（线程）
  ant beta:messages create --beta message-threads-2026-08-12 <<'YAML'
  model: claude-opus-5-5
  max_tokens: 1024
  cache_control:
    type: ephemeral
  system: "You are a support agent for Acme. Keep replies short."
  messages:
    - role: user
      content: "Hi! I have a question about an order."
  thread:
    type: create
  YAML

  # 第 2 轮：仅发送新消息。将 message_id 设置为第 1 轮响应的 id。
  # 省略 system 和 tools：线程会保留它们。
  ant beta:messages create --beta message-threads-2026-08-12 <<YAML
  model: claude-opus-5-5
  max_tokens: 1024
  cache_control:
    type: ephemeral
  messages:
    - role: user
      content: "How long does standard shipping usually take?"
  thread:
    type: continue
    previous_message_id: $message_id
  YAML
  ```

  ```python Python
  client = anthropic.Anthropic()

  # 第 1 轮：发送完整对话并创建一个 thread（线程）
  first = client.beta.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      cache_control={"type": "ephemeral"},
      system="You are a support agent for Acme. Keep replies short.",
      messages=[{"role": "user", "content": "Hi! I have a question about an order."}],
      thread={"type": "create"},
      betas=["message-threads-2026-08-12"],
  )

  # 第 2 轮：仅发送新消息以及第 1 轮响应的 id。
  # 省略 system 和 tools：线程会保留它们。
  second = client.beta.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      cache_control={"type": "ephemeral"},
      messages=[
          {"role": "user", "content": "How long does standard shipping usually take?"}
      ],
      thread={"type": "continue", "previous_message_id": first.id},
      betas=["message-threads-2026-08-12"],
  )

  for block in second.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  // 第 1 轮：发送完整对话并创建一个 thread（线程）
  const first = await client.beta.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    cache_control: { type: "ephemeral" },
    system: "You are a support agent for Acme. Keep replies short.",
    messages: [{ role: "user", content: "Hi! I have a question about an order." }],
    thread: { type: "create" },
    betas: ["message-threads-2026-08-12"]
  });

  // 第 2 轮：仅发送新消息以及第 1 轮响应的 id。
  // 省略 system 和 tools：线程会保留它们。
  const second = await client.beta.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    cache_control: { type: "ephemeral" },
    messages: [{ role: "user", content: "How long does standard shipping usually take?" }],
    thread: { type: "continue", previous_message_id: first.id },
    betas: ["message-threads-2026-08-12"]
  });

  for (const block of second.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  // 第 1 轮：发送完整对话并创建一个 thread（线程）
  var first = await client.Beta.Messages.Create(
      new()
      {
          Model = Messages::Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          CacheControl = new(),
          System = "You are a support agent for Acme. Keep replies short.",
          Messages =
          [
              new BetaUserMessageParam { Content = "Hi! I have a question about an order." },
          ],
          Thread = new BetaThreadCreateParams(),
          Betas = [AnthropicBeta.MessageThreads2026_08_12],
      }
  );

  // 第 2 轮：仅发送新消息以及第 1 轮响应的 id。
  // 省略 System 和 Tools：线程会保留它们。
  var second = await client.Beta.Messages.Create(
      new()
      {
          Model = Messages::Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          CacheControl = new(),
          Messages =
          [
              new BetaUserMessageParam { Content = "How long does standard shipping usually take?" },
          ],
          Thread = new BetaThreadContinueParams(first.ID),
          Betas = [AnthropicBeta.MessageThreads2026_08_12],
      }
  );

  foreach (var block in second.Content)
  {
      if (block.TryPickText(out var textBlock))
      {
          Console.WriteLine(textBlock.Text);
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()
  ctx := context.Background()

  // 第 1 轮：发送完整对话并创建一个 thread（会话线程）
  threadCreate := anthropic.NewBetaThreadCreateParams()
  first, err := client.Beta.Messages.New(ctx, anthropic.BetaMessageNewParams{
  	Model:        anthropic.ModelClaudeOpus5_5,
  	MaxTokens:    1024,
  	CacheControl: anthropic.NewBetaCacheControlEphemeralParam(),
  	System: []anthropic.BetaTextBlockParam{
  		{Text: "You are a support agent for Acme. Keep replies short."},
  	},
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hi! I have a question about an order.")),
  	},
  	Thread: anthropic.BetaMessageThreadParamUnion{OfCreate: &threadCreate},
  	Betas:  []anthropic.AnthropicBeta{anthropic.AnthropicBetaMessageThreads2026_08_12},
  })
  if err != nil {
  	panic(err)
  }

  // 第 2 轮：仅发送新消息以及第 1 轮响应的 id。
  // 省略 System 和 Tools：线程会保留它们。
  second, err := client.Beta.Messages.New(ctx, anthropic.BetaMessageNewParams{
  	Model:        anthropic.ModelClaudeOpus5_5,
  	MaxTokens:    1024,
  	CacheControl: anthropic.NewBetaCacheControlEphemeralParam(),
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("How long does standard shipping usually take?")),
  	},
  	Thread: anthropic.BetaMessageThreadParamOfContinue(first.ID),
  	Betas:  []anthropic.AnthropicBeta{anthropic.AnthropicBetaMessageThreads2026_08_12},
  })
  if err != nil {
  	panic(err)
  }

  for _, block := range second.Content {
  	if textBlock, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  ```

  ```java Java
  var client = AnthropicOkHttpClient.fromEnv();

  // 第 1 轮：发送完整对话并创建一个 thread（线程）
  var first = client.beta().messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .cacheControl(BetaCacheControlEphemeral.builder().build())
          .system("You are a support agent for Acme. Keep replies short.")
          .addUserMessage("Hi! I have a question about an order.")
          .thread(BetaThreadCreateParams.builder().build())
          .addBeta(AnthropicBeta.MESSAGE_THREADS_2026_08_12)
          .build()
  );

  // 第 2 轮：仅发送新消息以及第 1 轮响应的 id。
  // 省略 system 和 tools：线程会保留它们。
  var second = client.beta().messages().create(
      MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .cacheControl(BetaCacheControlEphemeral.builder().build())
          .addUserMessage("How long does standard shipping usually take?")
          .continueThread(first.id())
          .addBeta(AnthropicBeta.MESSAGE_THREADS_2026_08_12)
          .build()
  );

  second.content().stream()
      .flatMap(block -> block.text().stream())
      .forEach(textBlock -> IO.println(textBlock.text()));
  ```

  ```php PHP
  $client = new Client();

  // 第 1 轮：发送完整对话并创建一个线程
  $first = $client->beta->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 1024,
      cacheControl: new BetaCacheControlEphemeral,
      system: 'You are a support agent for Acme. Keep replies short.',
      messages: [
          ['role' => 'user', 'content' => 'Hi! I have a question about an order.'],
      ],
      thread: BetaThreadCreateParams::with(),
      betas: ['message-threads-2026-08-12'],
  );

  // 第 2 轮：仅发送新消息以及第 1 轮响应的 id。
  // 省略 system 和 tools：线程会保留它们。
  $second = $client->beta->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 1024,
      cacheControl: new BetaCacheControlEphemeral,
      messages: [
          ['role' => 'user', 'content' => 'How long does standard shipping usually take?'],
      ],
      thread: BetaThreadContinueParams::with(previousMessageID: $first->id),
      betas: ['message-threads-2026-08-12'],
  );

  foreach ($second->content as $block) {
      if ($block instanceof \Anthropic\Beta\Messages\BetaTextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  # 第 1 轮：发送完整对话并创建一个线程
  first = client.beta.messages.create(
    model: :"claude-opus-5-5",
    max_tokens: 1024,
    cache_control: {type: "ephemeral"},
    system_: "You are a support agent for Acme. Keep replies short.",
    messages: [{role: "user", content: "Hi! I have a question about an order."}],
    thread: {type: "create"},
    betas: ["message-threads-2026-08-12"]
  )

  # 第 2 轮：仅发送新消息以及第 1 轮响应的 id。
  # 省略 system_ 和 tools：线程会保留它们。
  second = client.beta.messages.create(
    model: :"claude-opus-5-5",
    max_tokens: 1024,
    cache_control: {type: "ephemeral"},
    messages: [{role: "user", content: "How long does standard shipping usually take?"}],
    thread: {type: "continue", previous_message_id: first.id},
    betas: ["message-threads-2026-08-12"]
  )

  second.content.each do |block|
    puts block.text if block.type == :text
  end
  ```
</CodeGroup>

## 每个请求要发送的内容

| 分组  | 字段                                                                                                                                                                    | 规则                                                                                        |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| 仅创建 | `system`、`tools`                                                                                                                                                      | 仅在创建请求上发送它们，并在其上附带任何 `cache_control` 断点。在继续请求上发送其中任何一个（即使为 `null`）都会返回 400 错误。            |
| 固定  | `model`、`tool_choice`、`thinking`、`mcp_servers`（每个服务器的授权令牌除外）、`context_management`、`output_config`、`container` 中的 `skills`、顶层 `cache_control`，以及 `anthropic-beta` 值的集合 | 在线程的每个请求上发送它们，且值与创建请求上的相同。不同的值会返回代码为 `thread_fingerprint_mismatch` 的 400 错误，错误消息会指出不同的字段。 |
| 自由  | `max_tokens`、`stop_sequences`、`stream`、`metadata`、`service_tier`，以及本页未提及的其他字段                                                                                         | 这些字段可以在各轮之间更改。                                                                            |

API 不会存储 MCP 服务器 URL、MCP 授权令牌或容器 `id`。请在每个使用它们的请求上发送 `mcp_servers` 和 `container`。MCP 授权令牌可以在请求之间更改。

SDK 线程辅助工具会为您在继续请求中省略 `system` 和 `tools`，并在每一轮再次发送固定字段。

如果您使用[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)，请在线程的每个请求上都发送 `fallbacks` 字段，或者都不发送。请参阅[在线程中使用回退](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#fallbacks)。

若要更改仅创建字段或固定字段，请开始一个新线程：在创建请求中使用新值发送完整对话。

**如果您更改 `system` 或 `tools`：** 在以下模型上，请在您通过新的创建请求发送的对话中省略所有 `thinking` 和 `redacted_thinking` 块：Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5。这样，模型在回答时将不会使用那些早先的推理。

* **原因：** 更改 `system` 或 `tools` 会使对话中已有的 `thinking` 和 `redacted_thinking` 块失效。请参阅[哪些算作编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#what-counts-as-an-edit)。
* **如果您仍然发送这些块：** 创建请求可能会返回 400 错误。是否返回错误取决于模型和您的账户。请参阅[未设置 `prefix_mismatch_behavior` 时](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#enforcement)和[API 如何处理无效的块](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#mismatch-behavior)。

在继续请求上只发送新消息。API 不会将它们与已有的消息进行比对。您再次发送的消息可能会在对话中出现两次，或者请求可能返回错误。例如，以上一个响应开头的继续请求会返回错误。

新消息通常以 `user` 消息开头：您的下一个用户轮次，或回答上一个响应中 `tool_use` 的 `tool_result` 块。如果响应以 [`stop_reason: "pause_turn"`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#pause-turn) 停止，请使用从该响应的 `id` 发起的继续请求和空的 `messages` 列表（`"messages": []`）来恢复它。如果该请求返回 404 错误，请在新的创建请求中重放完整对话，包括已暂停的响应。在轮次暂停期间不要发送新的 `user` 消息：API 会拒绝它。使用 SDK 线程辅助工具时，请不带消息调用 `send()` (go, csharp: `Send()`; ruby: `send_message()`)：辅助工具会发送此请求，并在出现 404 错误时进行重放。

线程是仅追加的，每个响应 `id` 只能被继续一次。请始终从线程的最新响应 `id` 继续。若要编辑或分支较早的消息，请使用创建请求开始一个新线程。若要用摘要替换较早的轮次，请参阅[压缩对话](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#compaction)。

## 线程上的提示缓存

线程不会为您缓存提示。[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)的工作方式与不使用线程时相同：由您选择缓存断点的位置。

若要缓存每一轮，请在线程的每个请求上发送顶层 `cache_control`，如前面的示例所示。API 会将其应用于每个请求的 `messages` 中最后一个可缓存的块。这样，每一轮都会从缓存中读取较早的对话，并将新的一轮写入缓存。顶层 `cache_control` 是固定字段：请在每个请求上发送相同的值。

您也可以自行在块上放置 `cache_control` 断点：

* 您在创建请求中发送的断点，如果位于 `tools`、`system` 或最后一个消息块之前的某个消息块上，则在之后的每一轮中都会持续生效。位于请求 `messages` 最后一个块上的断点会在该请求中写入缓存。要缓存每个新的轮次，请在每个请求中都发送一个断点。不带断点的请求仍可能从缓存中读取部分对话。
* 断点发送后便无法移动或移除，因为继续请求无法再次发送较早的消息。
* 带有 `thread` 的请求最多可携带 4 个 `cache_control` 断点。顶层 `cache_control` 计为一个。创建请求中 `system` 和 `tools` 上的断点会在每个继续请求中再次计数。较早存储的消息上的断点不计数。多出一个断点就会返回 400 错误，错误代码为 `thread_unsupported_request`。
* 如果您自己在消息块上放置断点，那么在新的创建请求中重放对话时，请从较早的消息中移除这些断点，并在最后一个块上保留一个，以使请求保持在 4 个的限制之内。

## 流式传输

线程可与[流式传输](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)一起使用。响应 `id` 会在 `message_start` 事件中到达，但请仅在收到 `message_stop` 事件后再从它继续。更早发送的继续请求可能会返回 404 错误。

如果继续请求的流在 `message_stop` 之前中断，请再次发送相同的继续请求。如果该请求返回 `thread_already_continued` 或 `thread_continue_in_flight` 错误，请在新的创建请求中重放完整对话。

您的连接可能在服务器仍在生成响应时中断，也可能在服务器保存响应之后中断。这就是 API 可能拒绝重试的原因。重放始终是安全的。

SDK 线程辅助工具在流中断后不会重试继续请求。请参阅[SDK 辅助工具的工作方式](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#sdk-helper-handles)。

## 线程无法继续时的恢复

当 API 无法继续线程时，继续请求会返回 404 错误，且 `error.details.error_code` 设置为 `thread_not_found`。响应不会说明原因。原因包括以下几种：

* 线程已过期。
* API 提供了上一轮的服务，但没有存储它。例如，API 不会存储以被 `max_tokens` 截断的工具调用结尾的响应。
* 上一个响应由回退模型编写。请参阅[在线程中使用回退](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#fallbacks)。
* 上一个响应仍在生成中。
* 请求来自不同的组织或工作区。
* 请求来自与创建线程的用户或服务账户不同的用户或服务账户。使用[工作区密钥（旧版）](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)创建的线程会绑定到该密钥，因此请在整个线程中使用同一个密钥。
* 新消息添加了存储的对话无法接受的内容。例如，在原本没有图像的对话中加入第一张图像可能会返回此错误。

若要恢复，请从您自己的副本中，在一个新的创建请求中发送完整对话，并附带 `system` 和 `tools`。SDK 线程辅助工具会自行发送一次此创建请求，并在发送时省略较早轮次的 `cache_control` 断点。如果您副本中较早的消息带有 `cache_control` 断点，请先将其移除（请参阅[线程上的提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#prompt-caching)）。然后从新响应的 `id` 继续。如果上一个响应由回退模型生成，请参阅[在线程中使用回退](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#fallbacks)。若要排除所有权方面的原因，请对一个线程的每个请求使用相同的 API 密钥和工作区。在 Claude Platform on AWS 上，请使用创建该线程的 IAM 角色或 IAM 用户的凭证为该线程的每个请求签名。

请求不存在的模型也会返回 404 错误，因此请在重放之前检查 `error.details.error_code`。

如果在重放后继续请求仍然立即返回 404 错误，则 API 可能无法存储该对话，例如非常大的对话。请在不使用 `thread` 的情况下发送它。当您在创建 SDK 线程辅助工具时将 `unsupported_request_behavior` (typescript, java, php: `unsupportedRequestBehavior`; go, csharp: `UnsupportedRequestBehavior`) 设置为 `continue_unthreaded` (ruby: `:continue_unthreaded`; java: `CONTINUE_UNTHREADED`; csharp: `ContinueUnthreaded`; go: `BetaMessageThreadUnsupportedRequestBehaviorContinueUnthreaded`) 时，它会这样做。否则，它会在每一轮都进行重放。

### 拒绝之后

当响应包含 `stop_reason: "refusal"` 时，请不要继续该线程。API 可能已经保存了被拒绝的轮次，而继续请求无法将其移除。请使用一个省略或改写导致拒绝的轮次的创建请求来启动新的线程。请参阅[拒绝与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)。SDK 线程辅助工具会为您完成此操作：它不会保留被拒绝的轮次，并且其下一轮会启动新的线程。

### 在线程中使用回退

[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)可在 Claude API 上与线程一起使用。请在创建线程时做出决定：在线程的每个请求上都发送 `fallbacks`，或者都不发送。`fallbacks` 的值可以在请求之间更改。添加或删除 `fallbacks` 的继续请求会返回代码为 `thread_fingerprint_mismatch` 的 400 错误。

一个线程只保留一个模型，因此 API 不会存储由回退模型编写的轮次。您会照常收到响应，但无法从其 `id` 继续。在每个响应之后，请按照以下步骤操作：

1. 如果 `stop_reason` 为 `"refusal"`，请按照[拒绝之后](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#after-a-refusal)操作。

2. 否则，在 `usage.iterations` 中查找 `fallback_message` 条目。在流式传输中，请从最后的 `message_delta` 事件中读取它。如果没有该条目，请照常继续该线程。

3. 如果有，则说明该响应由回退模型生成。请在一个新的创建请求中发送完整对话，包括该响应和您的下一条消息。在该请求上保留 `server-side-fallback-2026-07-01` beta 标头。从以下两种请求中选择一种：

   * **再次发送相同的请求。** 保持 `model`、`fallbacks` 和您的其他参数不变。如果由您请求的模型作答，API 会存储新线程。如果由回退模型作答，响应中会再次包含 `fallback_message` 条目，且 API 不会存储它，因此请在下一轮重复此步骤。[粘性路由](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#sticky-routing)可能会将这些请求直接发送到回退模型，因此每一轮都可能需要发送完整的对话。SDK 线程辅助工具就是以这种方式工作的。
   * **在回退模型上启动新的线程。** 将 `model` 设置为该 `fallback_message` 条目中的 `model`，省略 `fallbacks`，并使用该模型接受的参数。API 会立即存储此线程，并且它会一直使用回退模型。回退尝试可能使用了与您发送的设置不同的设置运行，因此 API 可能会拒绝从您的第一个请求中复制的值。

4. 使用该请求的响应返回第 1 步。当响应中没有 `fallback_message` 条目时，从其 `id` 继续新的线程。

[粘性路由](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#sticky-routing)在请求发生回退（创建请求，或不带 `thread` 的请求）之后开始。在大约一小时内，它可能会将同一对话中后续包含 `fallbacks` 的创建请求直接发送到回退模型，而不尝试您请求的模型。API 不会存储这些轮次中的任何一轮，并且每个得到回答的轮次都会重新开始计算这一小时。继续请求永远不会启动此路由，也永远不会使用它。如果您的组织或工作区使用零数据保留或客户管理的加密密钥，则粘性路由不适用。

## 压缩对话

线程是仅追加的，因此您无法用摘要替换其较早的轮次。若要压缩对话，请从摘要开始一个新线程。在您的代码中编写摘要，并在新的创建请求中将其作为普通消息发送，同时附带 `system` 和 `tools`。[按需压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand)无法与线程一起使用（请参阅[线程不支持的功能](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#unsupported)）。

## SDK 辅助工具的工作方式

辅助工具为您完成的工作：

* 为每个使用该线程的请求添加 `message-threads-2026-08-12` beta 标头。
* 在第一轮创建线程，之后继续该线程，仅发送新消息，并省略 `system` 和 `tools`。
* 当继续请求返回 `thread_not_found`、`thread_already_continued` 或 `thread_continue_in_flight` 时，在一个新的创建请求中重放一次完整对话。
* 当您不带任何消息调用 `send()` (go, csharp: `Send()`; ruby: `send_message()`) 时，恢复 `pause_turn` 响应。
* 在拒绝之后启动新的线程。您会像往常一样收到 `stop_reason: "refusal"` 响应，但辅助工具不会保留该轮次：它会撤回您在该次调用中提供的消息。您通过 `thread.append(...)` (csharp, go: `thread.Append(...)`; php: `$thread->append(...)`) 添加的消息，或在创建辅助工具时提供的消息，仍保持暂存状态。您的下一轮会在一个新的创建请求中，将它们与被拒绝轮次之前的对话一起发送。请发送改写后的消息；或者，如果是某条暂存消息导致了拒绝，请从不含该消息的参数启动新的线程。
* 在收到来自回退模型的响应后发送一个新的创建请求。当响应的 `usage.iterations` 中包含 `fallback_message` 条目时，辅助工具会保留该响应，并在下一轮中通过一个新的创建请求发送完整对话。它不会等待 404 错误。该请求会保留您设置的 `model` 和 `fallbacks`。
* 每当再次发送较早轮次时（例如在重放中或在分叉的第一轮中），省略这些轮次的 `cache_control` 断点。您最新的消息、`system`、`tools` 以及顶层 `cache_control` 会按您提供的原样发送。

您需要自己处理的工作：

* **回退。** 当 API 持续由回退模型作答时，辅助工具会在每一轮中通过一个新的创建请求发送完整对话，因为 API 不会存储这些轮次中的任何一个。在收到来自您所请求模型的响应后，它会恢复使用继续请求。若要改为将对话转移到回退模型，请从 params 启动一个新的线程，使用该 `model` 且不带 `fallbacks`，如[在线程中使用回退](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#fallbacks)中所述。
* **中断的流。** 如果流在响应中途结束，辅助工具会保留已完成的块以及已到达的任何文本，下一轮会在新的创建请求中发送完整对话。辅助工具不会自行重试：如果您想改为重新发送被中断的轮次，请不带任何消息调用 `send()` (go, csharp: `Send()`; ruby: `send_message()`)。如果保留的部分以工具调用结尾，请在下一条消息中回应它们。如果您在流显示 `stop_reason: "refusal"` 后停止读取，辅助工具会将该轮次视为拒绝。
* **其他错误。** `thread_fingerprint_mismatch` 和 `thread_unsupported_request` 错误会传递到您的代码。若要在出现 `thread_unsupported_request` 错误后不使用 `thread` 继续，请在创建辅助工具时将 `unsupported_request_behavior` (typescript, java, php: `unsupportedRequestBehavior`; go, csharp: `UnsupportedRequestBehavior`) 设置为 `continue_unthreaded` (ruby: `:continue_unthreaded`; java: `CONTINUE_UNTHREADED`; csharp: `ContinueUnthreaded`; go: `BetaMessageThreadUnsupportedRequestBehaviorContinueUnthreaded`)。使用该设置时，如果重放后的第一个继续请求再次返回 `thread_not_found`，辅助工具也会停止使用 `thread`。API 可能无法存储该对话。如果没有该设置，辅助工具会在每一轮都进行重放。自行创建线程的工具运行器默认使用该设置。

SDK 的 beta 工具运行器默认也使用线程，您可以在创建运行器时将其关闭。

### 其他辅助方法

| 要执行的操作            | 使用                                                                                                                                                                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 流式传输一个轮次          | `thread.stream(...)` (csharp: `thread.SendStreaming(...)`; go: `thread.SendStreaming(ctx, params)`; java: `thread.sendStreaming(...)`; php: `$thread->sendStream(...)`)                                                                                                                                                              |
| 流式传输轮次的完整响应       | `stream.get_final_message()` (typescript: `stream.finalMessage()`; csharp: `thread.LastMessage`; go: `thread.LastMessage()`; java: `thread.lastMessage()`; php: `$thread->lastMessage()`; ruby: `stream.accumulated_message`)                                                                                                        |
| 完整对话              | `thread.messages` (csharp: `thread.Messages`; go: `thread.Messages()`; java: `thread.messages()`; php: `$thread->messages()`)                                                                                                                                                                                                        |
| 下一轮次将从中继续的响应 `id` | `thread.previous_message_id` (typescript: `thread.previousMessageId`; csharp: `thread.PreviousMessageID`; go: `thread.PreviousMessageID()`; java: `thread.previousMessageId()`; php: `$thread->previousMessageId()`)                                                                                                                 |
| 添加消息但不发送          | `thread.append(...)` (csharp, go: `thread.Append(...)`; php: `$thread->append(...)`)                                                                                                                                                                                                                                                 |
| 为后续轮次更改字段         | `thread.update(...)` (csharp: `thread.Update(p => p with { ... })`; go: `thread.Params.MaxTokens = ...`; java: `thread.update(b -> ...)`; php: `$thread->update([...])`)                                                                                                                                                             |
| 复制线程              | `thread.fork()` (csharp, go: `thread.Fork()`; php: `$thread->fork()`; ruby: `thread.fork`)                                                                                                                                                                                                                                           |
| 用于启动新线程的字段和对话     | `thread.params` (csharp, go: `thread.Params`; java: `thread.params()`; php: `$thread->params()`)                                                                                                                                                                                                                                     |
| 此线程上的工具运行器        | `thread.tool_runner()` (typescript, java: `thread.toolRunner()`; csharp: `thread.ToolRunner(tools)`; go: `thread.NewToolRunner(tools)`; php: `$thread->toolRunner()`; ruby: `thread.tool_runner`)                                                                                                                                    |
| 不使用线程的工具运行器       | `tool_runner(..., thread=False)` (typescript: `toolRunner({ ..., thread: false })`; csharp: `ToolRunner(parameters, tools, thread: false)`; go: `BetaToolRunnerParams{DisableThread: true}`; java: `ToolRunnerCreateParams.builder().thread(false)`; php: `toolRunner(..., thread: false)`; ruby: `tool_runner(..., thread: false)`) |

**流式传输。** 辅助工具会在流到达 `message_stop` 时记录响应，因此请将流读取到结束。如果您提前停止读取，请结束该流，以便线程可用于下一轮：

* Python：离开 `with` 块。
* TypeScript：离开循环或调用 `abort()`。
* Go：调用 `Close`。
* Java 和 Ruby：调用 `close()`。
* C#：释放枚举器。离开 `await foreach` 即可完成此操作。
* PHP：调用 `close()`。

在 C# 中，请求会在您开始读取流时发出。

在 C#、Go、Java 和 PHP 中，发生拒绝后，`thread.LastMessage` (go: `thread.LastMessage()`; java: `thread.lastMessage()`; php: `$thread->lastMessage()`) 仍会返回上一轮的响应，如果是第一轮则不返回任何内容。请从流的事件中读取拒绝信息。

**更改字段。** 更新不会发送任何内容，并应用于之后的每一轮。一旦线程有了响应，辅助工具会拒绝更改 `system` 或 `tools` 的轮次，而 API 会拒绝对任何其他固定字段的更改（请参阅[每个请求要发送的内容](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#what-to-send)）。若要更改这些字段之一，请从 params 开始一个新线程，或者复制（fork）该线程并在副本的第一轮之前更新它。

**复制。** 复制会拷贝对话、字段和请求选项，且不发送任何内容。其第一轮会在新的创建请求中发送完整对话，此后两个线程相互独立。复制始终从整个对话开始，而不是从较早的响应开始。

**Params。** params 包含辅助工具的字段和完整对话。将它们传递给创建辅助工具的方法即可启动一个新的线程，例如使用不同的 `system`（请参阅[每个请求要发送的内容](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#what-to-send)），或将 `messages` 截短以丢弃较晚的轮次。从 params 启动的线程会按您提供的原样发送每条消息，包括 `cache_control` 断点，因此请保持在 4 个的限制之内（请参阅[线程上的提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#prompt-caching)）。除 Ruby 外，params 不包含请求选项，因此请再次传递这些选项。

**结构化输出。** 在创建辅助工具时指定输出格式。后续轮次无法更改它。获取解析结果的方式取决于 SDK：

* Python 和 TypeScript：调用 `thread.parse(...)`。
* C#：使用 `client.Beta.Messages.Thread<T>(...)` 创建辅助工具。
* Go：`Send` 会反序列化到 `Params.OutputConfig.Format.Schema` 所指向的结构体中。
* Java：使用 `thread(...)` 和 `outputConfig(MyClass.class)` 创建辅助工具。
* PHP：当 `outputConfig` 指定模型类时，`send()` 会在文本块上设置 `parsed`。
* Ruby：将 `output_config: {format: MyModel}` 传递给 `thread(...)`，然后读取 `reply.parsed_output`。

## 错误

API 会在模型运行之前返回此表中的错误。请求不消耗令牌，线程及其过期时间也不会改变。对于 400 错误，`error.type` 为 `invalid_request_error`；对于 404 错误，则为 `not_found_error`。

| 状态  | `error_code`                  | 含义                                                        | 处理方法                                 |
| --- | ----------------------------- | --------------------------------------------------------- | ------------------------------------ |
| 404 | `thread_not_found`            | API 无法继续该线程。                                              | 在新的创建请求中发送完整对话。                      |
| 400 | `thread_fingerprint_mismatch` | 某个固定字段与线程存储的值不同。错误消息会指出这些字段。                              | 保持固定字段不变，或开始一个新线程来更改某个字段。            |
| 400 | `thread_already_continued`    | 每个响应 `id` 只能被继续一次。                                        | 从线程的最新响应 `id` 继续。如果您的客户端从未收到该响应，请重放。 |
| 400 | `thread_continue_in_flight`   | 从此响应 `id` 发起的另一个继续请求仍在运行。这通常是您自己之前的尝试。                    | 等待其完成并从其响应 `id` 继续，或者重放。             |
| 400 | `thread_unsupported_request`  | 请求使用了线程不支持的功能或组织设置。如果存在，`error.details.unsupported` 会指出它。 | 从请求中删除该功能，或不使用 `thread` 发送完整对话。      |

线程请求的某些 400 错误不带 `error_code`，例如缺少 `previous_message_id` 的继续请求，或包含 `system` 或 `tools` 的继续请求。

以下示例展示了一个错误正文：

```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "thread: a maximum of 4 blocks with cache_control may be provided when `thread` is set. Found 5.",
    "details": {
      "error_code": "thread_unsupported_request",
      "unsupported": ["cache_control"]
    }
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```

## 线程不支持的功能

线程不支持以下功能：

* [Message Batches](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)。包含 `thread` 的请求项会在批处理结果中以 `invalid_request_error` 结束。
* [令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)。包含 `thread` 的令牌计数请求会返回 400 错误。请改为对完整对话（连同 `system` 和 `tools`）进行计数。
* [压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction)。[按需压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand)无法与线程一起使用。包含 `thread` 的请求如果设置了 `compaction` 参数，或者在带有 `compact-2026-09-04` beta 标头的情况下发送 `compaction` 块，则会返回 400 错误。请参阅[压缩对话](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#compaction)。
* [上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)，但带有 `keep: "all"` 的 `clear_thinking_20251015` 除外。
* 已启用 [HIPAA 就绪](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#hipaa-readiness)的组织。
* 已开启[推理钩子](https://platform.claude.com/docs/zh-CN/manage-claude/inference-hooks)的组织。

API 会以 400 错误和代码 `thread_unsupported_request` 拒绝大多数线程不支持的请求。当 `error.details.unsupported` 存在时，它包含以下值之一：

| 值                    | 含义                                                                                                                                    |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `context_management` | 请求使用了线程不支持的上下文管理或压缩。                                                                                                                  |
| `cache_control`      | 请求包含超过 4 个 `cache_control` 断点。请参阅[线程上的提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#prompt-caching)。 |
| `fallbacks`          | 请求设置了 `fallbacks`，而您的组织无法在线程中使用 `fallbacks`。                                                                                          |
| `hipaa`              | 您的组织已启用 HIPAA 就绪。                                                                                                                     |
| `inference_hooks`    | 您的组织已开启推理钩子。                                                                                                                          |

API 可能会向此列表添加值。如果您收到无法识别的值，请不使用 `thread` 发送完整对话。满足多个条件的请求收到的错误只会指出其中一个。

## 限制

* 没有用于列出、读取或删除线程的 API。
* 线程可在 Claude API、[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)、Claude Models on Google Cloud (Anthropic managed) 以及 [Microsoft Foundry 中的 Claude](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) 上使用。

## 定价和速率限制

线程不会改变定价或[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)。

## 数据保留

消息线程不符合 “zero data retention”（零数据保留），即 ZDR 的条件。使用此功能不会影响您对其他符合 ZDR 条件的功能的使用。API 将线程保存在内存中，使用服务器管理的密钥加密，从不将其写入持久存储，并在其最后一次使用后不超过 1 小时内将其删除。

线程保存 API 发送给模型的对话、创建请求中的 `system` 和 `tools`，以及固定字段的指纹（用于检查之后的请求）。它通过引用保存文件，因此在使用线程期间，请确保对话使用的每个文件都保持可用。

有关所有功能的 ZDR 资格，请参阅 [API 和数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)。

## 另请参阅

* [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)
* [流式传输消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)
* [Claude API 错误](https://platform.claude.com/docs/zh-CN/api/errors)

## 可用性

消息线程在所有模型上处于 beta 阶段。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态   | 云平台                                                                                                                                                                                                                                                                                                                                                                                             |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `message-threads-2026-08-12`

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                                                                         | HIPAA |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| 状态   | 不具备资格                                                                                                                                                                                         | 不具备资格 |
| 详细信息 | 线程的内容（对话、系统提示和工具）保存在内存中，使用服务器管理的密钥加密，从不写入持久存储，并在线程最后一次使用后 1 小时内删除。使用此功能不会影响您使用其他具备 ZDR 资格的功能。 [了解更多](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads#data-retention) | —     |

适用的端点：

* `/v1/messages` 使用 `thread`
