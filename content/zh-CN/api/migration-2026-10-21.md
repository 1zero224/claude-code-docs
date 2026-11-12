---
title: 迁移到 API 版本 2026-10-21
url: https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21
description: 将 Claude API 集成从 API 版本 2023-06-01 升级到 2026-10-21：需要安装的 SDK 版本以及需要进行的代码更改。
---

本指南列出了从 API 版本 `2023-06-01` 迁移到 `2026-10-21` 所需的代码更改。[升级步骤](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#migration-checklist)按顺序说明了需要执行的操作。

升级代码最简单的方式

```text
/claude-api api-version
```

## 升级步骤

<Steps>
  <Step title="切换到 2026-10-21">
    如果您使用 SDK，您的请求会使用该 SDK 主版本发布时的当前 API 版本。下面为您的语言列出的版本是最早使用 `2026-10-21` 的版本，因此请升级到这些版本。如果您不通过 SDK 调用 API，请将 `anthropic-version` 请求头设置为 `2026-10-21`。

    <Tabs>
      <Tab title="cURL">
        将 `anthropic-version` 请求头设置为 `2026-10-21`：

        ```bash
        curl https://api.anthropic.com/v1/messages \
          -H "x-api-key: $ANTHROPIC_API_KEY" \
          -H "anthropic-version: 2026-10-21" \
          -H "content-type: application/json" \
          -d '{
            "model": "claude-opus-5-5",
            "max_tokens": 1024,
            "messages": [{"role": "user", "content": "Hello, Claude"}]
          }'
        ```
      </Tab>

      <Tab title="CLI">
        将 `ant` CLI 升级到 2.0.0 版本。Homebrew 命令和发布版下载方式保持不变。

        Go 要求主版本为 2 或更高的模块路径以该版本结尾，因此 CLI 的模块路径现在是 `github.com/anthropics/anthropic-cli/v2`。如果您使用 `go install` 安装，请在路径中添加 `/v2`：

        ```bash
        # 之前
        go install 'github.com/anthropics/anthropic-cli/cmd/ant@latest'

        # 之后
        go install 'github.com/anthropics/anthropic-cli/v2/cmd/ant@latest'
        ```

        旧路径会继续安装最新的 v1 版本，且不会给出任何错误或警告。
      </Tab>

      <Tab title="Python">
        将 `anthropic` 包升级到 2.0.0 版本。
      </Tab>

      <Tab title="TypeScript">
        将 `@anthropic-ai/sdk` 包升级到 1.0.0 版本。

        `@anthropic-ai/bedrock-sdk`、`@anthropic-ai/vertex-sdk`、`@anthropic-ai/foundry-sdk`、`@anthropic-ai/aws-sdk` 和 `@anthropic-ai/google-cloud-sdk` 的 v1 需要 `@anthropic-ai/sdk` 的 v1，而它们的早期版本需要低于 1 的版本。如果您使用其中任何一个，请在同一次升级中将其迁移到 v1。否则，您的包管理器会为它额外安装一份较旧的 `@anthropic-ai/sdk`。
      </Tab>

      <Tab title="C#">
        将 `Anthropic` 包升级到 13.0.0 版本。

        `Anthropic.Bedrock`、`Anthropic.Vertex`、`Anthropic.Foundry`、`Anthropic.Aws`、`Anthropic.GoogleCloud` 和 `Anthropic.Mcp` 的 1.0.0 版本需要 `Anthropic` 的 13.0.0 版本。如果您使用其中任何一个，请在同一次升级中将其迁移到 1.0.0。早期版本仍可与 13.0.0 一起安装，但运行时可能会失败。
      </Tab>

      <Tab title="Go">
        升级到 Go SDK 的 2.0.0 版本，其模块路径为 `github.com/anthropics/anthropic-sdk-go/v2`。Go 模块的新主版本具有新的导入路径，因此请将 `github.com/anthropics/anthropic-sdk-go` 及其包（例如 `github.com/anthropics/anthropic-sdk-go/option`）的每个导入都改为以 `github.com/anthropics/anthropic-sdk-go/v2` 开头。然后运行 `go get github.com/anthropics/anthropic-sdk-go/v2`。
      </Tab>

      <Tab title="Java">
        将 `com.anthropic:anthropic-java` 构件升级到 3.0.0 版本。

        `anthropic-java-bedrock`、`anthropic-java-vertex`、`anthropic-java-aws`、`anthropic-java-foundry`、`anthropic-java-google-cloud` 和 `anthropic-java-mcp` 的 3.0.0 版本与 `anthropic-java` 3.0.0 配套。如果您使用其中任何一个，请在同一次升级中将其迁移到 3.0.0。
      </Tab>

      <Tab title="PHP">
        运行 `composer require "anthropic-ai/sdk"` 将包升级到 1.0.0 版本。仅运行 `composer update` 会停留在您当前的版本约束范围内。
      </Tab>

      <Tab title="Ruby">
        将 `anthropic` gem 升级到 2.0.0 版本。
      </Tab>
    </Tabs>

    <Warning>
      在 Amazon Bedrock 上，`2026-10-21` 仅适用于 Claude 4.7 及更高版本的模型。您可能会在 Bedrock 上调用版本号为 4.6 或更低的模型（请参阅 [Amazon Bedrock 上的 Claude（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)）。如果是这样，请让这些调用继续使用它们当前使用的 API 版本。如果您使用 SDK，请继续使用您当前的 SDK 版本。如果您自己设置 API 版本，请继续发送您当前发送的版本。要将这些调用迁移到 `2026-10-21`，请先将它们切换到 Claude 4.7 或更高版本的模型。
    </Warning>

    请将此次升级与下一步中的修复一起发布。
  </Step>

  <Step title="针对破坏性变更更新您的代码">
    逐一查看[破坏性变更](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#breaking-changes)，并针对影响您的每一项更新代码。同时完成[新版 SDK 或 CLI 所需的代码更改](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#sdk-changes)。
  </Step>

  <Step title="针对 2026-10-21 进行测试">
    在发布之前，针对 `2026-10-21` 测试您的集成。
  </Step>
</Steps>

## 破坏性变更

### Messages API

这些变更适用于对 Messages API（`POST /v1/messages`）的请求、[令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)，以及使用 `2026-10-21` 创建的[消息批次](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)中的每个请求。

#### assistant 消息中的文档块会返回错误

如果您直接在 `assistant` 消息中发送 `document` 块，则会受到影响。

在 `2026-10-21` 中，直接在 `assistant` 消息中放置 `document` 块的请求会返回 400 错误。请改为在 `user` 消息中发送文档。嵌套在其他块中的文档（例如 Claude 返回的网页获取工具结果）仍然被接受。

#### user 消息中的引用会返回错误

如果您在 `user` 消息的 `text` 块上发送 `citations`（包括在 `search_result` 块内），则会受到影响。

在 `2026-10-21` 中，在 `user` 消息中发送带有非空 `citations` 数组的 `text` 块的请求会返回 400 错误。请从这些块中移除 `citations`。`assistant` 消息中 `text` 块上的引用（例如您回传的 Claude 先前回复）仍然被接受，文档和搜索结果上的 `"citations": {"enabled": true}` 设置也同样被接受。

#### 采样参数已移除

如果您在 Messages 请求或消息批次请求中发送 `temperature`、`top_p` 或 `top_k`，则会受到影响。

在 `2026-10-21` 中，发送 `temperature`、`top_p` 或 `top_k` 的请求会返回 400 错误，即使参数值为默认值也是如此。

请从您的请求中移除这三个参数。

从最新的主版本开始，SDK 和 CLI 不再包含这三个参数。

没有这些参数时，请求会使用它们的默认值进行采样，包括 `temperature` 为 1.0，这与省略这些参数的 `2023-06-01` 请求相同。已经只接受默认值的模型（例如 Claude 4.7 及更高版本的模型）的采样方式与之前相同。在接受其他值的模型（例如 Claude Sonnet 4.6）上，通过降低 `temperature` 来获得更一致输出的请求将失去该效果。没有其他设置可以替代它。如果某个请求依赖非默认值，请停止发送该参数，或让该请求继续使用 `2023-06-01`。

### Admin API 和 Claude Enterprise Analytics API

如果您的代码读取下表中的 Admin API 或 Claude Enterprise Analytics API 响应字段，则会受到影响。

在 `2026-10-21` 中，Admin API 和 [Claude Enterprise Analytics API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api) 的响应不再包含已重命名字段的旧名称。请改用新名称。

Admin API：

| 端点                                                                                                                                                                                                                                                                                       | 旧名称            | 新名称                                                                                                                              |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| [RBAC 组](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/list)：创建、列出、检索和更新                                                                                                                                                                                      | `roles`        | `role_ids`                                                                                                                       |
| [RBAC 组成员](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/members/list)：列出和添加，以及[移除成员](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_groups/members/remove)的响应                                                                             | `group_id`     | `rbac_group_id`                                                                                                                  |
| [RBAC 角色](https://platform.claude.com/docs/zh-CN/api/beta/organization/rbac_roles/list)：列出和检索                                                                                                                                                                                            | `name`         | `display_name`                                                                                                                   |
| [组织](https://platform.claude.com/docs/zh-CN/api/organization/rate_limits/list)和[工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/rate_limits/list)速率限制：列出，以及[设置工作区速率限制](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/rate_limits/update)的响应 | `group_type`   | `group.type`                                                                                                                     |
| [API 密钥](https://platform.claude.com/docs/zh-CN/api/organization/api_keys/list)：列出、检索和更新                                                                                                                                                                                                 | `workspace_id` | `scope`（值不同；请参阅 [API 密钥上的 `workspace_id`](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#api-key-workspace-id)） |

Claude Enterprise Analytics API：

| 端点                                                                                                                                                                                                            | 旧名称           | 新名称                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- | --------------------- |
| 按用户的[使用量](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics/user_usage_report/list)和[成本](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics/user_cost_report/list)报告 | `actor.email` | `actor.email_address` |
| [活动摘要](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics/summaries/list)                                                                                                                 | `summaries`   | `data`                |

速率限制列表端点上的 `group_type` 查询参数以及 API 密钥列表端点上的 `workspace_id` 筛选器保持不变。

#### API 密钥上的 `workspace_id`

如果您的代码读取 API 密钥上的 `workspace_id`，则会受到影响。

在 `2023-06-01` 中，API 密钥的 `workspace_id` 是该密钥所绑定工作区的 ID；对于绑定到默认工作区的密钥以及未绑定到单个工作区的密钥（全工作区密钥），该值为 `null`。

在 `2026-10-21` 中，`workspace_id` 已被移除。请改为读取 `scope`：对于绑定到某个工作区（包括默认工作区）的密钥，`scope.workspace_id` 是该工作区的真实 ID；全工作区密钥的 `scope` 为 `{"type": "organization"}`，不含 `workspace_id`。

如果您的代码将 `null` 的 `workspace_id` 视为默认工作区，请改为将 `scope.workspace_id` 与您的默认工作区的 ID 进行比较。要查找该 ID，请使用 `include_default=true` 调用[列出工作区](https://platform.claude.com/docs/zh-CN/api/organization/workspaces/list)，并取名为 `"Default"` 的工作区。

## SDK 和 CLI 变更

早期的 SDK 和 CLI 版本在每个请求中都发送 `anthropic-version: 2023-06-01`。[切换到 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#upgrade) 中列出的版本会发送 `2026-10-21`，包括它们的 Amazon Bedrock 和 Google Cloud 客户端。[破坏性变更](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#breaking-changes)中的变更同样适用于它们：已移除的参数和旧字段名称也已从 SDK 的类型以及 CLI 的标志和输出中删除。

### `MessageParam` 是一个联合类型，每个角色对应一种类型

在早期版本中，`MessageParam` 是适用于所有角色的单一类型。现在每个角色都有自己的类型：`UserMessageParam`、`AssistantMessageParam` 和 `SystemMessageParam`。`BetaMessageParam` 也以相同方式拆分为 `BetaUserMessageParam`、`BetaAssistantMessageParam` 和 `BetaSystemMessageParam`。SDK 发送的请求具有与以前相同的字段和值。

<Tabs exclude="shell">
  <Tab title="Python">
    以 dict 字面量编写的消息（例如 `{"role": "user", "content": "Hello"}`）无需更改。

    在 v1 中，`anthropic.types.MessageParam` 是一个 `TypedDict`。在 v2 中，它是这三种类型的联合，而 `anthropic.types.beta.BetaMessageParam` 是其对应三种类型的联合。这两个名称在类型注解中仍然可用，例如 `list[MessageParam]`。

    现在调用这两个名称中的任何一个都会引发 `TypeError`，类型检查器也会标记它。请将消息写成 dict：

    ```python
    # 之前
    message = MessageParam(role="user", content="Hello")

    # 之后
    message: MessageParam = {"role": "user", "content": "Hello"}
    ```

    对这两个名称进行子类化同样会引发 `TypeError`。请对相应角色的类型进行子类化，例如 `UserMessageParam`。

    其余变更仅影响类型检查。如果 dict 的 `role` 是一个可能包含多个角色的变量，它将无法匹配联合中的任何类型，因此请一次为一个角色构建 dict：

    ```python
    # 之前
    def to_message(role: Literal["user", "assistant"], text: str) -> MessageParam:
        return {"role": role, "content": text}


    # 之后
    def to_message(role: Literal["user", "assistant"], text: str) -> MessageParam:
        if role == "user":
            return {"role": "user", "content": text}
        return {"role": "assistant", "content": text}
    ```

    还有两项仅适用于 `BetaMessageParam`：

    * `clear_at` 和 `output_config` 是 `BetaSystemMessageParam` 的键，因此用户消息或助手消息不能包含它们。
    * `content` 在 `BetaSystemMessageParam` 中是可选的，因此 pyright 会报告 `message["content"]`。请使用 `message.get("content")` 读取它，或先检查 `message["role"]` 不是 `"system"`。
  </Tab>

  <Tab title="TypeScript">
    `MessageParam` 以前是一个接口。现在它是 `UserMessageParam | AssistantMessageParam | SystemMessageParam`，而 `BetaMessageParam` 是其三种类型的相同联合。只有类型发生了变化。指明了角色的消息字面量（例如 `{ role: "user", content: "Hello, Claude" }`）的类型检查结果与以前相同，因此大多数代码无需更改。

    系统类型与另外两种不同。它的 `content` 是字符串或文本块，在 `beta` 中还包括 `tool_addition` 和 `tool_removal` 块，因此在其中使用图像块将无法再通过编译。在 `beta` 中，只有它具有 `clear_at` 和 `output_config`，且其 `content` 是可选的。

    处理任意角色消息的代码在替换或添加内容块之前，以及在读取 `clear_at` 或 `output_config` 之前，必须先检查 `role`：

    ```typescript
    // 之前
    const copy: Anthropic.MessageParam = { ...message, content: blocks };

    // 之后
    const copy: Anthropic.MessageParam =
      message.role === "system" ? message : { ...message, content: blocks };
    ```

    `BetaMessageParam` 的 `content` 可以是 `undefined`：

    ```typescript
    // 之前
    const types =
      typeof message.content === "string" ? [] : message.content.map((block) => block.type);

    // 之后
    const types = Array.isArray(message.content) ? message.content.map((block) => block.type) : [];
    ```

    类型 `BetaMessageParam["content"]` 同样包含 `undefined`。对于用户消息或助手消息的内容，请使用 `BetaMessageContentParam`。

    接口不能扩展联合类型，类也不能实现联合类型。请改用类型别名，或指定某个角色的类型。

    ```typescript
    // 之前
    interface StoredMessage extends Anthropic.MessageParam {
      id: string;
    }

    // 之后
    type StoredMessage = Anthropic.MessageParam & { id: string };
    ```
  </Tab>

  <Tab title="C#">
    在 v12 中，`MessageParam` 和 `BetaMessageParam` 是由您设置 `Role` 和 `Content` 的类。在 v13 中，它们各自是其角色对应的三个类的联合，并且 `Anthropic.Models.Messages` 和 `Anthropic.Models.Beta.Messages` 中的 `Role` 枚举已被移除。

    构建消息时，请指定其角色对应的类并删除 `Role`。每个类都可以隐式转换为联合类型：

    ```csharp
    // 之前
    Messages = [new() { Role = Role.User, Content = "Hello, Claude" }],

    // 之后
    Messages = [new UserMessageParam { Content = "Hello, Claude" }],
    ```

    `new UserMessageParam("Hello, Claude")` 同样可用。

    承载 `Content` 的类也发生了变化：

    * `MessageParamContent` 现在是 `MessageContentParam`，`BetaMessageParamContent` 现在是 `BetaMessageContentParam`。
    * 系统消息有自己的内容类。`SystemMessageParamContent` 包含字符串或 `TextBlockParam` 列表，`BetaSystemMessageParamContent` 包含字符串或 `SystemMessageContentBlock` 列表。`List<ContentBlockParam>` 或 `List<BetaContentBlockParam>` 不再能转换为它。
    * `ClearAt` 和 `OutputConfig` 仅存在于 `BetaSystemMessageParam` 上。

    读取消息时，联合类型仍然具有 `Role` 和 `Content`，但您无法设置它们，因此 `with` 表达式需要作用于该角色对应的类。`Role` 现在包含 `MessageParamRole` 或 `BetaMessageParamRole`。`BetaMessageParam.Content` 可以为 `null`，因为 `Content` 在 `BetaSystemMessageParam` 上是可选的。

    ```csharp
    // 之前
    if (message.Role.Value() == Role.Assistant) { }

    // 之后
    if (message.Role.Value() == MessageParamRole.Assistant) { }
    ```

    `TryPickUser`、`TryPickAssistant`、`TryPickSystem`、`Switch` 和 `Match` 可为您提供该角色对应的类。
  </Tab>

  <Tab title="Go">
    在 v1 中，`MessageParam` 是一个带有 `Role` 和 `Content` 字段的结构体。在 v2 中，它是一个联合：`OfUser` 包含 `*UserMessageParam`，`OfAssistant` 包含 `*AssistantMessageParam`，`OfSystem` 包含 `*SystemMessageParam`。`BetaMessageParam` 也以相同方式变化。除 beta 系统消息中键的顺序外，SDK 发送的 JSON 与以前相同。

    `NewUserMessage`、`NewAssistantMessage`、`NewBetaUserMessage` 以及 `Message` 和 `BetaMessage` 的 `ToParam` 方法与以前一样工作，因此使用它们构建消息的代码无需更改。

    结构体字面量需指明其角色对应的类型。请省略 `Role`，因为每种类型都会自行序列化其角色：

    ```go
    // 之前
    anthropic.MessageParam{
    	Role:    anthropic.MessageParamRoleAssistant,
    	Content: blocks,
    }

    // 之后
    anthropic.MessageParam{OfAssistant: &anthropic.AssistantMessageParam{
    	Content: blocks,
    }}
    ```

    读取消息的代码需要检查设置了哪个字段：

    ```go
    // 之前
    if message.Role == anthropic.MessageParamRoleUser {
    	blocks = message.Content
    }

    // 之后
    if message.OfUser != nil {
    	blocks = message.OfUser.Content
    }
    ```

    * `MessageParamRole` 和 `BetaMessageParamRole` 类型及其常量已被移除。`message.GetRole()` 返回指向角色 `string` 的指针，当没有设置任何字段时返回 `nil`。
    * `ClearAt` 和 `OutputConfig` 已从 `BetaMessageParam` 移至 `BetaSystemMessageParam`。`BetaMessageParamClearAt` 类型及其常量的名称现在以 `BetaSystemMessageParamClearAt` 开头。
    * `SystemMessageParam.Content` 是 `[]TextBlockParam`。`BetaSystemMessageParam.Content` 是 `[]BetaSystemMessageParamContentUnion`，其中包含 `text`、`tool_addition` 和 `tool_removal` 块。`NewBetaSystemMessage` 接受该类型的块，因此 `anthropic.NewBetaTextBlock("Be brief.")` 变为 `anthropic.BetaSystemMessageParamContentUnion{OfText: &anthropic.BetaTextBlockParam{Text: "Be brief."}}`。
    * `MessageParam` 的副本与原始值指向同一条消息。设置 `duplicate.OfUser.Content` 会同时改变两者，而在 v1 中设置 `duplicate.Content` 只会改变副本。
    * 对于 `user` 或 `assistant` 以外的角色，`mcp.ToMessage` 会返回 `*mcp.UnsupportedValueError`。v1 会将任何角色复制到消息中。
  </Tab>

  <Tab title="Java">
    在 v2 中，`MessageParam` 是一个带有 `role` 和 `content` 的单一类。在 v3 中，它是这三个类的联合，而 `MessageParam.Content` 现在是顶层的 `MessageContentParam`。`BetaMessageParam` 也以相同方式变化，其类名相同但带有 `Beta` 前缀。SDK 发送的 JSON 相同，只是 beta 系统消息现在将 `role` 放在 `content` 之前。

    接受 `String` 的 `addUserMessage`、`addAssistantMessage` 和 `addSystemMessage`，接受 `Message` 或 `MessageParam` 的 `addMessage`，以及 `Message.toParam()` 均保持不变。

    `MessageParam.builder()` 和 `toBuilder()` 已被移除。请调用该角色对应的工厂方法，即 `ofUser`、`ofAssistant` 或 `ofSystem`：

    ```java
    // 之前
    MessageParam message = MessageParam.builder()
        .role(MessageParam.Role.USER)
        .content("Hello, Claude")
        .build();

    // 之后
    MessageParam message = MessageParam.ofUser("Hello, Claude");
    ```

    在 `MessageCreateParams`、`StructuredMessageCreateParams`、`MessageCountTokensParams` 和 `BatchCreateParams.Request.Params` 的构建器上，接受内容块列表的方法有了新名称：

    | v2                                 | v2 beta                                       | v3                                        |
    | ---------------------------------- | --------------------------------------------- | ----------------------------------------- |
    | `addUserMessageOfBlockParams`      | `addUserMessageOfBetaContentBlockParams`      | `addUserMessageOfContentBlockParams`      |
    | `addAssistantMessageOfBlockParams` | `addAssistantMessageOfBetaContentBlockParams` | `addAssistantMessageOfContentBlockParams` |
    | `addSystemMessageOfBlockParams`    | `addSystemMessageOfBetaContentBlockParams`    | `addSystemMessageOfBlocks`                |

    在内容类上，`ofBlockParams`、`blockParams()`、`isBlockParams()`、`asBlockParams()` 以及访问者的 `visitBlockParams` 中的 `BlockParams`（在 beta 中为 `BetaContentBlockParams`）变为 `ContentBlockParams`。

    系统消息有自己的内容类 `SystemMessageParam.Content`，`addSystemMessage` 现在接受该类。它的块是 `TextBlockParam`，而不是 `ContentBlockParam`。在 beta 中，它们是 `BetaSystemMessageParam.Content.SystemMessageContentBlock`，具有 `ofText`、`ofToolAddition` 和 `ofToolRemoval`。

    ```java
    // 之前
    builder.addSystemMessageOfBlockParams(List.of(ContentBlockParam.ofText(textBlock)));

    // 之后
    builder.addSystemMessageOfBlocks(List.of(textBlock));
    ```

    读取消息时，`role()` 保持不变。`content()` 返回 `MessageContentParam`，在 beta 中返回 `Optional<BetaMessageContentParam>`，因为系统消息可以没有内容。`_role()`、`_content()` 和 `_additionalProperties()` 现在位于该角色对应的类上，该类由 `asUser()`、`asAssistant()` 或 `asSystem()` 返回。
  </Tab>

  <Tab title="PHP">
    这些类位于 `Anthropic\Messages` 中，beta 类位于 `Anthropic\Beta\Messages` 中。以数组形式编写的消息保持不变，并且仍需要 `'role'`。

    `MessageParam::with()` 和 `BetaMessageParam::with()` 仍然保留，并返回与 `role` 对应的类。使用命名参数的调用无需更改。`role` 现在是第一个参数，因此在位置参数调用中，请交换 `content` 和 `role`，或者在该角色对应的类上调用 `with()`，它不接受 `role`。对于按旧顺序传递的参数，PHP 会抛出 `UnhandledMatchError`（PHPStan 不会对此发出警告）；如果 `content` 是列表或 `role` 是枚举成员，则会抛出 `TypeError`。

    ```php
    // 之前
    MessageParam::with('Hello, Claude', 'user');

    // 之后
    UserMessageParam::with('Hello, Claude');
    ```

    不再有任何对象是 `MessageParam`：`instanceof MessageParam` 为 false，并且在参数类型或返回类型指定它的地方，PHP 会抛出 `TypeError`。请改为指定这三个类：

    ```php
    // 之前
    function send(MessageParam $message): void {}

    // 之后
    function send(UserMessageParam|AssistantMessageParam|SystemMessageParam $message): void {}
    ```

    * `MessageParam::fromArray()` 和 `(new MessageParam())->withContent()` 会抛出 “Call to undefined method”。请在该角色对应的类上调用它们，该类会自行设置 `role`。
    * `withRole()` 现在只接受字符串：对于 `Role` 成员，PHP 会抛出 `TypeError`。要为消息指定另一个角色，请构建该角色对应的类。
    * `clearAt` 和 `outputConfig` 仅存在于 `BetaSystemMessageParam` 上，对于另外两个角色，`BetaMessageParam::with()` 会丢弃它们。`BetaMessageParam\ClearAt` 枚举现在是 `BetaSystemMessageParam\ClearAt`。
    * `MessageParam\Content` 现在是 `MessageContentParam`，`BetaMessageParam\Content` 现在是 `BetaMessageContentParam`。它们的 PHPStan 类型也采用新名称：`ContentShape` 现在是 `MessageContentParamShape`。
    * `'role'` 为 `Role` 成员的数组仍然可用，但 PHPStan 现在会报告它。请改写为字符串。
  </Tab>

  <Tab title="Ruby">
    以哈希形式编写的消息保持不变。

    在 v2 中，`Anthropic::MessageParam` 是 `Anthropic::UserMessageParam`、`Anthropic::AssistantMessageParam` 和 `Anthropic::SystemMessageParam` 的联合。同样，`Anthropic::Beta::BetaMessageParam` 是其三个类的联合。

    `Anthropic::MessageParam.new(role: :user, content: "Hello, Claude")` 仍然可用，并返回该角色对应类的实例。您也可以自己构建该类，无需 `role:`，写作 `Anthropic::UserMessageParam.new(content: "Hello, Claude")`。对于构建或检查这些实例的代码，有四处变化：

    * `MessageParam.new` 仅接受关键字参数，因此请以 `MessageParam.new(**hash)` 的形式传入哈希。当缺少 `role:` 或它不是这三个角色之一时，会引发 `ArgumentError`。
    * `message.is_a?(Anthropic::MessageParam)` 为 `false`。`when Anthropic::MessageParam` 分支和 `messages.grep(Anthropic::MessageParam)` 仍然能够匹配。
    * 使用 `.new` 构建的实例在其 `to_h` 中没有 `:role`，且 `message[:role]` 为 `nil`。`message.role` 仍然返回角色。
    * 只有 `BetaSystemMessageParam` 具有 `clear_at` 和 `output_config`。在用户消息或助手消息上，它们以前返回 `nil`，现在会引发 `NoMethodError`。

    ```ruby
    # 之前
    typed = messages.select { _1.is_a?(Anthropic::MessageParam) }
    roles = typed.map { _1[:role] }

    # 之后
    typed = messages.grep(Anthropic::MessageParam)
    roles = typed.map(&:role)
    ```

    `Role` 枚举模块保留。以下常量已迁移。以 `Beta` 开头的名称位于 `Anthropic::Beta` 下，其他名称位于 `Anthropic` 下：

    | 已移除                         | 改用                                                                    |
    | --------------------------- | --------------------------------------------------------------------- |
    | `MessageParam::Content`     | `MessageContentParam`，对于系统消息则使用 `SystemMessageParam::Content`         |
    | `BetaMessageParam::Content` | `BetaMessageContentParam`，对于系统消息则使用 `BetaSystemMessageParam::Content` |
    | `BetaMessageParam::ClearAt` | `BetaSystemMessageParam::ClearAt`                                     |

    如果您使用 Sorbet 进行类型检查，`MessageParam::OrHash` 已被移除，且作为类型的 `Anthropic::MessageParam` 不再接受实例。请对这三个类使用 `Anthropic::MessageParam::Variants`，每个类都有自己的 `OrHash`。在 RBS 中，类型为 `Anthropic::MessageParam` 的值没有 `role` 或 `content`。请使用 `Anthropic::Models::message_param` 别名，它以前是消息的哈希形式，现在是这三个类的联合。beta 对应的名称是 `BetaMessageParam::Variants` 和 `Anthropic::Models::Beta::beta_message_param`。
  </Tab>
</Tabs>

### 单值枚举变为常量

这会影响 C#、Go、Java、PHP 和 Ruby SDK 以及一个 CLI 标志。在早期版本中，只能包含一个值的必填字段拥有自己的枚举类型，且该枚举只有一个成员。这些字段现在是常量，相应的枚举类型已被移除。SDK 发送和读取的 JSON 保持不变。

<Tabs exclude="shell:cURL, python, typescript">
  <Tab title="CLI">
    `--type` 在 `ant beta:sessions:resources add` 上是可选的。`file` 是 `--type` 在此处唯一可取的值，因此 v2 会为您发送它。仍然传入 `--type file` 的命令可以继续正常工作，因此无需进行任何更改。
  </Tab>

  <Tab title="C#">
    在 v12 中，此类 `required` 属性拥有一个只有单个成员的枚举，例如 `BetaManagedAgentsTextBlockType.Text`。它们几乎都是 `Anthropic.Models.Beta` 下某个类的 `Type` 属性：`Agents`、`DeploymentRuns`、`Deployments`、`Dreams`、`Environments`、`MemoryStores`、`Sessions`、`UserProfiles` 和 `Vaults`。在 v13 中，这些属性是由构造函数填充的 `JsonElement`，就像 `TextBlockParam.Type` 之前那样。

    可选属性（例如 `BetaManagedAgentsGlobToolConfigParams.Type`）保留其枚举。其中两个枚举有了新名称：

    | v12                                         | v13                                       |
    | ------------------------------------------- | ----------------------------------------- |
    | `BetaManagedAgentsBashToolConfigParamsType` | `Anthropic.Models.Beta.Agents.Type`       |
    | `BetaPackagesType`                          | `Anthropic.Models.Beta.Environments.Type` |

    构建对象时，请删除该属性：

    ```csharp
    // 之前
    new BetaManagedAgentsTextBlock
    {
        Type = BetaManagedAgentsTextBlockType.Text,
        Text = "Hello Claude!",
    };

    // 之后
    new BetaManagedAgentsTextBlock { Text = "Hello Claude!" };
    ```

    只剩一个必填属性的类还会新增一个接受该属性的构造函数，因此 `new BetaManagedAgentsTextBlock("Hello Claude!")` 同样可用。没有剩余必填属性的类使用 `new BetaManagedAgentsAlwaysAllowPolicy()` 构建。

    读取时，请比较字符串：

    ```csharp
    // 之前
    if (message.Type.Value() == BetaManagedAgentsAgentMessageEventType.AgentMessage) { }

    // 之后
    if (message.Type.GetString() == "agent.message") { }
    ```

    要区分联合类型的各个变体，请继续使用其 `TryPick*`、`Switch` 和 `Match` 方法，这些方法保持不变。
  </Tab>

  <Tab title="Go">
    此类枚举类型及其常量的一个示例是 `BetaManagedAgentsTextBlockType` 和 `BetaManagedAgentsTextBlockTypeText`。它们几乎都是 `client.Beta` 下某个结构体的 `Type` 字段：Agents、Deployments、Deployment Runs、Dreams、Environments、Memory Stores、Sessions、User Profiles 和 Vaults。在 v2 中，这些字段使用 `shared/constant` 包中的类型，就像 `TextBlockParam.Type` 之前那样。可选字段（例如 `BetaManagedAgentsBashToolConfigParams.Type`）保留其枚举。

    在请求中，请删除该字段。省略它时，它会被序列化为其唯一的值：

    ```go
    // 之前
    anthropic.BetaManagedAgentsTextBlockParam{
    	Type: anthropic.BetaManagedAgentsTextBlockTypeText,
    	Text: "Hello Claude!",
    }

    // 之后
    anthropic.BetaManagedAgentsTextBlockParam{
    	Text: "Hello Claude!",
    }
    ```

    唯一字段为该 `Type` 的结构体现在有了构造函数，例如 `NewBetaManagedAgentsAlwaysAllowPolicyParam()`。作为联合类型成员时，指向空结构体的指针同样可用：`OfAlwaysAllow: &anthropic.BetaManagedAgentsAlwaysAllowPolicyParam{}`。

    在响应中，请与字符串进行比较：

    ```go
    // 之前
    switch event.Type {
    case string(anthropic.BetaManagedAgentsAgentMessageEventTypeAgentMessage):
    }

    // 之后
    switch event.Type {
    case "agent.message":
    }
    ```

    您自己使用已移除类型之一声明的变量、字段或参数，需改用该字段的新类型，例如 `constant.AgentMessage`。`constant.ValueOf[constant.AgentMessage]()` 返回其值。

    联合类型构造函数随字段一起变化。以下五个构造函数失去了 `type_` 参数：

    * `BetaManagedAgentsEventParamsOfUserToolConfirmation`
    * `BetaManagedAgentsEventParamsOfUserDefineOutcome`
    * `BetaManagedAgentsDeploymentInitialEventParamsOfUserDefineOutcome`
    * `BetaManagedAgentsMultiagentRosterEntryParamsOfBetaManagedAgentsAgents`
    * `BetaManagedAgentsMultiagentRosterEntryParamsOfBetaManagedAgentsAdvisors`

    以下四个构造函数不接受其他参数，因此已被移除。请将联合类型的字段设置为指向空结构体的指针，例如 `BetaManagedAgentsEventParamsUnion{OfUserInterrupt: &anthropic.BetaManagedAgentsUserInterruptEventParams{}}`：

    * `BetaManagedAgentsEventParamsOfUserInterrupt`
    * `BetaManagedAgentsMultiagentRosterEntryParamsOfBetaManagedAgentsMultiagentSelfs`
    * `BetaManagedAgentsCredentialNetworkingParamsOfUnrestricted`
    * `BetaOutputBehaviorParamOfCreateNew`
  </Tab>

  <Tab title="Java">
    此类枚举的一个示例是嵌套类 `BetaManagedAgentsTextBlock.Type`，其唯一成员为 `TEXT`。它们几乎都是 `com.anthropic.models.beta` 下某个类的 `type`：`agents`、`deploymentruns`、`deployments`、`dreams`、`environments`、`memorystores`、`sessions`、`userprofiles` 和 `vaults`。在 v3 中，这些字段是由构建器填充的 `JsonValue`，就像 `TextBlockParam` 的该字段之前那样。可选字段（例如 `BetaManagedAgentsBashToolConfigParams` 的 `type`）保留其枚举。

    构建对象时，请删除该调用：

    ```java
    // 之前
    BetaManagedAgentsTextBlock block = BetaManagedAgentsTextBlock.builder()
        .type(BetaManagedAgentsTextBlock.Type.TEXT)
        .text("Hello Claude!")
        .build();

    // 之后
    BetaManagedAgentsTextBlock block = BetaManagedAgentsTextBlock.builder()
        .text("Hello Claude!")
        .build();
    ```

    只剩一个必填字段的类还会新增一个接受该字段的工厂方法，因此 `BetaManagedAgentsTextBlock.of("Hello Claude!")` 同样可用。没有剩余必填字段的类会失去 `of(type)`，其联合类型也会失去接受该枚举的重载，例如 `ofAlwaysAllow(type)`。请使用 `BetaManagedAgentsAlwaysAllowPolicy.builder().build()` 构建它。

    读取时，`type()` 已被移除，`_type()` 返回 `com.anthropic.core.JsonValue`：

    ```java
    // 之前
    if (message.type().equals(BetaManagedAgentsAgentMessageEvent.Type.AGENT_MESSAGE)) {}

    // 之后
    if (message._type().equals(JsonValue.from("agent.message"))) {}
    ```

    要区分联合类型的各个变体，请继续使用其 `is*()` 和 `as*()` 方法及其访问者，这些保持不变。
  </Tab>

  <Tab title="PHP">
    此类枚举成员的一个示例是 `Anthropic\Beta\Sessions\Events\ManagedAgentsTextBlock\Type::TEXT`。它们几乎都是 `Anthropic\Beta` 下某个 Managed Agents 类的 `type`。这些属性现在是以该唯一值为默认值的字符串，就像 `Anthropic\Messages\TextBlockParam` 的 `type` 之前那样。可选属性（例如 `Anthropic\Beta\Environments\BetaPackages` 的 `type`）保留其枚举。

    `with()` 不再接受 `type`。请将其移除：对于残留的 `type:`，PHP 会抛出 “Unknown named parameter”；在位置参数调用中，其后的参数会前移一位。

    ```php
    // 之前
    ManagedAgentsTextBlock::with(text: 'Hello Claude!', type: Type::TEXT);

    // 之后
    ManagedAgentsTextBlock::with(text: 'Hello Claude!');
    ```

    在您与枚举成员进行比较的地方，请改为与字符串比较：

    ```php
    // 之前
    $event->type === Type::AGENT_MESSAGE->value;

    // 之后
    $event->type === 'agent.message';
    ```

    * `withType()` 以及 `$client->beta->sessions->resources->add()` 的 `type` 参数现在只接受字符串。在 `add()` 中它默认为 `'file'`，因此您可以省略它。
    * 以数组形式编写的请求保持不变，并且仍然需要其 `'type'`。
  </Tab>

  <Tab title="Ruby">
    此类枚举常量的一个示例是 `Anthropic::Beta::Sessions::BetaManagedAgentsTextBlock::Type::TEXT`。它们几乎都是 `Anthropic::Beta` 下某个 Managed Agents 类的 `type`。在 v2 中，这些字段被声明为常量，就像 `Anthropic::TextBlockParam` 的 `type` 之前那样。可选字段（例如 `Anthropic::Beta::BetaManagedAgentsGlobToolConfigParams` 的 `type`）保留其枚举。

    请将已移除的常量替换为其符号：

    ```ruby
    # 之前
    event.type == Anthropic::Beta::Sessions::BetaManagedAgentsAgentMessageEvent::Type::AGENT_MESSAGE

    # 之后
    event.type == :"agent.message"
    ```

    以哈希形式编写的请求保持不变，并且仍然需要其 `type`：

    ```ruby
    client.beta.sessions.events.send_(
      session.id,
      events: [{type: "user.message", content: [{type: "text", text: "Hello Claude!"}]}]
    )
    ```

    构建模型实例时，`type:` 现在有默认值，因此您可以省略它：`Anthropic::Beta::Sessions::BetaManagedAgentsTextBlock.new(text: "Hello Claude!")`。`client.beta.sessions.resources.add` 的 `type:` 关键字参数同样如此。

    如果您使用 Sorbet 进行类型检查，这些模块的 `Type::OrSymbol` 和 `Type::TaggedSymbol` 别名也随之移除，字段类型为 `Symbol`。`OrSymbol` 也允许 `String`，因此 `.new(type: "text")` 不再能通过类型检查。请删除该关键字参数或传入 `:text`。在 RBS 中，每个类的 `type_` 别名已被移除，字段为字面量，例如 `:text`。
  </Tab>
</Tabs>

### 工具运行器中的客户端压缩已移除

这会影响 TypeScript 和 Ruby SDK。工具运行器中的客户端压缩已被移除，改用服务器端压缩，后者在 API 内部总结对话，而无需客户端额外发送请求。

<Tabs exclude="shell, python, csharp, go, java, php">
  <Tab title="TypeScript">
    `compactionControl` 选项和 `@anthropic-ai/sdk/lib/tools/CompactionControl` 模块已被移除。该选项在 v0.129.0 中已停止工作，当时传入它会抛出 `AnthropicError`。它不在工具运行器的参数类型中，因此 TypeScript 会将其报告为未知属性。

    要继续在达到令牌阈值时自动压缩：

    ```typescript
    // 之前
    const runner = client.beta.messages.toolRunner({
      // ...
      compactionControl: { enabled: true, contextTokenThreshold: 100_000 }
    });

    // 之后
    const runner = client.beta.messages.toolRunner({
      // ...
      betas: ["compact-2026-01-12"],
      context_management: {
        edits: [{ type: "compact_20260112", trigger: { type: "input_tokens", value: 100_000 } }]
      }
    });
    ```

    要自行决定何时压缩对话，请调用 [`runner.compactBeforeNextTurn()`](https://github.com/anthropics/anthropic-sdk-typescript/blob/main/helpers.md#compaction)。
  </Tab>

  <Tab title="Ruby">
    `compaction_control:` 选项已被移除。它在 v1.74.0 中已停止工作，传入它会引发 `ArgumentError`。

    要继续在达到令牌阈值时自动压缩：

    ```ruby
    # 之前
    runner = client.beta.messages.tool_runner(
      # ...
      compaction_control: {enabled: true, context_token_threshold: 100_000}
    )

    # 之后
    runner = client.beta.messages.tool_runner(
      # ...
      betas: ["compact-2026-01-12"],
      context_management: {
        edits: [{type: :compact_20260112, trigger: {type: :input_tokens, value: 100_000}}]
      }
    )
    ```

    要自行决定何时压缩对话，请在运行器上调用 [`#compact_before_next_turn`](https://github.com/anthropics/anthropic-sdk-ruby/blob/main/helpers.md#compact_before_next_turn---compact-the-conversation)。
  </Tab>
</Tabs>

触发阈值必须至少为 50,000 个令牌。有关其他选项（`pause_after_compaction`、自定义 `instructions`），请参阅[压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold)。

### Text Completions 已移除

旧版 Text Completions API 已从 TypeScript、Go、Java 和 Ruby SDK 以及 CLI 中移除。较新的模型和功能不支持它。请改用 [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)。

### 非 beta 的 Models 方法中已移除 `betas`

非 beta 的 Models 方法不再接受 `betas` (csharp, go: `Betas`; java: `.addBeta()`)，`ant models` 命令也不再接受 `--beta`。要发送 beta 标头，请在 `client.beta.models` (csharp, go: `client.Beta.Models`; java: `client.beta().models()`; php: `$client->beta->models`) 上调用相同的方法，或在 CLI 中使用 `ant beta:models`。在 SDK 中，这些方法返回 `BetaModelInfo` (ruby: `Anthropic::Beta::BetaModelInfo`)，而非 beta 方法返回 `ModelInfo` (ruby: `Anthropic::ModelInfo`)。

<Tabs exclude="shell:cURL">
  <Tab title="CLI">
    ```bash
    # 之前
    ant models list --beta "$BETA"

    # 之后
    ant beta:models list --beta "$BETA"
    ```
  </Tab>

  <Tab title="Python">
    `client.models.retrieve()` 和 `client.models.list()` 不再接受 `betas`（传入它会引发 `TypeError`，类型检查器也会标记它），并且 `anthropic.types.ModelListParams` 不再具有 `betas` 键。

    ```python
    # 之前
    page = client.models.list(betas=[beta])

    # 之后
    page = client.beta.models.list(betas=[beta])
    ```
  </Tab>

  <Tab title="TypeScript">
    `Anthropic.ModelRetrieveParams` 和 `Anthropic.ModelListParams` 不再具有 `betas` 属性。

    ```typescript
    // 之前
    const page = await client.models.list({ betas: [beta] });

    // 之后
    const page = await client.beta.models.list({ betas: [beta] });
    ```
  </Tab>

  <Tab title="C#">
    `Anthropic.Models.Models` 中的 `ModelRetrieveParams` 和 `ModelListParams` 不再具有 `Betas` 属性。`client.Beta.Models` 上的方法接受 `Anthropic.Models.Beta.Models` 中同名的类。

    ```csharp
    // 之前
    using Anthropic.Models.Models;

    var page = await client.Models.List(new ModelListParams { Betas = [beta] });

    // 之后
    using Anthropic.Models.Beta.Models;

    var page = await client.Beta.Models.List(new ModelListParams { Betas = [beta] });
    ```
  </Tab>

  <Tab title="Go">
    `ModelGetParams` 和 `ModelListParams` 不再具有 `Betas` 字段。

    ```go
    // 之前
    page, err := client.Models.List(ctx, anthropic.ModelListParams{
    	Betas: []anthropic.AnthropicBeta{beta},
    })

    // 之后
    page, err := client.Beta.Models.List(ctx, anthropic.BetaModelListParams{
    	Betas: []anthropic.AnthropicBeta{beta},
    })
    ```
  </Tab>

  <Tab title="Java">
    `com.anthropic.models.models` 中的 `ModelRetrieveParams` 和 `ModelListParams` 不再具有 `betas()`，它们的构建器也不再具有 `betas` 和 `addBeta`。`client.beta().models()` 上的方法接受 `com.anthropic.models.beta.models` 中同名的类。

    ```java
    // 之前
    import com.anthropic.models.models.ModelListPage;
    import com.anthropic.models.models.ModelListParams;

    ModelListPage page = client.models().list(ModelListParams.builder()
        .addBeta(beta)
        .build());

    // 之后
    import com.anthropic.models.beta.models.ModelListPage;
    import com.anthropic.models.beta.models.ModelListParams;

    ModelListPage page = client.beta().models().list(ModelListParams.builder()
        .addBeta(beta)
        .build());
    ```
  </Tab>

  <Tab title="PHP">
    `$client->models->retrieve()` 和 `$client->models->list()` 不再接受 `betas`（传入它会抛出 `Error: Unknown named parameter $betas`），并且 `Anthropic\Models\ModelRetrieveParams` 和 `Anthropic\Models\ModelListParams` 不再具有 `betas` 属性。

    ```php
    // 之前
    $page = $client->models->list(betas: [$beta]);

    // 之后
    $page = $client->beta->models->list(betas: [$beta]);
    ```

    如果您按位置传递参数，在这两个方法中，`workspaceID` 和 `requestOptions` 都会各自前移一位。
  </Tab>

  <Tab title="Ruby">
    `client.models.retrieve` 和 `client.models.list` 不再接受 `betas:`，并且 `Anthropic::ModelRetrieveParams` 和 `Anthropic::ModelListParams` 不再具有 `betas` 字段。

    ```ruby
    # 之前
    page = client.models.list(betas: [beta])

    # 之后
    page = client.beta.models.list(betas: [beta])
    ```

    向 `client.models` 传入 `betas:` 不会引发错误，但它不再设置 `anthropic-beta` 请求头。
  </Tab>
</Tabs>

### API 密钥以 `Authorization: Bearer` 形式发送

SDK 和 CLI 以 `Authorization: Bearer <key>` 的形式发送您的 API 密钥，而不再放在 `X-Api-Key` 请求头中。因此，请将任何读取或设置 `X-Api-Key` 的代理或网关改为使用 `Authorization`，或者[自行在 `X-Api-Key` 中发送密钥](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#sdk-proxy-x-api-key)。同时，请修改您自己程序中读取 `X-Api-Key` 的代码，例如中间件、日志过滤器或测试桩。Claude API 接受任一请求头中的密钥。

```text wrap
# Before
X-Api-Key: <key>

# After
Authorization: Bearer <key>
```

<Tabs>
  <Tab title="CLI">
    这适用于 `ANTHROPIC_API_KEY` 中的密钥，以及通过管道传给 `--api-key-stdin` 的密钥。

    使用 `--debug` 时，请求日志现在显示 `Authorization: Bearer <REDACTED>`，而以前显示的是 `X-Api-Key: <REDACTED>`。
  </Tab>

  <Tab title="Python">
    `Authorization` 是 `auth_token` 已在使用的请求头。

    这适用于 `Anthropic` 和 `AsyncAnthropic` 的 `api_key` 参数和 `ANTHROPIC_API_KEY` 环境变量，以及 `AnthropicAWS` 的 `api_key` 和 `ANTHROPIC_AWS_API_KEY`。`AnthropicFoundry` 仍以 `x-api-key` 发送其密钥，而 `AnthropicBedrockMantle` 此前就已发送 bearer 令牌。
  </Tab>

  <Tab title="TypeScript">
    `Authorization` 是 `authToken` 选项已在使用的请求头。

    这适用于 `apiKey` 选项、`ANTHROPIC_API_KEY` 环境变量，以及 `@anthropic-ai/aws-sdk` 中的 `apiKey` 和 `ANTHROPIC_AWS_API_KEY`。`@anthropic-ai/foundry-sdk` 中的 `apiKey` 仍以 `X-Api-Key` 发送，而 `@anthropic-ai/bedrock-sdk` 中的 `apiKey` 此前就已是 bearer 令牌。
  </Tab>

  <Tab title="C#">
    `Authorization` 是 `AuthToken` 已在使用的请求头。

    这适用于 `AnthropicClient` 和 `ClientOptions` 上的 `ApiKey` 以及 `ANTHROPIC_API_KEY` 环境变量。在依赖 v13 的 `Anthropic.Aws` 版本中，它也适用于 `AwsClientOptions` 中的 `ApiKey` 和 `ANTHROPIC_AWS_API_KEY`。`AnthropicFoundryApiKeyCredentials` 仍以 `x-api-key` 发送其密钥，而 `MantleAwsClientOptions` 中的 `ApiKey` 此前就已是 bearer 令牌。
  </Tab>

  <Tab title="Go">
    这适用于 `option.WithAPIKey`、`ANTHROPIC_API_KEY` 环境变量以及 `aws.ClientConfig` 中的 `APIKey`（或 `ANTHROPIC_AWS_API_KEY`）。`foundry.ClientConfig` 中的 `APIKey`（或 `ANTHROPIC_FOUNDRY_API_KEY`）仍以 `X-Api-Key` 发送，而 `bedrock.MantleClientConfig` 中的 `APIKey` 此前就已是 bearer 令牌。
  </Tab>

  <Tab title="Java">
    `Authorization` 是 `authToken` 已在使用的请求头。

    这适用于 `AnthropicOkHttpClient`、`AnthropicOkHttpClientAsync` 和 `AnthropicBackend` 构建器上的 `apiKey`、`ANTHROPIC_API_KEY` 环境变量以及 `anthropic.apiKey` 系统属性。`FoundryBackend` 构建器上的 `apiKey` 仍以 `x-api-key` 发送。`AwsBackend`、`BedrockBackend` 和 `BedrockMantleBackend` 构建器上的 `apiKey` 此前就已是 bearer 令牌。
  </Tab>

  <Tab title="PHP">
    `Authorization` 是 `authToken` 已在使用的请求头。

    这适用于 `Anthropic\Client` 的 `apiKey` 参数和 `ANTHROPIC_API_KEY` 环境变量，以及 `Anthropic\Aws\Client` 的 `apiKey` 和 `ANTHROPIC_AWS_API_KEY`。`Anthropic\Foundry\Client` 仍以 `x-api-key` 发送其密钥，而 `Anthropic\Bedrock\MantleClient` 此前就已发送 bearer 令牌。
  </Tab>

  <Tab title="Ruby">
    `Authorization` 是 `auth_token:` 已在使用的请求头。

    这适用于 `Anthropic::Client` 的 `api_key:` 参数和 `ANTHROPIC_API_KEY` 环境变量，以及 `Anthropic::AWSClient` 的 `api_key:` 和 `ANTHROPIC_AWS_API_KEY`。`Anthropic::FoundryClient` 以 `x-api-key` 发送其密钥，而 `Anthropic::BedrockMantleClient` 此前就已发送 bearer 令牌。

    SDK 在跟随重定向到另一个源时会丢弃 `Authorization`，这包括同一主机上从 `http` 到 `https` 的重定向。现在在这种情况下密钥也会被丢弃。
  </Tab>
</Tabs>

#### 如果代理或网关读取 `X-Api-Key`

<Tabs exclude="shell:cURL">
  <Tab title="CLI">
    如果 `--base-url` 或 `ANTHROPIC_BASE_URL` 将 `ant` 指向读取 `X-Api-Key` 的代理或网关，请将其改为从 `Authorization` 读取密钥。`ant` 没有用于添加自定义标头的标志或环境变量，因此它无法发送 `X-Api-Key`。
  </Tab>

  <Tab title="Python">
    在 HTTP 客户端上使用 `auth` 函数将密钥移回原处。它会在客户端发送的每个请求上运行，包括重试：

    ```python
    import httpx2
    from anthropic import Anthropic, DefaultHttpxClient


    def send_x_api_key(request: httpx2.Request) -> httpx2.Request:
        if request.headers.get("Authorization") == f"Bearer {api_key}":
            del request.headers["Authorization"]
            request.headers["X-Api-Key"] = api_key
        return request


    client = Anthropic(api_key=api_key, http_client=DefaultHttpxClient(auth=send_x_api_key))
    ```

    `AsyncAnthropic` 接受相同的函数，写法为 `http_client=DefaultAsyncHttpxClient(auth=send_x_api_key)`。
  </Tab>

  <Tab title="TypeScript">
    自行设置该标头。`Authorization: null` 还可以阻止密钥通过 `Authorization` 标头发出：

    ```typescript
    const client = new Anthropic({
      apiKey,
      defaultHeaders: { "X-Api-Key": apiKey, Authorization: null }
    });
    ```

    如果由代理添加凭据而您的客户端没有凭据，v0 允许您在 `defaultHeaders` 中使用 `"X-Api-Key": null` 或 `Authorization: null` 来表明这一点。v1 只接受 `Authorization: null`。

    环境工作轮询器和工作器会获得自己的密钥，因此它们会在自己的请求中省略 `X-Api-Key` 和 `Authorization` 默认标头。
  </Tab>

  <Tab title="C#">
    使用 `ExtraHeaders` 自行设置该标头。将 `ApiKey` 和 `AuthToken` 设置为 `null` 还可以阻止在 `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` 中找到的凭据被发出：

    ```csharp
    AnthropicClient client = new(
        new ClientOptions
        {
            ApiKey = null,
            AuthToken = null,
            ExtraHeaders = new Dictionary<string, string> { ["X-Api-Key"] = apiKey },
        }
    );
    ```
  </Tab>

  <Tab title="Go">
    自行设置该标头。`option.WithHeaderDel` 还可以阻止在 `ANTHROPIC_API_KEY` 中找到的密钥被发出：

    ```go
    client := anthropic.NewClient(
    	option.WithHeader("X-Api-Key", apiKey),
    	option.WithHeaderDel("Authorization"),
    )
    ```

    `environments.WorkPoller` 和 `environments.EnvironmentWorker` 会获得自己的密钥，因此它们会在自己的请求中省略手动设置的 `X-Api-Key`。
  </Tab>

  <Tab title="Java">
    自行设置该标头，而不是调用 `apiKey`：

    ```java
    AnthropicClient client = AnthropicOkHttpClient.builder()
        .putHeader("X-Api-Key", apiKey)
        .build();
    ```
  </Tab>

  <Tab title="PHP">
    使用中间件将密钥移回原处：

    ```php
    use Anthropic\Client;
    use Psr\Http\Message\RequestInterface;
    use Psr\Http\Message\ResponseInterface;

    $sendXApiKey = fn (RequestInterface $request, callable $next): ResponseInterface => $next(
        $request->withoutHeader('Authorization')->withHeader('X-Api-Key', $apiKey)
    );

    $client = new Client(apiKey: $apiKey, requestOptions: ['middleware' => [$sendXApiKey]]);
    ```

    不要为此使用客户端的 `extraHeaders`。带有自身 `extraHeaders` 的请求会替换客户端的 `extraHeaders`，而 `$client->beta` 下每个需要 beta 标头的端点都带有自身的 `extraHeaders`。
  </Tab>

  <Tab title="Ruby">
    使用中间件将密钥移回原处：

    ```ruby
    send_x_api_key = lambda do |req, nxt|
      next nxt.call(req) unless req.headers["authorization"] == "Bearer #{api_key}"
      nxt.call(req.with(headers: {**req.headers.except("authorization"), "x-api-key" => api_key}))
    end

    client = Anthropic::Client.new(api_key: api_key, middleware: send_x_api_key)
    ```

    第一行会在请求不携带密钥时（例如被重定向到另一个源的请求）不对其做任何处理。

    `Anthropic::AWSClient` 不接受 `middleware:` 参数。请改为在每个请求中传入中间件，写法为 `request_options: {middleware: send_x_api_key}`。
  </Tab>
</Tabs>

#### 如果您在请求上设置或移除了 `X-Api-Key`

此变更影响 Python、TypeScript、Go、PHP 和 Ruby SDK。

如果您此前移除了 `X-Api-Key`，以便请求只发送身份验证令牌，请删除该代码而不是修改它，因为移除 `Authorization` 会移除该令牌。[一个客户端只持有一个凭据](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#sdk-one-credential)，因此请确保它是该令牌：如果代码中未传入任何凭据，且同时设置了 `ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN`，客户端会使用密钥。

<Tabs exclude="shell, csharp, java">
  <Tab title="Python">
    传入 `extra_headers={"X-Api-Key": omit}` 以使请求不携带密钥的代码不再起作用。请改为传入 `extra_headers={"Authorization": omit}`。

    `extra_headers={"X-Api-Key": other_key}` 不再替换客户端的密钥：两个密钥都会被发出。要在单个请求中发送另一个密钥，请传入 `extra_headers={"Authorization": f"Bearer {other_key}"}`。
  </Tab>

  <Tab title="TypeScript">
    请求选项中的 `headers: { "X-Api-Key": otherKey }` 不再替换客户端的密钥：两个密钥都会被发出。要在单个请求中发送另一个密钥，请传入 ``headers: { Authorization: `Bearer $\{otherKey}` }``。
  </Tab>

  <Tab title="Go">
    `option.WithHeaderDel("X-Api-Key")` 不再能使请求不携带密钥。请改为删除 `Authorization`：

    ```go
    // 之前
    client.Messages.New(ctx, params, option.WithHeaderDel("X-Api-Key"))

    // 之后
    client.Messages.New(ctx, params, option.WithHeaderDel("Authorization"))
    ```

    `option.WithHeader("X-Api-Key", otherKey)` 不再替换客户端的密钥：两个密钥都会被发出。要在单个请求中发送另一个密钥，请传入 `option.WithAPIKey(otherKey)`。
  </Tab>

  <Tab title="PHP">
    `'extraHeaders' => ['X-Api-Key' => null]` 不再能使请求不携带密钥。要发送不带凭据的请求，请改为将 `Authorization` 设置为 `null`；如果您是对整个客户端这样做的，则在中间件中移除它。

    `'extraHeaders' => ['X-Api-Key' => $otherKey]` 不再替换客户端的密钥：两个密钥都会被发出。要在单个请求中发送另一个密钥，请将 `Authorization` 设置为 `"Bearer {$otherKey}"`。
  </Tab>

  <Tab title="Ruby">
    `extra_headers: {"x-api-key" => nil}` 不再能使请求不携带密钥。请改为将 `"authorization"` 设置为 `nil`。要在单个请求中发送另一个密钥，请将 `"authorization"` 设置为 `"Bearer #{other_key}"`，而不是设置 `"x-api-key"`。
  </Tab>
</Tabs>

### 一个客户端只持有一个凭据

API 密钥和身份验证令牌现在共用一个请求头，因此一个 SDK 客户端只持有一个凭据。如果代码中未传入任何凭据，它会遵循[凭据优先级](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#credential-precedence)顺序。

<Tabs exclude="shell">
  <Tab title="Python">
    当客户端同时拥有 API 密钥和身份验证令牌时，v1 会将两者分别放在各自的请求头中一并发送。凭据是 `api_key`、`auth_token`、`credentials`、`config` 或 `profile` 之一。

    * **最多传入一个。** 在构造函数中，或在一次 `copy()` 或 `with_options()` 调用中传入两个或更多，会引发 `ValueError`。`None` 不计入。v1 引发的是 `TypeError`，且仅针对 `credentials`、`config` 和 `profile` 中的两个同时传入的情况。
    * **您传入的那个是唯一的凭据。** 与 v1 一样，客户端不会在环境中或磁盘上查找其他凭据。
    * **如果未传入任何凭据，只会使用以下各项中第一个已设置的：** `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_PROFILE` 所指定的配置文件、工作负载身份联合变量、磁盘上的活动配置文件（`active_config` 文件所指定的那个，否则为 `default`）。当前两项同时设置时，客户端会记录一次警告。当设置了 `ANTHROPIC_CONFIG_DIR` 或存在 `active_config` 文件时，v1 会优先使用活动配置文件而不是 federation 变量；如需继续使用它，请设置 `ANTHROPIC_PROFILE`。
    * **`copy()` 和 `with_options()` 会替换凭据。** 您传入的凭据会替换客户端自身的凭据，无论两者是哪种类型。来自旧凭据的配置或配置文件的基础 URL 和请求头会随之移除，并使用新凭据的对应设置。您自己传入的基础 URL 会保留。

    ```python
    client = Anthropic(api_key=key, auth_token=token)

    # 之前：同时发送 `X-Api-Key: <key>` 和 `Authorization: Bearer <token>`
    # 之后：抛出 `ValueError`
    ```

    平台客户端保留各自的规则。`AnthropicAWS` 接受与 `api_key` 同时传入的 `auth_token` 而不报错，并发送密钥。

    [`_client.py`](https://github.com/anthropics/anthropic-sdk-python/blob/main/src/anthropic/_client.py) 中 `Anthropic.__init__` 和 `Anthropic.copy` 的文档字符串提供了详细信息。

    如果您依赖同时发送两者，例如为网关提供令牌、为其后端的 API 提供密钥，请将令牌作为 `auth_token` 传入，并在 `default_headers` 中设置 `X-Api-Key`。让网关在转发请求之前移除其 `Authorization` 请求头。Claude API 要求每个请求只有一个凭据。
  </Tab>

  <Tab title="TypeScript">
    在 v0 中，同时拥有 API 密钥和身份验证令牌的客户端会将两者分别放在各自的请求头中一并发送。凭据是 `apiKey`、`authToken`、`credentials`、`config` 或 `profile` 之一。

    * **最多传入一个。** 在 `new Anthropic()` 中，或在一次 `withOptions()` 调用中传入两个或更多，会抛出 `TypeError`。`null` 和 `undefined` 不计入。
    * **您传入的那个是唯一的凭据。** 客户端不会在环境中或磁盘上查找其他凭据。
    * **如果未传入任何凭据，将使用以下各项中的第一个：** `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_PROFILE` 所指定的配置文件、工作负载身份联合变量、磁盘上的活动配置文件（`active_config` 文件所指定的那个，否则为 `default`）。当前两项同时设置时，客户端会记录一次警告。在 v0 中，活动配置文件排在 federation 变量之前。在 v0 中，指定了不存在的配置文件的 `ANTHROPIC_PROFILE` 会被跳过，而现在第一个请求会抛出 `AnthropicError`。不带参数的 `loadConfig()` 遵循相同的顺序，并抛出相同的错误。
    * **`withOptions` 会替换凭据。** 您传入的凭据会替换您所复制客户端的凭据，因此 `client.withOptions({ authToken })` 不再需要同时传入 `apiKey: null`。
    * **平台客户端只接受其平台的凭据。** AWS、Bedrock 和 Foundry 客户端以及 `AnthropicVertex` 会对它们不用于身份验证的凭据选项抛出 `AnthropicError`，`AnthropicGoogleCloud` 此前已是如此。在 AWS 和 Bedrock 客户端上，`apiKey: null` 现在会清除密钥，使客户端使用 AWS 凭据进行签名。`AnthropicBedrock` 将其密钥保存在 `client.apiKey` 中，而 v0 将其保存在 `client.authToken` 中。

    ```typescript
    // 修改前：发送 API 密钥并忽略 `credentials`
    const client = new Anthropic({ apiKey, credentials });

    // 修改后：上一行会抛出 TypeError
    const client = new Anthropic({ credentials });
    ```

    [`client.ts`](https://github.com/anthropics/anthropic-sdk-typescript/blob/main/src/client.ts) 中 `ClientOptions` 的文档注释说明了每个选项的行为，而这些平台客户端的构造函数的文档注释列出了各自拒绝的选项。

    如果您依赖同时发送两者，例如为网关提供令牌、为其后端的 API 提供密钥，请在 `defaultHeaders` 中设置 `X-Api-Key` 并保留 `authToken`。让网关在转发请求之前移除其 `Authorization` 请求头。Claude API 要求每个请求只有一个凭据。
  </Tab>

  <Tab title="C#">
    在 v12 中，同时拥有 API 密钥和身份验证令牌的客户端会将两者分别放在各自的请求头中一并发送。凭据是 `ApiKey`、`AuthToken` 或 `ClientOptions.Credentials` 之一。

    * **传入两个会报错。** 设置第二个会抛出 `ArgumentException`。`null` 不计入，因此要在您自己构建的 `ClientOptions` 中替换凭据，请先将旧凭据设为 `null`。
    * **传入一个时使用它并忽略环境。** 不会读取任何凭据环境变量或磁盘上的配置文件。同样，当设置了 `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` 时，`AnthropicCredentials.Resolve(profile: "name")` 现在会返回该配置文件的凭据。
    * **未传入任何凭据时，使用环境中的第一个匹配项：** `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_PROFILE`、工作负载身份联合变量，然后是磁盘上的活动配置文件。空的 `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` 视为未设置。当前两项同时设置时，会向 stderr 输出一次警告。客户端在首次使用时才进行查找，因此不存在的 `ANTHROPIC_PROFILE` 现在会在第一个请求时抛出异常，而不是在构造函数中。
    * **`WithOptions` 会替换凭据。** 您在修改器中传入的凭据会替换客户端的凭据，无论其原来是哪种类型，因此 `AuthToken = token` 不再需要同时设置 `ApiKey = null`。如果未传入任何凭据，副本会保留客户端的凭据、`BaseUrl` 和 `ExtraHeaders`，即使修改器返回新的 `ClientOptions` 也是如此。

    ```csharp
    // 之前：两者都会发送，各自位于单独的标头中
    AnthropicClient client = new() { ApiKey = apiKey, AuthToken = token };

    // 之后：上一行会抛出 ArgumentException
    AnthropicClient client = new() { AuthToken = token };
    ```

    [`ClientOptions`](https://github.com/anthropics/anthropic-sdk-csharp/blob/main/src/Anthropic/Core/ClientOptions.cs) 中这三个属性的 XML 文档提供了详细信息。

    云平台客户端保留构建时使用的凭据。Bedrock、Bedrock Mantle、Foundry、Google Cloud 和 AWS 客户端以及 `AnthropicVertexClient` 在构建时确定身份验证方式，其中大多数为此使用 `Authorization` 请求头。这些包中依赖 v13 的版本会忽略您通过 `WithOptions` 设置的 `ApiKey` 或 `AuthToken`。要在 `AnthropicAwsClient` 中使用另一个密钥，请使用 `AwsClientOptions` 中的 `ApiKey` 构建一个新客户端。

    如果您依赖同时发送两者，例如为网关提供令牌、为其后端的 API 提供密钥，请通过 `ExtraHeaders` 设置 `X-Api-Key` 并保留 `AuthToken`。让网关在转发请求之前移除其 `Authorization` 请求头。Claude API 要求每个请求只有一个凭据。
  </Tab>

  <Tab title="Go">
    在 v1 中，同时拥有 API 密钥和身份验证令牌的客户端会将两者分别放在各自的请求头中一并发送。凭据来自 `option.WithAPIKey`、`option.WithAuthToken`、`option.WithConfig`、`option.WithProfile` 或 `option.WithFederationTokenProvider`。

    * **向 `anthropic.NewClient` 传入其中两个会报错。** `NewClient` 不返回错误，因此客户端发出的每个请求都会因此失败。`NewClient` 已经应用了 `anthropic.DefaultClientOptions()`，因此不要再额外传入其结果：否则它在环境中找到的凭据会被视为您传入的凭据。
    * **传入其中一个时按原样使用。** 不会从环境或磁盘上的配置文件中读取任何凭据。在 v1 中，`ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN` 优先于后三者。
    * **如果一个都未传入，将使用以下各项中第一个已设置的，** 顺序与 v1 相同：`ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_PROFILE`、工作负载身份联合变量、磁盘上的活动配置文件。当前两项同时设置时，SDK 会记录一次警告。
    * **在请求上传入其中一个，会在该请求中替换客户端的凭据，** 因此 `option.WithAuthToken` 不再需要同时传入 `option.WithHeaderDel("X-Api-Key")`。

    ```go
    // 之前：会发送 API 密钥并记录一条警告
    client := anthropic.NewClient(
    	option.WithAPIKey(apiKey),
    	option.WithProfile("work"),
    )

    // 之后：上述客户端的每个请求都会失败，因此请传入您想使用的那个
    client := anthropic.NewClient(option.WithAPIKey(apiKey))
    ```

    v1 记录的警告已被移除，因此用于抑制该警告的 `option.WithConfigQuiet` 现在与 `option.WithConfig` 的作用相同，并已弃用。

    [`client.go`](https://github.com/anthropics/anthropic-sdk-go/blob/main/client.go) 中 `anthropic.NewClient` 和 `anthropic.DefaultClientOptions` 的文档注释提供了详细信息。

    如果您依赖同时发送两者，例如为网关提供令牌、为其后端的 API 提供密钥，请通过 `option.WithHeader` 设置 `X-Api-Key` 并保留 `option.WithAuthToken`。让网关在转发请求之前移除其 `Authorization` 请求头。Claude API 要求每个请求只有一个凭据。

    `ANTHROPIC_CUSTOM_HEADERS` 和 `ANTHROPIC_WEBHOOK_SIGNING_KEY` 现在适用于 `anthropic.NewClient` 构建的每个客户端。在 v1 中，只有当环境和磁盘上的配置文件都不包含凭据时才会读取它们。
  </Tab>

  <Tab title="Java">
    在 v2 中，同时拥有 API 密钥和身份验证令牌的客户端会将两者分别放在各自的请求头中一并发送。凭据是 `AnthropicOkHttpClient`、`AnthropicOkHttpClientAsync` 和 `AnthropicBackend` 构建器上的 `apiKey`、`authToken` 或 `configurationProvider` 之一。

    * **最多设置一个。** 如果在一个构建器上设置了两个，`build()` 会抛出指明它们的 `IllegalStateException`。`null` 或空 `Optional` 不计入。
    * **您设置的那个是唯一的凭据。** 无论在之前还是之后调用，`fromEnv()` 都不会从环境或磁盘上的配置文件中获取凭据。在 v2 中，`fromEnv()` 找到的密钥或令牌可能会被一并发送，或取而代之被发送。
    * **如果未设置任何凭据，`fromEnv()` 会使用它找到的以下各项中的第一个：** `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_PROFILE`、工作负载身份联合变量、磁盘上的活动配置文件。当前两项同时设置时，会向 `System.err` 输出一次警告。`anthropic.apiKey` 和 `anthropic.authToken` 系统属性视为环境的一部分。空的密钥或令牌（无论在变量还是系统属性中）视为未设置。在 v2 中，`fromEnv()` 会将空值用作密钥或令牌。

    ```java
    // 之前：两者都会被发送。在 v3 中，build() 会抛出 IllegalStateException
    AnthropicClient client = AnthropicOkHttpClient.builder()
        .apiKey(apiKey)
        .authToken(authToken)
        .build();

    // 之后
    AnthropicClient client = AnthropicOkHttpClient.builder()
        .apiKey(apiKey)
        .build();
    ```

    这些构建器上 `fromEnv()` 的文档提供了详细信息。

    如果您依赖同时发送两者，例如为网关提供令牌、为其后端的 API 提供密钥，请自行设置这两个请求头：

    ```java
    AnthropicClient client = AnthropicOkHttpClient.builder()
        .putHeader("Authorization", "Bearer " + gatewayToken)
        .putHeader("X-Api-Key", apiKey)
        .build();
    ```

    让网关在转发请求之前移除其 `Authorization` 请求头。Claude API 要求每个请求只有一个凭据。
  </Tab>

  <Tab title="PHP">
    当客户端同时拥有 API 密钥和身份验证令牌时，早期版本会将两者分别放在各自的请求头中一并发送。`Anthropic\Client` 接受一个凭据，来自 `apiKey`、`authToken` 或 `credentials`：

    * 传入其中多个会抛出 `\InvalidArgumentException`。只有 `null` 视为未传入。空字符串视为已传入，因此 `new Client(apiKey: '')` 不会发送任何凭据。
    * 传入一个时使用它，且不会从环境或磁盘上的配置文件中读取任何凭据。在设置了 `ANTHROPIC_API_KEY` 的情况下，`new Client(authToken: $token)` 会发送该令牌。早期版本会同时发送环境中的密钥。
    * 未传入任何凭据时，使用以下各项中第一个已设置的：`ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_PROFILE` 所指定的配置文件、工作负载身份联合变量、磁盘上的活动配置文件（`active_config` 文件所指定的那个，否则为 `default`）。早期版本会在 federation 变量之前使用活动配置文件，并且在没有 `active_config` 文件时不使用任何配置文件。它们还会跳过指定了不存在或无效配置文件的 `ANTHROPIC_PROFILE`。对于这种情况，`new Client()` 现在会抛出 `Anthropic\Core\Exceptions\AnthropicException`。在同时设置了 `ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN` 的情况下，客户端会发送密钥，且其 `authToken` 属性为空。以这种方式构建的第一个客户端还会通过 `error_log()` 写入一条警告，每个进程一次，在 Web 服务器上即意味着每个请求一次：`Anthropic: ANTHROPIC_API_KEY and ANTHROPIC_AUTH_TOKEN are both set. Using ANTHROPIC_API_KEY.` 要停止该警告，请取消设置您不使用的变量。

    ```php
    // 之前：将密钥作为 X-Api-Key 发送，将令牌作为 Authorization 发送。
    // 之后：抛出错误 "Pass at most one of `apiKey`, `authToken`, or `credentials`. Received `apiKey`, `authToken`."
    $client = new Client(apiKey: $key, authToken: $token);
    ```

    如果您依赖同时发送两者，例如为网关提供令牌、为其后端的 API 提供密钥，请将令牌作为 `authToken` 传入，并在中间件中添加 `X-Api-Key`，中间件可参照[如果代理或网关读取 `X-Api-Key`](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#sdk-proxy-x-api-key) 中的示例，但不包括 `withoutHeader` 调用。让网关在转发请求之前移除其 `Authorization` 请求头。Claude API 要求每个请求只有一个凭据。
  </Tab>

  <Tab title="Ruby">
    `Anthropic::Client.new` 的凭据参数是 `api_key:`、`auth_token:`、`credentials:` 和 `config:`。`nil` 参数视为未传入。平台客户端保留各自的规则。

    * **传入两个或更多：** `Anthropic::Client.new` 会引发 `ArgumentError`。早期版本仅在同时传入 `credentials:` 和 `config:` 时引发错误，其他情况下如果有密钥就发送密钥，否则发送令牌。
    * **传入一个：** 客户端使用它，不会回退到环境或磁盘上配置文件中的凭据。
    * **未传入任何凭据：** 客户端使用它找到的以下各项中的第一个：`ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_PROFILE`、工作负载身份联合变量、磁盘上的活动配置文件。顺序没有改变，但在同时设置了 `ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN` 的情况下，`client.auth_token` 现在为 `nil`。空的 `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` 现在视为未设置，而早期版本中空的 `ANTHROPIC_API_KEY` 会遮蔽令牌及其后的所有项。

    当未传入凭据且同时设置了 `ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN` 时，SDK 还会通过 `Kernel#warn` 在标准错误输出上为每个进程警告一次：`Anthropic: ANTHROPIC_API_KEY and ANTHROPIC_AUTH_TOKEN are both set. Using ANTHROPIC_API_KEY.` 取消设置您不使用的变量即可消除该警告。

    ```ruby
    # 之前：发送密钥并忽略令牌。
    # 之后：ArgumentError, "Pass at most one of `api_key:`, `auth_token:`, `credentials:`, or `config:`. Received `api_key:`, `auth_token:`."
    Anthropic::Client.new(api_key: api_key, auth_token: auth_token)

    # 只传入您希望发送的那一个。
    Anthropic::Client.new(api_key: api_key)
    ```
  </Tab>
</Tabs>

### 仅影响某一个 SDK 或 CLI 的变更

以下每项变更仅影响某一个 SDK 或 CLI。

<Tabs exclude="shell:cURL, python, typescript">
  <Tab title="CLI">
    **`--api-key` 和 `--auth-token` 已被移除。** `ant` 不再接受以标志值形式提供的 API 密钥或身份验证令牌，因为命令行上的值会出现在 shell 历史记录和进程列表中。自 v1.27.0 起，这两个标志都会打印弃用警告。仍传入其中之一的命令会以 `flag provided but not defined` 终止。

    | 已移除                    | 改用                                                       |
    | ---------------------- | -------------------------------------------------------- |
    | `--api-key <key>`      | `ANTHROPIC_API_KEY`、`--api-key-stdin` 或 `ant auth login` |
    | `--auth-token <token>` | `ANTHROPIC_AUTH_TOKEN` 或 `--auth-token-stdin`            |

    ```bash
    # 之前
    ant --api-key "$KEY" models list

    # 之后，从环境变量读取
    export ANTHROPIC_API_KEY="$KEY"
    ant models list

    # 之后，从密钥管理器读取
    op read op://vault/anthropic/api-key | ant --api-key-stdin models list
    ```

    `--api-key-stdin` 和 `--auth-token-stdin` 会读取全部标准输入，因此使用其中之一的命令需要通过标志获取请求体。

    `ant auth status`、`ant apply` 的首次运行提示以及关于多个身份验证来源的说明现在将这些凭据称为 `ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN`，而以前打印的是 `--api-key / ANTHROPIC_API_KEY` 和 `--auth-token / ANTHROPIC_AUTH_TOKEN`。
  </Tab>

  <Tab title="C#">
    **联合类型的 `Type` 是一个枚举。** 在 v12 中，联合类的 `Type` 属性（例如 `ContentBlock.Type` 或 `RawMessageStreamEvent.Type`）是 `JsonElement`。在 v13 中，它是 `ApiEnum<string, T>`，其中 `T` 是一个新枚举，每个变体对应一个成员，例如 `ContentBlockType`。少数联合类型通过 `Name` 或 `Command` 来区分，例如 `BetaMemoryTool20250818Command.Command`，这些属性也以相同方式变化。变体类的 `Type`（例如 `TextBlock.Type`）仍然是 `JsonElement`。

    ```csharp
    // 之前
    if (block.Type.GetString() == "tool_use") { }

    // 之后
    if (block.Type.Value() == ContentBlockType.ToolUse) { }
    ```

    `block.Type.Raw()` 返回字符串，包括此版本 SDK 中没有对应枚举成员的字符串。

    **`IChatClient` 忽略采样选项。** `IChatClient` 适配器（`AsIChatClient()`）现在会忽略 `ChatOptions.Temperature`、`ChatOptions.TopK` 和 `ChatOptions.TopP`。请参阅[采样参数已移除](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#sampling-parameters)。
  </Tab>

  <Tab title="Go">
    **需要 Go 1.25 或更高版本。**

    **`vertex.DefaultVersion` 和 `bedrock.DefaultVersion` 已被移除。**
  </Tab>

  <Tab title="Java">
    **现在返回 `Optional` 的 getter。** `AnthropicBackend` 上的 `apiKey()` 和 `authToken()` 现在返回 `Optional<String>`，与 `AwsBackend` 和 `BedrockMantleBackend` 上的 `apiKey()` 此前的行为一致。在 v2 中，它们各自返回 `String`，当 `AnthropicBackend` 没有该类凭据时为 `null`。

    ```java
    // 之前
    String apiKey = backend.apiKey();
    if (apiKey != null) {}

    // 之后
    Optional<String> apiKey = backend.apiKey();
    if (apiKey.isPresent()) {}
    ```

    以下 getter 现在也返回 `Optional`。在 v2 中，它们各自直接返回值本身，没有值时为 `null`：

    * `BedrockBackend` 上的 `apiKey()` 和 `awsCredentialsProvider()`
    * `McpResourceFile` 上的 `filename()` 和 `mimeType()`
    * `CredentialSource` 上的 `detail()`
    * `HttpRequest` 上的 `body()`
    * `FoundryBackend` 上的 `resource()`，原为 `getResource()`
    * `RequestOptions` 上的 `responseValidation()`、`timeout()`、`followRedirects()`、`fallbackState()` 和 `openTelemetry()`，原为 `getResponseValidation()`、`getTimeout()`、`getFollowRedirects()`、`getFallbackState()` 和 `getOpenTelemetry()`

    ```java
    // 之前
    String filename = file.filename();
    if (filename != null) {}
    String resource = backend.getResource();

    // 之后
    Optional<String> filename = file.filename();
    if (filename.isPresent()) {}
    Optional<String> resource = backend.resource();
    ```

    `FoundryBackend` 上的 `getBaseUrl()` 已被移除。请调用 `baseUrl()`，它返回请求发往的 URL。该 URL 是您设置的基础 URL，或根据资源构建的 URL，并以 `/anthropic` 结尾。

    `AnthropicBackend` 上的 `getBaseUrl()` 也已被移除。请调用 `baseUrl()`，它返回相同的值。

    **`HttpRequest` 需要基础 URL。** 如果您自己构建 `HttpRequest`（例如在拦截器的测试中），请设置其基础 URL。没有基础 URL 时，`build()` 现在会抛出 `IllegalStateException`。在 v2 中，它会返回一个 `baseUrl()` 为 `null` 的请求。SDK 构建的请求此前就已带有基础 URL。

    ```java
    // 之前
    HttpRequest request = HttpRequest.builder()
        .method(HttpMethod.GET)
        .build();

    // 之后
    HttpRequest request = HttpRequest.builder()
        .method(HttpMethod.GET)
        .baseUrl("https://api.anthropic.com")
        .build();
    ```

    `baseUrl()` 仍然返回 `String`，现在永远不会为 `null`。在构建器上，`baseUrl(Optional<String>)` 已被移除，且 `baseUrl(String)` 不再接受 `null`。

    **拦截器可以看到 `anthropic-version`。** 客户端现在会在拦截器运行之前添加默认的 `anthropic-version` 请求头。在 v2 中，后端是在拦截器之后添加它的。如果您的拦截器设置了自己的版本，请调用 `replaceHeaders`，因为 `putHeader` 现在会添加第二个值：

    ```java
    // 之前
    request.toBuilder()
        .putHeader("anthropic-version", version)
        .build();

    // 之后
    request.toBuilder()
        .replaceHeaders("anthropic-version", version)
        .build();
    ```
  </Tab>

  <Tab title="PHP">
    **您传入的枚举成员在读回时是其值。** `with()` 以及 `withAllowedCallers()` 等 setter 仍然接受枚举成员或其值。早期版本会按您传入的原样保留枚举成员。v1 保留的是枚举成员的值，这正是属性类型此前已声明的内容。SDK 根据响应构建的对象不受影响，SDK 发送的 JSON 也相同。如果您读回某个属性并将其与枚举成员进行比较，比较结果现在为 false 且不会抛出任何异常，因此请将其与枚举成员的值进行比较：

    ```php
    $tool = CodeExecutionTool20250825::with(allowedCallers: [AllowedCaller::DIRECT]);

    // 之前
    in_array(AllowedCaller::DIRECT, $tool->allowedCallers, true);

    // 之后
    in_array(AllowedCaller::DIRECT->value, $tool->allowedCallers, true);
    ```
  </Tab>

  <Tab title="Ruby">
    **`Anthropic::VertexClient::DEFAULT_VERSION` 和 `Anthropic::BedrockClient::DEFAULT_VERSION` 已被移除。**
  </Tab>
</Tabs>

### 许可证变更为 Apache 2.0

从[切换到 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#upgrade) 中列出的版本开始，SDK 和 CLI 采用 Apache License, Version 2.0 许可。早期版本仍采用 MIT License。无需更改任何代码。如果您跟踪依赖项的许可证，请更新相应条目。

* **TypeScript：** 平台包（`@anthropic-ai/bedrock-sdk`、`@anthropic-ai/vertex-sdk`、`@anthropic-ai/foundry-sdk`、`@anthropic-ai/aws-sdk` 和 `@anthropic-ai/google-cloud-sdk`）在各自的 1.0.0 版本中变更许可证。
* **C#：** 其他包（`Anthropic.Bedrock`、`Anthropic.Vertex`、`Anthropic.Foundry`、`Anthropic.Aws`、`Anthropic.GoogleCloud` 和 `Anthropic.Mcp`）在各自的 1.0.0 版本中变更许可证。
* **Java：** 此变更涵盖所有 `com.anthropic` 构件。
