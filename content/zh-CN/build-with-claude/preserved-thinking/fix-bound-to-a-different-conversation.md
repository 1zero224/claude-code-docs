---
title: 修复绑定到其他对话的思考块
url: https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation
description: 在出现提示思考块绑定到其他对话的 400 错误后让对话继续进行，找出您的代码更改了什么，并测试该更改已被消除。
featureMetadata:
  status: stable
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-fable-5-1
    - claude-mythos-5-1
    - claude-opus-5-5
    - claude-sonnet-5-5
    - claude-haiku-5-5
  supportedPlatforms:
    Claude API: stable
    Amazon Bedrock: stable
    Amazon Bedrock (bedrock-runtime): stable
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Anthropic managed): stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

再次发送相同的请求无法成功，因为您的历史记录中仍包含您的代码所做的更改。发送 `"drop_block"` 以[立即停止错误](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#prefix-mismatch-error)。长期修复是让您的代码成为 “append-only”（[仅追加](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#prefix-check)）。

## 错误：The block is bound to a different conversation

该思考块未通过 “prefix check”（[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#prefix-check)）：在 Claude 生成它之后，它之前的某些内容发生了更改（请参阅[什么算作更改](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#what-counts-as-an-edit)），或者位于其他思考块之间的某个思考块被省略或移动了位置。该错误如下所示，并且可能会进一步[指出更改了什么](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#prefix-mismatch-error-details)：

```text wrap
messages.1.content.0: Invalid `signature` in `thinking` block. The block is bound to a different conversation. Remove the block, or set `thinking.block_binding.prefix_mismatch_behavior` to "drop_block".
```

“A different conversation” 指的是该块之前的内容不同：[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#prefix-check)不会比较会话或用户。错误消息说要移除该块，但[只移除该块是不够的](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#remove-by-hand)。[其他错误](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#similar-errors)可能看起来与此类似。

如果该应用不是您的，请开始一个新对话并告知其维护者。如果您[移除了机密、个人数据或注入的文本](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#remove-sensitive-content)，或者您的请求[关闭了思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#turning-thinking-off)，请[手动移除思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#remove-by-hand)，而不是执行步骤 1。

<Steps>
  <Step title="立即停止错误">
    将 `thinking.block_binding.prefix_mismatch_behavior` 设置为 `"drop_block"`，并发送 “beta header”（[beta 标头](https://platform.claude.com/docs/zh-CN/api/beta-headers#beta-features-on-other-platforms)）`anthropic-beta: thinking-binding-controls-2026-08-01`。请在该对话之后的每个请求中都这样做，因为 API 不会在请求之间存储该设置。即使您修复了代码，已保存的对话仍包含该更改，因此请将该设置与对话一起存储。将 `type` 设置为 `"adaptive"`，如下所示：

    ```json
    {
      "thinking": {
        "type": "adaptive",
        "block_binding": {
          "prefix_mismatch_behavior": "drop_block"
        }
      }
    }
    ```

    在 SDK 中，调用 beta Messages 方法并传入该 beta，如[测试程序](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#how-to-tell-whether-your-integration-is-impacted-details)中所示。如果错误[仍要求提供标头](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#setting-not-received)，则说明标头没有送达。

    这会通过丢弃无效的思考来停止错误，但它[可能花费更多](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#drop-block-cost)，并可能影响输出质量和 “latency”（延迟）。下一个请求会成功，其 `input_transformations` 数组会列出 API 丢弃的每个块。

    如果您的请求没有设置 `thinking` 参数，添加它不会改变其他任何内容，因为[具有保留思考的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)默认进行自适应思考。
  </Step>

  <Step title="长期修复">
    让您的集成成为仅追加：

    1. 找出更改。错误可能会指出它。要找出您的代码在何处进行该更改，请运行 `claude update`，然后在您代码的仓库中于 Claude Code 里运行 `/claude-api preserved-thinking-migration`。或者[手动查找](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#find-the-change-details)。
    2. 停止进行该更改，否则新对话也可能收到该错误。[改为在末尾发送更新](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions)。
    3. 对于每个正在进行的对话，继续发送[其最后一次成功请求](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#saved-sessions)的 `system` 和 `tools`。
  </Step>
</Steps>

如果该更改是最近发生的，您也可以撤销它：恢复您的代码所更改的内容。更改之前的思考会再次有效。此后生成的思考会未通过前缀检查，因此请移除它，或继续发送 `"drop_block"`。

## 丢弃思考的代价

Claude 可能会花费更多输出令牌来重新推导它已无法读取的推理。丢弃这些块还会使 “prompt cache”（提示缓存）从第一个被丢弃的块起失效。

当某些内容在每一轮都发生更改时，每一轮都会失去之前各轮的思考，因此成本增加会反复出现。在依赖 `"drop_block"` 之前，请在您自己的工作负载上运行评估（evals）。

<Frame>
  ![更改之后的四个请求：第一个被拒绝；后三个丢弃旧的思考并保留每个新块](https://platform.claude.com/docs/zh-CN/images/preserved-thinking-drop-block-loss.svg)
</Frame>

使用 `"drop_block"` 可以让现有对话在包含您正在修复的更改时继续进行，也可用于计划内的丢弃，例如[在您自己的摘要之后](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-thinking-blocks#own-summary)。不要一直开启它来掩盖您的代码持续进行的更改。一旦无效块已不在历史记录中，您就可以[停止发送它](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#drop-block-cost-details)。

## 找出您的代码更改了什么

更改位于 API 指出的第一个块之前。400 错误可能会说明更改了什么，例如 `system`、`tools`、缺失的思考块，或第一条不同的消息。如果没有错误，或者错误没有说明，请[从您已有的信息入手](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#find-the-change-details)，例如 `prefix_binding_mismatch` 条目或增多的输出令牌。

## 测试您的代码是否存在未通过前缀检查的情况

看到测试失败过之后，您就可以信任它。首先，通过您自己的代码运行一个让 Claude 进行思考的会话，并有意更改某些内容。在此步骤中发送 `"drop_block"`，这样响应会列出每个无效块，而不是在第一个处停止，如[五个步骤和一个程序](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#how-to-tell-whether-your-integration-is-impacted-details)所示。

然后撤掉该更改，并运行您的真实会话。在每个请求中，发送 beta 标头 `thinking-binding-controls-2026-08-01`，并将 [`prefix_mismatch_behavior`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#preserved-thinking-controls) 设置为 `"error"`，这样您的代码在这些会话中进行的更改会以 400 错误使构建失败。

请测试进行压缩、后期添加工具、更改指令、[切换模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)或恢复的会话：普通的轮次不会触及这些代码。

一次干净的运行没有 `prefix_binding_mismatch` 条目，也没有错误。它仅表明在您运行的会话中，它所检查的块是有效的。在以下情况下，它什么也证明不了：

* 会话中没有思考块
* 您的代码移除了所有思考，或者该模型不具有保留思考
* 您没有发送 beta 标头，而[模型可能会放行未通过前缀检查的块](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)且不报错

## 监控生产环境，并找出无错误丢失的推理

记录每个 400 错误，并[在代码中处理它](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#handle-the-error-in-code)，让对话继续进行。如果您的请求改为不包含 `prefix_mismatch_behavior`，请[监控生产流量](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#watch-production-traffic)，查找已送达 Claude 的未通过前缀检查的思考。

推理可能会在没有错误的情况下丢失，途径包括 `"drop_block"`、批处理项、模型切换、转移到[另一个账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)，或不回传任何思考的代码。使用 beta 标头时，除最后一种情况外，每种情况都会留下一个条目。对于最后一种情况，请检查您的请求是否包含 `thinking` 块。

## 技术细节

### 错误消息还可能说明什么

在 API 能够判断的情况下，错误消息会进一步指出更改了什么，例如 `system`、`tools`、第一条不同的消息或块、缺失或新增的内容，或者缺失或顺序错乱的较早思考块。该句子的措辞可能会变化，因此不要对其进行匹配。如果请求没有 beta 标头，错误消息会附加以下句子：

```text wrap
That setting requires the `thinking-binding-controls-2026-08-01` value in the `anthropic-beta` header.
```

### 错误仍要求提供标头，或显示 `Extra inputs are not permitted`

如果错误仍要求提供标头，则说明标头没有到达 API：代理或库可能已将其移除。另一种以 `block_binding: Extra inputs are not permitted` 结尾的错误含义相同，或者表示思考已关闭。

### 手动移除无效的思考

在您无法发送该设置时，或在以下情况之一中使用此方法：

* 在您移除了机密、个人数据或注入的文本之后，因为 `"drop_block"` 会将思考留在您存储的历史记录中。
* 当思考关闭时（`"disabled"` 或 `"between_tools"`），因为 `"drop_block"` 需要 `"adaptive"`，而这会开启思考并增加输出令牌。

1. 移除错误指出的块，以及其后的每个 `thinking` 和 [`redacted_thinking`](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#redacted-thinking-blocks) 块，否则同样的错误接着会指出下一个无效块。但如果您的历史记录包含 `role: "system"` 消息，或连续两条相同角色的消息，请移除每个 `thinking` 和 `redacted_thinking` 块。错误会将此类消息计为其前一条消息的一部分，因此它指出的位置可能低于您数组中的索引。`input_transformations` 条目的 `path` 则会单独计算每条消息。
2. 保留其他所有块，然后再次发送请求。不要把移除的块放回去。

### 看起来与此类似的错误

这些错误不会说明该块绑定到了其他对话，设置 `prefix_mismatch_behavior` 可能无法消除它们：

* 在 “Invalid `signature` in `thinking` block” 之后就结束、说明 “cannot be modified”，或与 `budget_tokens` 有关的消息：[思考功能故障排查](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting)
* 与 `temperature` 或预填充回复有关的消息：[限制与功能兼容性](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#limits-and-feature-compatibility)

### 何时可以停止发送 `"drop_block"`

要停止发送 `"drop_block"`，请移除它所丢弃的块，并保留此后生成的思考。如果没有其他更改，对话随后在使用 `"error"` 时即可通过。

### 在代码中处理错误

要在代码中处理该错误，请在每个请求中发送 beta 标头和 `"error"`，这样在每个模型上，未通过前缀检查都会表现为 400 错误。当错误出现时，仅将该对话切换为对之后的每个请求使用 `"drop_block"`，将该选择与对话一起存储，再发送一次，并记录该错误。如果同样的错误再次出现，则说明该设置没有到达 API：请参阅[错误仍要求提供标头，或显示 `Extra inputs are not permitted`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#setting-not-received)。

您的代码必须通过其 `message` 字段识别此错误，这是"不要匹配错误文本"这一通用建议的例外情况。每个 400 错误的类型都是 `invalid_request_error`，而此错误没有 `error.details.error_code`。只匹配 “The block is bound to a different conversation”，因为消息的其余部分可能会变化。

### 您没有收到错误，或者错误没有说明更改了什么

* **收到 400 错误，但它没有指出任何更改：**&#x8BF7;自行比较请求。一个可能的原因是发送了压缩块，却没有移除它所总结的消息。

* **出现 `prefix_binding_mismatch` 条目：**&#x5C06;直到第一个条目的 `path` 为止的 `system`、`tools` 和 `messages`，与生成该块时的对话进行比较。如果每个块都被列出，请从 `system` 和 `tools` 开始。

  如果没有任何不同，请查找[内容已更改的图片或文档 URL](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#files-by-id)、MCP 服务器已更改的工具列表（请参阅[固定 MCP 服务器的工具列表](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#pin-mcp-tool-list)），或者被省略、移动或放回的思考块。如果没有请求日志，请使用 `"error"` 再次发送请求，并阅读说明差异的句子（如果有的话）。

* **中间有一个不属于您的代理或库：**&#x6BD4;较它发送的内容与您发送的内容，并向其维护者提供[库、代理和网关](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#libraries-proxies-gateways)。

* **只是怀疑，且没有 Claude Code：**&#x6355;获一个对话的请求正文，其中要包含一次压缩或工具更改。将每个请求正文与前一个进行比较，不计其中新增的轮次。

### 可能失败的测试的五个步骤，以及运行它们的程序

1. 在您自己的代码中，对一个会话的每个请求发送 beta 标头和 `"drop_block"`，发往[具有保留思考的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)。
2. 使用让 Claude 进行思考的提示，并检查第一个响应是否包含 `thinking` 块。使用自适应思考时，Claude 可能会对简单的提示跳过思考。
3. 仅在出现 reason 为 `prefix_binding_mismatch` 的 `input_transformations` 条目时判定失败。期望数组为空的测试在模型切换后也会失败，因为此时出现 `model_binding_mismatch` 或 `organization_binding_mismatch` 条目是预期行为。
4. 有意更改某些内容，例如仅在第三个请求中添加 `system` 提示。
5. 检查 `input_transformations` 是否列出了位于更改之后的思考块。如果它没有列出任何块，请检查历史记录是否包含思考块。

该程序执行步骤 1、2、4 和 5。它会打印每个响应包含多少个思考块和 `input_transformations` 条目，然后打印每个条目，这正是步骤 3 要检查的内容：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 来运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 来运行它。
  </CodeGroupItem>

  ```python Python
  client = anthropic.Anthropic()

  user_turns = [
      "How many positive integers below 500 have exactly 6 positive divisors?",
      "How many have exactly 8?",
      "How many have exactly 10?",
  ]

  messages = []
  for turn, user_turn in enumerate(user_turns, start=1):
      messages.append({"role": "user", "content": user_turn})
      response = client.beta.messages.create(
          model="claude-opus-5-5",
          max_tokens=16000,
          thinking={
              "type": "adaptive",
              "block_binding": {"prefix_mismatch_behavior": "drop_block"},
          },
          # 只有最后一个请求添加了系统提示，这是有意更改前缀
          system="Answer briefly." if turn == len(user_turns) else anthropic.omit,
          messages=messages,
          betas=["thinking-binding-controls-2026-08-01"],
      )
      messages.append({"role": "assistant", "content": response.content})
      thinking_blocks = sum(block.type == "thinking" for block in response.content)
      entries = response.input_transformations or []
      print(f"thinking blocks: {thinking_blocks}, entries: {len(entries)}")
      for entry in entries:
          if entry.type == "thinking_dropped":
              print(entry.type, entry.path, entry.reason)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const userTurns = [
    "How many positive integers below 500 have exactly 6 positive divisors?",
    "How many have exactly 8?",
    "How many have exactly 10?"
  ];

  const messages: Anthropic.Beta.BetaMessageParam[] = [];
  for (const [turnIndex, userTurn] of userTurns.entries()) {
    messages.push({ role: "user", content: userTurn });
    const response = await client.beta.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 16000,
      thinking: {
        type: "adaptive",
        block_binding: { prefix_mismatch_behavior: "drop_block" }
      },
      // 只有最后一个请求添加了系统提示，这会有意改变前缀
      system: turnIndex === userTurns.length - 1 ? "Answer briefly." : undefined,
      messages,
      betas: ["thinking-binding-controls-2026-08-01"]
    });
    messages.push({ role: "assistant", content: response.content });
    const thinkingBlocks = response.content.filter((block) => block.type === "thinking");
    const entries = response.input_transformations ?? [];
    console.log(`thinking blocks: ${thinkingBlocks.length}, entries: ${entries.length}`);
    for (const entry of entries) {
      if (entry.type === "thinking_dropped") {
        console.log(entry.type, entry.path, entry.reason);
      }
    }
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta;
  using Anthropic.Models.Beta.Messages;
  using Model = Anthropic.Models.Messages.Model;

  AnthropicClient client = new();

  string[] userTurns =
  [
      "How many positive integers below 500 have exactly 6 positive divisors?",
      "How many have exactly 8?",
      "How many have exactly 10?",
  ];

  List<BetaMessageParam> messages = [];
  foreach (var (turnIndex, userTurn) in userTurns.Index())
  {
      messages.Add(new BetaUserMessageParam { Content = userTurn });
      var response = await client.Beta.Messages.Create(
          new()
          {
              Model = Model.ClaudeOpus5_5,
              MaxTokens = 16000,
              Thinking = new BetaThinkingConfigAdaptive
              {
                  BlockBinding = new()
                  {
                      PrefixMismatchBehavior = BetaThinkingPrefixMismatchBehavior.DropBlock,
                  },
              },
              // 只有最后一个请求添加了系统提示，这是有意更改前缀
              System = turnIndex == userTurns.Length - 1 ? new("Answer briefly.") : null,
              Messages = messages,
              Betas = [AnthropicBeta.ThinkingBindingControls2026_08_01],
          }
      );
      messages.Add(new BetaAssistantMessageParam
      {
          Content = response.Content.Select(block => new BetaContentBlockParam(block.Json)).ToList(),
      });
      var thinkingBlocks = response.Content.Count(block => block.TryPickThinking(out _));
      var entries = response.InputTransformations ?? [];
      Console.WriteLine($"thinking blocks: {thinkingBlocks}, entries: {entries.Count}");
      foreach (var entry in entries)
      {
          if (entry.TryPickThinkingDropped(out var dropped))
          {
              Console.WriteLine($"{dropped.Type} {dropped.Path} {dropped.Reason.Raw()}");
          }
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  userTurns := []string{
  	"How many positive integers below 500 have exactly 6 positive divisors?",
  	"How many have exactly 8?",
  	"How many have exactly 10?",
  }

  messages := []anthropic.BetaMessageParam{}
  for turnIndex, userTurn := range userTurns {
  	messages = append(messages, anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock(userTurn)))
  	// 只有最后一个请求添加了系统提示，这会有意更改前缀
  	var system []anthropic.BetaTextBlockParam
  	if turnIndex == len(userTurns)-1 {
  		system = []anthropic.BetaTextBlockParam{{Text: "Answer briefly."}}
  	}
  	response, err := client.Beta.Messages.New(context.TODO(), anthropic.BetaMessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 16000,
  		Thinking: anthropic.BetaThinkingConfigParamUnion{
  			OfAdaptive: &anthropic.BetaThinkingConfigAdaptiveParam{
  				BlockBinding: anthropic.BetaThinkingBlockBindingParam{
  					PrefixMismatchBehavior: anthropic.BetaThinkingPrefixMismatchBehaviorDropBlock,
  				},
  			},
  		},
  		System:   system,
  		Messages: messages,
  		Betas:    []anthropic.AnthropicBeta{anthropic.AnthropicBetaThinkingBindingControls2026_08_01},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}
  	messages = append(messages, response.ToParam())
  	thinkingBlocks := 0
  	for _, block := range response.Content {
  		if block.Type == "thinking" {
  			thinkingBlocks++
  		}
  	}
  	fmt.Printf("thinking blocks: %d, entries: %d\n", thinkingBlocks, len(response.InputTransformations))
  	for _, entry := range response.InputTransformations {
  		if entry.Type == "thinking_dropped" {
  			fmt.Println(entry.Type, entry.Path, entry.Reason)
  		}
  	}
  }
  ```

  ```java Java
  import com.anthropic.models.beta.AnthropicBeta;
  import com.anthropic.models.beta.messages.BetaContentBlock;
  import com.anthropic.models.beta.messages.BetaInputTransformation;
  import com.anthropic.models.beta.messages.BetaMessage;
  import com.anthropic.models.beta.messages.BetaThinkingBlockBinding;
  import com.anthropic.models.beta.messages.BetaThinkingConfigAdaptive;
  import com.anthropic.models.beta.messages.BetaThinkingPrefixMismatchBehavior;
  import com.anthropic.models.beta.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      List<String> userTurns = List.of(
          "How many positive integers below 500 have exactly 6 positive divisors?",
          "How many have exactly 8?",
          "How many have exactly 10?");

      MessageCreateParams.Builder conversation = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(16000L)
          .thinking(BetaThinkingConfigAdaptive.builder()
              .blockBinding(BetaThinkingBlockBinding.builder()
                  .prefixMismatchBehavior(BetaThinkingPrefixMismatchBehavior.DROP_BLOCK)
                  .build())
              .build())
          .addBeta(AnthropicBeta.THINKING_BINDING_CONTROLS_2026_08_01);

      for (int turnIndex = 0; turnIndex < userTurns.size(); turnIndex++) {
          if (turnIndex == userTurns.size() - 1) {
              // 仅最后一个请求添加了系统提示，特意改变了前缀
              conversation.system("Answer briefly.");
          }
          conversation.addUserMessage(userTurns.get(turnIndex));
          BetaMessage response = client.beta().messages().create(conversation.build());
          conversation.addMessage(response);
          long thinkingBlocks = response.content().stream()
              .filter(BetaContentBlock::isThinking)
              .count();
          List<BetaInputTransformation> entries = response.inputTransformations().orElse(List.of());
          IO.println("thinking blocks: " + thinkingBlocks + ", entries: " + entries.size());
          for (BetaInputTransformation entry : entries) {
              entry.thinkingDropped().ifPresent(dropped ->
                  IO.println(entry.type() + " " + dropped.path() + " " + dropped.reason()));
          }
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\AnthropicBeta;
  use Anthropic\Beta\Messages\BetaThinkingBlock;
  use Anthropic\Beta\Messages\BetaThinkingBlockBinding;
  use Anthropic\Beta\Messages\BetaThinkingConfigAdaptive;
  use Anthropic\Beta\Messages\BetaThinkingDroppedInputTransformation;
  use Anthropic\Beta\Messages\BetaThinkingPrefixMismatchBehavior;
  use Anthropic\Client;
  use Anthropic\Messages\Model;

  $client = new Client();

  $userTurns = [
      'How many positive integers below 500 have exactly 6 positive divisors?',
      'How many have exactly 8?',
      'How many have exactly 10?',
  ];

  $messages = [];
  foreach ($userTurns as $turnIndex => $userTurn) {
      $messages[] = ['role' => 'user', 'content' => $userTurn];
      $response = $client->beta->messages->create(
          model: Model::CLAUDE_OPUS_5_5,
          maxTokens: 16000,
          thinking: BetaThinkingConfigAdaptive::with(
              blockBinding: BetaThinkingBlockBinding::with(
                  prefixMismatchBehavior: BetaThinkingPrefixMismatchBehavior::DROP_BLOCK,
              ),
          ),
          // 只有最后一个请求添加了系统提示，这是有意更改前缀
          system: $turnIndex === array_key_last($userTurns) ? 'Answer briefly.' : null,
          messages: $messages,
          betas: [AnthropicBeta::THINKING_BINDING_CONTROLS_2026_08_01],
      );
      $messages[] = ['role' => 'assistant', 'content' => $response->content];
      $thinkingBlocks = array_filter(
          $response->content,
          fn ($block) => $block instanceof BetaThinkingBlock,
      );
      $entries = $response->inputTransformations ?? [];
      echo 'thinking blocks: ', count($thinkingBlocks), ', entries: ', count($entries), PHP_EOL;
      foreach ($entries as $entry) {
          if ($entry instanceof BetaThinkingDroppedInputTransformation) {
              echo $entry->type, ' ', $entry->path, ' ', $entry->reason, PHP_EOL;
          }
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  user_turns = [
    "How many positive integers below 500 have exactly 6 positive divisors?",
    "How many have exactly 8?",
    "How many have exactly 10?"
  ]

  messages = []
  user_turns.each_with_index do |user_turn, turn_index|
    messages << { role: "user", content: user_turn }
    # 只有最后一个请求添加了系统提示，这是有意更改前缀
    system_param = (turn_index == user_turns.length - 1) ? { system_: "Answer briefly." } : {}
    response = client.beta.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 16_000,
      thinking: {
        type: "adaptive",
        block_binding: { prefix_mismatch_behavior: "drop_block" }
      },
      messages: messages,
      betas: [Anthropic::AnthropicBeta::THINKING_BINDING_CONTROLS_2026_08_01],
      **system_param
    )
    messages << { role: "assistant", content: response.content }
    thinking_blocks = response.content.count { |block| block.is_a?(Anthropic::Beta::BetaThinkingBlock) }
    entries = response.input_transformations || []
    puts "thinking blocks: #{thinking_blocks}, entries: #{entries.length}"
    entries.each do |entry|
      if entry.is_a?(Anthropic::Beta::BetaThinkingDroppedInputTransformation)
        puts "#{entry.type} #{entry.path} #{entry.reason}"
      end
    end
  end
  ```
</CodeGroup>

```text Output wrap
thinking blocks: 1, entries: 0
thinking blocks: 1, entries: 0
thinking blocks: 1, entries: 2
thinking_dropped messages.1.content.0 prefix_binding_mismatch
thinking_dropped messages.3.content.0 prefix_binding_mismatch
```

将 `"drop_block"` 换成 `"error"` 后，第三个请求会收到 [400 错误](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#prefix-mismatch-error)。

### 每个 `input_transformations` 条目的含义，以及哪些需要您修复

使用 beta 标头时，来自支持思考的模型的响应会在 `usage` 旁边包含 `input_transformations` 数组。当没有任何内容被丢弃、也没有无效内容被放行时，它为空。使用流式传输时，它位于 `message_start` 事件的 `message` 中；如果在流中途发生回退，则以最终 `message_delta` 事件携带的那个为准。与错误中的位置不同，`path` 始终是该块在您发送内容中的位置：

```json
{
  "input_transformations": [
    {
      "type": "thinking_dropped",
      "path": "messages.1.content.0",
      "reason": "prefix_binding_mismatch"
    }
  ]
}
```

每个条目的 `type` 和 `reason` 及其含义：

* **`thinking_dropped`、`prefix_binding_mismatch`：**&#x8BE5;块之前的内容与它所记录的不一致。需要您修复：[找出您的代码更改了什么](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#find-the-change)。
* **`thinking_mismatch_allowed`、`prefix_binding_mismatch`：**&#x60C5;况相同，但 Claude 原样读取了该块，并且会对其计费。此条目仅在请求不包含该设置时，才会在[某些模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)上出现。需要您修复。
* **`thinking_dropped`、`model_binding_mismatch`：**&#x6B64;模型无法读取由生成它的模型所产生的块。无需您修复：[继续发送整个对话](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#switching-models)。
* **`thinking_dropped`、`organization_binding_mismatch`：**&#x8BE5;块由另一个账户生成，因此 [API 丢弃了它](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)。无需您修复。

忽略 `type` 或 `reason` 是您无法识别的条目。

### 批处理和令牌计数

* **[消息批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)：**&#x4E0D;包含 `prefix_mismatch_behavior` 的批处理项永远不会收到 400 错误。API 会丢弃无效块，或者在[某些模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)上可能会放行它们。设置了 `"error"` 的批处理项在某个块未通过前缀检查时会以 `errored` 结束。`block_binding` 放在该批处理项的 `params.thinking` 中。
* **[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)：**&#x5728; Claude API 上，令牌计数可能会返回相同的 400 错误，并在使用相同标头时接受 `prefix_mismatch_behavior`。它不会将被丢弃的块计入计数，也不会返回 `input_transformations`。

## 后续步骤

<CardGroup cols={2}>
  <Card title="更改工具或指令而不会导致前缀检查失败" icon="edit" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions">
    针对您的代码对对话所做的每种更改，改为发送什么内容。
  </Card>

  <Card title="保留思考" icon="book" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking">
    什么是保留思考、思考块与什么绑定，以及切换模型或账户。
  </Card>

  <Card title="思考功能故障排查" icon="wrench" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-troubleshooting">
    其他思考错误的修复方法，例如签名无法验证。
  </Card>
</CardGroup>
