---
title: 分析 API
url: https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api
description: 了解您的组织需要哪种分析 API 和 API 密钥，然后配置对 Claude Code 生产力指标或 Claude Enterprise 参与度和采用数据的访问权限。
---

Anthropic 提供两种产品分析 API，您使用哪一种取决于您的组织管理的是哪种 Claude 产品：

* **Claude Code 分析 API** 为使用 Claude Platform 的组织报告每日 Claude Code 生产力指标。它是 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 的一部分，使用 Admin API 密钥。
* **Claude Enterprise Analytics API** 为 Claude Enterprise 组织报告跨 Claude 产品（聊天、项目、Claude Code 等）的全组织范围的参与度、采用情况和成本数据。它使用在 claude.ai 中创建的 Analytics API 密钥。

这两种 API 使用不同的密钥类型，由不同的角色在不同的位置创建。本页介绍哪种 API 适合您的组织，以及如何创建正确的密钥。

## 您需要哪种 API？

| API                                 | 密钥类型                               | 创建位置                                                                                                                   | 谁可以创建 | 涵盖内容                                                     |
| ----------------------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ----- | -------------------------------------------------------- |
| **Claude Code 分析 API**              | Admin API 密钥（`sk-ant-admin01-...`） | Claude Console 中的[管理员密钥](https://platform.claude.com/settings/admin-keys)页面（`platform.claude.com/settings/admin-keys`） | 组织管理员 | 每位用户的每日 Claude Code 指标：会话、代码行数、提交、拉取请求、工具接受情况，以及按模型估算的成本 |
| **Claude Enterprise Analytics API** | Analytics API 密钥                   | [claude.ai > 组织设置 > API](https://claude.ai/admin-settings/api-access)                                                  | 主要所有者 | 全组织范围的参与度和采用情况（用户活动、活跃用户摘要、项目、技能和连接器使用情况），以及成本和使用报告      |

这两种密钥类型不可互换：Admin API 密钥无法调用 Claude Enterprise Analytics API，Analytics API 密钥也无法调用 Admin API。唯一的例外是 [用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)，Claude Enterprise 组织使用其 Analytics API 密钥调用该 API。两种产品分析 API 都出现在 [Admin API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization)下，但它们是使用不同密钥类型的独立 API。如果您的组织同时使用 Claude Platform 和 Claude Enterprise，您可以配置两种密钥，并分别使用每种 API 获取其对应的数据。

<Note>
  想要获取令牌使用量和成本数据而非产品分析数据？请参阅 [用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)，它同时服务于 Claude Console 和 Claude Enterprise 组织。
</Note>

<Note>
  如果您希望在产品中而非以编程方式查看参与度和采用数据，请使用 claude.ai 中的[分析仪表板](https://claude.ai/analytics/activity)。对于治理和审计用例（单个用户操作、原始活动事件、对话内容），请参阅 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)。
</Note>

## 获取 Claude Code Analytics API 的访问权限

Claude Code Analytics API 对所有有权访问 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 的组织开放，并且可免费使用。

<Steps>
  <Step title="创建 Admin API 密钥">
    按照[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-console-organization)中的步骤操作。
  </Step>

  <Step title="调用 API">
    在 `x-api-key` 标头中传递密钥：

    ```bash
    curl "https://api.anthropic.com/v1/organizations/usage_report/claude_code?starting_at=2025-09-08" \
      -H "anthropic-version: 2026-10-21" \
      -H "x-api-key: $ANTHROPIC_ADMIN_KEY"
    ```
  </Step>
</Steps>

有关可用指标、请求参数和响应架构，请参阅 [Claude Code 分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/claude-code-analytics-api) 指南和 [API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization/usage_report/retrieve_claude_code)。

## 获取 Claude Enterprise Analytics API 的访问权限

Claude Enterprise Analytics API 对 Claude Enterprise 组织开放。参与度和采用数据在所有 Enterprise 计划中均可用。成本和使用端点适用于基于用量的 Enterprise 计划；对于基于席位的 Enterprise 计划，它们仅反映使用额度。

<Steps>
  <Step title="以主要所有者身份登录">
    只有组织的主要所有者才能启用 API 访问并创建 Analytics API 密钥。
  </Step>

  <Step title="启用 API 访问并创建密钥">
    前往 [claude.ai > 组织设置 > API](https://claude.ai/admin-settings/api-access) 并启用公共 API 访问，然后创建 Analytics API 密钥。密钥具有 `read:analytics` 作用域。复制显示的密钥并将其存储在您的密钥管理器中。
  </Step>

  <Step title="调用 API">
    在 `x-api-key` 标头中传递密钥，并在每个请求中包含 [`anthropic-version`](https://platform.claude.com/docs/zh-CN/api/versioning) 标头。端点位于 `https://api.anthropic.com/v1/organizations/analytics/` 下。有关请求示例、参数和响应架构，请参阅 [Claude Enterprise Analytics API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics)。
  </Step>
</Steps>

Claude Enterprise Analytics API 提供：

* **用户活动：** 每位用户的每日指标，涵盖聊天（对话、消息、项目、文件、artifacts）、Claude Code（会话、提交、拉取请求、代码行数、工具操作）、Chat and Cowork unified（聊天活动和 Cowork 会话）以及其他 Claude 产品
* **活动摘要：** 组织级别的每日、每周和每月活跃用户、席位数量和待处理邀请
* **项目、技能和连接器使用情况：** 聊天项目、技能和连接器的采用情况细分
* **成本和用量报告：** 按用户和组织级别随时间变化的令牌用量和成本（基于用量的 Enterprise 计划）

<Note>
  `/v1/organizations/analytics/` 下的成本和用量报告端点（`usage_report`、`cost_report`、`user_usage_report` 和 `user_cost_report`）属于旧版端点，但仍受支持。对于新的集成，请使用[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)。要迁移现有集成，请参阅[从 Claude Enterprise 用量和成本报告迁移](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#migrate-from-enterprise-analytics-reports)。其他 Claude Enterprise Analytics API 端点不属于旧版。
</Note>

有关端点详情、参数和响应架构，请参阅 [Claude Enterprise Analytics API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics)。以下各节介绍适用于这些端点的数据新鲜度、指标定义和操作指南。

## 数据可用性和新鲜度

Claude Enterprise Analytics API 数据适用于 2026 年 1 月 1 日及之后的日期。

**参与度和采用端点**（用户活动、摘要、项目、技能、连接器）返回您指定日期的每日快照。某一天的数据通常在次日约 13:00–13:30 UTC 可用（1 天延迟）。在此之前，最近可用的日期通常是当前 UTC 日期的 2 天前。当上游数据管道运行延迟时，数据会更晚到达，且确切的新鲜度因查询而异。不要假设固定时间，而应检查错误响应：请求尚不可用的日期会返回 400 错误，其中会指明最近可用的日期。如果数据在远超典型延迟后仍不可用，通常表示 Anthropic 端的数据管道出现故障；如果缺口持续存在，请联系支持团队。

**成本和用量端点**遵循不同的新鲜度模型。数据通常在底层用量发生后 4 小时内可用，但可能需要长达 24 小时。随着延迟事件到达和对账运行，数值可能会被修订，直至日历月结束后约 7 天。例如，10 月 1 日的数值可能会一直变化到 11 月 7 日左右。如需用于开票级别的总计，请仅查询已结束至少 7 天的月份。

<Note>
  成本和使用响应包含 `data_refreshed_at` 时间戳。如果省略 `ending_at`，其默认值为 `starting_at` 之后 31 天或当前时间中较早的一个。该范围可能超过 `data_refreshed_at`，而该时间之后的数据是不完整的。为了在重复调用中获得稳定的结果，请将 `ending_at` 设置为等于或早于先前返回的 `data_refreshed_at` 的值。
</Note>

## 指标的定义方式

**活跃用户。** 如果满足以下任一条件，用户即被计为当天的活跃用户：

* 他们在 Claude 中发送了至少一条聊天消息。
* 他们有至少一个与您的 Claude Enterprise 组织关联的 Claude Code 会话（本地或远程），且该会话包含工具使用或 git 活动。
* 他们有至少一个包含工具使用或消息活动的 Cowork 会话。

**Chat and Cowork unified。** Chat and Cowork unified 将 Cowork 的功能引入 claude.ai 聊天中。Chat and Cowork unified 中的大多数活动会在 `chat_cowork_unified` 产品值下报告，而不是 `chat` 或 `cowork`。

在用户活动以及技能、连接器和插件使用情况中，这部分使用量出现在 `chat_cowork_unified_metrics` 对象中。在活动摘要中，它计入 `chat_cowork_unified_daily_active_user_count`、`chat_cowork_unified_weekly_active_user_count` 和 `chat_cowork_unified_monthly_active_user_count`。

当用户开始使用 Chat and Cowork unified 时，其使用量会从您的聊天和 Cowork 数据中移出，转入 Chat and Cowork unified 数据中。用户开始使用 Chat and Cowork unified 之前各天的使用量仍保留在聊天或 Cowork 下。组织范围的数据（例如总令牌使用量、总成本和总体活跃用户数）不会改变。按产品筛选的成本和使用报告不包含 Chat and Cowork unified 的使用量，除非 `products[]` 包含 `chat_cowork_unified`。

Chat and Cowork unified 用户的某些活动（例如语音模式）仍可能在聊天或 Cowork 下报告。在用户开始使用 Chat and Cowork unified 的当天，其活动也可能在多个产品之间拆分。在为其关闭该功能的当天也是如此。活动的拆分方式不作保证。

用户可能同时出现在聊天或 Cowork 计数以及 Chat and Cowork unified 计数中。当用户在某天使用了 Chat and Cowork unified，但其部分活动仍在聊天或 Cowork 下报告时，就可能出现这种情况。当时间窗口内存在两种活动时（即使在不同日期），每周和每月计数可能会在两者中都包含该用户。例如，时间窗口可能涵盖他们开始使用 Chat and Cowork unified 之前和之后的日期。

当您按项目筛选用户活动时，每天的项目活动只会计入一处。如果用户当天在 Chat and Cowork unified 中发送了聊天消息，则其所有项目活动都计入 `chat_cowork_unified_metrics.chat`。否则，计入 `chat_metrics`。

**按产品划分的指标块。** 按产品划分的指标对象（例如用户活动记录中的 Office Agent 或 Cowork 指标）始终存在于每条记录中。没有使用该产品的组织会看到零计数，而不是 `null` 对象。例外情况是 Chat and Cowork unified：在不提供该功能的部署中，其字段会从响应中省略。这些对象中的单个字段可能为 `null`，表示该值不可用。例如，当您按项目筛选用户活动时，`chat_metrics` 中的 `distinct_skills_used_count` 为 `null`。[Claude Enterprise Analytics API 参考](https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics)中列出了哪些字段可能为 `null`。

**连接器名称。** 连接器名称会跨来源进行规范化。例如，`Atlassian MCP server`、`mcp-atlassian` 和 `atlassian_MCP` 在连接器使用端点中都显示为 `atlassian`。

## 使用 API

**分页游标与发出它们的查询绑定。** 在成本和使用端点上，不要在序列中途更改查询参数：如果您更改 `products[]`、`group_by[]`、`order_by`、日期范围或任何筛选条件并传递旧游标，请求将返回 400 错误。要更改参数，请在不带游标的情况下从第一页重新开始。

**列表参数使用方括号表示法。** 为每个值重复该参数，例如 `products[]=chat&products[]=claude_code`。

**金额字段是以美分为单位的十进制字符串。** 货币金额以十进制字符串形式返回，例如 `"41280.000000"`（表示 412.80 美元）。要转换为美元，请将其解析为十进制数并除以 100。对于可能超过数百万美元的值，请避免使用二进制浮点解析。

**速率限制在组织级别应用**，而不是针对每个密钥，默认值为此 API 中所有端点每分钟 60 个请求。如果这不能满足您的用例，请联系您的 Anthropic 客户团队讨论调整限制。

## 版本控制

在每个请求中发送 `anthropic-version` 标头；有关可用版本，请参阅 [API 版本](https://platform.claude.com/docs/zh-CN/api/versioning)。

## 已知限制

### 通过 Amazon Bedrock 使用 Claude Code

如果您的组织通过 Amazon Bedrock 使用 Claude Code，Claude Enterprise Analytics API 不会返回该使用情况的 Claude Code 活动。

### 使用客户管理的加密密钥的 Claude Code 和 Cowork

如果您的组织使用[客户管理的加密密钥](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)，为了保护您的数据，Anthropic 会在其用量分析中省略 Claude Code 和 Cowork 使用的部分详细信息。因此，请不要使用以下 Claude Enterprise Analytics API 数据来衡量您的组织对 Claude Code 或 Cowork 的使用程度。这些数值可能为零或低于实际使用量：

* **代码更改：** 在用户活动中，`claude_code_metrics.core_metrics` 中的以下字段可能为零：`lines_of_code`（添加和删除的行数）和 `commit_count`。`pull_request_count` 字段可能低于实际的拉取请求数。
* **通过斜杠命令启动的技能：** 如果用户在 Claude Code 或 Cowork 中通过输入斜杠命令（例如 `/skill-name`）启动技能，则该使用不会被计入。这适用于技能使用情况，以及用户活动中的 Cowork 技能计数。Claude 在没有斜杠命令的情况下调用的技能仍会被计入。

有关使用客户管理的加密密钥时的其他分析变更，请参阅 [CMEK 在 Claude Enterprise 上涵盖的内容](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise#turned-off-or-changed)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="Claude Code 分析 API" href="https://platform.claude.com/docs/zh-CN/manage-claude/claude-code-analytics-api">
    使用 Admin API 密钥跟踪 Claude Code 会话、代码更改和工具使用情况。
  </Card>

  <Card title="用量和成本分析 API" href="https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api">
    跟踪您组织的令牌使用量和成本。
  </Card>

  <Card title="Claude Enterprise Analytics API 参考" href="https://platform.claude.com/docs/zh-CN/api/beta/organization/analytics">
    参与度、采用情况和成本数据的端点参考。
  </Card>

  <Card title="设置 Compliance API" href="https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access">
    审计和合规数据使用其自己的密钥类型。
  </Card>
</CardGroup>
