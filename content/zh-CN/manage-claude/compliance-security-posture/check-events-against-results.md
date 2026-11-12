---
title: 将已存储的 Security posture 事件与当前结果进行核对
url: https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/check-events-against-results
description: 每天一次，将您的 SIEM 已存储的声明活动与端点的当前结果进行比较，并决定哪些不一致需要人工处理。
---

<Note>
  要启用 Compliance API，请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)。
</Note>

活动源可能会延迟交付活动，而端点不保留历史记录，因此每个来源都可能显示另一个来源所没有的变更。每日核对可以找出您的安全信息和事件管理（security information and event management，SIEM）存储中缺少的内容。活动源并不记录[每项变更](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference#what-the-feed-does-not-record)，因此某个声明可能早于活动源中该安全控制项的第一个活动，在极少数情况下，某项变更根本没有活动。如果您只报告当前结果，则不需要此核对。

您需要一个覆盖该组织的 Compliance Access Key，其中包含用于端点的 `read:compliance_org_data` 和用于活动源的 `read:compliance_activities`，或者包含同时适用于两者的 `read:org_audit`。

## 运行核对

请在当天的[每日重新读取](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events#re-read-15-days-once-a-day)完成后再开始每次运行，以确保延迟的活动已被存储。

1. 使用[列出组织](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-organizations)列出已关联的组织。您会获得每个组织的 `uuid`。
2. 对每个组织[调用一次端点](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#request)。您会得到 200；对于端点不提供服务的组织（请参阅 [Security posture 可用性](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#availability)），每次运行都会得到 404。404 表示本次运行未读取该组织。记录返回 200 的组织列表，并为每个组织记录其首个 200 的 `evaluated_at`：即您对该组织的首次运行。
3. 将[搁置规则](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/check-events-against-results#set-aside-what-cannot-be-compared)中列出的内容搁置。
4. 将该组织剩余的已存储声明活动按安全控制项分组，并为每个控制项取 `created_at` 最新的一个。每个控制项会得到一个活动，或者没有活动。
5. 对剩下的每个控制项，在[判定表](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/check-events-against-results#decide-what-each-mismatch-means)中找到其对应的行。您会得到一致、预期内或需核查。
6. 仅当下一次读取该组织的运行再次显示某个“需核查”项时，才对其采取行动；对第 6 行则立即采取行动，对于第 10 行和第 11 行，则改为遵循[占位记录规则](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/check-events-against-results#keep-a-placeholder-where-the-two-sources-cannot-be-matched)。未读取该组织的运行既不计入，也不会重新开始等待。

请控制调用频率，并按照 [429 Too Many Requests](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors#429-too-many-requests) 中的说明处理 429；如果某个组织的调用在您重试后仍然失败，则本次运行未读取该组织，失败的调用也不代表变更。

## 匹配字段

使用以下字段，将响应中的条目与已存储的活动进行匹配。

| 在响应中                      | 在活动中                                                  | 两者的关系                                                                                                                                                                   |
| ------------------------- | ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `organization_id`         | `organization_uuid`                                   | 相同的值；活动自身的 `organization_id` 带有 `org_` 前缀。                                                                                                                              |
| `controls[].id`           | `control`                                             | 相同的标识符，[规则 S2](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/check-events-against-results#set-aside-what-cannot-be-compared) 中的情况除外。 |
| `attestation.attested_at` | 最新的 `attested`、`changed` 或 `renewed` 活动的 `created_at` | 在响应中包含该声明时为同一时间；请按时间而非字符串进行比较。                                                                                                                                          |
| `status`                  | `status`                                              | 在声明生效中时，`attested_completed` 对应 `completed`，`attested_accepted_risk` 和 `failed_check_accepted_risk` 对应 `accepted_risk`；没有任何值对应 `control_failed`。                        |

## 搁置无法比较的内容

在比较之前，先将以下内容搁置。

| #  | 内容                                                                       | 规则                                         |
| -- | ------------------------------------------------------------------------ | ------------------------------------------ |
| S1 | `action` 不是 `attested`、`changed`、`renewed`、`cleared` 或 `lapsed` 的已存储声明活动 | 在整个核对中排除它；对于每个新的 `action` 值，标记一次以供审查。      |
| S2 | 指向 `mdm_cowork_required_organization` 的已存储声明活动                           | 将其计为 `mdm_chat_required_organization` 的活动。 |
| S3 | 其控制项在响应中没有条目的已存储声明活动                                                     | 无可比较的内容；将该控制项报告为不在响应中，绝不报告为“通过”（`passed`）。 |
| S4 | `status` 为 `descriptive`、`not_applicable` 或 `not_evaluated` 的条目          | 跳过它；其为 `not_evaluated` 的运行不计入该控制项。         |

## 判断每个不一致的含义

剩下的每个控制项恰好符合一行。

| #  | 最新的已存储活动                                                                      | 响应中的条目                                                                                                         | 判定  | 处理方式                                                                                                                                                                                                                |
| -- | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | `attested`、`changed` 或 `renewed`                                              | `attested_at` 等于其 `created_at`；`state` 为 `in_force`；`status` 与其值对应。                                            | 一致  | 无需操作。                                                                                                                                                                                                               |
| 2  | `cleared`                                                                     | `attestation` 为 `null`，第 11 行中的情况除外。                                                                           | 一致  | 无需操作。                                                                                                                                                                                                               |
| 3  | `lapsed`，值为“已接受风险”（`accepted_risk`）                                           | `attestation` 为 `null`，或者带有较早活动的时间且 `state` 为 `check_passing` 或 `setting_changed`。                             | 一致  | 无需操作。                                                                                                                                                                                                               |
| 4  | `lapsed`，值为 `completed` 或 `control_failed`                                    | `attestation` 为 `null`。                                                                                        | 一致  | 无需操作。                                                                                                                                                                                                               |
| 5  | `attested`、`changed` 或 `renewed`                                              | `attested_at` 等于其 `created_at`；`state` 为 `review_due`、`check_passing` 或 `setting_changed`。                     | 预期内 | 不处于生效状态：对每个新的状态采取一次行动，无需等待活动；恢复为 `in_force` 会重置此项。                                                                                                                                                                  |
| 6  | `attested`、`changed` 或 `renewed`，值为“已完成”（`completed`），且 `created_at` 晚于您的首次运行 | 在首次比较该活动的运行中，`attestation` 为 `null`，且 `status` 为带有 `current_value` 的 `failed`。                                 | 需核查 | Security posture 页面可能会将检查未通过的项显示为“通过”：标记以供审查，无需等待。                                                                                                                                                                  |
| 7  | `attested`、`changed` 或 `renewed`，值为 `completed`                               | `attestation` 为 `null`；`status` 既不是 `not_attested`，也不是 `current_value` 为 `null` 的 `failed`；且不符合第 6 行。          | 预期内 | 无可比较的内容；响应中从不包含它。                                                                                                                                                                                                   |
| 8  | `attested`、`changed` 或 `renewed`，值为“未通过”（`control_failed`）                    | `attestation` 为 `null`；`status` 为以下三者之一：`failed`；自动检查给出的任意状态；或者，从结束日期（`review_date`，UTC 时间）起，为 `not_attested`。 | 预期内 | 无可比较的内容；从结束日期起，当 `status` 不再是 `failed` 时，采取一次行动。                                                                                                                                                                    |
| 9  | `attested`、`changed` 或 `renewed`，值为 `accepted_risk`                           | `attestation` 为 `null`，且 `status` 为 `passed`、`matches_profile` 或 `exceeds_baseline`。                           | 预期内 | 存储一条占位记录（[占位记录规则 7](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/check-events-against-results#keep-a-placeholder-where-the-two-sources-cannot-be-matched)）。                     |
| 10 | 无，或者所有已存储活动都早于 `attested_at`                                                  | 存在 `attestation`。                                                                                              | 需核查 | 遵循占位记录规则。                                                                                                                                                                                                           |
| 11 | 无，或者有一个值不是 `control_failed` 的活动                                               | `current_value` 为 `null`，且 `status` 为 `failed`。                                                                | 需核查 | 您的存储中缺少一个值为“未通过”（`control_failed`）的声明：遵循占位记录规则，并在 `status` 不再是 `failed` 时采取一次行动。                                                                                                                                    |
| 12 | `attested`、`changed`、`renewed` 或 `cleared`，晚于 `attested_at`，且存在另一个几乎同一时间的活动   | 存在 `attestation`。                                                                                              | 预期内 | 几乎同一时间做出的变更可能以任一顺序存储。                                                                                                                                                                                               |
| 13 | 同上，但没有这样的第二个活动                                                                | 存在 `attestation`。                                                                                              | 需核查 | 联系一次 Anthropic 支持团队。                                                                                                                                                                                                |
| 14 | `attested`、`changed` 或 `renewed`                                              | `attested_at` 等于其 `created_at`；`state` 为 `in_force`；`status` 与其值不对应。                                           | 需核查 | 联系 Anthropic 支持团队，仅联系一次。                                                                                                                                                                                            |
| 15 | `attested`、`changed` 或 `renewed`；之前的各行均不符合                                    | `attestation` 为 `null`。                                                                                        | 需核查 | 该声明最可能已被撤销：存储一条占位记录（[占位记录规则 7](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/check-events-against-results#keep-a-placeholder-where-the-two-sources-cannot-be-matched)）并标记一次以供审查。 |
| 16 | 无                                                                             | `attestation` 为 `null`，且不符合第 11 行。                                                                             | 一致  | 无需操作。                                                                                                                                                                                                               |
| 17 | 任何其他情况                                                                        | 任何其他情况。                                                                                                        | 需核查 | 标记一次以供审查。                                                                                                                                                                                                           |

## 在两个来源无法匹配时保留占位记录

占位记录代表两个来源之一所缺少的内容，您需要将其与已存储的活动分开保存。对于第 10 行和第 11 行，请再次读取活动源，仅当在 15 天内（即[每日重新读取](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/send-events#re-read-15-days-once-a-day)所覆盖的范围）没有任何活动到达时，才存储并标记该占位记录。在首次运行时，立即存储占位记录，无需审查。

第 10 行的占位记录，以控制项和 `attested_at` 作为唯一标识：

1. 从该控制项最新的已存储活动开始再次读取；如果您尚未存储任何活动，则从 `attested_at`（含）开始读取；15 天从 `attested_at` 开始计算。根据该条目构建占位记录，并标明其没有活动；在判定表中，将其视为一个位于该 `attested_at` 时间、值与其 `status`（如有）对应的 `attested` 活动。
2. 在每次显示该声明的运行中，更新其 `status` 和 `state`。
3. 当 `created_at` 等于该 `attested_at` 的活动到达时，用该活动替换占位记录。
4. 当连续两次运行都未显示该声明时，解决该占位记录；除非该控制项较新的已存储活动或未解决的第 10 行不一致可以解释这种情况，或者两次运行都显示 `descriptive`，否则将其标记以供审查。
5. 未读取该组织或显示 `not_evaluated` 的运行，既不计入这两次运行，也不会使其重新开始计数。
6. 如果该声明再次出现，重新打开同一条占位记录。

第 9、11 和 15 行的占位记录，仅以控制项作为唯一标识：

7. 对于第 11 行，15 天从首次显示该 `status` 的运行开始计算。在占位记录未解决期间，不再为该控制项标记任何内容；当较新的活动到达、该声明出现，或连续两次运行符合另一行时，解决该占位记录。
8. 当第 9 行的占位记录因该声明显示为 `failed_check_accepted_risk` 而解决时，标记一次以供审查：该风险处于生效中。

## 示例详解

以下两次运行读取的是结果参考中[响应示例](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#response)里的组织；运行 1 是您的首次运行。

* 第 1 次运行，`directory_sync_enabled`。已存储：`attested`、`accepted_risk`，创建于 `2026-10-20T15:04:00.220114Z`。响应：`failed_check_accepted_risk`、`in_force`，相同的 `attested_at`。第 1 行：一致。
* 第 1 次运行，`allowed_invite_domains`。已存储：无。响应：`descriptive`，`attestation` 为 `null`。规则 S4：搁置。
* 第 1 次运行，`oauth_conditional_access`。已存储：无。响应：一个 `state` 为 `review_due` 的 `attestation`。第 10 行：需核查。在您的首次运行中，立即存储一条占位记录，不进行审查。
* 第 2 次运行，`oauth_conditional_access`。占位记录：第 1 次运行中的那条。响应：相同。第 5 行：预期内。对 `review_due` 采取一次行动。

## 核对无法发现的情况

对于响应在自动控制项上省略的声明，每日核对无法发现其缺失的活动。响应不会给出此类声明的任何迹象，或者仅给出 `status` `failed`，而这看起来与自动检查给出的状态相同。

## 当组织停止响应时

某个组织曾返回 200，随后连续多次运行都返回 404，此时需要人工处理一次。请询问有权访问 Security posture 页面的人：针对该组织，Security posture 页面是否已被关闭（请参阅[Security posture 可用性](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#availability)）。如果 Security posture 页面未被关闭，请联系您的 Anthropic 代表。该组织可能已被删除，或者您对该组织的访问权限、Security posture 或端点可能已终止。无论原因如何，都请解决该组织的占位记录和未解决的不一致，无需审查，继续调用该组织，如果该组织再次返回 200，则重新打开这些记录。
