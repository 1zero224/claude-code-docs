---
title: 工具使用故障排查
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/troubleshooting-tool-use
description: 通过症状到修复的诊断表格，修复最常见的工具使用错误。
---

找到您遇到的症状或错误，然后应用为其列出的修复方法。

## Claude 调用了错误的工具

| 症状                       | 可能原因                | 修复方法                                                                                                                           |
| ------------------------ | ------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| 您想要工具 B，但 Claude 调用了工具 A | 描述含糊不清              | 使描述更加精确。通过"何时"使用工具来区分它们，而不仅仅是它们"做什么"。请参阅[定义工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools)。 |
| Claude 从不调用您的工具          | 工具名称冲突或 schema 过于笼统 | 检查工具列表中是否存在重复名称。添加 `input_examples` 以使预期用法更加具体。                                                                                |
| Claude 调用时使用了错误的参数类型     | schema 存在歧义         | 添加 `strict: true`（如果您的 schema 属于受支持的子集）或添加 `input_examples`。                                                                   |

## Claude 编造工具参数

| 症状                   | 可能原因           | 修复方法                                                                                                                         |
| -------------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| 出现了您的 schema 中不存在的参数 | 未启用严格模式时模型过度生成 | 如果您的 schema 属于[受支持的子集](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/strict-tool-use)，请添加 `strict: true`。 |
| 参数值超出您的枚举范围          | 缺少严格模式或枚举过大    | 缩小枚举范围，或添加展示有效选项的 `input_examples`。                                                                                          |

## 并行工具调用不起作用

| 症状                                | 可能原因      | 修复方法                                                                                                                                    |
| --------------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| 在并行更合适的情况下，Claude 却按顺序调用工具        | 消息历史格式问题  | 在一条用户消息中发送多个 `tool_result` 块，而不是每轮发送一个。请参阅[并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use)。 |
| `disable_parallel_tool_use` 似乎被忽略 | 在对话中设置得太晚 | 在返回 `tool_use` 的请求上设置它。在后续请求上设置它对之前的工具调用没有影响。                                                                                           |

## 缓存不断失效

