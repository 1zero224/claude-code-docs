---
title: Security posture 结果参考
url: https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference
description: 查询返回组织当前 Security posture 结果的 Compliance API 端点：请求、响应字段、每个状态的含义以及错误。
---

<Note>
  要启用 Compliance API，请参阅[设置 Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access)。
</Note>

此端点为组织的大部分安全控制项各返回一个条目（在请求时进行评估），并附带响应中包含的每个控制项的 [Security posture 声明](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#attestations)。您需要一个覆盖该组织、具有 `read:compliance_org_data` 或 `read:org_audit` 作用域的 Compliance Access Key；[Security posture 可用性](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#availability)说明了可以读取哪些组织。

## 请求

向 `/v1/compliance/organizations/{organization_id}/security_posture` 发送 `GET` 请求。每次调用读取一个组织，且响应不分页。

```bash cURL
curl --fail-with-body -sS \
  "https://api.anthropic.com/v1/compliance/organizations/91012d09-e48b-438e-a489-1bebfd8fa6f9/security_posture" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

| 项                   | 规则                                                                                                                                                                                                                                     |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `organization_id`   | 来自[列出组织](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-organizations)的[已关联组织](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api#how-the-compliance-api-works)的 `uuid`；任何其他值都会返回 404。 |
| 密钥                  | [父组织的密钥](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api-access#set-up-the-compliance-api)可以指定任一已关联组织，而单个组织的密钥只能指定该组织。                                                                                             |
| `anthropic-version` | 必需；不带该标头的请求返回 400。                                                                                                                                                                                                                     |
| 查询参数                | 无；带有任何查询参数的请求都会返回 400。                                                                                                                                                                                                                 |

## 响应

以下节选示例展示了响应中可能包含的四个控制项。它是对一个持有 `read:compliance_org_data` 和 `read:compliance_user_data` 的密钥的响应。

```json Response
{
  "type": "security_posture",
  "organization_id": "91012d09-e48b-438e-a489-1bebfd8fa6f9",
  "evaluated_at": "2027-02-09T09:00:02.417305Z",
  "comments_status": "not_enabled",
  "controls": [
    {
      "id": "sso_enabled",
      "display_name": "Single sign-on (SSO)",
      "section": "identity_access",
      "setting_name": "sso_enabled",
      "status": "passed",
      "rating": "passed",
      "current_value": "On",
      "desired_state": "On",
      "attestation": null
    },
    {
      "id": "directory_sync_enabled",
      "display_name": "Directory sync (SCIM)",
      "section": "identity_access",
      "setting_name": "directory_sync_enabled",
      "status": "failed_check_accepted_risk",
      "rating": "passed",
      "current_value": "Off",
      "desired_state": "On",
      "attestation": {
        "state": "in_force",
        "attested_at": "2026-10-20T15:04:00.220114Z",
        "comment": null,
        "attested_by": {
          "type": "user_actor",
          "user_id": "user_01WxYzAbCdEfGhJkMnPqRsTu"
        }
      }
    },
    {
      "id": "allowed_invite_domains",
      "display_name": "Allowed email domains",
      "section": "identity_access",
      "setting_name": "allowed_invite_domains",
      "status": "descriptive",
      "rating": "descriptive",
      "current_value": "example.com",
      "desired_state": null,
      "attestation": null
    },
    {
      "id": "oauth_conditional_access",
      "display_name": "OAuth conditional access on connected SaaS",
      "section": "connectors_integrations",
      "setting_name": null,
      "status": "not_attested",
      "rating": "not_attested",
      "current_value": null,
      "desired_state": "Completed",
      "attestation": {
        "state": "review_due",
        "attested_at": "2026-11-02T09:30:00.008771Z",
        "comment": null,
        "attested_by": {
          "type": "user_actor",
          "user_id": "user_01AbCdEfGhJkLmNpQrStUvWx"
        }
      }
    }
  ]
}
```

每个字段始终存在，没有值的字段为 `null`，绝不会被省略。时间要么没有小数位，要么有六位小数，因此请将时间作为时间进行比较，而不是作为字符串比较。

### 顶层字段

| 字段                | 类型                     | 说明                                                                                                                                                   |
| ----------------- | ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`            | string                 | 始终为 `security_posture`。                                                                                                                              |
| `organization_id` | string                 | 组织的 UUID。                                                                                                                                            |
| `evaluated_at`    | string (RFC 3339, UTC) | 评估所有控制项的时间。                                                                                                                                          |
| `comments_status` | string                 | 是否读取了声明备注；请参阅[声明者与备注](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#who-attested-and-comments)。 |
| `controls`        | array of objects       | 组织的控制项，绝不为空。                                                                                                                                         |

### `controls[]` 字段

| 字段              | 类型            | 说明                                                                                                                                                                                                                                                       |
| --------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`            | string        | 控制项的标识符，对每个组织都相同，且永不改变。                                                                                                                                                                                                                                  |
| `display_name`  | string        | 控制项的显示名称，为英文显示文本，其措辞可能会改变。                                                                                                                                                                                                                               |
| `section`       | string        | 控制项所属的部分，为本表之后列出的十个部分之一；可能会添加更多值，因此请将未知值视为一个独立的部分。                                                                                                                                                                                                       |
| `setting_name`  | string 或 null | 此控制项所读取的组织有效设置中条目的 `name`；当它不读取此类条目时为 `null`。                                                                                                                                                                                                            |
| `status`        | string        | 控制项结果的原因；请参阅[状态值](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#status-values)。                                                                                                                     |
| `rating`        | string        | 控制项的结果，由 `status` 决定；请参阅[状态值](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#status-values)。                                                                                                         |
| `current_value` | string 或 null | 当前值，为英文显示文本，其措辞可能会改变；在手动控制项上或未读取到值时为 `null`。                                                                                                                                                                                                             |
| `desired_state` | string 或 null | 推荐值，为英文显示文本，其措辞可能会改变；当此组织没有推荐值时为 `null`。                                                                                                                                                                                                                 |
| `attestation`   | object 或 null | 响应中包含的声明，在声明未处于生效中时也会包含；当 `status` 为 `descriptive`、`not_applicable` 或 `not_evaluated` 时，以及在[这些情况](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#how-an-attestation-shows-in-the-response)下为 `null`。 |

`section` 为以下值之一：`identity_access`、`data_privacy`、`capabilities`、`connectors_integrations`、`chats_desktop_mdm`、`cowork_admin_console`、`cowork_desktop_mdm`、`claude_code_admin_console`、`claude_code_managed_settings` 和 `healthy_adoption`。

要在代码中比较值，请在同一组织的[有效设置](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#get-effective-organization-settings)中查找 `setting_name`。查找可能找不到任何结果，且两个控制项可能共享同一个 `setting_name`。

### `attestation` 字段

| 字段            | 类型                     | 说明                                                                                                                                          |
| ------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `state`       | string                 | 声明是否生效；请参阅[声明的生效状态](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/results-reference#attestation-states)。 |
| `attested_at` | string (RFC 3339, UTC) | 作出声明的时间。                                                                                                                                    |
| `comment`     | string 或 null          | 随声明保存的备注，最多 2,000 个字符；如果没有备注，或 `comments_status` 不是 `available`，则为 `null`。                                                                  |
| `attested_by` | object 或 null          | 作出声明的人；当密钥既不持有 `read:compliance_user_data` 也不持有 `read:org_audit`、该人员的账户已被删除或正在被删除，或端点不返回该组织的声明人时为 `null`。                                   |

### `attested_by` 字段

| 字段        | 类型     | 说明                   |
| --------- | ------ | -------------------- |
| `type`    | string | 始终为 `user_actor`。    |
| `user_id` | string | 用户的 ID，以 `user_` 开头。 |

## 状态值

`status` 给出控制项结果的原因，每个状态对应一个 `rating`。

| `status`                     | 控制项类型 | 含义                                   | `rating`       | 页面上的显示                   |
| ---------------------------- | ----- | ------------------------------------ | -------------- | ------------------------ |
| `passed`                     | 自动    | 当前值满足推荐值。                            | `passed`       | “通过”                     |
| `matches_profile`            | 自动    | 当前值等于推荐值。                            | `passed`       | “通过”                     |
| `exceeds_baseline`           | 自动    | 当前值比推荐值更严格。                          | `passed`       | “通过”                     |
| `failed_check_accepted_risk` | 自动    | 自动检查未通过，且有一项已接受风险处于生效状态。             | `passed`       | “通过”                     |
| `failed`                     | 自动或手动 | 当前值不满足推荐值，或响应中未包含的某项声明导致此结果。         | `failed`       | “未通过”，或在有“已完成”声明生效时为“通过” |
| `review`                     | 自动    | 当前值比推荐值更宽松。                          | `failed`       | “未通过”                    |
| `attested_completed`         | 手动    | 值为“已完成”（`completed`）的声明处于生效状态。       | `passed`       | “通过”                     |
| `attested_accepted_risk`     | 手动    | 值为“已接受风险”（`accepted_risk`）的声明处于生效状态。 | `passed`       | “通过”                     |
| `not_attested`               | 手动    | 没有声明处于生效状态。                          | `not_attested` | “未声明”                    |
| `attestation_not_enabled`    | 手动    | 任何人都无法对该控制项作出声明。                     | `not_attested` | “描述性”                    |
| `descriptive`                | 仅供参考  | 该控制项仅作报告，未评定。                        | `descriptive`  | “描述性”                    |
| `not_applicable`             | 任意    | 该控制项不适用于该组织。                         | `descriptive`  | “描述性”                    |
| `not_evaluated`              | 任意    | 无法读取该设置，或无法读取决定该控制项是否适用的另一项设置。       | `descriptive`  | “描述性”                    |

没有字段标明控制项的类型；对于 `failed`，`current_value` 为 `null` 表示这是手动控制项。可能会添加更多值，因此请按 `rating` 统计具有未知 `status` 的控制项，并将未知的 `rating` 视为未获“通过”。状态 `review` 与生效状态 `review_due` 无关。

## 声明的生效状态

`state` 说明响应中包含的声明是否生效。

| `state`           | 含义                                   |
| ----------------- | ------------------------------------ |
| `in_force`        | 该声明生效中。                              |
| `review_due`      | 已到结束日期。                              |
| `setting_changed` | 检查在已接受风险的值以外的其他值上未通过，或该已接受风险已被记录为终止。 |
| `check_passing`   | 检查通过，因此没有需要接受的风险。                    |

当适用的原因超过一个时，响应按 `check_passing`、`setting_changed`、`review_due` 的顺序给出第一个适用的原因。从结束日期当天 00:00 UTC 起，声明不再生效。响应中没有结束日期，因此由 `state` 决定；已保存的结束日期即声明活动的 [`review_date`](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/events-reference#attestation-activity)。请将未知的 `state` 视为未生效。

## 声明在响应中的显示方式

响应会包含某些声明而省略其他声明，因此 `attestation` 为 `null` 并不能证明没有人作出声明。

| 控制项类型      | 声明为                     | `status`                                                 | `rating`       | `attestation`              |
| ---------- | ----------------------- | -------------------------------------------------------- | -------------- | -------------------------- |
| 手动         | 无声明                     | `not_attested`；如果任何人都无法作出声明，则为 `attestation_not_enabled` | `not_attested` | `null`                     |
| 手动         | “已完成”                   | `attested_completed`                                     | `passed`       | 显示                         |
| 手动         | “已接受风险”                 | `attested_accepted_risk`                                 | `passed`       | 显示                         |
| 手动         | “未通过”（`control_failed`） | `failed`                                                 | `failed`       | `null`：从不显示                |
| 自动，检查未通过   | 无声明                     | `failed` 或 `review`                                      | `failed`       | `null`                     |
| 自动，检查未通过   | “已接受风险”                 | `failed_check_accepted_risk`                             | `passed`       | 显示                         |
| 自动，检查未通过   | “已完成”                   | `failed`，即使自动检查给出的是 `review`                             | `failed`       | `null`：从不显示                |
| 自动，检查未通过   | “未通过”                   | 自动检查给出的 `status`                                         | `failed`       | `null`：从不显示                |
| 自动，检查通过    | 无声明                     | `passed`、`matches_profile` 或 `exceeds_baseline`          | `passed`       | `null`                     |
| 自动，检查通过    | “已完成”                   | 自动检查给出的 `status`                                         | `passed`       | `null`：从不显示                |
| 自动，检查通过    | “已接受风险”                 | 自动检查给出的 `status`                                         | `passed`       | 在检查基于已接受的值通过期间为 `null`     |
| 自动，检查通过    | “未通过”                   | `failed`                                                 | `failed`       | `null`：从不显示                |
| 仅供参考       | 任一值                     | `descriptive`                                            | `descriptive`  | `null`：从不显示                |
| 任意，声明未生效期间 | 前面某行中显示的值               | 与无声明时相同                                                  | 与无声明时相同        | 显示，且 `state` 不为 `in_force` |
| 任意，声明未生效期间 | 前面某行中省略的值               | 与无声明时相同                                                  | 与无声明时相同        | `null`                     |

除最后两行外，每一行都适用于声明生效期间。在自动控制项上，只要检查读取的值与保存声明时的值不同，该声明就不生效。响应在自动控制项上包含的每个声明都是已接受风险。[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)也会记录响应所省略的那些类型的声明。

## 统计结果

按 `rating` 统计控制项时需要检查以下四点。

| 如果您统计                | 还需检查                                                                                      |
| -------------------- | ----------------------------------------------------------------------------------------- |
| “通过”（`passed`）       | 它包括每个有已接受风险生效的控制项，因此请查找 `failed_check_accepted_risk` 和 `attested_accepted_risk`。          |
| “未通过”（`failed`）      | 它包括声明为“未通过”（`control_failed`）的控制项，以及自动检查未通过但有“已完成”声明生效的控制项，Security posture 页面将后者显示为“通过”。 |
| “描述性”（`descriptive`） | 它可能掩盖无法读取的控制项，因此请查找 `not_evaluated`。                                                      |
| 所有控制项                | 响应中缺失的控制项不计为“通过”，也不会被列出，因此请将 `id` 值与您以前在该组织中见过的值进行比较。                                     |

## 声明者与备注

只有当密钥同时持有 `read:compliance_user_data` 和 `read:compliance_org_data`，或持有 `read:org_audit` 时，才会填充 `attested_by`。仅持有 `read:compliance_user_data` 的密钥会收到 403。`attested_by` 中的 `user_id` 与声明活动上 `actor` 的 `user_id` 相同。`attested_by` 仅包含用户 ID：[列出组织用户](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-org-data#list-organization-users)会提供每位当前成员的姓名和电子邮件地址。在添加这些作用域之前，请参阅[谁可以看到声明者](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#who-can-see-who-attested)。

此端点不返回备注，即使 Security posture 页面上显示了备注：每个 `comment` 都为 `null`，且 `comments_status` 为 `not_enabled` 或 `not_permitted`。因此，`not_enabled` 和为 `null` 的备注并不表示该声明没有备注。`comments_status` 说明如何解读为 `null` 的备注，schema 定义了以下值。

| `comments_status`           | 含义                                                                                                       |
| --------------------------- | -------------------------------------------------------------------------------------------------------- |
| `not_permitted`             | API 密钥既不持有 `read:compliance_user_data` 也不持有 `read:org_audit`，因此每个 `comment` 和每个 `attested_by` 都为 `null`。 |
| `requires_organization_key` | API 密钥不是在该组织中创建的（例如父组织的密钥），因此每个 `comment` 都为 `null`。重试无济于事。                                              |
| `available`                 | 已读取备注，因此为 `null` 的备注表示没有备注，或备注是刚刚保存的。                                                                    |
| `not_enabled`               | 此端点不为该组织返回备注，因此每个 `comment` 都为 `null`。                                                                   |
| `unavailable`               | 无法为此响应读取备注；稍后的请求可能会成功，但某些原因会一直持续，直到客户采取行动。                                                               |

可能会添加更多值；请将未知值视为 `unavailable`。

## 错误

错误使用与 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors) 其余部分相同的正文；下表列出此端点以其特有方式返回的错误。

| 状态和 `error.type`                                                                  | 何时                                                                    | 处理方法                                                                                                                                              |
| --------------------------------------------------------------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 400 `invalid_request_error`                                                       | `anthropic-version` 缺失或无效、父组织未启用 Compliance API，或存在查询参数。              | 修正 `error.message` 指出的问题。                                                                                                                         |
| 401 `authentication_error`                                                        | 密钥缺失、无法识别，或不是 Compliance Access Key 或 Admin API 密钥。                   | 在 `x-api-key` 中发送有效的 Compliance Access Key。                                                                                                       |
| 403 `permission_error`                                                            | 该密钥不得调用此端点，通常是因为它既缺少 `read:compliance_org_data` 也缺少 `read:org_audit`。 | 使用具有其中任一作用域的密钥。                                                                                                                                   |
| 404 `not_found_error`                                                             | 请参阅此表格之后列出的原因。                                                        | 不要立即重试，也绝不要将其视为没有未通过的控制项。                                                                                                                         |
| 429 `rate_limit_error`                                                            | 共享的 Compliance API 速率限制或此端点更低的限制已用尽；`error.message` 会说明是哪一个。          | 等待 `retry-after` 指定的时间；请参阅 [429 Too Many Requests](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors#429-too-many-requests)。 |
| 503 `overloaded_error`，且 `error.details.error_code` 为 `posture_evaluation_failed` | 无法生成结果。                                                               | 等待 `retry-after` 指定的时间后重试；如果在之后的运行中某个组织再次出现此问题，请联系 Anthropic 支持一次。                                                                                |
| 不带 `error_code` 的 503，或 504 `timeout_error`                                       | 临时故障。                                                                 | 使用指数退避重试。                                                                                                                                         |

404 的原因：

* `organization_id` 不是 UUID、未指向任何组织，或指向已删除的组织。
* `organization_id` 指向父组织、另一个父组织下的组织，或单组织密钥未覆盖的组织。
* 该组织未使用 Claude Enterprise，或 Security posture 对其不可用，例如 Claude Console 组织。
* 该组织的 Security posture 页面已被关闭（请参阅 [Security posture 可用性](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-security-posture/overview#availability)）。
* Security posture 对该组织可用，但此端点不为其提供服务。
* 无法检查可用性；稍后的请求可能会成功。

每个 404 都具有相同的正文，不存在的路径也是如此。

## 时效性和副作用

此端点在您调用时评估每个控制项，`evaluated_at` 即为该时间。

* 最近几分钟内的变更可能不会显示。
* 即使没有任何设置发生变更，结果也可能改变，例如到达结束日期时，或 Anthropic 更改推荐值时。
* 如果 Anthropic 改写了某个当前值的措辞，则基于该值的已接受风险不处于生效状态。
* 调用不会存储任何结果，也不会终止任何声明。
* 每个通过密钥、作用域和版本检查且不带查询参数的请求都会记录一个 `compliance_api_accessed` 活动，无论请求成功还是失败；其 `url` 指明所读取的组织，对于父组织的密钥，`organization_uuid` 为 `null`。
