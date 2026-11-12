---
title: 提示最佳实践
url: https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices
description: 面向 Claude 最新模型的提示工程技术综合指南，涵盖清晰性、示例、XML 结构化、思考以及智能体系统。
---

本文是使用当前 Claude 模型进行 “prompt engineering”（提示工程）的参考资料，适用模型包括 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5、Claude Opus 5.5、Claude Opus 5、Claude Opus 4.8、Claude Opus 4.7、Claude Opus 4.6、Claude Sonnet 5.5、Claude Sonnet 5、Claude Sonnet 4.6、Claude Haiku 5.5 和 Claude Haiku 4.5。本页分为三个部分：

* 首先是 **[特定模型指南](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices#model-specific-guidance)**：说明某个模型在哪些方面表现不同，以及您需要在提示中做出哪些调整。
* 其后是 **适用于所有当前模型的技术**：通用原则、输出与格式、工具使用、思考以及智能体系统。
* 最后是 **迁移注意事项**，适用于从早期模型迁移过来的提示。

<Tip>
  有关模型能力的概述，请参阅[模型概述](https://platform.claude.com/docs/zh-CN/models/overview)。有关 Claude Fable 5.2 的能力和 API 变更，请参阅 [Claude Fable 5.2 的新功能](https://platform.claude.com/docs/zh-CN/models/fable-5-2/whats-new-fable-5-2)。有关迁移指导，请参阅[迁移指南](https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide)。有关 Claude Opus 5.5，请参阅 [Claude Opus 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/opus-5-5/whats-new-opus-5-5)。有关 Claude Sonnet 5.5，请参阅 [Claude Sonnet 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/whats-new-sonnet-5-5)。有关 Claude Haiku 5.5，请参阅 [Claude Haiku 5.5 的新功能](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/whats-new-haiku-5-5)。
</Tip>

## 特定模型指南

以下每个模型都有各自的提示页面。请先阅读与您所用模型对应的页面，然后再阅读后续的技术内容。

| 模型                                   | 指南                                                                                                                                  | 差异之处                                                                                                                             |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Claude Fable 5.2 和 Claude Mythos 5.2 | [为 Claude Fable 5.2 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2)   | 与早期模型的差异：effort 校准、完成长任务、审批、代码所有权、面向用户的进度更新、子智能体委派、逐渐累积的谨慎、为早期模型设计的提示和 harness、最终报告以及安全防护拒绝。                                     |
| Claude Fable 5.1 和 Claude Mythos 5.1 | [为 Claude Fable 5.1 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-1)   | 与 Claude Fable 5 的差异：effort 级别、完成长任务、面向用户的进度更新、原样回传思考块、智能体循环中的工具调用批处理、低 effort 下的搜索触发、格式以及写作密度。                                  |
| Claude Fable 5 和 Claude Mythos 5     | [为 Claude Fable 5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5)       | 与 Claude Opus 4.8 的差异：effort 级别、指令遵循、长时间运行中的进度声明、记忆系统，以及 `reasoning_extraction` 拒绝类别。                                            |
| Claude Sonnet 5.5                    | [为 Claude Sonnet 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5) | 与 Claude Sonnet 5 的差异：effort 校准、主动性与范围、在没有前置思考的情况下运行、推理任务中的 JSON 输出、面向用户的进度更新、聊天中的工具使用、轮次中途的用户消息、编码任务中的验证、工具调用处理、复杂视觉输入以及安全防护拒绝。 |
| Claude Sonnet 5                      | [为 Claude Sonnet 5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5)     | 与 Claude Sonnet 4.6 的差异：响应长度、effort 与思考深度校准、工具使用触发、字面化的指令遵循，以及设计和前端默认行为。                                                         |
| Claude Opus 5.5                      | [为 Claude Opus 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5)     | 与 Claude Opus 5 的差异：effort 校准、为禁用思考而编写的提示、面向用户的进度更新、安全防护拒绝，以及用于复杂视觉输入的工具。                                                        |
| Claude Opus 5                        | [为 Claude Opus 5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5)         | 与之前 Opus 模型的差异：响应长度与冗长程度、面向用户的进度更新、书面交付物长度、任务范围与过度验证、子智能体控制以及自我纠正。                                                               |
| Claude Opus 4.8                      | [为 Claude Opus 4.8 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-4-8)     | 响应长度、effort 与思考深度校准、工具使用触发、字面化的指令遵循、子智能体控制，以及设计和前端默认行为。                                                                          |
| Claude Haiku 5.5                     | [为 Claude Haiku 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5)   | 与 Claude Haiku 4.5 的差异：effort 级别、搜索、使用您自己的工具时的 JSON 输出、长智能体提示中的过早停止、编码任务中的验证、轮次中途的用户消息、聊天机器人中的系统提示遵循以及安全防护拒绝。                    |

## 通用原则

本节及后续各节中的技术适用于当前的 Claude 模型，包括 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5 和 Claude Mythos 5。如果某项技术指明了特定模型，请将其视为在该模型上测得的结果，并在应用于其他模型之前，先用您自己的评估重新验证。

### 清晰直接

Claude 对清晰、明确的指令响应良好。明确说明您想要的输出有助于提升结果。如果您希望 Claude 做的超出任务的严格要求，请明确提出，而不是依赖模型从模糊的提示中推断。

请把 Claude 想象成一位才华出众但刚入职的新员工，他对您的规范和工作流程缺乏了解。您对需求的解释越精确，结果就越好。

**关键测试：** 把您的提示拿给一位对该任务几乎不了解的同事看，让他照着执行。如果他会感到困惑，Claude 也会。

* 明确说明您想要的输出格式和约束条件。
* 当步骤的顺序或完整性很重要时，使用编号列表或项目符号将指令写成按顺序排列的步骤。

<Accordion title="示例：创建分析仪表盘" defaultOpen>
  **效果较差：**

  ```text wrap
  Create an analytics dashboard
  ```

  **效果更好：**

  ```text wrap
  Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation.
  ```
</Accordion>

### 添加上下文以提升表现

为您的指令提供上下文或动机，例如向 Claude 解释为什么某种行为很重要，可以让 Claude 更多地了解您的目标，帮助它给出更有针对性的响应。

<Accordion title="示例：格式偏好" defaultOpen>
  **效果较差：**

  ```text wrap
  NEVER use ellipses
  ```

  **效果更好：**

  ```text wrap
  Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them.
  ```
</Accordion>

Claude 能够从解释中进行泛化。

### 有效使用示例

示例是引导 Claude 输出格式、语气和结构最可靠的方法之一。几个精心设计的示例（称为 “few-shot” 或 “multishot prompting”，即少样本或多样本提示）可以提高准确性和一致性。

添加示例时，请确保它们：

* **相关：**&#x7D27;密贴合您的实际用例。
* **多样：**&#x8986;盖边界情况，并具有足够的差异性，以免 Claude 学到非预期的模式。
* **结构化：**&#x5C06;示例包裹在 `<example>` 标签中（多个示例放在 `<examples>` 标签中），以便 Claude 将其与指令区分开来。

<Tip>
  包含 3–5 个示例可获得最佳效果。您也可以请 Claude 评估您的示例的相关性和多样性，或基于您的初始示例集生成更多示例。
</Tip>

### 使用 XML 标签构建提示结构

XML 标签有助于 Claude 无歧义地解析复杂提示，尤其是当您的提示混合了指令、上下文、示例和可变输入时。将每种类型的内容包裹在各自的标签中（例如 `<instructions>`、`<context>`、`<input>`）可以减少误解。

最佳实践：

* 在您的各个提示中使用一致且具有描述性的标签名称。
* 当内容具有天然的层级结构时嵌套标签（文档放在 `<documents>` 内，每个文档放在 `<document index="n">` 内）。

### 为 Claude 设定角色

在 system prompt（系统提示）中设定角色，可以使 Claude 的行为和语气聚焦于您的用例。即使只有一句话也会产生效果：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "system": "You are a helpful coding assistant specializing in Python.",
      "messages": [
        {"role": "user", "content": "How do I sort a list of dictionaries by key?"}
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --system "You are a helpful coding assistant specializing in Python." \
    --message '{role: user, content: "How do I sort a list of dictionaries by key?"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      system="You are a helpful coding assistant specializing in Python.",
      messages=[
          {"role": "user", "content": "How do I sort a list of dictionaries by key?"}
      ],
  )

  print(message.content)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    system: "You are a helpful coding assistant specializing in Python.",
    messages: [{ role: "user", content: "How do I sort a list of dictionaries by key?" }]
  });

  console.log(message.content);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      System = "You are a helpful coding assistant specializing in Python.",
      Messages = [new UserMessageParam { Content = "How do I sort a list of dictionaries by key?" }]
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
  		{Text: "You are a helpful coding assistant specializing in Python."},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("How do I sort a list of dictionaries by key?")),
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
      .system("You are a helpful coding assistant specializing in Python.")
      .addUserMessage("How do I sort a list of dictionaries by key?")
      .build();

  Message message = client.messages().create(params);
  System.out.println(message.content());
  ```

  ```php PHP
  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [
          ['role' => 'user', 'content' => 'How do I sort a list of dictionaries by key?']
      ],
      model: Model::CLAUDE_OPUS_5_5,
      system: 'You are a helpful coding assistant specializing in Python.',
  );

  echo json_encode($message->content, JSON_PRETTY_PRINT), PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    system: "You are a helpful coding assistant specializing in Python.",
    messages: [
      { role: "user", content: "How do I sort a list of dictionaries by key?" }
    ]
  )

  puts message.content
  ```
</CodeGroup>

### 长上下文提示

在处理大型文档或数据密集型输入（20k+ 令牌）时，请仔细构建提示结构以获得最佳结果：

* **将长篇数据放在顶部：** 将长文档和输入放在提示的靠前位置，位于查询、指令和示例之上。这可以提升所有模型的表现。

  <Note>
    在测试中，将查询放在末尾可将响应质量提升最多 30%，尤其是在处理复杂的多文档输入时。
  </Note>

* **使用 XML 标签组织文档内容和元数据：**&#x4F7F;用多个文档时，将每个文档包裹在 `<document>` 标签中，并使用 `<document_content>` 和 `<source>`（以及其他元数据）子标签以保持清晰。

  <Accordion title="多文档结构示例">
    ```xml
    <documents>
      <document index="1">
        <source>annual_report_2023.pdf</source>
        <document_content>
          {{ANNUAL_REPORT}}
        </document_content>
      </document>
      <document index="2">
        <source>competitor_analysis_q2.xlsx</source>
        <document_content>
          {{COMPETITOR_ANALYSIS}}
        </document_content>
      </document>
    </documents>

    Analyze the annual report and competitor analysis. Identify strategic advantages and recommend Q3 focus areas.
    ```
  </Accordion>

* **以引文为依据作答：**&#x5BF9;于长文档任务，请让 Claude 在执行任务之前先引用文档中的相关部分。这有助于 Claude 聚焦于相关内容并忽略文档的其余部分。

  <Accordion title="引文提取示例">
    ```xml
    You are an AI physician's assistant. Your task is to help doctors diagnose possible patient illnesses.

    <documents>
      <document index="1">
        <source>patient_symptoms.txt</source>
        <document_content>
          {{PATIENT_SYMPTOMS}}
        </document_content>
      </document>
      <document index="2">
        <source>patient_records.txt</source>
        <document_content>
          {{PATIENT_RECORDS}}
        </document_content>
      </document>
      <document index="3">
        <source>patient01_appt_history.txt</source>
        <document_content>
          {{PATIENT01_APPOINTMENT_HISTORY}}
        </document_content>
      </document>
    </documents>

    Find quotes from the patient records and appointment history that are relevant to diagnosing the patient's reported symptoms. Place these in <quotes> tags. Then, based on these quotes, list all information that would help the doctor diagnose the patient's symptoms. Place your diagnostic information in <info> tags.
    ```
  </Accordion>

### 模型自我认知

如果您希望 Claude 在您的应用中正确标识自身或使用特定的 API 字符串：

```text Sample prompt for model identity wrap
The assistant is Claude, created by Anthropic. The current model is Claude Opus 5.5.
```

对于需要指定模型字符串的 LLM 驱动应用：

```text Sample prompt for model string wrap
When an LLM is needed, please default to Claude Opus 5.5 unless the user requests
otherwise. The exact model string for Claude Opus 5.5 is claude-opus-5-5.
```

## 输出与格式

### 沟通风格与冗长程度

与之前的模型相比，Claude 的最新模型具有更简洁、更自然的沟通风格：

* **更直接、更务实：** 提供基于事实的进度报告，而不是自我庆祝式的更新
* **更具对话感：** 略微更流畅、更口语化，不那么机械
* **更简洁：** 出于效率考虑，除非另有提示，否则可能会省略详细摘要

这意味着 Claude 可能会在工具调用后省略口头总结，直接进入下一个操作。如果您希望更清楚地了解它的工作过程：

```text Sample prompt wrap
After completing a task that involves tool use, provide a quick summary of the work you've done.
```

Claude Opus 5 在冗长程度方面是个例外：它默认面向用户的响应比之前的模型更长，而且提高或降低 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 并不能可靠地改变可见的响应长度。请改为在提示中明确要求简洁。示例指令请参阅[为 Claude Opus 5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5#response-length-and-verbosity)。Claude Fable 5.1 在智能体工作中则有相反的倾向：它在工具调用之间写的面向用户的更新更少。请明确要求提供进度文本，并删除任何要求其保持简短的指令。请参阅[要求提供面向用户的进度更新](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-1#ask-for-user-facing-progress-updates)。有关 Claude Fable 5.2，请参阅[在长轮次中让用户了解进展](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#keep-users-informed-during-long-turns)。

### 控制响应格式

有几种特别有效的方法可以引导输出格式：

1. **告诉 Claude 该做什么，而不是不该做什么**

   * 不要说："不要在响应中使用 markdown"
   * 尝试说："您的响应应由流畅连贯的散文段落组成。"

2. **使用 XML 格式指示符**

   * 尝试说："将响应中的散文部分写在 \<smoothly\_flowing\_prose\_paragraphs> 标签中。"

3. **使提示风格与您想要的输出相匹配**

   提示中使用的格式风格可能会影响 Claude 的响应风格。如果您在输出格式的可引导性方面仍有问题，请尝试让您的提示风格尽可能接近您想要的输出风格。例如，从提示中删除 markdown 可以减少输出中 markdown 的数量。

4. **针对特定格式偏好使用详细提示**

   如需对 markdown 和格式使用进行更多控制，请提供明确的指导：

````text Sample prompt to minimize markdown wrap
<avoid_excessive_markdown_and_bullet_points>
When writing reports, documents, technical explanations, analyses, or any long-form
content, write in clear, flowing prose using complete paragraphs and sentences. Use
standard paragraph breaks for organization and reserve markdown primarily for `inline
code`, code blocks (```...```), and simple headings (## and ###). Avoid using **bold**
and *italics*.

DO NOT use ordered lists (1. ...) or unordered lists (*) unless: a) you're presenting
truly discrete items where a list format is the best option, or b) the user explicitly
requests a list or ranking

Instead of listing items with bullets or numbers, incorporate them naturally into
sentences. This guidance applies especially to technical writing. Using prose instead of
excessive formatting will improve user satisfaction. NEVER output a series of overly
short bullet points.

Your goal is readable, flowing text that guides the reader naturally through ideas
rather than fragmenting information into isolated points.
</avoid_excessive_markdown_and_bullet_points>
````

Claude Fable 5.1 的格式化程度本就低于早期模型，因此在该模型上，像这样的指令块可能会抑制内容所需的结构。请将其删除，或替换为[聊天中的格式](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-1#formatting-in-chat)中更简短的规则。

### LaTeX 输出

Claude 的最新模型默认使用 LaTeX 来表示数学表达式、方程和技术说明。如果您更喜欢纯文本，请在提示中添加以下指令：

```text Sample prompt wrap
Format your response in plain text only. Do not use LaTeX, MathJax, or any markup
notation such as \( \), $, or \frac{}{}. Write all math expressions using standard text
characters (e.g., "/" for division, "*" for multiplication, and "^" for exponents).
```

### 文档创建

Claude 的最新模型在创建演示文稿、动画和可视化文档时具有很强的指令遵循能力，通常第一次尝试就能产出可用的结果。

要在文档创建中获得最佳效果：

```text Sample prompt wrap
Create a professional presentation on [topic]. Include thoughtful design elements,
visual hierarchy, and engaging animations where appropriate.
```

### 从预填充响应迁移

从 Claude 4.6 模型和 [Claude Mythos Preview](https://anthropic.com/glasswing) 开始，不再支持在最后一个 assistant 轮次上使用预填充响应（即提供部分 assistant 消息供 Claude 续写）。向这些模型发送带有预填充 assistant 消息的请求将返回 400 错误。模型智能和指令遵循能力已经进步到大多数预填充用例不再需要它的程度。早期模型继续支持预填充，在对话其他位置添加 assistant 消息不受影响。

以下是常见的预填充场景以及如何从中迁移：

<Accordion title="控制输出格式">
  预填充曾被用于强制特定的输出格式，例如 JSON/YAML、分类以及类似模式，通过预填充将 Claude 约束在特定结构中。

  **迁移：** [结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)功能专门用于约束 Claude 的响应遵循给定的 schema。请先尝试要求模型遵循您的输出结构，因为较新的模型在被告知时能够可靠地匹配复杂的 schema，尤其是在配合重试机制实现时。对于分类任务，可以使用带有包含有效标签的 enum 字段的工具，或者使用结构化输出。
</Accordion>

<Accordion title="消除开场白">
  诸如 `Here is the requested summary:\n` 之类的预填充曾被用于跳过介绍性文字。

  **迁移方法：**&#x5728;系统提示中使用直接指令："直接回答，不要开场白。不要以'以下是……’、'根据……’等短语开头。"或者，指示模型在 XML 标签内输出、使用结构化输出，或使用工具调用。如果偶尔仍有开场白漏出，请在后处理中将其去除。
</Accordion>

<Accordion title="避免不当拒绝">
  预填充曾被用于绕开不必要的拒绝。

  **迁移方法：**&#x43;laude 现在在恰当拒绝方面表现好得多。在 `user` 消息中给出清晰的提示而不使用预填充应该就足够了。
</Accordion>

<Accordion title="续写">
  预填充曾被用于继续未完成的补全、恢复被中断的响应，或从上一次生成结束的位置继续扩展。

  **迁移方法：**&#x5C06;续写移至 user 消息中，并包含被中断响应的末尾文本："您之前的响应被中断，结尾是 \`\[previous\_response]\`。请从中断处继续。"如果这是错误处理或不完整响应处理的一部分，且没有用户体验上的代价，请重试该请求。
</Accordion>

<Accordion title="上下文补充与角色一致性">
  预填充曾被用于定期确保上下文得到刷新或注入。

  **迁移方法：**&#x5BF9;于非常长的对话，将之前通过预填充 assistant 消息实现的提醒注入到 user 轮次中。如果上下文补充是更复杂的智能体系统的一部分，请考虑通过工具进行补充（根据轮次数等启发式规则，暴露或鼓励使用包含上下文的工具），或在[上下文压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction)期间进行补充。
</Accordion>

## 工具使用

### 工具用法

Claude 的最新模型经过精确指令遵循的训练，明确指示其使用特定工具会有所助益。如果您说"你能建议一些修改吗"，Claude 有时会提供建议而不是直接实施，即使进行修改可能正是您的本意。要了解如何定义工具以及排查工具触发问题，请参阅 [Claude 的 tool use（工具使用）](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)。

要让 Claude 采取行动，请更加明确：

<Accordion title="示例：明确的指令" defaultOpen>
  **效果较差（Claude 只会给出建议）：**

  ```text wrap
  Can you suggest some changes to improve this function?
  ```

  **效果更好（Claude 会进行修改）：**

  ```text wrap
  Change this function to improve its performance.
  ```

  或者：

  ```text wrap
  Make these edits to the authentication flow.
  ```
</Accordion>

要让 Claude 默认更主动地采取行动，您可以将以下内容添加到系统提示中：

```text Sample prompt for proactive action wrap
<default_to_action>
By default, implement changes rather than only suggesting them. If the user's intent is
unclear, infer the most useful likely action and proceed, using tools to discover any
missing details instead of guessing. Try to infer the user's intent about whether a tool
call (e.g., file edit or read) is intended or not, and act accordingly.
</default_to_action>
```

另一方面，如果您希望模型默认更加谨慎，不那么倾向于立即开始实施，只在被要求时才采取行动，可以使用如下提示来引导这种行为：

```text Sample prompt for conservative action wrap
<do_not_act_before_instructions>
Do not jump into implementation or change files unless clearly instructed to make
changes. When the user's intent is ambiguous, default to providing information, doing
research, and providing recommendations rather than taking action. Only proceed with
edits, modifications, or implementations when the user explicitly requests them.
</do_not_act_before_instructions>
```

Claude Opus 4.5 和 Claude Opus 4.6 对系统提示的响应也比先前模型更敏感。如果您的提示原本是为了减少工具或技能触发不足而设计的，这些模型现在可能会过度触发。解决方法是收敛任何激进的措辞。原本您可能会说"关键：您必须在……时使用此工具"，现在可以使用更平常的提示，如"在……时使用此工具"。

### 优化并行工具调用

Claude 的最新模型会并行运行相互独立的工具调用。这些模型会：

* 在研究过程中运行多个推测性搜索
* 一次读取多个文件以更快地构建上下文
* 并行运行 bash 命令（这甚至可能成为系统性能的瓶颈）

这种行为是可引导的。尽管模型在没有提示的情况下并行工具调用的成功率已经很高，但您可以将其提升至接近 100%，或调整其积极程度：

```text Sample prompt for maximum parallel efficiency wrap
<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between the tool
calls, make all of the independent tool calls in parallel. Prioritize calling tools
simultaneously whenever the actions can be done in parallel rather than sequentially.
For example, when reading 3 files, run 3 tool calls in parallel to read all 3 files into
context at the same time. Maximize use of parallel tool calls where possible to increase
speed and efficiency. However, if some tool calls depend on previous calls to inform
dependent values like the parameters, do NOT call these tools in parallel and instead
call them sequentially. Never use placeholders or guess missing parameters in tool
calls.
</use_parallel_tool_calls>
```

```text Sample prompt to reduce parallel execution wrap
Execute operations sequentially with brief pauses between each step to ensure stability.
```

在 Claude Fable 5.1 的长智能体循环中，请在每轮工具结果之后，将并行调用指令作为轮次范围的系统消息发送。请参阅[在智能体循环中批量处理独立的工具调用](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-1#batch-independent-tool-calls-in-agent-loops)。

## 思考与推理

### 过度思考与过度周全

Claude Opus 4.6 比之前的模型进行更多的前期探索，尤其是在较高的 [`effort`](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 设置下。这些初始工作通常有助于优化最终结果，但模型可能会在未经提示的情况下收集大量上下文或同时展开多条研究线索。如果您的提示以前鼓励模型更加彻底，则应针对 Claude Opus 4.6 调整这些指导：

* **用更有针对性的指令替换笼统的默认设置。**&#x4E0D;要说"默认使用 \[tool]"，而是添加类似"当 \[tool] 能增进您对问题的理解时使用它"的指导。
* **移除过度提示。**&#x5728;先前模型中触发不足的工具现在很可能会恰当触发。像"如有疑问，请使用 \[tool]"这样的指令会导致过度触发。
* **将 effort 作为后备手段。**&#x5982;果 Claude 仍然过于激进，请使用较低的 `effort` 设置。

在某些情况下，Claude Opus 4.6 可能会进行大量思考，这会增加思考令牌的消耗并减慢响应速度。如果不希望出现这种行为，您可以添加明确的指令来约束其推理，或者降低 `effort` 设置以减少整体思考和令牌用量。

```text Sample prompt wrap
When you're deciding how to approach a problem, choose an approach and commit to it.
Avoid revisiting decisions unless you encounter new information that directly
contradicts your reasoning. If you're weighing two approaches, pick one and see it
through. You can always course-correct later if the chosen approach fails.
```

如果您需要对思考成本设置硬性上限，带 `budget_tokens` 上限的 extended thinking（扩展思考）在 Opus 4.6 和 Sonnet 4.6 上仍然可用，但已被弃用。在 Claude 4.7 及更高版本的模型上，设置 `budget_tokens` 会返回 400 错误。建议优先降低 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 设置，或配合[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)使用 `max_tokens` 作为硬性限制。

### 利用思考与交错思考能力

Claude 的最新模型提供思考能力，这对于涉及工具使用后反思或复杂多步推理的任务特别有帮助。您可以引导其初始思考或 “interleaved thinking”（交错思考）以获得更好的结果。

Claude 4.6 及更高版本的模型以及 Claude Mythos Preview 使用[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)（`thinking: {type: "adaptive"}`），由 Claude 动态决定何时思考以及思考多少。在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5 和 Claude Opus 5.5 上，思考始终开启，自适应思考是唯一的模式。Claude 根据两个因素校准其思考：`effort` 参数和查询复杂度。更高的 effort 会引发更多思考，更复杂的查询也是如此。对于不需要思考的较简单查询，模型会直接响应。Claude Fable 5.2 和 Claude Mythos 5.2 是例外：在 `high` 及更高 effort 下，它们几乎在每次回复之前都会进行思考。在内部评估中，自适应思考始终比扩展思考带来更好的表现。请考虑迁移到自适应思考。

对于需要智能体行为的工作负载，例如多步工具使用、复杂编码任务和长周期智能体循环，请使用自适应思考。较旧的模型使用带有 `budget_tokens` 的手动[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)；有关每个模型接受哪种配置，请参阅[各模型配置表](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting#supported-models)。

您可以引导 Claude 的思考行为：

```text Example prompt wrap
After receiving tool results, carefully reflect on their quality and determine optimal
next steps before proceeding. Use your thinking to plan and iterate based on this new
information, and then take the best next action.
```

自适应思考的触发行为可以通过提示来控制。如果您发现模型思考的频率高于预期（在系统提示较大或较复杂时可能会出现这种情况），请添加指导来引导它：

```text Sample prompt wrap
Thinking adds latency and should only be used when it will meaningfully improve
answer quality - typically for problems that require multistep reasoning. When in
doubt, respond directly.
```

在 Claude Fable 5.2 上，请改为降低 effort 级别。在 Anthropic 的测试中，要求直接回答而不加斟酌的指令并不能阻止模型在 `medium` 及更高 effort 下进行思考。请参阅[校准 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2#calibrate-effort)。

如果您正在从带有 `budget_tokens` 的[扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)迁移，请替换您的思考配置，并将预算控制转移到 `effort`。以下示例展示了迁移前后的同一请求（有关可用级别和各模型的可用性，请参阅 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)）：

<CodeGroup>
  ```bash cURL
  # 之前：使用手动预算的扩展思考（旧模型）
  curl https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-sonnet-4-5-20250929",
      "max_tokens": 16000,
      "thinking": {"type": "enabled", "budget_tokens": 10000},
      "messages": [
        {"role": "user", "content": "..."}
      ]
    }'

  # 之后：使用 effort 的自适应思考
  curl https://api.anthropic.com/v1/messages \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 16000,
      "thinking": {"type": "adaptive"},
      "output_config": {"effort": "high"},
      "messages": [
        {"role": "user", "content": "..."}
      ]
    }'
  ```

  ```bash CLI
  # 之前：使用手动预算的扩展思考（旧模型）
  ant messages create <<'YAML'
  model: claude-sonnet-4-5-20250929
  max_tokens: 16000
  thinking:
    type: enabled
    budget_tokens: 10000
  messages:
    - role: user
      content: "..."
  YAML

  # 之后：使用 effort 的自适应思考
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
  # 之前：使用手动预算的扩展思考（旧模型）
  client.messages.create(
      model="claude-sonnet-4-5-20250929",
      max_tokens=16000,
      thinking={"type": "enabled", "budget_tokens": 10000},
      messages=[{"role": "user", "content": "..."}],
  )

  # 之后：使用 effort 的自适应思考
  client.messages.create(
      model="claude-opus-5-5",
      max_tokens=16000,
      thinking={"type": "adaptive"},
      output_config={"effort": "high"},
      messages=[{"role": "user", "content": "..."}],
  )
  ```

  ```typescript TypeScript
  // 之前：使用手动预算的扩展思考（旧模型）
  await client.messages.create({
    model: "claude-sonnet-4-5-20250929",
    max_tokens: 16000,
    thinking: { type: "enabled", budget_tokens: 10000 },
    messages: [{ role: "user", content: "..." }]
  });

  // 之后：使用 effort 的自适应思考
  await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 16000,
    thinking: { type: "adaptive" },
    output_config: { effort: "high" },
    messages: [{ role: "user", content: "..." }]
  });
  ```

  ```csharp C#
  // 之前：使用手动预算的 extended thinking（扩展思考）（旧版模型）
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeSonnet4_5_20250929,
      MaxTokens = 16000,
      Thinking = new ThinkingConfigEnabled(budgetTokens: 10000),
      Messages = [new UserMessageParam { Content = "..." }]
  });

  // 之后：使用 effort 的 adaptive thinking（自适应思考）
  await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 16000,
      Thinking = new ThinkingConfigAdaptive(),
      OutputConfig = new OutputConfig { Effort = Effort.High },
      Messages = [new UserMessageParam { Content = "..." }]
  });
  ```

  ```go Go
  // 之前：使用手动预算的扩展思考（旧模型）
  client.Messages.New(ctx, anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeSonnet4_5_20250929,
  	MaxTokens: 16000,
  	Thinking: anthropic.ThinkingConfigParamUnion{
  		OfEnabled: &anthropic.ThinkingConfigEnabledParam{BudgetTokens: 10000},
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("...")),
  	},
  })

  // 之后：使用 effort 的自适应思考
  client.Messages.New(ctx, anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 16000,
  	Thinking: anthropic.ThinkingConfigParamUnion{
  		OfAdaptive: &anthropic.ThinkingConfigAdaptiveParam{},
  	},
  	OutputConfig: anthropic.OutputConfigParam{
  		Effort: anthropic.OutputConfigEffortHigh,
  	},
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("...")),
  	},
  })
  ```

  ```java Java
  // 之前：使用手动预算的扩展思考（旧模型）
  client.messages().create(MessageCreateParams.builder()
      .model(Model.CLAUDE_SONNET_4_5_20250929)
      .maxTokens(16000L)
      .thinking(ThinkingConfigEnabled.builder()
          .budgetTokens(10000L)
          .build())
      .addUserMessage("...")
      .build());

  // 之后：使用 effort 的自适应思考
  client.messages().create(MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(16000L)
      .thinking(ThinkingConfigAdaptive.builder().build())
      .outputConfig(OutputConfig.builder()
          .effort(OutputConfig.Effort.HIGH)
          .build())
      .addUserMessage("...")
      .build());
  ```

  ```php PHP
  // 之前：使用手动预算的扩展思考（旧模型）
  $client->messages->create(
      model: 'claude-sonnet-4-5-20250929',
      maxTokens: 16000,
      thinking: ['type' => 'enabled', 'budget_tokens' => 10000],
      messages: [['role' => 'user', 'content' => '...']],
  );

  // 之后：使用 effort 的自适应思考
  $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 16000,
      thinking: ['type' => 'adaptive'],
      outputConfig: ['effort' => 'high'],
      messages: [['role' => 'user', 'content' => '...']],
  );
  ```

  ```ruby Ruby
  # 之前：使用手动预算的扩展思考（旧模型）
  client.messages.create(
    model: "claude-sonnet-4-5-20250929",
    max_tokens: 16000,
    thinking: { type: "enabled", budget_tokens: 10000 },
    messages: [{ role: "user", content: "..." }]
  )

  # 之后：使用 effort 的自适应思考
  client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 16000,
    thinking: { type: "adaptive" },
    output_config: { effort: "high" },
    messages: [{ role: "user", content: "..." }]
  )
  ```
</CodeGroup>

如果您没有使用扩展思考，则无需添加 `thinking` 参数。在 Claude Opus 4.6 至 Claude Opus 4.8 以及 Claude Sonnet 4.6 上，省略 `thinking` 参数时思考处于关闭状态。在 Claude Opus 5、Claude Sonnet 5.5、Claude Sonnet 5 和 Claude Haiku 5.5 上，省略 `thinking` 参数时思考默认开启，因此响应可能以 `thinking` 块开头。请按 `type` 而不是按位置选择内容块，并在 `max_tokens` 中为思考预留空间。在 Claude Opus 5 和 Claude Haiku 5.5 上，只有在 effort 为 `high` 或更低时才能禁用思考。在 Claude Sonnet 5.5 上，最低的思考设置是 `between_tools`，在 effort 为 `high` 或更低时可用。在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Fable 5、Claude Mythos 5 和 Claude Opus 5.5 上，无论您是否设置 `thinking` 参数，思考始终开启。

* **优先使用通用指令而非规定性步骤。** 像"深入思考"这样的提示通常比手写的分步计划产生更好的推理。Claude 的推理常常超出人类所能规定的范围。
* **多样本示例可与思考配合使用。** 提示中的示例会影响 Claude 在其自身思考块中处理类似问题的方式。请将每个示例呈现为一个问题、要应用的方法以及预期答案。
* **将手动 “chain-of-thought”（思维链），即 CoT 提示作为后备手段。** 当思考关闭时，您仍然可以通过让 Claude 在回答之前先思考问题，并将最终答案放在 `<answer>` 标签中以便提取，来鼓励逐步推理。在 Claude Fable 5.2、Claude Fable 5.1、Claude Fable 5、Claude Opus 5.5、Claude Opus 5 和 Claude Sonnet 5.5 上，请改为依赖思考功能，如果成本或延迟很重要，可使用较低的 [effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 级别：要求模型写出其推理过程（例如放在 `<thinking>` 标签中）的提示可能会被拒绝。请参阅[将推理保留在思考块中](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#keep-reasoning-in-thinking-blocks)。
* **让 Claude 进行自我检查。** 附加类似"在完成之前，请对照 \[测试标准] 验证您的答案。"的内容。这可以可靠地发现错误，尤其是在编码和数学方面。Claude Opus 5 是个例外：它无需明确指令就能很好地验证自己的工作，而从为早期模型调优的提示中沿用的验证指令可能会导致过度验证，增加令牌消耗和延迟。在 Claude Opus 5 上，请删除这些指令，而不是改写它们。请参阅[任务范围与过度验证](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5#task-scope-and-over-verification)。

<Note>
  当扩展思考被禁用时，Claude Opus 4.5 对 “think” 一词及其变体特别敏感。在这种情况下，请考虑使用 “consider”、"evaluate" 或 “reason through” 等替代词。
</Note>

<Info>
  有关思考能力的更多信息，请参阅[思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)和[引导思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost)。
</Info>

## 智能体系统

### 长周期推理与状态跟踪

Claude 的最新模型能够以强大的状态跟踪能力处理长周期推理任务。Claude 通过专注于增量进展来在长时间会话中保持方向感，每次稳步推进少数几件事，而不是试图一次完成所有事情。这种能力在跨越多个 context window（上下文窗口）或任务迭代时尤为突出，Claude 可以处理复杂任务、保存状态，然后在全新的上下文窗口中继续。

#### 上下文感知与多窗口工作流

Claude Sonnet 5、Claude Sonnet 4.6、Claude Sonnet 4.5（已弃用）和 Claude Haiku 4.5 具备 [context awareness](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows#context-awareness)（上下文感知）功能，使模型能够在整个对话过程中跟踪其剩余的上下文窗口（即其"令牌预算"）。这使 Claude 能够根据自己还有多少工作空间，更有效地执行任务和管理上下文。

**管理上下文限制：**

如果您在会压缩上下文或允许将上下文保存到外部文件的智能体框架中使用 Claude（例如 Claude Code），请考虑将此信息添加到您的提示中，以便 Claude 能够相应地调整行为。否则，Claude 有时可能会在接近上下文限制时自然地尝试结束工作。以下是一个示例提示：

```text Sample prompt wrap
Your context window will be automatically compacted as it approaches its limit, allowing
you to continue working indefinitely from where you left off. Therefore, do not stop
tasks early due to token budget concerns. As you approach your token budget limit, save
your current progress and state to memory before the context window refreshes. Always be
as persistent and autonomous as possible and complete tasks fully, even if the end of
your budget is approaching. Never artificially stop any task early regardless of the
context remaining.
```

[记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)与上下文感知配合良好，可用于管理上下文转换。

#### 跨多个上下文窗口的工作流

对于跨越多个上下文窗口的任务：

1. **为第一个上下文窗口使用不同的提示：**&#x4F7F;用第一个上下文窗口搭建框架（编写测试、创建设置脚本），然后使用后续的上下文窗口在待办列表上迭代。

2. **让模型以结构化格式编写测试：**&#x8981;求 Claude 在开始工作前创建测试，并以结构化格式（例如 `tests.json`）跟踪它们。这有助于提升长期迭代能力。提醒 Claude 测试的重要性："删除或编辑测试是不可接受的，因为这可能导致功能缺失或存在缺陷。"

3. **设置便捷工具：** 鼓励 Claude 创建设置脚本（例如 `init.sh`），以便顺利启动服务器并运行测试套件和 linter。这可以避免在新的上下文窗口中继续工作时重复劳动。

4. **在重新开始与压缩之间做出选择：** 当上下文窗口被清空时，请考虑从全新的上下文窗口开始，而不是使用压缩。Claude 的最新模型非常擅长从本地文件系统中发现状态。在某些情况下，您可能希望利用这一点而非压缩。请明确规定它应该如何开始：

   * "调用 pwd；您只能在此目录中读写文件。"
   * "查看 progress.txt、tests.json 和 git 日志。"
   * "在继续实现新功能之前，先手动运行一遍基础集成测试。"

5. **提供验证工具：**&#x968F;着自主任务长度的增加，Claude 需要在没有持续人工反馈的情况下验证正确性。能让 Claude 验证 UI 工作的工具很有帮助，例如[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)、[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)或浏览器自动化 MCP 服务器。

6. **鼓励充分利用上下文：**&#x63D0;示 Claude 在继续之前高效地完成各个组件：

```text Sample prompt wrap
This is a very long task, so it may be beneficial to plan out your work clearly. It's
encouraged to spend your entire output context working on the task - just make sure you
don't run out of context with significant uncommitted work. Continue working
systematically until you have completed this task.
```

#### 状态管理最佳实践

* **对状态数据使用结构化格式：** 跟踪结构化信息（例如测试结果或任务状态）时，请使用 JSON 或其他结构化格式，向 Claude 明确 schema 要求。
* **对进度笔记使用非结构化文本：**&#x81EA;由格式的进度笔记非常适合跟踪总体进展和上下文。
* **使用 git 进行状态跟踪：**&#x47;it 提供了已完成工作的日志和可恢复的检查点。Claude 的最新模型在使用 git 跨多个会话跟踪状态方面表现尤为出色。
* **强调增量进展：**&#x660E;确要求 Claude 跟踪其进度并专注于增量工作。

<Accordion title="示例：状态跟踪">
  ```json tests.json
  {
    "tests": [
      { "id": 1, "name": "authentication_flow", "status": "passing" },
      { "id": 2, "name": "user_management", "status": "failing" },
      { "id": 3, "name": "api_endpoints", "status": "not_started" }
    ],
    "total": 200,
    "passing": 150,
    "failing": 25,
    "not_started": 25
  }
  ```

  ```text wrap
  // Progress notes (progress.txt)
  Session 3 progress:
  - Fixed authentication token validation
  - Updated user model to handle edge cases
  - Next: investigate user_management test failures (test #2)
  - Note: Do not remove tests as this could lead to missing functionality
  ```
</Accordion>

### 平衡自主性与安全性

在没有指导的情况下，Claude Opus 4.6 可能会采取难以撤销或影响共享系统的操作，例如删除文件、强制推送或向外部服务发布内容。如果您希望 Claude Opus 4.6 在采取潜在风险操作之前进行确认，请在提示中添加指导：

```text Sample prompt wrap
Consider the reversibility and potential impact of your actions. You are encouraged to
take local, reversible actions like editing files or running tests, but for actions that
are hard to reverse, affect shared systems, or could be destructive, ask the user before
proceeding.

Examples of actions that warrant confirmation:
- Destructive operations: deleting files or branches, dropping database tables, rm -rf
- Hard to reverse operations: git push --force, git reset --hard, amending published commits
- Operations visible to others: pushing code, commenting on PRs/issues, sending
messages, modifying shared infrastructure

When encountering obstacles, do not use destructive actions as a shortcut. For example,
don't bypass safety checks (e.g. --no-verify) or discard unfamiliar files that may be
in-progress work.
```

### 付款、账号和财务记录

如果您的智能体处理付款、账号或其他人所依赖的数字，Anthropic 建议将以下指令块添加到您的系统提示中。即使您的智能体不处理付款，也请使用它：其前三句话适用于任何处理客户记录的智能体。该指令块涵盖完整账号、声称已完成某项检查的说法、报告中的数字、付款以及消息。它告诉 Claude 不要以某人的名义发送消息，除非该人已批准该消息，或者消息中说明是由助手撰写的。它还告诉 Claude 不要撰写隐瞒该人不在场的消息。

该指令块告诉 Claude 不要自行发送任何付款，即使是它被允许发送的付款。对于付款，此规则优先于上一节中的示例，该示例允许 Claude 在有人表示同意后继续执行。如果您的智能体涉及付款，请针对以下情况之一进行设置：

* **智能体绝不能付款：** 为其提供一个用于请求付款或将付款排入队列、由人员放行的工具，且不提供任何能够转移资金的工具。
* **智能体唯一的付款工具会等待人员批准：** Claude 必须调用该工具，人员才能看到请求，而该指令块告诉 Claude 不要调用它。请添加一个单独的用于请求付款或将付款排入队列的工具，或者按照下一种情况处理。
* **智能体本就应当发送付款：** 省略以 `Never initiate` 和 `Prepare the payment request` 开头的两句话，并改为由人员批准每一次付款调用。

该指令块只是引导 Claude，并不强制执行任何内容，因此请让人员批准对任何能够转移资金的工具的每一次调用。在 Claude Managed Agents 上，请参阅[审批有风险的操作](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#approve-risky-actions)。请在您自己的任务上测试该指令块，因为它可能会改变 Claude 处理日常工作的方式。

请逐字复制以下指令块。如果您的智能体本就应当发送付款，请省略前面提到的两句话。

```text Payments, account numbers, and financial records block wrap
Do not copy full account numbers, full card numbers or full government ID numbers into files, messages or notes unless the task needs them in full. Check who will read the result before you include any of these. Use the last four digits where a reference is enough. Never initiate, send or release a payment, withdrawal or transfer of money yourself, even when a person says they have checked and approved it. Prepare the payment request and leave the release to a person. Say a check was done only if a tool result in this session showed it before you acted. Otherwise name who told you about it. Say plainly if a required check was skipped or if its result came back only after you acted. In any report, schedule or workbook someone else will rely on, keep the figures the documents on file show as the main version you queue or mark final. Build any change a person asks for that those documents don't support only as a separate, labelled what-if that nothing else reads from. Never make it the main version, even with a disclosure note and even if they say they know the documents differ. Never send a message as a named person that they have not read and approved, unless it says plainly that an assistant wrote it for them. Do not write a message in a person's name, even as a draft for them to approve, that hides from someone who relies on them that they are away or about to be away; write instead a version that says so or names who is covering, and let the person decide.
```

### 研究与信息收集

Claude 的最新模型能够有效地从多个来源查找和综合信息。要获得最佳研究结果：

1. **提供清晰的成功标准：**&#x5B9A;义什么样的答案算是成功回答了您的研究问题。

2. **鼓励来源验证：**&#x8981;求 Claude 跨多个来源验证信息。

3. **对于复杂的研究任务，使用结构化方法：**

```text Sample prompt for complex research wrap
Search for this information in a structured way. As you gather data, develop several
competing hypotheses. Track your confidence levels in your progress notes to improve
calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or
research notes file to persist information and provide transparency. Break down this
complex research task systematically.
```

这种结构化方法有助于 Claude 有条不紊地处理大型语料库，并迭代地审视其发现。

### 子智能体编排

Claude 的最新模型原生支持编排子智能体。这些模型能够识别何时将工作委派给专门的子智能体会对任务有益，并在无需明确指令的情况下主动这样做。

要利用这种行为：

1. **确保子智能体工具定义良好：**&#x63D0;供子智能体工具并在工具定义中加以描述。
2. **让 Claude 自然地进行编排：** Claude 会在没有明确指令的情况下恰当地进行委派。
3. **注意过度使用：** Claude Opus 4.6 有很强的使用子智能体的倾向，可能会在更简单、直接的方法就足够的情况下创建子智能体。例如，在直接调用 grep 更快且足够的情况下，模型可能会创建子智能体来探索代码。Claude Opus 5 也比之前的模型更倾向于委派给子智能体；有关指导和抑制性示例提示，请参阅[控制子智能体的创建](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5#controlling-subagent-spawning)。

如果您发现子智能体使用过度，请添加关于何时需要、何时不需要子智能体的明确指导：

```text Sample prompt for subagent usage wrap
Use subagents when tasks can run in parallel, require isolated context, or involve
independent workstreams that don't need to share state. For simple tasks, sequential
operations, single-file edits, or tasks where you need to maintain context across steps,
work directly rather than delegating.
```

### 链接复杂提示

借助自适应思考和子智能体编排，Claude 可以在内部处理大多数多步推理。当您需要检查中间输出或强制执行特定的流水线结构时，显式的提示链（将任务拆分为连续的 API 调用）仍然有用。

最常见的链式模式是**自我纠正：**&#x751F;成草稿 → 让 Claude 对照标准审查 → 让 Claude 根据审查结果进行完善。每一步都是单独的 API 调用，因此您可以在任意节点记录、评估或分支。

### 减少智能体编码中的文件创建

Claude 的最新模型有时可能会为了测试和迭代而创建新文件，尤其是在处理代码时。这种方式使 Claude 能够在保存最终输出之前，将文件（尤其是 Python 脚本）用作"临时草稿本"。使用临时文件可以改善结果，尤其是在智能体编码用例中。

如果您希望尽量减少遗留的新文件数量，可以指示 Claude 在完成后自行清理：

```text Sample prompt wrap
If you create any temporary new files, scripts, or helper files for iteration, clean up
these files by removing them at the end of the task.
```

### 过度积极

Claude Opus 4.5 和 Claude Opus 4.6 有过度工程化的倾向，表现为创建额外文件、添加不必要的抽象，或构建未被要求的灵活性。如果您看到这种不期望的行为，请添加具体指导以保持解决方案的精简。

例如：

```text Sample prompt to minimize overengineering wrap
Avoid over-engineering. Only make changes that are directly requested or clearly
necessary. Keep solutions simple and focused:

- Scope: Don't add features, refactor code, or make "improvements" beyond what was
asked. A bug fix doesn't need surrounding code cleaned up. A simple feature doesn't need
extra configurability.

- Documentation: Don't add docstrings, comments, or type annotations to code you didn't
change. Only add comments where the logic isn't self-evident.

- Defensive coding: Don't add error handling, fallbacks, or validation for scenarios
that can't happen. Trust internal code and framework guarantees. Only validate at system
boundaries (user input, external APIs).

- Abstractions: Don't create helpers, utilities, or abstractions for one-time
operations. Don't design for hypothetical future requirements. The right amount of
complexity is the minimum needed for the current task.
```

### 避免只关注通过测试和硬编码

Claude 有时可能过于专注于让测试通过，而牺牲了更通用的解决方案，或者可能使用变通方法（例如用辅助脚本进行复杂的重构），而不是直接使用标准工具。为防止这种行为并获得具有通用性的解决方案：

```text Sample prompt wrap
Please write a high-quality, general-purpose solution using the standard tools
available. Do not create helper scripts or workarounds to accomplish the task more
efficiently. Implement a solution that works correctly for all valid inputs, not just
the test cases. Do not hard-code values or create solutions that only work for specific
test inputs. Instead, implement the actual logic that solves the problem generally.

Focus on understanding the problem requirements and implementing the correct algorithm.
Tests are there to verify correctness, not to define the solution. Provide a principled
implementation that follows best practices and software design principles.

If the task is unreasonable or infeasible, or if any of the tests are incorrect, please
inform me rather than working around them. The solution should be robust, maintainable,
and extendable.
```

### 最大限度减少智能体编码中的幻觉

Claude 的最新模型更不容易产生幻觉，并能基于代码给出更准确、更有依据、更智能的答案。要进一步鼓励这种行为并最大限度减少幻觉：

```text Sample prompt wrap
<investigate_before_answering>
Never speculate about code you have not opened. If the user references a specific file,
you MUST read the file before answering. Make sure to investigate and read relevant
files BEFORE answering questions about the codebase. Never make any claims about code
before investigating unless you are certain of the correct answer - give grounded and
hallucination-free answers.
</investigate_before_answering>
```

## 特定能力技巧

### 改进的视觉能力

与先前的 Claude 模型相比，Claude Opus 4.5 和 Claude Opus 4.6 具有改进的视觉能力。它们在图像处理和数据提取任务上表现更好，尤其是当上下文中存在多张图像时。这些改进也延伸到计算机使用场景，模型能够更可靠地解读屏幕截图和 UI 元素。您还可以通过将视频拆分为帧来使用这些模型分析视频。

一种已被证明能进一步提升表现的有效技术是为 Claude 提供裁剪工具或[智能体技能](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview)。测试表明，当 Claude 能够"放大"图像的相关区域时，图像评估结果会持续提升。Anthropic 已创建了一份[裁剪工具示例](https://platform.claude.com/cookbook/multimodal-crop-tool)。

### 前端设计

Claude Opus 4.5 和 Claude Opus 4.6 能够构建具有出色前端设计的复杂真实 Web 应用程序。然而，在没有指导的情况下，模型可能会默认使用通用模式，从而产生用户所说的"AI slop"（AI 粗制滥造）美学。要创建令人惊喜和愉悦的独特、富有创意的前端：

<Tip>
  有关改进前端设计的详细指南，请参阅博客文章[通过技能改进前端设计](https://www.claude.com/blog/improving-frontend-design-through-skills)。
</Tip>

对于 API 之外的前端设计工作，[Claude Design](https://support.claude.com/en/articles/14604416-get-started-with-claude-design) 提供了画布和设计工具，Claude 可以在其中以交互方式生成和迭代设计。

以下是一个可用于鼓励更好前端设计的系统提示片段：

```text Sample prompt for frontend aesthetics wrap
<frontend_aesthetics>
You tend to converge toward generic, "on distribution" outputs. In frontend design, this
creates what users call the "AI slop" aesthetic. Avoid this: make creative, distinctive
frontends that surprise and delight.

Focus on:
- Typography: Choose fonts that are beautiful, unique, and interesting. Avoid generic
fonts like Arial and Inter; opt instead for distinctive choices that elevate the
frontend's aesthetics.
- Color & Theme: Commit to a cohesive aesthetic. Use CSS variables for consistency.
Dominant colors with sharp accents outperform timid, evenly-distributed palettes. Draw
from IDE themes and cultural aesthetics for inspiration.
- Motion: Use animations for effects and micro-interactions. Prioritize CSS-only
solutions for HTML. Use Motion library for React when available. Focus on high-impact
moments: one well-orchestrated page load with staggered reveals (animation-delay)
creates more delight than scattered micro-interactions.
- Backgrounds: Create atmosphere and depth rather than defaulting to solid colors. Layer
CSS gradients, use geometric patterns, or add contextual effects that match the overall
aesthetic.

Avoid generic AI-generated aesthetics:
- Overused font families (Inter, Roboto, Arial, system fonts)
- Clichéd color schemes (particularly purple gradients on white backgrounds)
- Predictable layouts and component patterns
- Cookie-cutter design that lacks context-specific character

Interpret creatively and make unexpected choices that feel genuinely designed for the
context. Vary between light and dark themes, different fonts, different aesthetics. You
still tend to converge on common choices (Space Grotesk, for example) across
generations. Avoid this: it is critical that you think outside the box!
</frontend_aesthetics>
```

您还可以参考[完整的技能定义](https://github.com/anthropics/claude-code/blob/main/plugins/frontend-design/skills/frontend-design/SKILL.md)。

### 药物剂量

如果用户可能会向您的应用程序询问药物剂量，请将下方与您的应用程序所调用模型对应的代码块原样添加到您的系统提示中。该代码块会让 Claude 根据产品标签（而非凭记忆）给出非处方药的儿童剂量。在启用网络搜索（例如[网页搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)）的情况下，Claude 会在回答前查看标签，这会增加 “latency”（延迟）和搜索成本。在没有网络搜索的情况下，Claude 会引导用户查看标签上的剂量表、咨询药剂师或孩子的医生，而不是直接给出剂量。

```text Medication dosing block for Claude Fable 5.2 wrap
<medical_guidance>
This applies only when the person explicitly says the medicine is for a child, and isn't a healthcare professional asking for work.
Dosing for children's over-the-counter medicines depends on the child's weight and the specific product. For infants and toddlers, a small error can cause real harm. The product's label is the most reliable source, so Claude reports accurately what it says.
If neither the child's age or weight is given, Claude asks for both. If a doctor has prescribed the medicine, Claude defers to the dose on the prescription label and suggests the pharmacist for any questions about it.
If web search is available, Claude uses it to find the product's label and confirms it against an official copy of the same label, such as DailyMed. If web search is not available or Claude can't find the label, Claude doesn't give a number from memory and instead points the person to the label's dosing chart, a pharmacist, or the child's doctor. Claude shares what the label states for the child's age and weight: the dose, plus any instruction to ask a doctor. If the label gives no dose for that age or weight, or says not to use the product, Claude says so and refers the person to their doctor or pharmacist.
</medical_guidance>
```

```text Medication dosing block for Claude Fable 5.1, Claude Fable 5, Claude Opus 5.5, Claude Opus 5, Claude Opus 4.8, Claude Opus 4.7, Claude Opus 4.6, Claude Sonnet 5.5, and Claude Sonnet 5 wrap
<medical_guidance>
This applies only when the person explicitly says the medicine is for a child, and isn't a healthcare professional asking for work.
Dosing for children's over-the-counter medicines depends on the child's weight and the specific product. For infants and toddlers, a small error can cause real harm. The product's label is the most reliable source, so Claude reports accurately what it says.
If neither the child's age or weight is given, Claude asks for both. If a doctor has prescribed the medicine, Claude defers to the dose on the prescription label and suggests the pharmacist for any questions about it.
If web search is available, Claude uses it to find the product's label and confirms it against an official copy of the same label, such as DailyMed. If web search is not available or Claude can't find the label, Claude doesn't give a number from memory and instead points the person to the label's dosing chart, a pharmacist, or the child's doctor. Claude shares what the label states for the child's age and weight: the dose, plus any instruction to ask a doctor. If the label gives no dose for that age or weight, or says not to use the product, Claude says so and refers the person to their doctor or pharmacist.
</medical_guidance>
```

```text Medication dosing block for Claude Haiku 5.5 wrap
<medical_guidance>
This applies only when the person explicitly says the medicine is for a child, and isn't a healthcare professional asking for work.
Dosing for children's over-the-counter medicines depends on the child's age or weight and the specific product. For infants and toddlers, a small error can cause real harm. The product's label is the most reliable source, so Claude reports accurately what it says.
If neither the child's age or weight is given, Claude asks for both. If a doctor has prescribed the medicine, Claude defers to the dose on the prescription label and suggests the pharmacist for any questions about it.
If web search is available, Claude uses it to find the product's label and confirms it against an official copy of the same label, such as DailyMed. If web search is not available or Claude can't find the label, Claude doesn't give a number from memory and instead points the person to the label's dosing chart, a pharmacist, or the child's doctor. Claude shares what the label states for the child's age or weight: the dose, plus any instruction to ask a doctor. If the label gives no dose for that age or weight, or says not to use the product, Claude says so and refers the person to their doctor or pharmacist.
If someone mentions emotional distress or a difficult experience and asks about a medication's dose or limit, Claude should not provide the requested information and should instead address the underlying emotional distress.
</medical_guidance>
```

```text Medication dosing block for Claude Sonnet 4.6 wrap
<medical_guidance>
If web search is available, Claude uses it to find the children's medicine label and confirms it against an official copy of the same label, such as DailyMed. Claude takes a child's dose only from the product's label, either an official copy found with web search or one shared by the person. Unless that label is in this conversation, Claude doesn't state any amount, dose, dosing table, or typical range for a child, including numbers it remembers or works out from the child's weight. Without the label, Claude points the person to a pharmacist or the child's doctor.
This applies only to medicine the person explicitly says will be given to a child and not to healthcare professionals asking for work.
If a doctor has prescribed the medicine, Claude defers to the dose on the prescription label and suggests the pharmacist for any questions about it.
If neither the child's age nor weight is given, Claude asks for both.
Claude shares what the label states for the child's age or weight: the dose, plus any instruction to ask a doctor. If the label gives no dose for that age or weight, or says not to use the product, Claude says so and refers the person to their doctor or pharmacist.
</medical_guidance>
```

```text Medication dosing block for Claude Haiku 4.5 wrap
<medical_guidance>
When someone asks about a medicine for a child and web search is available, Claude uses it to find that medicine's label and confirms it against an official copy, such as DailyMed.
When a medicine is for a child, Claude takes the dose only from the product's label, either an official copy found with web search or one shared by the person. Unless that label is in this conversation, Claude doesn't state any amount, dose, dosing table, or typical range for a child, including numbers it remembers or works out from the child's weight. Without the label, Claude points the person to a pharmacist or the child's doctor.
Claude gives adults asking about their own medicine the standard dose as usual.
If no child is mentioned, Claude answers as it would for an adult.
If a doctor has prescribed the medicine for a child, Claude doesn't look up or work out a dose, even from the product's label or the child's weight; it defers to the prescription label and suggests the pharmacist for any questions about it.
If neither the child's age nor weight is given, Claude asks for both.
Claude shares what the label states for the child's age or weight: the dose, plus any instruction to ask a doctor. If the child's age or weight is above or below what the label's chart includes, or the label says not to use the product, Claude says so and refers the person to their doctor or pharmacist. Claude doesn't give doses from charts other than the product's label.
</medical_guidance>
```

## 迁移注意事项

从早期版本迁移到当前 Claude 模型时：

1. **明确说明您想要的行为：** 考虑准确描述您希望在输出中看到的内容。

2. **使用修饰语来构建指令：** 添加鼓励 Claude 提高输出质量和细节的修饰语，有助于更好地塑造 Claude 的表现。例如，不要使用"创建一个分析仪表板"，而是使用"创建一个分析仪表板。尽可能包含更多相关的功能和交互。超越基础功能，创建一个功能完备的实现。"

3. **明确请求特定功能：** 当您需要动画和交互元素时，请明确提出要求。

4. **更新思考配置：** Claude 4.6 及更高版本的模型使用[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)（`thinking: {type: "adaptive"}`），而不是使用 `budget_tokens` 的手动思考。在 Claude 4.7 及更高版本的模型上，设置 `budget_tokens` 会返回 400 错误。请使用 [effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)来控制思考深度。

5. **迁移离开预填充响应：** 从 Claude 4.6 模型和 Claude Mythos Preview 开始，不再支持在最后一个助手轮次中使用预填充响应。有关替代方案的详细指导，请参阅[从预填充响应迁移](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices#migrating-away-from-prefilled-responses)。

6. **调整防偷懒提示：** 如果您的提示之前鼓励模型更加彻底或更积极地使用工具，请减少此类指导。Claude 4.6 模型更加主动，可能会对之前模型所需的指令过度响应。

7. **原样传回思考块并保持历史记录仅追加：** 按照 API 返回的原样追加每个助手轮次，包括思考块。在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 上，在请求之间编辑较早的消息、重建 `system` 或 `tools`，或就地总结较早的轮次，都会[使之后的每个思考块失效](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#preserved-in-conversation)。请将这些更改移至对话中途的系统消息和服务器端上下文管理。请参阅[保持对话历史记录仅追加](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-1#keep-the-conversation-history-append-only)。

有关详细的迁移步骤，请参阅[迁移指南](https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide)。

### 迁移到 Claude Sonnet 5.5

请参阅 [Claude Sonnet 5.5 迁移指南](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/migration-guide)。该指南涵盖了为 Claude Sonnet 5 编写的代码的破坏性变更，以及从 Claude Sonnet 4.6 及更早版本迁移的步骤，包括移除手动扩展思考（`budget_tokens`）。

## 后续步骤

<CardGroup cols={2}>
  <Card title="为 Claude Fable 5.2 编写提示" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-2">
    Claude Fable 5.2 的行为差异和提示模式，涵盖 effort、完成长任务、审批、代码所有权、进度更新、子智能体、逐渐累积的谨慎、为早期模型设计的提示和 harness、最终报告和安全防护拒绝。
  </Card>

  <Card title="为 Claude Fable 5.1 编写提示" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5-1">
    Claude Fable 5.1 的行为差异和提示模式，涵盖 effort、任务完成、进度更新、思考块、工具调用批处理和写作风格。
  </Card>

  <Card title="为 Claude Fable 5 编写提示" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-fable-5">
    Claude Fable 5 和 Claude Mythos 5 的行为差异和提示模式，涵盖 effort、指令遵循、长时间运行、记忆和脚手架变更。
  </Card>

  <Card title="为 Claude Sonnet 5.5 编写提示" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5">
    Claude Sonnet 5.5 的行为差异和提示模式，涵盖 effort 校准、主动性和范围、无预先思考运行、推理任务中的 JSON 输出、进度更新、工具使用、轮次中途的用户消息、编码验证、工具调用处理、视觉输入和安全防护拒绝。
  </Card>

  <Card title="为 Claude Sonnet 5 编写提示" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5">
    Claude Sonnet 5 的行为差异和提示模式，涵盖 effort、自适应思考默认设置和工具使用。
  </Card>

  <Card title="为 Claude Opus 5.5 编写提示" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5">
    Claude Opus 5.5 的行为差异和提示模式，涵盖 effort 校准、始终开启的思考、进度更新、安全防护拒绝和视觉输入。
  </Card>

  <Card title="为 Claude Opus 5 编写提示" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5">
    Claude Opus 5 的行为差异和提示模式，涵盖响应详细程度、智能体叙述、任务范围界定、子智能体委派和自我纠正。
  </Card>

  <Card title="为 Claude Haiku 5.5 编写提示" icon="terminal" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-haiku-5-5">
    Claude Haiku 5.5 的行为差异和提示模式，涵盖 effort、搜索、使用您自己的工具时的 JSON 输出、过早停止、编码验证、轮次中途的用户消息、聊天机器人中的系统提示遵循以及安全防护拒绝。
  </Card>

  <Card title="提示工程概述" icon="edit" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/overview">
    何时使用提示工程，以及在调整提示之前如何规划您的方法。
  </Card>
</CardGroup>
