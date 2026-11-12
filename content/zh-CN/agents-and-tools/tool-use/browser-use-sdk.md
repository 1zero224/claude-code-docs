---
title: 使用 SDK 工具集进行浏览器和计算机使用
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk
description: 从 Python 或 TypeScript SDK 运行浏览器使用工具或计算机使用工具。SDK 负责运行循环和您配置的检查，您负责提供浏览器或桌面。
featureMetadata:
  status: beta
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
  supportedPlatforms:
    Claude API: beta
    Amazon Bedrock: beta
    Claude Platform on AWS: beta
    Claude Models on Google Cloud (Google managed): beta
    Microsoft Foundry: beta
---

借助 Python 和 TypeScript SDK 中的浏览器和计算机工具集，您可以构建在 Web 浏览器或桌面上工作的智能体。它可以使用没有编程接口的应用程序。通过[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)，Claude 可以导航、读取网页并与之交互。通过[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)，Claude 接收桌面截图并使用鼠标和键盘。

SDK 将 Claude 连接到您的浏览器或桌面。它会运行您配置的检查，例如针对 `navigate` 的 URL 策略或批准回调。它将每个通过检查的调用发送给您的 “driver”（驱动程序），并根据驱动程序返回的内容构建 `tool_result`。[工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)可以为您运行智能体循环。

