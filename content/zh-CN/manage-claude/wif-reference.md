---
title: WIF 参考
url: https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference
description: 工作负载身份联合的环境变量、验证规则、配置文件配置和错误参考。
---

本页汇总了[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)（WIF）的配置项、验证约束和错误映射。有关设置演练，请参阅[身份提供商](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#identity-providers)。

## 令牌交换请求

`POST /v1/oauth/token` 接受使用 [RFC 7523](https://www.rfc-editor.org/rfc/rfc7523) `jwt-bearer` 授权的 JSON 请求体。SDK 会根据[环境变量](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#environment-variables)为您构建此请求。

| 字段                   | 必需    | 描述                                                                                                                                 |
| -------------------- | ----- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `grant_type`         | 是     | 始终为 `urn:ietf:params:oauth:grant-type:jwt-bearer`。                                                                                 |
| `assertion`          | 是     | 由您的身份提供商签发的 OIDC JWT。                                                                                                              |
| `federation_rule_id` | 是     | 要评估的联合规则的带标签 ID（`fdrl_...`）。                                                                                                       |
| `organization_id`    | 是     | 您的 Anthropic 组织的 UUID。                                                                                                             |
| `service_account_id` | 是     | 目标服务账户的带标签 ID（`svac_...`）。                                                                                                         |
| `workspace_id`       | 视情况而定 | 签发的令牌所限定的工作区的带标签 ID（`wrkspc_...`）。当规则在多个工作区中启用时为必需。省略时，服务器会选择该规则唯一启用的工作区。字面值 `default` 也可用于组织的默认工作区，但已弃用。请使用该工作区的 `wrkspc_...` ID。 |

## 令牌交换响应

`POST /v1/oauth/token` 返回标准的 OAuth 2.0 令牌响应（[RFC 6749 §5.1](https://www.rfc-editor.org/rfc/rfc6749#section-5.1)）：

| 字段             | 类型      | 描述                                                                                |
| -------------- | ------- | --------------------------------------------------------------------------------- |
| `access_token` | string  | 短期有效的 Anthropic 令牌，前缀为 `sk-ant-oat01-...`。以 `Authorization: Bearer <token>` 形式传递。 |
| `token_type`   | string  | 始终为 `Bearer`。                                                                     |
| `expires_in`   | integer | 令牌过期前的秒数。                                                                         |
| `scope`        | string  | 匹配规则授予的 OAuth 范围。                                                                 |

## 环境变量

SDK 读取这些变量，以便在不传入任何构造函数参数的情况下执行联合令牌交换。

| 变量                              | 必需                           | 描述                                                                                                                                                                                                                                                       | 示例                                     |
| ------------------------------- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| `ANTHROPIC_FEDERATION_RULE_ID`  | 是                            | 要评估的联合规则的带标签 ID。                                                                                                                                                                                                                                         | `fdrl_...`                             |
| `ANTHROPIC_ORGANIZATION_ID`     | 是                            | 您的 Anthropic 组织的 UUID。可在 Claude Console 的[组织](https://platform.claude.com/settings/organization)页面（`platform.claude.com/settings/organization`）上找到。                                                                                                      | `00000000-0000-0000-0000-000000000000` |
| `ANTHROPIC_IDENTITY_TOKEN_FILE` | `_TOKEN_FILE` 或 `_TOKEN` 二选一 | 由您的身份提供商（IdP）签发的 JWT 的文件系统路径。SDK 在每次交换时都会重新读取此文件，以确保在磁盘上轮换的投射令牌始终是最新的。                                                                                                                                                                                   | `/var/run/secrets/anthropic.com/token` |
| `ANTHROPIC_IDENTITY_TOKEN`      | `_TOKEN_FILE` 或 `_TOKEN` 二选一 | 字符串形式的 JWT 字面值。当您的平台以环境变量而非文件的形式注入令牌时使用。                                                                                                                                                                                                                 | `eyJhbGciOiJSUzI1NiIs...`              |
| `ANTHROPIC_SERVICE_ACCOUNT_ID`  | 是                            | 签发的访问令牌所代表的目标 Anthropic 服务账户的带标签 ID。                                                                                                                                                                                                                     | `svac_...`                             |
| `ANTHROPIC_WORKSPACE_ID`        | 视情况而定                        | 签发的令牌所限定的工作区的带标签 ID。当联合规则在多个工作区中启用时为必需。当规则绑定到单个工作区时为可选。签发的令牌在交换时即被限定到此工作区，因此切换工作区需要重新交换。在没有配置文件的情况下，`ant` CLI 不会将此变量传入交换。使用 `ant` CLI 时，请在联合配置文件中设置 `workspace_id`，并将 `ANTHROPIC_PROFILE` 设置为该配置文件的名称。字面值 `default` 也可用于默认工作区，但已弃用。请使用其 `wrkspc_...` ID。 | `wrkspc_...`                           |
| `ANTHROPIC_PROFILE`             | 否                            | 要加载的[配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#profile-configuration-file)名称。优先于此表中的联合环境变量。                                                                                                                               | `staging-profile`                      |

仅当 `ANTHROPIC_FEDERATION_RULE_ID`、`ANTHROPIC_ORGANIZATION_ID`、`ANTHROPIC_SERVICE_ACCOUNT_ID` 以及 `ANTHROPIC_IDENTITY_TOKEN_FILE` 或 `ANTHROPIC_IDENTITY_TOKEN` 之一全部设置时，直接环境变量联合路径才会激活。SDK 也会读取 `ANTHROPIC_WORKSPACE_ID`，但无论是否设置该变量，此路径都会激活。

<Warning>
  设置为空字符串的变量可能占据其在凭据优先级链中的位置。请取消设置未使用的凭据变量，而不是将其置空。
</Warning>

### 凭据优先级

SDK 按以下顺序解析凭据。第一个产生凭据的来源胜出。

| 顺序 | 来源                                           | 说明                                                                                                                                 |
| -- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| 1  | 构造函数参数（API 密钥、身份验证令牌或凭据）                     | 始终覆盖其他所有来源。                                                                                                                        |
| 2  | `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` | 完全覆盖联合。从 API 密钥迁移时请取消设置这些变量。                                                                                                       |
| 3  | `ANTHROPIC_PROFILE`                          | 加载 `<config_dir>/configs/<name>.json`。如果缺少指定名称的配置文件，SDK 会报错，而不会尝试下一个来源。                                                            |
| 4  | 联合环境变量                                       | `ANTHROPIC_FEDERATION_RULE_ID` + `ANTHROPIC_ORGANIZATION_ID` + `ANTHROPIC_SERVICE_ACCOUNT_ID` + `ANTHROPIC_IDENTITY_TOKEN[_FILE]`。 |
| 5  | 活动配置文件                                       | 从 `<config_dir>/active_config` 解析，回退到名为 `default` 的配置文件。                                                                           |

在 `ant` CLI 中，[Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#authentication) 的设置优先级低于联合身份（federation）环境变量，高于活动配置文件（profile）。这些设置包括项目、工作区 ID 和位置。您可以通过 `--google-cloud-*` 标志或 `ANTHROPIC_GOOGLE_CLOUD_*` 变量来设置它们。只要设置了其中任意一项，CLI 就会使用这些设置。使用这些设置时，CLI 会从 Google Application Default Credentials（应用默认凭据）获取凭据。

加载配置文件时，环境变量会填充配置文件省略的字段，但绝不会覆盖配置文件显式设置的字段。例如，仅当活动配置文件未设置 `workspace_id` 时，`ANTHROPIC_WORKSPACE_ID` 才会填充该字段。

## 配置文件

“Profile”（配置文件）是 SDK 和 `ant` CLI 都会读取的命名配置文件。借助配置文件，您可以将联合参数随容器镜像一起发布，或在不更改代码的情况下切换环境。

### 配置目录

SDK 按以下顺序定位配置目录：

1. `$ANTHROPIC_CONFIG_DIR`
2. Linux 和 macOS 上为 `~/.config/anthropic`
3. Windows 上为 `%APPDATA%\Anthropic`

### 活动配置文件

活动配置文件名称按以下顺序解析：

1. `$ANTHROPIC_PROFILE`
2. `<config_dir>/active_config` 的内容（由 `ant profile activate <name>` 写入的单行文件）
3. 字面名称 `default`

Claude Code 和 Claude Agent SDK 遵循相同的解析顺序，因此在此处配置的联合配置文件无需额外设置即可为这些工具进行身份验证。

### 文件布局

| 路径                                        | 内容                                                                          | 敏感性                      |
| ----------------------------------------- | --------------------------------------------------------------------------- | ------------------------ |
| `<config_dir>/configs/<profile>.json`     | `version`、`authentication` 块、`organization_id`、`workspace_id` 和 `base_url`。 | 非机密。可以安全地提交或打包进镜像。       |
| `<config_dir>/credentials/<profile>.json` | `version`、缓存的 `access_token`、`expires_at`，以及（用于交互式登录的）`refresh_token`。      | 机密。SDK 以 `0600` 模式写入此文件。 |

配置文件和凭据文件都带有一个 `major.minor` 格式的顶层字符串 `version` 字段，例如 `"1.0"`。SDK 会自动写入此字段。如果您在手动编写配置时省略该字段，SDK 会将该文件视为当前版本。

### 联合配置文件示例

```json configs/production.json
{
  "version": "1.0",
  "authentication": {
    "type": "oidc_federation",
    "federation_rule_id": "fdrl_...",
    "service_account_id": "svac_...",
    "identity_token": {
      "source": "file",
      "path": "/var/run/secrets/anthropic.com/token"
    }
  },
  "organization_id": "00000000-0000-0000-0000-000000000000",
  "workspace_id": "wrkspc_...",
  "base_url": "https://api.anthropic.com"
}
```

如果您省略 `authentication.identity_token`，SDK 会回退到环境中的 `ANTHROPIC_IDENTITY_TOKEN_FILE` 或 `ANTHROPIC_IDENTITY_TOKEN`。

## OAuth 范围

您在联合规则上设置的 `oauth_scope` 决定了签发的访问令牌可以调用哪些 Claude API 端点。

| 范围                         | 授予访问权限                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `workspace:developer`      | 规则所在工作区中所有非管理类 Claude API 端点：[Messages](https://platform.claude.com/docs/zh-CN/api/messages)（包括流式传输和令牌计数）、[Models](https://platform.claude.com/docs/zh-CN/api/models/list)、[Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 及其会话、[Files](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 和 [Skills](https://platform.claude.com/docs/zh-CN/build-with-claude/skills-guide)。这与同一工作区中工作区 API 密钥所拥有的访问权限一致。 |
| `workspace:inference`      | 规则所在工作区中的推理端点：[Messages](https://platform.claude.com/docs/zh-CN/api/messages)（包括流式传输和令牌计数）、[Models](https://platform.claude.com/docs/zh-CN/api/models/list) 以及 [OpenAI 兼容的聊天端点](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/openai-sdk)。适用于只需调用 Claude、从不需要管理 Files、Skills 或其他资源的工作负载。                                                                                                                                           |
| `workspace:manage_tunnels` | [MCP 隧道 API](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/reference#tunnels-api)：创建、列出和获取隧道，注册和归档 CA 证书，显示和轮换隧道令牌，以及归档隧道。当您从 Console 的创建隧道模态窗口创建规则时，该窗口会锁定此范围。                                                                                                                                                                                                                                                                    |
| `org:admin`                | 对 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 的完全访问权限（组织成员、邀请、工作区、API 密钥等）。OAuth `org:admin` 令牌只能创建或修改范围为 `workspace:developer` 或 `workspace:inference` 的规则。它不能更新支撑任何其他范围规则的颁发者。请参阅[权限和约束](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#permissions-and-constraints)。                                                                                                                              |

对令牌范围之外的端点发出的请求会返回 HTTP 403。不提供更细粒度的范围（按资源，或按读与写区分）。

### 权限边界

联合规则的 `oauth_scope` 是一个上限：签发的令牌永远不能超出它。目标服务账户的 `organization_role`（`developer` 或 `admin`）决定了规则可以授予哪些范围。授予 `org:admin` 的规则必须以 `organization_role=admin` 的服务账户为目标。有效权限是规则范围与服务账户角色的交集。

| 规则 `oauth_scope`      | 服务账户 `organization_role` | 有效权限                                                                                                                                                                      |
| --------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `workspace:developer` | `admin`                  | 仅限规则所在工作区中的 Claude API 访问。范围将令牌限制在角色之下。                                                                                                                                   |
| `org:admin`           | `admin`                  | 完全的 Admin API 访问权限（组织成员、邀请、工作区、API 密钥等），但需减去针对 OAuth 调用方的例外情况。请参阅[权限和约束](https://platform.claude.com/docs/zh-CN/manage-claude/wif-admin-api#permissions-and-constraints)。 |

## 验证规则

Anthropic 在您创建或更新颁发者和规则时，以及在交换时验证传入的 JWT 时，会强制执行这些约束。

有关完整的参数详情和响应架构，请参阅[服务账户 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/service_accounts)、[联合颁发者 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/federation/issuers)和[联合规则 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/federation/rules)。

### 资源字段

| 字段                          | 约束                                                                                                                                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 颁发者、规则和服务账户的 `name`         | 必须匹配 `^[a-z0-9-]+$`，长度为 1 到 255 个字符。                                                                                                                                                                    |
| `workspace_id`              | 创建时必需，除非 `applies_to_all_workspaces` 为 true。在此规则下签发的令牌所适用的配额、计费和速率限制所属的工作区（`wrkspc_...`）。必须是同一组织中的工作区，且目标服务账户必须是该工作区的成员。                                                                                |
| `applies_to_all_workspaces` | 布尔值。设置为 `true` 可在组织的每个工作区中启用该规则，而不是指定某一个工作区。创建时必须提供此字段或 `workspace_id` 之一。                                                                                                                              |
| `token_lifetime_seconds`    | 介于 `60` 和 `86400` 之间的整数（1 分钟到 24 小时）。默认值为 `3600`。Anthropic 会在请求时拒绝超出此范围的值。请参阅[令牌生命周期与刷新](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#token-lifetime-and-refresh)。 |

### URL 字段

Anthropic 会验证 `issuer_url`、`jwks.discovery_base` 和 `jwks.url` 字段：

| 约束 | 详情                                                            |
| -- | ------------------------------------------------------------- |
| 协议 | 必须为 `https`。                                                  |
| 端口 | 必须为 `443`（显式或默认）。                                             |
| 主机 | 必须是您的 OIDC 提供商的公共 DNS 主机名。必须解析为公共 IP 地址。Anthropic 不接受 IP 字面量。 |

URL 验证失败会返回 `400 invalid_request_error`，错误消息以字段名作为前缀（例如 `issuer_url: url must use https scheme`）。

<Note>
  URL 约束仅适用于 Anthropic 会连接的 URL。在 `explicit_url` 和 `inline` JSON Web Key Set（JSON Web 密钥集，JWKS）模式下，以及在设置了 `jwks.discovery_base` 的 `discovery` 模式下，Anthropic 从不获取 `issuer_url`。它仅将该 URL 作为字符串与 JWT `iss` 声明进行比较，因此该 URL 可以引用内部主机名或非标准端口。
</Note>

### JWT 验证

| 约束    | 详情                                                                                                                                                                                                                  |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 最大大小  | `assertion` JWT 最多为 16 KiB。                                                                                                                                                                                         |
| 签名算法  | Anthropic 仅接受非对称算法（RSA 和 ECDSA 系列：ES256、ES384、ES512、RS256、RS384、RS512、PS256、PS384、PS512）。它会拒绝 HMAC（`HS256`、`HS384`、`HS512`）和 `none`。                                                                                |
| 密钥 ID | JWT 头部必须携带与颁发者 JWKS 中某个密钥匹配的 `kid`。Anthropic 会拒绝没有 `kid` 的令牌。                                                                                                                                                       |
| 必需声明  | 必须存在 `sub`。必须存在 `iat` 且不能是未来时间。必须存在 `exp` 且必须是未来时间。                                                                                                                                                                 |
| 一次性使用 | 携带 `jti` 声明的断言在每个颁发者下只能交换一次。Anthropic 会将使用相同 `jti` 的重复交换视为重放而拒绝。颁发者的 `check_jti` 字段（默认启用）控制此检查。没有 `jti` 声明的断言不受此检查约束。请参阅[联合颁发者 API 参考](https://platform.claude.com/docs/zh-CN/api/organization/federation/issuers)。 |
| 最长有效期 | 令牌的有效期（`exp` 减去 `iat`）不得超过颁发者配置的最大值（默认为 1 小时，可在 Claude Console 中为每个颁发者配置）。                                                                                                                                          |
| 时钟偏差  | Anthropic 对 `exp`、`nbf` 和 `iat` 应用 30 秒的容差。                                                                                                                                                                         |

## 规则匹配语义

联合规则的 `match` 块决定 Anthropic 是否接受传入的 JWT。JWT 必须满足每个已填写的匹配器。请至少设置 `subject_prefix`、`claims` 或 `condition` 之一。Anthropic 会拒绝仅包含 `audience` 或完全没有匹配器的 `match` 块。这可以防止出现接受某个颁发者所有令牌的规则。

| 匹配器              | 类型                   | 语义                                                                                                              |
| ---------------- | -------------------- | --------------------------------------------------------------------------------------------------------------- |
| `subject_prefix` | string               | 与 JWT 的 `sub` 声明精确匹配。末尾的 `*` 使其成为前缀匹配（`sub` 值必须以 `*` 之前的字符开头）。区分大小写。                                            |
| `audience`       | string               | JWT `aud` 声明必须包含此精确字符串。当 `aud` 为数组时，任一元素精确匹配即满足检查。                                                              |
| `claims`         | map\<string, string> | 每个键是一个顶层声明名称，每个值是所需的精确字符串值。对于嵌套、数值、布尔或列表和映射等复杂声明，请改用带有 Common Expression Language（通用表达式语言，CEL）表达式的 `condition`。 |
| `condition`      | string (CEL)         | 必须求值为 `true` 的 [CEL](https://cel.dev/) 表达式。                                                                     |

### CEL 求值环境

`condition` 表达式可以访问一个变量：

| 变量       | 类型  | 内容                           |
| -------- | --- | ---------------------------- |
| `claims` | map | 完整解码的 JWT 声明集。嵌套对象可作为嵌套映射访问。 |

示例：

```text wrap
claims.sub.startsWith("repo:acme-corp/") && claims.ref in ["refs/heads/main", "refs/heads/release"]
```

<Warning>
  CEL 条件是安全边界。对超出预期范围的输入求值为 `true` 的表达式会授予超出预期的访问权限。当静态匹配器能够表达您的约束时，请优先使用静态匹配器。
</Warning>

## 错误

### 令牌交换错误

`POST /v1/oauth/token` 以标准的 [API 错误格式](https://platform.claude.com/docs/zh-CN/api/errors)返回错误。SDK 将交换失败报告为 `WorkloadIdentityError` (ruby: `Anthropic::Credentials::WorkloadIdentityError`; go: `error`; java: `UnexpectedStatusCodeException`; csharp: `WorkloadIdentityException`; php: `OAuthException`)，其中包含 HTTP 状态以及响应体中的错误详情。除 C# 和 PHP 外，SDK 还会提供请求 ID：位于错误对象上、异常的响应头中，或错误消息中。

| 状态  | 错误                      | 原因                                                                           | 解决方法                                                                                                                                                                                                                                                                      |
| --- | ----------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 400 | `invalid_request_error` | `federation_rule_id` 格式错误，或缺少必需的请求字段。                                        | 检查 `fdrl_` ID，并确认请求体包含所有必需字段。                                                                                                                                                                                                                                             |
| 400 | `invalid_request_error` | 存在 `workspace_id`，但它既不是格式正确的 `wrkspc_...` ID，也不是字面值 `default`。               | 修正 `workspace_id` 值。响应消息会指明预期格式。                                                                                                                                                                                                                                          |
| 401 | `authentication_error`  | JWT `iss` 声明与注册的 `issuer_url` 不完全相等。                                         | 逐字节比较，包括末尾斜杠和协议：`jq -rR 'split(".")[1] \| gsub("-";"+") \| gsub("_";"/") \| @base64d \| fromjson \| .iss' <<< "$JWT"`。                                                                                                                                                    |
| 401 | `authentication_error`  | JWKS 获取失败、JWKS 已过时，或 JWT 使用了不在 JWKS 中的密钥签名。                                  | 对于 `inline` 模式，使用轮换后的密钥更新颁发者。对于 `discovery` 和 `explicit_url`，确认 JWKS 端点可通过 443 端口访问。如果颁发者最近轮换了签名密钥，请参阅[密钥轮换和缓存](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#key-rotation-and-caching)。                                                             |
| 401 | `authentication_error`  | JWT `exp` 声明已过期（超出 30 秒偏差窗口）。                                                | 确认您的身份提供商正在投射新的令牌，并且 SDK 正在重新读取令牌文件。                                                                                                                                                                                                                                      |
| 401 | `authentication_error`  | JWT 已通过验证，但其声明不满足规则的 `match` 块。                                              | 解码 JWT 并将每个声明与规则进行比较。`subject_prefix` 区分大小写。`audience` 要求元素精确匹配。                                                                                                                                                                                                          |
| 401 | `authentication_error`  | `federation_rule_id` 不存在或已归档，或 JWT 未获授权使用该规则（这些情况使用同一错误以防止枚举）。               | 在 Claude Console 中确认规则 ID，并确认该规则未被归档。                                                                                                                                                                                                                                     |
| 401 | `authentication_error`  | 联合规则在多个工作区中启用，而请求省略了 `workspace_id`。身份验证历史记录条目显示原因为 `workspace_id_required`。 | 将 `ANTHROPIC_WORKSPACE_ID`（或原始请求中的 `workspace_id` 请求体字段）设置为您希望令牌限定到的 `wrkspc_...` ID。使用 `ant` CLI 时，请在联合配置文件中设置 `workspace_id`，并将 `ANTHROPIC_PROFILE` 设置为该配置文件的名称。请参阅[令牌交换请求](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#token-exchange-request)。 |

无论哪项检查失败，每次断言拒绝都会返回相同的不透明 `401` `authentication_error`，消息固定为 `Authentication failed`。可区分的错误会让调用方得以探测规则配置。Claude Console 中的[身份验证历史记录](https://platform.claude.com/settings/workload-identity-federation?tab=history)会在该次尝试的条目上记录拒绝原因。例如，当 `sub` 声明不满足规则的 `subject_prefix` 时显示 `match_subject_prefix`，或当规则跨越多个工作区而请求未指定任何工作区时显示 `workspace_id_required`。在规则所属组织得到确认之前就被拒绝的请求（上表中的 `400 invalid_request_error` 类错误）不会留下历史记录条目。它们的响应消息会直接指明问题。没有对应历史记录条目的 `401` 通常意味着 Anthropic 未识别 `federation_rule_id`。

### 常见的 SDK 端故障

| 症状                           | 原因                                                                                                                                             | 解决方法                                                                                                               |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| SDK 报告缺少凭据而不是执行交换            | `ANTHROPIC_FEDERATION_RULE_ID`、`ANTHROPIC_ORGANIZATION_ID`、`ANTHROPIC_SERVICE_ACCOUNT_ID` 或 `ANTHROPIC_IDENTITY_TOKEN[_FILE]` 之一未设置，且没有活动配置文件。 | 设置全部四个变量，或配置一个配置文件。                                                                                                |
| SDK 使用 API 密钥进行身份验证而不是联合     | 设置了 `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN`，并在优先级中胜出。                                                                                     | 取消设置密钥或令牌变量。                                                                                                       |
| 首次请求时出现文件未找到错误               | `ANTHROPIC_IDENTITY_TOKEN_FILE` 中的路径不存在。SDK 在交换时才延迟打开该文件。                                                                                      | 确认投射令牌卷已挂载且路径匹配。                                                                                                   |
| 令牌交换成功，但 Claude API 请求返回 403 | 签发令牌的范围未授予对该端点的访问权限。                                                                                                                           | 对照 [OAuth 范围](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#oauth-scopes)检查规则的 `oauth_scope`。 |
| 因凭据为空而身份验证失败                 | 某个凭据环境变量已导出但设置为空字符串。空值可能会占据其优先级位置。                                                                                                             | 使用 `unset VAR` 而不是 `VAR=""` 取消设置该变量。                                                                               |

## 排查交换失败

`401` `authentication_error` 响应是有意设计为不透明的，其消息始终为 `Authentication failed`。拒绝原因显示在身份验证历史记录中，而不在响应中。

<Tip>
  请从 Claude Console 中的[身份验证历史记录页面](https://platform.claude.com/settings/workload-identity-federation?tab=history)开始排查。最近的交换尝试会显示 Anthropic 所评估的颁发者和规则、其所检查的 JWT 声明，以及哪个验证步骤失败。这些信息通常可以让您无需进行以下检查。
</Tip>

一种常见的不透明故障是断言重放。携带 `jti` 声明的断言[只能交换一次](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#jwt-verification)。重新发送同一 JWT 的工作负载（重试循环，或重新读取未轮换令牌的刷新操作）会在第二次交换时失败。身份验证历史记录页面会以原因 `jti_reused` 显示这些尝试。要解决此问题，请为每次交换签发新的断言。

如果您需要从 JWT 本身进行调试，请按顺序完成以下检查：

<Steps>
  <Step title="解码 JWT">
    解码您发送的断言，以便将每个声明与您的颁发者和规则配置进行比较：

    ```bash
    jq -rR 'split(".")[1] | gsub("-";"+") | gsub("_";"/") | @base64d | fromjson' <<< "$JWT"
    ```
  </Step>

  <Step title="检查 iss 是否与颁发者匹配">
    解码后的 `iss` 声明必须与注册的 `issuer_url` 逐字节相等，包括协议、端口和任何末尾斜杠。哪怕只有一个字符不匹配，验证也会失败。
  </Step>

  <Step title="检查 aud 是否与规则匹配">
    解码后的 `aud` 声明必须包含与规则的 `audience` 值精确匹配的内容。当 `aud` 为数组时，必须有一个元素精确匹配。
  </Step>

  <Step title="检查 sub 和每个 claims 条目">
    将 `sub` 与规则的 `subject_prefix` 进行比较。该比较区分大小写。末尾的 `*` 表示前缀匹配，其他情况均为精确匹配。将规则 `claims` 映射中的每个键与同名的顶层声明进行比较。
  </Step>

  <Step title="检查 exp、nbf 和 iat">
    `exp` 必须是未来时间，`nbf`/`iat` 必须是过去时间，均在 30 秒偏差窗口内。如果工作负载主机的时钟发生漂移，Anthropic 会拒绝原本有效的令牌。
  </Step>

  <Step title="检查 JWKS 可达性">
    对于 `discovery` 模式，通过 443 端口上的公共 HTTPS 获取 `<jwks.discovery_base or issuer_url>/.well-known/openid-configuration`，并确认 `jwks_uri` 可以解析。对于 `explicit_url`，直接获取 JWKS URL。对于 `inline`，确认自您注册密钥以来颁发者的签名密钥未发生轮换。

    如果颁发者轮换了签名密钥并立即开始使用它签名，在 Anthropic 的 JWKS 缓存刷新期间，交换可能会失败长达一分钟。请参阅[密钥轮换和缓存](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#key-rotation-and-caching)。
  </Step>
</Steps>

## JWKS 来源模式

注册联合颁发者时，`jwks` 字段控制 Anthropic 如何获取用于验证该颁发者 JWT 签名的公钥。它的 `type` 字段用于选择来源模式：

| `jwks.type`     | `jwks` 结构                                                                                                     | 行为                                                                                                               | 适用场景                                                                                     |
| --------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `discovery`（默认） | `{ "type": "discovery", "discovery_base": "https://..." }`（当发现 URL 与 `issuer_url` 不同时，设置可选的 `discovery_base`） | Anthropic 获取 `<discovery_base or issuer_url>/.well-known/openid-configuration`，从发现文档中读取 `jwks_uri`，然后从该处获取 JWKS。 | 您的 IdP 在公共互联网上提供标准的 OIDC 发现文档。大多数托管提供商（EKS、GKE、Cloud Run、GitHub Actions、Entra ID）都支持此方式。 |
| `explicit_url`  | `{ "type": "explicit_url", "url": "https://..." }`                                                            | Anthropic 直接从 `url` 获取 JWKS。Anthropic 仅将 `issuer_url` 用于与 JWT `iss` 声明进行字符串比较，从不连接它。                             | 您的 IdP 不提供发现文档，或发现文档仅限内部访问但 JWKS 可公开访问。                                                  |
| `inline`        | `{ "type": "inline", "keys": [...] }`                                                                         | 您以内联方式提供 JWK 对象数组（JWKS 文档中的 `keys` 数组，而不是外层包装对象）。Anthropic 不发出任何出站请求。Anthropic 仅将 `issuer_url` 用于 `iss` 比较。      | 隔离网络环境、使用集群内部颁发者 URL 的自管理 Kubernetes 集群，或者您希望显式控制密钥轮换时。                                  |

每种 `type` 都有各自的配套字段，因此这些字段相互排斥。`discovery` 和 `explicit_url` 还都接受一个可选的 `ca_cert_pem` 字符串，用于通过私有 CA 提供 TLS 的颁发者。

### 密钥轮换和缓存

在 `discovery` 和 `explicit_url` 模式下，Anthropic 会缓存获取到的 JWKS。在您的身份提供商发布新的签名密钥后，在缓存刷新期间，使用该密钥签名的令牌的交换可能会因签名错误而失败，持续时间最长 1 分钟。

为避免出现此窗口期，请在您的身份提供商开始使用新签名密钥签名令牌之前至少 15 分钟将其发布到 JWKS 中。将被替换的密钥保留在 JWKS 中，直到由其签名的令牌过期。托管身份提供商通常会自行做到这一点。如果您运营自己的颁发者，请确认您的轮换策略会在首次使用之前发布新密钥。例如自管理 Kubernetes 集群、SPIRE OIDC 发现提供商，以及配置了轮换计划的 Okta 自定义授权服务器。

<Warning>
  在 `inline` 模式下没有自动密钥刷新。当您的身份提供商轮换其签名密钥时，您必须使用新的 JWKS 更新颁发者配置，否则所有令牌交换都将无法通过签名验证。
</Warning>
