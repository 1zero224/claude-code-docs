---
title: 多智能体编排
url: https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration
description: 在单个会话中协调多个智能体。
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

多智能体编排（multiagent orchestration）让一个智能体与其他智能体协调完成复杂工作。智能体可以并行行动，各自拥有隔离的上下文，这有助于提高输出质量，也可以缩短完成时间。

不确定多智能体设置是否适合您的问题？请参阅[何时使用多智能体系统（以及何时不使用）](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them)。

## 将工作交给其他智能体

会话所运行的智能体可以通过两种方式将工作交给其他智能体。使用 **subagents**（子智能体）时，它自己委派任务并读取每个子智能体的报告。使用 **dynamic workflows**（动态工作流）时，它编写一个工作流：一个在后台运行多个智能体并合并其结果的程序。它还可以在自己完成工作的同时咨询一个 **advisor**（顾问）模型以获取指导。

您决定智能体可以使用其中哪些方式，而智能体决定何时使用它们。为了引导这一选择，请在智能体的系统提示中告诉它何时使用工作流运行。请参阅[告诉智能体何时使用运行](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#tell-the-agent-when-to-use-a-run)。您还可以将智能体限制为只使用您列出的智能体。

使用子智能体时，由智能体本身决定接下来发生什么。[子智能体的线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)在您归档之前一直可用，因此智能体可以向其发送后续消息。使用动态工作流时，Claude 编写一个程序来编排智能体，无需 Claude 直接参与。上下文和结果以编程方式从一个智能体传递到另一个智能体，从而让主会话线程腾出来与用户沟通，并查看一个或多个正在运行的工作流以报告进度。智能体无法向运行的线程发送后续消息，并且服务器最迟会在每个运行结束时将其线程归档。

| 方式                                                                                                                    | 发生什么                                                                                                                                         | 适用场景                                                          | 注意事项                                                                                                                                                 |
| --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| [委派给子智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#delegate-to-subagents)       | 智能体将任务委派给其子智能体。每个子智能体在自己的[会话线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)中工作，您可以列出并流式传输这些线程。                    | 智能体需要在子智能体报告后对其进行跟进，或者它需要您列出的专家智能体，这些智能体拥有自己的系统提示和工具。         | 委派只有一层深度，并且一个会话同时最多可以有 25 个子线程，包括空闲线程。顾问线程和工作流运行的线程不计入其中。                                                                                            |
| [动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#dynamic-workflows)             | 智能体编写一个工作流：一个分阶段运行多个智能体并合并其结果的程序。服务器在后台将其作为一个工作流运行来执行，您可以跟踪该运行。工作流自行定义智能体，或从您提供的列表中选择智能体。                                                    | 大多数需要多个智能体的工作：包含许多部分的工作、并行工作，或需要更快完成的长任务。示例包括审计、迁移、深度研究和交叉核对。 | 运行中的每个智能体都会消耗令牌，因此请设置[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)来限制会话的支出（包括运行）。在智能体的系统提示中告诉它何时使用运行。您可以按阶段跟踪运行，并可以读取它的每个线程。 |
| [为会话提供顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor) | 会话的[主线程](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#how-it-works)在轮次中途咨询顾问模型以获取指导，例如规划方法或审查工作，并继续自己完成工作。 | 应由一个智能体完成工作，并在关键时刻（例如规划或最终审查）借助顾问模型的判断。                       | 只有主线程可以咨询顾问，并且咨询按顾问模型的费率计费。                                                                                                                          |

您可以在智能体定义的 `multiagent` 块中设置这些内容，该块有一个 `type`。使用 `multiagent_20261001` 类型时，智能体可以同时使用这三种方式，并且您可以分别开启或关闭每一种。默认情况下，`subagents` 和 `workflows` 均已启用。`subagents` 和 `workflows` 都启用了 `inline_agents`，这是用于[智能体或工作流自行定义的智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#predefined-and-inline-agents)的设置：

```json
{
  "multiagent": { "type": "multiagent_20261001" }
}
```

要设置智能体可以调用哪些智能体，请参阅[预定义智能体和内联智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#predefined-and-inline-agents)。要关闭某项设置，请参阅[开启动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#turn-on-dynamic-workflows)。

## 预定义智能体和内联智能体

每个子智能体以及工作流运行中的每个智能体，都属于以下两种之一：

* **预定义智能体（predefined agent）：** 您已经[创建](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)并在 `multiagent` (csharp, go: `Multiagent`) 块中列出的智能体。它使用自己的配置：模型、系统提示、工具、MCP 服务器和技能。
* **内联智能体（inline agent）：** 未保存的智能体。由会话所运行的智能体或工作流在分派工作时定义。它使用该智能体的模型、工具、MCP 服务器和技能。

两种智能体在 `subagents` 和 `workflows` 下都可以使用：

| 智能体类型 | 在 `subagents` 下                                        | 在 `workflows` 下                                                  |
| ----- | ------------------------------------------------------ | ---------------------------------------------------------------- |
| 预定义   | 智能体可以委派给您在 `subagents.predefined_agents` 中列出的智能体。      | 工作流可以使用您在 `workflows.predefined_agents` 中列出的智能体。                 |
| 内联    | 智能体在委派时可以定义内联智能体。`subagents.inline_agents` 用于开启或关闭此功能。 | 工作流可以定义内联智能体，并为每个智能体编写系统提示。`workflows.inline_agents` 用于开启或关闭此功能。 |

`subagents.predefined_agents` 和 `workflows.predefined_agents` 是两个独立的列表。一个列表中的智能体不会被添加到另一个列表中。两个列表默认都为空。任一列表中的条目采用以下形式之一：

* `{"type": "agent", "id": agent.id}` 通过 ID 引用先前创建的 `agent`。如果未指定 `version`，则在创建列出它的智能体时，或者在更新请求发送该列表时，引用会固定到该智能体的最新版本。
* `{"type": "agent", "id": agent.id, "version": agent.version}` 固定特定的智能体版本。
* 单独的 `agent.id` 字符串是 `{"type": "agent", "id": agent.id}` 的简写。
* `{"type": "self"}` 列出智能体本身，以便由它的副本来完成工作。如果会话是使用[智能体配置覆盖](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#override-agent-configuration-for-a-session)创建的，这些覆盖也适用于这些副本。通过 ID 引用的条目不受影响。

这些条目以及它们所指向的智能体的规则适用于两个列表。请参阅[列出子智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#list-the-subagents)。

在两项设置下，内联智能体默认都是开启的。要只允许您列出的智能体，请在 `subagents` 下、`workflows` 下或两者下将 `inline_agents` 设置为 `{"type": "disabled"}`。关闭内联智能体的设置需要在其 `predefined_agents` 列表中至少有一个智能体。如果列表为空，请求会失败并返回 400 错误。在更新时，服务器会按更新后的设置进行检查。

以下智能体只允许它列出的智能体。它可以委派给一个智能体和它自身的副本，并且工作流可以使用另一个智能体的版本 2：

```json
{
  "multiagent": {
    "type": "multiagent_20261001",
    "subagents": {
      "type": "enabled",
      "inline_agents": { "type": "disabled" },
      "predefined_agents": ["agent_01J8XkN5uT3vHpLqRfWdY2", { "type": "self" }]
    },
    "workflows": {
      "type": "enabled",
      "inline_agents": { "type": "disabled" },
      "predefined_agents": [
        { "type": "agent", "id": "agent_01Lm4cV8yQ2tNs7XbKdR5h", "version": 2 }
      ]
    }
  }
}
```

## 委派给子智能体

### 委派什么

多智能体协调最适合复杂任务，这些任务要么需要跨多种界面开展工作，要么由多个范围明确、共同服务于总体目标的任务组成。

效果良好的模式：

* **并行化：** 同时分发相互独立的子任务（搜索多个来源、分析不同文件），并让智能体综合结果。
* **专业化：** 路由到具有领域专注系统提示和工具的智能体，例如安全智能体或文档智能体，而不是让单个智能体承载所有能力。
* **升级：** 针对一部分复杂子任务咨询能力更强的智能体或模型。要咨询模型，请[为会话提供顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)。

### 工作原理

所有智能体共享相同的沙箱、文件系统和[保管库凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)，但每个智能体都在自己的 **session thread**（会话线程）中运行，这是一个具有自己对话历史的上下文隔离事件流。会话所运行的智能体在 **primary thread**（主线程）中报告活动，主线程即会话级的[事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)。当它委派工作时，会在运行时生成额外的线程。工作流运行也会创建线程。

子智能体的线程是持久的。智能体可以向之前调用过的子智能体发送后续消息，该子智能体会保留其先前轮次中的所有内容。

子智能体使用哪种配置，取决于它是[预定义智能体还是内联智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#predefined-and-inline-agents)。会话级的[智能体配置覆盖](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#override-agent-configuration-for-a-session)适用于会话所运行的智能体及其 `self` 副本。每个智能体都保留自己的对话历史。

### 列出子智能体

在[定义您的智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)时，设置 `multiagent` (csharp, go: `Multiagent`) 块中 `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表，以列出它可以委派的智能体：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Engineering Lead",
      "model": "claude-opus-5-5",
      "system": "You coordinate engineering work. Delegate code review to the reviewer agent and test writing to the test agent.",
      "tools": [
        {
          "type": "agent_toolset_20260401"
        }
      ],
      "multiagent": {
        "type": "multiagent_20261001",
        "subagents": {
          "type": "enabled",
          "predefined_agents": [
            {"type": "agent", "id": "'"$REVIEWER_AGENT_ID"'"},
            {"type": "agent", "id": "'"$TEST_WRITER_AGENT_ID"'"}
          ]
        }
      }
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply engineering-lead.md reviewer.md test-writer.md
    ```

    <File filename="engineering-lead.md">
      ```markdown
      ---
      name: Engineering Lead
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
      multiagent:
        type: multiagent_20261001
        subagents:
          type: enabled
          predefined_agents: # ant apply replaces each path with {type: agent, id, version}
            - ./reviewer.md
            - ./test-writer.md
      ---

      You coordinate engineering work. Delegate code review to the reviewer agent
      and test writing to the test agent.
      ```
    </File>

    <File filename="reviewer.md">
      ```markdown
      ---
      name: reviewer
      model: claude-haiku-5-5
      ---

      You are a code reviewer.
      ```
    </File>

    <File filename="test-writer.md">
      ```markdown
      ---
      name: test-writer
      model: claude-haiku-5-5
      ---

      You write unit tests.
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  lead_agent = client.beta.agents.create(
      name="Engineering Lead",
      model="claude-opus-5-5",
      system="You coordinate engineering work. Delegate code review to the reviewer agent and test writing to the test agent.",
      tools=[
          {"type": "agent_toolset_20260401"},
      ],
      multiagent={
          "type": "multiagent_20261001",
          "subagents": {
              "type": "enabled",
              "predefined_agents": [
                  {"type": "agent", "id": reviewer_agent.id},
                  {"type": "agent", "id": test_writer_agent.id},
              ],
          },
      },
  )
  ```

  ```typescript TypeScript
  const leadAgent = await client.beta.agents.create({
    name: "Engineering Lead",
    model: "claude-opus-5-5",
    system:
      "You coordinate engineering work. Delegate code review to the reviewer agent and test writing to the test agent.",
    tools: [{ type: "agent_toolset_20260401" }],
    multiagent: {
      type: "multiagent_20261001",
      subagents: {
        type: "enabled",
        predefined_agents: [
          { type: "agent", id: reviewerAgent.id },
          { type: "agent", id: testWriterAgent.id }
        ]
      }
    }
  });
  ```

  ```csharp C#
  var leadAgent = await client.Beta.Agents.Create(new()
  {
      Name = "Engineering Lead",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      System = "You coordinate engineering work. Delegate code review to the reviewer agent and test writing to the test agent.",
      Tools = [new BetaManagedAgentsAgentToolset20260401Params()],
      Multiagent = new BetaManagedAgentsMultiagent20261001Params
      {
          Subagents = new BetaManagedAgentsMultiagentSubagentsEnabledParams
          {
              PredefinedAgents = [reviewerAgent.ID, testWriterAgent.ID],
          },
      },
  });
  ```

  ```go Go
  leadAgent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name:   "Engineering Lead",
  	Model:  anthropic.BetaManagedAgentsModelConfigParams{ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5},
  	System: anthropic.String("You coordinate engineering work. Delegate code review to the reviewer agent and test writing to the test agent."),
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{{
  		OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{},
  	}},
  	Multiagent: anthropic.BetaManagedAgentsMultiagentParamsUnion{
  		OfMultiagent20261001: &anthropic.BetaManagedAgentsMultiagent20261001Params{
  			Subagents: anthropic.BetaManagedAgentsMultiagentSubagentsParamsUnion{
  				OfEnabled: &anthropic.BetaManagedAgentsMultiagentSubagentsEnabledParams{
  					PredefinedAgents: []anthropic.BetaManagedAgentsMultiagentPredefinedAgentParamsUnion{
  						{OfString: anthropic.String(reviewerAgent.ID)},
  						{OfString: anthropic.String(testWriterAgent.ID)},
  					},
  				},
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var leadAgent = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("Engineering Lead")
          .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
          .system("You coordinate engineering work. Delegate code review to the reviewer agent and test writing to the test agent.")
          .addTool(BetaManagedAgentsAgentToolset20260401Params.builder().build())
          .multiagent(BetaManagedAgentsMultiagent20261001Params.builder()
              .subagents(BetaManagedAgentsMultiagentSubagentsEnabledParams.builder()
                  .addPredefinedAgent(BetaManagedAgentsAgentParams.builder()
                      .id(reviewerAgent.id())
                      .build())
                  .addPredefinedAgent(BetaManagedAgentsAgentParams.builder()
                      .id(testWriterAgent.id())
                      .build())
                  .build())
              .build())
          .build()
  );
  ```

  ```php PHP
  $leadAgent = $client->beta->agents->create(
      name: 'Engineering Lead',
      model: 'claude-opus-5-5',
      system: 'You coordinate engineering work. Delegate code review to the reviewer agent and test writing to the test agent.',
      tools: [
          ['type' => 'agent_toolset_20260401'],
      ],
      multiagent: [
          'type' => 'multiagent_20261001',
          'subagents' => [
              'type' => 'enabled',
              'predefined_agents' => [
                  ['type' => 'agent', 'id' => $reviewerAgent->id],
                  ['type' => 'agent', 'id' => $testWriterAgent->id],
              ],
          ],
      ],
  );
  ```

  ```ruby Ruby
  lead_agent = client.beta.agents.create(
    name: "Engineering Lead",
    model: Anthropic::Beta::BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
    system: "You coordinate engineering work. Delegate code review to the reviewer agent and test writing to the test agent.",
    tools: [
      {type: "agent_toolset_20260401"}
    ],
    multiagent: {
      type: "multiagent_20261001",
      subagents: {
        type: "enabled",
        predefined_agents: [
          {type: "agent", id: reviewer_agent.id},
          {type: "agent", id: test_writer_agent.id}
        ]
      }
    }
  )
  ```
</CodeGroup>

除非您将其关闭，否则智能体还可以委派给内联智能体。有关该设置以及 `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中条目可采用的形式，请参阅[预定义智能体和内联智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#predefined-and-inline-agents)。

`"advisor": {"type": "enabled", "model": "<model id>"}` 为会话的主线程提供一个可在轮次中途咨询的顾问。顾问是 `multiagent` (csharp, go: `Multiagent`) 块中的一项设置，而不是此列表中的条目。请参阅[为会话提供顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)。

使用此类型时，动态工作流默认也已启用，因此该智能体可以规划在后台运行多个智能体的大型工作。要关闭它们，或列出工作流可以使用的智能体，请参阅[开启动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#turn-on-dynamic-workflows)。

以下规则适用于您在 `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中列出的智能体，也适用于您在 `workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中列出的智能体：

* **固定版本：** 智能体的配置（包括 `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表）会在创建或更新智能体时生成快照。被引用的智能体会固定在当时解析出的版本，不会获取之后对其定义的更新。未发送该列表的更新会保留已固定的版本。要委派给被引用智能体的较新版本，请[更新智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#update-an-agent)，使 `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表引用该版本。
* **一层：** 智能体只能委派给一层智能体。引用另一个设置了 `multiagent` (csharp, go: `Multiagent`) 的智能体，会导致创建或更新请求失败并返回 400 验证错误。
* **最多 20 个智能体：** `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表最多可以列出 20 个不同的智能体。该限制针对每个列表：`workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表也最多可以列出 20 个。在会话的[线程限制](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)内，智能体可以调用每个子智能体的多个副本。
* **推理地理位置：** 智能体以及您在 `subagents` (csharp, go: `Subagents`) 或 `workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中列出的每个智能体，必须固定相同的[推理地理位置](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)（[智能体定义](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)中 `model` (csharp, go: `Model`) 里的 `inference_geo` (java, php: `inferenceGeo`; csharp, go: `InferenceGeo`)），或者它们都不固定。任一列表中的不匹配都会被拒绝并返回 400 验证错误。该检查既在保存智能体时运行，也在[创建会话时的覆盖](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#override-agent-configuration-for-a-session)更改任何固定设置时运行。

### 创建会话

创建一个引用该智能体的会话。智能体会根据需要委派给您在 `subagents` (csharp, go: `Subagents`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中列出的智能体。除非您将其关闭，否则它还可以委派给[内联智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#predefined-and-inline-agents)。

<CodeGroup>
  ```bash cURL
  curl -fsSL https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$LEAD_AGENT_ID"'",
      "environment_id": "'"$ENVIRONMENT_ID"'"
    }'
  ```

  ```bash CLI
  ant beta:sessions create \
    --agent "$LEAD_AGENT_ID" \
    --environment-id "$ENVIRONMENT_ID"
  ```

  ```python Python
  session = client.beta.sessions.create(
      agent=lead_agent.id,
      environment_id=environment.id,
  )
  ```

  ```typescript TypeScript
  const session = await client.beta.sessions.create({
    agent: leadAgent.id,
    environment_id: environment.id
  });
  ```

  ```csharp C#
  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = leadAgent.ID,
      EnvironmentID = environment.ID,
  });
  ```

  ```go Go
  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfString: anthropic.String(leadAgent.ID),
  	},
  	EnvironmentID: environment.ID,
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var session = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(leadAgent.id())
      .environmentId(environment.id())
      .build());
  ```

  ```php PHP
  $session = $client->beta->sessions->create(
      agent: $leadAgent->id,
      environmentID: $environment->id,
  );
  ```

  ```ruby Ruby
  session = client.beta.sessions.create(
    agent: lead_agent.id,
    environment_id: environment.id
  )
  ```
</CodeGroup>

### 将智能体连接到 MCP 服务器

MCP 服务器的作用域是智能体：每个智能体定义都声明自己的服务器和工具。内联智能体没有智能体定义，因此它使用会话所运行的智能体的 MCP 服务器和工具。保管库凭据的作用域是会话：在创建会话时传入的 `vault_ids` (java: `vaultIds`; php: `vaultIDs`; csharp: `VaultIds`; go: `VaultIDs`) 适用于每个线程。这对您的集成有两点影响：

* 要对 MCP 服务器进行身份验证，请为任一智能体使用的每个 MCP 服务器都包含一个保管库凭据。
* 要限制智能体的访问权限，请在其智能体定义中只声明它所需的服务器。您无法通过这种方式限制内联智能体。要只允许您列出的智能体，请在 `subagents` (csharp, go: `Subagents`) 和 `workflows` (csharp, go: `Workflows`) 中都禁用 `inline_agents` (java, php: `inlineAgents`; csharp, go: `InlineAgents`)，并在每个中至少列出一个智能体。请参阅[预定义智能体和内联智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#predefined-and-inline-agents)。

创建会话时的[智能体配置覆盖](https://platform.claude.com/docs/zh-CN/managed-agents/sessions#override-agent-configuration-for-a-session)可以替换会话所运行的智能体及其 `self` 副本的 MCP 服务器。

使用 `limited` [环境](https://platform.claude.com/docs/zh-CN/managed-agents/environments#networking)时，如果智能体或您在 `subagents` (csharp, go: `Subagents`) 或 `workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中列出的智能体声明了一个主机不在 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中的 MCP 服务器，会话创建将失败并返回 400 错误。在环境的网络设置中将 `allow_mcp_servers` (java: `allowMcpServers`; php: `allowMCPServers`; csharp: `AllowMcpServers`; go: `AllowMCPServers`) 设置为 `true` 可关闭此检查。

创建声明了 GitHub MCP 服务器的研究员智能体，以及委派给该研究员的智能体：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  # 创建 researcher
  curl --fail-with-body -sS https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "researcher",
      "model": "claude-haiku-5-5",
      "mcp_servers": [{"type": "url", "name": "github", "url": "https://api.githubcopilot.com/mcp/"}],
      "tools": [{"type": "mcp_toolset", "mcp_server_name": "github"}]
    }'

  # 创建委派给它的 agent：research_agent_id 是第一个响应中的 ID
  curl --fail-with-body -sS https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "lead",
      "model": "claude-opus-5-5",
      "tools": [{"type": "agent_toolset_20260401"}],
      "multiagent": {
        "type": "multiagent_20261001",
        "subagents": {
          "type": "enabled",
          "predefined_agents": [{"type": "agent", "id": "'"$research_agent_id"'"}]
        }
      }
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply lead.md researcher.md
    ```

    <File filename="lead.md">
      ```markdown
      ---
      name: lead
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
      multiagent:
        type: multiagent_20261001
        subagents:
          type: enabled
          predefined_agents: # ant apply replaces each path with {type: agent, id, version}
            - ./researcher.md
      ---
      ```
    </File>

    <File filename="researcher.md">
      ```markdown
      ---
      name: researcher
      model: claude-haiku-5-5
      mcp_servers:
        - type: url
          name: github
          url: https://api.githubcopilot.com/mcp/
      tools:
        - type: mcp_toolset
          mcp_server_name: github
      ---
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  research_agent = client.beta.agents.create(
      name="researcher",
      model="claude-haiku-5-5",
      mcp_servers=[
          {"type": "url", "name": "github", "url": "https://api.githubcopilot.com/mcp/"},
      ],
      tools=[{"type": "mcp_toolset", "mcp_server_name": "github"}],
  )

  lead_agent = client.beta.agents.create(
      name="lead",
      model="claude-opus-5-5",
      tools=[{"type": "agent_toolset_20260401"}],
      multiagent={
          "type": "multiagent_20261001",
          "subagents": {
              "type": "enabled",
              "predefined_agents": [{"type": "agent", "id": research_agent.id}],
          },
      },
  )
  ```

  ```typescript TypeScript
  const researchAgent = await client.beta.agents.create({
    name: "researcher",
    model: "claude-haiku-5-5",
    mcp_servers: [{ type: "url", name: "github", url: "https://api.githubcopilot.com/mcp/" }],
    tools: [{ type: "mcp_toolset", mcp_server_name: "github" }]
  });

  const leadAgent = await client.beta.agents.create({
    name: "lead",
    model: "claude-opus-5-5",
    tools: [{ type: "agent_toolset_20260401" }],
    multiagent: {
      type: "multiagent_20261001",
      subagents: {
        type: "enabled",
        predefined_agents: [{ type: "agent", id: researchAgent.id }]
      }
    }
  });
  ```

  ```csharp C#
  var researchAgent = await client.Beta.Agents.Create(new()
  {
      Name = "researcher",
      Model = BetaManagedAgentsModel.ClaudeHaiku5_5,
      McpServers =
      [
          new()
          {
              Name = "github",
              Url = "https://api.githubcopilot.com/mcp/",
          },
      ],
      Tools = [new BetaManagedAgentsMcpToolsetParams { McpServerName = "github" }],
  });

  var leadAgent = await client.Beta.Agents.Create(new()
  {
      Name = "lead",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      Tools = [new BetaManagedAgentsAgentToolset20260401Params()],
      Multiagent = new BetaManagedAgentsMultiagent20261001Params
      {
          Subagents = new BetaManagedAgentsMultiagentSubagentsEnabledParams
          {
              PredefinedAgents = [new BetaManagedAgentsAgentParams { ID = researchAgent.ID }],
          },
      },
  });
  ```

  ```go Go
  researcher, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name:  "researcher",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{ID: anthropic.BetaManagedAgentsModelClaudeHaiku5_5},
  	MCPServers: []anthropic.BetaManagedAgentsURLMCPServerParams{{
  		Name: "github",
  		URL:  "https://api.githubcopilot.com/mcp/",
  	}},
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{{
  		OfMCPToolset: &anthropic.BetaManagedAgentsMCPToolsetParams{
  			MCPServerName: "github",
  		},
  	}},
  })
  if err != nil {
  	panic(err)
  }

  leadAgent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name:  "lead",
  	Model: anthropic.BetaManagedAgentsModelConfigParams{ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5},
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{{
  		OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{},
  	}},
  	Multiagent: anthropic.BetaManagedAgentsMultiagentParamsUnion{
  		OfMultiagent20261001: &anthropic.BetaManagedAgentsMultiagent20261001Params{
  			Subagents: anthropic.BetaManagedAgentsMultiagentSubagentsParamsUnion{
  				OfEnabled: &anthropic.BetaManagedAgentsMultiagentSubagentsEnabledParams{
  					PredefinedAgents: []anthropic.BetaManagedAgentsMultiagentPredefinedAgentParamsUnion{{
  						OfBetaManagedAgentsAgents: &anthropic.BetaManagedAgentsAgentParams{
  							ID: researcher.ID,
  						},
  					}},
  				},
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var researcher = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("researcher")
          .model(BetaManagedAgentsModel.CLAUDE_HAIKU_5_5)
          .addMcpServer(BetaManagedAgentsUrlMcpServerParams.builder()
              .name("github")
              .url("https://api.githubcopilot.com/mcp/")
              .build())
          .addTool(BetaManagedAgentsMcpToolsetParams.builder()
              .mcpServerName("github")
              .build())
          .build()
  );

  var leadAgent = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("lead")
          .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
          .addTool(BetaManagedAgentsAgentToolset20260401Params.builder().build())
          .multiagent(BetaManagedAgentsMultiagent20261001Params.builder()
              .subagents(BetaManagedAgentsMultiagentSubagentsEnabledParams.builder()
                  .addPredefinedAgent(BetaManagedAgentsAgentParams.builder()
                      .id(researcher.id())
                      .build())
                  .build())
              .build())
          .build()
  );
  ```

  ```php PHP
  $researchAgent = $client->beta->agents->create(
      name: 'researcher',
      model: 'claude-haiku-5-5',
      mcpServers: [
          ['type' => 'url', 'name' => 'github', 'url' => 'https://api.githubcopilot.com/mcp/'],
      ],
      tools: [
          ['type' => 'mcp_toolset', 'mcp_server_name' => 'github'],
      ],
  );

  $leadAgent = $client->beta->agents->create(
      name: 'lead',
      model: 'claude-opus-5-5',
      tools: [
          ['type' => 'agent_toolset_20260401'],
      ],
      multiagent: [
          'type' => 'multiagent_20261001',
          'subagents' => [
              'type' => 'enabled',
              'predefined_agents' => [
                  ['type' => 'agent', 'id' => $researchAgent->id],
              ],
          ],
      ],
  );
  ```

  ```ruby Ruby
  research_agent = client.beta.agents.create(
    name: "researcher",
    model: "claude-haiku-5-5",
    mcp_servers: [
      {type: "url", name: "github", url: "https://api.githubcopilot.com/mcp/"}
    ],
    tools: [
      {type: "mcp_toolset", mcp_server_name: "github"}
    ]
  )

  lead_agent = client.beta.agents.create(
    name: "lead",
    model: "claude-opus-5-5",
    tools: [
      {type: "agent_toolset_20260401"}
    ],
    multiagent: {
      type: "multiagent_20261001",
      subagents: {
        type: "enabled",
        predefined_agents: [
          {type: "agent", id: research_agent.id}
        ]
      }
    }
  )
  ```
</CodeGroup>

然后使用保存 GitHub 凭据的保管库创建会话：

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS https://api.anthropic.com/v1/sessions \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "agent": "'"$lead_agent_id"'",
      "environment_id": "'"$environment_id"'",
      "vault_ids": ["'"$vault_id"'"]
    }'
  ```

  ```bash CLI
  ant beta:sessions create \
    --agent "$lead_agent_id" \
    --environment-id "$environment_id" \
    --vault-id "$vault_id"
  ```

  ```python Python
  session = client.beta.sessions.create(
      agent=lead_agent.id,
      environment_id=environment.id,
      vault_ids=[vault.id],
  )
  print(session.id)
  ```

  ```typescript TypeScript
  const session = await client.beta.sessions.create({
    agent: leadAgent.id,
    environment_id: environment.id,
    vault_ids: [vault.id]
  });
  console.log(session.id);
  ```

  ```csharp C#
  var session = await client.Beta.Sessions.Create(new()
  {
      Agent = leadAgent.ID,
      EnvironmentID = environment.ID,
      VaultIds = [vault.ID],
  });
  Console.WriteLine(session.ID);
  ```

  ```go Go
  session, err := client.Beta.Sessions.New(ctx, anthropic.BetaSessionNewParams{
  	Agent: anthropic.BetaSessionNewParamsAgentUnion{
  		OfString: anthropic.String(leadAgent.ID),
  	},
  	EnvironmentID: environment.ID,
  	VaultIDs:      []string{vault.ID},
  })
  if err != nil {
  	panic(err)
  }
  fmt.Println(session.ID)
  ```

  ```java Java
  var session = client.beta().sessions().create(SessionCreateParams.builder()
      .agent(leadAgent.id())
      .environmentId(environment.id())
      .vaultIds(List.of(vault.id()))
      .build());
  IO.println(session.id());
  ```

  ```php PHP
  $session = $client->beta->sessions->create(
      agent: $leadAgent->id,
      environmentID: $environment->id,
      vaultIDs: [$vault->id],
  );
  echo "{$session->id}\n";
  ```

  ```ruby Ruby
  session = client.beta.sessions.create(
    agent: lead_agent.id,
    environment_id: environment.id,
    vault_ids: [vault.id]
  )
  puts session.id
  ```
</CodeGroup>

在此示例中，只有研究员声明了 GitHub MCP 服务器，因此会话所运行的智能体没有访问权限。会话的 `vault_ids` (java: `vaultIds`; php: `vaultIDs`; csharp: `VaultIds`; go: `VaultIDs`) 为研究员的线程提供 GitHub 凭据。

<Tip>
  如果在声明服务器后智能体的 MCP 调用身份验证失败，请确认凭据的 `mcp_server_url` (java: `mcpServerUrl`; php: `mcpServerURL`; csharp: `McpServerUrl`; go: `MCPServerURL`) 与智能体的 `mcp_servers[].url` (java, php: `mcpServers[].url`; csharp: `McpServers[].Url`; go: `MCPServers[].URL`) 指向同一服务器。两个 URL 在匹配前都会被规范化（协议和主机转换为小写，去除默认端口和末尾斜杠），因此主机大小写、默认端口或末尾斜杠的差异不会妨碍匹配；但不同的路径、子域名或非默认端口会导致不匹配。
</Tip>

### 线程

每个智能体的会话线程都有自己的事件流。要列出、中断或归档线程，读取其事件，以及跨线程处理工具权限，请参阅[会话线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)。

## 动态工作流

借助动态工作流，智能体可以承担包含许多部分的工作，例如审查数百份文档或交叉核对许多来源。智能体编写一个 **workflow**（工作流）：一个分阶段运行多个智能体并合并其返回结果的程序。服务器在后台将其作为一个 **workflow run**（工作流运行）来执行，同时智能体继续工作或结束其轮次。您可以通过智能体 `multiagent` (csharp, go: `Multiagent`) 块中的 `workflows` (csharp, go: `Workflows`) 设置开启或关闭动态工作流。运行中的智能体可以同时工作，因此长任务可以比由一个智能体依次完成每个部分更快地完成。

* **运行如何开始：** 您在 `user.message` 中描述工作。智能体据此决定是否以及何时开始运行，因此无需额外的 API 调用。您可以通过在 `user.message` 或智能体的系统提示中描述何时使用运行来影响智能体的决定。（权限策略适用于运行中的智能体所调用的工具，而不适用于开始运行本身。）
* **使用哪些智能体：** 工作流定义的[内联智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#predefined-and-inline-agents)、您在 `workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中列出的预定义智能体，或两者兼有。如果您没有列出任何智能体，则全部由工作流定义。内联智能体使用会话所运行的智能体的模型。要让运行对其中部分智能体使用其他模型，请将它们创建为智能体，并在 `workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中列出它们。
* **如何跟踪：** 运行事件会出现在会话的事件流中，运行中的每个智能体都在一个您可以列出、读取和流式传输的[会话线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)中工作。有关事件、中断、[限制](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#limits)以及[运行的线程显示什么](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#a-runs-threads)，请参阅[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)。

### 开启动态工作流

在[定义您的智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup)时，将 `multiagent.type` 设置为 `"multiagent_20261001"` 并启用 `workflows` (csharp, go: `Workflows`)。使用此类型时，动态工作流和委派（`subagents` (csharp, go: `Subagents`)）默认都处于开启状态。如果只需要动态工作流，还需将 `subagents` (csharp, go: `Subagents`) 设置为 `{"type": "disabled"}`。

在更改现有智能体的 `multiagent.type` 之前，请参阅[如何将智能体迁移到此类型](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#move-from-the-coordinator-type)。

此示例还禁用了网页搜索和网页抓取，因为审查合同不会用到网页：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Contract Reviewer",
      "model": "claude-opus-5-5",
      "system": "You review contracts. When you are asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run.",
      "tools": [
        {
          "type": "agent_toolset_20260401",
          "configs": [
            {"name": "web_search", "enabled": false},
            {"name": "web_fetch", "enabled": false}
          ]
        }
      ],
      "multiagent": {"type": "multiagent_20261001", "workflows": {"type": "enabled"}}
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply contract-reviewer.md
    ```

    <File filename="contract-reviewer.md">
      ```markdown
      ---
      name: Contract Reviewer
      model: claude-opus-5-5
      tools:
        - type: agent_toolset_20260401
          configs:
            - name: web_search
              enabled: false
            - name: web_fetch
              enabled: false
      multiagent:
        type: multiagent_20261001
        workflows:
          type: enabled
      ---

      You review contracts. When you're asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run.
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="Contract Reviewer",
      model="claude-opus-5-5",
      system="You review contracts. When you're asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run.",
      tools=[
          {
              "type": "agent_toolset_20260401",
              "configs": [
                  {"name": "web_search", "enabled": False},
                  {"name": "web_fetch", "enabled": False},
              ],
          },
      ],
      multiagent={"type": "multiagent_20261001", "workflows": {"type": "enabled"}},
  )
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "Contract Reviewer",
    model: "claude-opus-5-5",
    system:
      "You review contracts. When you're asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run.",
    tools: [
      {
        type: "agent_toolset_20260401",
        configs: [
          { name: "web_search", enabled: false },
          { name: "web_fetch", enabled: false }
        ]
      }
    ],
    multiagent: { type: "multiagent_20261001", workflows: { type: "enabled" } }
  });
  ```

  ```csharp C#
  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Contract Reviewer",
      Model = BetaManagedAgentsModel.ClaudeOpus5_5,
      System = "You review contracts. When you're asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run.",
      Tools =
      [
          new BetaManagedAgentsAgentToolset20260401Params
          {
              Configs =
              [
                  new BetaManagedAgentsWebSearchToolConfigParams { Enabled = false },
                  new BetaManagedAgentsWebFetchToolConfigParams { Enabled = false },
              ],
          },
      ],
      Multiagent = new BetaManagedAgentsMultiagent20261001Params
      {
          Workflows = new BetaManagedAgentsMultiagentWorkflowsEnabledParams(),
      },
  });
  ```

  ```go Go
  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name:   "Contract Reviewer",
  	Model:  anthropic.BetaManagedAgentsModelConfigParams{ID: anthropic.BetaManagedAgentsModelClaudeOpus5_5},
  	System: anthropic.String("You review contracts. When you're asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run."),
  	Tools: []anthropic.BetaAgentNewParamsToolUnion{{
  		OfAgentToolset20260401: &anthropic.BetaManagedAgentsAgentToolset20260401Params{
  			Configs: []anthropic.BetaManagedAgentsAgentToolConfigParamsUnion{
  				{OfWebSearch: &anthropic.BetaManagedAgentsWebSearchToolConfigParams{
  					Enabled: anthropic.Bool(false),
  				}},
  				{OfWebFetch: &anthropic.BetaManagedAgentsWebFetchToolConfigParams{
  					Enabled: anthropic.Bool(false),
  				}},
  			},
  		},
  	}},
  	Multiagent: anthropic.BetaManagedAgentsMultiagentParamsUnion{
  		OfMultiagent20261001: &anthropic.BetaManagedAgentsMultiagent20261001Params{
  			Workflows: anthropic.BetaManagedAgentsMultiagentWorkflowsParamsUnion{
  				OfEnabled: &anthropic.BetaManagedAgentsMultiagentWorkflowsEnabledParams{},
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  ```

  ```java Java
  var agent = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("Contract Reviewer")
          .model(BetaManagedAgentsModel.CLAUDE_OPUS_5_5)
          .system("You review contracts. When you're asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run.")
          .addTool(BetaManagedAgentsAgentToolset20260401Params.builder()
              .addConfig(BetaManagedAgentsWebSearchToolConfigParams.builder()
                  .enabled(false)
                  .build())
              .addConfig(BetaManagedAgentsWebFetchToolConfigParams.builder()
                  .enabled(false)
                  .build())
              .build())
          .multiagent(BetaManagedAgentsMultiagent20261001Params.builder()
              .workflows(BetaManagedAgentsMultiagentWorkflowsEnabledParams.builder().build())
              .build())
          .build()
  );
  ```

  ```php PHP
  use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
  use Anthropic\Beta\Agents\BetaManagedAgentsModel;
  use Anthropic\Beta\Agents\BetaManagedAgentsWebFetchToolConfigParams;
  use Anthropic\Beta\Agents\BetaManagedAgentsWebSearchToolConfigParams;

  $agent = $client->beta->agents->create(
      name: 'Contract Reviewer',
      model: BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
      system: "You review contracts. When you're asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run.",
      tools: [
          BetaManagedAgentsAgentToolset20260401Params::with(
              configs: [
                  BetaManagedAgentsWebSearchToolConfigParams::with(enabled: false),
                  BetaManagedAgentsWebFetchToolConfigParams::with(enabled: false),
              ],
          ),
      ],
      multiagent: ['type' => 'multiagent_20261001', 'workflows' => ['type' => 'enabled']],
  );
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Contract Reviewer",
    model: Anthropic::Beta::BetaManagedAgentsModel::CLAUDE_OPUS_5_5,
    system_: "You review contracts. When you're asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run.",
    tools: [
      {
        type: "agent_toolset_20260401",
        configs: [
          {name: "web_search", enabled: false},
          {name: "web_fetch", enabled: false}
        ]
      }
    ],
    multiagent: {type: "multiagent_20261001", workflows: {type: "enabled"}}
  )
  ```
</CodeGroup>

要让工作流使用您已创建的智能体，请将它们列在 `workflows.predefined_agents` 中。`subagents` 和 `advisor` 设置放在同一个 `multiagent` 块中。要列出子智能体，请参阅[委派给子智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#delegate-to-subagents)。要设置顾问，请参阅[为会话配置顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)。以下智能体允许工作流使用您创建的一个智能体，列出了两个子智能体，并配置了一个顾问：

```json
{
  "multiagent": {
    "type": "multiagent_20261001",
    "workflows": {
      "type": "enabled",
      "predefined_agents": [{ "type": "agent", "id": "agent_01Lm4cV8yQ2tNs7XbKdR5h" }]
    },
    "subagents": {
      "type": "enabled",
      "predefined_agents": [
        { "type": "agent", "id": "agent_01J8XkN5uT3vHpLqRfWdY2" },
        { "type": "agent", "id": "agent_01HqR2k7vXbZ9mNpL3wYcT" }
      ]
    },
    "advisor": { "type": "enabled", "model": "claude-opus-5-5" }
  }
}
```

每项设置都可以单独启用或禁用：

| 设置          | 开启的功能                         | 智能体列于                                 | 默认值 |
| ----------- | ----------------------------- | ------------------------------------- | --- |
| `workflows` | 动态工作流                         | `workflows.predefined_agents`，最多 20 个 | 启用  |
| `subagents` | 委派给子智能体：您列出的智能体，以及智能体自行定义的智能体 | `subagents.predefined_agents`，最多 20 个 | 启用  |
| `advisor`   | 顾问模型                          | 无。设置 `model`。                         | 禁用  |

关于每个列表中应包含的内容，以及如何仅允许您列出的智能体，请参阅[预定义智能体和内联智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#predefined-and-inline-agents)。

请牢记以下几点：

* **更新：**&#x4FDD;持 `multiagent_20261001` 类型的更新只会更改其发送的内容：

  * 您省略的设置或字段保留其存储的值。
  * 您以 `null` 发送的设置或字段采用其默认值，其内部的所有内容也是如此，无论您之前存储了什么。
  * 您发送的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表会替换存储的列表。
  * 您以不同 `type` 发送的设置会替换存储的设置，您在其中省略的字段采用其默认值。
  * 您发送的每个对象都需要包含其 `type`，已启用的 `advisor` (csharp, go: `Advisor`) 需要包含其 `model` (csharp, go: `Model`)。

* **关闭：**&#x5C06; `workflows` (csharp, go: `Workflows`) 设置为 `{"type": "disabled"}`。它会保持关闭，直到某次更新再次将其开启。发送 `null` 会将其开启，因为其默认值为启用。`subagents` (csharp, go: `Subagents`) 同理。

* **现有会话：**&#x4F1A;话在创建时复制设置，因此之后更改智能体不会影响已存在的会话。

* **可列出的智能体：**&#x8BBE;置了 `multiagent` (csharp, go: `Multiagent`) 的智能体不能被列为另一个智能体的子智能体，也不能列在另一个智能体的 `workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中。智能体可以使用 `{"type": "self"}` 将自身列在任一列表中，读回时显示为其自身的 `id` 和 `version`。

* **工具名称：**`ant__` 前缀是保留的。如果您的智能体已有名称以 `ant__` 开头的自定义工具，则更新会以 400 错误失败，直到您发送不含该名称的 `tools` (csharp, go: `Tools`)。如果会话的智能体或任一列表中的智能体拥有此类工具，新会话也会以 400 错误被拒绝。请在开启动态工作流的同一次更新中重命名或移除该工具。请参阅[自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/tools#custom-tools)。

* **预算：**&#x5728;创建会话时设置[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)以限制会话的支出（包括运行）；您无法为现有会话添加预算。当会话达到预算时，运行会暂停；当您提高或移除预算时，因预算而暂停的运行会恢复，除非它们同时也被中断暂停。请参阅[成本与预算](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#cost-and-budgets)。

### 告诉智能体何时使用运行

开启动态工作流后，智能体便能够启动工作流运行。由智能体决定何时启动。为了引导这一选择，请在智能体的系统提示中添加如下指令。[开启动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#turn-on-dynamic-workflows)中的合同审查智能体使用的就是这个系统提示：

```text wrap
You review contracts. When you're asked to review more than a few contracts, start a workflow run that reads them in parallel and combines the findings. Review one or two contracts yourself, without a run.
```

要对其进行调整，请列出您的智能体领域中需要运行的任务，以及智能体应自行完成的小任务。您还可以引导运行如何完成工作，如以下示例所示：

* **如何拆分工作：**"在运行中，每份合同使用一个智能体。然后让第二个智能体审查每一份合同，而不仅仅是第一个智能体发现问题的那些，并查找第一个智能体遗漏的内容。任何未通过审查的内容都要重做并再次审查。"
* **智能体失败时怎么办：**"一个智能体失败不得导致运行失败。如果某个智能体无法读取某份合同，请将该合同列为未覆盖。运行结束时，告诉我哪些合同未被覆盖。"
* **时间限制：**"为运行设置一小时的时间限制。"
* **先做计划：**"在启动运行之前，描述其各个阶段以及每个阶段运行多少个智能体。然后等待我的许可。"运行启动前没有审批步骤，因此没有任何机制强制执行这一点。

请仅在需要运行的任务中使用运行，因为运行中的每个智能体都会消耗令牌。

然后像使用任何智能体一样，使用该智能体[创建会话](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)，并在 `user.message` 中描述工作。您也可以在该消息中请求启动运行。

关于运行的事件、在有运行进行中时中断会话、运行进行期间会发生哪些变化，以及预算和限制，请参阅[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs)。

## 为会话配置顾问

智能体 `multiagent` (csharp, go: `Multiagent`) 块中已启用的 `advisor` (csharp, go: `Advisor`) 设置会为会话的主线程提供一个**顾问（advisor）：**&#x4E00;个可在轮次中途咨询以获取战略指导的模型，例如规划方法、克服障碍，或在完成前审查工作。该设置默认处于禁用状态。要启用它，请将 `advisor` (csharp, go: `Advisor`) 设置为 `{"type": "enabled", "model": "..."}`，它恰好有两个字段：`type` 和 `model`：

<CodeGroup defaultLanguage="CLI">
  ```bash cURL
  curl -fsS https://api.anthropic.com/v1/agents \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: managed-agents-2026-04-01" \
    -H "content-type: application/json" \
    -d '{
      "name": "Backend engineer",
      "model": "claude-sonnet-5",
      "system": "You implement backend features end to end. Consult the advisor before major backend design decisions.",
      "multiagent": {
        "type": "multiagent_20261001",
        "advisor": {"type": "enabled", "model": "claude-opus-5-5"}
      }
    }'
  ```

  <CodeGroupItem>
    ```bash CLI
    ant apply backend-engineer.md
    ```

    <File filename="backend-engineer.md">
      ```markdown
      ---
      name: Backend engineer
      model: claude-sonnet-5
      multiagent:
        type: multiagent_20261001
        advisor:
          type: enabled
          model: claude-opus-5-5
      ---

      You implement backend features end to end. Consult the advisor before major backend design decisions.
      ```
    </File>
  </CodeGroupItem>

  ```python Python
  agent = client.beta.agents.create(
      name="Backend engineer",
      model="claude-sonnet-5",
      system="You implement backend features end to end. Consult the advisor before major backend design decisions.",
      multiagent={
          "type": "multiagent_20261001",
          "advisor": {"type": "enabled", "model": "claude-opus-5-5"},
      },
  )
  print(agent.id)
  ```

  ```typescript TypeScript
  const agent = await client.beta.agents.create({
    name: "Backend engineer",
    model: "claude-sonnet-5",
    system:
      "You implement backend features end to end. Consult the advisor before major backend design decisions.",
    multiagent: {
      type: "multiagent_20261001",
      advisor: { type: "enabled", model: "claude-opus-5-5" }
    }
  });
  console.log(agent.id);
  ```

  ```csharp C#
  var agent = await client.Beta.Agents.Create(new()
  {
      Name = "Backend engineer",
      Model = BetaManagedAgentsModel.ClaudeSonnet5,
      System = "You implement backend features end to end. Consult the advisor before major backend design decisions.",
      Multiagent = new BetaManagedAgentsMultiagent20261001Params
      {
          Advisor = new BetaManagedAgentsMultiagentAdvisorEnabledParams { Model = "claude-opus-5-5" },
      },
  });
  Console.WriteLine(agent.ID);
  ```

  ```go Go
  agent, err := client.Beta.Agents.New(ctx, anthropic.BetaAgentNewParams{
  	Name:   "Backend engineer",
  	Model:  anthropic.BetaManagedAgentsModelConfigParams{ID: anthropic.BetaManagedAgentsModelClaudeSonnet5},
  	System: anthropic.String("You implement backend features end to end. Consult the advisor before major backend design decisions."),
  	Multiagent: anthropic.BetaManagedAgentsMultiagentParamsUnion{
  		OfMultiagent20261001: &anthropic.BetaManagedAgentsMultiagent20261001Params{
  			Advisor: anthropic.BetaManagedAgentsMultiagentAdvisorParamsUnion{
  				OfEnabled: &anthropic.BetaManagedAgentsMultiagentAdvisorEnabledParams{
  					Model: "claude-opus-5-5",
  				},
  			},
  		},
  	},
  })
  if err != nil {
  	panic(err)
  }
  fmt.Println(agent.ID)
  ```

  ```java Java
  var agent = client.beta().agents().create(
      AgentCreateParams.builder()
          .name("Backend engineer")
          .model(BetaManagedAgentsModel.CLAUDE_SONNET_5)
          .system("You implement backend features end to end. Consult the advisor before major backend design decisions.")
          .multiagent(BetaManagedAgentsMultiagent20261001Params.builder()
              .advisor(BetaManagedAgentsMultiagentAdvisorEnabledParams.builder()
                  .model("claude-opus-5-5")
                  .build())
              .build())
          .build()
  );
  IO.println(agent.id());
  ```

  ```php PHP
  $agent = $client->beta->agents->create(
      name: 'Backend engineer',
      model: BetaManagedAgentsModel::CLAUDE_SONNET_5,
      system: 'You implement backend features end to end. Consult the advisor before major backend design decisions.',
      multiagent: [
          'type' => 'multiagent_20261001',
          'advisor' => ['type' => 'enabled', 'model' => 'claude-opus-5-5'],
      ],
  );
  echo $agent->id, PHP_EOL;
  ```

  ```ruby Ruby
  agent = client.beta.agents.create(
    name: "Backend engineer",
    model: Anthropic::Beta::BetaManagedAgentsModel::CLAUDE_SONNET_5,
    system_: "You implement backend features end to end. Consult the advisor before major backend design decisions.",
    multiagent: {
      type: "multiagent_20261001",
      advisor: {type: "enabled", model: "claude-opus-5-5"}
    }
  )
  puts agent.id
  ```
</CodeGroup>

此示例仅设置了 `advisor` (csharp, go: `Advisor`)。另外两项设置 `subagents` (csharp, go: `Subagents`) 和 `workflows` (csharp, go: `Workflows`) 保持其默认值，因此两者均已启用，智能体还可以委派给内联智能体并启动工作流运行。如果希望智能体咨询顾问并自行完成所有工作，请将两者都设置为 `{"type": "disabled"}`。

您不能将顾问列在 `subagents` (csharp, go: `Subagents`) 或 `workflows` (csharp, go: `Workflows`) 的 `predefined_agents` (java, php: `predefinedAgents`; csharp, go: `PredefinedAgents`) 列表中。已启用的 `advisor` (csharp, go: `Advisor`) 会保留名称 `anthropic.advisor`。在其启用期间，两个列表都不能包含名称恰好为 `anthropic.advisor` 的智能体：此类请求会以 400 验证错误被拒绝。

顾问模型必须达到最低能力水平，且智能体自身的模型能力不得高于其顾问；能力相当的模型可以配对。无效的配对会在保存智能体时以 400 验证错误被拒绝，在创建会话时也会再次被拒绝。每次咨询开始时也会检查配对：如果配对已不再有效，该次咨询会失败，而会话会继续进行。有效的配对遵循 Advisor 工具的[模型兼容性](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#model-compatibility)表。

顾问也可作为 [Messages API 上的服务器工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)使用。Managed Agents 接口在配置和交付方式上有所不同：`advisor` (csharp, go: `Advisor`) 设置没有 `max_uses`、`max_tokens` 或 `caching` 字段，且建议通过线程事件而非 `advisor_tool_result` 块送达。

### 咨询的工作方式

每次咨询都作为平台生成的、名为 `anthropic.advisor` 的线程运行，该线程在咨询完成时自行终止，建议会以 `agent.thread_message_received` 事件的形式送达主线程。咨询会发出标准线程事件，并以保留名称 `anthropic.advisor` 标识（线程生命周期事件将其作为 `agent_name` (java, php: `agentName`; csharp, go: `AgentName`) 携带，建议送达事件将其作为 `from_agent_name` (java, php: `fromAgentName`; csharp, go: `FromAgentName`) 携带），通常按以下顺序：

1. `session.thread_created`
2. `session.thread_status_running`
3. `agent.thread_message_received`（建议）
4. `session.thread_status_idle`（`stop_reason: end_turn`）
5. `session.thread_status_terminated`

咨询不会发出 `agent.tool_use` 事件，会话的事件流中也不会出现 `agent.thread_message_sent` 事件，因为咨询输入是由平台组装的，而不是由智能体发送的。如果您列出顾问线程自身的事件，建议也会以 `agent.thread_message_sent` 事件的形式出现在其中。建议送达（事件 3）不保证在顾问线程的空闲和终止事件之前到达，因此不要将这些事件视为建议已送达的信号。

您的客户端能否读取建议取决于顾问模型的策略。这与 Messages API Advisor 工具上的[结果变体](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#result-variants)划分一致。在那里返回明文结果的顾问模型，在这里会送达可读文本。返回加密结果的模型则会在每个客户端接口上送达 `[{"type": "redacted"}]` 占位符，而智能体仍会在服务器端读取完整的建议。前面的示例使用您可用的最新 Opus 模型作为顾问。该模型返回加密结果，因此您的客户端只能看到占位符。要在事件流上读取建议，请使用返回明文的顾问，例如 Claude Opus 4.8。模型的策略可能会在 API 不变的情况下发生变化，因此请同时处理 `text` 和 `redacted` 块。某些智能体模型只能与返回加密结果的顾问配对；请参阅 Advisor 工具的[模型兼容性](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#model-compatibility)部分。顾问的思考内容永远不会公开。客户端不能自行发送 `redacted` 块；包含此类块的事件会以 400 验证错误被拒绝。

失败或被中断的咨询永远不会导致智能体的轮次失败：智能体会在收到咨询失败的通用通知后继续运行。在咨询期间发出的会话级 `user.interrupt` 会终止顾问线程，且不会送达任何建议；带有顾问线程 `session_thread_id` (java: `sessionThreadId`; php: `sessionThreadID`; csharp, go: `SessionThreadID`) 的 `user.interrupt` 只会放弃该次咨询。

### 顾问线程

顾问不是子智能体。智能体的 `list_agents` 工具不会显示它，用于向子智能体发送后续消息的 `send_to_agent` 也无法触达它。只有会话的主线程可以咨询它；子智能体不能。

顾问线程不受 25 个子线程的限制。它们会出现在会话的[线程列表](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)中。它们的 `agent` 为顾问形式 `{"type": "advisor", "model": ...}`，其中包含您配置的模型，它们的 `parent_thread_id` 为主线程。

顾问一侧的 “prompt caching”（提示缓存）是自动进行的；无需任何配置。咨询按顾问模型的费率计费，其令牌会出现在顾问线程的用量和会话的用量总计中。它们也会按顾问模型的标价计入[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)。每次咨询都会将主线程迄今为止的对话发送给顾问，因此在长会话后期进行的咨询会使用更多输入令牌。

### 移除顾问

要移除顾问，请[更新智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#update-an-agent)，将 `advisor` (csharp, go: `Advisor`) 设置为 `{"type": "disabled"}`。对于已具有 `multiagent_20261001` 类型的智能体，更新中省略的设置会保留其存储的值，因此您无需再次发送 `subagents` (csharp, go: `Subagents`) 或 `workflows` (csharp, go: `Workflows`)。出于同样的原因，省略 `advisor` (csharp, go: `Advisor`) 的更新会保留顾问。在更改现有智能体的 `multiagent.type` 之前，请参阅[如何将智能体迁移到此类型](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#move-from-the-coordinator-type)。

## 从 `coordinator` 类型迁移

与 `multiagent_20261001` 一样，`coordinator` 类型允许智能体委派给您列出的子智能体并咨询顾问模型。使用 `coordinator` 类型时，智能体无法启动工作流运行，也无法自行定义子智能体。API 接受这两种类型。

要将智能体迁移到 `multiagent_20261001`，请使用该类型的 `multiagent` 块[更新智能体](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#update-an-agent)：

1. 将 `type` 设置为 `"multiagent_20261001"`。
2. 在 `subagents.predefined_agents` 中列出该智能体可以调用的智能体。条目保持其现有形式。
3. 如果智能体有顾问，请将 `advisor` 设置为 `{"type": "enabled", "model": "..."}`，并填入顾问的模型。
4. 在一次更新中发送整个块。

例如，以下块为智能体配置了两个子智能体和一个顾问：

```json
{
  "multiagent": {
    "type": "multiagent_20261001",
    "subagents": {
      "type": "enabled",
      "predefined_agents": [
        { "type": "agent", "id": "agent_01J8XkN5uT3vHpLqRfWdY2" },
        { "type": "agent", "id": "agent_01HqR2k7vXbZ9mNpL3wYcT" }
      ]
    },
    "advisor": { "type": "enabled", "model": "claude-opus-5-5" }
  }
}
```

该块省略了 `workflows` 和 `subagents.inline_agents`，因此两者都采用其默认值并处于启用状态。迁移后的智能体还可以启动工作流运行并自行定义子智能体。要保持其中任一项关闭，请在同一次更新中将其设置为 `{"type": "disabled"}`。请参阅[开启动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#turn-on-dynamic-workflows)和[预定义智能体和内联智能体](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#predefined-and-inline-agents)。

如果智能体有顾问但没有子智能体，请将 `subagents` 设置为 `{"type": "disabled"}` 以保持委派关闭。禁用 `subagents.inline_agents` 且未列出任何智能体的请求会以 400 错误失败。

此次更新会替换整个 `multiagent` 块，因此请像示例那样再次发送智能体列表和顾问。您省略的内容会采用其默认值：

* **`subagents.predefined_agents`：**&#x5217;表为空，因此智能体会失去您列出的子智能体。
* **`advisor`：**&#x987E;问被禁用。

一旦智能体具有 `multiagent_20261001` 类型，之后的更新中省略的设置会保留其存储的值。

此次更新不会更改已存在的会话。要使用新块，请在更新后创建会话。

## 可用性

多智能体编排在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台     |
| ---- | ------- |
| Beta | 所有支持的平台 |

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
