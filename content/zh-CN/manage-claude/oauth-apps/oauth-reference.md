---
title: OAuth 参考
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference
description: 端点、客户端身份验证、请求参数、范围、令牌响应、撤销、有效期、重定向 URI 规则和错误。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：它是根据您与 Anthropic 之间的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在 Beta 期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

此页面汇总了 [Claude Platform OAuth](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/overview) 的端点、参数、验证规则和错误映射。有关操作演练，请参阅 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections)。

## 端点

| 端点 | URL |
| ----- | ----- |
| 授权端点（登录和同意页面） | `https://auth.claude.com/oauth/authorize` |
| “Pushed authorization request”（推送授权请求），即 PAR 端点 | `https://auth.claude.com/v1/oauth/par` |
| 令牌端点 | `https://auth.claude.com/v1/oauth/token` |
| 撤销端点 | `https://auth.claude.com/v1/oauth/revoke` |
| Claude API（用户配置文件以及向客户计费的请求） | `https://api.anthropic.com` |

## 客户端身份验证

`auth.claude.com` 使用 `private_key_jwt` 方法（[RFC 7523](https://www.rfc-editor.org/rfc/rfc7523)）对您的应用进行身份验证。请在每个发往 PAR 端点、令牌端点和撤销端点的请求中发送这些参数。切勿在 URL 查询字符串中发送凭据：`auth.claude.com` 会拒绝它们。

| 参数 | 值 |
| ----- | ----- |
| `client_id` | 您的 OAuth 应用的客户端 ID |
| `client_assertion_type` | `urn:ietf:params:oauth:client-assertion-type:jwt-bearer` |
| `client_assertion` | 来自您的身份提供商的 JWT，包含[客户端断言声明](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#client-assertion-claims)中的声明 |

### 客户端断言声明

客户端断言必须包含以下所有声明，并且其标头必须包含您 JWKS 中某个密钥的 `kid`。

| 声明 | 要求 |
| ----- | ----- |
| `iss` | 您的身份提供商的颁发者 URL，必须与通过[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)注册的值完全一致 |
| `sub` | 如果您在应用的客户端身份验证设置中设置了主体模式，则为与该模式匹配的值 |
| `aud` | 恰好一个受众：即您应用的客户端身份验证设置中的值。`auth.claude.com` 会拒绝列出多个受众的断言。 |
| `iat` | 您的身份提供商签发 JWT 的时间 |
| `exp` | 过期时间，不得晚于 `iat` 之后 1 小时。请尽可能使用更短的有效期：任何截获断言的人都可以在其过期前使用它。 |

断言负载示例：

```json
{
  "iss": "https://idp.example.com",
  "sub": "workload:prod:claude-integration-backend",
  "aud": "YOUR_CONFIGURED_AUDIENCE",
  "iat": 1791230000,
  "exp": 1791230300
}
```

请在您的后端允许的范围内，尽可能收窄主体模式和声明条件。`auth.claude.com` 会接受来自您的身份提供商、且与这些条件匹配的任何 JWT，作为您应用的身份验证。

## 授权请求参数

以下所有参数均为必需。请在推送授权请求的 JSON 正文中发送除 `request_uri` 以外的所有参数。

| 参数 | 描述 |
| ----- | ----- |
| `response_type` | 值 `code` |
| `client_id` | 您的 OAuth 应用的客户端 ID |
| `redirect_uri` | 与已注册的重定向 URI 完全匹配的 URI，最多 2048 个字符 |
| `scope` | `billing:attribution` 或 `billing:attribution offline_access`。请参阅[计费范围](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#scopes)。 |
| `state` | 一个随机的一次性值，最多 2048 个字符，您需要在回调时检查它 |
| `code_challenge` | `BASE64URL(SHA256(code_verifier))`，其中 `code_verifier` 为 43 到 128 个字符 |
| `code_challenge_method` | 值 `S256` |
| `connection_type` | 值 `organization_shared`。由管理员代表整个组织批准该连接。 |
| `connection_auth_target` | 值 `platform`。连接的请求会路由到 OAuth 应用的所有者。 |
| `user_profile_id` | 您组织中代表该客户的有效用户配置文件的 ID |
| `request_uri` | PAR 端点返回的值。将其与 `client_id` 一起发送到授权端点，且不附带其他参数。 |

PAR 端点返回 HTTP 201，包含 `request_uri` 和 `expires_in`（600 秒）。

## 范围

请在 `scope` 参数中请求这些范围。

| 范围 | 描述 |
| ----- | ----- |
| `offline_access` | 为应用提供一个代表其他所请求范围的刷新令牌。在计费连接上，仅可与 `billing:attribution` 一起发送。 |
| `billing:attribution` | 一种连接，可将客户指示您发出的 Claude API 请求计费到该客户的 Claude 组织。可单独发送，或与 `offline_access` 一起发送以同时获取刷新令牌。请参阅[计费连接的 OAuth 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#oauth-tokens-for-a-billing-connection)。 |
| `org:billing_attribution` | 已弃用：请使用 `billing:attribution`。 |

## 令牌请求

使用 JSON 正文将令牌请求发送到 `https://auth.claude.com/v1/oauth/token`。

| 参数 | 值 |
| ----- | ----- |
| `grant_type` | `authorization_code` |
| `code` | 回调中的授权码 |
| `redirect_uri` | 与您在授权请求中发送的相同的 `redirect_uri` |
| `code_verifier` | PKCE 代码验证器 |
| `client_id`, `client_assertion_type`, `client_assertion` | 如[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#client-authentication)中所述 |

要刷新，请发送设置为 `refresh_token` 的 `grant_type`、您的 `refresh_token`，以及[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#client-authentication)中的参数。

## 计费连接的 OAuth 令牌

令牌端点会返回该连接的访问令牌。该令牌以您应用的[运营方服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#terms)身份进行身份验证，作用于应用的 `connection_token_workspace_id` 设置所指定的工作区。在 `/v1/messages` 和 `/v1/chat/completions` 上，使用该访问令牌的请求会计费到该连接的客户，而不会计费到其他任何人。当客户还批准了 `offline_access` 时，令牌端点也会返回刷新令牌，`refresh_token` 授权会同时续订这两个令牌。有关设置和操作演练，请参阅 Linked billing 页面上的[每个连接的 OAuth 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#oauth-tokens-for-each-connection)。

两种授权都使用[令牌请求](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#token-request-and-response)和[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#client-authentication)中的参数。刷新请求不得发送 `scope` 参数。令牌响应包含以下字段：

| 响应字段 | 值 |
| ----- | ----- |
| `access_token` | 连接的访问令牌 |
| `token_type` | `Bearer` |
| `expires_in` | 访问令牌过期前的秒数，最多 `3600` |
| `scope` | 始终为 `workspace:inference` |
| `connection_id` | 连接 ID，例如 `ocon_EXAMPLE`。请存储它。仅 `authorization_code` 授权会返回它。 |
| `refresh_token` | 连接的刷新令牌。仅当客户批准了 `offline_access` 时返回。 |
| `refresh_token_expires_in` | `refresh_token` 过期前的秒数。没有 `refresh_token` 时不存在。 |

每次刷新都会返回新的 `access_token` 和新的 `refresh_token`。您发送的刷新令牌会被替换，先前的访问令牌会在几分钟内失效。您可以在替换它的那次刷新之后 30 秒内，再发送一次已被替换的刷新令牌，以重复一个丢失了响应的请求。该第二次请求会返回另一对新令牌，并撤销丢失的响应中的那对令牌：请仅使用最新的一对。使用已被替换令牌的第三次请求会收到 `invalid_grant`，在 30 秒之后到达的第二次请求也是如此。在这 30 秒内，此类请求不会撤销任何内容。超过 30 秒后，它会撤销该连接的所有令牌并终止该连接。[刷新令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#refresh-the-connection)介绍了刷新的规则和错误。

以下每个事件都会撤销该连接的令牌，包括当前的访问令牌：

* 客户或您解除关联。
* 客户针对同一用户配置文件批准了一个范围不同的新授权请求，且您兑换了该批准的授权码。新连接将取代先前的连接。
* 令牌所运行的工作区被归档。

有关 Claude API 如何对使用连接访问令牌的请求进行计费，请参阅[使用连接的访问令牌调用 Claude](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#call-claude-with-a-connections-access-token)。

## 撤销

使用 JSON 或表单编码的正文将撤销请求发送到 `https://auth.claude.com/v1/oauth/revoke`。该端点遵循 [RFC 7009](https://www.rfc-editor.org/rfc/rfc7009)，并且还接受连接 ID 和用户配置文件 ID。

请发送[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#client-authentication)中的参数，以及此表中恰好一个参数。客户端断言的 `aud` 是您应用的客户端身份验证设置中的值，而不是撤销 URL。

| 参数 | 终止的内容 |
| ----- | ----- |
| `connection_id` | 您应用的一个连接，以 `ocon_` ID 形式提供 |
| `user_profile_id` | 您应用在某个用户配置文件上的所有连接，以 `uprof_` ID 形式提供 |
| `token` | 访问令牌或刷新令牌所属的连接，以及该连接的所有令牌 |

通过已弃用的 `org:billing_attribution` 范围批准的连接的访问令牌是一个例外：发送它只会终止该令牌。要终止该连接，请发送 `connection_id` 或 `user_profile_id`。

`auth.claude.com` 接受 `token_type_hint`，但会忽略它。

该端点返回 HTTP 200 以及该请求所终止的连接的 ID：

```json
{
  "revoked_connection_ids": ["ocon_EXAMPLE"]
}
```

在以下任一情况下，列表为空，且响应不会说明是哪一种：

* 令牌或 ID 未知或属于其他应用。
* 连接已终止。
* 请求终止了令牌但未终止任何连接。

任何计费连接终止后，Claude API 都会拒绝向该连接的客户计费的请求。有关响应，请参阅[计费错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#billing-errors)。

`billing:attribution` 连接终止后，其 OAuth 令牌将停止工作。通过已弃用的 `org:billing_attribution` 范围批准的连接终止后，请勿依赖其访问令牌。它可能立即停止工作，也可能一直有效直到过期。

## 有效期

| 项目 | 有效期 |
| ----- | ----- |
| 来自 PAR 端点的 `request_uri` | 10 分钟，一次性使用 |
| 授权码 | 5 分钟，一次性使用 |
| `access_token` | 最多 1 小时 |
| `refresh_token` | 自授权码交换起最多 1 年。刷新不会延长其有效期。 |

请从每个响应中读取 `expires_in` 和 `refresh_token_expires_in`，而不要依赖固定值。

## 重定向 URI 规则

* 每个 `redirect_uri` 都必须与您应用上注册的 URI 完全匹配。`auth.claude.com` 不使用通配符或前缀匹配。
* 每个重定向 URI 都必须使用 HTTPS，且不得包含片段。位于 `localhost`、`127.0.0.1` 或 `[::1]` 上的本地开发 URI 可以使用 HTTP。
* 以星号作为端口注册的本地开发 URI（例如 `http://localhost:*/callback`）可匹配任意端口。
* 每个应用最多可注册 10 个重定向 URI。
* 仅当 `redirect_uri` 与已注册的 URI 匹配时，`auth.claude.com` 才会将错误发送到您的重定向 URI。对于任何其他 `redirect_uri`，`auth.claude.com` 不会重定向浏览器。

## 错误

### 重定向中的错误

`auth.claude.com` 会以 `error`、`error_description` 和 `state` 查询参数的形式在您的重定向 URI 上返回错误。这些是标准的 OAuth 2.0 错误（[RFC 6749](https://www.rfc-editor.org/rfc/rfc6749)），但 `user_profile_already_linked` 除外，它是 Anthropic 特有的。

| 错误 | 何时出现 | 向用户显示的内容 |
| ----- | ----- | ----- |
| `access_denied` | 用户拒绝了请求，或无法批准此请求。该用户必须是某个 Claude 组织的管理员。 | 说明连接未完成，且不要假定用户拒绝了请求。请按照 Linked billing 指南中的[您的重定向 URI 收到 `error=access_denied`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#access-denied) 操作，其中列出了各种原因以及应告知用户的内容。 |
| `server_error` | `auth.claude.com` 处理请求失败。 | 请用户重试。 |
| `user_profile_already_linked` | 客户的组织已通过另一个用户配置文件关联到您的应用。每个客户组织只能与您的应用建立一个连接。`auth.claude.com` 已请管理员与您联系。 | 说明其组织已关联到您的应用。 |

如果 `request_uri` 已过期或您已使用过它，`auth.claude.com` 会显示错误页面，且不会将浏览器重定向回您的应用。请使用新的推送授权请求重新开始流程。

### 来自 PAR 端点和令牌端点的错误

这些端点返回包含 `error` 和 `error_description` 的 JSON 正文。

| 错误 | 何时出现 |
| ----- | ----- |
| `invalid_request` | 缺少必需参数或参数无效，例如缺少 `state` 或 PKCE 质询，或者 `user_profile_id` 不是您组织中有效用户配置文件的 ID。 |
| `invalid_client` | `auth.claude.com` 无法对您的应用进行身份验证：客户端 ID 未知，或客户端断言验证失败。请根据您应用的客户端身份验证设置检查断言的 `iss`、`sub` 和 `aud`，并确认您的身份提供商使用了您 JWKS 中的密钥对其进行签名。 |
| `invalid_scope` | 所请求的范围不适用于此应用。如果刷新请求发送了 `scope` 参数，也会收到此错误。如果 `billing:attribution` 已不再为您的应用注册，任何令牌请求都会收到此错误。在授权码交换时，缺少 `billing:attribution` 会消耗掉该授权码。 |
| `invalid_grant` | 授权码已过期、您已使用过它，或它与 `redirect_uri` 或 `code_verifier` 不匹配。对于刷新，刷新令牌已过期或不再有效：请按照[刷新令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#refresh-the-connection)操作。 |
| `unauthorized_client` | 您的应用未设置为处理此请求。请先检查应用的运营方服务账户及其工作区：请参阅[代码兑换返回 `unauthorized_client`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#code-exchange-unauthorized-client)。`error_description` 可能会给出原因，例如缺少工作区。此拒绝不会消耗授权码，也不会撤销刷新令牌。 |
| `user_profile_already_linked` | 来自令牌端点。该用户配置文件已通过另一个客户组织关联到您的应用，或客户的组织已通过另一个用户配置文件关联到您的应用。该关联是在批准与此请求之间建立的。 |

如果 `auth.claude.com` 对请求进行了速率限制或发生故障，它会以不同的格式返回错误，该格式可能不是 JSON。对于未知的 `grant_type` 或超出长度限制的参数，它也会这样做。仅在 HTTP 429 和 5xx 响应时采用退避策略重试。将任何其他状态视为请求本身的问题。对于刷新，请按照[刷新令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#refresh-the-connection)操作，而不是采用通用的退避策略。如果响应包含 `retry-after` 标头，请至少等待相应时长。请从您自己的代码发送下一个请求。不要依赖 HTTP 客户端库的自动重试：对于此类响应，它们可能不会运行。[刷新令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#refresh-the-connection)还说明了何时可以安全重试。再次发送已使用过的刷新令牌可能会终止连接。

### 来自撤销端点的错误

对于 `auth.claude.com` 拒绝的大多数请求，撤销端点会返回 HTTP 400 以及包含 `error` 和 `error_description` 的 JSON 正文。

| 错误 | 何时出现 |
| ----- | ----- |
| `invalid_request` | 缺少必需参数或参数无效，例如缺少 `client_id`。请求未发送 `token`、`connection_id` 和 `user_profile_id` 中的任何一个，或发送了其中多个。某个 ID 格式错误，例如前缀错误。凭据或 `token` 位于 URL 查询字符串中。该用户配置文件的连接数量过多，无法在一个请求中处理：该请求可能已终止其中部分连接，请使用 `connection_id` 终止其余连接。 |
| `invalid_client` | `auth.claude.com` 无法对您的应用进行身份验证。有关需要检查的内容，请参阅[来自 PAR 端点和令牌端点的错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#errors-from-the-par-and-token-endpoints)中的 `invalid_client`。 |
| `unauthorized_client` | 您的应用未设置为处理此请求。请联系您的 Anthropic 客户团队。 |

下表中的响应使用不同的格式，该格式可能不是 JSON。当正文为 JSON 时，`error` 是一个对象，而不是字符串。

| 状态 | 何时出现 | 处理方式 |
| ----- | ----- | ----- |
| 400 | 某个参数超出其长度限制。 | 修正请求。不要重试。 |
| 429 | 来自您 IP 地址的失败请求过多。 | 采用退避策略重试。 |
| 500 | `auth.claude.com` 处理请求失败。 | 采用退避策略重试。如果问题持续出现，请联系您的 Anthropic 客户团队。 |
| 501 | Anthropic 尚未为您的组织启用此端点。 | 不要重试。请联系您的 Anthropic 客户团队。 |
| 503 | 请求未完成。已终止的内容保持终止状态。 | 采用退避策略重试，使用相同的参数和新的客户端断言。重试会完成剩余部分，其列表会省略先前请求已终止的连接。如果您发送的是 `user_profile_id`，请在一天内重试：超过一天后，重试可能返回 200 和空列表，且不会完成任何操作。使用每个 `connection_id` 进行重试则随时有效。 |

### 计费错误

这些错误适用于将 `billing_policy.bill_to` 设置为 `"user_profile"`、或使用[计费连接的访问令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#oauth-tokens-for-a-billing-connection)进行身份验证的 Claude API 请求。对于被拒绝的请求，Claude API 不会向任何一个组织计费。所有 402 和 403 响应均不可重试，但在新连接建立后前 10 秒内出现的 `user_profile_not_billable` 拒绝除外。

| 状态 | 错误类型和代码 | 何时出现 |
| ----- | ----- | ----- |
| 400 | `invalid_request_error` | 请求发送了 `billing_policy` 但未附带 `user-profiles-billing-2026-10-01` beta 标头。或者 `billing_policy` 对象缺少字段或字段无效。使用计费连接的访问令牌时，可以省略 `bill_to`。或者请求在允许的组合之外使用了代码执行。 |
| 400 | `invalid_request_error`, `user_profile_required` | `bill_to` 为 `"user_profile"`，请求未包含用户配置文件 ID，且未使用计费连接的访问令牌进行身份验证。 |
| 400 | `invalid_request_error`，无错误代码 | 请求使用计费连接的访问令牌进行身份验证，并且将 `bill_to` 设置为 `"self"`，或指定了该令牌所限定的用户配置文件以外的其他用户配置文件。这两种拒绝都会在 `error.message` 中说明问题所在。 |
| 401 | `authentication_error` | 请求使用的计费连接访问令牌已过期、已被刷新替换，或其连接已终止。消息会说明令牌是已过期还是已被撤销。请使用当前的访问令牌重新发送请求；如果连接已终止，请重新关联客户。连接终止后，每个使用其访问令牌的新请求都可能收到此 401 或 403 `user_profile_not_billable`，顺序不固定。 |
| 402 | `billing_error`, `linked_organization_spend_limit_reached` | 客户组织已达到其设置的支出限制，该限制可能针对您的 OAuth 应用，也可能针对整个组织。当该组织没有可用于支付该请求的额度时，Claude API 也会发送相同的代码。响应不会说明是哪一种。请让客户检查这两项限制及其额度余额。Linked billing 的[预期错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#expected-errors)部分说明了每项的设置位置以及需要更改的内容。 |
| 402 | `billing_error`, `linked_organization_billing_unavailable` | 由于该客户组织自身的 Anthropic 账户存在问题，Anthropic 目前无法向其计费。当该组织的额度耗尽时，相同的代码也会终止 Claude 已开始的回复。请让客户向 Anthropic 检查其计费状态。 |
| 403 | `permission_error`，无错误代码 | Anthropic 不直接向您的组织计费，因此无法代表您向客户计费。 |
| 403 | `permission_error`, `user_profile_not_billable` | 该用户配置文件未与任何客户组织建立连接、管理员或您的后端已终止连接、客户组织未就其 API 用量直接向 Anthropic 付款（预付额度和发票计费均符合条件），或请求未以您应用的服务账户身份进行身份验证。响应不会说明是哪一种。连接终止后，连接的访问令牌也可能收到此错误。[`user_profile_not_billable` 的原因](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#user-profile-not-billable)说明了每种情况需要检查的内容。 |
| 503 | `overloaded_error` | Claude API 暂时无法确认是否能向该客户计费。请重试该请求。如果响应包含 `retry-after` 标头，请等待相应时长后再重试。 |

在 `/v1/messages` 上，错误代码位于 `error.details.error_code` 中。在 `/v1/chat/completions` 上，`error.type` 始终为 `invalid_request_error`，`error.code` 包含 Anthropic 错误类型（例如 `permission_error` 或 `billing_error`），且响应中没有错误代码，因此请根据 HTTP 状态和 `error.code` 进行匹配。

## 标准

| 标准 | 适用范围 |
| ----- | ----- |
| [OAuth 2.0 (RFC 6749)](https://www.rfc-editor.org/rfc/rfc6749) | 授权码流程、令牌端点和错误响应 |
| [PKCE (RFC 7636)](https://www.rfc-editor.org/rfc/rfc7636) | 每个授权请求都必须使用 `S256` |
| [推送授权请求 (RFC 9126)](https://www.rfc-editor.org/rfc/rfc9126) | 每个授权请求都必须使用 |
| [令牌撤销 (RFC 7009)](https://www.rfc-editor.org/rfc/rfc7009) | 撤销端点，它还接受连接 ID 和用户配置文件 ID，并返回其所终止的连接的 ID |
| [JWT 客户端身份验证 (RFC 7523)](https://www.rfc-editor.org/rfc/rfc7523) | PAR 端点、令牌端点和撤销端点上的 `private_key_jwt` 客户端身份验证，以及向客户计费的请求所用的服务账户令牌交换 |

## 这些页面中使用的术语

以下含义仅适用于 Claude Platform OAuth。

| 术语 | 含义 |
| ----- | ----- |
| Claude 组织 | 在 Anthropic 的组织账户。您的公司拥有一个，每个为计费而关联其组织的客户也各拥有一个。 |
| 管理员 | 在 Claude 组织中拥有管理员或账单角色的成员。 |
| OAuth 应用 | 在您的 Claude 组织中的注册项，用于向 `auth.claude.com` 标识您的产品。它包含客户端 ID、名称、徽标、主页 URL 和重定向 URI。 |
| 身份提供商 | 您自有基础设施中的服务，为您的后端签发短期 JWT，并将其公钥发布为 JSON Web Key Set（JWKS），例如您的云提供商的工作负载身份服务或您自己的符合 OIDC 的颁发者。在这些页面中，"您的身份提供商"始终指此服务，而非 Anthropic。 |
| 客户端断言 | 由您的身份提供商签名、由您的后端发送以对您的 OAuth 应用进行身份验证的短期 JWT。这就是 `private_key_jwt` 客户端身份验证方法（RFC 7523）。 |
| 推送授权请求（PAR） | 一种请求，您的后端通过它将授权参数直接发送到 `auth.claude.com`，并接收一个用于浏览器重定向的 `request_uri`（RFC 9126）。 |
| 工作负载身份联合 | Claude Console 中的一项功能，您的组织通过它注册您的身份提供商，使 Anthropic 接受使用您 JWKS 中密钥签名的 JWT。 |
| 用户配置文件 | 您在自己的 Claude 组织中创建的记录，用于代表您的某个客户。计费连接会将用户配置文件关联到该客户的 Claude 组织。 |
| 服务账户 | 您的 Claude 组织中的非人类账户。您的 OAuth 应用与一个服务账户（即其运营方服务账户）相关联。当您的后端发送向客户计费的请求时，会以该服务账户的身份进行身份验证。 |
| 连接的 OAuth 令牌 | `auth.claude.com` 为计费连接返回的访问令牌，以及当客户还批准了 `offline_access` 时返回的刷新令牌。该访问令牌以运营方服务账户身份进行身份验证，且只能向该连接的用户配置文件计费。请参阅[计费连接的 OAuth 令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/oauth-reference#oauth-tokens-for-a-billing-connection)。 |
