---
title: 工具使用故障排查
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/troubleshooting-tool-use
description: 通过症状到修复的诊断表格，修复最常见的工具使用错误。
---

针对最常见的 “tool use”（工具使用）错误的症状到修复对照表。每项修复都交叉引用了负责该功能的页面。

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

| 错误                                                                     | 原因                                                                                                                                                                             | 修复方法                                                                                                                                                                                                                                                             |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tool_use ids were found without tool_result blocks immediately after` | 某些 `tool_use` ID 缺少 `tool_result`，或 `tool_result` 不是用户消息中的第一个内容块                                                                                                               | 为助手响应中的每个 `tool_use` 块返回一个 `tool_result`。将 `tool_result` 块放在任何文本之前。请参阅[处理工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/handle-tool-calls)和[并行工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/parallel-tool-use)。 |
| `was found without a corresponding <name>_tool_result block`           | 上一个助手轮次中有一个没有结果块的 `server_tool_use` 块（最常见的情况是 Claude 将其与客户端工具一起调用），并且您的下一条用户消息结束了该轮次（例如，在 `tool_result` 块之后添加了文本），或者恢复请求不再定义该服务器工具（此时消息以 `but no <name> tool was provided` 结尾） | 发送一条仅包含客户端 `tool_use` ID 对应的 `tool_result` 块的用户消息，并保持相同的 `tools` 数组。请参阅[停止原因与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#tool-use)。                                                                                       |
| `Unsupported regex feature in pattern field: ...`                      | 严格工具的 `input_schema` 中的某个 `pattern` 使用了严格模式无法编译的正则表达式特性，例如反向引用、环视断言、单词边界或较大的 `{n,m}` 范围                                                                                        | 简化该模式。支持带有基本量词、字符类和分组的锚定模式；请参阅 [JSON Schema 限制](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#json-schema-limitations)。                                                                                                            |
| `All tools have defer_loading: true`                                   | 没有对模型可见的工具                                                                                                                                                                     | 至少必须有一个工具被立即加载。工具搜索工具本身绝不能设置 `defer_loading: true`。                                                                                                                                                                                                              |

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
