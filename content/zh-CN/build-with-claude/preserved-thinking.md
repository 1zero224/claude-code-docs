---
title: 保留思考
url: https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking
description: 在运行前缀检查的 Claude 模型上，保留思考决定模型能否使用您从早期轮次发回的思考块。
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

当您的代码发回思考块时，Claude 可以在早期轮次的推理基础上继续推理。*Preserved thinking*（保留思考）决定它可以使用其中哪些块。在[具有保留思考的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)上，可用的块会留在上下文中，并按输入计费。请[原样发回助手轮次](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#append-assistant-turns-exactly-as-returned)，与 API 返回时完全一致。

如果请求已因 `The block is bound to a different conversation` 而失败，请从[修复绑定到其他对话的思考块](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation)开始。

## 思考块与什么绑定

在这些模型上，思考块与以下内容绑定：

* **它之前的内容：** `system`、`tools` 以及每条更早的消息。如果其中任何一项发生变化，该块就无法通过["prefix check"（前缀检查）](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#prefix-check)，请求可能会收到 400 错误。因此，只向对话中添加内容（请参阅[更改工具或指令而不会导致前缀检查失败](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions)）。
* **模型：** 一个模型只能读取部分其他模型的思考块（请参阅[在 Claude API 上哪些模型切换会保留推理](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#which-model-switches-keep-reasoning)）。
* **账户：** 来自部分模型的思考块与账户绑定（请参阅[账户和云平台](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#account-bound-thinking)）。

[`prefix_mismatch_behavior`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#preserved-thinking-controls) 用于选择未通过前缀检查的请求是收到 400 错误，还是由 API 丢弃未通过前缀检查的块。响应中的 [`input_transformations`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#input-transformations) 会列出 API 丢弃或放行的思考块。丢弃思考是有代价的（请参阅[丢弃思考的代价](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation#drop-block-cost)）。

## 切换模型再切换回来

更改 `model`，并持续发送完整对话。新模型或账户无法使用的思考块永远不会导致错误。API 会在该次请求中丢弃它且不对其计费，Claude 会在没有该推理的情况下作答。

在[具有保留思考的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)上，[`thinking: {"type": "enabled"}`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#thinking-type-on-switch) 会导致 400 错误，而未通过[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#prefix-check)的请求也可能会收到该错误。

<Frame>
  ![同一历史记录依次发送给 Claude Fable 5.1、Claude Opus 5.5，再发送给 Claude Fable 5.1：其思考块被丢弃一次，然后被再次读取](https://platform.claude.com/docs/zh-CN/images/preserved-thinking-switch-and-back.svg)
</Frame>

Claude Opus 5.5 无法读取 Claude Fable 5.1 的思考块，因此 API 会在发给它的请求中丢弃这些块。回到 Claude Fable 5.1 后，这些块会被再次读取，因为 API 从不更改您的 `messages`。切换模型时，只有当您的代码移除推理时，推理才会永久丢失。因此，不要移除思考，也不要为每个模型单独维护一份历史记录。

被丢弃的块不算作对对话的更改，因此两个模型的思考块都保持有效。

## 并非由您的代码进行的切换

路由器、[拒绝后的回退](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#after-a-refusal)或服务器端回退可能会替您切换模型，[其他提供商的模型也可能接管某一轮次](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#other-provider-turns)。请持续发送完整对话。包含 `fallback` 块的轮次有其自身的规则。关于应排除哪些块，请参阅[继续对话](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#continuing-the-conversation)。

任何切换都可能选中不接受系统消息的模型。请参阅[当模型不接受系统消息时](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#unchosen-switches-details)。

## 账户和云平台

迁移到另一个账户或云平台的方式相同：持续发送完整历史记录。

来自 Claude Fable 5.2、Claude Mythos 5.2、Claude Sonnet 5.5 和 Claude Haiku 5.5 的思考块仅在生成它们的账户或与之关联的账户中有效。在其他任何账户中，API 会丢弃这些块，且不报错。

[什么算作您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#accounts-by-platform)取决于平台。

## 机密信息和不受信任的文本

对于[您的用户不会期望 Claude 信任的文件](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#untrusted-text)，请仅在 `tool_result` 块中发送。当您从历史记录中[移除机密信息、个人数据或注入的文本](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions#remove-sensitive-content)时，还要移除它们首次出现位置之后的每个思考块。之后 Claude 会在没有该推理的情况下工作。

## 让智能体检查您的代码

在 Claude Code 中，先运行 `claude update`，然后在构建请求的代码仓库中运行以下提示：

让 Claude Code 为您检查代码

```text
/claude-api preserved-thinking-migration
```

使用其他智能体时，请向它提供[更改工具或指令而不会导致前缀检查失败](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions)，并让它修复您的代码在请求之间更改 `system`、`tools` 或更早消息的每一处。

## 技术细节

### 在 Claude API 上哪些模型切换会保留推理

短横线表示您切换到的模型无法读取这些块。经过多次切换后，请为历史记录中的每个模型各查看一行。

| 切换前的模型 | 切换到 Claude Fable 5.2 或 Claude Mythos 5.2 | 切换到 Claude Fable 5.1 或 Claude Mythos 5.1 | 切换到 Claude Opus 5.5 | 切换到 Claude Sonnet 5.5 | 切换到 Claude Haiku 5.5 |
| ----- | ----- | ----- | ----- | ----- | ----- |
| Claude Haiku 5.5 | ✅ | — | ✅ | ✅ | ✅ |
| Claude Sonnet 5.5 | ✅ | — | ✅ | ✅ | — |
| Claude Opus 5 | ✅ | ✅ | ✅ | — | — |
| Claude Opus 5.5 | ✅ | ✅ | ✅ | — | — |
| Claude Fable 5 或 Claude Mythos 5 | ✅ | ✅ | — | — | — |
| Claude Fable 5.1 或 Claude Mythos 5.1 | ✅ | ✅ | — | — | — |
| Claude Fable 5.2 或 Claude Mythos 5.2 | ✅ | — | — | — | — |

如果某个模型没有对应的行或列，请参阅[较早的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#earlier-models)。

### 较早的模型，例如 Claude Opus 5 和 Claude Sonnet 5

每个 Claude 模型都能读取来自 Claude Opus 4.8、Claude Sonnet 5、Claude Haiku 4.5 以及更早的 Claude Opus、Claude Sonnet 和 Claude Haiku 模型的思考块。

| 您切换到的模型 | 仅读取来自以下模型的块 |
| ----- | ----- |
| Claude Opus 4.8、Claude Sonnet 5、Claude Haiku 4.5 以及更早的 Claude Opus、Claude Sonnet 和 Claude Haiku 模型 | 彼此 |
| Claude Opus 5 | 其自身，以及第一行中的模型 |
| Claude Fable 5 和 Claude Mythos 5 | 彼此、Claude Mythos Preview、Claude Opus 5，以及第一行中的模型 |

### 新模型上的思考设置

[具有保留思考的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)会以 400 错误拒绝 `thinking: {"type": "enabled"}`。当您将对话从请求中设置了 `"enabled"` 的模型迁移到这些模型之一时，请改为发送 `"adaptive"`。在这些模型上省略 `thinking` 同样会启用自适应思考。

### 当模型不接受系统消息时

如果某个模型不在[接受系统消息的模型列表](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)中，请仅在发给该模型的请求中省略系统消息。让它们继续留在您存储的历史记录中。回到接受系统消息的模型后，请再次发送完整的已存储历史记录。回来时不带这些消息属于更改，请求可能会收到该 400 错误。

### 拒绝之后

[允许的回退模型](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#naming-your-own-fallback-models)都无法读取拒绝模型的思考，因此回退模型会在没有该思考的情况下作答。回到您请求的模型后，Claude 会读取两个模型的思考。

### 来自其他提供商模型的轮次

来自其他提供商模型的轮次以 `text` 和 `tool_use` 内容的形式放在末尾。Claude 之前的思考保持有效。

### 各平台上什么算作您的账户

| 您调用 Claude 的位置 | 您的账户 |
| ----- | ----- |
| Claude API、Claude Platform on AWS、Claude Models on Google Cloud (Anthropic managed) 或 Microsoft Foundry | 您的组织 |
| Amazon Bedrock | 您的 AWS 账户 |
| Claude Models on Google Cloud (Google managed) | 您的 Google Cloud 项目 |

如果某个思考块由另一个账户生成，而该账户的信誉已不再良好，API 也可能会省略该思考块。这适用于发往 Claude Fable 和 Claude Mythos 模型、Claude Opus 5、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 的请求。请求仍会成功，模型会在没有该块推理的情况下作答。

## 可用性

保留思考在部分模型上可用。

### API 支持

具有保留思考的模型列于[未通过前缀检查时各模型的行为](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks#outcomes)中。

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：始终开启

| 状态 | 云平台 | 模型 |
| ----- | ----- | ----- |
| 可用 | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview) |
| 可用 | [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) | [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview) |
| 可用 | [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：始终开启

| 状态 | 云平台 | 模型 |
| ----- | ----- | ----- |
| 可用 | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview), [Claude Mythos 5.2](https://platform.claude.com/docs/zh-CN/models/mythos-5-2/overview), [Claude Fable 5.1](https://platform.claude.com/docs/zh-CN/models/fable-5-1/overview), [Claude Mythos 5.1](https://platform.claude.com/docs/zh-CN/models/mythos-5-1/overview), [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview), [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview), [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview) |

## 后续步骤

<CardGroup cols={2}>
  <Card title="思考块的前缀检查" icon="shield" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/prefix-checks">
    前缀检查比较哪些内容、请求未通过前缀检查时会发生什么，以及如何处理这两种情况。
  </Card>

  <Card title="更改工具或指令而不会导致前缀检查失败" icon="edit" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/change-tools-or-instructions">
    针对您的代码对对话所做的每种更改，应改为发送什么。
  </Card>

  <Card title="修复绑定到其他对话的思考块" icon="wrench" href="https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking/fix-bound-to-a-different-conversation">
    恢复失败的对话，并了解丢弃思考的代价。
  </Card>
</CardGroup>
