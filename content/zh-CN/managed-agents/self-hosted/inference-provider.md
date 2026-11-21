---
title: 推理提供商
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider
description: 将模型调用从您自托管的 Claude Managed Agents 服务器发送到 Amazon Bedrock、Claude Models on Google Cloud (Google managed)、Microsoft Foundry 或您自己运行的模型网关，而不是 Claude API。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

默认情况下，您的服务器会将模型调用发送到 Claude API。本页介绍如何将它们发送到云提供商，或发送到您自己运行的模型网关。要更改提供商：

1. 将本页中您的提供商对应部分的示例放入服务器的[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)中。
2. 按照同一部分所示，向服务器提供该提供商的凭据。
3. 按照[设置的存放位置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)所示重启服务器。在 Docker Compose 上，如果第 2 步更改了 `.env` 或 `docker-compose.yml`，请改为运行 `docker compose up -d`，因为重启不会重新读取它们。
4. [检查提供商](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider#check-the-provider)。

`inference.provider` 设置可取以下值：

| `inference.provider` | 模型调用发送到 | 服务器的身份验证方式 |
| ----- | ----- | ----- |
| `anthropic`（默认） | Claude API | `ANTHROPIC_API_KEY` |
| `bedrock` | Amazon Bedrock | 您的 AWS 凭据 |
| `vertex` | Claude Models on Google Cloud (Google managed) | Google Application Default Credentials |
| `foundry` | Microsoft Foundry | Foundry API 密钥，或 Microsoft Entra ID |
| `gateway` | 您自己运行的模型网关 | 您的网关接受的密钥或令牌 |

在除 `anthropic` 之外的所有取值下，服务器都不需要 Claude API 密钥。它会忽略 `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN` 和 `ANTHROPIC_BASE_URL`。唯一的例外是您让服务器从这些变量之一读取其密钥的[模型网关](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider#model-gateway)。

智能体使用模型的 Claude API ID 来指定模型，例如 `claude-opus-5-5`。服务器会转换该 ID，因此同一个智能体定义无需修改即可在任何提供该模型的提供商下运行。

某些工具和智能体设置取决于提供商。请参阅[取决于推理提供商的差异](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#differences-that-depend-on-your-inference-provider)。

## 向服务器提供云凭据

云提供商的凭据如何到达服务器取决于您的安装方式：

* **Docker Compose：** [将每个变量和文件传递给服务器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#pass-a-credential-to-the-compose-stack)。
* **Kubernetes：** 服务器 chart 不传递任何 AWS、Google Cloud 或 Azure 变量，因此请使用您云平台的工作负载身份。创建一个带有您云平台工作负载身份注解的服务账号，并将 `server.serviceAccountName` 设置为其名称。该注解在 chart 的 `values.yaml` 中有说明。在 Azure Kubernetes Service 上，还需设置 `server.azureWorkloadIdentity: true`。

## Amazon Bedrock

```yaml
inference:
  provider: bedrock
  region: us-east-1
  #model_id_prefix: global.anthropic.
  #model_id_overrides:
  #  claude-opus-5-5: us.anthropic.claude-opus-5-5
```

服务器使用 AWS SDK 默认凭据链在主机上找到的任何凭据为其调用签名。这可以是任务角色或 pod 身份。在工作站上，可以是 `AWS_PROFILE` 或 `AWS_ACCESS_KEY_ID`。

在账户中启用您使用的模型。服务器用于签名的 AWS 身份需要以下权限：

* 对您使用的模型和推理配置文件拥有 `bedrock:InvokeModel` 和 `bedrock:InvokeModelWithResponseStream` 权限。
* 对资源 `*` 拥有 `bedrock:ListFoundationModels` 和 `bedrock:ListInferenceProfiles` 权限。服务器会读取 Amazon Bedrock 的模型列表，以便智能体可以使用比您服务器版本更新的模型。要跳过此项，请设置 `inference.list_models: false`。

默认情况下，服务器会在每个模型 ID 前加上 `us.anthropic.`，这指定了一个 “cross-region inference profile”（跨区域推理配置文件）。要使用其他系列的配置文件，请取消注释 `model_id_prefix`。

## Claude Models on Google Cloud (Google managed)

```yaml
inference:
  provider: vertex
  region: us-east5
  project_id: my-project
  #model_id_overrides:
  #  claude-haiku-4-5: claude-haiku-4-5@20251001
```

`region` 是必需的。它是提供您所用模型的 Google Cloud’s Agent Platform 位置，或 `global`。`project_id` 是发起调用并计费的项目。

服务器使用 Google Application Default Credentials：即 `GOOGLE_APPLICATION_CREDENTIALS` 指定的文件，或 Compute Engine、Cloud Run 或 GKE 附加的服务账号。该身份需要在项目上拥有 `roles/aiplatform.user` 角色。项目需要启用 `aiplatform.googleapis.com` API 以及您使用的 Claude 模型。

## Microsoft Foundry

```yaml
inference:
  provider: foundry
  foundry:
    resource: my-resource
  #model_id_overrides:
  #  claude-opus-5-5: team-a-opus
```

`resource` 是您的 Foundry 资源名称，即其主机名的第一部分：`<resource>.services.ai.azure.com`。

凭据来自服务器的环境：

* **Foundry API 密钥：** 设置 `ANTHROPIC_FOUNDRY_API_KEY`。在 Compose 堆栈中，[将该变量传递给服务器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#pass-a-credential-to-the-compose-stack)。在 Kubernetes 上，将其放入 `cma-server-upstream` Secret 中。如果您的服务器 values 文件中设置了 `earlierNames: true`，则该 Secret 使用另一个名称。请在 [Kubernetes：您的对象名称](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#object-names)中查找。
* **Microsoft Entra ID：** 不设置 `ANTHROPIC_FOUNDRY_API_KEY`。服务器随后会为服务主体、Azure Kubernetes Service 上的工作负载身份或托管身份获取令牌。该身份需要在 Foundry 资源上拥有允许调用其模型部署的 Azure 角色。对于服务主体，请设置 `AZURE_TENANT_ID`、`AZURE_CLIENT_ID` 和 `AZURE_CLIENT_SECRET`。

对 Foundry 的请求会指定一个部署，服务器将模型 ID 作为部署名称发送。如果某个部署使用其他名称，请使用 [`model_id_overrides`](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider#map-a-model-id-that-differs) 进行映射。

## 映射不同的模型 ID

您的云账户中某个模型的 ID 可能与服务器发送的 ID 不同。在这种情况下，请取消注释该提供商示例中的 `model_id_overrides` 行，并将 Claude API ID 映射到您账户的 ID。两个 ID 必须指向同一个模型，因为服务器会针对智能体指定的模型构建每个请求。

## 模型网关

首先确认您的 “model gateway”（模型网关）接受哪种请求：

* **与 Claude API 相同的请求，使用一个长期凭据：** 保留默认提供商。将 `ANTHROPIC_BASE_URL` 设置为网关的地址，并将 `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` 设置为网关所需的凭据。
* **与 Claude API 相同的请求，但使用短期令牌、在其他标头中传递的凭据或其自己的模型 ID：** 设置 `inference.provider: gateway` 并使用 `format: anthropic-messages`。
* **Amazon Bedrock 的 InvokeModel 格式：** 设置 `inference.provider: gateway` 并使用 `format: bedrock-invoke`。
* **Claude Models on Google Cloud (Google managed) 的 `rawPredict` 格式：** 设置 `inference.provider: gateway` 并使用 `format: vertex-raw-predict`。

此示例设置 `inference.provider: gateway` 并使用 `format: bedrock-invoke`，适用于接受一个长期密钥的网关。将密钥放入环境变量中，并填写变量名称，切勿填写其值：

```yaml
inference:
  provider: gateway
  gateway:
    url: "https://llm-gateway.example.com/v1/model/{model}/invoke"
    format: bedrock-invoke
    # Quote every model_map value.
    model_map:
      claude-opus-5-5: "us.anthropic.claude-opus-5-5"
      claude-sonnet-5: "us.anthropic.claude-sonnet-5"
  credentials:
    supplier: env
    env: { var: LLM_GATEWAY_KEY }
    send_as: { header: x-api-key }
```

* `url` 是网关的模型路由。`{model}` 标记模型 ID 的位置。该 URL 必须使用 `https://` 并以 `/invoke` 结尾。

* `model_map` 将智能体可指定的每个模型 ID 映射到网关所需的 ID。服务器会拒绝没有对应条目的模型。

* `send_as` 指定携带密钥的标头。如果不设置，服务器会发送 `Authorization: Bearer <key>`。

* 在 Compose 堆栈中，[将该变量传递](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#pass-a-credential-to-the-compose-stack)给服务器。

* 在 Kubernetes 上，服务器 chart 无法传递您自己的变量。但它确实会传递 `ANTHROPIC_AUTH_TOKEN`，该变量从 `cma-server-upstream` Secret 中读取，因此请使用该名称：

  1. 设置 `env: { var: ANTHROPIC_AUTH_TOKEN }`。
  2. 将网关的密钥以 `ANTHROPIC_AUTH_TOKEN` 为名称放入 `cma-server-upstream` Secret 中。如果您的服务器 values 文件中设置了 `earlierNames: true`，则该 Secret 使用另一个名称。请在 [Kubernetes：您的对象名称](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#object-names)中查找。

对于 `format: anthropic-messages`，请将 `gateway` 块替换为以下内容：

```yaml
inference:
  gateway:
    url: "https://llm-gateway.example.com/v1/messages"
    format: anthropic-messages
    # Quote every model_map value.
    model_map:
      claude-opus-5-5: "corp-claude-opus"
      claude-sonnet-5: "corp-claude-sonnet"
```

此处，`url` 是网关 Messages 路由的完整 URL，不含 `{model}`。

对于 `format: vertex-raw-predict`，请将 `gateway` 块替换为以下内容：

```yaml
inference:
  gateway:
    url: "https://llm-gateway.example.com/v1/projects/my-project/locations/global/publishers/anthropic/models/{model}:rawPredict"
    format: vertex-raw-predict
    # Quote every model_map value.
    model_map:
      claude-opus-5-5: "claude-opus-5-5"
      claude-sonnet-5: "claude-sonnet-5"
```

此处，`url` 是网关的 `rawPredict` 路由，且必须以 `/{model}:rawPredict` 结尾。`model_map` 的值不能包含冒号。`credentials` 块保持不变。服务器发送的是您网关自己的凭据，不会获取任何 Google 凭据。

[配置参考](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#model-gateway)列出了网关的其他设置（例如 `streaming`），以及 `inference.headers` 设置。

### 短期令牌

如果您的网关接受短期令牌，请将 `credentials` 块替换为以下内容：

```yaml
inference:
  credentials:
    supplier: oauth_exchange
    assertion:
      url: "https://app.example.com/assertion"
      auth: { type: mtls, cert_file: /etc/callback/client.pem, key_file: /etc/callback/client.key }
    exchange:
      token_url: "https://login.example.com/oauth2/token"
      grant_type: "urn:ietf:params:oauth:grant-type:jwt-bearer"
```

服务器随后分两步获取每个令牌。首先，它调用您构建的 HTTP 回调。该回调返回一个 “assertion”（断言），即一份已签名的声明，通常是 “JSON Web Token”（JSON Web 令牌），即 JWT。然后，服务器在您的 OAuth 令牌端点用该断言换取访问令牌，并将其发送给网关。

* `auth` 是服务器向回调进行身份验证的方式。示例使用客户端证书。要使用服务器从文件读取的 bearer 令牌，请设置 `auth: { type: bearer_file, token_file: /etc/callback/token }`。
* `exchange` 指定您的 OAuth 令牌端点。服务器只向该端点发送 `grant_type` 和断言，不包含客户端 ID 或密钥。
* 如果您的令牌端点需要更多信息，请省略 `exchange`。让您的回调自行获取访问令牌，并在 `assertion` 字段中返回它，服务器随后会将其发送给网关。
* 在 Kubernetes 上，将证书文件放入一个 Secret 中，在 chart 的 `server.extensionFiles.secretName` 中指定其名称，并将 `cert_file` 和 `key_file` 指向 `/run/cma-server/extension-files/`。

服务器向回调发送一个 `POST` 请求，请求体如下：

```json
{
  "version": 1,
  "purpose": "inference",
  "session": {
    "id": "sesn_011CZkZAtmR3yMPDzynEDxu7",
    "agent_id": "agent_011CYm1BLqPXpQRk5khsSXrs",
    "environment_id": "env_01595EKxaaTTGwwY3kyXdtbs"
  },
  "principal": {
    "issuer": "https://login.example.com/oauth2/default",
    "subject": "00u1a2b3c4d5e6f7g8h9"
  },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```

`principal` 是创建会话的调用方。对于[单点登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#single-sign-on)调用方，`issuer` 是该调用方的 `issuer_url`，`subject` 是该调用方的 subject。对于服务器 API 密钥，`principal` 为 `{"issuer": "cma", "subject": "server"}`。让回调忽略它不认识的字段。

回调以状态 `200` 和如下响应体作答：

```json
{ "assertion": "<signed JWT>", "reuse": "session" }
```

`reuse` 指定哪些会话可以使用该令牌。使用默认值 `session` 时，只有请求中的会话可以使用，适用于代表单个用户的令牌。使用 `deployment` 时，各会话共享该令牌，因此只有当令牌代表您的组织而非任何个人时，才返回 `deployment`。

如果回调或令牌端点失败，该轮次会以类型为 `model_request_failed_error` 的 `session.error` 事件结束，而会话仍然可用。原因记录在服务器日志中包含 `credential supplier failed` 的行上。

如果令牌端点的回复中没有 `expires_in`，且令牌没有 `exp` 声明，服务器会发送该令牌但不会存储它。随后它会在每次模型调用时请求新的令牌。

有关其他设置（例如 `renew_before` 和 `ca_file`），请参阅配置参考中的[模型网关设置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#model-gateway)表和[短期令牌设置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#short-lived-gateway-tokens)表。

### Microsoft Entra ID 令牌

如果您的网关根据其自己的应用注册来检查 Microsoft Entra ID 令牌，请将 `credentials` 块替换为以下内容：

```yaml
inference:
  credentials:
    supplier: azure
    azure: { scope: "api://11111111-2222-3333-4444-555555555555/.default" }
```

`scope` 是网关应用注册的应用程序 ID URI 或应用程序 ID，后跟 `/.default`。只能指定网关自己的注册。针对其他资源的令牌会让运行网关的任何人都能在那里以服务器的身份操作。

该令牌颁发给服务器自己的 Azure 身份，其设置方式与 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider#microsoft-foundry) 中 Microsoft Entra ID 的设置相同。如果该注册需要应用角色，请在启动服务器之前将该角色分配给该身份。之后的分配只有在服务器获取新令牌时才会生效。

该令牌指定的是该身份而非任何个人，因此需要知道是哪个人在调用的网关需要使用 `oauth_exchange`。在依赖此提供方之前，请使用您的服务器将要使用的身份，通过您的网关运行一个轮次进行验证。

## 检查提供商

更改设置并重启服务器后，在服务器日志中查找以下这一行。它表示服务器已找到该提供商的凭据：

```text wrap
level=INFO msg="upstream credential resolved" provider=bedrock
```

这一行并不能证明您的账户已启用某个模型。缺失的模型会在首次使用它的轮次中以 `session.error` 的形式出现。

您在 `ANTHROPIC_BASE_URL` 中指定的网关可能会拒绝[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)字段或标头。如果每个轮次都因此失败，请在[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)中设置 `inference.model_fallback: false`。
