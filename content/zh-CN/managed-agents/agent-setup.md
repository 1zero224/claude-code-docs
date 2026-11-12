---
title: 定义您的智能体
url: https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup
description: 创建可复用、带版本控制的智能体配置。
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
    Claude Models on Google Cloud (Anthropic managed): beta
---

“Agent”（智能体）是一种可复用、带版本控制的配置，用于定义角色设定和能力。它将模型、"system prompt"（系统提示）、工具、MCP 服务器（"Model Context Protocol"，即 MCP）以及技能打包在一起，这些共同决定了 Claude 在会话期间的行为方式。

只需将智能体创建一次作为可复用资源，之后每次[启动会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)时通过 ID 引用它即可。智能体带有版本控制，更便于在众多会话之间进行管理。

## 智能体配置字段

| 字段                                                                              | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name` (csharp, go: `Name`)                                                     | 必填。智能体的可读名称。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `model` (csharp, go: `Model`)                                                   | 必填。为智能体提供支持的 Claude [模型](https://platform.claude.com/docs/zh-CN/models/overview)。接受模型 ID 字符串或对象，例如 `{"id": "claude-opus-5-5"}`。支持 Claude 4.5 及更高版本的模型。对象形式还接受 `speed` (csharp, go: `Speed`)、`effort` (csharp, go: `Effort`) 和 `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`) 字段；请参阅[创建智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#create-an-agent)下的提示、[努力程度级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#effort-levels)以及[固定推理地理位置](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#pin-the-inference-geo)。 |
| `system` (csharp, go: `System`; ruby: `system_`)                                | 定义智能体行为和角色设定的[系统提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices#give-claude-a-role)。系统提示不同于[用户消息](https://platform.claude.com/docs/zh-CN/managed-agents/reference#event-types)，后者应描述要完成的工作。                                                                                                                                                                                                                                                                                                                                              |
| `tools` (csharp, go: `Tools`)                                                   | 智能体可用的工具。包括[预构建的智能体工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools)、[MCP 工具](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector)和[自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#custom-tools)。                                                                                                                                                                                                                                                                                                                                                     |
| `mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) | 提供标准化第三方能力的 [MCP 服务器](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `skills` (csharp, go: `Skills`)                                                 | 通过渐进式披露提供特定领域上下文的[技能](https://platform.claude.com/docs/zh-CN/managed-agents/skills)。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `multiagent` (csharp, go: `Multiagent`)                                         | 允许智能体将工作交给其他智能体（作为子智能体或在[动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#dynamic-workflows)中），并可选择咨询[顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)模型。使用 `multiagent_20261001` 类型时，除非您将其禁用，否则动态工作流默认开启。请参阅[多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)。                                                                                                                                                                                               |
| `description` (csharp, go: `Description`)                                       | 对智能体功能的描述。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `metadata` (csharp, go: `Metadata`)                                             | 供您自行跟踪使用的任意键值对。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |

您还可以针对单个会话覆盖 `model` (csharp, go: `Model`)、`system` (csharp, go: `System`; ruby: `system_`)、`tools` (csharp, go: `Tools`)、`mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`) 和 `skills` (csharp, go: `Skills`)，而无需更改智能体本身。`model` (csharp, go: `Model`) 覆盖会完整替换智能体的 `model` (csharp, go: `Model`) 对象，因此智能体自身的 `effort` (csharp, go: `Effort`) 不会被沿用。若要以特定 effort 级别运行会话，请在覆盖的 `model` (csharp, go: `Model`) 对象中设置 `effort` (csharp, go: `Effort`)。请参阅[为会话覆盖智能体配置](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#override-agent-configuration-for-a-session)。

## 创建智能体

以下示例定义了一个编码智能体，它使用最新的 Claude Opus 模型，并可访问预构建的智能体工具集。该工具集使智能体能够编写代码、读取文件、搜索网络等。有关受支持工具的完整列表，请参阅[工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools)。

示例使用 curl、`ant` CLI 或 SDK。如果您尚未设置其中任何一种，[快速入门](https://platform.claude.com/docs/zh-CN/managed-agents/quickstart#before-you-begin)介绍了安装和客户端设置。

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Coding Assistant",
      "model": "claude-opus-5-5",
      "system": "You are a helpful coding agent.",
      "tools": [{"type": "agent_toolset_20260401"}]
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply coding-assistant.md
    ```

    <File filename="coding-assistant.md">
      ```markdown
      ---
      name: Coding Assistant
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
      ---

      You are a helpful coding agent.
      ```
    </File>

    [`ant apply`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/apply) 会根据 `coding-assistant.md` 创建智能体，打印其 ID，并将其记录在 `claude-lock.json` 中。请提交 `claude-lock.json`，这样下一次 `ant apply` 会更新此智能体，而不是再创建一个新的。
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="Coding Assistant",
      model="claude-opus-5-5",
      system="You are a helpful coding agent.",
      tools=[
          {"type": "agent_toolset_20260401"},
      ],
  )
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "Coding Assistant",
    model: "claude-opus-5-5",
    system: "You are a helpful coding agent.",
    tools: [{ type: "agent_toolset_20260401" }]
  });
  ```

  ```csharp C#
  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Coding Assistant",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      System = "You are a helpful coding agent.",
      Tools = [new BetaManagedAgentsAgentToolset20260401Params()],
  });
  ```

  ```go Go
  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name: "Coding Assistant",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  	},
  	System: anthropic.String("You are a helpful coding agent."),
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{{
  		OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{},
  	}},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var agent = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("Coding Assistant")
          .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
          .system("You are a helpful coding agent.")
          .addTool(BetaManagedAgentsAgentToolset20260401Params.builder().build())
          .build()
  );
  ```

  ```php PHP
  $agent = $client->beta->agents->create(
      name: 'Coding Assistant',
      model: 'claude-opus-5-5',
      system: 'You are a helpful coding agent.',
      tools: [
          BetaManagedAgentsAgentToolset20260401Params::with(),
      ],
  );
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Coding Assistant",
    model: Anthropic::Beta::BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
    system_: "You are a helpful coding agent.",
    tools: [{type: "agent_toolset_20260401"}]
  )
  ```
</CodeGroup>

响应会回显您的配置，并添加 `id`、`type`、`version`、`created_at`、`updated_at` 和 `archived_at` 字段，同时用默认值填充您省略的 `model` 字段（例如 `effort`）。`version` 从 1 开始，每次更新更改智能体时递增。

```json
{
  "id": "agent_01HqR2k7vXbZ9mNpL3wYcT8f",
  "type": "agent",
  "name": "Coding Assistant",
  "model": {
    "id": "claude-opus-5-5",
    "effort": { "type": "medium" },
    "speed": "standard"
  },
  "system": "You are a helpful coding agent.",
  "description": null,
  "tools": [
    {
      "type": "agent_toolset_20260401",
      "default_config": {
        "permission_policy": { "type": "always_allow" }
      }
    }
  ],
  "skills": [],
  "mcp_servers": [],
  "multiagent": null,
  "metadata": {},
  "version": 1,
  "created_at": "2026-04-03T18:24:10.412Z",
  "updated_at": "2026-04-03T18:24:10.412Z",
  "archived_at": null
}
```

工具集上的 `default_config` 显示了其默认的[权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies) `always_allow`，除非您另行配置，否则将应用该策略。

<Tip>
  要使用[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)，请将 `model` (csharp, go: `Model`) 作为对象传递，例如：`{"id": "claude-opus-5-5", "speed": "fast"}`。Claude Opus 4.8 上的快速模式已弃用。有关在 Claude Managed Agents 上支持快速模式的模型，请参阅快速模式页面的[支持的模型](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode#supported-models)。
</Tip>

<Tip>
  要设置模型的 effort 级别，请将 `model` (csharp, go: `Model`) 作为对象传递，例如：`{"id": "claude-opus-5-5", "effort": "high"}`。`effort` (csharp, go: `Effort`) 字段接受级别字符串（`low`、`medium`、`high`、`xhigh` 或 `max`）或诸如 `{"type": "high"}` 的对象。有关各级别的作用，请参阅 [Effort 级别](https://platform.claude.com/docs/zh-CN/build-with-claude/effort#effort-levels)。
</Tip>

### 固定推理地理位置

与 `speed` (csharp, go: `Speed`) 和 `effort` (csharp, go: `Effort`) 一样，`inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`) 通过 `model` (csharp, go: `Model`) 的对象形式进行设置：将 `model` (csharp, go: `Model`) 作为对象传递，并在 `id` (csharp, go: `ID`) 旁边设置 `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`)。该字段接受 `"us"` 或 `"global"`。未设置时，每个模型请求在处理时都遵循工作区当时的默认推理地理位置。有关工作区级别的地理位置控制和定价，请参阅[数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)。

以下示例将智能体固定到美国推理。响应会在智能体的 `model` (csharp, go: `Model`) 对象中包含 `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`) 值：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Geo-pinned assistant",
      "model": {"id": "claude-opus-5-5", "inference_geo": "us"},
      "system": "You are a helpful assistant."
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply geo-pinned-assistant.md
    ```

    <File filename="geo-pinned-assistant.md">
      ```markdown
      ---
      name: Geo-pinned assistant
      model:
        id: claude-opus-5-5
        inference_geo: us
      ---

      You are a helpful assistant.
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="Geo-pinned assistant",
      model={
          "id": "claude-opus-5-5",
          "inference_geo": "us",
      },
      system="You are a helpful assistant.",
  )

  print(f"Inference geo: {agent.model.inference_geo}")
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "Geo-pinned assistant",
    model: { id: "claude-opus-5-5", inference_geo: "us" },
    system: "You are a helpful assistant."
  });

  console.log(`Inference geo: ${agent.model.inference_geo}`);
  ```

  ```csharp C#
  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Geo-pinned assistant",
      Model = new BetaManagedAgentsModelConfigParams
      {
          ID = BetaManagedAgentsModel.ClaudeOpus5_5,
          InferenceGeo = "us",
      },
      System = "You are a helpful assistant.",
  });

  Console.WriteLine($"Inference geo: {agent.Model.InferenceGeo}");
  ```

  ```go Go
  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name: "Geo-pinned assistant",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{
  		ID:           anthropic.BetaManagedAgentsModelClaudeOpus5_5,
  		InferenceGeo: anthropic.String("us"),
  	},
  	System: anthropic.String("You are a helpful assistant."),
  })
  if err != nil {
  	panic(err)
  }

  fmt.Printf("Inference geo: %s\n", agent.Model.InferenceGeo)
  ```

  ```java Java
  var agent = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("Geo-pinned assistant")
          .model(
              BetaManagedAgentsModelConfigParams.builder()
                  .id(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
                  .inferenceGeo("us")
                  .build()
          )
          .system("You are a helpful assistant.")
          .build()
  );

  IO.println("Inference geo: " + agent.model().inferenceGeo().orElseThrow());
  ```

  ```php PHP
  $agent = $client->beta->agents->create(
      name: 'Geo-pinned assistant',
      model: BetaManagedAgentsModelConfigParams::with(
          id: 'claude-opus-5-5',
          inferenceGeo: 'us',
      ),
      system: 'You are a helpful assistant.',
  );

  echo "Inference geo: {$agent->model->inferenceGeo}\n";
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Geo-pinned assistant",
    model: {id: "claude-opus-5-5", inference_geo: "us"},
    system_: "You are a helpful assistant."
  )

  puts "Inference geo: #{agent.model.inference_geo}"
  ```
</CodeGroup>

`inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`) 固定值会在保存智能体时、基于该智能体创建会话时以及会话处理的每一轮中，根据工作区的 [`allowed_inference_geos`](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency#workspace-level-restrictions) 进行验证。如果工作区允许列表收窄，导致某个固定值不再被允许，则无法再基于该智能体创建新会话，正在运行的会话也会拒绝后续轮次；固定值永远不会被豁免，因为工作区依赖它们来满足合规和数据驻留要求。

在不支持地理推理固定的模型上设置 `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`) 会返回 400 错误；有关支持该功能的模型，请参阅[模型可用性](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency#model-availability)。在 `multiagent` (csharp, go: `Multiagent`) 配置中，该智能体的固定值以及 `subagents` (csharp, go: `Subagents`) 或 `workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中列出的每个智能体的固定值，必须全部设置为相同的值或全部不设置；请参阅[多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)。若要稍后更改或清除固定值，请更新智能体的 `model` (csharp, go: `Model`) 对象；提供不含 `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`) 的 `model` (csharp, go: `Model`) 会将其清除，如[更新语义](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#update-semantics)中所述。

## 更新智能体

当配置发生变化时，更新智能体会生成一个新版本。`version` (csharp, go: `Version`) 字段是可选的：提供它可实现 “optimistic concurrency”（乐观并发）控制（不匹配时返回 409），省略它则无条件应用更新（最后写入者胜出）。对已归档智能体的更新会被拒绝。

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsSL "https://api.anthropic.com/v1/agents/$AGENT_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "version": '"$AGENT_VERSION"',
      "system": "You are a helpful coding agent. Always write tests."
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply coding-assistant.md
    ```

    <File filename="coding-assistant.md">
      ```markdown
      ---
      name: Coding Assistant
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
      ---

      You are a helpful coding agent. Always write tests.
      ```
    </File>

    编辑智能体的文件并再次运行 `ant apply`。该命令会为您提供 `version`。
  </CodeGroupItem>

  ```python Python
  updated_agent = client.beta.agents.update(
      agent.id,
      version=agent.version,
      system="You are a helpful coding agent. Always write tests.",
  )

  print(f"New version: {updated_agent.version}")
  ```

  ```typescript TypeScript
  const updatedAgent = await client.beta.agents.update(agent.id, {
    version: agent.version,
    system: "You are a helpful coding agent. Always write tests."
  });

  console.log(`New version: ${updatedAgent.version}`);
  ```

  ```csharp C#
  var updatedAgent = await client.Beta.Agents.Update(agent.ID, new()
  {
      Version = agent.Version,
      System = "You are a helpful coding agent. Always write tests.",
  });

  Console.WriteLine($"New version: {updatedAgent.Version}");
  ```

  ```go Go
  updatedAgent, err := client.Beta.Agents.Update(ctx, agent.ID, anthropic.BetaAgentUpdateParams{
  	Version: anthropic.Int(agent.Version),
  	System:  anthropic.String("You are a helpful coding agent. Always write tests."),
  })
  if err != nil {
  	panic(err)
  }

  fmt.Printf("New version: %d\n", updatedAgent.Version)
  ```

  ```java Java
  var updatedAgent = client.beta().agents().update(
      agent.id(),
      AgentUpdateParams.builder()
          .version(agent.version())
          .system("You are a helpful coding agent. Always write tests.")
          .build()
  );

  IO.println("New version: " + updatedAgent.version());
  ```

  ```php PHP
  $updatedAgent = $client->beta->agents->update(
      $agent->id,
      version: $agent->version,
      system: 'You are a helpful coding agent. Always write tests.',
  );

  echo "New version: {$updatedAgent->version}\n";
  ```

  ```ruby Ruby
  updated_agent = client.beta.agents.update(
    agent.id,
    version: agent.version,
    system_: "You are a helpful coding agent. Always write tests."
  )

  puts "New version: #{updated_agent.version}"
  ```
</CodeGroup>

上述示例提供了 `version` (csharp, go: `Version`)，因此只有在智能体仍处于该版本时，更新才会生效。要无条件应用更新，请在请求中省略 `version` (csharp, go: `Version`)：

<CodeGroup>
  ```bash cURL
  curl -fsSL "https://api.anthropic.com/v1/agents/$AGENT_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "description": "Writes and reviews code."
    }'
  ```

  ```bash CLI
  ant beta:agents update \
    --agent-id "$AGENT_ID" \
    --description "Writes and reviews code."
  ```

  ```python Python
  unconditional_agent = client.beta.agents.update(
      agent.id,
      description="Writes and reviews code.",
  )

  print(f"New version: {unconditional_agent.version}")
  ```

  ```typescript TypeScript
  const unconditionalAgent = await client.beta.agents.update(agent.id, {
    description: "Writes and reviews code."
  });

  console.log(`New version: ${unconditionalAgent.version}`);
  ```

  ```csharp C#
  var unconditionalAgent = await client.Beta.Agents.Update(agent.ID, new()
  {
      Description = "Writes and reviews code.",
  });

  Console.WriteLine($"New version: {unconditionalAgent.Version}");
  ```

  ```go Go
  unconditionalAgent, err := client.Beta.Agents.Update(ctx, agent.ID, anthropic.BetaAgentUpdateParams{
  	Description: anthropic.String("Writes and reviews code."),
  })
  if err != nil {
  	panic(err)
  }

  fmt.Printf("New version: %d\n", unconditionalAgent.Version)
  ```

  ```java Java
  var unconditionalAgent = client.beta().agents().update(
      agent.id(),
      AgentUpdateParams.builder()
          .description("Writes and reviews code.")
          .build()
  );

  IO.println("New version: " + unconditionalAgent.version());
  ```

  ```php PHP
  $unconditionalAgent = $client->beta->agents->update(
      $agent->id,
      description: 'Writes and reviews code.',
  );

  echo "New version: {$unconditionalAgent->version}\n";
  ```

  ```ruby Ruby
  unconditional_agent = client.beta.agents.update(
    agent.id,
    description: "Writes and reviews code."
  )

  puts "New version: #{unconditional_agent.version}"
  ```
</CodeGroup>

### 更新语义

* **`version` (csharp, go: `Version`)** 是可选的，提供时必须至少为 1。提供时，如果它与智能体的当前版本不匹配，请求将返回 409，即使您发送的字段已与存储的值一致也是如此；请重新读取智能体并重试。省略时，更新将无条件应用，最近的更新会静默替换任何并发更新，且不会向任何一方调用者返回错误。对于交互式调用者，建议默认提供 `version` (csharp, go: `Version`)；而省略它适用于声明式 apply 循环，例如同步已签入智能体定义的 CI 作业，此时由该循环拥有智能体。

* **省略的字段会被保留。** 您只需包含想要更改的字段。

* **标量字段**（`model` (csharp, go: `Model`)、`system` (csharp, go: `System`; ruby: `system_`)、`name` (csharp, go: `Name`)、`description` (csharp, go: `Description`)）会被替换为新值。`system` (csharp, go: `System`; ruby: `system_`) 和 `description` (csharp, go: `Description`) 可以通过传递 `null` (python: `None`; ruby: `nil`; java: `Optional.empty()`; go: `param.Null[string]()`; php: `requestOptions: ['extraBodyParams' => ['system' => null, 'description' => null]]`) 来清除。`model` (csharp, go: `Model`) 和 `name` (csharp, go: `Name`) 是必填项，无法清除。在您提供的 `model` (csharp, go: `Model`) 对象中，`effort` (csharp, go: `Effort`) 是唯一的例外：如果模型 `id` (csharp, go: `ID`) 未变，省略 `effort` (csharp, go: `Effort`) 会保持存储的 effort 级别不变。如果您更改了模型 `id` (csharp, go: `ID`)，省略的 `effort` (csharp, go: `Effort`) 会重置为新模型的默认值。其他 `model` (csharp, go: `Model`) 字段会随对象一起被替换：提供不含 `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`) 的 `model` (csharp, go: `Model`) 会清除智能体的推理地理位置固定值。

* **数组字段**（`tools` (csharp, go: `Tools`)、`mcp_servers` (java, php: `mcpServers`; csharp: `McpServers`; go: `MCPServers`)、`skills` (csharp, go: `Skills`)）会被新数组完全替换。若要完全清除某个数组字段，请传递 `null` (python: `None`; ruby: `nil`; java: `Optional.empty()`; go: `param.NullSlice[[]anthropic.BetaAgentUpdateParamsToolUnion]()`; php: `requestOptions: ['extraBodyParams' => ['mcp_servers' => null]]`) 或空数组。

* **`multiagent` (csharp, go: `Multiagent`)** 在更新保持 `multiagent_20261001` 类型时，会逐层合并到存储的块中。您省略的设置会保留其存储值，而您发送的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表会替换存储的列表。任何其他发送 `multiagent` (csharp, go: `Multiagent`) 的更新都会整体替换存储的块，其省略的设置将采用默认值。传递 `"multiagent": null` (python: `multiagent=None`; typescript: `multiagent: null`; ruby: `multiagent: nil`; go: `Multiagent: param.NullStruct[anthropic.BetaManagedAgentsMultiagentParamsUnion]()`; java: `.multiagent(Optional.empty())`; csharp: `Multiagent = null`; php: `requestOptions: ['extraBodyParams' => ['multiagent' => null]]`) 可清除整个块。在块内部，`null` 会将设置重置为其默认值，因此将 `workflows` (csharp, go: `Workflows`) 设置为 `null` 会开启动态工作流。请参阅[开启动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#turn-on-dynamic-workflows)。

* **元数据**在键级别进行合并。您提供的键会被添加或更新。您省略的键会被保留。若要删除特定键，请将其值设置为 `null`。

* **无操作检测。** 如果更新相对于当前版本没有产生任何变化，则不会创建新版本，并返回现有版本。

* **其他智能体的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表不会被更新。** 在 `subagents` (csharp, go: `Subagents`) 或 `workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中列出此智能体的其他智能体，会保留在它们创建时或上次更新发送该列表时所固定的版本，即使引用中省略了 `version` (csharp, go: `Version`) 也是如此。若要委派给新版本，请[更新列出它的智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#list-the-subagents)，使列表引用新版本。

以下更新会清除描述，从 `metadata` (csharp, go: `Metadata`) 中删除 `team` 键，并将 `workflows` (csharp, go: `Workflows`) 设置为 `null`，从而开启动态工作流：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsSL "https://api.anthropic.com/v1/agents/$AGENT_ID" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "description": null,
      "metadata": {"team": null},
      "multiagent": {"type": "multiagent_20261001", "workflows": null}
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply code-reviewer.md
    ```

    <File filename="code-reviewer.md">
      ```markdown
      ---
      name: Code Reviewer
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
      metadata:
        owner: docs
      multiagent:
        type: multiagent_20261001
        workflows: null
      ---

      You are a helpful coding agent.
      ```
    </File>

    该文件不再包含 `description` 或 `team` 键，因此 `ant apply` 会为它们发送 `null`。删除 `multiagent` 内的某个设置会保留其存储值，因此该文件将 `workflows` 设置为 `null`。
  </CodeGroupItem>

  ```python Python
  cleared_agent = client.beta.agents.update(
      agent.id,
      description=None,
      metadata={"team": None},
      multiagent={"type": "multiagent_20261001", "workflows": None},
  )

  print(f"Description: {cleared_agent.description}")
  ```

  ```typescript TypeScript
  const clearedAgent = await client.beta.agents.update(agent.id, {
    description: null,
    metadata: { team: null },
    multiagent: { type: "multiagent_20261001", workflows: null }
  });

  console.log(`Description: ${clearedAgent.description}`);
  ```

  ```csharp C#
  var clearedAgent = await client.Beta.Agents.Update(agent.ID, new()
  {
      Description = null,
      Metadata = new Dictionary<string, string?> { ["team"] = null },
      Multiagent = new BetaManagedAgentsMultiagent20261001Params { Workflows = null },
  });

  Console.WriteLine($"Description: {clearedAgent.Description}");
  ```

  ```go Go
  // Metadata 是 map[string]string，因此将 team 的 null 作为请求选项发送。
  clearedAgent, err := client.Beta.Agents.Update(ctx, agent.ID, anthropic.BetaAgentUpdateParams{
  	Description: param.Null[string](),
  	Multiagent: anthropic.BetaManagedAgentsMultiagentParamsUnion{
  		OfMultiagent20261001: &anthropic.BetaManagedAgentsMultiagent20261001Params{
  			Workflows: param.NullStruct[anthropic.BetaManagedAgentsMultiagentWorkflowsParamsUnion](),
  		},
  	},
  }, option.WithJSONSet("metadata.team", nil))
  if err != nil {
  	panic(err)
  }

  fmt.Printf("Description: %q\n", clearedAgent.Description)
  ```

  ```java Java
  var clearedAgent = client.beta().agents().update(
      agent.id(),
      AgentUpdateParams.builder()
          .description(Optional.empty())
          .metadata(
              AgentUpdateParams.Metadata.builder()
                  .putAdditionalProperty("team", JsonValue.from(null))
                  .build()
          )
          .multiagent(
              BetaManagedAgentsMultiagent20261001Params.builder()
                  .workflows(Optional.empty())
                  .build()
          )
          .build()
  );

  IO.println("Description: " + clearedAgent.description().orElse(null));
  ```

  ```php PHP
  // update() 会丢弃 null 参数，因此请在 extraBodyParams 中传入这些 null 值。
  $clearedAgent = $client->beta->agents->update(
      $agent->id,
      requestOptions: [
          'extraBodyParams' => [
              'description' => null,
              'metadata' => ['team' => null],
              'multiagent' => ['type' => 'multiagent_20261001', 'workflows' => null],
          ],
      ],
  );

  echo 'Description: ' . json_encode($clearedAgent->description) . "\n";
  ```

  ```ruby Ruby
  cleared_agent = client.beta.agents.update(
    agent.id,
    description: nil,
    metadata: {team: nil},
    multiagent: {type: :multiagent_20261001, workflows: nil}
  )

  puts "Description: #{cleared_agent.description.inspect}"
  ```
</CodeGroup>

## 智能体生命周期

| 操作       | 行为                            |
| -------- | ----------------------------- |
| **更新**   | 当配置发生变化时生成新的智能体版本。            |
| **列出版本** | 返回完整的版本历史，以便您跟踪随时间发生的变化。      |
| **归档**   | 使智能体变为只读。新会话无法引用它，但现有会话会继续运行。 |

### 列出版本

获取完整的版本历史，以跟踪智能体随时间的变化。结果是分页的，SDK 可以为您获取所有页面。

<CodeGroup>
  ```bash cURL
  curl -fsSL "https://api.anthropic.com/v1/agents/$AGENT_ID/versions" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:agents:versions list --agent-id "$AGENT_ID"
  ```

  ```python Python
  for version in client.beta.agents.versions.list(agent.id):
      print(f"Version {version.version}: {version.updated_at.isoformat()}")
  ```

  ```typescript TypeScript
  for await (const version of client.beta.agents.versions.list(agent.id)) {
    console.log(`Version ${version.version}: ${version.updated_at}`);
  }
  ```

  ```csharp C#
  var versions = await client.Beta.Agents.Versions.List(agent.ID);
  await foreach (var version in versions.Paginate())
  {
      Console.WriteLine($"Version {version.Version}: {version.UpdatedAt:O}");
  }
  ```

  ```go Go
  iter := client.Beta.Agents.Versions.ListAutoPaging(ctx, agent.ID, anthropic.BetaAgentVersionListParams{})
  for iter.Next() {
  	version := iter.Current()
  	fmt.Printf("Version %d: %s\n", version.Version, version.UpdatedAt.Format(time.RFC3339))
  }
  if err := iter.Err(); err != nil {
  	panic(err)
  }
  ```

  ```java Java
  for (var version : client.beta().agents().versions().list(agent.id()).autoPager()) {
      IO.println("Version " + version.version() + ": " + version.updatedAt());
  }
  ```

  ```php PHP
  foreach ($client->beta->agents->versions->list($agent->id)->pagingEachItem() as $version) {
      echo "Version {$version->version}: {$version->updatedAt->format(\DateTimeInterface::ATOM)}\n";
  }
  ```

  ```ruby Ruby
  client.beta.agents.versions.list(agent.id).auto_paging_each do |agent_version|
    puts "Version #{agent_version.version}: #{agent_version.updated_at.iso8601}"
  end
  ```
</CodeGroup>

### 归档智能体

归档会使智能体变为只读，且无法撤销。现有会话会继续运行，但新会话无法引用该智能体。响应会将 `archived_at` (java, php: `archivedAt`; csharp, go: `ArchivedAt`) 设置为归档时间戳。

<CodeGroup>
  ```bash cURL
  curl -fsSL -X POST "https://api.anthropic.com/v1/agents/$AGENT_ID/archive" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01"
  ```

  ```bash CLI
  ant beta:agents archive --agent-id "$AGENT_ID"
  ```

  ```python Python
  archived = client.beta.agents.archive(agent.id)

  print(f"Archived at: {archived.archived_at.isoformat()}")
  ```

  ```typescript TypeScript
  const archived = await client.beta.agents.archive(agent.id);
  console.log(`Archived at: ${archived.archived_at}`);
  ```

  ```csharp C#
  var archived = await client.Beta.Agents.Archive(agent.ID);
  Console.WriteLine($"Archived at: {archived.ArchivedAt:O}");
  ```

  ```go Go
  archived, err := client.Beta.Agents.Archive(ctx, agent.ID, anthropic.BetaAgentArchiveParams{})
  if err != nil {
  	panic(err)
  }
  fmt.Printf("Archived at: %s\n", archived.ArchivedAt.Format(time.RFC3339))
  ```

  ```java Java
  var archived = client.beta().agents().archive(agent.id());
  IO.println("Archived at: " + archived.archivedAt().orElseThrow());
  ```

  ```php PHP
  $archived = $client->beta->agents->archive($agent->id);

  echo "Archived at: {$archived->archivedAt->format(\DateTimeInterface::ATOM)}\n";
  ```

  ```ruby Ruby
  archived = client.beta.agents.archive(agent.id)
  puts "Archived at: #{archived.archived_at.iso8601}"
  ```
</CodeGroup>

## 后续步骤

<CardGroup cols={2}>
  <Card title="工具" icon="tool" href="https://platform.claude.com/docs/zh-CN/managed-agents/tools">
    配置智能体可用的工具。
  </Card>

  <Card title="技能" icon="graduation-cap" href="https://platform.claude.com/docs/zh-CN/managed-agents/skills">
    为您的智能体附加可复用、基于文件系统的专业知识，以支持特定领域的工作流。
  </Card>

  <Card title="启动会话" icon="play" href="https://platform.claude.com/docs/zh-CN/managed-agents/sessions">
    创建会话以运行您的智能体并开始执行任务。
  </Card>

  <Card title="参考" icon="book" href="https://platform.claude.com/docs/zh-CN/managed-agents/reference">
    Claude Managed Agents 的事件类型、自托管 worker CLI 标志、支持的 MCP 服务器类型、速率限制和品牌指南。
  </Card>
</CardGroup>

## 可用性

智能体配置在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台                                                                                                                                                                                                                                                                                  | 模型                                                             |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed) | [所有模型](https://platform.claude.com/docs/zh-CN/models/overview) |

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
