---
title: 将 Security posture 事件发送到您的 SIEM
url: https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events
description: 从 Compliance API 的活动源请求 Security posture 活动，逐个字段读取其中一条活动，并通过能捕获延迟活动的定时读取让您的 SIEM 保持最新。
---

<Note>
  要启用 Compliance API，请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)。
</Note>

本页介绍如何从一个 Compliance Access Key 开始，让 Security posture 活动按计划作为事件到达您的安全信息和事件管理（security information and event management，SIEM）系统。概述中说明了[哪些组织拥有这些活动](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#availability)。您需要一个覆盖该组织且具有 `read:compliance_activities` 或 `read:org_audit` 作用域的 Compliance Access Key。

## 请求活动

若只读取这些活动，请使用 `activity_types[]` 筛选活动源。在此请求中，`created_at.gte` 设置要读取的最早 `created_at`，`order=asc` 则先返回最早的活动：

```bash cURL
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/activities" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21" \
  --data-urlencode "activity_types[]=org_security_control_attestation_updated" \
  --data-urlencode "created_at.gte=2026-10-20T00:00:00Z" \
  --data-urlencode "order=asc" \
  --data-urlencode "limit=5000"
```

[API 参考](https://platform.claude.com/docs/zh-CN/api/compliance/activities/list)介绍了每个参数。

## 读取一条活动

响应的 `data` 数组中的每个元素都是一条类似下面这样的活动。

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

* `control` 指明[安全控制项](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#controls)，即一项推荐的设置或做法。
* `action` 说明该[控制项的声明](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#attestations)（某人针对该控制项记录下的陈述）发生了什么。此处 `attested` 表示为一个此前没有声明的控制项记录了一条声明。
* `status` 是该声明的值。此处为“已接受风险”（`accepted_risk`）：该控制项未落实，且组织接受该风险。
* `actor` 此处通过用户 ID 和电子邮件地址标识[作出声明的人](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#who-can-see-who-attested)。
* `created_at` 此处是其作出声明的时间。

[Security posture 事件参考](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference)列出了更多字段和值。

## 如果响应为空

空的 `data` 数组是有效的。活动源只包含自其[开始为该组织记录这些活动](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference#what-the-feed-does-not-record)以来所做的变更。在有人作出或更改声明之前，它是空的。

请检查该组织是否[拥有这些活动](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#availability)。然后请某人对一个控制项作出声明。过一段时间后，再重复该请求。

## 按计划运行

这些活动中的任何一条都可能比活动源通常 1 分钟的[索引延迟](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-integration-patterns#window-polling)更晚才可查询，且其 `created_at` 已经是过去的时间。因此，只向前推进（通过游标或不断前移的 `created_at.gte`）的集成永远看不到这样的活动。请在每次运行时重新读取固定的 4 小时回溯窗口，并丢弃您已存储的活动：

```text
query = activity_types[]=org_security_control_attestation_updated
      & created_at.gte={now - 4 hours}
      & order=asc
      & limit=5000
cursor = null
loop:
  if cursor is null:
    page = GET /v1/compliance/activities?{query}
  else:
    page = GET /v1/compliance/activities?{query}&after_id={cursor}
  for activity in page.data:
    if activity.id is not in stored_ids:
      store(activity)
      add activity.id to stored_ids
  if not page.has_more: break
  cursor = page.last_id
```

* **频繁运行：** 按照告警所需的频率运行该循环，例如每 5 分钟一次。
* **整个运行期间保持同一查询：** 在每次运行开始时计算 `created_at.gte`，使用 UTC 并进行 URL 编码。在每一页上发送相同的筛选条件和 `order`。使用 `order=asc` 时，将 `last_id` 作为 `after_id` 传入会返回下一页较新的活动。[分页浏览结果](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed#paginate-results)解释了 `after_id` 和 `last_id`。
* **按 `id` 去重：** 此步骤是必需的，因为连续的运行会相互重叠。
* **处理失败的运行：** 如果某个请求在经过[处理 Compliance API 错误](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors)中的重试后仍然失败，请结束本次运行。下一次运行会重新读取其整个窗口。如果运行失败或停止超过 4 小时，下一次[每日重新读取](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events#re-read-15-days-once-a-day)会读取它们漏掉的活动。

## 每天重新读取 15 天

每天一次，以 15 天的回溯窗口代替 4 小时来运行同一个循环。这一每日重新读取还会读到在其 `created_at` 之后最多 14 天才可查询的活动。

* 记录每次已完成的每日重新读取的开始时间。
* 仅当某个已存储的 `id` 的存储时间早于最近一次记录的开始时间超过 16 天时，才删除它。
* 如果没有记录任何开始时间，请将 `created_at.gte` 设置为您需要回溯到的最早时间；活动源会将活动保留 6 年。
* 如果距最近一次记录的开始时间已超过一天，请在下一次每日重新读取中将 `created_at.gte` 设置为该开始时间之前 15 天。

## 如果您已在摄取完整的活动源

读取整个活动源且没有使用会排除这些活动的 `activity_types[]` 筛选条件的集成，已经会收到这些活动。请检查两件事：

* **无法识别的值：** 您的处理程序会原样传递任何它无法识别的 `type` 或[字段值](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference)。请参阅[了解 Activity 对象](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed#understand-the-activity-object)。
* **延迟活动：** 只请求较新的活动的集成会漏掉延迟才可查询的活动。请为该集成添加 [4 小时重新读取](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events#run-it-on-a-schedule)和 [每日 15 天重新读取](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events#re-read-15-days-once-a-day)。

## 检查是否正常工作

在有人对某个控制项作出声明后，运行该循环，直到它存储了这条新活动。然后再运行一次：它不会再次存储该活动。

## 后续步骤

若要根据您存储活动的时间（而非其 `created_at`）发出告警，请参阅[检测 Security posture 中的变更](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/detect-changes#before-you-write-a-rule)。有关开放网络安全架构框架（Open Cybersecurity Schema Framework，OCSF），请参阅[将 Security posture 事件映射到 OCSF](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/ocsf-mapping)。有关当前结果，请参阅[Security posture 结果参考](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference)。
