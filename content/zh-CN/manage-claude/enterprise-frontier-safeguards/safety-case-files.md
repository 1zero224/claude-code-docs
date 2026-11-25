---
title: Safety Case 文件
url: https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files
description: Anthropic 写入您的 Case 存储桶的文件，以及您的团队写回的响应文件。
---

<Warning>
  Enterprise Frontier Safeguards 处于测试阶段。它依据您与 Anthropic 之间的协议提供，例如仅限内部使用条款、[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)。在测试期间，设置和行为可能会发生变化。
</Warning>

本页面介绍您的 [Case 存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#glossary)中的 Safety Case 和对话记录文件，以及每种文件的格式。您在 **Safety Case and Transcript storage** 下选择该存储桶。要设置该存储桶，请参阅[设置客户自有存储](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage)。要了解您的团队如何审查和响应 Safety Case，请参阅[审查 Safety Case](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/flagged-cases)。

Safety Case 要求您的团队调查一组消息。其工作方式如下：

1. Anthropic 将 Safety Case 以 JSON 文件形式写入您的存储桶，并为 Anthropic 保有副本的每条消息写入一个对话记录文件。
2. 您的团队进行调查并将响应文件写入同一存储桶。
3. Anthropic 关闭该 Safety Case。

Anthropic 还可以写入有关您的 Safety Case 的 Safety Notice。

部分细节取决于您的存储提供商：

* **Amazon S3：** 本页面中的路径、权限和过期设置适用于 Amazon S3。
* **Google Cloud Storage：** 在 Claude Console 中连接的存储桶中，每个路径都以一个以您的 Anthropic 组织 ID 命名的文件夹开头（请参阅[在工作区中使用您的 Cloud Storage 存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#gcs-use-your-bucket-in-a-workspace)）。

## 存储桶中的内容

| 前缀 | 存放内容 | 谁需要它 | 过期时间 |
| ----- | ----- | ----- | ----- |
| `review-cases/` | 每个 Safety Case 对应一个 [Safety Case 文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#safety-case-file) | 分诊人员和审查人员，用于读取 | 默认 2,555 天（约 7 年） |
| `review-notices/` | Anthropic 以文件形式写入的 [Safety Notice](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#notice)，每条 Safety Notice 一个文件 | 分诊人员和审查人员，用于读取 | 默认 2,555 天 |
| `transcripts/flagged/` | Safety Case 中每条消息的[对话记录](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#transcript) | 审查人员，用于读取 | 最长 30 天 |
| `review-responses/` | 您为每个 Safety Case 编写的[响应文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#response-file) | 审查人员，用于写入 | 默认 2,555 天 |

Anthropic 建议按前缀划分访问权限，这样负责分诊警报的人员就不会看到提示和响应。每个角色需要以下权限：

* **进行读取：** 人员需要对该前缀拥有 `s3:GetObject` 权限、对存储桶拥有 `s3:ListBucket` 权限，并对存储桶的密钥拥有 `kms:Decrypt` 权限。使用 `s3:prefix` 条件将 `s3:ListBucket` 限制在该前缀内。`transcripts/` 的其余部分可能包含不会产生 Safety Case 的请求。
* **写入响应：** 审查人员还需要对 `review-responses/*` 拥有 `s3:PutObject` 权限，并对该密钥拥有 `kms:GenerateDataKey` 权限。

要将 Safety Case、Safety Notice 或响应保留得比默认时间更长，请在部署[设置文件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage#template-reference)时设置 `ReviewExpiryDays`（CloudFormation）或 `review_expiry_days`（Terraform）。当这些文件之一过期、被替换或被删除后，它会作为非当前版本保留最多该天数。

不要直接编辑存储桶的生命周期规则。缺少必需规则或添加了过期规则的存储桶将无法通过 **Test setup** 或之后的检查。对话记录最多在 30 天后过期，因此请将需要更长时间保留的对话记录复制到其他存储中。

## 新文件警报

为每个新的 Safety Case 文件和 Safety Notice 文件设置警报，以便您的团队能够及时响应。

### Amazon S3

有两种方法可以在不更改存储桶的情况下获取警报：

* **按计划列出。** 运行一个定期作业，使用 `ListObjectsV2` 列出 `review-cases/` 和 `review-notices/`。让它将每次的列表与上次运行的列表进行比较，并对每个新对象发出警报。
* **在 AWS CloudTrail 中匹配写入操作。** 为这两个前缀[开启 CloudTrail 数据事件](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/external-storage-manage#cloudtrail)，并在 Amazon EventBridge 规则中匹配 `PutObject` 和 `CompleteMultipartUpload`。

Anthropic 不建议通过更改存储桶自身的通知设置来获取警报。设置文件要求您在部署后不要编辑存储桶的配置，设置 `ReviewExpiryDays` 除外。

Anthropic 可能会覆盖 Safety Case 文件，例如在 Safety Case 关闭时，因此 Amazon EventBridge 规则可能会多次匹配同一文件。请读取文件中的 `status`。

### Google Cloud Storage

在 Google Cloud Storage 存储桶中，`review-cases/` 和 `review-notices/` 位于以您的 Anthropic 组织 ID 命名的文件夹内。使用 JSON API 的 `Objects: list` 方法按计划列出 `<organization ID>/review-cases/` 和 `<organization ID>/review-notices/`，并对每个新对象发出警报。

## Safety Case 文件

当 Anthropic 向您发送新的 Safety Case 时，会将其以下列格式写入 `review-cases/<id>.json`：

```json
{
  "schema_version": 1,
  "id": "rvc_01Hq3Zt8KpWv5NcR2yLbXs7D",
  "status": { "type": "open" },
  "created_at": "2026-11-23T18:42:07Z",
  "updated_at": "2026-11-23T18:42:07Z",
  "workspace_id": "wrkspc_01AbCdEfGhJkLmNpQrStUvWx",
  "policy_categories": ["cyber"],
  "enforcement_action": null,
  "transcripts": [
    {
      "item_id": "msg_01XFDUDYJgAACzvnptvVoYEL",
      "object_key": "transcripts/flagged/4da8b33b-8235-3151-ffb5-1e5cc2230505/MESSAGE_SOURCE_PUBLIC_API_SAMPLING_CALL-msg_01XFDUDYJgAACzvnptvVoYEL.json"
    }
  ],
  "derived_data_object_keys": []
}
```

| 字段 | 存放内容 |
| ----- | ----- |
| `schema_version` | 格式的版本。更高版本可能会添加字段。 |
| `id` | Safety Case 的 ID，以 `rvc_` 开头。它用于命名此文件和您的响应文件，[Safety Cases API](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api) 也使用相同的 ID。 |
| `status.type` | `open` 或 `closed` |
| `status.reason` | 仅出现在已关闭的文件中。如果 Anthropic 未给出原因，则为 `null`。否则为一个对象，其 `type` 说明 Safety Case 关闭的原因。下表列出了这些值。 |
| `created_at` | Anthropic 发送 Safety Case 的时间，采用 UTC。7 个日历日的响应期限从此时间开始计算。 |
| `updated_at` | Anthropic 最后一次更新此文件的时间，采用 UTC |
| `workspace_id` | 发送这些消息的工作区的 ID；如果消息未指明工作区，则为 `null`。该 ID 以 `wrkspc_` 开头。 |
| `policy_categories` | 自动化安全监控发现的风险类型，例如 `cyber` 或 `bio`。Anthropic 可能会添加新值，因此请将您不认识的值路由到默认队列。 |
| `enforcement_action` | 一个对象，其 `type` 指明 Anthropic 在发送 Safety Case 之前采取的措施；或为 `null`。对 `null` 以外的任何值发出警报。 |
| `transcripts[].item_id` | 消息的 ID：Messages API 响应的 `id` |
| `transcripts[].user_profile_id` | 发送该消息的用户配置文件。即使您发送了用户配置文件，Anthropic 也可能省略此键。 |
| `transcripts[].object_key` | 该消息的对话记录在您存储桶中的位置；如果 Anthropic 未保留副本，则为 `null`。该对象也可能不存在。该键的文件夹是工作区的 UUID（而不是其 `wrkspc_` ID）；如果消息未指明工作区，则为 `default`。 |
| `derived_data_object_keys` | Anthropic 添加的有关该 Safety Case 的任何额外文件的对象键。如果没有，则为空列表。 |

`status.reason.type` 具有以下值：

| `status.reason.type` | 含义 |
| ----- | ----- |
| `responded` | Anthropic 已收到您的响应。 |
| `window_ended` | 在 `created_at` 之后 7 个日历日内没有有效响应。 |
| `ended_by_anthropic` | Anthropic 在 7 个日历日结束之前结束了该 Safety Case。 |

Anthropic 可能会添加新值。

不要移动或编辑此文件，并且在其 `status.type` 为 `open` 时不要删除它。Anthropic 会就地更新它，例如在 Safety Case 关闭时。如果 Anthropic 找不到有效的文件，就不会写入已关闭的文件。

## 对话记录

对话记录是一条消息的可读副本：您的提示和 Claude 的响应。Safety Case 文件在 `object_key` 中给出每条对话记录的位置。每条对话记录是一个包含以下字段的 JSON 对象：

| 字段 | 存放内容 |
| ----- | ----- |
| `id` | 消息 ID |
| `request`、`response` | 该消息，采用 Messages API 的格式。`request` 包含您发送的 [`metadata`](https://platform.claude.com/docs/zh-CN/api/messages/create#create.metadata) 参数，其中包含 `user_id`。`response` 可能为 `null`。 |
| `request_time` | Anthropic 记录该消息的时间 |
| `organization_uuid`、`workspace_uuid` | 发送该消息的组织和工作区。`workspace_uuid` 是工作区的 UUID，而不是其 `wrkspc_` ID；如果消息未指明工作区，则为 `null`。 |
| `source` | 消息的来源 |
| `schema_version` | 格式的版本 |

对话记录并不是消息的精确副本。Anthropic 会遮盖 MCP 服务器令牌并移除思考块，图像或文档可能会以占位符形式出现。

## Safety Notice

Safety Notice 是 Anthropic 发送的、关于您的一个或多个 Safety Case 或关于您整个组织的消息。它可以是要求您响应的请求、警告或问题。Anthropic 并不会将每条 Safety Notice 都写成文件，但 Safety Cases API 会列出所有 Safety Notice（请参阅[轮询 Safety Notice](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api#poll-for-notices)）。Safety Notice 文件位于 `review-notices/<id>.json`，Anthropic 永远不会更改它：

```json
{
  "schema_version": 1,
  "id": "rvn_01Jt5Yw2MnQx8VbK4zRcHd6F",
  "case_ids": ["rvc_01Hq3Zt8KpWv5NcR2yLbXs7D"],
  "kind": { "type": "response_requested" },
  "created_at": "2026-11-27T18:00:00Z",
  "message": "Anthropic has found no valid response to the Case in case_ids. Review the Case and write your response file before the time in respond_by. If you wrote a valid response file in the last hour, ignore this notice.",
  "respond_by": "2026-11-30T18:42:07Z"
}
```

| 字段 | 存放内容 |
| ----- | ----- |
| `id` | Safety Notice 的 ID，以 `rvn_` 开头。它用于命名此文件。 |
| `case_ids` | 该 Safety Notice 所涉及的每个 Safety Case 的 `id`。空列表表示针对您的整个组织。 |
| `kind.type` | `response_requested`、`warning` 或 `question` |
| `message` | 请求、警告或问题的文本，最多 2,000 个字符。它从不引用对话记录。 |
| `respond_by` | Anthropic 要求答复的时间，或为 `null`。它不会延长响应 Safety Case 的 7 个日历日期限。 |

要答复 `response_requested`，请编写您的响应文件。要回答问题，请联系您的 Anthropic 客户团队。

## 响应文件

响应文件告知 Anthropic 您的团队已调查了某个 Safety Case，以及调查发现了什么。这是您在不使用 [Safety Cases API](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api) 的情况下响应 Safety Case 的方式。将其写入 `review-responses/<case_id>.json`，在 Google Cloud Storage 中则写入 `<organization ID>/review-responses/<case_id>.json`：

```json
{
  "schema_version": 1,
  "case_id": "rvc_01Hq3Zt8KpWv5NcR2yLbXs7D",
  "submitted_at": "2026-11-25T16:02:10Z",
  "determination": "violation",
  "policy_categories": ["cyber"],
  "verdicts": [
    {
      "item_id": "msg_01XFDUDYJgAACzvnptvVoYEL",
      "determination": "violation",
      "policy_category": "cyber"
    }
  ]
}
```

| 字段 | 发送内容 |
| ----- | ----- |
| `schema_version` | 您响应的架构版本。唯一接受的值是数字 `1`。 |
| `case_id` | Safety Case 的 `id`，与文件名中的相同 |
| `submitted_at` | 您响应的时间，采用带偏移量的 ISO 8601 时间格式，例如 `Z` |
| `determination` | `violation` 或 `no_violation`，针对整个 Safety Case |
| `policy_categories` | 使用 `violation` 时，填写一到四个您确认的类别。使用该 Safety Case 自身 `policy_categories` 中的值；如果都不适用，则使用 `other`。使用 `no_violation` 时，省略此字段或发送空列表。 |
| `verdicts` | 可选，且绝不能为 `null`。要分别判定消息，请提供一个列表，为您判定的每条消息提供一个条目。每个条目包含一个 `item_id`、一个 `determination`，以及在 `violation` 时可选的 `policy_category`。该列表不必涵盖所有消息。 |

### 有效文件的规则

Anthropic 不会告知您文件无效，因此请根据以下规则检查您的文件：

* 它位于 `review-responses/<case_id>.json`（在 Google Cloud Storage 中为 `<organization ID>/review-responses/<case_id>.json`），其中的 ID 与其 `case_id` 字段相同。请使用 Safety Case 的 `id`，它以 `rvc_` 开头。
* 它包含 `schema_version`、`case_id`、`submitted_at` 和 `determination` 字段，且不包含表中没有的字段。
* 使用 `violation` 时，`policy_categories` 有一到四个值；使用 `no_violation` 时没有值，且任何值都不重复出现。`verdicts` 中没有重复出现的 `item_id`。
* 每个类别都是 Safety Case 可以包含的值，例如 `cyber` 或 `other`。
* `verdicts` 中的每个 `item_id` 都在该 Safety Case 中。
* 它是最大 256 kB 的纯 JSON，不含字节顺序标记，且没有重复的键。Windows PowerShell 5.1 会添加字节顺序标记，这会使文件无效，因此 Anthropic 会将其视为没有响应。
* 它使用您存储桶的默认加密。

Anthropic 会采纳它读取到的第一个有效响应，之后您将无法更改。在此之前，您可以替换该文件。

在更改或移除某个工作区的[安全数据存储桶](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/faq#glossary)或 Case 存储桶之前，请先响应其开启的 Safety Case，并等待它们关闭。如果您先更改或移除存储桶，Anthropic 可能无法读取您的响应文件。

### Safety Case 为何保持开启状态

Anthropic 会按计划检查 `review-responses/`，因此请预留一天时间。您在 7 个日历日内写入的有效文件都会被计入，即使 Anthropic 之后才读取它。如果在您写入文件一天后 `status.type` 仍为 `open`，请根据[有效文件的规则](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-case-files#response-file-rules)检查您的文件并替换它。

当 Anthropic 读取到有效响应时，会重写 Safety Case 文件，将 `status.type` 设置为 `closed`。请读取该文件中的 `status.reason`：

* **`status.reason.type` 为 `responded`：** 这表示 Anthropic 已收到您的响应。
* **`status.reason` 为 `null`：** 即使 Anthropic 已收到您的响应，也可能出现这种情况。在 Claude Platform 上，请通过 [Safety Cases API](https://platform.claude.com/docs/zh-CN/manage-claude/enterprise-frontier-safeguards/safety-cases-api) 读取该 Safety Case，API 始终会给出原因。

已关闭的 Safety Case 不会重新开启，Anthropic 会忽略其关闭后写入的响应文件。