您需要提供驱动程序，由它来操作浏览器或桌面。可以从[提供商的驱动程序](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#use-a-driver-from-a-provider)开始。使用现成的驱动程序，只需几行代码即可构建智能体。如果没有合适的，可以使用 Playwright 等浏览器自动化库编写您自己的驱动程序。

本页大部分内容介绍浏览器工具集。[计算机工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#computer-toolset)介绍两者的不同之处。

编写您自己的驱动程序属于高级用法，详见[实现驱动程序](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#implement-a-driver)。claude-quickstarts 仓库中还提供了 Python 和 TypeScript 版本的[浏览器驱动程序示例](https://github.com/anthropics/claude-quickstarts/tree/main/browser-toolset)。它通过 “Chrome DevTools Protocol”（Chrome 开发者工具协议），即 CDP 控制 Chromium，并且不是生产代码。

<Note>
  浏览器工具集类目前处于 beta 阶段，可在 [Python SDK](https://github.com/anthropics/anthropic-sdk-python/blob/main/browser-toolset.md) 和 [TypeScript SDK](https://github.com/anthropics/anthropic-sdk-typescript/blob/main/browser-toolset.md) 中使用。计算机工具集类在相同的 SDK 中同样处于 beta 阶段，这些 SDK 提供了 [Python 计算机工具集指南](https://github.com/anthropics/anthropic-sdk-python/blob/main/computer-toolset.md)和 [TypeScript 计算机工具集指南](https://github.com/anthropics/anthropic-sdk-typescript/blob/main/computer-toolset.md)。
</Note>

## 使用提供商的驱动程序

借助来自提供商的驱动程序（例如[合作伙伴集成](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#partner-integrations)下列出的驱动程序），您只需几行代码即可构建智能体。此示例中的提供商是虚构的。请将 `example_browser_provider` (typescript: `example-browser-provider`) 和 `ExampleBrowser` 替换为真实提供商的包和驱动程序类，并按照提供商文档的说明进行设置。

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  from anthropic import Anthropic
  from example_browser_provider import ExampleBrowser  # made-up provider

  client = Anthropic()
  with ExampleBrowser() as browser:
      runner = client.beta.messages.tool_runner(
          model="claude-opus-5-5",
          max_tokens=1024,
          tools=[browser],
          messages=[
              {"role": "user", "content": "Open example.com and tell me its heading."}
          ],
      )
      for message in runner:
          print(message)
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";
  import { ExampleBrowser } from "example-browser-provider"; // made-up provider

  const client = new Anthropic();
  const browser = new ExampleBrowser();
  try {
    const runner = client.beta.messages.toolRunner({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      tools: [browser],
      messages: [{ role: "user", content: "Open example.com and tell me its heading." }]
    });
    for await (const message of runner) {
      console.log(message);
    }
  } finally {
    await browser.close();
  }
  ```
</CodeGroup>

将驱动程序实例本身作为 `tools` 条目传入。运行器从不关闭工具集，因此一个实例可以服务多次运行。使用完毕后请将其关闭。

该示例的客户端调用 Claude API。对于 [API 支持](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#availability-feature-support)下列出的其他平台，请改为创建该平台的 SDK 客户端。每个平台的页面都展示了其对应的客户端。

要编写您自己的驱动程序，请参阅[实现驱动程序](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#implement-a-driver)。

## 合作伙伴集成

Browser Use、Browserbase、Daytona 和 E2B 为 SDK 工具集发布了各自的驱动程序。

<CardGroup cols={2}>
  <Card id="browser-use" title="Browser Use" image="https://platform.claude.com/docs/images/partner-logos/browser-use.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/browser-use-dark.svg" href="https://docs.browser-use.com/open-source/customize/integrations/toolsets-for-claude">
    适用于浏览器工具集的 Browser Use 驱动程序。
  </Card>

  <Card id="browserbase" title="Browserbase" image="https://platform.claude.com/docs/images/partner-logos/browserbase.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/browserbase-dark.svg" href="https://docs.stagehand.dev/v4/integrations/agent-frameworks/claude-cua-toolset-quickstart">
    适用于浏览器工具集的 Browserbase Stagehand 驱动程序。
  </Card>

  <Card id="daytona" title="Daytona" image="https://platform.claude.com/docs/images/partner-logos/daytona.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/daytona-dark.svg" href="https://www.daytona.io/docs/en/guides/claude/claude-draws-daytona-sandbox/">
    适用于计算机工具集的 Daytona 驱动程序。
  </Card>

  <Card id="e2-b" title="E2B" image="https://platform.claude.com/docs/images/partner-logos/e2b.svg" imageDark="https://platform.claude.com/docs/images/partner-logos/e2b-dark.svg" href="https://docs.e2b.dev/agents/claude-toolsets">
    适用于浏览器和计算机工具集的 E2B 驱动程序。
  </Card>
</CardGroup>

合作伙伴代码也可在 GitHub 上获取：[Browser Use 快速入门](https://github.com/browser-use/browser-use/tree/main/examples/integrations/toolsets-for-claude)、[Browserbase 工具集](https://github.com/browserbase/claude-cua-toolset)以及 [E2B 示例](https://github.com/e2b-dev/e2b-cookbook/tree/main/examples/anthropic-computer-use-orangehrm-js)。

<Warning>
  在针对一次性浏览器以外的任何浏览器运行驱动程序之前，请先完成以下两件事：

  * [编写 URL 策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#write-a-url-policy)。如果没有 URL 策略，SDK 不会检查任何 URL。对于由提供商托管的浏览器，还请阅读[远程和托管浏览器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#remote-and-hosted-browsers)。
  * 确保所用的浏览器配置文件未登录任何您不愿将其数据或操作交给 Claude 的账户。请参阅[隔离浏览器主机](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#isolate-the-browser-host)。

  [安全地运行工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#run-the-toolset-safely)介绍了其余事项。
</Warning>

## 自定义驱动程序

本节中的示例使用 `MyBrowser`，即本页后面[高级示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#quick-start)中的驱动程序。

### 启用或禁用成员

`configs` 接受[配置工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#configure-the-toolset)中描述的各成员设置。只需列出您要更改的成员：

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  # 同时实现了 read_console 的 MyBrowser
  browser = MyBrowser(
      backend, configs={"read_console": {"enabled": True}, "navigate": {"enabled": False}}
  )
  ```

  ```typescript TypeScript
  // 同时实现了 read_console 的 MyBrowser
  const browser = new MyBrowser(backend, {
    configs: { read_console: { enabled: true }, navigate: { enabled: false } }
  });
  ```
</CodeGroup>

SDK 会在您的代码运行之前拒绝对已禁用成员的调用。启用您的类未实现的成员属于配置错误，除非该类重写了 `execute`。

### 添加前置和后置钩子

重写 `execute` 并调用父类的 `execute`。该调用之前的代码在 [URL 策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#write-a-url-policy)、[文件策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#confine-uploads-and-downloads)和 [`confirm`](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#gate-consequential-members) 之后运行，并且可以修改输入。SDK 不会再次检查修改后的输入。该调用之后的代码会接收结果，并且可以修改结果。抛出 `ToolError`（在 TypeScript 中使用 throw）即可拒绝该调用。

重写 `execute` 会改变向 Claude 提供哪些成员。SDK 会将每个成员都视为已实现，因此 Claude 会获得所有默认开启的成员。[高级示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#quick-start)中的 `MyBrowser` 只支持三个成员，因此下面的 `TracedBrowser` 会向 Claude 提供它无法支持的成员。使用之前，请通过 `configs` 关闭这些成员。

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  import time


  class TracedBrowser(MyBrowser):
      def execute(self, context, name, input):
          started = time.monotonic()
          result = super().execute(context, name, input)
          elapsed_ms = (time.monotonic() - started) * 1000
          call_id = context.tool_use.id if context.tool_use else "-"
          log.info("%s %s %.0fms", call_id, name, elapsed_ms)
          return redact(result) if name == "get_page_text" else result
  ```

  ```typescript TypeScript
  import type { BetaToolsetCallContext } from "@anthropic-ai/sdk/helpers/beta/toolsets";
  import type {
    BetaBrowserMemberInput,
    BetaBrowserMemberName
  } from "@anthropic-ai/sdk/resources/beta";

  class TracedBrowser extends MyBrowser {
    protected override async execute(
      ctx: BetaToolsetCallContext,
      name: BetaBrowserMemberName,
      input: BetaBrowserMemberInput
    ) {
      const started = performance.now();
      const result = await super.execute(ctx, name, input);
      log.info(`${ctx.toolUse?.id ?? "-"} ${name} ${Math.round(performance.now() - started)}ms`);
      return name === "get_page_text" ? redact(result as string) : result;
    }
  }
  ```
</CodeGroup>

## 实现驱动程序

驱动程序是您编写的 `BetaAbstractBrowserToolset20260801` 子类。在其中重写您的浏览器所支持的成员。您未实现的成员会以禁用状态发送给 API。如果 Claude 仍然调用它，SDK 会返回错误，运行继续进行。重写 `execute` 会改变哪些成员以禁用状态发送（[添加前置和后置钩子](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#add-before-and-after-hooks)）。每个成员都会接收调用上下文以及该成员的输入（一个类型化对象，例如 `BetaBrowserNavigateInput`）。输入类型来自 `anthropic.types.beta` (typescript: `@anthropic-ai/sdk/resources/beta`)。[成员工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#member-tools)列出了每种输入的字段。

在 TypeScript 中，请将 `type` 成员拼写为 `type_`，并将成员编写为方法而不是箭头函数字段，因为 SDK 会在原型上查找它们。

### 返回结果

成员返回的内容决定了 Claude 读取的内容。成功的结果以根据您的状态报告构建的 `browser_state` 块结尾。错误结果不带该块。

| 成员                                                                                 | 返回                                                                           | Claude 读取的内容                                   |
| ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------- |
| `screenshot`、`zoom`                                                                | `BetaScreenshotResult`                                                       | 一个图像块                                          |
| `navigate`                                                                         | `BetaBrowserNavigateResult`                                                  | `Navigated to {url} — {title} (HTTP {status})` |
| `new_tab`、`switch_tab`、`list_tabs`、`close_tab`                                     | 一个标签页条目（`new_tab`、`switch_tab`）、一个标签页条目列表（`list_tabs`），或不返回任何内容（`close_tab`） | 仅 `browser_state` 块                            |
| `read_page`、`get_page_text`、`find`、`read_console`、`read_network`、`javascript_exec` | 一个字符串                                                                        | 该字符串；如果是空字符串，则为 `(empty)`                      |
| 其他所有成员                                                                             | 不返回任何内容，或一行文本                                                                | 一条简短的确认信息（例如 `Clicked.`），然后是位于单独文本块中的返回行       |

### 报告浏览器状态

SDK 会在每个返回结果的调用之后调用 `_browser_state` (typescript: `browserState`)，包括被拒绝和失败的调用。返回每个打开的标签页，以及自上次报告以来发生的变化：

* 打开的标签页和下载事件
* 针对您的[请求钩子](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#intercept-requests-in-the-driver)拦截的每次导航的一个 `BetaNavigationRefused`
* 针对您的驱动程序关闭的每个原生对话框的一个 `BetaDialogDismissed`

将所有这些内容放入 `state_changes`。被拒绝的导航是 `anthropic.tools.browser.BetaNavigationRefused()` (typescript: `{ type: "navigation_refused" }`)，被关闭的对话框是 `anthropic.tools.browser.BetaDialogDismissed(kind=..., message=...)` (typescript: `{ type: "dialog_dismissed", kind, message }`)。

后两者不是 API 状态变化。SDK 会以 `browser_state` 块之外的文本形式将它们报告给 Claude：所有被拒绝的导航合并为一行（不包含 URL），前三个被关闭的对话框各占一行，其余的则给出数量。

当有任何标签页打开时，必须恰好有一个处于活动状态。每个接受 `tab_id` 的成员都必须作用于它所指定的标签页。API 对报告的限制列在[使用 `browser_state` 跟踪标签页](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#track-tabs-and-page-state)中。

Claude 会按照您的驱动程序所写的原样读取每个标签页的 URL、每个下载的 URL 以及您的 `navigate` 返回的 URL。SDK 不会解析这些 URL。它会将换行符和其他控制字符转换为空格，修剪两端，并将每个 URL 截断为 4,096 个字符。因此，`data:` URL 的内容、`file:` URL 的路径以及 URL 中的用户名或密码都会传递给 Claude。

### 处理错误

| 由成员或 SDK 引发         | Claude 读取的内容                                               | 运行状态 |
| ------------------- | ---------------------------------------------------------- | ---- |
| `ToolError`         | 其消息，作为错误结果                                                 | 继续   |
| `ToolsetUsageError` | 无                                                          | 停止   |
| 任何其他异常              | `ClassName: message` (typescript: `Error: message`)，作为错误结果 | 继续   |

SDK 会在出现配置错误、调用期间误用 SDK，或在 `close` 之后进行调用时抛出 `ToolsetUsageError`。当 `_browser_state` (typescript: `browserState`) 抛出任何异常（即使是 `ToolError`）时，它也会抛出该错误。工具运行器和 `tool_result` (typescript: `toolResult`) 不会捕获它，因此它会传递到它们的调用方。

SDK 不会隐藏以下内容中的本地路径：成员的错误文本、`left_click` 等操作返回的行、失败下载的错误，以及被关闭对话框的消息。

来自您的 URL 策略、文件策略或 `confirm` 可调用对象的 `ToolError` 会按原样传达给 Claude，因此请不要在其文本中包含本地路径。请在您的成员中捕获异常，并使用您自己的文本抛出 `ToolError`（在 TypeScript 中使用 throw）。

### 在响应流式传输时启动调用

默认情况下，工具运行器会在 Claude 的响应结束后运行该轮次的调用。启用提前启动后，它可以在响应仍在流式传输时启动浏览器或计算机调用。

要启用此功能，请向工具运行器传入 `stream=True` (typescript: `stream: true`) 和 `run_tools_eagerly=True` (typescript: `runToolsEagerly: true`)，如[高级示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#quick-start)所示。使用 `stream` 时，您对运行器的循环每次迭代得到的是一个流，而不是一条消息。

提前启动改变的是调用何时可以开始，而不是工具集如何运行它：

* **检查仍会先运行：**&#x53;DK 会在调用运行之前调用您的 URL 策略、文件策略和 `confirm`。启用提前启动时，它可能会在响应仍在流式传输时调用 `confirm`。
* **调用仍会逐个运行：**&#x5DE5;具集会按照 Claude 编写的顺序运行其调用。失败的调用仍会停止该工具集在本轮中的后续调用。
* **已开始的调用无法撤回：**&#x5982;果响应随后被截断（例如在 `max_tokens` 处），或者您的循环提前停止，该操作仍会发生，而 Claude 永远不会读取其结果。

### 不使用工具运行器运行

在 `tools` 中传递 `browser` (typescript: `browser.toJSON()`)，并使用 `tool_result` (typescript: `toolResult`) 响应每个成员调用。浏览器使用工具要求您在第一个失败的调用处停止（[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#batch-actions)）。

此循环使用[高级示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#quick-start)中的 `MyBrowser` 和 URL 策略。在某个调用失败后，它会响应该轮次的后续调用而不运行它们：

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  from anthropic.types.beta import BetaMessageParam, BetaToolResultBlockParam

  NOT_EXECUTED = "Not executed: an earlier action in this turn failed."
  MAX_TURNS = 10

  with MyBrowser(backend, url_policy=url_policy) as browser:
      messages: list[BetaMessageParam] = [{"role": "user", "content": "Open example.com"}]
      for _ in range(MAX_TURNS):
          response = client.beta.messages.create(
              model="claude-opus-5-5",
              max_tokens=1024,
              tools=[browser],
              messages=messages,
          )
          messages.append({"role": "assistant", "content": response.content})
          calls = [
              block
              for block in response.content
              if block.type == "tool_use" and block.toolset_name == browser.toolset_name
          ]
          if not calls:
              break
          results: list[BetaToolResultBlockParam] = []
          failed = False
          for call in calls:
              if failed:
                  # 某次调用失败后，本轮其余调用只返回应答，不再实际执行。
                  results.append(
                      {
                          "type": "tool_result",
                          "tool_use_id": call.id,
                          "toolset_name": call.toolset_name,
                          "content": NOT_EXECUTED,
                          "is_error": True,
                      }
                  )
                  continue
              result = browser.tool_result(call)
              failed = bool(result.get("is_error"))
              results.append(result)
          messages.append({"role": "user", "content": results})
  ```

  ```typescript TypeScript
  const NOT_EXECUTED = "Not executed: an earlier action in this turn failed.";
  const MAX_TURNS = 10;

  const browser = new MyBrowser(backend, { urlPolicy });
  try {
    const messages: Anthropic.Beta.BetaMessageParam[] = [
      { role: "user", content: "Open example.com" }
    ];
    for (let turn = 0; turn < MAX_TURNS; turn++) {
      const response = await client.beta.messages.create({
        model: "claude-opus-5-5",
        max_tokens: 1024,
        tools: [browser.toJSON()],
        messages
      });
      messages.push({ role: "assistant", content: response.content });
      const calls = response.content.filter(
        (block): block is Anthropic.Beta.BetaToolUseBlock =>
          block.type === "tool_use" && block.toolset_name === browser.toolsetName
      );
      if (calls.length === 0) {
        break;
      }
      const results: Anthropic.Beta.BetaToolResultBlockParam[] = [];
      let failed = false;
      for (const call of calls) {
        if (failed) {
          // 调用失败后，本轮其余部分仅作应答，不再执行。
          results.push({
            type: "tool_result",
            tool_use_id: call.id,
            toolset_name: call.toolset_name,
            content: NOT_EXECUTED,
            is_error: true
          });
          continue;
        }
        const result = await browser.toolResult(call);
        failed = result.is_error === true;
        results.push(result);
      }
      messages.push({ role: "user", content: results });
    }
  } finally {
    await browser.close();
  }
  ```
</CodeGroup>

对每个被跳过调用的响应都带有 `is_error`、该调用的 `toolset_name`，以及[批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#batch-actions)所要求的确切文本。工具运行器会发送相同的响应。

## 高级示例：编写您自己的驱动程序

此示例将[实现驱动程序](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#implement-a-driver)中的主要部分整合到一个驱动程序 `MyBrowser` 中，并使用工具运行器和[提前启动](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#start-calls-while-the-response-streams)来运行它。`MyBrowser` 实现了 `navigate`、`screenshot` 和 `left_click`，以及每个驱动程序都需要的状态报告 `_browser_state` (typescript: `browserState`)。

`backend` 代表您自己对浏览器自动化库（例如 Playwright）的封装。URL 策略只是一个示例。[编写 URL 策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#write-a-url-policy)对其进行了说明。

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  import re
  from urllib.parse import urlsplit

  from anthropic import Anthropic
  from anthropic.tools import ToolError
  from anthropic.tools.browser import (
      BetaAbstractBrowserToolset20260801,
      BetaBrowserNavigateResult,
      BetaBrowserState,
      BetaScreenshotResult,
      BetaToolsetCallContext,
      BetaURLContext,
  )
  from anthropic.types.beta import (
      BetaBrowserLeftClickInput,
      BetaBrowserNavigateInput,
      BetaBrowserScreenshotInput,
      BetaBrowserStateTabEntryParam,
  )


  class MyBrowser(BetaAbstractBrowserToolset20260801):
      def __init__(self, backend, **options):
          super().__init__(**options)
          self.backend = backend

      def _browser_state(self, context: BetaToolsetCallContext) -> BetaBrowserState:
          return BetaBrowserState(
              tabs=[
                  BetaBrowserStateTabEntryParam(
                      tab_id=tab.id,
                      title=tab.title,
                      url=tab.url,
                      active=tab.id == self.backend.active,
                  )
                  for tab in self.backend.tabs()
              ],
              state_changes=self.backend.drain_changes(),
          )

      def navigate(
          self, context: BetaToolsetCallContext, input: BetaBrowserNavigateInput
      ) -> BetaBrowserNavigateResult:
          # input.url 为 "back"、"forward"、"reload"，或是您的 URL 策略
          # 已允许的 URL（保持 Claude 所写的原样）。与 is_allowed 一样，backend.goto 会为
          # 不以协议开头的 URL 添加 https://。
          page = self.backend.goto(input.url, input.tab_id)
          return BetaBrowserNavigateResult(
              url=page.url, status=page.status, title=page.title
          )

      def screenshot(
          self, context: BetaToolsetCallContext, input: BetaBrowserScreenshotInput
      ) -> BetaScreenshotResult:
          data = self.backend.png_base64(input.tab_id)
          return BetaScreenshotResult(data=data, media_type="image/png")

      def left_click(
          self, context: BetaToolsetCallContext, input: BetaBrowserLeftClickInput
      ) -> None:
          # 无需返回内容：Claude 读取到的是 "Clicked."
          self.backend.click(input.target, input.tab_id)

      def close(self) -> None:
          super().close()  # first, so no call is still using the browser when it closes
          if not self.backend.closed:
              self.backend.close()


  ALLOWED_HOSTS = ("example.com", "iana.org")
  SCHEME_PREFIX = re.compile(r"[a-z][a-z0-9+.-]*:", re.IGNORECASE)


  def is_allowed(url: str) -> bool:
      # 仅为示例，并非生产环境策略。请参阅 "编写 URL 策略"。
      if url.lower() == "about:blank":
          return True
      with_scheme = url if SCHEME_PREFIX.match(url) else f"https://{url}"
      # 在 Web URL 中，浏览器会将 "\" 视为 "/"。
      try:
          parts = urlsplit(with_scheme.replace("\\", "/"))
      except ValueError:
          return False
      host = parts.hostname or ""
      listed = any(host == name or host.endswith(f".{name}") for name in ALLOWED_HOSTS)
      return parts.scheme in ("http", "https") and listed


  def url_policy(context: BetaURLContext, url: str) -> None:
      if not is_allowed(url):
          raise ToolError(f"blocked: {url} is not on an allowed host")


  client = Anthropic()
  with MyBrowser(backend, url_policy=url_policy) as browser:
      runner = client.beta.messages.tool_runner(
          model="claude-opus-5-5",
          max_tokens=1024,
          tools=[browser],
          messages=[
              {
                  "role": "user",
                  "content": "Open example.com and tell me the page heading.",
              }
          ],
          stream=True,
          run_tools_eagerly=True,  # so a call can start before the response ends
      )
      for stream in runner:
          print(stream.get_final_message())
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";
  import {
    BetaAbstractBrowserToolset20260801,
    type BetaBrowserNavigateResult,
    type BetaBrowserToolsetOptions,
    type BetaScreenshotResult,
    type BetaToolsetCallContext,
    type BetaURLContext,
    ToolError
  } from "@anthropic-ai/sdk/helpers/beta/toolsets";
  import type {
    BetaBrowserLeftClickInput,
    BetaBrowserNavigateInput,
    BetaBrowserScreenshotInput
  } from "@anthropic-ai/sdk/resources/beta";

  class MyBrowser extends BetaAbstractBrowserToolset20260801 {
    constructor(
      private backend: Backend,
      options: Omit<BetaBrowserToolsetOptions, "browserState"> = {}
    ) {
      super({
        ...options,
        // 必需：每个打开的标签页，以及自上次报告以来发生的变化。
        browserState: () => ({
          tabs: backend.tabs().map((tab) => ({
            tab_id: tab.id,
            title: tab.title,
            url: tab.url,
            active: tab.id === backend.active
          })),
          state_changes: backend.drainChanges()
        })
      });
    }

    protected override async navigate(
      ctx: BetaToolsetCallContext,
      input: BetaBrowserNavigateInput
    ): Promise<BetaBrowserNavigateResult> {
      // input.url 为 "back"、"forward"、"reload"，或您的 URL 策略
      // 所允许的 URL（按 Claude 写出的原样）。与 isAllowed 一样，backend.goto 会为
      // 不以协议开头的 URL 添加 https://。
      const page = await this.backend.goto(input.url, input.tab_id);
      return { url: page.url, status: page.status, title: page.title };
    }

    protected override async screenshot(
      ctx: BetaToolsetCallContext,
      input: BetaBrowserScreenshotInput
    ): Promise<BetaScreenshotResult> {
      return { data: await this.backend.pngBase64(input.tab_id), mediaType: "image/png" };
    }

    protected override async left_click(
      ctx: BetaToolsetCallContext,
      input: BetaBrowserLeftClickInput
    ): Promise<void> {
      // 无需返回内容：Claude 读到的是 "Clicked."
      await this.backend.click(input.target, input.tab_id);
    }

    override async close(): Promise<void> {
      await super.close(); // first, so no call is still using the browser when it closes
      if (!this.backend.closed) {
        await this.backend.close();
      }
    }
  }

  const ALLOWED_HOSTS = ["example.com", "iana.org"];
  const SCHEME_PREFIX = /^[a-z][a-z0-9+.-]*:/i;

  // 这只是示例，并非生产环境策略。请参阅"编写 URL 策略"。
  function isAllowed(url: string): boolean {
    if (url.toLowerCase() === "about:blank") {
      return true;
    }
    const withScheme = SCHEME_PREFIX.test(url) ? url : `https://${url}`;
    // 在 Web URL 中，浏览器会将 "\" 视为 "/"。
    const parsed = URL.parse(withScheme.replaceAll("\\", "/"));
    const host = parsed?.hostname ?? "";
    const listed = ALLOWED_HOSTS.some((name) => host === name || host.endsWith(`.${name}`));
    return (parsed?.protocol === "http:" || parsed?.protocol === "https:") && listed;
  }

  function urlPolicy(ctx: BetaURLContext, url: string): void {
    if (!isAllowed(url)) {
      throw new ToolError(`blocked: ${url} is not on an allowed host`);
    }
  }

  const client = new Anthropic();
  const browser = new MyBrowser(backend, { urlPolicy });
  try {
    const runner = client.beta.messages.toolRunner({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      tools: [browser],
      messages: [{ role: "user", content: "Open example.com and tell me the page heading." }],
      stream: true,
      runToolsEagerly: true // so a call can start before the response ends
    });
    for await (const stream of runner) {
      console.log(await stream.finalMessage());
    }
  } finally {
    await browser.close();
  }
  ```
</CodeGroup>

## 安全地运行工具集

Claude 的下一步操作取决于它读取的页面。页面或注入其中的文本可能会试图访问内部服务或从主机上提取文件。它还可能试图触发具有实际影响的操作。在针对一次性浏览器以外的任何对象运行驱动程序之前，请执行以下六个步骤：

1. [编写 URL 策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#write-a-url-policy)，拒绝任务不需要的每个 `navigate` URL。
2. [在驱动程序中拦截请求](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#intercept-requests-in-the-driver)，并使用相同的规则检查每个请求。
3. [在容器上设置出站策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#put-egress-policy-on-the-container)，让网络阻止驱动程序无法看到的内容。
4. [限制上传和下载](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#confine-uploads-and-downloads)，或保持上传关闭。
5. 使用 `confirm` [对有重大影响的成员进行把关](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#gate-consequential-members)。
6. 为每个会话在专用容器或虚拟机中[隔离浏览器主机](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#isolate-the-browser-host)。

SDK 通过您传入的策略和可调用对象来执行第 1、4 和 5 步。第 2、3 和 6 步由您的驱动程序和部署负责。[安全注意事项](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#security-considerations)中的预防措施同样适用。

### 编写 URL 策略

<Warning>
  如果没有 URL 策略，SDK 不会检查任何 URL，API 也不会过滤 Claude 打开的 URL。即使有策略，Claude 点击的链接或重定向也可能将浏览器带到环回地址（`localhost`、`127.0.0.1`）、链路本地地址（例如云元数据地址 `169.254.169.254`）以及私有网络范围。[容器上的出口规则](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#put-egress-policy-on-the-container)可以阻止链路本地地址和私有地址。在容器外部强制执行的规则无法阻止环回地址。

  SDK 不会检查 URL 的协议（scheme），因此您的驱动程序必须拒绝所有您不打算让它打开的协议，例如 `javascript:`、`view-source:`、`data:` 和 `file:`。大多数驱动程序只需要 `http` 和 `https`。本节后面的示例策略仅允许 `http` 和 `https` URL，以及 `about:blank`。

  `file:` URL 允许 Claude 读取浏览器主机上的文件。`javascript:` URL 会在当前页面中运行脚本，即使 `javascript_exec` 已关闭。出口规则无法阻止这两者。
</Warning>

URL 策略是一个您作为 `url_policy` (typescript: `urlPolicy`) 传入的函数。策略由您编写，您负责使其达到生产就绪状态。在您的 `navigate` 运行之前，SDK 会使用调用的上下文和 Claude 编写的原始 URL 调用该策略。

不返回任何内容即表示允许该 URL。要拒绝它，请抛出 `ToolError`（在 TypeScript 中使用 throw）。Claude 会读取该错误的消息，且 `navigate` 不会运行。该策略只检查每个 `navigate` 调用中的 URL。它不是网络控制手段。

以下是[高级示例](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#quick-start)中的策略。它允许 `about:blank`，以及通过 `http` 和 `https` 访问的两个站点及其子域名：

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  import re
  from urllib.parse import urlsplit

  from anthropic.tools import ToolError
  from anthropic.tools.browser import BetaURLContext

  ALLOWED_HOSTS = ("example.com", "iana.org")
  SCHEME_PREFIX = re.compile(r"[a-z][a-z0-9+.-]*:", re.IGNORECASE)


  def is_allowed(url: str) -> bool:
      # 仅为示例，并非生产环境策略。
      if url.lower() == "about:blank":
          return True
      with_scheme = url if SCHEME_PREFIX.match(url) else f"https://{url}"
      # 在 Web URL 中，浏览器会将 "\" 视为 "/"。
      try:
          parts = urlsplit(with_scheme.replace("\\", "/"))
      except ValueError:
          return False
      host = parts.hostname or ""
      listed = any(host == name or host.endswith(f".{name}") for name in ALLOWED_HOSTS)
      return parts.scheme in ("http", "https") and listed


  def url_policy(context: BetaURLContext, url: str) -> None:
      if not is_allowed(url):
          raise ToolError(f"blocked: {url} is not on an allowed host")


  browser = MyBrowser(backend, url_policy=url_policy)
  ```

  ```typescript TypeScript
  import { type BetaURLContext, ToolError } from "@anthropic-ai/sdk/helpers/beta/toolsets";

  const ALLOWED_HOSTS = ["example.com", "iana.org"];
  const SCHEME_PREFIX = /^[a-z][a-z0-9+.-]*:/i;

  // 仅为示例，并非生产环境策略。
  function isAllowed(url: string): boolean {
    if (url.toLowerCase() === "about:blank") {
      return true;
    }
    const withScheme = SCHEME_PREFIX.test(url) ? url : `https://${url}`;
    // 浏览器会将 Web URL 中的 "\" 视为 "/"。
    const parsed = URL.parse(withScheme.replaceAll("\\", "/"));
    const host = parsed?.hostname ?? "";
    const listed = ALLOWED_HOSTS.some((name) => host === name || host.endsWith(`.${name}`));
    return (parsed?.protocol === "http:" || parsed?.protocol === "https:") && listed;
  }

  function urlPolicy(ctx: BetaURLContext, url: string): void {
    if (!isAllowed(url)) {
      throw new ToolError(`blocked: ${url} is not on an allowed host`);
    }
  }

  const browser = new MyBrowser(backend, { urlPolicy });
  ```
</CodeGroup>

SDK 只对 `navigate` 调用该策略，而不会对 `"back"`、`"forward"` 或 `"reload"` 调用。Claude 可能会省略协议，例如 `example.com`，因此在读取主机之前，示例会为不以协议开头的 URL 添加 `https://`。您的 `navigate` 收到的是 Claude 编写的原始 URL。请在那里使用相同的判断添加协议，以便浏览器打开的正是您的策略检查过的 URL。

该示例并非生产级别。生产策略需要更多内容，涉及以下方面：

* Claude 点击的链接、提交的表单、重定向以及页面自行发出的请求。只有您的驱动程序能看到它们。请参阅[在驱动程序中拦截请求](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#intercept-requests-in-the-driver)。
* 来自页面内容的主机名和 URL，攻击者可以选择这些内容。
* 按照浏览器的方式解析每个 URL，而不是通过简单的字符串拆分来查找主机名。
* URL 协议。SDK 不会检查它们，因此请拒绝所有您不打算让驱动程序打开的协议。
* 容器上的出口规则，用于处理任何 URL 检查都无法看到的内容。请参阅[在容器上设置出口策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#put-egress-policy-on-the-container)。

如果您传入 `url_policy=None` (typescript: `urlPolicy: null`)，SDK 会拒绝每个导航到 URL 的 `navigate`，因此从您的配置中读取到的 `None` (typescript: `null`) 不会意外地让 URL 不受检查。在 TypeScript 中，`undefined` 会使该选项保持未设置状态，与不传入它相同。

### 在驱动程序中拦截请求

URL 策略仅检查 `navigate` 调用中的 URL。Claude 点击的链接、提交的表单、重定向，或者允许页面上的图像或脚本，仍然可能访问策略会拒绝的主机。请使用[出口规则](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#put-egress-policy-on-the-container)来阻止这些主机。

支持请求拦截的驱动程序还可以在其请求钩子中检查这些请求，请求钩子是您的自动化库在每个请求之前调用的函数。调用您的策略所使用的函数，这样规则只需编写一次：

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  class MyBrowser(BetaAbstractBrowserToolset20260801):
      ...

      # 通过 context.route("**/*", self._guard) 注册到 Playwright 浏览器上下文
      def _guard(self, route):
          # is_allowed 是“编写 URL 策略”一节中的函数
          if not is_allowed(route.request.url):
              return route.abort("blockedbyclient")
          route.continue_()
  ```

  ```typescript TypeScript
  // 在驱动类中，作用于 Playwright 浏览器上下文。
  // isAllowed 是“编写 URL 策略”一节中的函数。
  await context.route("**/*", (route) =>
    isAllowed(route.request().url()) ? route.continue() : route.abort("blockedbyclient")
  );
  ```
</CodeGroup>

像这样的请求钩子看不到 WebSocket 握手、service worker 请求或重定向跳转。请阻止 service worker。使用能够看到 WebSocket 握手和重定向跳转的钩子来检查它们。示例策略会拒绝 `ws://` 和 `wss://` URL，因此在将 WebSocket URL 传递给 `is_allowed()` (typescript: `isAllowed()`) 之前，请将 `ws://` 改为 `http://`，将 `wss://` 改为 `https://`。

### 在容器上设置出站策略

拦截无法看到浏览器发出的每个请求，而 URL 策略看不到名称解析到的位置。由容器网络强制执行的 “egress policy”（出站策略）可以同时覆盖这两种情况：

* 阻止 IPv4 和 IPv6 中的链路本地地址段和私有地址段。这包括云元数据地址 `169.254.169.254`。
* 只允许到任务所需主机的出站连接。如果您的规则匹配 IP 地址，请在容器启动时解析您允许的主机名。
* 只允许向容器的解析器发送 DNS 请求。

在浏览器容器外部执行的规则（例如 Kubernetes NetworkPolicy 或云防火墙）看不到容器内部的回环地址。如果只有这些规则，页面就可以访问在那里监听的任何服务，包括 DevTools 端口，该端口会显示浏览器已打开标签页的列表以及控制每个标签页的地址。

### 限制上传和下载

`file_upload` 默认处于关闭状态。如果没有 `file_policy` (typescript: `filePolicy`)，SDK 会拒绝每个提供路径或[文档 ID](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool#upload-files) 的上传。要启用上传，请传入一个 `BetaLocalFilePolicy` (typescript: `BetaNodeFilePolicy`)，并指定一个只包含任务文件的上传目录：

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  from anthropic.tools.browser import BetaLocalFilePolicy

  # 同时实现了 file_upload 的 MyBrowser
  browser = MyBrowser(
      backend,
      configs={"file_upload": {"enabled": True}},
      confirm=make_confirm(),  # required for file_upload; see Gate consequential members
      url_policy=url_policy,
      file_policy=BetaLocalFilePolicy(
          upload_roots=["/task/uploads"],
          download_dir="/task/downloads",
          expose_download_paths=False,
      ),
  )
  ```

  ```typescript TypeScript
  import { BetaNodeFilePolicy } from "@anthropic-ai/sdk/helpers/beta/toolsets/node";

  // 同时实现了 file_upload 的 MyBrowser
  const browser = new MyBrowser(backend, {
    configs: { file_upload: { enabled: true } },
    confirm: makeConfirm(), // required for file_upload; see Gate consequential members
    urlPolicy,
    filePolicy: new BetaNodeFilePolicy({
      uploadRoots: ["/task/uploads"],
      downloadDir: "/task/downloads",
      exposeDownloadPaths: false
    })
  });
  ```
</CodeGroup>

SDK 会解析每个上传路径（跟随符号链接），并拒绝上传根目录之外的任何路径。如果下载目录是某个上传根目录、位于某个上传根目录内部，或包含某个上传根目录，文件策略会抛出配置错误。该检查可能会遗漏符号链接，因此不要让符号链接使下载目录与上传根目录相同，或使其中一个位于另一个内部。只有当 `expose_download_paths` (typescript: `exposeDownloadPaths`) 为 true 且文件位于下载目录内时，SDK 才会在状态报告中包含下载的 `path`。

随附的文件策略检查的是运行 SDK 的进程所在文件系统上的路径。它只保护与该文件系统共享的浏览器。对于远程浏览器，请改为遵循[远程和托管浏览器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#remote-and-hosted-browsers)。

请按以下方式设置下载：

* 自行创建模式为 `0700` 的下载目录，并以 `noexec,nosuid,nodev` 方式挂载。
* 确保 Claude 可以调用的其他工具（例如 shell 或文件工具）无法访问该目录。
* 在 `download_failed` 状态变更中，将 `error` 写为固定短语。异常的文本可能包含路径或 URL。
* 在有人做出决定之前，不要将下载的文件读入对话，也不要运行它。

### 对有重大影响的成员进行把关

`javascript_exec` 和 `file_upload` 默认关闭。如果您在没有 `confirm` 可调用对象的情况下启用其中任何一个，构造函数会抛出配置错误。有 `confirm` 可调用对象时，SDK 会在每个即将运行的调用之前调用它。没有时，则不会询问任何内容。在 TypeScript 中，`confirm: null` 不会引发配置错误，而 SDK 会拒绝对任何成员的每个调用。

返回 `True` (typescript: `true`) 以运行该调用，或返回 `False` (typescript: `false`) 以拒绝它。当您的可调用对象询问某个人时，请向其展示成员、页面的 URL 以及调用的输入。首先，对 URL 和输入中所有可打印 ASCII 之外的字符进行转义，因为两者都可能携带来自页面的文本。

此示例通过您自己的 `ask_user()` (typescript: `askUser()`) 函数询问两个受控成员，并批准其余成员：

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  import json
  from collections.abc import Callable
  from urllib.parse import urlsplit

  from anthropic.tools.browser import BetaConfirmContext

  GATED = {"javascript_exec", "file_upload"}


  def shown(value: object) -> str:
      """A value as JSON, with every character outside printable ASCII escaped."""
      return json.dumps(value, ensure_ascii=True, indent=2)


  def make_confirm() -> Callable[[BetaConfirmContext], bool]:
      granted: set[tuple[str, str, str]] = set()

      def confirm(context: BetaConfirmContext) -> bool:
          name = context.member
          if name not in GATED:
              return True
          detail = shown(context.input.to_dict())
          page = context.tab_url
          where = shown(page) if page else "a tab with no URL"
          question = f"Allow {name} on {where}?\n{detail}"
          if page is None or urlsplit(page).scheme not in ("http", "https"):
              return ask_user(question)  # no web origin: ask every time
          # 一次批准仅适用于此页面上的这一确切输入。
          key = (name, page, detail)
          if key not in granted and ask_user(question):
              granted.add(key)
          return key in granted

      return confirm


  # 同时实现了 javascript_exec 和 file_upload 的 MyBrowser
  # file_upload 还需要“限制上传和下载”中的 file_policy
  browser = MyBrowser(
      backend,
      configs={"javascript_exec": {"enabled": True}, "file_upload": {"enabled": True}},
      confirm=make_confirm(),
      url_policy=url_policy,
  )
  ```

  ```typescript TypeScript
  import type { BetaConfirmContext } from "@anthropic-ai/sdk/helpers/beta/toolsets";

  const GATED = new Set(["javascript_exec", "file_upload"]);

  /** A value as JSON, with every character outside printable ASCII escaped. */
  const shown = (value: unknown): string =>
    JSON.stringify(value, null, 2).replace(
      /[^\n\x20-\x7e]/g,
      (match) => `\\u${match.charCodeAt(0).toString(16).padStart(4, "0")}`
    );

  const makeConfirm = () => {
    const granted = new Set<string>();
    return async (ctx: BetaConfirmContext): Promise<boolean> => {
      const name = ctx.member;
      if (!GATED.has(name)) {
        return true;
      }
      const detail = shown(ctx.input);
      const page = ctx.tabURL;
      const where = page ? shown(page) : "a tab with no URL";
      const question = `Allow ${name} on ${where}?\n${detail}`;
      const scheme = page === undefined ? undefined : URL.parse(page)?.protocol;
      if (scheme !== "http:" && scheme !== "https:") {
        return askUser(question); // no web origin: ask every time
      }
      // 一次批准仅适用于此页面上的这一确切输入。
      const key = `${name} ${page} ${detail}`;
      if (!granted.has(key) && (await askUser(question))) {
        granted.add(key);
      }
      return granted.has(key);
    };
  };

  // 同时实现了 javascript_exec 和 file_upload 的 MyBrowser
  // file_upload 还需要“限制上传和下载”中的 filePolicy
  const browser = new MyBrowser(backend, {
    configs: { javascript_exec: { enabled: true }, file_upload: { enabled: true } },
    confirm: makeConfirm(),
    urlPolicy
  });
  ```
</CodeGroup>

每次调用 `make_confirm()` (typescript: `makeConfirm()`) 都会返回一个没有任何批准记录的可调用对象。为每个工具集调用一次，并为每个用户提供各自的工具集。

批准仅涵盖上一次状态报告所显示的页面，而页面可能在调用运行之前发生变化。购买、发送消息和接受条款都是通过 `left_click` 和 `type` 等普通成员完成的，因此 `confirm` 无法按名称将它们挑选出来。如果要让人员批准这些操作，也请询问这些成员。

除非[出站规则](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#put-egress-policy-on-the-container)将浏览器的出站连接限制在任务所需的主机上，否则不要启用 `javascript_exec`。脚本可以将页面内容发送到浏览器能够访问的任何地址。

`navigate` 调用中的 `javascript:` URL 也会在当前页面中运行脚本，即使 `javascript_exec` 处于关闭状态。`confirm` 将该调用视为 `navigate`，因此示例中的 `confirm` 可调用对象会批准它。您的驱动程序必须拒绝 `javascript:` URL。在示例中，[URL 策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#write-a-url-policy)会在 `confirm` 运行之前拒绝它们。

### 隔离浏览器主机

为每个会话在专用的、最小权限的容器或虚拟机中运行浏览器：

* 以非 root 用户身份运行，并在浏览器允许的情况下使用只读根文件系统。
* 除了您配置的上传和下载目录（如果有）之外，不要从主机挂载任何内容。
* 不要在环境中放置凭据，并从全新的浏览器配置文件开始。
* 不与 Claude 可以调用的其他工具共享任何文件系统。

<Warning>
  浏览器配置文件不得登录任何您不愿将其数据或操作交给 Claude 的账户。`javascript_exec` 以页面自身的权限运行：包括其 cookie、存储以及已登录的会话。
</Warning>

在浏览器容器之外运行调用 API 的代码，因为该代码持有您的 API 密钥和对话内容。工具运行器和 `tool_result` (typescript: `toolResult`) 都在该代码的进程中运行工具集，因此浏览器不与工具集共享文件系统。请将浏览器视为远程浏览器：[远程和托管浏览器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#remote-and-hosted-browsers)适用。

将页面返回的所有内容都视为不可信内容，包括页面文本、屏幕截图、控制台和网络条目、标签页标题以及下载名称。

### 远程和托管浏览器

有些浏览器不与运行 SDK 的进程共享文件系统。例如，位于另一个容器中的浏览器、通过 DevTools URL 访问的浏览器，以及来自托管浏览器服务的浏览器。对于这些浏览器，URL 策略、请求拦截和 `confirm` 仍在您的进程中运行。

对于托管浏览器，由提供商控制出站流量和主机隔离。您自己的出站策略在那里不适用，因此驱动程序的请求钩子是您对浏览器请求的唯一检查。请了解浏览器的网络可以访问哪些内容。

随附的路径检查无法保护远程浏览器。`BetaLocalFilePolicy` (typescript: `BetaNodeFilePolicy`) 检查的是运行 SDK 的进程所在文件系统上的路径，而浏览器读写的是它自己的文件系统。

SDK 无法检测浏览器是否为远程浏览器。因此，对于远程浏览器，请保持 `file_upload` 关闭，除非您的驱动程序在浏览器运行的位置使用其自己的 `BetaFilePolicy` 检查上传路径。`BetaFilePolicy` 会审查每次上传的路径和文档 ID，并决定 Claude 是否能看到下载的路径。

让远程浏览器拒绝下载，除非其自身主机具有[限制上传和下载](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#confine-uploads-and-downloads)中所述的下载设置。

不要让提供商的 API 密钥和会话的连接 URL（其中可能包含密钥）出现在日志、工具结果和错误文本中。如果提供商会录制会话，则录制内容是 Claude 所见和所输入的一切内容的另一份副本，并且适用提供商的保留条款。

## 计算机工具集

`BetaAbstractComputerToolset20260801` 是用于[计算机使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)的类。驱动程序是一个子类，每个工具对应一个方法，例如 `screenshot` 或 `left_click`。可以从[合作伙伴集成](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#partner-integrations)下的合作伙伴驱动程序开始。如果要编写自己的驱动程序，claude-quickstarts 仓库提供了 Python 和 TypeScript 版本的[最小 VNC 示例](https://github.com/anthropics/claude-quickstarts/tree/main/computer-toolset)。它通过 VNC 控制桌面，但不是生产代码。

该类接受 `configs`、`confirm` 和 `tool_configs` (typescript: `toolConfigs`)，传入方式与浏览器类相同。[自定义驱动程序](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#customize-a-driver)、[处理错误](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#handle-errors)、[在响应流式传输时启动调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#start-calls-while-the-response-streams)和[不使用工具运行器运行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#run-without-the-tool-runner)同样适用于它，但有以下不同之处：

* **仅限浏览器的选项：**&#x8BE5;类没有 [URL 策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#write-a-url-policy)、[文件策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#confine-uploads-and-downloads)和[状态报告](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#report-browser-state)。
* **`configs`：**&#x60A8;实现的每个工具默认都是开启的。每个工具的设置列在[工具参数](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#tool-parameters)下。
* **`confirm`：**&#x5982;果您的类实现了 `type`、`key` 或 `hold_key`，则除非您传递 `confirm` 可调用对象或使用 `configs` 关闭这些工具，否则构造函数会引发配置错误。重写 `execute` 的类被视为实现了全部三个工具。
* **跳过的调用：**&#x5728;计算机调用失败后，工具运行器会用 `Not executed: an earlier computer action in this turn failed.` 响应本轮中后续的计算机调用。浏览器调用会收到不同的文本。在您自己编写的循环中，请使用完全相同的文本响应被跳过的计算机调用（[计算机使用工具中的批量操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#batch-actions)）。
* **`execute`：**&#x5728; TypeScript 中，`execute` 重写会将 `name` 的类型设为 `BetaComputerMemberName`，将 `input` 的类型设为 `BetaComputerMemberInput`。两者都来自 `@anthropic-ai/sdk/resources/beta`。
* **错误文本：**&#x53;DK 不会隐藏计算机工具错误文本或工具返回文本中的本地路径。Claude 会读取这两者，截断为 4,096 个字符。请在您的工具中捕获异常，并引发 `ToolError`（在 TypeScript 中为抛出）并附上您自己的文本。

### 编写桌面驱动程序

此驱动程序实现了[可用操作](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#available-actions)下列出的 17 个工具中的 5 个。您未实现的工具会以禁用状态发送到 API，如[实现驱动程序](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#implement-a-driver)中所述。在示例中，`display` 代表您自己对控制桌面的任何工具（例如 VNC 客户端或 `xdotool`）的封装，`ask_user()` (typescript: `askUser()`) 代表您自己的询问人员的函数。

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  from anthropic import Anthropic
  from anthropic.tools.computer import (
      BetaAbstractComputerToolset20260801,
      BetaComputerCursorPositionResult,
      BetaScreenshotResult,
      BetaToolsetCallContext,
  )
  from anthropic.types.beta import (
      BetaComputerCursorPositionInput,
      BetaComputerKeyInput,
      BetaComputerLeftClickInput,
      BetaComputerScreenshotInput,
      BetaComputerTypeInput,
  )


  class MyDesktop(BetaAbstractComputerToolset20260801):
      def __init__(self, display, **options):
          super().__init__(**options)
          self.display = display

      def screenshot(
          self, context: BetaToolsetCallContext, input: BetaComputerScreenshotInput
      ) -> BetaScreenshotResult:
          # Claude 的坐标是此图像中的像素位置。
          data = self.display.png_base64()
          return BetaScreenshotResult(data=data, media_type="image/png")

      def cursor_position(
          self, context: BetaToolsetCallContext, input: BetaComputerCursorPositionInput
      ) -> BetaComputerCursorPositionResult:
          x, y = self.display.cursor()
          return BetaComputerCursorPositionResult(x=x, y=y)

      def left_click(
          self, context: BetaToolsetCallContext, input: BetaComputerLeftClickInput
      ) -> None:
          # input.coordinate 为 [x, y]，或为 None 表示在光标处点击。input.text 是
          # 要按住的修饰键，例如 "shift"。无需返回内容：Claude 读取 "Clicked."
          self.display.click(input.coordinate, input.text)

      def type(
          self, context: BetaToolsetCallContext, input: BetaComputerTypeInput
      ) -> None:
          self.display.type(input.text)

      def key(self, context: BetaToolsetCallContext, input: BetaComputerKeyInput) -> None:
          self.display.press(input.text, input.repeat or 1)

      def close(self) -> None:
          super().close()  # first, so no call is still using the desktop when it closes
          self.display.close()


  client = Anthropic()
  with MyDesktop(
      display,
      # type 和 key 操作必需；请参阅“安全运行计算机工具集”
      confirm=lambda context: ask_user(f'Allow the "{context.member}" tool?'),
  ) as desktop:
      runner = client.beta.messages.tool_runner(
          model="claude-opus-5-5",
          max_tokens=1024,
          tools=[desktop],
          messages=[
              {"role": "user", "content": "Open the calculator and compute 17 * 23."}
          ],
      )
      for message in runner:
          print(message)
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";
  import {
    BetaAbstractComputerToolset20260801,
    type BetaComputerCursorPositionResult,
    type BetaComputerToolsetOptions,
    type BetaScreenshotResult,
    type BetaToolsetCallContext
  } from "@anthropic-ai/sdk/helpers/beta/toolsets";
  import type {
    BetaComputerCursorPositionInput,
    BetaComputerKeyInput,
    BetaComputerLeftClickInput,
    BetaComputerScreenshotInput,
    BetaComputerTypeInput
  } from "@anthropic-ai/sdk/resources/beta";

  class MyDesktop extends BetaAbstractComputerToolset20260801 {
    constructor(
      private display: Display,
      options: BetaComputerToolsetOptions = {}
    ) {
      super(options);
    }

    protected override async screenshot(
      ctx: BetaToolsetCallContext,
      input: BetaComputerScreenshotInput
    ): Promise<BetaScreenshotResult> {
      // Claude 的坐标是此图像中的像素位置。
      return { data: await this.display.pngBase64(), mediaType: "image/png" };
    }

    protected override async cursor_position(
      ctx: BetaToolsetCallContext,
      input: BetaComputerCursorPositionInput
    ): Promise<BetaComputerCursorPositionResult> {
      const { x, y } = await this.display.cursor();
      return { x, y };
    }

    protected override async left_click(
      ctx: BetaToolsetCallContext,
      input: BetaComputerLeftClickInput
    ): Promise<void> {
      // input.coordinate 为 [x, y]，或为 null 或 undefined 以在光标处点击。input.text
      // 为要按住的修饰键，例如 "shift"。无需返回内容：Claude 会读取到 "Clicked."
      await this.display.click(input.coordinate, input.text);
    }

    // type 工具在 TypeScript 中写作 type_。
    protected override async type_(
      ctx: BetaToolsetCallContext,
      input: BetaComputerTypeInput
    ): Promise<void> {
      await this.display.type(input.text);
    }

    protected override async key(
      ctx: BetaToolsetCallContext,
      input: BetaComputerKeyInput
    ): Promise<void> {
      await this.display.press(input.text, input.repeat ?? 1);
    }

    override async close(): Promise<void> {
      await super.close(); // first, so no call is still using the desktop when it closes
      await this.display.close();
    }
  }

  const client = new Anthropic();
  const desktop = new MyDesktop(display, {
    // type 和 key 必需；请参阅“安全地运行计算机工具集”
    confirm: (ctx) => askUser(`Allow the "${ctx.member}" tool?`)
  });
  try {
    const runner = client.beta.messages.toolRunner({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      tools: [desktop],
      messages: [{ role: "user", content: "Open the calculator and compute 17 * 23." }]
    });
    for await (const message of runner) {
      console.log(message);
    }
  } finally {
    await desktop.close();
  }
  ```
</CodeGroup>

<Warning>
  此示例中的 `confirm` 会在驱动程序运行的每个调用之前询问某个人。在针对一次性机器以外的任何环境运行该驱动程序之前，请阅读[安全地运行计算机工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#run-the-computer-toolset-safely)。
</Warning>

有四个工具没有输入字段：`screenshot`、`cursor_position`、`left_mouse_down` 和 `left_mouse_up`。每种输入类型（包括它们的）都来自 `anthropic.types.beta` (typescript: `@anthropic-ai/sdk/resources/beta`)。

在 TypeScript 中，请将工具写成方法，而不是箭头函数字段。[实现驱动程序](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#implement-a-driver)给出了原因。

### 缩放坐标并调整图像尺寸

Claude 发送的每个点都是您返回的屏幕截图中的像素位置：每个 `coordinate` 和 `start_coordinate`，以及 `zoom` `region` 的两个角。SDK 会将它们原样传递给您的工具。

如果您的屏幕截图尺寸与显示器不同，请在驱动程序中进行双向转换：将 Claude 的点转换为显示器像素，并将您报告的光标位置转换为屏幕截图像素。在整个会话中保持屏幕截图尺寸不变，并使其与显示器的宽高比一致：

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  from anthropic.tools import ToolError

  SCREENSHOT_SIZE = (1280, 720)  # the size of every full screenshot the driver returns
  DISPLAY_SIZE = (2560, 1440)


  def to_display(coordinate: list[int]) -> tuple[int, int]:
      x, y = coordinate
      width, height = SCREENSHOT_SIZE
      if not (0 <= x < width and 0 <= y < height):
          # 拒绝该坐标点。若将其移入范围内，点击位置将并非 Claude 所选。
          raise ToolError(f"[{x}, {y}] is outside the {width}x{height} screenshot")
      return x * DISPLAY_SIZE[0] // width, y * DISPLAY_SIZE[1] // height


  def to_screenshot(x: int, y: int) -> tuple[int, int]:
      width, height = DISPLAY_SIZE
      return x * SCREENSHOT_SIZE[0] // width, y * SCREENSHOT_SIZE[1] // height


  to_display([640, 360])  # (1280, 720)
  ```

  ```typescript TypeScript
  import { ToolError } from "@anthropic-ai/sdk/helpers/beta/toolsets";

  const SCREENSHOT_SIZE = { width: 1280, height: 720 }; // the size of every full screenshot
  const DISPLAY_SIZE = { width: 2560, height: 1440 };

  function toDisplay(coordinate: number[]): [number, number] {
    const [x, y] = coordinate;
    const { width, height } = SCREENSHOT_SIZE;
    if (x < 0 || x >= width || y < 0 || y >= height) {
      // 拒绝该坐标点。若将其移入范围内，点击位置将不是 Claude 所选的位置。
      throw new ToolError(`[${x}, ${y}] is outside the ${width}x${height} screenshot`);
    }
    return [
      Math.floor((x * DISPLAY_SIZE.width) / width),
      Math.floor((y * DISPLAY_SIZE.height) / height)
    ];
  }

  function toScreenshot(x: number, y: number): [number, number] {
    return [
      Math.floor((x * SCREENSHOT_SIZE.width) / DISPLAY_SIZE.width),
      Math.floor((y * SCREENSHOT_SIZE.height) / DISPLAY_SIZE.height)
    ];
  }

  toDisplay([640, 360]); // [1280, 720]
  ```
</CodeGroup>

在对每个 `coordinate` 和 `start_coordinate` 执行操作之前，对其调用 `to_display()` (typescript: `toDisplay()`)，并对 `cursor_position` 报告的位置调用 `to_screenshot()` (typescript: `toScreenshot()`)。没有 `coordinate` 的点击或滚动会在光标处执行，因此无需转换。请按相同的比例缩放 `zoom` 的 `region`。`zoom` 图像不会改变这一点：Claude 接下来发送的点仍然是完整截图中的像素位置。

SDK 从不读取或调整图像尺寸。API 会拒绝超出模型限制的图像，这会终止运行。请在驱动程序中缩放每张屏幕截图和 `zoom` 图像。请参阅[调整截图大小以符合图像限制](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#handle-coordinate-scaling-for-higher-resolutions)。在长时间运行中，一个请求可能包含超过 20 张图像，此时 API 会对每张图像施加更严格的限制。请参阅[管理截图历史](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#manage-screenshot-history)。

### 输入检查与结果

不要依赖 SDK 来检查调用的输入。让您的驱动程序检查它使用的字段，并拒绝无法执行的调用：抛出 `ToolError`（在 TypeScript 中使用 throw）。Claude 会读取该消息，并可以发送更正后的调用。

SDK 不会限制 `duration`、`repeat` 次数或 `scroll_amount`。[安全运行计算机工具集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#run-the-computer-toolset-safely)下的第 4 步涵盖了这些内容。

工具返回的内容决定了 Claude 读取的内容：

| 工具                  | 返回                                 | Claude 读取                          |
| ------------------- | ---------------------------------- | ---------------------------------- |
| `screenshot`、`zoom` | `BetaScreenshotResult`             | 一个图像块                              |
| `cursor_position`   | `BetaComputerCursorPositionResult` | `X={x},Y={y}`                      |
| 其他所有工具              | 无，或一行文本                            | 一条简短确认，例如 `Clicked.`，然后是单独文本块中的返回行 |

### 安全地运行计算机工具集

屏幕上的内容会引导 Claude 的下一步操作。页面或通知可能会试图让 Claude 在错误的窗口中输入内容，或触发具有实际影响的操作。

<Warning>
  实现了 `type`、`key` 或 `hold_key` 的驱动程序需要一个 `confirm` 可调用对象。如果没有，构造函数会抛出配置错误，除非 `configs` 关闭了这些工具。SDK 本身不会询问任何人：它会运行您的 `confirm` 批准的每个调用；如果没有 `confirm`，它会运行每个调用，包括每次点击。您实现的每个计算机工具默认都是启用的。
</Warning>

在将驱动程序用于一次性机器以外的任何环境之前，请采取以下五个步骤：

1. 隔离桌面。为每个会话在专用的、最小权限的容器或虚拟机中运行它，不包含凭据，也不挂载主机目录。只安装任务所需的应用程序。在其外部运行调用 API 的代码。
2. 使用 `confirm` 对具有实际影响的操作进行把关。购买或发送消息只是普通的 `left_click`、`type` 或 `key`，而 `confirm` 接收的是工具及其输入，而不是屏幕。因此，请根据 Claude 在桌面上可以访问的应用程序来选择哪些工具需要批准。
3. 除非任务需要，否则不要在桌面上放置终端、运行对话框和启动器。当其中之一获得键盘焦点时，`type`、`key` 和 `hold_key` 会运行 Claude 输入的任何内容，而一次点击就可能启动程序。如果任务需要其中之一，请在键盘输入和点击之前询问人员，并隔离桌面，使命令的影响范围不超出任务所需。
4. 拒绝桌面不应执行的输入，例如屏幕截图之外的点、长达数分钟的 `wait`，或非常大的 `repeat` 或 `scroll_amount`。抛出 `ToolError`（在 TypeScript 中使用 throw），而不是将值调整到范围内。
5. 将屏幕上的所有内容视为不可信，包括窗口标题和剪贴板文本。不要在未经检查的情况下运行或传递它们。

SDK 通过您传递的可调用对象应用第 2 步。其他步骤取决于您的驱动程序和部署。[计算机使用工具的安全注意事项](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool#security-considerations)下的预防措施同样适用。

以下 `confirm` 会在任何键盘输入之前通过您自己的 `ask_user()` (typescript: `askUser()`) 函数询问人员，并批准其他所有调用。`context.member` (typescript: `ctx.member`) 是工具的名称。如果也要让人员批准点击操作，请将 `ASK_FIRST` 设置为 `KEYBOARD | CLICKS` (typescript: `new Set([...KEYBOARD, ...CLICKS])`)。

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  from anthropic.tools.computer import BetaComputerConfirmContext

  KEYBOARD = {"type", "key", "hold_key"}
  CLICKS = {
      "left_click",
      "right_click",
      "middle_click",
      "double_click",
      "triple_click",
      "left_click_drag",
      "left_mouse_down",
      "left_mouse_up",
  }
  ASK_FIRST = KEYBOARD


  def confirm(context: BetaComputerConfirmContext) -> bool:
      if context.member not in ASK_FIRST:
          return True
      # 此处所示为“对有重大影响的成员进行把关”中的函数。
      detail = shown(context.input.to_dict())
      return ask_user(f'Allow the "{context.member}" tool with this input?\n{detail}')


  desktop = MyDesktop(display, confirm=confirm)
  ```

  ```typescript TypeScript
  import type { BetaComputerConfirmContext } from "@anthropic-ai/sdk/helpers/beta/toolsets";

  const KEYBOARD = ["type", "key", "hold_key"];
  const CLICKS = [
    "left_click",
    "right_click",
    "middle_click",
    "double_click",
    "triple_click",
    "left_click_drag",
    "left_mouse_down",
    "left_mouse_up"
  ];
  const ASK_FIRST = new Set(KEYBOARD);

  async function confirm(ctx: BetaComputerConfirmContext): Promise<boolean> {
    if (!ASK_FIRST.has(ctx.member)) {
      return true;
    }
    // 此处所示为“对有重大影响的成员进行把关”中的函数。
    return askUser(`Allow the "${ctx.member}" tool with this input?\n${shown(ctx.input)}`);
  }

  const desktop = new MyDesktop(display, { confirm });
  ```
</CodeGroup>

被拒绝的调用会得到一个错误结果，运行会继续。对于被拒绝的 `type` 调用，Claude 会读取 `The user did not grant permission to run 'type'. Do not retry it unless the user asks you to.`

此 `confirm` 会批准所有不在 `ASK_FIRST` 中的工具，包括您以后添加的任何工具。若要采用失败即关闭（fail closed）的方式，请只批准对桌面没有影响的工具（`screenshot`、`zoom`、`cursor_position` 和 `wait`），并对其他所有工具进行询问。

### 在一个请求中使用两个工具集

在 `tools` 中传入两个实例。每个调用都带有其 `toolset_name`（`browser` 或 `computer`），工具运行器会将调用发送到匹配的实例。调用失败后，工具运行器只会跳过该轮中对同一工具集的后续调用，因此失败的计算机调用不会跳过该轮的浏览器调用。

<CodeGroup exclude="shell:cURL, shell:CLI, csharp, go, java, php, ruby">
  ```python Python
  # MyBrowser、backend 和 url_policy 来自高级示例。
  # confirm 是“安全运行计算机工具集”中的函数。
  with (
      MyBrowser(backend, url_policy=url_policy) as browser,
      MyDesktop(display, confirm=confirm) as desktop,
  ):
      runner = client.beta.messages.tool_runner(
          model="claude-opus-5-5",
          max_tokens=1024,
          tools=[browser, desktop],
          messages=[
              {
                  "role": "user",
                  "content": "Copy the heading on example.com into the open text editor.",
              }
          ],
          stream=True,
          run_tools_eagerly=True,  # so a call can start before the response ends
      )
      for stream in runner:
          print(stream.get_final_message())
  ```

  ```typescript TypeScript
  // MyBrowser、backend 和 urlPolicy 来自高级示例。
  // confirm 是“安全地运行计算机工具集”中的函数。
  const browser = new MyBrowser(backend, { urlPolicy });
  const desktop = new MyDesktop(display, { confirm });
  try {
    const runner = client.beta.messages.toolRunner({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      tools: [browser, desktop],
      messages: [
        { role: "user", content: "Copy the heading on example.com into the open text editor." }
      ],
      stream: true,
      runToolsEagerly: true // so a call can start before the response ends
    });
    for await (const stream of runner) {
      console.log(await stream.finalMessage());
    }
  } finally {
    await Promise.all([browser.close(), desktop.close()]);
  }
  ```
</CodeGroup>

在您自己编写的循环中，使用 `toolset_name` (typescript: `toolsetName`) 与调用的 `toolset_name` 匹配的实例上的 `tool_result` (typescript: `toolResult`) 来响应每个调用。在每一轮中，分别跟踪每个工具集的失败调用，并使用该工具集自己的文本来响应被跳过的调用。[不使用工具运行器运行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#run-without-the-tool-runner)中提供了浏览器的文本，本节顶部的"被跳过的调用"要点中提供了计算机的文本。

## 参考

构造函数选项在两个 SDK 中的含义相同：

| Python              | TypeScript     | 设置                                          |
| ------------------- | -------------- | ------------------------------------------- |
| `configs`           | `configs`      | 启用哪些成员                                      |
| `confirm`           | `confirm`      | 批准或拒绝每个调用的可调用对象                             |
| `url_policy`        | `urlPolicy`    | 您的 URL 策略，SDK 会在每次 `navigate` 到某个 URL 之前调用它 |
| `file_policy`       | `filePolicy`   | 上传根目录和下载路径暴露                                |
| `tool_configs`      | `toolConfigs`  | `tools` 条目的字段，例如 `cache_control`            |
| `_browser_state` 方法 | `browserState` | 状态报告                                        |

浏览器类接受表中的所有选项。计算机类接受 `configs`、`confirm` 和 `tool_configs` (typescript: `toolConfigs`)。

构造之后无法更改选项。[Python 浏览器工具集指南](https://github.com/anthropics/anthropic-sdk-python/blob/main/browser-toolset.md)、[TypeScript 浏览器工具集指南](https://github.com/anthropics/anthropic-sdk-typescript/blob/main/browser-toolset.md)、[Python 计算机工具集指南](https://github.com/anthropics/anthropic-sdk-python/blob/main/computer-toolset.md)和 [TypeScript 计算机工具集指南](https://github.com/anthropics/anthropic-sdk-typescript/blob/main/computer-toolset.md)介绍了每个选项。Python 指南还涵盖了异步类 `BetaAsyncAbstractBrowserToolset20260801` 和 `BetaAsyncAbstractComputerToolset20260801`。

## 限制

* **SDK 不包含浏览器、桌面、现成的驱动程序或 URL 策略：** 这些由您提供。请参阅[合作伙伴集成](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#partner-integrations)、[实现驱动程序](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#implement-a-driver)和[编写 URL 策略](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#write-a-url-policy)。
* **URL 策略只能看到 `navigate` 调用：**&#x5B83;看不到页面发出的请求，也看不到以其他方式打开的页面，例如通过 Claude 点击的链接或重定向打开的页面。请参阅[在驱动程序中拦截请求](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#intercept-requests-in-the-driver)。
* **批准基于上一次状态报告：** 页面可能在该报告之后发生变化。SDK 不会在调用运行之前再次检查页面。
* **同一工具集上的调用一次只运行一个：** 您无法关闭此行为。
* **SDK 不检查标签页 ID 是否唯一或标签页数量，也不总是检查是否只有一个标签页处于活动状态：**&#x41;PI 会拒绝违反这些规则的报告。
* **对于计算机调用，`confirm` 接收的是工具及其输入，而不是屏幕：**&#x60A8;无法从中判断在给定点的点击会产生什么效果。
* **SDK 不会调整计算机工具集图像的尺寸：**&#x41;PI 会拒绝超出模型限制的图像。请参阅[缩放坐标并调整图像尺寸](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-sdk#scale-coordinates-and-size-images)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="浏览器使用工具" icon="browser" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool">
    成员工具、`browser_state` 块以及该工具的安全注意事项。
  </Card>

  <Card title="计算机使用工具" icon="computer" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool">
    17 个工具、坐标和图像限制，以及该工具的安全注意事项。
  </Card>

  <Card title="工具运行器（SDK）" icon="arrows-clockwise" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner">
    SDK 如何运行循环，以及如何更改它发送的消息。
  </Card>

  <Card title="缓解越狱和提示注入" icon="shield" href="https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks">
    适用于任何读取不可信内容的应用程序的防护措施。
  </Card>
</CardGroup>

## 可用性

浏览器和计算机使用 SDK 工具集在部分模型上处于 beta 阶段。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态   | 云平台                                                                                                                                                                                                                  | 模型                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)                                                  | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), Claude Mythos Preview, [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview) |
| Beta | [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)                        |
| Beta | [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)                                                                                                                  | [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Fable 5](https://platform.claude.com/docs/zh-CN/models/fable-5/overview), [Claude Mythos 5](https://platform.claude.com/docs/zh-CN/models/mythos-5/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Opus 5](https://platform.claude.com/docs/zh-CN/models/opus-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview), [Claude Opus 4.8](https://platform.claude.com/docs/zh-CN/models/opus-4-8/overview)                                                                                                                                                                                                      |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：不可用
