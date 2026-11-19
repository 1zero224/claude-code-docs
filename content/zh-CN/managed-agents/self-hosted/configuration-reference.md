---
title: 配置参考
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference
description: 查看自托管 Claude Managed Agents 服务器的设置存放在何处，并查阅每项设置的键、环境变量、默认值和用途。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

本页说明自托管 Claude Managed Agents 服务器的设置存放在何处，并列出这些设置。要设置某项功能，请参阅[单点登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure)、[智能体身份和凭据](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity)和[推理提供商](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider)。

## 设置的存放位置

服务器从一个 YAML 文件读取其设置，并从其环境读取其凭据。

* **Docker Compose：** 该文件是解压后归档中与 `docker-compose.yml` 位于同一目录的 `compose.config.yaml`。编辑该文件，然后在该目录中运行 `docker compose restart cma-server`。
* **Kubernetes：** 该文件是服务器 Helm chart 的 `server.config` 值。在您的 values 文件中编辑它，运行 `helm upgrade`，然后使用 `kubectl -n cma-server rollout restart deployment -l anthropic.com/role=server` 重启服务器。请参阅[在 Kubernetes 上部署](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes)。

如果服务器拒绝自托管页面中所示的某项设置，请[升级服务器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade)。归档顶层的 `cma-server.example.yaml` 为常用键提供了注释。

凭据从不放在该文件中。服务器从环境变量中读取凭据，或从您的云平台附加到主机的身份中读取。

### 将凭据传递给 Compose 栈

在 Compose 栈中，服务器容器仅接收 `docker-compose.yml` 中 `cma-server` 服务的 `environment:` 列表里的变量。要传递其他变量：

1. 将其名称添加到该列表中，并在 `.env` 中设置其值。
2. 如果凭据是一个文件，请将该文件放在 `docker-compose.yml` 旁边，并在 `docker-compose.yml` 的 `x-server-mounts` 列表中添加一行，例如 `- ./gcp-key.json:/etc/cma-server/gcp-key.json:ro`。第一个冒号之后的路径是该文件在容器内的路径。请将该路径用作 `.env` 中该变量的值，或用作指定该文件的设置的值。
3. 运行 `docker compose up -d`。重启不会重新读取 `.env`。

升级会替换 `docker-compose.yml`，因此请在启动新版本的栈之前，在新版本的该文件中重复这些编辑。

## 如何阅读表格

每项设置在[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)中都有一个键，大多数设置还有一个环境变量。变量会覆盖文件，文件会覆盖默认值。如果设置文件中有未知的键，服务器将不会启动，错误信息会指明该键所在的位置。

* **键：** 键中的每个点表示文件中的一层嵌套。例如，`server.addr` 是 `server` 部分中的 `addr` 键。
* **变量：** 变量名由 `CMA_` 加上大写形式的键组成，每个点替换为两个下划线。
* **仅文件：** 该设置没有变量。
* **仅环境变量：** 如果该设置出现在文件中，服务器将无法启动。
* **仅限 `bedrock` 及类似标签：** 该设置仅适用于相应的提供商、存储或密钥管理器。如果您在使用其他提供商、存储或密钥管理器时设置了它，服务器将不会启动。
* **列表：** 列表在文件中是 YAML 列表，在变量中是逗号分隔的字符串。
* **时长：** 时长是带单位的数字，例如 `30s`、`5m` 或 `1h`。
* **整数：** 整数是纯数字，不带符号、前导零或下划线。

启动时，服务器会记录您设置的设置以及大多数默认值，每项都附带其值的来源，例如 `server.addr=0.0.0.0:8400 (file)`。对于机密值，服务器会记录 `***`，但服务器自行生成的服务器 API 密钥除外。

## 服务器和日志

