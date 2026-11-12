---
title: Security posture 事件参考
url: https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference
description: 查找活动源中记录 Security posture 页面变更的活动：它们的字段和值、每项活动的一个示例，以及活动源不记录的内容。
---

<Note>
  要启用 Compliance API，请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)。
</Note>

[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)将 [Security posture](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview) 页面上对声明的变更记录为本页所描述的活动。要将它们导入安全信息和事件管理（security information and event management，SIEM）系统，请从[将 Security posture 事件发送到您的 SIEM](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events) 开始。对于 `control`、`action` 或其他字段中本页未列出的任何值，请原样传递；忽略意外的字段，并将缺失的键与 `null` 值同等对待。[了解 Activity 对象](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed#understand-the-activity-object)介绍了标准字段和未知类型。

您需要一个覆盖该组织的 Compliance Access Key，并具有 `read:compliance_activities` 或 `read:org_audit` 作用域。

## 声明活动

`org_security_control_attestation_updated` 活动记录某人为某个安全控制项记录、更改、重新保存或撤销了组织的声明，或该声明自动终止。其 `created_at` 是个人所做的变更发生的时间；对于自动变更，则是记录该变更的时间，可能晚于变更本身。对于个人所做的变更，其 `actor` 是 `user_actor`；对于自动变更（例如 `lapsed`），则是 `system_actor`。

### 字段

此类型新增以下字段，其中 `status` 是声明的值，`review_date` 是其结束日期。

| 字段                | 类型     | 描述                                                                                                                                         |
| ----------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `control`         | string | 声明所适用的安全控制项的标识符，例如 `oauth_conditional_access`；请将其视为不透明字符串。                                                                                 |
| `action`          | string | 发生了什么变更，为 [`action` 值](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference#action-values)之一。 |
| `status`          | string | 声明的值：“已完成”（`completed`）、“已接受风险”（`accepted_risk`）或声明为“未通过”（`control_failed`）；撤销声明后不存在。                                                      |
| `previous_status` | string | 此次变更之前声明的值，为上述三者之一；当 `action` 为 `attested` 时不存在。                                                                                           |
| `review_date`     | string | 声明预定终止的 UTC 日期（ISO 8601 日期）；如果保存时未设置结束日期，或声明已被撤销，则不存在。                                                                                     |

对于仅供参考控制项，值“已审阅”记录为 `completed`。

### `action` 值

| `action`   | 含义                                                                           |
| ---------- | ---------------------------------------------------------------------------- |
| `attested` | 为一个此前没有声明的控制项记录了声明。                                                          |
| `changed`  | 声明以另一个值重新保存；或者在 `status` 等于 `previous_status` 的情况下，以相同的值重新保存，与 `renewed` 相同。 |
| `renewed`  | 声明以相同的值重新保存。                                                                 |
| `cleared`  | 某人撤销了声明。                                                                     |
| `lapsed`   | 声明自动终止，而非由某人操作。                                                              |

在 `renewed` 和 `lapsed` 活动中，`status` 等于 `previous_status`。在 `lapsed` 活动之后以相同的值保存，会记录为 `renewed`，而不是 `attested`。

### `lapsed` 活动的含义

`lapsed` 活动表示检查结果或控制项的当前值与保存声明时所依据的不同；即使没有更改任何设置，当前值也可能发生变化，例如 Anthropic 改写其措辞时。`status` 是已终止的值：可以是三者中的任一个。手动控制项的声明永远不会产生 `lapsed` 活动。该活动可能晚于导致它的变更出现。

### 示例

```json
{
  "id": "activity_01Hq7ZtYc4Vb2NmKx8RsPd3W",
  "type": "org_security_control_attestation_updated",
  "created_at": "2026-10-20T15:04:00.220114Z",
  "organization_id": "org_01JuY4bkwkH1x4PodBJaWmh2",
  "organization_uuid": "91012d09-e48b-438e-a489-1bebfd8fa6f9",
  "actor": {
    "type": "user_actor",
    "email_address": "jane.doe@example.com",
    "user_id": "user_01WxYzAbCdEfGhJkMnPqRsTu",
    "ip_address": "192.0.2.34",
    "user_agent": "Mozilla/5.0..."
  },
  "control": "directory_sync_enabled",
  "action": "attested",
  "status": "accepted_risk"
}
```

## 活动源不记录的内容

不要将没有活动视为没有任何变化的迹象。有关当前结果，请参阅 [Security posture 结果参考](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference)；其端点可能会省略活动中指明的控制项。

* 在[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)开始为该组织记录这些活动之前，或在 Compliance API 关闭期间发生的变更，之后也不会补记。
* 极少数情况下，任何类型的变更。
* 每一个到来的结束日期：并非所有结束日期都会产生活动，因此不要等待活动。
* 每次关闭或开启 Security posture 页面：不要依赖[检测规则](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/detect-changes)来检测它。
* 与声明一起保存的备注，或任何表明存在备注的标记。更改备注需要重新保存，活动源会像记录任何其他以相同的值进行的保存一样记录此次保存（请参阅 [`action` 值](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference#action-values)）。
* 声明所针对的控制项类型：自动、手动或仅供参考。
* 每一个可能终止声明的变更：并非所有变更都会产生 `lapsed` 活动。