| 症状             | 可能原因                                                 | 修复方法                                                                                                                                                                                                                                                                                                      |
| -------------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 每个请求都是缓存未命中    | `tool_choice`、思考配置或 `output_config.effort` 在请求之间发生变化 | 保持 `tool_choice` 稳定，或将 `cache_control` 断点放在变化点之前；在缓存对话的整个生命周期内保持思考配置和 effort 级别不变。请参阅[工具使用与提示缓存](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-use-with-prompt-caching)和[思考与提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking#thinking-and-prompt-caching)。 |
| 在对话中途添加工具会破坏缓存 | 工具被添加到 tools 数组的开头                                   | 从第一个请求起就使用 `defer_loading: true` 声明该工具，并让工具搜索以内联方式加载它，而不是稍后再将其添加到数组中。                                                                                                                                                                                                                                     |

## 请求时的错误

"错误"列显示每条消息的一部分，`...` 代表可变的文本。

| 错误                                                                     | 原因                                                               | 修复方法                                                                                                                                                                                                                                                             |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tool_use ids were found without tool_result blocks immediately after` | 某些 `tool_use` ID 缺少 `tool_result`，或 `tool_result` 不是用户消息中的第一个内容块 | 为助手响应中的每个 `tool_use` 块返回一个 `tool_result`。将 `tool_result` 块放在任何文本之前。请参阅[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)和[并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use)。 |
| `was found without a corresponding ... block`                          | 后续请求阻止了一个等待中的服务器工具运行                                             | 请参阅[服务器工具调用没有结果块](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/troubleshooting-tool-use#a-server-tool-call-has-no-result-block)。                                                                                                             |
| `Unsupported regex feature in pattern field: ...`                      | 严格工具的 `pattern` 使用了严格模式无法编译的正则表达式特性，例如反向引用、单词边界或较大的 `{n,m}` 范围   | 简化该模式。请参阅 [JSON Schema 限制](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#json-schema-limitations)。                                                                                                                                 |
| `Invalid regex in pattern field: ...`                                  | 严格工具的 `pattern` 包含先行断言、后行断言或语法错误                                 | 删除所有先行断言或后行断言，并修复所有语法错误。请参阅 [JSON Schema 限制](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#json-schema-limitations)。                                                                                                               |
| `All tools have defer_loading: true`                                   | 没有对模型可见的工具                                                       | 至少必须有一个工具被立即加载。工具搜索工具本身绝不能设置 `defer_loading: true`。                                                                                                                                                                                                              |

### 服务器工具调用没有结果块

当[服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/server-tools)的 `server_tool_use` 或 `mcp_tool_use` 块没有结果块时，该工具仍在等待运行。错误 `was found without a corresponding ... block` 表示后续请求阻止了该等待中的工具运行。错误消息和上一个 `stop_reason` 会说明原因。请采用第一个匹配的情况：

* **消息以 ``but no `<name>` tool was provided`` 或 `is not available after the last message` 结尾：** 此处的 `<name>` 是工具的名称，例如 `web_search`。请求不再定义该工具。请发送与之前相同的 `tools` 数组；对于 MCP 工具，还需发送相同的 `mcp_servers`。
* **上一个 `stop_reason` 是 `tool_use`：** 您的用户消息在 `tool_result` 块之后还有其他内容。请仅发送 `tool_result` 块，每个客户端 `tool_use` 块对应一个。
* **上一个 `stop_reason` 是 `pause_turn`：** 您添加了一条用户消息。请将其删除，使请求以作为 `assistant` 消息的已暂停响应结尾。

有关完整步骤，请参阅[当服务器工具仍在等待时](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#when-a-server-tool-is-still-waiting)和 [`pause_turn` 停止原因](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#pause-turn)。

## 错误：思考块无法修改

如果在工具调用后继续对话时，请求失败并返回 400 `invalid_request_error`，且其消息包含 `` `thinking` or `redacted_thinking` blocks in the latest assistant message cannot be modified ``，则说明您的应用程序在将助手的思考块发回之前对其进行了修改。请将整条助手消息原样发回，然后追加您的 `tool_result`。

请参阅[思考块无法修改](https://platform.claude.com/docs/zh-CN/api/errors#thinking-blocks-cannot-be-modified)了解完整的错误信息和修复步骤。

## Claude 将工具结果标记为提示注入

| 症状                                   | 可能原因                          | 修复方法                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------ | ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude 拒绝根据工具结果采取行动，或要求用户确认来自工具结果的指令 | 您自己的指令被放在 `tool_result` 内容中传递 | Claude 经过训练，会将工具结果中的指令视为可能不受信任的第三方内容。请将您的指令移出工具结果：在 `tool_result` 块之后的 `user` 轮次中发送它们，或者在受支持的模型上，通过[对话中途的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)发送。让工具结果仅包含数据。请参阅[缓解越狱和提示注入](https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks#indirect-prompt-injection)。 |

## 不同模型版本之间的 JSON 转义差异

| 症状                     | 原因                           | 修复方法                                                         |
| ---------------------- | ---------------------------- | ------------------------------------------------------------ |
| 在较新的模型上，对工具输入进行字符串比较失败 | 不同模型版本之间的 Unicode 和正斜杠转义方式不同 | 使用 `json.loads()` 或 `JSON.parse()` 进行解析。切勿对序列化后的输入进行原始字符串匹配。 |

## 后续步骤

<CardGroup cols={3}>
  <Card title="定义工具" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools">
    编写能够引导 Claude 选择正确工具的 schema 和描述。
  </Card>

  <Card title="处理工具调用" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls">
    运行工具并以所需的消息格式返回结果。
  </Card>

  <Card title="工具参考" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-reference">
    Anthropic 提供的工具及其版本字符串的完整目录。
  </Card>
</CardGroup>
