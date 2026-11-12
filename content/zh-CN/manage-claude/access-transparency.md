---
title: 访问透明度
url: https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency
description: 通过 Compliance API 接收 Anthropic 人员对您组织数据进行人工访问的审计记录。
---

了解 Access Transparency（访问透明度）如何为 Anthropic 人员对您组织数据的人工访问创建记录、它涵盖哪些内容，以及如何通过 Compliance API 接收事件。

<Note>
  当您的组织启用访问透明度后：

  * Anthropic 员工每次人工查看您的留存数据（请参阅[涵盖的内容](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#what-access-transparency-covers)）时，都会向您的 [Compliance API 活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)写入一条 `anthropic_access` 活动。
  * 访问仅出于安全审查或事件响应的目的而发生。请参阅[原因代码](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#reason-codes)。

  访问透明度可应符合条件的客户请求提供，并非自助开通。有关资格要求，请参阅您的合同条款或联系您的 Anthropic 客户代表。
</Note>

## 访问透明度的工作原理

Anthropic 人员仅在明确规定的条件下访问客户内容。访问透明度旨在让此类访问对您可见。其设计基于以下原则：

* **人工访问仅在已公布的原因代码下进行。**
* **对您涵盖内容的人工查看会被记录。** Anthropic 能够触及您涵盖内容的内部工具均已进行埋点，每次查看都会发出一个事件。
* **事件代表人工访问，而非自动化处理。** Anthropic 的自动化安全系统在一个安全的管道中处理您的内容，不存在交互式人工访问；该处理不会生成 `anthropic_access` 事件。自动化处理唯一可能触发的事件是 `cmek_preserve` 保全记录（请参阅 [CMEK 内容保全](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#cmek-content-preservation)）。
* **事件会出现在您现有的活动源中。** 活动可通过您的 [Compliance API 活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)访问。Compliance API 现有的凭证、审计、导出和 SIEM 集成仍然适用。
* **对事件的任何篡改都能被检测到。** 在您组织的[透明度日志](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log)（beta）创建之后记录的每个事件，也会提交到该日志中。该日志是一份仅追加、经过签名的记录，您可以独立于 Anthropic 的服务系统对其进行验证。

## 访问透明度涵盖的内容

* **覆盖的内容：** 访问透明度覆盖通过 Claude Messages API 或 Claude Code 会话发送的提示和响应内容。[API 和数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)页面说明了哪些 API 和功能受 ZDR 覆盖。访问透明度覆盖相同的 API 和功能。
* **Anthropic 人员的人工查看：** Anthropic 审查人员对您涵盖内容的人工查看会生成事件。

## 访问透明度不涵盖的内容

* **自动化处理：** 模型服务、安全分类器和滥用检测管道在正常运行过程中处理您的内容，不会生成 `anthropic_access` 事件。由自动化处理触发的保全确实会生成 `cmek_preserve` 事件（请参阅 [CMEK 内容保全](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#cmek-content-preservation)）。
* **您自己组织的活动：** 您的 API 调用、管理员操作和 Compliance API 读取由标准的[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)事件类型涵盖。
* **Claude for Enterprise 和 Claude 应用：** claude.ai Enterprise 席位、Claude for Work、Cowork 和 Claude in Chrome 不在涵盖范围内。
* **Claude 消费者产品：** Claude Free、Pro、Max 或 Ultra 计划。
* **合作伙伴运营的平台：** Claude Models on Google Cloud (Google managed) 和 Amazon Bedrock。请参阅这些平台的透明度控制措施。
* **ZDR 不覆盖的任何内容：** 不受 ZDR 覆盖的产品（例如 Files API、Anthropic 托管的有状态应用程序以及 Batch API）也不受访问透明度覆盖。有关更多详细信息，请参阅[零数据保留 (ZDR)](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#zero-data-retention-zdr-scope)。

## 快速开始

要启用访问透明度：

<Steps>
  <Step title="申请访问透明度">
    联系您的 Anthropic 客户代表。
  </Step>

  <Step title="Anthropic 审核资格">
    Anthropic 确认您的组织符合资格标准，并在组织级别启用该功能。
  </Step>

  <Step title="通过 Compliance API 接收事件">
    `anthropic_access` 活动会出现在您现有的活动源中，使用您现有的 Compliance Access Key；无需新的端点或凭证。
  </Step>
</Steps>

访问透明度在组织级别启用，涵盖所有工作区。目前不支持按工作区单独注册。

## 接收访问透明度事件

访问透明度事件以 `anthropic_access` 活动类型在 Compliance API 活动源上交付。使用 `activity_types[]` 进行筛选：

```bash
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/activities" \
  --data-urlencode "activity_types[]=anthropic_access" \
  --data-urlencode "limit=50" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

分页、日期范围筛选（`created_at.gte` / `.lt`）以及响应封装（`has_more`、`first_id`、`last_id`）与活动源的其余部分共用。请参阅[查询活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)。

关闭 Compliance API 会停止您对访问透明度事件的访问，但不会停止对这些事件的记录。重新开启 Compliance API 后，活动源会返回这些事件。

每条 `anthropic_access` 活动都包含标准的 Activity 字段以及以下字段：

| 字段                            | 类型            | 描述                                                                                                                                                                            |
| ----------------------------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                          | string        | 此活动的唯一标识符                                                                                                                                                                     |
| `accessed_at`                 | RFC 3339 字符串  | 访问发生的时间。可能早于该活动在您的活动源中可见的时间                                                                                                                                                   |
| `created_at`                  | RFC 3339 字符串  | Anthropic 记录该事件的时间。事件通常在此之后不久即可在您的活动源中看到；有关可能出现延迟的情况，请参阅[时间安排](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#timing)                           |
| `actor`                       | object        | 始终为 `{ "type": "anthropic_actor", "email_address": null }`。不会披露员工个人身份                                                                                                         |
| `accessor_department`         | string        | 执行访问的 Anthropic 团队（例如 `Safeguards`）                                                                                                                                           |
| `reason_code`                 | enum          | 请参阅[原因代码](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#reason-codes)                                                                              |
| `resource_details.type`       | enum          | 资源类型，目前仅有 `message`。可扩展以支持未来的资源类型                                                                                                                                             |
| `resource_details.id`         | string 或 null | 被访问内容的标识符                                                                                                                                                                     |
| `resource_details.parent`     | string 或 null | 内容父级的标识符，例如包含某条消息的对话 ID。在支持具有父级的资源之前，目前为 `null` 或省略                                                                                                                           |
| `organization_id`             | string        | 内容所属的组织。带标签的 ID 格式（`org_...`）                                                                                                                                                 |
| `organization_uuid`           | string        | 内容所属的组织。UUID 格式                                                                                                                                                               |
| `workspace_id`                | string 或 null | 内容所属的工作区                                                                                                                                                                      |
| `workspace_uuid`              | string        | 内容所属的工作区。UUID 格式。当访问限定于某个工作区时存在，否则不存在。它不属于透明度日志的[叶子字段](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#how-an-event-becomes-a-leaf)，因此包含证明不涵盖该字段 |
| `transparency_log_leaf_index` | integer       | 该事件在您组织的透明度日志（beta）中从零开始的位置。只要事件有叶子就会存在，否则不存在。请参阅[使用透明度日志验证访问透明度事件](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log)                             |
| `user_profile_id`             | string        | 被访问或保全的内容在其下创建的[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)。带标签的 ID 格式（`uprof_...`）。仅当内容是在某个用户配置文件下创建时存在                                      |

JSON 消息示例：

```json
{
  "id": "activity_013b013744txqZtFHLUaRqLr",
  "type": "anthropic_access",
  "created_at": "2026-06-08T17:12:09.812446Z",
  "accessed_at": "2026-06-08T17:12:06.478035Z",
  "organization_id": "org_0910d9133038914eta7i3vt",
  "actor": { "type": "anthropic_actor", "email_address": null },
  "resource_details": { "type": "message", "id": "msg_1234ABCD" },
  "accessor_department": "Safeguards",
  "reason_code": "safety_review",
  "organization_uuid": "5b236db4-3fb4-4bf3-a560-b5e266038a15",
  "transparency_log_leaf_index": 41
}
```

## CMEK 内容保全

在极少数情况下，Anthropic 会将特定内容保留超过标准留存期限（例如，当安全审查确认存在严重有害内容且必须为正在进行的调查而保留时）。保全本身是一项有记录且对客户可见的操作：

* **保全事件会写入您的活动源。** 当内容被保全时，一个类型为 `cmek_preserve` 的事件会写入您的 Compliance API 活动源。保全事件携带与 `anthropic_access` 事件相同的字段；仅事件类型不同，因此能处理其中一种的解析器也能处理另一种。请参阅[原因代码](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#reason-codes)。
* **无论保全是如何触发的，都会写入保全事件。** 保全通常发生在对内容进行人工审查之后，但无论保全是由人工审查人员还是由自动化安全管道触发，都会写入该事件：该记录反映的是您内容的留存状态发生了变化，与由谁更改无关。
* **对于 CMEK 组织，保全是一次可见的密钥迁移。** 被保全的内容会在您的客户管理密钥之外重新加密，以便调查可以独立于您的密钥继续进行。保全事件就是此事发生的记录。所有其他留存内容仍受您的密钥保护。

筛选保全事件的方式与访问事件相同：

```bash
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/activities" \
  --data-urlencode "activity_types[]=cmek_preserve" \
  --data-urlencode "limit=50" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

JSON 消息示例：

```json
{
  "id": "activity_01AbCdEfGhJkMnPqRsTuVwXy",
  "type": "cmek_preserve",
  "created_at": "2026-07-02T09:41:53.204118Z",
  "accessed_at": "2026-07-02T09:41:50.118764Z",
  "organization_id": "org_0123456789abcdefghijklmn",
  "actor": { "type": "anthropic_actor", "email_address": null },
  "resource_details": { "type": "message", "id": "msg_0ExampleExampleExample" },
  "accessor_department": "Safeguards",
  "reason_code": "policy_violation_investigation",
  "organization_uuid": "00000000-1111-2222-3333-444444444444",
  "transparency_log_leaf_index": 57
}
```

对于保全事件，`accessed_at` 记录的是内容被保全的时间。

## 原因代码

原因代码集合是封闭的。如果 Anthropic 引入新代码，将会更新本页面。

| 代码                               | 含义                     |
| -------------------------------- | ---------------------- |
| `safety_review`                  | 内容作为使用政策或安全调查的一部分被查看   |
| `incident_response`              | 内容在调查影响您组织的事件时被查看      |
| `policy_violation_investigation` | 内容在信任与安全政策违规调查期间被保全    |
| `csae_report`                    | 内容作为儿童安全（CSAE）报告的证据被保全 |

## 产品界面资格

下表列出了哪些产品界面受访问透明度涵盖。涵盖意味着对来自该界面的内容进行人工访问会生成 `anthropic_access` 事件。

| 产品界面                                                          | 是否涵盖 | 详情                                                                   |
| ------------------------------------------------------------- | ---- | -------------------------------------------------------------------- |
| Claude API（`api.anthropic.com`）                               | 是    | 提示、补全以及直接嵌入 API 输入中的数据                                               |
| Claude Code（使用 API 密钥）                                        | 是    | 来自 Claude Code 的 API 流量作为 Claude API 流量被涵盖                           |
| Claude Platform on AWS                                        | 是    | Claude Platform on AWS 在 Compliance API（而非 AWS CloudTrail）中生成访问透明度事件 |
| Claude API（`api.anthropic.com`）（Batch、Files）                  | 否    | Claude API 的 Batch 和 Files API 不在涵盖范围内，正如它们不受 ZDR 涵盖一样               |
| Claude for Enterprise（claude.ai 席位）                           | 否    | 不涵盖                                                                  |
| Claude for Work                                               | 否    | 不涵盖                                                                  |
| Claude Free、Pro、Max、Ultra                                     | 否    | 消费者计划不符合资格                                                           |
| Playground（Claude Console）                                    | 否    | 不涵盖                                                                  |
| Microsoft Foundry                                             | 否    | 不可用                                                                  |
| Amazon Bedrock、Claude Models on Google Cloud (Google managed) | 否    | 合作伙伴运营的平台；请参阅这些平台的透明度控制                                              |

## 限制与排除

### 涵盖时间

访问透明度自为您的组织启用之时起生效。启用时已处于保留期限内的内容在被访问时也可能生成事件，但 Anthropic 不保证覆盖启用前写入的内容。请将您的启用日期视为可靠覆盖的开始时间。从启用访问透明度到您的内容被覆盖，可能会有最长 2 小时的延迟。

### 通知时间

`anthropic_access` 和 `cmek_preserve` 事件会在其所记录的访问或保全发生后 2 个工作日内交付到您的 Compliance API 活动源。请勿将此活动源视为实时告警渠道。`accessed_at` 时间戳反映访问发生的时间，该时间可能比活动在您的活动源中可见的时间早最多 2 个工作日。`created_at` 字段反映 Anthropic 记录该事件的时间，事件通常在此时间之后不久即可在您的活动源中看到。这些事件不遵循活动源通常的 1 分钟[索引延迟](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-integration-patterns#window-polling)：事件可能在其 `created_at` 之后最多 2 个工作日才可见。如果您按 `created_at` 时间窗口轮询活动源，请针对 `anthropic_access` 和 `cmek_preserve` 事件让相邻窗口至少重叠 2 个工作日，以免遗漏延迟索引的事件。

### 自动化处理不会生成访问事件

`anthropic_access` 事件仅记录人工访问。Anthropic 的自动化安全系统和分类器在正常运行过程中会继续处理您的内容，该处理不会生成 `anthropic_access` 事件。自动化处理唯一可能触发的事件是 `cmek_preserve` 保全记录（请参阅 [CMEK 内容保全](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#cmek-content-preservation)）。活动源为空意味着 Anthropic 没有任何人查看过您的内容；这并不意味着您的内容未被自动化系统处理。

### 访问透明度不会改变 Anthropic 可以访问的内容

访问透明度记录访问；它不授予也不限制访问。Anthropic 人员可以访问您内容的目的受您与 Anthropic 的协议以及[使用政策](https://www.anthropic.com/legal/aup)约束，无论是否启用访问透明度，这些目的都相同。

### CMEK 密钥使用日志并非逐次读取记录

对于同时启用 CMEK 的组织，您的云 KMS 审计日志（CloudTrail、Cloud Audit Logs 或 Azure Monitor）会记录 Anthropic 对您密钥的使用。由于密钥在运行期间会被短时缓存，单次人工读取不一定会产生一条独立的 KMS 解密条目。请将访问透明度活动源用作逐次访问记录；您的 KMS 日志则独立确认密钥使用模式。

## 常见问题

<AccordionGroup>
  <Accordion title="我如何知道我的组织是否已启用访问透明度？">
    联系您的 Anthropic 客户代表。
  </Accordion>

  <Accordion title="每次安全分类器在我的流量上运行时，我都会看到一个事件吗？">
    不会。自动化处理不会生成 `anthropic_access` 事件；只有当人工审查人员随后查看该内容时，您才会看到 `anthropic_access` 事件。另外，当内容被保全时会写入 `cmek_preserve` 事件，无论保全是由人工审查人员还是自动化安全管道触发。
  </Accordion>

  <Accordion title="向其自身最终用户提供 Claude 服务的平台可以启用访问透明度吗？">
    访问透明度不适用于平台部署。请联系您的 Anthropic 客户代表讨论您的用例。
  </Accordion>

  <Accordion title="对于我的组织注册之前发生的访问，或其较早的数据，我会看到事件吗？">
    访问透明度不保证具有追溯性。它涵盖对您注册日期当天或之后写入 Claude API 的内容的人工访问。您可能会看到针对注册之前写入内容的访问事件。
  </Accordion>

  <Accordion title="访问发生后多久我能看到事件？">
    在访问发生后的 2 个工作日内。请为任何 SIEM 告警或定时导出配置相应的回溯窗口，而不要假设事件会实时到达。
  </Accordion>

  <Accordion title="我如何知道 anthropic_access 事件指的是哪个请求？">
    使用 `resource_details.id` 字段。它包含与 [Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create) 在每个响应体的 `id` 字段中返回的相同消息 ID（`msg_...`）。为使其发挥作用，请在您自己的系统中记录 `id`，并附上您的内部元数据，例如产生该请求的应用程序、终端用户或对话。当事件到达时，将其 `resource_details.id` 与您的日志进行关联，即可准确识别被查看的是哪个请求。
  </Accordion>

  <Accordion title="我可以为单个工作区启用访问透明度吗？">
    访问透明度在组织级别启用，涵盖所有工作区。
  </Accordion>

  <Accordion title="访问透明度与 CMEK 有何关系？">
    两者相互独立。使用 CMEK 时，在您的密钥之外进行的安全保全会在同一活动源上发出一个单独的 `cmek_preserve` 事件。请参阅 [CMEK 内容保全](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#cmek-content-preservation)和[您的密钥之外的有限保存](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#limited-preservation-outside-your-key)。
  </Accordion>

  <Accordion title="如何确认我的访问透明度记录未被篡改？">
    验证您组织的透明度日志（beta）：这是一份仅追加、经过签名的访问透明度事件记录，附带您可以在自己的基础设施上检查的包含证明和一致性证明。请参阅[使用透明度日志验证访问透明度事件](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log)。
  </Accordion>

  <Accordion title="我如何申请访问透明度？">
    联系您的 Anthropic 客户代表。
  </Accordion>
</AccordionGroup>

## 相关资源

* [使用透明度日志验证访问透明度事件（beta）](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log)
* [Compliance API 概述](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api)
* [活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)
* [API 与数据留存](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)
* [客户管理的加密密钥 (CMEK)](https://platform.claude.com/docs/zh-CN/manage-claude/cmek)
* [Claude Code 数据使用](https://code.claude.com/docs/zh-CN/data-usage)
* [信任中心](https://trust.anthropic.com/resources)
