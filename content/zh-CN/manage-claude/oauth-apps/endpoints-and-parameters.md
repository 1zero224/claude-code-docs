---
title: OAuth 端点和参数
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters
description: Claude Platform OAuth 的端点、参数、作用域、OAuth 令牌、有效期和错误，按请求使用它们的顺序排列。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：根据您与 Anthropic 之间的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在 Beta 测试期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

本页按请求顺序列出 [Claude Platform OAuth](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/overview) 请求所使用的值，然后列出[这些页面中使用的术语](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#terms)。

## 端点

| 端点 | URL |
| ----- | ----- |
| 授权端点（批准页面） | `https://auth.claude.com/oauth/authorize` |
| 推送授权请求（PAR）端点 | `https://auth.claude.com/v1/oauth/par` |
| 令牌端点 | `https://auth.claude.com/v1/oauth/token` |
| 撤销端点 | `https://auth.claude.com/v1/oauth/revoke` |
| Claude API（[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)以及向客户计费的请求） | `https://api.anthropic.com` |

## 客户端身份验证

`auth.claude.com` 使用 `private_key_jwt` 方法（[RFC 7523](https://www.rfc-editor.org/rfc/rfc7523)）的一种变体来验证您的应用。签发并签署 client assertion（客户端断言）的是您的身份提供商，而不是您的应用。其 `iss` 是您身份提供商的颁发者。其 `aud` 是您应用配置的受众，而不是您的客户端 ID 或端点的完整 URL。`auth.claude.com` 会根据为该颁发者注册的密钥以及您应用的设置对其进行检查。

在每个发往 PAR、令牌和撤销端点的 `POST` 请求正文中发送以下参数。

| 参数 | 值 |
| ----- | ----- |
| `client_id` | 您的 OAuth 应用的客户端 ID，即使使用客户端断言也必须提供 |
| `client_assertion_type` | `urn:ietf:params:oauth:client-assertion-type:jwt-bearer` |
| `client_assertion` | 来自您身份提供商的 JWT，包含[客户端断言声明](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#client-assertion-claims)中的声明 |

以下规则适用于每个请求：

* **正文：**&#x53EF;以是 JSON 或表单编码（`application/x-www-form-urlencoded`）。请相应设置 `Content-Type` 标头。
* **查询字符串：**`auth.claude.com` 会拒绝 URL 查询字符串中的凭据。
* **HTTP Basic：**&#x4E0D;要发送 HTTP Basic `Authorization` 标头。`auth.claude.com` 不会从中读取凭据，并且 PAR 端点可能会拒绝带有该标头的请求。

### 客户端断言声明

客户端断言必须包含以下声明。需要匹配的值位于您应用的[客户端身份验证设置](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#set-up-client-authentication)中。

| 声明 | 要求 |
| ----- | ----- |
| `iss` | 您身份提供商的颁发者 URL，必须与通过 [工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation) 注册的完全一致 |
| `sub` | 您后端的主体。如果您应用的设置中有主体模式，则主体必须与之匹配。 |
| `aud` | 恰好一个受众：即您应用设置中的值，而不是您所调用端点的完整 URL |
| `iat` | 您的身份提供商签发该 JWT 的时间 |
| `exp` | 过期时间，距 `iat` 不得超过您颁发者的[最长有效期](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#jwt-verification)（默认为 1 小时） |

客户端断言的标头和签名还必须满足以下规则：

* **算法：**`RS256`、`RS384`、`RS512`、`ES256`、`ES384`、`ES512`、`PS256`、`PS384` 或 `PS512`，与 [WIF 参考](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#jwt-verification)中的列表相同。`auth.claude.com` 会拒绝任何其他算法，包括 HMAC 算法（`HS256`、`HS384`、`HS512`）和 `none`。
* **密钥 ID：**&#x6807;头的 `kid` 必须指向您 JWKS 中的某个密钥。如果该密钥声明了 `alg`，则它必须是签署该断言所用的算法。
* **您的 JWKS：**&#x6BCF;个签名密钥都必须有 `kid`。缺少 `kid` 的密钥会导致整个密钥集无法使用。

断言的载荷如下所示。其 `aud` 是您应用自己的 **Expected audience**，来自应用的 **Client authentication** 选项卡：

```json
{
  "iss": "https://idp.example.com",
  "sub": "workload:prod:claude-integration-backend",
  "aud": "https://auth.claude.com",
  "iat": 1791230000,
  "exp": 1791230300
}
```

`auth.claude.com` 接受来自您身份提供商的、与您应用的主体模式和声明条件相匹配的任何 JWT。要缩小范围，请参阅[设置客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#set-up-client-authentication)。

## 授权请求

要构建使用这些值的批准流程，请参阅[引导客户进行批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#send-them-to-approve)。

### 授权请求参数

将以下参数以及[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#client-authentication)中的参数发送到 PAR 端点。

| 参数 | 描述 |
| ----- | ----- |
| `response_type` | 值为 `code` |
| `client_id` | 您的 OAuth 应用的客户端 ID |
| `redirect_uri` | 与已注册的某个重定向 URI 完全匹配的 URI，最多 2,048 个字符 |
| `scope` | `billing:attribution` 或 `billing:attribution offline_access`。请参阅[各作用域的权限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#scopes)。 |
| `state` | 一个随机的一次性值，最多 2,048 个字符，您需要在回调时对其进行检查 |
| `code_challenge` | 不带填充的 `BASE64URL(SHA256(code_verifier))`，其中 `code_verifier` 为 43 到 128 个字符 |
| `code_challenge_method` | 值为 `S256` |
| `connection_type` | 值为 `organization_shared`。由管理员为整个组织批准该连接。 |
| `connection_auth_target` | 值为 `platform`。该连接的请求会路由到 OAuth 应用的所有者。 |
| `user_profile_id` | 您组织中代表该客户且未归档的用户配置文件的 ID |

PAR 端点返回 HTTP 201，并附带 `request_uri` 和 `expires_in`。然后，将该用户的浏览器引导至授权端点，并且只带上以下两个参数：

| 参数 | 值 |
| ----- | ----- |
| `client_id` | 您的 OAuth 应用的客户端 ID |
| `request_uri` | PAR 端点返回的值 |

用户批准后，`auth.claude.com` 会将浏览器重定向到您的 `redirect_uri`，并附带以下查询参数：

| 参数 | 值 |
| ----- | ----- |
| `code` | 用于在令牌端点交换的授权码 |
| `state` | 您发送的 `state` 值。请检查其是否匹配。 |
| `iss` | `https://identity.anthropic.com/users`，与端点的主机不同。请检查其是否存在且匹配。 |

### 作用域

| 作用域 | 描述 |
| ----- | ----- |
| `billing:attribution` | 一种连接，允许您就客户指示您发出的 Claude API 请求向该客户的 Claude 组织计费。可以单独发送，也可以与 `offline_access` 一起发送。 |
| `offline_access` | 同时为您的应用提供刷新令牌。在计费连接上，仅与 `billing:attribution` 一起发送。 |
| `org:billing_attribution` | 已弃用：请使用 `billing:attribution`。请参阅[如果您仍在使用已弃用的作用域](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-still-use-the-deprecated-scope)。 |

### 重定向 URI 规则

* 每个 `redirect_uri` 都必须与您应用上注册的某个 URI 完全匹配。不支持通配符或前缀匹配。唯一的例外是以星号作为端口注册的本地开发 URI，例如 `http://localhost:*/callback`，它可匹配任何端口。
* 每个重定向 URI 都必须使用 HTTPS，且不得包含片段。位于 `localhost`、`127.0.0.1` 或 `[::1]` 上的本地开发 URI 可以使用 HTTP。
* 每个应用最多可注册 10 个重定向 URI。
* 仅当 `redirect_uri` 与已注册的 URI 匹配时，`auth.claude.com` 才会将错误发送到您的重定向 URI。对于任何其他 `redirect_uri`，它不会重定向浏览器。

## OAuth 令牌

### 令牌请求和响应

要交换授权码，请将以下参数发送到[令牌端点](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#endpoints)：

| 参数 | 值 |
| ----- | ----- |
| `grant_type` | `authorization_code` |
| `code` | 回调中的授权码 |
| `redirect_uri` | 与您在授权请求中发送的 `redirect_uri` 相同 |
| `code_verifier` | PKCE 代码验证器 |
| `client_id`、`client_assertion_type`、`client_assertion` | 同[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#client-authentication)中所述 |

要刷新，请发送设为 `refresh_token` 的 `grant_type`、`refresh_token` 以及[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#client-authentication)中的参数。每个刷新令牌只能使用一次，并且计费连接有其自身的规则：请参阅下一节以及[如果刷新失败](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#if-a-refresh-fails)。

### 计费连接的 OAuth 令牌

对于 `billing:attribution` 连接，刷新时不得发送 `scope` 参数。两种授权类型都会返回以下字段：

| 响应字段 | 值 |
| ----- | ----- |
| `access_token` | 该连接的访问令牌 |
| `token_type` | `Bearer` |
| `expires_in` | 距访问令牌过期的秒数，最多为 `3600` |
| `scope` | 始终为 `workspace:inference`，而不是您请求的作用域 |
| `connection_id` | 连接 ID，例如 `ocon_01HV9Z`。仅 `authorization_code` 授权类型会返回它。 |
| `refresh_token` | 仅当客户批准了 `offline_access` 时才返回。刷新会返回一个新的刷新令牌，替换您发送的那个。 |
| `refresh_token_expires_in` | 距 `refresh_token` 过期的秒数。没有 `refresh_token` 时不存在。 |

该访问令牌以您应用的[操作员服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#terms)身份进行身份验证，位于应用的[默认工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#default-workspace)中，API 将其称为 `connection_token_workspace_id` 设置。在 `/v1/messages` 和 `/v1/chat/completions` 上，使用该令牌的请求会计费给该连接的客户。它仅适用于[使用客户的访问令牌调用 Claude](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#call-claude-with-a-token) 中列出的端点，其他任何端点都会拒绝它。有关如何使用、刷新和恢复这些令牌，请参阅[为每个客户使用单独的凭据](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer)。

以下每个事件都会撤销 `billing:attribution` 连接的令牌，包括当前的访问令牌：

* 客户或您取消关联。
* 客户针对同一用户配置文件批准了作用域不同的新授权请求，并且您交换了其授权码。新连接会替换先前的连接。
* 令牌所运行的工作区被归档。
* 已被刷新替换的刷新令牌在超过 30 秒后再次被发送。
* 您在[撤销端点](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#revocation)撤销了该连接的某个令牌。

### 有效期

| 项目 | 有效期 |
| ----- | ----- |
| PAR 端点返回的 `request_uri` | 10 分钟，一次性使用 |
| 授权码 | 5 分钟，一次性使用 |
| `access_token` | 最长 1 小时 |
| `refresh_token` | 自授权码交换起最长 1 年。刷新不会延长其有效期。 |

如果响应中提供了 `expires_in` 或 `refresh_token_expires_in`，请以其中的有效期为准，而不是本表。

### 撤销

将撤销请求发送到撤销端点 `https://auth.claude.com/v1/oauth/revoke`。除令牌外，它还接受连接 ID 和用户配置文件 ID。要取消关联客户或重新关联客户，请参阅[管理已关联的客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers)。发送[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#client-authentication)中的参数，以及以下参数中的恰好一个：

| 参数 | 终止的内容 |
| ----- | ----- |
| `connection_id` | 您应用的某个连接，以 `ocon_` ID 的形式提供 |
| `user_profile_id` | 您应用在某个用户配置文件上的所有连接，以 `uprof_` ID 的形式提供 |
| `token` | 访问令牌或刷新令牌所属的连接，以及该连接的所有令牌 |

通过已弃用的 `org:billing_attribution` 作用域批准的连接所产生的访问令牌是一个例外：发送它只会终止该令牌本身。要终止该连接，请发送 `connection_id` 或 `user_profile_id`。

`auth.claude.com` 接受 `token_type_hint` 但会忽略它。

该端点返回 HTTP 200 以及该请求所终止的连接的 ID：

```json
{
  "revoked_connection_ids": ["ocon_01HV9Z"]
}
```

在以下任一情况下，该列表为空，且响应不会说明是哪种情况：

* 令牌或 ID 未知，或属于其他应用。
* 该连接已经终止。
* 该请求终止了一个令牌，但没有终止任何连接。

连接终止后，Claude API 会以 `user_profile_billing_not_authorized` 拒绝向该客户计费的请求，并且该连接的令牌将停止工作。通过已弃用作用域批准的连接所产生的访问令牌则可能在过期前仍然有效。

## 协议错误

### 重定向中的错误

`auth.claude.com` 会在您的重定向 URI 上以 `error`、`error_description`、`state` 和 `iss` 查询参数的形式返回错误。除 `user_profile_already_linked` 外，它们都是标准的 OAuth 2.0 错误（[RFC 6749](https://www.rfc-editor.org/rfc/rfc6749)）。

| 错误 | 发生情况 |
| ----- | ----- |
| `access_denied` | 用户拒绝了该请求，或无法批准该请求。请参阅 [`access_denied`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#access-denied)。 |
| `server_error` | `auth.claude.com` 未能处理该请求。请让用户重试。 |
| `user_profile_already_linked` | 客户的组织已通过其他用户配置文件关联到您的应用，或者该用户配置文件已通过其他组织关联到您的应用。请参阅 [`user_profile_already_linked`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-already-linked)。 |

如果 `request_uri` 已过期或已被使用，`auth.claude.com` 会显示错误页面，且不会重定向浏览器。请使用新的推送授权请求重新开始。

### PAR 和令牌端点的错误

这些端点返回包含 `error` 和 `error_description` 的 JSON 正文。

| 错误 | 发生情况 |
| ----- | ----- |
| `invalid_request` | 缺少必需参数或参数无效，例如缺少 `state`、缺少 PKCE 质询，或者 `user_profile_id` 不是您组织中的有效用户配置文件 |
| `invalid_client` | 客户端 ID 未知，或客户端断言未通过[客户端断言声明](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#client-assertion-claims)中的某项检查。Claude Console 中您应用的 **Client authentication history** 会说明哪项检查失败。 |
| `invalid_scope` | 该作用域不允许用于此应用、计费连接的刷新请求发送了 `scope` 参数，或者 `billing:attribution` 已从您的应用中移除。在最后一种情况下，授权码交换也会消耗掉该授权码。 |
| `invalid_grant` | 在授权码交换时：授权码已过期或已被使用、批准者已退出登录，或者授权码与 `redirect_uri` 或 `code_verifier` 不匹配。不匹配会消耗掉该授权码。 |
| `invalid_grant` | 在授权码交换时：客户在其连接拥有刷新令牌的情况下以相同的作用域再次批准。请参阅[如果客户再次批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#if-a-customer-approves-again)。 |
| `invalid_grant` | 在刷新时：请参阅[如果刷新失败](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#if-a-refresh-fails) |
| `unauthorized_client` | 您的应用未针对此请求进行设置。它不会消耗授权码，也不会撤销刷新令牌。请参阅 [`unauthorized_client`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#code-exchange-unauthorized-client)，如果是刷新，请参阅[如果刷新失败](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#if-a-refresh-fails)。 |
| `user_profile_already_linked` | 来自令牌端点：在批准和交换之间，该配置文件或组织已通过其他方式被关联 |

某些响应采用其他格式，可能不是 JSON。当 `auth.claude.com` 对请求进行速率限制或发生中断时会出现这种情况。对于未知的 `grant_type` 或过长的参数也会出现这种情况。对于不含字符串 `error` 的响应，请根据其状态码进行处理，使用第一个适用的行：

| 状态码 | 处理方式 |
| ----- | ----- |
| 任何状态码，在刷新时 | 按照[如果刷新失败](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#if-a-refresh-fails)处理。 |
| `501` | Anthropic 已关闭该端点。不要重试。 |
| `429` 或 `5xx` | 使用退避策略重试，并且等待时间至少为任何 `retry-after` 标头所指示的时长。请从您自己的代码发送每次重试：HTTP 客户端的自动重试可能不会针对此类响应运行。 |
| 任何其他状态码 | 将其视为请求中的错误。 |

### 撤销端点的错误

对于大多数拒绝，撤销端点返回 HTTP 400 以及包含 `error` 和 `error_description` 的 JSON 正文。

| 错误 | 发生情况 |
| ----- | ----- |
| `invalid_request` | 参数缺失或无效：`token`、`connection_id` 和 `user_profile_id` 均未提供或提供了多个、ID 格式错误，或者 URL 查询字符串中包含凭据 |
| `invalid_request` | 该用户配置文件的连接过多，无法在一个请求中处理。该请求可能已终止了部分连接，因此请使用各个 `connection_id` 终止其余连接。 |
| `invalid_client` | `auth.claude.com` 无法验证您的应用。请对照[客户端断言声明](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#client-assertion-claims)检查您的客户端断言。您应用的 **Client authentication history** 不会记录此端点。 |
| `unauthorized_client` | 您的应用未针对此请求进行设置。请联系 [Anthropic 支持](https://support.claude.com/en/articles/9015913-how-to-get-support)，如果您有 Anthropic 客户团队，也可联系他们。 |

其他拒绝只带有状态码，其正文可能不是 JSON。当此类正文是 JSON 时，其 `error` 是一个对象，而不是字符串：

| 状态码 | 发生情况 | 处理方式 |
| ----- | ----- | ----- |
| `400` | 某个参数超出其长度限制 | 更正请求。不要重试。 |
| `429` | 来自您 IP 地址的失败请求过多 | 使用退避策略重试。 |
| `500` | `auth.claude.com` 未能处理该请求 | 使用退避策略重试。如果问题持续出现，请联系支持人员或您的客户团队。 |
| `501` | Anthropic 已关闭该端点 | 不要重试。请联系支持人员或您的客户团队。 |
| `503` | 请求未完成，但已终止的内容保持终止状态 | 使用退避策略重试。使用相同的参数和新的客户端断言。 |

在 `503` 之后重试会完成其余部分，其列表中不包含先前请求已终止的连接。使用 `connection_id` 时，可随时重试。使用 `user_profile_id` 时，请在一天内重试。超过一天后，重试仍会终止活跃连接，但不会撤销失败请求所终止连接的剩余部分，例如它们的 OAuth 令牌。要撤销这些内容，请发送每个已存储的 `connection_id`。

### 计费错误

关于向客户计费或使用客户访问令牌的请求，Claude API 返回的错误请参阅 [Linked billing 错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#while-billing)。

## 标准

| 标准 | 适用范围 |
| ----- | ----- |
| [OAuth 2.0 (RFC 6749)](https://www.rfc-editor.org/rfc/rfc6749) | 授权码流程、令牌端点和错误响应 |
| [PKCE (RFC 7636)](https://www.rfc-editor.org/rfc/rfc7636) | 每个授权请求都必须使用 `S256` |
| [推送授权请求 (RFC 9126)](https://www.rfc-editor.org/rfc/rfc9126) | Linked billing 必需 |
| [授权服务器签发者标识 (RFC 9207)](https://www.rfc-editor.org/rfc/rfc9207) | 每次重定向到您应用时的 `iss` 参数 |
| [令牌撤销 (RFC 7009)](https://www.rfc-editor.org/rfc/rfc7009) | 撤销端点，它还接受连接 ID 和用户配置文件 ID |
| [JWT 客户端身份验证 (RFC 7523)](https://www.rfc-editor.org/rfc/rfc7523) | PAR、令牌和撤销端点的客户端身份验证 |

## 这些页面中使用的术语

以下含义仅适用于 Claude Platform OAuth。

| 术语 | 含义 |
| ----- | ----- |
| Claude 组织 | 在 Anthropic 的组织账户。您的公司拥有一个，每个为计费而关联其组织的客户也各有一个。 |
| 管理员 | Claude 组织中具有 Admin 或 Billing 角色的成员 |
| OAuth 应用 | 您的 Claude 组织中的注册项，用于向 `auth.claude.com` 标识您的产品。它具有客户端 ID、名称、徽标、主页和重定向 URI。 |
| 身份提供商 | 您自己基础设施中的服务，为您的后端签署短期 JWT，并以 JSON Web Key Set（JWKS）的形式发布其公钥。它绝不是 Anthropic。 |
| 客户端断言 | 由您的身份提供商签署、并由您的后端发送以验证您 OAuth 应用身份的短期 JWT |
| 推送授权请求（PAR） | 一种请求，您的后端通过它将授权参数发送到 `auth.claude.com`，并获取一个用于浏览器重定向的 `request_uri` |
| 工作负载身份联合（Workload Identity Federation） | Claude Console 中的一项功能，您的组织在其中注册您的身份提供商，以便 Anthropic 接受使用您的密钥签署的 JWT |
| 用户配置文件 | 您自己的 Claude 组织中代表您某个客户的记录 |
| 连接 | 将客户的 Claude 组织与您的某个用户配置文件绑定的记录，在客户关联您的应用时创建。其 ID 以 `ocon_` 开头。 |
| 服务账户 | 您的 Claude 组织中的非人类账户。您应用的操作员服务账户是您的后端在向客户计费时进行身份验证所使用的账户。 |
| 连接的 OAuth 令牌 | `auth.claude.com` 为连接返回的访问令牌，以及在使用 `offline_access` 时随之返回的刷新令牌。该访问令牌只能向该连接的客户计费。 |
