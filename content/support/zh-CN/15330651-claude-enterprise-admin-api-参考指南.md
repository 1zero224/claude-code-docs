# Claude Enterprise Admin API 参考指南

本指南涵盖使用 Claude Enterprise Admin API 为 Claude Enterprise 组织设置的**支出限额**和**支出限额增加请求**。支出限额让您可以限制每个成员在循环周期内的使用额度支出，查看每个成员的限额来源，以及审查或处理成员的更高限额请求。

有关按用户和时间分段的使用情况和成本报告，请参阅**[Analytics API 参考指南](https://platform.claude.com/docs/en/api/admin/analytics)**。要使用 Admin API 设置项目标签（测试版），请参阅**[使用项目标签按成本代码跟踪支出](https://support.claude.com/en/articles/18251257-track-spend-by-project-with-tags)**。

Claude Enterprise Admin API 目前处于公开测试版，可供启用了使用额度的 Enterprise 计划组织使用。

## 概述

共有八个端点跨越两个资源：

| **资源**       | **端点**                                                                                                                                                                                                                                                         | **用途**                            |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **支出限额**     | `GET /v1/organizations/spend_limits/effective`<br>`GET /v1/organizations/spend_limits/{spend_limit_id}`<br>`POST /v1/organizations/spend_limits`<br>`DELETE /v1/organizations/spend_limits/{spend_limit_id}`                                                   | 读取每个成员的有效限额和周期至今的支出；设置或清除按用户的覆盖。  |
| **支出限额增加请求** | `GET /v1/organizations/spend_limit_increase_requests`<br>`GET /v1/organizations/spend_limit_increase_requests/{id}`<br>`POST /v1/organizations/spend_limit_increase_requests/{id}/approve`<br>`POST /v1/organizations/spend_limit_increase_requests/{id}/deny` | 列出成员的更高限额请求及做出决定所需的上下文；批准或拒绝每个请求。 |

使用**支出限额**端点来回答"每个成员适用什么限额、它来自哪里以及他们离限额有多近？"并设置按用户的覆盖。使用**支出限额增加请求**端点来处理成员提交的请求队列。

## 前置条件和身份验证

- 您的组织必须在 Claude Enterprise 计划上。

- 必须为您的组织启用使用额度。您的主要所有者可以在 Claude 中的**计费设置**下启用此功能。

- 主要所有者需要创建具有以下一个或两个作用域的 Admin API 密钥：

  - **`read:spend_limits`**（所有 `GET` 端点必需）

  - **`write:spend_limits`**（`POST` 和 `DELETE` 端点必需）

在每个请求的**`x-api-key`**标头中传递密钥。

**重要：**不要公开共享 API 密钥或将其检入源代码管理。

## 为您的 Claude Enterprise 组织创建 Admin API 密钥

1. **以主要所有者身份登录**——只有您的 Claude Enterprise 父组织的主要所有者才能创建这些密钥。

2. **打开 API 设置**——转到**[组织设置 > API](https://claude.ai/admin-settings/api-access)**并找到**密钥**部分。

3. **点击并创建密钥**——命名密钥并从**[作用域表](https://platform.claude.com/docs/en/manage-claude/admin-api-keys#choose-scopes-for-a-claude-enterprise-key)**中选择所需的作用域。您可以在单个密钥上组合来自不同 API 的作用域（例如，`read:analytics` 和 `read:spend_limits`）。

4. **复制并存储密钥**——复制显示的密钥（以 `sk-ant-api01-` 开头）并将其存储在您的密钥管理器中。完整密钥仅显示一次。

## 基础 URL

```
https://api.anthropic.com
```

## 速率限制

所有八个端点共享单个每组织限制，为**每分钟 60 个请求**。超过限制的请求返回**429 Too Many Requests**。

## 分页

`GET /v1/organizations/spend_limits/effective` 和 `GET /v1/organizations/spend_limit_increase_requests` 使用**不透明游标**进行分页。第一个请求返回最多 `limit` 行加上 `next_page` 游标。在下一个请求中将该游标原样传递为 `page` 参数，并重复直到 `next_page` 为 `null`。

**重要：**不要在序列中途更改查询参数。游标绑定到发出它们的过滤器。如果您更改 `user_ids[]`、`status[]` 或 `actor_ids[]` 并传递旧游标，您将收到 400 错误，显示"cursor does not match current query parameters"。改为从第一页开始新序列。

将游标字符串视为不透明：不要自己解析、修改或构造它。

## 序列化列表参数

列表参数使用括号表示法：对每个值重复参数名称加上 `[]`。

```
user_ids[]=user_01AbCdEfGh&user_ids[]=user_01JkLmNoPq
```

## 错误响应

| **状态** | **含义**                                                      |
| ------ | ----------------------------------------------------------- |
| 400    | 输入无效、不支持的参数值、页面游标与当前参数不匹配，或不满足前置条件（请参阅每个端点的验证）。             |
| 401    | 缺少 `x-api-key` 标头。                                          |
| 403    | API 密钥缺少必需的作用域（`read:spend_limits` 或 `write:spend_limits`）。 |
| 404    | 找不到资源，或 API 密钥未知、已过期或已被撤销。                                  |
| 429    | 超出速率限制。                                                     |
| 500    | 内部错误。                                                       |

错误正文的形式为：

```
{"type": "error", "error": {"type": "<error_type>", "message": "..."}, "request_id": "req_..."}
```

`error.type` 是状态相关的判别器：`invalid_request_error` (400)、`authentication_error` (401)、`permission_error` (403)、`not_found_error` (404)、`rate_limit_error` (429)、`api_error` (500)。`request_id` 始终存在，是联系支持时应引用的值。每个端点下的"验证"表列出了具体的消息。

---

## 概念

### 支出限额层级

每个成员的使用额支出受到**有效支出限额**的限制，该限额从一个范围级别的层级中解析。当成员没有按用户覆盖时，他们继承为其座位层级、其组（如果您的组织使用基于组的限额）或组织范围默认值配置的限额。

读取 `GET /v1/organizations/spend_limits/effective` 返回每个当前成员及其已解析的有效限额、该限额的解析来源（`source`）和其期间至今的支出。通过 `POST /v1/organizations/spend_limits` 设置按用户覆盖会将成员固定到特定限额，无论他们本应继承什么。删除覆盖会将其返回到继承的限额。

### 范围

范围标识支出限额的写入或解析级别：

| **类型**         | **字段**          | **含义**                                                                                  |
| -------------- | --------------- | --------------------------------------------------------------------------------------- |
| `user`         | `user_id`       | 特定成员。`user_id` 与管理员 API 用户端点返回的 ID 匹配。                                                  |
| `seat_tier`    | `seat_tier`     | 座位层级默认值。`seat_tier` 值是完全限定的标识符，例如 `enterprise_standard` 或 `enterprise_tier_1`；可能会添加其他值。 |
| `rbac_group`   | `rbac_group_id` | 组默认值，当您的组织按组管理限额时。                                                                      |
| `organization` | —               | 组织范围的默认值。                                                                               |

`scope.type` 是一个开放字符串。客户端应将未知值视为不透明值并回退，而不是失败。未来可能会添加其他范围类型。

### 周期

`period` 是强制执行限额和支出重置的循环窗口。目前唯一的值是 `"monthly"`。

`period` 是一个开放字符串。客户端应将未知值视为不透明值并回退，而不是失败。未来可能会添加其他周期值。

### 金额和货币

所有货币值都是**组织计费货币的最小单位**（美元为美分）中的字符串。例如，`"50000"` 表示 500.00 美元。解析为十进制并除以 100 以显示美元。对于大值，避免使用二进制浮点数。

`amount` 是**可为空的**：`null` 表示**无限制**（无限额）。`"0"` 表示该成员**禁用**了使用额。

`period_to_date_spend` 是成员自当前 `period` 开始以来累积的使用额，采用相同的最小单位格式。它可能包含小数部分（例如，`"41280.125"`）。

### 增加请求生命周期

当成员在 claude.ai 中点击"请求更多使用"时，会创建**支出限额增加请求**。请求不通过此 API 创建。

| **状态**     | **含义**                                                                                                                               |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `pending`  | 等待管理员操作。请求通常包含实时 `spend_summary`，以便您在决定时可以看到成员的当前有效限额和期间至今的支出。如果无法计算，`spend_summary` 可能为 `null`，请处理该情况。                              |
| `approved` | 请求已通过批准解决：管理员明确批准了它（按管理员提供的金额写入按用户支出限额）、另一个管理员操作使成员可以使用使用额（例如，提高座位层级限额或为组织启用使用额计费），或 Anthropic 支持代表组织提高了限额。`spend_summary` 为 `null`。 |
| `denied`   | 管理员拒绝。`spend_summary` 为 `null`。Claude.ai 从 `resolved_at` 起 30 天内隐藏该成员的请求按钮；管理员仍可随时直接提高成员的限额。                                         |

`approved` 和 `denied` 都是终止状态。一个成员最多同时有一个 `pending` 请求。

通过 `POST …/approve` 批准会写入与 `POST /v1/organizations/spend_limits` 相同的每用户支出限额行。直接设置支出限额*不会*转换待处理请求。使用批准端点来解决请求。

默认情况下，Anthropic 会在成员的请求被批准或拒绝时向其发送电子邮件。在批准或拒绝时传递 `suppress_notification: true` 来抑制该电子邮件（例如，当您自己的系统通知成员时）。

---

## SpendLimit 对象

在一个范围级别配置的限额。

```
{
  "type": "spend_limit",
  "id": "spl_01AbCdEfGhIjKlMnOpQrSt",
  "created_at": "2026-05-01T12:00:00Z",
  "updated_at": "2026-05-03T09:14:11Z",
  "scope": { "type": "user", "user_id": "user_01AbCdEfGh" },
  "amount": "50000",
  "currency": "USD",
  "period": "monthly"
}
```

| **字段**       | **类型**            | **描述**                               |
| ------------ | ----------------- | ------------------------------------ |
| `type`       | string            | 始终为 `"spend_limit"`。                 |
| `id`         | string            | 前缀为 `spl_`。                          |
| `created_at` | string (RFC 3339) | 首次设置此限额的时间。                          |
| `updated_at` | string (RFC 3339) | 上次更改此限额的时间。                          |
| `scope`      | Scope             | 写入此限额的级别。请参阅 "Scope" 部分。             |
| `amount`     | string 或 null     | `period` 的限额，以最小单位表示。`null` 表示无限制。   |
| `currency`   | string            | ISO 4217。组织的计费货币。                    |
| `period`     | string            | 强制执行 `amount` 的循环窗口。请参阅 "Period" 部分。 |

## SpendSummary 对象

计算的每成员报告行：成员的有效限额、其来源以及其期间至今的支出。不是可寻址资源（无 `id`）。

```
{
  "scope": { "type": "user", "user_id": "user_01AbCdEfGh" },
  "amount": "50000",
  "currency": "USD",
  "period": "monthly",
  "source": { "type": "seat_tier", "seat_tier": "enterprise_standard" },
  "spend_limit_id": "spl_01XyZaBcDeFgHiJkLmNoPq",
  "period_to_date_spend": "31402.5"
}
```

| **字段**                 | **类型**                | **描述**                                                                                       |
| ---------------------- | --------------------- | -------------------------------------------------------------------------------------------- |
| `scope`                | Scope（`type: "user"`） | 此行所对应的成员。                                                                                    |
| `amount`               | string 或 null         | `period` 的有效限额，以最小单位表示。`null` 表示无限制；`"0"` 表示使用额度已禁用。                                         |
| `currency`             | string                | ISO 4217。                                                                                    |
| `period`               | string                | `source` 解析到的支出限额的期间。请参阅 "Period" 部分。                                                        |
| `source`               | 范围                    | 其中 `amount` 在层级中的解析位置。当成员具有按用户覆盖时等于 `scope`。                                                 |
| `spend_limit_id`       | 字符串                   | `source` 解析到的 `SpendLimit` 的 ID。使用 `GET /v1/organizations/spend_limits/{spend_limit_id}` 获取。 |
| `period_to_date_spend` | 字符串                   | 成员自当前 `period` 开始以来累积的使用额度，以最小单位计。                                                           |

## SpendLimitIncreaseRequest 对象

```
{
  "type": "spend_limit_increase_request",
  "id": "slir_01AbCdEfGhIjKlMnOpQrSt",
  "created_at": "2026-05-04T16:22:09Z",
  "status": "pending",
  "resolved_at": null,
  "resolved_by": null,
  "actor": {
    "type": "user_actor",
    "user_id": "user_01AbCdEfGh",
    "name": "Jane Smith",
    "email_address": "jane@example.com"
  },
  "spend_summary": {
    "scope": { "type": "user", "user_id": "user_01AbCdEfGh" },
    "amount": "50000",
    "currency": "USD",
    "period": "monthly",
    "source": { "type": "seat_tier", "seat_tier": "enterprise_standard" },
    "spend_limit_id": "spl_01XyZaBcDeFgHiJkLmNoPq",
    "period_to_date_spend": "48900"
  }
}
```

| **字段**          | **类型**                | **描述**                                                                                                                                                                                                                                                                                                                                 |
| --------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`          | 字符串                   | 始终为 `"spend_limit_increase_request"`。                                                                                                                                                                                                                                                                                                  |
| `id`            | 字符串                   | 前缀为 `slir_`。                                                                                                                                                                                                                                                                                                                           |
| `created_at`    | 字符串 (RFC 3339)        | 成员提交请求的时间。                                                                                                                                                                                                                                                                                                                             |
| `status`        | 字符串                   | `pending`、`approved` 或 `denied`。                                                                                                                                                                                                                                                                                                       |
| `resolved_at`   | 字符串 (RFC 3339) 或 null | 请求被批准或拒绝的时间。待处理时为 `null`。                                                                                                                                                                                                                                                                                                              |
| `resolved_by`   | Actor 或 null          | 谁批准或拒绝了请求：要么是 `user_actor`（管理员在 claude.ai 中操作），要么是 `scoped_api_key_actor`（通过此 API 解析）。当请求由管理员在 claude.ai 中的操作自动解决时（例如，提高座位层级限制、为组织启用使用额度计费或提高成员的限制），`resolved_by` 是操作管理员的 `user_actor`。待处理时为 `null`，当解决请求的管理员账户已被删除时为 `null`，或当请求由 Anthropic 支持解决时为 `null`。`scoped_api_key_actor` 可能引用已被删除或撤销的密钥。将 `scoped_api_key_id` 视为历史参考，并容忍查询失败。 |
| `actor`         | Actor (`user_actor`)  | 提交请求的成员。                                                                                                                                                                                                                                                                                                                               |
| `spend_summary` | SpendSummary 或 null   | 请求者的实时决策上下文：其有效限制和期间至今的支出。当 `status` 为待处理时存在（如果无法计算摘要，可能为 `null`）；解决后始终为 `null`。                                                                                                                                                                                                                                                       |

### Actor

| **字段**              | **类型**    | **描述**                                               |
| ------------------- | --------- | ---------------------------------------------------- |
| `type`              | 字符串       | `user_actor` 或 `scoped_api_key_actor`。               |
| `user_id`           | 字符串       | 在 `user_actor` 上存在。用户的 ID；与 `actor_ids[]` 接受的值相同。    |
| `name`              | 字符串或 null | 在 `user_actor` 上存在。用户的名称；如果账户已被删除或用户未设置名称，则为 `null`。 |
| `email_address`     | 字符串或 null | 出现在 `user_actor` 上。用户的电子邮件；如果账户已被删除，则为 `null`。       |
| `scoped_api_key_id` | 字符串       | 出现在 `scoped_api_key_actor` 上。前缀为 `apikey_`。          |

---

## 支出限额

### 1. 列出有效的支出限额

```
GET /v1/organizations/spend_limits/effective
```

返回组织的**每个当前成员**及其已解析的有效限额和期间至今的支出。没有按用户覆盖的成员显示 `source.type` 为 `seat_tier`、`rbac_group` 或 `organization`。前成员不在列表中。

需要范围：`read:spend_limits.`

#### 查询参数

| **字段**       | **类型**         | **必需** | **默认值** | **描述**                                             |
| ------------ | -------------- | ------ | ------- | -------------------------------------------------- |
| `user_ids[]` | 字符串，最多 100 个条目 | 否      | 所有成员    | 缩小到特定成员。接受 `user_...` ID。不是当前成员的条目会从 `data` 中静默省略。 |
| `limit`      | 整数 1–1000      | 否      | 20      | 每页行数。                                              |
| `page`       | 不透明游标字符串       | 否      | —       | 来自上一个响应的 `next_page` 值。                            |

#### 响应字段

| **字段**      | **类型**          | **描述**                        |
| ----------- | --------------- | ----------------------------- |
| `data`      | SpendSummary 数组 | 每个成员一个条目，按成员加入组织的时间排序（最新的在前）。 |
| `next_page` | 字符串或 null       | 下一页的不透明游标；当没有更多页面时为 `null`。   |

#### 示例请求

```
curl "https://api.anthropic.com/v1/organizations/spend_limits/effective?limit=20" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY"
```

#### 示例响应

```
{
  "data": [
    {
      "scope": { "type": "user", "user_id": "user_01AbCdEfGh" },
      "amount": "50000",
      "currency": "USD",
      "period": "monthly",
      "source": { "type": "seat_tier", "seat_tier": "enterprise_standard" },
      "spend_limit_id": "spl_01XyZaBcDeFgHiJkLmNoPq",
      "period_to_date_spend": "31402.5"
    }
  ],
  "next_page": "page_..."
}
```

#### 验证

| **条件**                        | **状态** | **消息**                                     |
| ----------------------------- | ------ | ------------------------------------------ |
| `user_ids[]` 条目格式错误           | 400    | `user_ids[]: entry is not a valid user ID` |
| `user_ids[]` 有超过 100 个条目      | 400    |                                            |
| `limit` 超出 1–1000 范围          | 400    |                                            |
| `page` 游标无效                   | 400    | `page: 游标无效`                               |
| `page` 游标与当前 `user_ids[]` 不匹配 | 400    | `page: 游标与当前查询参数不匹配`                       |
| `page` 游标来自不同的 API 版本         | 400    | `page: 游标由不同的 API 版本颁发`                    |
| 组织未采用企业计划                     | 400    | `此端点不支持此组织类型`                              |
| 使用额度计费未启用                     | 400    | `此组织未启用超额计费`                               |

---

### 2. 获取支出限额

```
GET /v1/organizations/spend_limits/{spend_limit_id}
```

按 ID 返回单个支出限额。使用此功能检查 `SpendSummary.spend_limit_id` 或 `POST` 响应引用的行。

需要范围：`read:spend_limits`。

#### 路径参数

| **字段**           | **类型** | **描述**      |
| ---------------- | ------ | ----------- |
| `spend_limit_id` | 字符串    | 前缀为 `spl_`。 |

#### 响应

SpendLimit 对象。

#### 示例请求

```
curl "https://api.anthropic.com/v1/organizations/spend_limits/spl_01AbCdEfGhIjKlMnOpQrSt" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY"
```

#### 验证

| **条件**                    | **状态** | **消息**        |
| ------------------------- | ------ | ------------- |
| `spend_limit_id` 在此组织中未找到 | 404    |               |
| 组织未采用企业计划                 | 400    | `此端点不支持此组织类型` |
| 使用额度计费未启用                 | 400    | `此组织未启用超额计费`  |

---

### 3. 设置支出限额

```
POST /v1/organizations/spend_limits
```

设置**按用户**支出限额覆盖。更新插入：为已有限额的用户设置限额会就地覆盖。

仅接受 `scope.type: "user"`。座位层级、群组和组织级别的默认值在 claude.ai 设置中配置。

直接设置支出限额*不会*转换成员的待处理增加请求。使用批准端点来解决请求。

需要范围：`write:spend_limits`。

#### 请求正文

| **字段**   | **类型** | **必需** | **描述**                                                              |
| -------- | ------ | ------ | ------------------------------------------------------------------- |
| `scope`  | 对象     | 是      | `{ "type": "user", "user_id": "user_..." }`。                        |
| `amount` | 字符串或空值 | 是      | `period` 的新限额，以小单位表示，为非负整数十进制字符串。`"0"` 禁用该成员的使用额度。`null` 移除限额（无限制）。 |
| `period` | 字符串    | 否      | 默认值 `"monthly"`。请参阅"周期"部分。                                          |

#### 响应

反映已写入覆盖的 SpendLimit 对象。

#### 示例请求

```
curl -X POST "https://api.anthropic.com/v1/organizations/spend_limits" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY" \
  -H "content-type: application/json" \
  -d '{"scope": {"type": "user", "user_id": "user_01AbCdEfGh"}, "amount": "75000"}'
```

#### 示例响应

```
{
  "type": "spend_limit",
  "id": "spl_01RsTuVwXyZaBcDeFgHiJk",
  "created_at": "2026-05-11T10:02:44Z",
  "updated_at": "2026-05-11T10:02:44Z",
  "scope": { "type": "user", "user_id": "user_01AbCdEfGh" },
  "amount": "75000",
  "currency": "USD",
  "period": "monthly"
}
```

#### 验证

| **条件**                    | **状态** | **消息**                                                      |
| ------------------------- | ------ | ----------------------------------------------------------- |
| `scope.type` 不是 `"user"`  | 400    | `scope.type: not yet supported`                             |
| `scope.user_id` 格式错误      | 400    | `scope.user_id: malformed`                                  |
| `scope.user_id` 不是此组织的成员  | 400    | `scope.user_id: not a member of this organization`          |
| `amount` 为负数、分数或无效的十进制字符串 | 400    |                                                             |
| `period` 不是 `"monthly"`   | 400    | `period: not yet supported`                                 |
| 组织不在企业计划中                 | 400    | `this endpoint is not supported for this organization type` |
| 未启用使用额度计费                 | 400    | `overage billing is not enabled for this organization`      |

---

### 4. 移除支出限额

```
DELETE /v1/organizations/spend_limits/{spend_limit_id}
```

移除**按用户**覆盖，使成员回退到继承的限额（座位层级、组或组织默认值）。座位层级、组和组织级别的行无法通过此端点删除。

需要范围：`write:spend_limits`。

#### 路径参数

| **字段**           | **类型** | **描述**                   |
| ---------------- | ------ | ------------------------ |
| `spend_limit_id` | 字符串    | 前缀为 `spl_`。必须是每用户覆盖的 ID。 |

#### 响应

```
{ "type": "spend_limit_deleted", "id": "spl_01RsTuVwXyZaBcDeFgHiJk" }
```

#### 示例请求

```
curl -X DELETE "https://api.anthropic.com/v1/organizations/spend_limits/spl_01RsTuVwXyZaBcDeFgHiJk" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY"
```

#### 验证

| **条件**                       | **状态** | **消息**                |
| ---------------------------- | ------ | --------------------- |
| 在此组织中找不到 `spend_limit_id`    | 404    |                       |
| `spend_limit_id` 是座位层级、组或组织行 | 400    | `只有每用户支出限额可以通过此端点删除。` |
| 组织不在企业计划上                    | 400    | `此端点不支持此组织类型`         |
| 未启用使用额度计费                    | 400    | `此组织未启用超额计费`          |

---

## 支出限额增加请求

### 5. 列出增加请求

```
GET /v1/organizations/spend_limit_increase_requests
```

列出增加请求，最近的优先。排除请求者不再是组织成员的请求。

需要范围：`read:spend_limits`。

#### 查询参数

| **字段**        | **类型**                                | **必需** | **默认值** | **描述**                   |
| ------------- | ------------------------------------- | ------ | ------- | ------------------------ |
| `status[]`    | `pending`、`approved`、`denied` 中的一个或多个 | 否      | 全部      | 按状态筛选。重复该参数以获取多个值。       |
| `actor_ids[]` | 字符串                                   | 否      | 全部      | 按请求者筛选。接受 `user_...` ID。 |
| `limit`       | 整数 1–1000                             | 否      | 20      | 每页行数。                    |
| `page`        | 不透明游标字符串                              | 否      | —       | 来自上一个响应的 `next_page` 值。  |

#### 响应字段

| **字段**      | **类型**                       | **描述**                      |
| ----------- | ---------------------------- | --------------------------- |
| `data`      | SpendLimitIncreaseRequest 数组 | 按 `created_at` 降序排列。        |
| `next_page` | 字符串或 null                    | 下一页的不透明游标；当没有更多页面时为 `null`。 |

#### 示例请求

```
curl "https://api.anthropic.com/v1/organizations/spend_limit_increase_requests?status[]=pending&limit=50" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY"
```

#### 示例响应

```
{
  "data": [
    {
      "type": "spend_limit_increase_request",
      "id": "slir_01AbCdEfGhIjKlMnOpQrSt",
      "created_at": "2026-05-04T16:22:09Z",
      "status": "pending",
      "resolved_at": null,
      "resolved_by": null,
      "actor": {
        "type": "user_actor",
        "user_id": "user_01AbCdEfGh",
        "name": "Jane Smith",
        "email_address": "jane@example.com"
      },
      "spend_summary": {
        "scope": { "type": "user", "user_id": "user_01AbCdEfGh" },
        "amount": "50000",
        "currency": "USD",
        "period": "monthly",
        "source": { "type": "seat_tier", "seat_tier": "enterprise_standard" },
        "spend_limit_id": "spl_01XyZaBcDeFgHiJkLmNoPq",
        "period_to_date_spend": "48900"
      }
    }
  ],
  "next_page": null
}
```

#### 验证

| **条件**                                      | **状态** | **消息**                                                                  |
| ------------------------------------------- | ------ | ----------------------------------------------------------------------- |
| `actor_ids[]` 条目格式错误                        | 400    | `actor_ids[]: invalid tagged user ID`                                   |
| `limit` 超出 1–1000 范围                        | 400    |                                                                         |
| `page` 游标格式错误                               | 400    | `invalid page cursor format` 或 `invalid page cursor`                    |
| `page` 游标与当前 `status[]` 或 `actor_ids[]` 不匹配 | 400    | `page cursor does not match current query parameters`                   |
| `page` 游标来自不同的 API 版本                       | 400    | `page cursor was issued by a different API version; restart pagination` |
| 组织不在企业计划中                                   | 400    | `this endpoint is not supported for this organization type`             |
| 未启用使用额度计费                                   | 400    | `overage billing is not enabled for this organization`                  |

---

### 6. 获取增加请求

```
GET /v1/organizations/spend_limit_increase_requests/{spend_limit_increase_request_id}
```

返回单个增加请求。

需要范围：`read:spend_limits`。

#### 路径参数

| **字段**                            | **类型** | **描述**       |
| --------------------------------- | ------ | ------------ |
| `spend_limit_increase_request_id` | 字符串    | 前缀为 `slir_`。 |

#### 响应

SpendLimitIncreaseRequest 对象。

#### 示例请求

```
curl "https://api.anthropic.com/v1/organizations/spend_limit_increase_requests/slir_01AbCdEfGhIjKlMnOpQrSt" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY"
```

#### 验证

| **条件**       | **状态** | **消息**        |
| ------------ | ------ | ------------- |
| 在此组织中找不到请求   | 404    |               |
| 请求者不再是此组织的成员 | 404    |               |
| 组织未采用企业计划    | 400    | `此组织类型不支持此端点` |
| 未启用使用额度计费    | 400    | `此组织未启用超额计费`  |

---

### 7. 批准增加请求

```
POST /v1/organizations/spend_limit_increase_requests/{spend_limit_increase_request_id}/approve
```

批准待处理请求。在 `amount` 处为请求者写入**每用户支出限额**，并将请求转换为 `approved`。请求不包含请求的金额，管理员在批准时提供新的限额。

需要范围：`write:spend_limits`。

#### 路径参数

| **字段**                            | **类型** | **描述**       |
| --------------------------------- | ------ | ------------ |
| `spend_limit_increase_request_id` | 字符串    | 前缀为 `slir_`。 |

#### 请求体

| **字段**                  | **类型** | **必需** | **默认值**     | **描述**                                     |
| ----------------------- | ------ | ------ | ----------- | ------------------------------------------ |
| `amount`                | 字符串    | 是      | —           | `period` 的新的每用户限额，以小单位表示，为非负整数十进制字符串。      |
| `period`                | 字符串    | 否      | `"monthly"` | 请参阅"周期"部分。                                 |
| `suppress_notification` | 布尔值    | 否      | `false`     | 如果为 `true`，Anthropic 不会向成员发送电子邮件通知其请求已被批准。 |

#### 响应

状态为 `approved` 的 SpendLimitIncreaseRequest，包含一个额外的 `spend_limit` 字段，其中包含已写入的 SpendLimit。

#### 示例请求

```
curl -X POST "https://api.anthropic.com/v1/organizations/spend_limit_increase_requests/slir_01AbCdEfGhIjKlMnOpQrSt/approve" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY" \
  -H "content-type: application/json" \
  -d '{"amount": "75000", "suppress_notification": true}'
```

#### 示例响应

```
{
  "type": "spend_limit_increase_request",
  "id": "slir_01AbCdEfGhIjKlMnOpQrSt",
  "created_at": "2026-05-04T16:22:09Z",
  "status": "approved",
  "resolved_at": "2026-05-11T10:05:02Z",
  "resolved_by": {
    "type": "scoped_api_key_actor",
    "scoped_api_key_id": "apikey_01ZyXwVuTsRqPoNmLkJiHg"
  },
  "actor": {
    "type": "user_actor",
    "user_id": "user_01AbCdEfGh",
    "name": "Jane Smith",
    "email_address": "jane@example.com"
  },
  "spend_summary": null,
  "spend_limit": {
    "type": "spend_limit",
    "id": "spl_01RsTuVwXyZaBcDeFgHiJk",
    "created_at": "2026-05-11T10:05:02Z",
    "updated_at": "2026-05-11T10:05:02Z",
    "scope": { "type": "user", "user_id": "user_01AbCdEfGh" },
    "amount": "75000",
    "currency": "USD",
    "period": "monthly"
  }
}
```

#### 验证

| **条件**               | **状态** | **消息**        |
| -------------------- | ------ | ------------- |
| 在此组织中找不到请求           | 404    |               |
| 请求者不再是此组织的成员         | 404    |               |
| 请求已`批准`或`拒绝`         | 400    | `支出限额增加请求已解决` |
| `金额`为负数、分数或无效的十进制字符串 | 400    |               |
| `期间`不是`"monthly"`    | 400    | `期间：尚不支持`     |
| 组织不在企业计划中            | 400    | `此端点不支持此组织类型` |
| 未启用使用额度计费            | 400    | `此组织未启用超额计费`  |

---

### 8. 拒绝增加请求

```
POST /v1/organizations/spend_limit_increase_requests/{spend_limit_increase_request_id}/deny
```

拒绝待处理请求。在`拒绝`时幂等：拒绝已拒绝的请求返回 200 和现有资源。拒绝已批准的请求被拒绝，以便自动化可以区分重试和冲突决定。

需要范围：`write:spend_limits`。

#### 路径参数

| **字段**                            | **类型** | **描述**      |
| --------------------------------- | ------ | ----------- |
| `spend_limit_increase_request_id` | 字符串    | 前缀为`slir_`。 |

#### 请求正文

| **字段**                  | **类型** | **必需** | **默认值** | **描述**                                   |
| ----------------------- | ------ | ------ | ------- | ---------------------------------------- |
| `suppress_notification` | 布尔值    | 否      | `false` | 如果`true`，Anthropic 不会向成员发送电子邮件，告知其请求被拒绝。 |

#### 响应

状态为拒绝的 SpendLimitIncreaseRequest。

#### 示例请求

```
curl -X POST "https://api.anthropic.com/v1/organizations/spend_limit_increase_requests/slir_01AbCdEfGhIjKlMnOpQrSt/deny" \
  -H "x-api-key: $ANTHROPIC_ADMIN_KEY" \
  -H "content-type: application/json" \
  -d '{"suppress_notification": true}'
```

#### 验证

| **条件**       | **状态**              | **消息**         |
| ------------ | ------------------- | -------------- |
| 在此组织中找不到请求   | 404                 |                |
| 请求者不再是此组织的成员 | 404                 |                |
| 请求已`批准`      | 400                 | `支出限额增加请求已被批准` |
| 请求已`拒绝`      | — (200, idempotent) |                |
| 组织未采用企业计划    | 400                 | `此组织类型不支持此端点`  |
| 未启用使用额度      | 400                 | `此组织未启用超额计费`   |