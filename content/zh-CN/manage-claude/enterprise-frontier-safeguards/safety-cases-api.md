---
title: Safety Cases API
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api
description: 使用 Compliance API 通过 HTTP 列出、读取和响应 Safety Case。
---

Safety Cases API 通过 HTTP 提供 Safety Case。它处于测试阶段。您可以在 **Safety Case and Transcript storage** 下为每个工作区选择一个 [Case 存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#glossary)。如果没有存储桶，对于带有用户配置文件的消息，您将不会收到任何 Safety Case。该 API 是 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api) 的一部分，并且仅在 Claude Platform 上可用。要了解您的团队如何审查和响应 Safety Case，请参阅[审查 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

## 获取访问权限

要获取 Safety Cases API 的访问权限：

1. **开启 Compliance API。** 您的组织必须已开启它。如果未开启，请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access#set-up-the-compliance-api) 将其开启。
2. **创建 Admin API 密钥。** 以组织管理员身份在 Claude Console 中创建一个 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-console-organization)。无需选择任何权限范围。只有在该 API 对您的组织可用期间创建的 Admin API 密钥才能访问 Safety Cases API。您之前创建的 Admin API 密钥永远不会获得此访问权限。如果您关闭 Compliance API，Safety Cases API 会以 400 错误拒绝您的密钥的请求。