| 设置<br />环境变量                                                                      | 默认值              | 描述                                                                                                                                                                                                                             |
| --------------------------------------------------------------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `server.addr`<br />`CMA_SERVER__ADDR`                                             | `127.0.0.1:8400` | 服务器监听的地址。                                                                                                                                                                                                                      |
| `server.api_key`<br />`CMA_SERVER__API_KEY`                                       | 无                | [服务器 API 密钥](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted#the-three-credentials)。使用 `store.backend: postgres` 时必需，除非 `auth.refuse_server_api_key` 为 `true`。使用 `memory` 时，如果您未设置密钥，服务器会生成一个密钥，并在启动时记录到日志中。 |
| `server.id_marker`<br />`CMA_SERVER__ID_MARKER`                                   | `local`          | 服务器创建的每个 ID 中前缀之后的内容。`local` 生成 `sesn_local_01…`，`none` 生成 `sesn_01…`，与托管服务相同。                                                                                                                                                 |
| `server.shutdown_delay`<br />`CMA_SERVER__SHUTDOWN_DELAY`                         | `0s`             | 收到停止信号后服务器继续提供服务的时长，在此期间其 “readiness probe”（就绪探针）会返回 `not_ready`。最大值为 `5m`。                                                                                                                                                    |
| `server.max_connections`<br />`CMA_SERVER__MAX_CONNECTIONS`                       | `0`              | 服务器在 `server.addr` 上同时保持打开的最大连接数。`0` 表示无限制。达到上限时，新连接会等待，直到另一个连接关闭或其客户端放弃。事件流、worker 轮询和健康探针都会计入。在 Kubernetes 上，如果 pod 的探针等待时间过长，kubelet 会重启该 pod。                                                                              |
| `logging.format`<br />`CMA_LOGGING__FORMAT`                                       | `text`           | 日志格式，`text` 或 `json`。                                                                                                                                                                                                          |
| `logging.level`<br />`CMA_LOGGING__LEVEL`                                         | `info`           | 服务器记录的最低级别。可选值为 `debug`、`info`、`warn` 和 `error`。                                                                                                                                                                               |
| `logging.errors`<br />`CMA_LOGGING__ERRORS`                                       | `false`          | 日志是否包含 `4xx` 和 `5xx` 响应的错误消息。消息可能会引用客户端发送的值。                                                                                                                                                                                   |
| `health.check_interval`<br />`CMA_HEALTH__CHECK_INTERVAL`                         | `60s`            | 服务器为就绪探针检查其推理凭据的频率。最小值为 `5s`。                                                                                                                                                                                                  |
| `health.upstream_affects_readiness`<br />`CMA_HEALTH__UPSTREAM_AFFECTS_READINESS` | `true`           | 当推理提供商拒绝服务器的凭据时，就绪探针是否返回 `503`。                                                                                                                                                                                                |

## 数据库

此表中除 `store.backend` 之外的每项设置仅在 `store.backend: postgres` 时适用。

| 设置<br />环境变量                                                                                                   | 默认值                     | 描述                                                                                                                                                                    |
| -------------------------------------------------------------------------------------------------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `store.backend`<br />`CMA_STORE__BACKEND`                                                                      | `memory`                | 服务器保存数据的位置。可选值为 `memory`（重启会清除数据）和 `postgres`。                                                                                                                        |
| `store.database_url`<br />`CMA_STORE__DATABASE_URL`                                                            | 无                       | PostgreSQL 连接字符串。使用 `postgres` 时必需。                                                                                                                                   |
| `store.control_database_url`<br />`CMA_STORE__CONTROL_DATABASE_URL`                                            | `store.database_url` 的值 | 服务器自身维护连接（例如其心跳）所用的连接字符串。                                                                                                                                             |
| `store.listen_database_url`<br />`CMA_STORE__LISTEN_DATABASE_URL`                                              | `store.database_url` 的值 | 服务器用于监听通知的那一个连接所用的连接字符串。处于事务模式的 “connection pooler”（连接池）无法承载该连接。                                                                                                      |
| `store.migrate_database_url`<br />`CMA_STORE__MIGRATE_DATABASE_URL`                                            | `store.database_url` 的值 | 服务器的 `migrate` 命令使用的连接字符串。处于事务模式的连接池无法承载该连接。                                                                                                                          |
| `store.pool_max_conns`<br />`CMA_STORE__POOL_MAX_CONNS`                                                        | `24`                    | 单个服务器的连接池最多打开的数据库连接数。最小值为 `4`。                                                                                                                                        |
| `store.event_poll_interval`<br />`CMA_STORE__EVENT_POLL_INTERVAL`                                              | `25ms`                  | 每个服务器向数据库查询其客户端正在流式传输的会话中最新事件的频率。另一个服务器记录的事件最多等待大约这么长时间就会到达该流。                                                                                                        |
| `store.credentials_encryption_key`<br />`CMA_STORE__CREDENTIALS_ENCRYPTION_KEY`<br />仅环境变量                     | 无                       | 加密数据库中存储的凭据的密钥，为 base64 格式的 32 个随机字节。Docker Compose 和 Kubernetes 安装会设置此项。没有此密钥时服务器不会启动，除非 `store.allow_unencrypted_credentials` 为 `true`。                             |
| `store.allow_unencrypted_credentials`<br />`CMA_STORE__ALLOW_UNENCRYPTED_CREDENTIALS`                          | `false`                 | 服务器是否可以在没有 `store.credentials_encryption_key` 的情况下启动。没有该密钥时，服务器会以明文将凭据存储在数据库中。如果 `git.allowed_hostnames` 列出了主机，或者您使用 Docker Compose 或服务器 Helm chart 进行安装，`true` 不起作用。 |
| `store.credentials_encryption_keys_retiring`<br />`CMA_STORE__CREDENTIALS_ENCRYPTION_KEYS_RETIRING`<br />仅环境变量 | 无                       | 密钥轮换期间服务器仍可用于解密的早期加密密钥。此设置需要 `store.credentials_encryption_key`。                                                                                                      |

## 推理提供商

要设置提供商，请参阅[推理提供商](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider)。

| 设置<br />环境变量                                                         | 默认值         | 描述                                                                                                                                                                                                                                                                                                                                      |
| -------------------------------------------------------------------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `inference.provider`<br />`CMA_INFERENCE__PROVIDER`                  | `anthropic` | 服务器发送模型调用的目标。可选值为 `anthropic`、`bedrock`、`vertex`、`foundry` 和 `gateway`。                                                                                                                                                                                                                                                                 |
| `inference.headers`<br />仅文件                                         | 无           | 服务器添加到每次模型调用的请求头，格式为请求头名称到值的映射。请为每个值加引号。如果某个未加引号的值不会被 YAML 读取为文本（例如 `true` 或 `0042`），服务器将拒绝启动。使用 `gateway` 时，值也可以是 `{ from: request_id }` 或 `{ from: session_id }`。                                                                                                                                                                     |
| `inference.model_fallback`<br />`CMA_INFERENCE__MODEL_FALLBACK`      | `true`      | 仅限 `anthropic`。模型调用是否请求[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)。这些调用携带 `fallbacks` 字段和请求头 `anthropic-beta: server-side-fallback-2026-07-01`。您在 `ANTHROPIC_BASE_URL` 中指定的代理可能会拒绝该字段或该请求头。如果代理以 `400` 或 `422` 响应，服务器会在不带它们的情况下再次发送该调用。                               |
| `inference.metadata.user_id`<br />`CMA_INFERENCE__METADATA__USER_ID` | 无           | 仅限 `anthropic` 和 `foundry`，或使用 `format: anthropic-messages` 的 `gateway`。服务器在每次模型调用中作为 `metadata.user_id` 发送的一个标识符。启动日志会打印该值。如果未设置此项，服务器会发送标签 `cma-server`。您无法关闭 `metadata.user_id`。如果提供商或代理拒绝它，每次模型调用都会失败。请参阅[哪些内容会离开您的网络](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-data-and-access#what-leaves-your-network)。 |

### 云提供商

| 设置<br />环境变量                                                         | 默认值             | 描述                                                                                                                                                                                                                                                                                                                                             |
| -------------------------------------------------------------------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `inference.region`<br />`CMA_INFERENCE__REGION`                      | 无               | 仅限 `bedrock` 和 `vertex`。AWS 区域，或 Google Cloud’s Agent Platform 中的位置。使用 `vertex` 时必需。如果未设置，`bedrock` 使用 AWS SDK 默认链找到的区域。                                                                                                                                                                                                                       |
| `inference.project_id`<br />`CMA_INFERENCE__PROJECT_ID`              | 无               | 仅限 `vertex`。发起调用所在的 Google Cloud 项目。除非 Google 凭据指定了项目，否则必需。                                                                                                                                                                                                                                                                                    |
| `inference.api`<br />`CMA_INFERENCE__API`                            | `invoke_model`  | 仅限 `bedrock`。承载调用的 Amazon Bedrock API，`invoke_model` 或 `messages`。`messages` 是 [Claude in Amazon Bedrock（`bedrock-mantle`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock) 所描述的端点。使用它时，服务器的 AWS 角色需要 `bedrock-mantle:CreateInference`，而不是 `bedrock:InvokeModel` 和 `bedrock:InvokeModelWithResponseStream`。 |
| `inference.list_models`<br />`CMA_INFERENCE__LIST_MODELS`            | `true`          | 仅限 `bedrock`。服务器是否读取 Amazon Bedrock 的 Claude 模型列表。借助这些列表，智能体可以使用在您的服务器版本发布之后推出的模型。                                                                                                                                                                                                                                                             |
| `inference.model_id_prefix`<br />`CMA_INFERENCE__MODEL_ID_PREFIX`    | `us.anthropic.` | 仅限 `bedrock`。服务器在每个模型 ID 前添加的文本。使用 `inference.api: messages` 时，默认值为 `anthropic.`。                                                                                                                                                                                                                                                              |
| `inference.model_id_overrides`<br />仅文件                              | 无               | 仅限 `bedrock`、`vertex` 和 `foundry`。模型 ID 到提供商所需确切 ID 的映射。使用 `foundry` 时，该 ID 是部署名称。                                                                                                                                                                                                                                                             |
| `inference.foundry.resource`<br />`CMA_INFERENCE__FOUNDRY__RESOURCE` | 无               | 仅限 `foundry`。Foundry 资源的名称。设置此项或 `inference.foundry.base_url`，不要同时设置。                                                                                                                                                                                                                                                                          |
| `inference.foundry.base_url`<br />`CMA_INFERENCE__FOUNDRY__BASE_URL` | 无               | 仅限 `foundry`。默认情况下，调用会发送到 `https://<resource>.services.ai.azure.com/anthropic`。要通过其他地址（例如代理）访问该资源，请改为在此处设置该 URL。它以 `/anthropic` 结尾。                                                                                                                                                                                                            |

### 模型网关

这些设置仅在 `inference.provider: gateway` 时适用。请参阅[模型网关](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider#model-gateway)。

| 设置<br />环境变量                                                                              | 默认值             | 描述                                                                                                                                                                                                                            |
| ----------------------------------------------------------------------------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `inference.gateway.url`<br />`CMA_INFERENCE__GATEWAY__URL`                                | 无               | 必需。接收模型调用的 URL。使用 `bedrock-invoke` 和 `vertex-raw-predict` 时，`{model}` 标记模型 ID 所在的位置。使用 `vertex-raw-predict` 时，URL 必须以 `/{model}:rawPredict` 结尾。                                                                               |
| `inference.gateway.format`<br />`CMA_INFERENCE__GATEWAY__FORMAT`                          | 无               | 必需。网关接受的请求格式：`bedrock-invoke`、`anthropic-messages` 或 `vertex-raw-predict`。                                                                                                                                                    |
| `inference.gateway.model_map`<br />仅文件                                                    | 无               | 必需。智能体可指定的每个模型 ID 到网关所需 ID 的映射。请为每个值加引号，与 `inference.headers` 相同。                                                                                                                                                             |
| `inference.gateway.streaming`<br />`CMA_INFERENCE__GATEWAY__STREAMING`                    | `false`         | 服务器是否发送该格式的流式调用，以便回复在模型生成时即到达。使用 `bedrock-invoke` 时，该调用会发送到附加了 `-with-response-stream` 的同一 URL。使用 `vertex-raw-predict` 时，该调用会发送到将 `:rawPredict` 替换为 `:streamRawPredict` 的同一 URL。请在 `inference.gateway.url` 中保留 `:rawPredict`。 |
| `inference.credentials.supplier`<br />`CMA_INFERENCE__CREDENTIALS__SUPPLIER`              | 无               | 必需。网关凭据的来源。`env` 表示一个长期有效的密钥，`oauth_exchange` 表示短期令牌，`azure` 表示颁发给服务器自身 Azure 身份的 Microsoft Entra ID 令牌。                                                                                                                      |
| `inference.credentials.env.var`<br />`CMA_INFERENCE__CREDENTIALS__ENV__VAR`               | 无               | 使用 `env` 时必需。保存密钥的环境变量名称。                                                                                                                                                                                                     |
| `inference.credentials.azure.scope`<br />`CMA_INFERENCE__CREDENTIALS__AZURE__SCOPE`       | 无               | 使用 `azure` 时必需。服务器为其请求令牌的作用域：您网关的应用注册的应用程序 ID URI 或应用程序 ID，后跟 `/.default`。                                                                                                                                                    |
| `inference.credentials.send_as.header`<br />`CMA_INFERENCE__CREDENTIALS__SEND_AS__HEADER` | `Authorization` | 承载凭据的请求头。                                                                                                                                                                                                                     |
| `inference.credentials.send_as.scheme`<br />`CMA_INFERENCE__CREDENTIALS__SEND_AS__SCHEME` | `Bearer`        | 请求头中位于凭据之前的词。对于 `Authorization` 以外的任何请求头，默认无方案。                                                                                                                                                                               |
| `inference.credentials.renew_before`<br />`CMA_INFERENCE__CREDENTIALS__RENEW_BEFORE`      | `90s`           | 使用 `oauth_exchange` 时，服务器提前多久替换令牌。服务器会在缓存令牌过期前提前这么长时间停止使用它，并为下一次模型调用获取另一个令牌。请将该值保持在令牌有效期以下。服务器会拒绝任何有效期短于此值的令牌。                                                                                                                |

### 短期网关令牌

这些设置仅在 `inference.credentials.supplier: oauth_exchange` 时适用。请参阅[短期令牌](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider#short-lived-tokens)。`exchange` 设置作为一组是可选的。如果不设置，服务器会将回调返回的内容发送给网关。

| 设置<br />环境变量                                                                                                      | 默认值          | 描述                                                                                          |
| ----------------------------------------------------------------------------------------------------------------- | ------------ | ------------------------------------------------------------------------------------------- |
| `inference.credentials.assertion.url`<br />`CMA_INFERENCE__CREDENTIALS__ASSERTION__URL`                           | 无            | 必需。返回已签名断言的 HTTP 回调的 URL。                                                                   |
| `inference.credentials.assertion.auth.type`<br />`CMA_INFERENCE__CREDENTIALS__ASSERTION__AUTH__TYPE`              | 无            | 必需。服务器向回调进行身份验证的方式，`mtls` 或 `bearer_file`。                                                  |
| `inference.credentials.assertion.auth.cert_file`<br />`CMA_INFERENCE__CREDENTIALS__ASSERTION__AUTH__CERT_FILE`    | 无            | 使用 `mtls` 时必需。服务器客户端证书的路径。                                                                  |
| `inference.credentials.assertion.auth.key_file`<br />`CMA_INFERENCE__CREDENTIALS__ASSERTION__AUTH__KEY_FILE`      | 无            | 使用 `mtls` 时必需。客户端证书私钥的路径。                                                                   |
| `inference.credentials.assertion.auth.token_file`<br />`CMA_INFERENCE__CREDENTIALS__ASSERTION__AUTH__TOKEN_FILE`  | 无            | 使用 `bearer_file` 时必需。保存 bearer 令牌的文件路径。                                                     |
| `inference.credentials.assertion.auth.ca_file`<br />`CMA_INFERENCE__CREDENTIALS__ASSERTION__AUTH__CA_FILE`        | 无            | PEM 文件的路径，其中包含服务器为回调信任的证书颁发机构。                                                              |
| `inference.credentials.assertion.forward_metadata`<br />`CMA_INFERENCE__CREDENTIALS__ASSERTION__FORWARD_METADATA` | 无            | 服务器将其值发送给回调的会话元数据键。它们作为请求正文中的 `session.metadata` 到达。这些值是 API 调用者写入的任意内容，因此请对照您自己的记录进行核对。    |
| `inference.credentials.exchange.token_url`<br />`CMA_INFERENCE__CREDENTIALS__EXCHANGE__TOKEN_URL`                 | 无            | 进行交换时必需。服务器用断言交换令牌的 OAuth 令牌端点。                                                             |
| `inference.credentials.exchange.grant_type`<br />`CMA_INFERENCE__CREDENTIALS__EXCHANGE__GRANT_TYPE`               | 无            | 进行交换时必需。服务器发送的 `grant_type`，例如 `urn:ietf:params:oauth:grant-type:jwt-bearer`。               |
| `inference.credentials.exchange.encoding`<br />`CMA_INFERENCE__CREDENTIALS__EXCHANGE__ENCODING`                   | `form`       | 服务器对交换请求的编码方式，`form` 或 `multipart`。                                                         |
| `inference.credentials.exchange.expiry`<br />`CMA_INFERENCE__CREDENTIALS__EXCHANGE__EXPIRY`                       | `expires_in` | 服务器读取令牌有效期的位置。`expires_in` 表示取响应的 `expires_in` 与令牌 `exp` 声明中较早的一个。`jwt_exp` 表示仅使用 `exp` 声明。 |
| `inference.credentials.exchange.headers`<br />仅文件                                                                 | 无            | 发往令牌端点的额外请求头，格式为请求头名称到值的映射。请为每个值加引号，与 `inference.headers` 相同。                               |
| `inference.credentials.exchange.ca_file`<br />`CMA_INFERENCE__CREDENTIALS__EXCHANGE__CA_FILE`                     | 无            | PEM 文件的路径，其中包含服务器为令牌端点信任的证书颁发机构。                                                            |

## 单点登录

要设置登录，请参阅[单点登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure)。

| 设置<br />环境变量                          | 默认值     | 描述                                                                                                                                                 |
| ------------------------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `auth.issuers`<br />仅文件               | 无       | 服务器接受其令牌的 OpenID Connect 颁发者。请参阅[颁发者条目](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#issuer-entries)。 |
| `auth.refuse_server_api_key`<br />仅文件 | `false` | 服务器是否在每个路由上拒绝服务器 API 密钥。`true` 需要至少一个颁发者。届时所有需要该密钥的管理 CLI 命令都会失败。                                                                                  |

### 颁发者条目

`auth.issuers` 的每个条目包含以下字段：

| 字段                | 默认值        | 描述                                                                                                                                                                                                                                                                   |
| ----------------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`            | 无          | 必需。日志行中颁发者的标签。服务器还会将其记录为每个资源创建者的一部分。                                                                                                                                                                                                                                 |
| `issuer_url`      | 无          | 必需。颁发者的 URL，与其令牌 `iss` 声明中的值完全一致。服务器从此主机以及颁发者的发现文档中指定的 `jwks_uri` 主机获取签名密钥。如果您的网络限制出站连接，请允许这两个主机。                                                                                                                                                                    |
| `audiences`       | 无          | 必需。令牌的 `aud` 声明可以携带的值。如果应用程序转发其已持有的令牌，请列出每个应用程序的客户端 ID。                                                                                                                                                                                                              |
| `roles`           | 无          | `group` 和 `role` 对的列表。组可以精确匹配，也可以作为模式匹配，其中 `*` 代表任意字符序列，`?` 代表一个字符。角色为 `admin`、`developer` 或 `user`。匹配的最高角色生效。设置 `roles`、`default_role` 或两者。                                                                                                                         |
| `default_role`    | 无          | 其所属组与任何 `roles` 条目都不匹配的调用者的角色。仅当所有能为此受众获取令牌的人都应具有访问权限时才设置此项。                                                                                                                                                                                                         |
| `subject_claim`   | `sub`      | 标识调用者的声明。以 `/` 开头的名称是指向嵌套声明的 JSON Pointer。                                                                                                                                                                                                                           |
| `stable_subject`  | `false`    | 仅当 `subject_claim` 指定 `sub` 或 `oid` 以外的声明时适用。设置为 `true` 以确认您的提供商从不更改某人在该声明中的值，也从不将其分配给其他人。服务器不会对此进行检查。为 `false` 时，[智能体令牌](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#agent-identity)会省略 `actor_sub` 和 `actor_iss` 声明。 |
| `groups_claim`    | `groups`   | 列出调用者所属组的声明。以 `/` 开头的名称是指向嵌套声明的 JSON Pointer。                                                                                                                                                                                                                        |
| `jwks_file`       | 无          | JSON Web Key Set 文件的路径。服务器从中读取签名密钥，且从不联系颁发者。                                                                                                                                                                                                                         |
| `login_client_id` | 无          | [终端登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#let-developers-log-in-from-a-terminal)所使用应用程序的客户端 ID。只能有一个颁发者设置此项。                                                                                                                |
| `login_scopes`    | `[openid]` | 登录请求的作用域。此字段需要 `login_client_id`。                                                                                                                                                                                                                                    |

## 密钥管理器

要设置存储，请参阅[密钥管理器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#secret-manager)。

| 设置<br />环境变量                                                | 默认值 | 描述                                                                                                                                                                              |
| ----------------------------------------------------------- | --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `secret_store.type`<br />`CMA_SECRET_STORE__TYPE`           | 无   | 保存 vault 凭据的密钥管理器。可选值为 `aws_secrets_manager`、`gcp_secret_manager`、`azure_key_vault` 和 `hashicorp_vault`。如果未设置，vault 凭据保留在数据库中。                                                  |
| `secret_store.prefix`<br />`CMA_SECRET_STORE__PREFIX`       | 无   | 使用 `secret_store.type` 时必需。服务器创建的每个密钥名称的开头部分。对于 `aws_secrets_manager` 和 `hashicorp_vault`，它必须以 `/` 结尾；对于 `gcp_secret_manager`，必须以 `-` 或 `_` 结尾；对于 `azure_key_vault`，必须以 `-` 结尾。 |
| `secret_store.region`<br />`CMA_SECRET_STORE__REGION`       | 无   | 仅限 `aws_secrets_manager`，且在该情况下必需。保存密钥的 AWS 区域。                                                                                                                                 |
| `secret_store.project`<br />`CMA_SECRET_STORE__PROJECT`     | 无   | 仅限 `gcp_secret_manager`，且在该情况下必需。保存密钥的 Google Cloud 项目的 ID。                                                                                                                     |
| `secret_store.vault_url`<br />`CMA_SECRET_STORE__VAULT_URL` | 无   | 仅限 `azure_key_vault`，且在该情况下必需。密钥保管库的 URL，例如 `https://my-vault.vault.azure.net`。                                                                                                 |

### HashiCorp Vault

这些设置仅在 `secret_store.type: hashicorp_vault` 时适用。只能设置一种登录方法：`token_file`、`kubernetes` 或 `approle`。

| 设置<br />环境变量                                                                                                         | 默认值                                                   | 描述                                                     |
| -------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------ |
| `secret_store.address`<br />`CMA_SECRET_STORE__ADDRESS`                                                              | 无                                                     | 必需。Vault 服务器的 `https://` URL。                          |
| `secret_store.namespace`<br />`CMA_SECRET_STORE__NAMESPACE`                                                          | 无                                                     | 包含挂载点和登录方法的 Vault Enterprise 命名空间。                     |
| `secret_store.mount`<br />`CMA_SECRET_STORE__MOUNT`                                                                  | `secret`                                              | KV 版本 2 密钥引擎的路径。                                       |
| `secret_store.ca_cert_file`<br />`CMA_SECRET_STORE__CA_CERT_FILE`                                                    | 无                                                     | PEM 文件的路径，其中包含签署 Vault 证书的证书颁发机构。                      |
| `secret_store.token_file`<br />`CMA_SECRET_STORE__TOKEN_FILE`                                                        | 无                                                     | 包含 Vault 令牌的文件路径，该令牌由另一个进程（例如 Vault Agent）保持更新。        |
| `secret_store.kubernetes.role`<br />`CMA_SECRET_STORE__KUBERNETES__ROLE`                                             | 无                                                     | 使用 `kubernetes` 时必需。服务器登录时使用的 Vault 角色。请将其绑定到服务器的服务账户。 |
| `secret_store.kubernetes.mount_path`<br />`CMA_SECRET_STORE__KUBERNETES__MOUNT_PATH`                                 | `kubernetes`                                          | Vault 挂载 Kubernetes 登录方法的位置。                           |
| `secret_store.kubernetes.service_account_token_file`<br />`CMA_SECRET_STORE__KUBERNETES__SERVICE_ACCOUNT_TOKEN_FILE` | `/var/run/secrets/kubernetes.io/serviceaccount/token` | 服务器每次登录时出示的服务账户令牌的路径。                                  |
| `secret_store.approle.role_id`<br />`CMA_SECRET_STORE__APPROLE__ROLE_ID`                                             | 无                                                     | 使用 `approle` 时必需。服务器登录时使用的 AppRole 角色的 ID。             |
| `secret_store.approle.secret_id_file`<br />`CMA_SECRET_STORE__APPROLE__SECRET_ID_FILE`                               | 无                                                     | 使用 `approle` 时必需。包含该角色 `secret_id` 的文件路径。              |
| `secret_store.approle.mount_path`<br />`CMA_SECRET_STORE__APPROLE__MOUNT_PATH`                                       | `approle`                                             | Vault 挂载 AppRole 登录方法的位置。                              |

## 智能体身份

要设置智能体令牌，请参阅[智能体身份](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#agent-identity)。此表中的任何设置都会启用该功能。此时服务器需要 `agent_identity.issuer`，以及签名密钥的唯一一个位置：`key_file`、`key_secret` 或 `kms_key`。

| 设置<br />环境变量                                                                        | 默认值                              | 描述                                                                                                                                                 |
| ----------------------------------------------------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `agent_identity.issuer`<br />`CMA_AGENT_IDENTITY__ISSUER`                           | 无                                | 智能体令牌作为其 `iss` 声明携带的 `https://` URL。                                                                                                               |
| `agent_identity.key_file`<br />`CMA_AGENT_IDENTITY__KEY_FILE`                       | 无                                | 签署令牌的 PEM 私钥的路径。                                                                                                                                   |
| `agent_identity.key_secret`<br />`CMA_AGENT_IDENTITY__KEY_SECRET`                   | 无                                | 在您的密钥管理器中保存 PEM 私钥的密钥名称，不含 `secret_store.prefix`。                                                                                                  |
| `agent_identity.kms_key.provider`<br />`CMA_AGENT_IDENTITY__KMS_KEY__PROVIDER`      | 无                                | 保存签名密钥的密钥管理服务，`aws_kms` 或 `gcp_kms`。                                                                                                               |
| `agent_identity.kms_key.key`<br />`CMA_AGENT_IDENTITY__KMS_KEY__KEY`                | 无                                | 使用 `kms_key.provider` 时必需。对于 `aws_kms` 为密钥的 ARN，对于 `gcp_kms` 为某个密钥版本的资源名称。                                                                         |
| `agent_identity.published_key_files`<br />`CMA_AGENT_IDENTITY__PUBLISHED_KEY_FILES` | 无                                | 密钥集中额外 PEM 公钥的路径。服务器会发布这些密钥，但从不使用它们进行签名。轮换签名密钥时请使用此设置。                                                                                             |
| `agent_identity.token_lifetime`<br />`CMA_AGENT_IDENTITY__TOKEN_LIFETIME`           | `5m`                             | 从令牌的 `iat` 声明到其 `exp` 声明的时间。已发送的令牌在其 `exp` 之前保持有效。最小值为 `limits.mcp_call_timeout` 加 `1m`，最大值为 `15m`。                                                |
| `agent_identity.jwks_uri`<br />`CMA_AGENT_IDENTITY__JWKS_URI`                       | `<issuer>/.well-known/jwks.json` | 颁发者的发现文档为密钥集提供的 URL。仅当您将密钥集托管在颁发者 URL 之外的位置时才设置此项。                                                                                                 |
| `agent_identity.direct.audiences`<br />仅文件                                          | 无                                | 可以获得令牌的接收方。每个条目包含一个 `audience` URL 和 `urls`，即该接收方的 MCP 服务器的 URL 前缀。                                                                                |
| `agent_identity.actor.enabled`<br />`CMA_AGENT_IDENTITY__ACTOR__ENABLED`            | `true`                           | 服务器是否将 `actor_sub` 和 `actor_iss` 声明添加到工具调用的令牌中。`actor_sub` 是创建会话的调用者的主体。`actor_iss` 是调用者登录所通过的 `auth.issuers` 条目的 `issuer_url`。这两个声明都不表示调用者批准了该调用。 |

## 出口网关

要设置网关，请参阅[环境变量凭据](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#environment-variable-credentials)。仅在 `egress.gateway_enabled: true` 时设置此表中的其他设置。否则，如果设置了其中任何一项，服务器将不会启动。

| 设置<br />环境变量                                                            | 默认值             | 描述                                                                                                                                  |
| ----------------------------------------------------------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `egress.gateway_enabled`<br />`CMA_EGRESS__GATEWAY_ENABLED`             | `false`         | 服务器是否与 “egress gateway”（出口网关）配合工作并接受 `environment_variable` vault 凭据。                                                               |
| `egress.allowed_hostnames`<br />`CMA_EGRESS__ALLOWED_HOSTNAMES`         | 无               | 沙箱可以通过网关访问的确切公共主机名。不带端口的条目（例如 `api.example.com`）允许 `https` URL 使用端口 443、`http` URL 使用端口 80。带端口的条目（例如 `api.example.com:8443`）仅允许该端口。 |
| `egress.swap_header_names`<br />`CMA_EGRESS__SWAP_HEADER_NAMES`         | `Authorization` | 网关可以放入凭据值的请求头。您自己的列表会替换默认值，因此请在其中包含 `Authorization`。                                                                                |
| `egress.gateway_listen`<br />`CMA_EGRESS__GATEWAY_LISTEN`               | `:8410`         | 服务器第二个监听器的地址，该监听器仅为网关提供服务。默认值在所有接口上监听。                                                                                              |
| `egress.gateway_key_file`<br />`CMA_EGRESS__GATEWAY_KEY_FILE`           | 无               | 必需。包含网关向服务器出示的密钥的文件路径。                                                                                                              |
| `egress.ca_cert_file`<br />`CMA_EGRESS__CA_CERT_FILE`                   | 无               | 必需。网关证书颁发机构的 PEM 证书的路径。                                                                                                             |
| `egress.gateway_proxy_address`<br />`CMA_EGRESS__GATEWAY_PROXY_ADDRESS` | 无               | 必需。沙箱访问网关代理端口所用的 `host:port`。                                                                                                       |

## MCP 服务器

请参阅[MCP 服务器需要允许列表条目](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#mcp-servers-need-an-allowlist-entry)。

| 设置<br />环境变量                                              | 默认值                | 描述                                             |
| --------------------------------------------------------- | ------------------ | ---------------------------------------------- |
| `mcp.allowed_hostnames`<br />`CMA_MCP__ALLOWED_HOSTNAMES` | 无                  | 服务器可以连接的 MCP 服务器的确切主机名。如果未设置，服务器不连接任何 MCP 服务器。 |
| `mcp.private_hostnames`<br />仅文件                          | 无                  | `mcp.allowed_hostnames` 中可以解析为私有地址的主机名。        |
| `mcp.max_result_bytes`<br />`CMA_MCP__MAX_RESULT_BYTES`   | `102400` (100 KiB) | 模型可以看到的单个 MCP 工具结果的最大字节数。最小值为 `1024`。          |

## Git 仓库

请参阅[Git 仓库需要允许列表条目](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#git-repositories-need-an-allowlist-entry)。

| 设置<br />环境变量                                                                | 默认值                     | 描述                                                                                                                         |
| --------------------------------------------------------------------------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `git.allowed_hostnames`<br />`CMA_GIT__ALLOWED_HOSTNAMES`                   | 无                       | 会话的仓库可以位于的 Git 主机。主机名精确匹配，不支持通配符，且条目不涵盖其子域名。条目允许端口 443，或其指定的那一个端口，例如 `git.example.com:8443`。如果未设置此项，服务器不接受任何仓库。            |
| `git.private_hostnames`<br />仅文件                                            | 无                       | `git.allowed_hostnames` 中解析为私有地址的主机名，不带端口写入。对于任何其他主机名，服务器仅连接到其公共地址。                                                        |
| `git.allowed_push_refs`<br />`CMA_GIT__ALLOWED_PUSH_REFS`                   | 无                       | 智能体可以推送到的分支，每个都位于 `refs/heads/` 下。以 `/` 结尾的条目允许其下的所有分支。其他任何条目允许该单个分支。如果未设置此项，智能体无法推送。                                      |
| `git.commit_author_name`<br />`CMA_GIT__COMMIT_AUTHOR_NAME`                 | `Claude`                | 智能体提交上的作者名称。请与 `git.commit_author_email` 一起设置。                                                                             |
| `git.commit_author_email`<br />`CMA_GIT__COMMIT_AUTHOR_EMAIL`               | `noreply@anthropic.com` | 智能体提交上的作者电子邮件地址。请与 `git.commit_author_name` 一起设置。                                                                          |
| `git.max_concurrent_transfers`<br />`CMA_GIT__MAX_CONCURRENT_TRANSFERS`     | `32`                    | 单个服务器同时承载的最大 Git 请求数。最小值为 `1`。                                                                                             |
| `git.max_session_transfer_bytes`<br />`CMA_GIT__MAX_SESSION_TRANSFER_BYTES` | `8589934592` (8 GiB)    | 单个服务器为单个会话承载的最大 Git 流量（以字节计），上传和下载合并计算。每次沙箱接管会话时，计数都会从零重新开始。达到上限时，服务器会以 `403` 拒绝该会话后续的 Git 请求。`0` 表示无限制。否则，最小值为 `1048576`。 |
| `git.max_push_bytes`<br />`CMA_GIT__MAX_PUSH_BYTES`                         | `2147483648` (2 GiB)    | 单次推送可以发送的最大字节数。最小值为 `1048576`。                                                                                             |

以下限制适用于 Git 仓库：

* **协议：** 服务器不支持 SSH 或 Git LFS，并且会拒绝重定向，例如针对已重命名仓库的重定向。
* **令牌：** Git 主机必须接受该令牌作为 HTTP Basic 身份验证的密码，用户名为 `x-access-token`。
* **URL：** 仓库 URL 只有一个所有者和一个仓库名称，因此您无法使用 GitLab 子组中的仓库。
* **主机：** 对于 `git.allowed_hostnames` 未列出的主机上的 URL，服务器会返回 `400`。错误消息会说明 URL 必须位于 `github.com` 上，即使您列出了其他主机。
* **推送：** 服务器始终拒绝删除分支的推送，或更改分支以外任何内容（例如标签）的推送。

## Web 工具和技能

请参阅[Web 工具需要服务器设置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#web-tools-need-a-server-setting)。

| 设置<br />环境变量                                                            | 默认值     | 描述                                                                           |
| ----------------------------------------------------------------------- | ------- | ---------------------------------------------------------------------------- |
| `web_tools.enabled`<br />`CMA_WEB_TOOLS__ENABLED`                       | `false` | 智能体是否可以使用 `web_search` 和 `web_fetch` 工具。如果您设置了其他 `web_tools` 设置，默认值为 `true`。 |
| `web_tools.allowed_domains`<br />`CMA_WEB_TOOLS__ALLOWED_DOMAINS`       | 无       | 智能体通过 Web 工具唯一可以读取的域名。一个条目涵盖其子域名。                                            |
| `web_tools.blocked_domains`<br />`CMA_WEB_TOOLS__BLOCKED_DOMAINS`       | 无       | 任何智能体都无法通过 Web 工具读取的域名。设置此项或 `web_tools.allowed_domains`，不要同时设置。             |
| `web_tools.max_content_tokens`<br />`CMA_WEB_TOOLS__MAX_CONTENT_TOKENS` | 无       | `web_fetch` 返回给模型的单个抓取页面的最大令牌数。                                              |
| `skills.max_name_chars`<br />`CMA_SKILLS__MAX_NAME_CHARS`               | `64`    | `SKILL.md` 文件的 `name` 的最大长度。最大值为 `255`。                                      |
| `skills.max_description_chars`<br />`CMA_SKILLS__MAX_DESCRIPTION_CHARS` | `1024`  | `SKILL.md` 文件的 `description` 的最大长度。                                          |

## 限制、超时和调度

| 设置<br />环境变量                                                                    | 默认值         | 描述                                                                                                                                     |
| ------------------------------------------------------------------------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `limits.max_active_sessions`<br />`CMA_LIMITS__MAX_ACTIVE_SESSIONS`             | `64`        | 仅限 `store.backend: memory`。同时运行轮次的最大会话数。`0` 表示无限制。                                                                                     |
| `limits.max_live_subagent_threads`<br />`CMA_LIMITS__MAX_LIVE_SUBAGENT_THREADS` | `25`        | 单个会话同时可持有的最大子智能体线程数。线程在被归档或终止之前都会计入，因此空闲线程也会计入。达到上限时，智能体无法再启动其他子智能体。最小值为 `1`，最大值为 `4000`。                                              |
| `scheduler.max_concurrent_turns`<br />`CMA_SCHEDULER__MAX_CONCURRENT_TURNS`     | `64`        | 仅限 `store.backend: postgres`。单个服务器同时运行的最大轮次数。最小值为 `1`。服务器会额外保留仅供子智能体轮次使用的槽位。其数量为此值的四分之一，且至少为 `4`。使用默认值时，单个服务器可以运行 64 个轮次外加 16 个子智能体轮次。 |
| `scheduler.claim_poll_interval`<br />`CMA_SCHEDULER__CLAIM_POLL_INTERVAL`       | `1s`        | 仅限 `store.backend: postgres`。有空闲轮次槽位的服务器在数据库中查找等待中工作的频率。最小值为 `50ms`。                                                                   |
| `scheduler.sandbox_claim`<br />`CMA_SCHEDULER__SANDBOX_CLAIM`                   | `immediate` | 工作进程可以启动会话沙箱的时机。`immediate` 表示会话一经创建即可启动，因此首次工具调用的等待时间最短。`tool_call` 表示在会话首次进行需要沙箱的工具调用时启动，因此不进行此类调用的会话不会使用沙箱。                         |
| `limits.model_call_timeout`<br />`CMA_LIMITS__MODEL_CALL_TIMEOUT`               | `20m`       | 单次模型调用（包括重试）在服务器取消之前可以运行的时长。                                                                                                           |
| `limits.stream_idle_timeout`<br />`CMA_LIMITS__STREAM_IDLE_TIMEOUT`             | `5m`        | 模型调用的响应在服务器放弃该次尝试之前可以不传递任何内容的时长。                                                                                                       |
| `limits.mcp_call_timeout`<br />`CMA_LIMITS__MCP_CALL_TIMEOUT`                   | `4m`        | 单次 MCP 工具调用在服务器放弃之前可以运行的时长。                                                                                                            |

## 模型和压缩

| 设置<br />环境变量                   | 默认值              | 描述                                                                                                                                          |
| ------------------------------ | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `models.settings`<br />仅文件     | 无                | 对服务器内置模型目录的更改，以模型 ID 为键。请参阅[模型条目](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#model-entries)。 |
| `compaction.triggers`<br />仅文件 | 约为每个模型上下文窗口的 90% | 某个模型上的轮次对其历史记录进行 “compaction”（压缩）时的上下文大小（以令牌计），以模型 ID 为键。最小值为 `50000`。每个模型都必须支持压缩。                                                          |

### 模型条目

`models.settings` 的每个条目会更改目录中的一个模型或添加一个模型。对于目录中的模型，您省略的字段保留其内置值。

| 字段                             | 描述                                                                                                    |
| ------------------------------ | ----------------------------------------------------------------------------------------------------- |
| `fast_supported`               | 模型是否接受 `speed: "fast"`。对于您添加的模型是必需的。                                                                  |
| `inference_geo_supported`      | 模型是否接受 `inference_geo`。                                                                               |
| `forced_tool_choice_supported` | 模型是否接受强制其调用某一个工具的请求。服务器仅在运行已获批准的 `web_search` 或 `web_fetch` 调用时才会发送此类请求。如果为 `false`，服务器从不在此模型上强制使用工具。 |
| `effort_supported`             | 模型是否接受努力级别。                                                                                           |
| `effort_levels`                | 模型接受的努力级别，取自 `low`、`medium`、`high`、`xhigh` 和 `max`。在没有内置级别的模型上使用 `effort_supported: true` 时必需。        |
| `effort_default`               | 其智能体未指定努力级别的会话所用的努力级别。它必须是 `effort_levels` 之一。在没有内置默认值的模型上使用 `effort_supported: true` 时必需。            |
| `thinking`                     | 服务器请求的思考模式，`enabled` 或 `adaptive`。                                                                    |
| `thinking_budget`              | 服务器在 `thinking: enabled` 时发送的思考预算（以令牌计）。                                                              |
| `max_tokens`                   | 每次模型调用的 `max_tokens` 值。                                                                               |

## 服务器上的控制台

| 设置<br />环境变量                              | 默认值    | 描述                                                                                         |
| ----------------------------------------- | ------ | ------------------------------------------------------------------------------------------ |
| `console.serve`<br />`CMA_CONSOLE__SERVE` | `true` | 是否提供[服务器上的控制台](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console)。 |
