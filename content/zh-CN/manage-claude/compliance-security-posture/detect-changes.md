---
title: 检测 Security posture 中的变更
url: https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/detect-changes
description: 从 SIEM 中针对 Security posture 活动的示例检测规则入手：已接受风险、被声明为“未通过”的控制项、意外的编辑者、突发的大量变更，以及自动终止的声明。
---

<Note>
  要启用 Compliance API，请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)。
</Note>

这些示例规则会在您组织的 [Security posture](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview) 发生变更时发出告警，所用的是 Compliance API 活动源记录的活动。这些规则假定您的安全信息和事件管理（security information and event management，SIEM）系统会按照[将事件发送到您的 SIEM](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events) 中的说明存储这些活动。

## 编写规则之前

每条规则都由一段简明描述和一个不限定特定查询语言的查询组成。

* 查询使用活动源的字段名称，外加 `stored_at`，这是本页为您存储某条活动的时间所起的名称。其他任何名称都是您自己的列表或别名。
* 在规则对活动进行计数之前，先按 `id` 对这些活动去重。
* 除非规则另有说明，否则请基于 `stored_at`（而非 `created_at`），在自该规则上次完成运行以来的时间段上运行该规则，因为活动可能会延迟到达；查询中省略了该时间范围。
* 编写的规则应在您已存储的活动上触发，切勿依赖您尚未存储的活动：它可能延迟到达，也可能是[活动源不记录](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference#what-the-feed-does-not-record)的变更。 如需将已存储的活动与当前结果进行比较，请参阅[将事件与结果进行核对](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/check-events-against-results)。

## 声明规则

以下五条规则读取 [`org_security_control_attestation_updated`](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference#attestation-activity) 活动。

### 某项风险被接受

当有人接受未满足某项安全控制项的风险（`accepted_risk`）时发出告警，以便由第二个人审查该决定。

```sql
type == "org_security_control_attestation_updated"
AND status == "accepted_risk"
AND actor.type != "system_actor"
```

该规则排除了 `system_actor`，因为自动变更（例如已终止的声明，即 `lapsed`）会保留 `status`，但并不属于接受风险。

### 某个控制项被声明为“未通过”

当有人将某个控制项声明为“未通过”（`control_failed`）时发出告警，以便有人跟进该控制项。

```sql
type == "org_security_control_attestation_updated"
AND status == "control_failed"
AND actor.type != "system_actor"
```

该规则排除 `system_actor` 的原因与[某项风险被接受](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/detect-changes#a-risk-was-accepted)相同。

### 允许列表之外的人更改了声明

当不在您的声明管理人员列表中的人记录、更改、再次保存或撤销声明时发出告警。请将这些人员的 `user_id` 值（`allowed_user_ids`）保存在您的 SIEM 中。

```sql
type == "org_security_control_attestation_updated"
AND actor.type != "system_actor"
AND NOT (actor.type == "user_actor" AND actor.user_id IN allowed_user_ids)
```

* 除列表中的 `user_actor` 或 `system_actor` 以外，任何其他操作者都会触发该规则。
* 请以 `actor.user_id` 作为列表的唯一标识：`actor.email_address` 是记录该活动时此人所用的地址（[与您的 SIEM 关联](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-integration-patterns#correlate-with-your-siem)）。

### 一个人进行了大量声明变更

当某个人在短时间内进行的声明变更多于您组织的常态时发出告警。阈值 10 和 1 小时的时间窗口仅为示例；请根据您自己的活动来选择这两个值。

```sql
type == "org_security_control_attestation_updated"
AND actor.type == "user_actor"
| group by organization_uuid, actor.user_id over a sliding 1 hour of created_at
| where count(distinct id) >= 10
```

* 仅对包含自该规则上次运行以来所存储活动的分组发出告警，并对您已存储的每条活动进行计数，这样延迟到达的活动也能并入一次突发，即使该突发中的其他活动是您更早存储的。
* 按 `created_at` 计数衡量的是该人员的操作节奏，而不是您的摄取节奏。

### 声明自动终止

将 `lapsed` 活动视为一个提示，用于审查其所指明的控制项。

```sql
type == "org_security_control_attestation_updated"
AND action == "lapsed"
```

[`lapsed` 活动](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference#what-a-lapsed-activity-means)表示该控制项的检查结果或当前值与保存该声明时所依据的值不同，而不是结束日期（`review_date`）已过。它可能会延迟到达。

## 在取值变化时保持规则有效

这些字段中的任何一个都可能出现新的值。

* 通过 `actor.type` 而不是 `actor.service` 的值来识别自动变更。
* 不要丢弃其 `control`、`action` 或 `status` 值未在规则中列出的活动；新的 `action` 值可能记录了一次接受风险。请参阅[事件参考](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference)。

## 测试规则

如需查看[允许列表规则](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/detect-changes#someone-outside-an-allowlist-changed-an-attestation)触发，请选择一位有权进行声明、且不在您列表中的人。让此人对一个没有声明、且无人依赖其结果的控制项进行声明，然后撤销该声明。在撤销之前，该声明可能会改变该控制项的结果。这两条活动都可能需要一段时间才能到达活动源。

如果您将这些活动存储为开放网络安全架构框架（Open Cybersecurity Schema Framework，OCSF）事件，[将 Security posture 事件映射到 OCSF](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/ocsf-mapping) 介绍了这些活动字段的映射方式。