要测试该密钥，请使用它[列出您的 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#poll)：

| 响应 | 含义及操作 |
| ----- | ----- |
| 200（即使列表为空） | 您的密钥可以列出 Safety Case。您只会收到未启用 Anthropic Human Review 的工作区的 Safety Case。 |
| 401、403 或 404 | 您的密钥无法列出 Safety Case。请检查您是否发送了完整的密钥，然后创建另一个 Admin API 密钥并重试。 |
| 新密钥返回 403 或 404 | 该 API 对您的组织不可用。 |

请删除您不需要的所有 Admin API 密钥。

该密钥只能访问其所属组织的 Safety Case 和消息，即使在父组织下也是如此。它是一个完整的 Admin API 密钥。在该 API 对您的组织可用期间，您创建的每个 Admin API 密钥都会获得此访问权限。因此，请限制可以创建和持有此类密钥的人员。测试组织的密钥不会获得此访问权限。

该密钥会获得以下全部三个权限范围：

| 权限范围 | 允许的操作 |
| ----- | ----- |
| `read:compliance_review_cases` | 列出和读取 Safety Case 和 Safety Notice，其中从不包含消息的提示或响应。 |
| `read:compliance_review_items` | 读取被标记消息的提示、响应和发送者。 |
| `write:compliance_review_cases` | 响应 Safety Case。 |

其他密钥也可以读取这些数据：

* **Safety Case 和 Safety Notice：** 在该 API 可用的情况下，具有 [`read:org_audit`](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#choose-scopes-for-a-claude-enterprise-key) 的密钥只能读取其自身组织的 Safety Case 和 Safety Notice。仅当该组织有父组织时，它们才能这样做。
* **被标记的消息：** 在某些组织中，具有 `read:org_audit` 的密钥还可以读取被标记的消息，具有 `read:compliance_user_data` 的密钥也可以。其中一些密钥可以访问其父组织下每个组织的被标记消息。

请在每个请求中发送 [`anthropic-version`](https://platform.claude.com/docs/zh-CN/api/versioning) 标头。未包含该标头的请求可能会返回 400 错误。出现 429 或 503 错误后，请稍后重试该请求。

## 轮询 Safety Case

请定期检查 Safety Cases API 中是否有新的 Safety Case，以便您能够及时响应。

首次运行时，请省略 `updated_at[gte]`。然后，存储您已读取的最大 `updated_at`，并在下次运行时将其作为 `updated_at[gte]` 发送：

```bash cURL
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/review/cases" \
  --data-urlencode "updated_at[gte]=2026-11-23T18:15:34Z" \
  -d "order_by=updated_at" \
  -d "order=asc" \
  -d "limit=100" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY" \
  -H "anthropic-version: 2026-10-21"
```

```json Response
{
  "data": [
    {
      "type": "compliance_review_case",
      "id": "rvc_01Hq3Zt8KpWv5NcR2yLbXs7D",
      "organization_id": "abcdef01-2345-6789-abcd-ef0123456789",
      "status": { "type": "open" },
      "created_at": "2026-11-23T18:42:07.106255Z",
      "updated_at": "2026-11-23T18:42:07.106255Z",
      "workspace_id": "wrkspc_01AbCdEfGhJkLmNpQrStUvWx",
      "policy_categories": ["cyber"],
      "enforcement_action": null,
      "transcripts": [
        {
          "item_id": "msg_01XFDUDYJgAACzvnptvVoYEL",
          "user_profile_id": null
        }
      ]
    }
  ],
  "next_page": null
}
```

此请求会返回自该时间以来开启、关闭或收到 Safety Notice 的 Safety Case。要读取单个 Safety Case，请发送 `GET /v1/compliance/review/cases/{case_id}`。

构建轮询作业时，请遵循以下建议：

* **读取每一页。** 要读取下一页，请将 `next_page` 作为 `page` 发送，直到请求返回的 `next_page` 为 `null`。即使设置了 `next_page`，页面也可能条目很少或为空。
* **不要在多次运行之间保留 `next_page`。** 它仅在一次列表结果内有效，如果筛选条件或排序不同，API 会以 400 错误拒绝它。
* **按 `id` 更新您的每个 Safety Case 副本。** API 还会返回比您发送的时间稍早的 Safety Case，并且在 Safety Case 关闭或收到 Safety Notice 时会再次返回它。
* **按 `updated_at` 而不是 `created_at` 进行筛选。** 按 `created_at` 筛选会遗漏在您首次读取后关闭或收到 Safety Notice 的 Safety Case，并且可能会遗漏新的 Safety Case。

已关闭的 Safety Case 的 `status.type` 设置为 `closed`，`status.reason.type` 说明关闭原因：

| `status.reason.type` | 含义 |
| ----- | ----- |
| `responded` | Anthropic 已收到您的响应。 |
| `window_ended` | Anthropic 在 `created_at` 之后的 7 个日历日内未收到有效响应。 |
| `ended_by_anthropic` | Anthropic 在此之前结束了该 Safety Case。 |

Anthropic 可能会新增字段、`policy_categories` 值、`enforcement_action` 类型和关闭原因，因此请接受您不认识的值。

### 轮询 Safety Notice

Safety Notice 是 Anthropic 发送的关于您的一个或多个 Safety Case 或关于您整个组织的消息。它可能要求在指定时间之前作答。其 `kind.type` 说明了 Safety Notice 的类型：

| `kind.type` | 含义 |
| ----- | ----- |
| `response_requested` | 要求您响应的请求。要答复，请[响应 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#respond)。 |
| `warning` | 警告。 |
| `question` | 问题。要答复，请联系您的 Anthropic 客户团队。 |

Anthropic 可能会新增其他类型，因此请接受您不认识的值。不要依赖 Safety Notice 来提醒您处于开启状态的 Safety Case：在您的 Case 存储桶中没有文件的 Safety Case 不会收到自动提醒。

要读取 Safety Notice，请轮询 `GET /v1/compliance/review/notices`：

* **发送 `created_at[gte]` 和 `order=asc`，** 并存储您已读取的最大 `created_at`。Safety Notice 没有 `updated_at` 或 `order_by`，并且永远不会更改。
* **读取每一页，** 就像读取 Safety Case 一样。
* **按 `id` 匹配 Safety Notice，** 因为结果可能会重复。

Safety Notice 列表如下所示：

```json Response
{
  "data": [
    {
      "type": "compliance_review_notice",
      "id": "rvn_01Jt5Yw2MnQx8VbK4zRcHd6F",
      "organization_id": "abcdef01-2345-6789-abcd-ef0123456789",
      "case_ids": ["rvc_01Hq3Zt8KpWv5NcR2yLbXs7D"],
      "kind": { "type": "response_requested" },
      "created_at": "2026-11-27T18:00:00Z",
      "message": "Anthropic has found no valid response to the Case in case_ids. Review the Case and write your response file before the time in respond_by. If you wrote a valid response file in the last hour, ignore this notice.",
      "respond_by": "2026-11-30T18:42:07Z"
    }
  ],
  "next_page": null
}
```

如果 Safety Notice 没有截止时间，`respond_by` 为 `null`。[Safety Notice 文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#notice)具有相同的字段，但不包含 `type` 和 `organization_id`，并包含 `schema_version`。

### 字段与文件的区别

Anthropic 还可以将 Safety Case 写入您自己云账户中的存储桶，即 [Safety Case 文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files)。这些文件是读取和响应 Safety Case 的另一种方式。文件使用 API 的字段名称，但存在以下区别：

| 在 API 中 | 在 [Safety Case 文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#safety-case-file)中 |
| ----- | ----- |
| `type`、`organization_id` | 不在文件中 |
| 如果没有，或者消息已超过 28 天，`transcripts[].user_profile_id` 为 `null` | 如果没有，则省略该键 |
| 不在 API 中 | `schema_version`、`transcripts[].object_key`、`derived_data_object_keys` |
| 已关闭的 Safety Case 始终具有 `status.reason` | `status.reason` 可以为 `null` |
| 当 Safety Notice 提及该 Safety Case 时，`updated_at` 也会更改 | 仅当 Anthropic 写入文件时，`updated_at` 才会更改 |
| 时间精确到秒的小数部分 | 时间精确到整秒。`created_at` 可能稍早。 |

## 读取消息

获取 `transcripts` 中的每个 `item_id`。您能否读取使用用户配置文件发送的消息取决于密钥：

* **来自 Claude Console 的 Admin API 密钥**无法读取此类消息。该请求会返回 404 错误。如果该工作区有 Case 存储桶，Anthropic 会将该消息的[对话记录](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#transcript)写入该工作区的 Case 存储桶。
* **其他密钥**在某些组织中可以读取此类消息（请参阅[获取访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#get-access)）。使用此类密钥时，请在 `anthropic-user-profile-id` 标头中发送该用户配置文件的 ID。对于任何其他消息，请省略该标头。错误或缺失的 ID 会返回 404 错误，对于您未使用用户配置文件发送的消息，发送 ID 同样会返回 404 错误。

```bash cURL
curl --fail-with-body -sS \
  "https://api.anthropic.com/v1/compliance/safeguards/items/msg_01XFDUDYJgAACzvnptvVoYEL" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY" \
  -H "anthropic-version: 2026-10-21"
```

响应包含该消息以及 Anthropic 记录的发送者：

```json Response
{
  "type": "compliance_safeguards_review_item",
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "organization_uuid": "abcdef01-2345-6789-abcd-ef0123456789",
  "sampled_at": "2026-11-23T17:58:12.482913Z",
  "content_expires_at": "2026-12-21T17:58:12.482913Z",
  "product_surface": "api",
  "model": "claude-opus-5-5",
  "workspace_id": "wrkspc_01AbCdEfGhJkLmNpQrStUvWx",
  "api_key_id": null,
  "user": { "id": "user_01JkWm4Rt7YpXc2NvBq9sZdH", "email_address": "user@example.com" },
  "user_profile_id": null,
  "refs": {
    "batch_id": null,
    "local_session_id": null,
    "claude_chat_id": null,
    "claude_chat_message_id": null,
    "remote_session_id": null,
    "remote_session_event_id": null,
    "agent_session_id": null
  },
  "content": {
    "request": {
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{ "role": "user", "content": "Hello, Claude" }]
    },
    "response": {
      "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
      "type": "message",
      "role": "assistant",
      "content": [{ "type": "text", "text": "Hello! How can I help you today?" }]
    }
  },
  "content_unavailable_reason": null
}
```

`content` 包含存储的请求和响应。它不是完全一致的副本：Anthropic 会将部分 URL 截断为其主机名、遮盖 MCP 服务器令牌、移除思考块，并可能将图像或文档替换为调整大小后的图像或占位符。

Anthropic 为每条消息记录一个发送者：

| 发送者 | `user` | `api_key_id` |
| ----- | ----- | ----- |
| [个人密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)或用户登录 | 该账户的 ID 和电子邮件地址 | `null` |
| 工作区密钥 | `null` | 该密钥的 ID |
| 服务账户，或未记录任何发送者 | `null` | `null` |

`user` 标识该账户，而不是编写提示的人员。要找到该人员，请使用您自己的日志（请参阅[记录消息 ID](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#log-message-ids)）。在没有父组织的组织中，`email_address` 始终为 `null`，在其他组织中可能为 `null`。

Safety Case 可能会列出您无法读取的消息。下表显示了请求可能返回的结果：

| 结果 | 含义 | 操作 |
| ----- | ----- | ----- |
| 404 错误 | 例如，消息已过期或不再被标记。 | 404 错误可能是暂时的，因此在将消息视为无法读取之前，请稍后重试。 |
| 每条消息都返回 404 错误 | 如果 Anthropic 尚未为您的组织开启消息读取功能，或者您的组织不再拥有 Anthropic 对关闭 Anthropic Human Review 的批准，就会出现这种情况。 | 请参阅[寻求帮助](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#get-help)。404 错误可能是暂时的，因此在将消息视为无法读取之前，请稍后重试。 |
| 503 错误 | 该错误可能是暂时的。 | 在将消息视为无法读取之前，请稍后重试。 |
| 返回消息，但 `content` 设置为 `null` | Anthropic 没有文本副本。`content_unavailable_reason` 为 `not_captured` 或 `use_ref`，并且 Anthropic 可能会新增其他值。 | 使用 `use_ref` 时，`refs` 可能包含一个指明该对话的 ID。请使用有权访问该 ID 的密钥，通过该 ID 对应的 Compliance API 端点读取该对话。 |

请在 Safety Case 送达后立即读取并保存每条消息，且不晚于 `content_expires_at`。该时间最多为 `sampled_at` 之后的 28 天，并且处于开启状态的 Safety Case 不会延长该时间。消息可能在该时间之前就无法读取。

## 响应 Safety Case

调查 Safety Case 后，请进行响应以告知 Anthropic 您的调查结果。[响应 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#respond-to-a-case) 说明了响应所确认的内容，[如果您不响应](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases#if-you-dont-respond) 说明了未响应时会发生的情况。

<Warning>
  响应会关闭 Safety Case，因此您之后无法更改您的响应。
</Warning>

请在 Safety Case 的 `created_at` 之后的 7 个日历日内发送一个请求：

```bash cURL
curl --fail-with-body -sS \
  "https://api.anthropic.com/v1/compliance/review/cases/rvc_01Hq3Zt8KpWv5NcR2yLbXs7D/respond" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY" \
  -H "anthropic-version: 2026-10-21" \
  -H "content-type: application/json" \
  -d '{
    "determination": "violation",
    "policy_categories": ["cyber"]
  }'
```

`determination` 为必填项，使用 `violation` 时 `policy_categories` 也为必填项，`verdicts` 为可选项：

| 字段 | 发送的内容 |
| ----- | ----- |
| `determination` | 针对整个 Safety Case 的 `violation` 或 `no_violation` |
| `policy_categories` | 使用 `violation` 时，提供您确认的一到四个类别，例如 `cyber`，如果没有合适的类别则使用 `other`。`benign` 和 `undetermined` 会返回 400 错误。使用 `no_violation` 时，请省略该字段或发送空列表。 |
| `verdicts` | 若要分别判定各条消息，请提供一个列表，其中每项包含 `item_id`、`determination` 和 `policy_category`（一个类别或 `null`，使用 `no_violation` 时始终为 `null`）。 |

200 响应会返回已关闭的 Safety Case，其 `status.reason.type` 设置为 `responded`。这表示 Anthropic 已收到您的响应。

Anthropic 以第一个响应为准，无论它是通过 API 还是以文件形式送达：

| 您接下来发送的内容 | API 返回的内容 |
| ----- | ----- |
| 再次发送相同的响应 | 200 响应。不会更改任何内容，因此可以安全地重试。 |
| 对已关闭的 Safety Case 发送任何其他有效响应 | 400 错误，其 `error.details.error_code` 设置为 `case_closed`。 |
