---
title: 将 Security posture 事件映射到 OCSF
url: https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/ocsf-mapping
description: 在存储 Compliance API 中的每个 Security posture 活动之前，将其映射到相应的 Open Cybersecurity Schema Framework（OCSF）类和属性。
---

<Note>
  要启用 Compliance API，请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)。
</Note>

[Open Cybersecurity Schema Framework](https://schema.ocsf.io/)（OCSF）是一种用于安全事件的开放架构。请在您自己的管道中，于读取 [Security posture](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview) 活动之后、存储之前应用此映射。若要将它们导入您的安全信息和事件管理（SIEM）系统，请从[将 Security posture 事件发送到您的 SIEM](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events) 开始。

## OCSF 版本

此映射使用 OCSF 1.6.0。如果您针对其他版本进行验证，请对照该版本检查本页中的属性。例如，在 OCSF 1.7.0 中，`metadata.uid` 是 OCSF 事件本身的标识符，而 `metadata.original_event_uid` 保存来自源的标识符。OCSF 1.9.0 弃用了 `actor.app_name`，改用 `actor.application`。

## 适用于每个 Security posture 活动的属性

请在每个事件上映射以下字段。

| 活动字段                | OCSF 属性                        | 说明                                                                                                                                                                              |
| ------------------- | ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                | `metadata.uid`                 | 原样复制。                                                                                                                                                                           |
| `type`              | `metadata.event_code`          | 原样复制。                                                                                                                                                                           |
| `created_at`        | `time`                         | 将 RFC 3339 字符串转换为自 Unix 纪元以来的毫秒数，并截断为整数。                                                                                                                                        |
| `created_at`        | `metadata.original_time`       | 原样复制该字符串。                                                                                                                                                                       |
| 无                   | `metadata.logged_time`         | 您存储该活动的时间（即 [检测变更](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/detect-changes#before-you-write-a-rule) 中的 `stored_at`），以自 Unix 纪元以来的毫秒数表示。 |
| `organization_uuid` | `metadata.tenant_uid`          | 原样复制。                                                                                                                                                                           |
| `organization_id`   | `unmapped.organization_id`     | 原样复制。                                                                                                                                                                           |
| 无                   | `metadata.version`             | 常量：`1.6.0`。                                                                                                                                                                     |
| 无                   | `metadata.product.vendor_name` | 常量：`Anthropic`。                                                                                                                                                                 |
| 无                   | `metadata.product.name`        | 常量：`Compliance API`。                                                                                                                                                            |
| 无                   | `type_uid`                     | `class_uid * 100 + activity_id`。                                                                                                                                                |
| 无                   | `severity_id`                  | 在每个事件上设置为 `1`（Informational），并在触发告警的[检测规则](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/detect-changes)中设置该告警的优先级。                          |

将每个没有对应 OCSF 属性的字段以其在活动源中的名称放入 `unmapped` 对象，包括您的处理程序无法识别的字段。如果某个字段不存在或为 `null`，则省略对应的 OCSF 属性。

映射 `actor` 对象：

| 活动字段                  | OCSF 属性                   | 说明                                         |
| --------------------- | ------------------------- | ------------------------------------------ |
| 无                     | `metadata.profiles`       | 添加 `host`，即承载 `actor` 的 OCSF 配置文件（一组可选属性）。 |
| `actor.type`          | `unmapped.actor.type`     | 原样复制。                                      |
| `actor.user_id`       | `actor.user.uid`          | 仅限 `user_actor`。                           |
| `actor.email_address` | `actor.user.email_addr`   | 仅限 `user_actor`。                           |
| `actor.ip_address`    | `src_endpoint.ip`         | 仅限 `user_actor`。                           |
| `actor.user_agent`    | `http_request.user_agent` | 仅限 `user_actor`。                           |
| `actor.service`       | `actor.app_name`          | 仅限 `system_actor`。                         |

## 声明活动：Entity Management

将 [`org_security_control_attestation_updated`](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference#attestation-activity) 活动映射到 OCSF Entity Management 类（3004）。Entity Management 事件报告对某个受管实体执行的操作，此处该实体是组织针对某个安全控制项的声明。

| 活动字段              | OCSF 属性                    | 说明                                        |
| ----------------- | -------------------------- | ----------------------------------------- |
| 无                 | `class_uid`                | 常量：`3004`（Entity Management）。             |
| 无                 | `category_uid`             | 常量：`3`（Identity & Access Management）。     |
| `action`          | `activity_id`              | 取自下方的 `action` 值表。                        |
| `action`          | `unmapped.action`          | 原样复制，因为有两个 `action` 值共用同一个 `activity_id`。 |
| 无                 | `entity.type_id`           | 常量：`99`（Other）。                           |
| 无                 | `entity.type`              | 常量：`Security control attestation`。        |
| `control`         | `entity.uid`               | 原样复制。                                     |
| `status`          | `entity.data.status`       | 原样复制：声明的值。                                |
| `review_date`     | `entity.data.review_date`  | 原样复制：声明的结束日期。                             |
| `previous_status` | `unmapped.previous_status` | 原样复制：变更之前声明的值。                            |

不要将 `status` 映射到 `status_id` 或任何通过/未通过的值：声明是某个人对控制项所作的陈述，而不是检查的结果。

| `action`   | `activity_id`    | `type_uid` |
| ---------- | ---------------- | ---------- |
| `attested` | `1`（Create）      | `300401`   |
| `changed`  | `3`（Update）      | `300403`   |
| `renewed`  | `3`（Update）      | `300403`   |
| `cleared`  | `4`（Delete）      | `300404`   |
| `lapsed`   | `11`（Deactivate） | `300411`   |
| 任何其他值      | `99`（Other）      | `300499`   |

对于 `99`（Other），请将 `activity_name` 设置为该 `action` 值。

`attested` 活动映射后的事件如下所示（使用示例值）：

```json
{
  "class_uid": 3004,
  "category_uid": 3,
  "activity_id": 1,
  "type_uid": 300401,
  "severity_id": 1,
  "time": 1794495872517,
  "metadata": {
    "version": "1.6.0",
    "profiles": ["host"],
    "uid": "activity_01Kp5RvNd8Wc3TgMy6BxQf4Z",
    "event_code": "org_security_control_attestation_updated",
    "original_time": "2026-11-12T15:04:32.517208Z",
    "logged_time": 1794495910000,
    "tenant_uid": "91012d09-e48b-438e-a489-1bebfd8fa6f9",
    "product": { "vendor_name": "Anthropic", "name": "Compliance API" }
  },
  "actor": {
    "user": {
      "uid": "user_01TuVwXyZaBcDeFgH2JkLmN4",
      "email_addr": "admin@example.com"
    }
  },
  "src_endpoint": { "ip": "192.0.2.34" },
  "http_request": { "user_agent": "ExampleBrowser/1.0" },
  "entity": {
    "type_id": 99,
    "type": "Security control attestation",
    "uid": "mdm_chat_required_organization",
    "data": { "status": "completed" }
  },
  "unmapped": {
    "action": "attested",
    "actor": { "type": "user_actor" },
    "organization_id": "org_01JuY4bkwkH1x4PodBJaWmh2"
  }
}
```
