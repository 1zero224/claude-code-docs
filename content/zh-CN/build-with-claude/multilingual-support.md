---
title: 多语言支持
url: https://platform.claude.com/docs/zh-CN/build-with-claude/multilingual-support
description: Claude 在多种语言的任务中表现出色，相对于英语保持了强大的跨语言性能。
featureMetadata:
  status: stable
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

## 概述

Claude 展现出强大的多语言能力，尤其在跨语言的 “zero-shot”（零样本）任务中表现出色。

除下表中进行基准测试的语言之外，Claude 还能够处理许多其他语言。请使用与您的具体用例相关的任何语言进行测试。

## 性能数据

下表展示了 Claude 模型在 [Global-MMLU](https://arxiv.org/abs/2412.03304) 上的零样本分数，该测试将 [“Massive Multitask Language Understanding”（大规模多任务语言理解），即 MMLU](https://en.wikipedia.org/wiki/MMLU) 测试集扩展到了 42 种语言。每个分数均为同一模型英语分数的百分比：

| 语言       | Claude Fable 5.2 | Claude Opus 5.5 | Claude Sonnet 5.5 | Claude Haiku 5.5 |
| -------- | ---------------- | --------------- | ----------------- | ---------------- |
| 英语       | 100%             | 100%            | 100%              | 100%             |
| 西班牙语     | 99.8%            | 99.3%           | 99.0%             | 98.1%            |
| 葡萄牙语（巴西） | 99.4%            | 99.2%           | 98.7%             | 98.1%            |
| 意大利语     | 99.5%            | 99.2%           | 98.4%             | 97.7%            |
| 法语       | 99.6%            | 99.3%           | 98.7%             | 97.7%            |
| 印度尼西亚语   | 99.4%            | 99.1%           | 98.6%             | 97.1%            |
| 德语       | 99.5%            | 99.1%           | 98.5%             | 97.4%            |
| 阿拉伯语     | 99.6%            | 99.3%           | 98.3%             | 96.6%            |
| 中文（简体）   | 99.6%            | 99.3%           | 98.5%             | 97.7%            |
| 韩语       | 99.6%            | 99.1%           | 98.3%             | 97.0%            |
| 日语       | 99.5%            | 99.2%           | 98.4%             | 97.4%            |
| 印地语      | 99.4%            | 99.2%           | 98.5%             | 96.6%            |
| 孟加拉语     | 99.2%            | 98.6%           | 97.2%             | 95.4%            |
| 斯瓦希里语    | 99.2%            | 98.3%           | 96.9%             | 94.0%            |
| 约鲁巴语     | 98.5%            | 96.3%           | 93.6%             | 87.0%            |

所有分数均在 `max` [effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)下使用 [“adaptive thinking”（自适应思考）](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)得出。分数不包括被 “safety classifier”（安全分类器）拦截或未获回答的问题。对于表中的每个分数，被排除的问题至多占 1.7%。

<Note>
  请使用此表比较同一模型内不同语言的表现，而不是对模型进行排名。小于约 1 个百分点的差异属于测量噪声范围内。
</Note>

***

## 设置响应语言

Claude 会从对话中推断响应语言，但对于生产应用，您应明确指定目标语言。最可靠的做法是在 “system prompt”（系统提示）中进行指定，这样可以使该指令在对话的每一轮中保持稳定。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "system": "Always respond in French, regardless of the language the user writes in.",
      "messages": [
        {"role": "user", "content": "How do I reset my password?"}
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --system "Always respond in French, regardless of the language the user writes in." \
    --message '{role: user, content: "How do I reset my password?"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      system="Always respond in French, regardless of the language the user writes in.",
      messages=[{"role": "user", "content": "How do I reset my password?"}],
  )

  print(message.content)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    system: "Always respond in French, regardless of the language the user writes in.",
    messages: [{ role: "user", content: "How do I reset my password?" }]
  });

  console.log(message.content);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      System = "Always respond in French, regardless of the language the user writes in.",
      Messages = [new UserMessageParam { Content = "How do I reset my password?" }]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  client := anthropic.NewClient()

  message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	System: []anthropic.TextBlockParam{
  		{Text: "Always respond in French, regardless of the language the user writes in."},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("How do I reset my password?")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }
  fmt.Println(message.Content)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  MessageCreateParams params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024)
      .system("Always respond in French, regardless of the language the user writes in.")
      .addUserMessage("How do I reset my password?")
      .build();

  Message message = client.messages().create(params);
  System.out.println(message.content());
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => 'How do I reset my password?']
      ],
      model: Model::CLAUDE_OPUS_5_5,
      system: 'Always respond in French, regardless of the language the user writes in.',
  );

  echo json_encode($message->content, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    system: "Always respond in French, regardless of the language the user writes in.",
    messages: [
      { role: "user", content: "How do I reset my password?" }
    ]
  )

  puts message.content
  ```
</CodeGroup>

如果您的应用允许用户在运行时选择语言，请将该选择插入到系统提示中，而不是依赖 Claude 从用户消息中推断。若要在两种特定语言之间进行翻译，请同时指明两种语言：`Translate the user's message from German to Korean. Respond with only the translation.`

***

## 最佳实践

处理多语言内容时：

1. **提供清晰的语言上下文：** 虽然 Claude 可以自动检测目标语言，但明确说明您想要的输入和输出语言可以提高可靠性。为了增强流畅度，您可以提示 Claude 使用"如同母语者般的地道表达"。
2. **使用原生文字：** 提交文本时请使用其原生文字而非音译，以获得最佳效果。
3. **考虑文化背景：** 有效的沟通通常需要超越纯粹翻译的文化和地区意识。

另请遵循[提示工程概述](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/overview)中的通用指导，以进一步提升输出质量。

***

## 语言支持注意事项

* Claude 可以处理大多数使用标准 Unicode 字符的世界语言的输入并生成输出。
* 性能因语言而异，在广泛使用的语言中能力尤为突出。
* 即使在数字资源较少的语言中，Claude 仍保持有意义的能力。

## 后续步骤

<CardGroup cols={2}>
  <Card title="提示工程概述" icon="edit" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/overview">
    应用通用提示技巧以提升多语言输出质量。
  </Card>

  <Card title="客户支持智能体" icon="headset" href="https://platform.claude.com/docs/zh-CN/about-claude/use-case-guides/customer-support-chat">
    构建一个客户支持聊天机器人。一个聊天机器人即可为使用多种语言的客户提供服务。
  </Card>

  <Card title="模型概述" icon="table" href="https://platform.claude.com/docs/zh-CN/models/overview">
    比较各模型层级，在多语言质量与成本和延迟之间取得平衡。
  </Card>

  <Card title="定义成功标准并构建评估" icon="scales" href="https://platform.claude.com/docs/zh-CN/test-and-evaluate/develop-tests">
    应用通用的评估方法来衡量您所支持的每种语言的输出质量。
  </Card>
</CardGroup>

## 可用性

多语言支持在所有模型上可用。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：始终开启

| 状态 | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：始终开启

| 状态 | 云平台                                                                                                                   |
| -- | --------------------------------------------------------------------------------------------------------------------- |
| 可用 | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |
