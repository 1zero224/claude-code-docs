---
title: 在 Console 中构建
url: https://platform.claude.com/docs/zh-CN/managed-agents/onboarding
description: 在 Claude Console 中以可视化方式创建、测试和迭代智能体，然后通过 API 从您的代码中运行它们。
featureMetadata:
  topic:
    title: Managed Agents
    url: https://platform.claude.com/docs/en/managed-agents/overview
  status: beta
  betaHeader: managed-agents-2026-04-01
  zdr:
    eligibility: not-eligible
    note: 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。
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
---

[Claude Console](https://platform.claude.com/workspaces/default/agent-quickstart/) 提供了一个用于创建和配置智能体的可视化界面。它让您可以在编写代码之前以交互方式迭代配置。

## 如何构建智能体

Claude Console 中的[可视化界面](https://platform.claude.com/workspaces/default/agent-quickstart/)会引导您逐一完成智能体定义中的每个字段：

* **模型和系统提示：** 选择一个模型，并在全宽编辑器中编写 “system prompt”（系统提示）。
* **MCP 服务器：** 通过 URL 添加远程 MCP 服务器，并对您的智能体进行身份验证，使其能够代表您执行操作。
* **工具：** 使用预构建的智能体工具集和 MCP 工具扩展您的智能体的能力。
* **Skills：** 从您组织的库中附加 Anthropic 技能或自定义技能。

在您进行配置时，Console 会显示等效的 API 请求，以便您在满意后将其复制到您的代码中。

## 测试智能体

Console 包含一个内联会话运行器。配置好智能体后，您可以直接启动测试会话、发送消息并观察事件流，而无需离开页面。这是检查您的系统提示和工具选择是否产生预期行为的最快方式。

## 从 Console 到您的代码库

一旦您的智能体按预期工作：

1. 从 Console 复制智能体 ID 和[环境 ID](https://platform.claude.com/docs/zh-CN/managed-agents/environments)。
2. 在[创建会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)时在您的代码中引用它们：

<CodeGroup>
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "agent_01J8XkN5uT3vHpLqRfWdY2",
      "environment_id": "env_01K2mPsT7hNwR4jXuLvCqD8",
      "title": "My first session"
    }'
  ```

  ```bash CLI
  ant beta:sessions create \
    --agent agent_01J8XkN5uT3vHpLqRfWdY2 \
    --environment-id env_01K2mPsT7hNwR4jXuLvCqD8 \
    --title "My first session"
  ```

  ```python Python
  session = client.beta.sessions.create(
      agent="agent_01J8XkN5uT3vHpLqRfWdY2",
      environment_id="env_01K2mPsT7hNwR4jXuLvCqD8",
      title="My first session",
  )
  ```

  ```typescript TypeScript
  const session = await client.beta.sessions.create({
    agent: "agent_01J8XkN5uT3vHpLqRfWdY2",
    environment_id: "env_01K2mPsT7hNwR4jXuLvCqD8",
    title: "My first session"
  });
  ```

  ```csharp C#
  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = "agent_01J8XkN5uT3vHpLqRfWdY2",
      EnvironmentID = "env_01K2mPsT7hNwR4jXuLvCqD8",
      Title = "My first session",
  });
  ```

  ```go Go
  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfString: anthropic.String("agent_01J8XkN5uT3vHpLqRfWdY2"),
  	},
  	EnvironmentID: "env_01K2mPsT7hNwR4jXuLvCqD8",
  	Title:         anthropic.String("My first session"),
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var session = client.beta().sessions().create(
      SessionCreateParams.builder()
          .agent("agent_01J8XkN5uT3vHpLqRfWdY2")
          .environmentId("env_01K2mPsT7hNwR4jXuLvCqD8")
          .title("My first session")
          .build()
  );
  ```

  ```php PHP
  $session = $client->beta->sessions->create(
      agent: 'agent_01J8XkN5uT3vHpLqRfWdY2',
      environmentID: 'env_01K2mPsT7hNwR4jXuLvCqD8',
      title: 'My first session',
  );
  ```

  ```ruby Ruby
  session = client.beta.sessions.create(
    agent: "agent_01J8XkN5uT3vHpLqRfWdY2",
    environment_id: "env_01K2mPsT7hNwR4jXuLvCqD8",
    title: "My first session"
  )
  ```
</CodeGroup>

## 可用性

Console 智能体构建器在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `managed-agents-2026-04-01`

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                         | HIPAA                             |
| ---- | ------------------------------------------------------------- | --------------------------------- |
| 状态   | 不具备资格                                                         | 不具备资格                             |
| 详细信息 | 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。 | 适用于所有 Managed Agents 子功能，包括自托管沙箱。 |

适用的端点：

* `/v1/agents`
* `/v1/sessions`
* `/v1/environments`
