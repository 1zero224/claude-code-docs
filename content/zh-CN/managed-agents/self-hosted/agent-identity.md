---
title: 智能体身份和凭据
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity
description: 让您自托管的 Claude Managed Agents 服务器签发一个短期令牌来标识每个智能体，将 vault 凭据保存在您的密钥管理器中，并借助出站网关让机密不进入沙箱。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

本页介绍如何让您服务器之外的系统信任智能体，以及将您的智能体使用的凭据保存在何处：

* [智能体身份](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#agent-identity)：让 MCP 服务器或网关为每个智能体授予各自的访问权限。服务器签发一个标识该智能体的短期令牌。
* [密钥管理器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#secret-manager)：服务器将 vault 凭据保存在 AWS Secrets Manager、Google Cloud Secret Manager、HashiCorp Vault 或 Azure Key Vault 中。
* [环境变量凭据](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#environment-variable-credentials)：让智能体的命令调用某个服务，而沙箱只持有一个占位符。出站网关将真实值放入智能体的请求中。

## 智能体身份

您的服务器可以签发一个标识某个智能体的短期 “JSON Web Token”（JSON Web 令牌），即 JWT，并随该智能体的 MCP 调用一起发送。接收方（MCP 服务器或网关）会根据您发布的公钥检查该令牌，然后可以为每个智能体授予各自的访问权限。

### 设置签发方

1. **选择签发方 URL。** 您将在第 3 步中在此 URL 发布您的公钥，接收方从该 URL 获取公钥。静态 Web 主机即可满足要求。不要使用您服务器的地址，因为服务器不会在接收方获取公钥的路径下提供这些密钥。请使用一个不带路径或末尾斜杠的 `https://` URL，并使用您可以长期保有的主机名。

2. **创建签名密钥。** 在解压后的归档的顶层运行以下命令：

   ```bash
   mkdir -m 700 agent-identity
   openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out agent-identity/signing-key.pem
   ```

3. **发布公钥。** 此命令会将公钥写入 `issuer-files/.well-known/`。请于签发方主机名的根路径下提供 `issuer-files`，内容类型为 `application/json`。

   ```bash
   CMA_AGENT_IDENTITY__ISSUER=https://agents.example.com \
   CMA_AGENT_IDENTITY__KEY_FILE=./agent-identity/signing-key.pem \
     ./cma-server export-issuer -out issuer-files
   ```

4. **将密钥提供给服务器。**

   <Tabs>
     <Tab title="Docker Compose">
       1. 如果归档中有 `compose` 目录，请将密钥所在的目录移入其中：`mv agent-identity compose/agent-identity`。
       2. 在 `docker-compose.yml` 中，删除 `cma-server` 服务的 `volumes: *server_mounts` 行，并取消注释其下方的 `volumes:` 块。如果您修改过 `x-server-mounts` 列表，请在该块中做相同的修改，因为 `cma-server` 服务不再读取该列表。
       3. 在包含 `agent-identity` 的目录中，运行 `sudo chown -R root:root agent-identity`，除非您在 macOS 上使用 Docker Desktop 或使用 rootless Docker。
     </Tab>

     <Tab title="Kubernetes">
       运行 `kubectl -n cma-server create secret generic cma-agent-identity --from-file=agent-identity/signing-key.pem`。然后在服务器 chart 的 values 文件中将 `server.agentIdentity.secretName` 设置为 `cma-agent-identity`。
     </Tab>
   </Tabs>

5. **修改[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)。** 在 [`mcp.allowed_hostnames`](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences#mcp-servers-need-an-allowlist-entry) 中列出接收方的主机名，并添加以下部分：

   ```yaml
   agent_identity:
     issuer: https://agents.example.com
     key_file: /run/cma-server/agent-identity/signing-key.pem
     direct:
       audiences:
         - audience: https://mcp-gateway.example.com
           urls:
             - https://mcp-gateway.example.com
   ```

   每个 `audience` 以 `https://` URL 的形式标识一个接收方。其 `urls` 是可以接收针对该接收方的令牌的 MCP 服务器的 URL 前缀。

6. **重启服务器。** 在 Compose 栈中，从 Compose 文件所在的目录运行 `docker compose up -d` 而不是重启。

若要避免将私钥存放在文件中，请使用 `key_secret`（您的[密钥管理器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#secret-manager)）或 `kms_key`（AWS KMS 或 Google Cloud KMS）。[配置参考](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#agent-identity)介绍了这两个设置。

### 将令牌发送到 MCP 服务器

[MCP 连接器](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector)介绍了智能体的 MCP 服务器条目的字段。在该条目中添加 `auth` 字段：

```yaml
mcp_servers:
  - type: url
    name: tickets
    url: https://mcp-gateway.example.com/tickets
    auth:
      type: agent_token
      audience: https://mcp-gateway.example.com
```

对于每次工具列举或工具调用，您的服务器都会签发一个新令牌。它以 `Authorization: Bearer <token>` 的形式发送令牌，以替代 vault 凭据。模型和 worker 都不会收到该令牌。

若要同时发送令牌和 [vault 凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)，请将 `auth` 字段替换为 `agent_token_header` 字段：

```yaml
mcp_servers:
  - type: url
    name: tickets
    url: https://mcp-gateway.example.com/tickets
    agent_token_header:
      audience: https://mcp-gateway.example.com
```

vault 凭据放在 `Authorization` 标头中，与两个字段都没有的条目相同。令牌放在 `Agent-Identity-Token` 标头中，且不带 `Bearer` 前缀。

SDK 中没有 `auth` 和 `agent_token_header` 字段。在 SDK 代码中，请将该字段作为额外字段添加到 MCP 服务器条目中，或以纯 JSON 形式发送请求。

以下页面介绍了如何发送额外字段：

[Python SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/python#undocumented-request-params) · [TypeScript SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript#undocumented-request-params) · [Go SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/go#request-fields) · [Java SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#undocumented-parameters) · [PHP SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/php#undocumented-request-parameters) · [Ruby SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/ruby#undocumented-request-params)

其中一些页面仅展示了在请求顶层发送额外字段的方式，而不是在列表条目内部。在这种情况下，请将 `mcp_servers` 本身作为额外字段。以这种方式发送整个列表，并在其条目中包含 `auth` 或 `agent_token_header`。

若要停止某个智能体的令牌，请从该智能体中移除该条目，或归档该智能体。已发送的令牌在其 `exp` 之前仍然有效，默认最长为 5 分钟。

### 接收方需要检查的内容

让接收方仅在以下所有条件都满足时才接受请求：

* 签名可以用您密钥集 `<issuer>/.well-known/jwks.json` 中的某个密钥验证通过。正确的密钥是 ID 与令牌标头中 `kid` 相匹配的那个密钥。对于 RSA 密钥，算法为 `RS256`。
* `iss` 等于您的签发方 URL，且 `aud` 等于接收方的 audience。
* `exp` 在未来，而 `iat` 不在未来。
* 令牌标头中的 `typ` 为 `agent+jwt`，且 `token_use` 为 `agent`。

然后应用以下规则：

* 请基于 `agent_id` 声明（即该智能体在 API 中的 ID）制定每个智能体的访问规则。
* 请接受重复的 `jti`，因为一次列举或调用可能会使用同一令牌发出多个请求。
* 让网关使用各工具自己的凭据调用每个工具，并且永远不要将令牌继续传递出去。

启用[单点登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#single-sign-on)后，工具调用的令牌还会在 `actor_sub` 和 `actor_iss` 声明中标识创建该会话的调用方。`actor_sub` 并不表示调用方批准了该调用。每次调用都由模型选择。

以下令牌没有 `actor_sub`：

* 工具列举的令牌
* 子智能体调用的令牌
* 由服务器 API 密钥创建的会话的所有令牌

如果某条规则依赖于调用方，请拒绝没有 `actor_sub` 的工具调用。

### 测试 MCP 网关

`check-gateway` 命令会向 MCP 网关发送有效和无效的令牌。为该命令提供一个单独的设置文件（此处为 `check-gateway.yaml`）：

1. 仅将服务器设置中的 `mcp` 和 `agent_identity` 部分复制到该文件中；如果服务器设置了 `limits.mcp_call_timeout`，也一并复制。
2. 在副本中，将 `key_file` 改为密钥在您运行该命令的机器上的路径，例如 `./agent-identity/signing-key.pem`。如果您已将该目录移入 `compose`，则路径为 `./compose/agent-identity/signing-key.pem`。

对于 `-agent`，请提供网关允许的某个智能体的 ID。如果密钥归 root 所有，请使用 `sudo`。

```bash
./cma-server check-gateway -config check-gateway.yaml -agent agent_011CYm1BLqPXpQRk5khsSXrs \
  -url https://mcp-gateway.example.com/tickets
```

该命令会为其发送的每个请求打印一行，并附上网关的响应。通过的运行随后会打印一行以 `check-gateway: PASS` 开头的内容，并以代码 `0` 退出。失败的运行会在以 `check-gateway:` 开头的一行中说明出错的原因，并以代码 `1` 退出。

## 密钥管理器

默认情况下，服务器将 [vault 凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)的机密存储在其数据库中。添加 `secret_store` 部分后，服务器会将每个新 vault 凭据的机密写入您的密钥管理器，存放在您为其预留的名称前缀下。数据库只保留一个引用。

将适用于您密钥管理器的块添加到[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)中：

<Tabs>
  <Tab title="AWS Secrets Manager">
    ```yaml
    secret_store:
      type: aws_secrets_manager
      region: us-east-1
      prefix: cma-server/prod/
    ```
  </Tab>

  <Tab title="Google Cloud Secret Manager">
    ```yaml
    secret_store:
      type: gcp_secret_manager
      project: my-project        # the project ID, not its number
      prefix: cma-server-prod--
    ```
  </Tab>

  <Tab title="Azure Key Vault">
    ```yaml
    secret_store:
      type: azure_key_vault
      vault_url: https://my-vault.vault.azure.net
      prefix: cma-server-prod--
    ```
  </Tab>

  <Tab title="HashiCorp Vault">
    ```yaml
    secret_store:
      type: hashicorp_vault
      address: https://vault.internal:8200
      prefix: cma-server/prod/
      approle:
        role_id: <role-id>
        secret_id_file: /run/vault/secret-id
      #kubernetes: { role: cma-server }
      #token_file: /run/vault/token
    ```

    请在三种登录方法中恰好设置一种：

    * **`approle`：** 用于 Docker Compose、主机上的部署或服务器 Helm chart。服务器使用角色 ID 以及从文件中读取的 secret ID 登录。
    * **`kubernetes`：** 仅用于您自行安装的 Kubernetes，且其服务器 pod 需具有服务账户令牌。服务器使用该令牌登录。服务器 Helm chart 不会为服务器的 pod 提供此类令牌，因此此方法不适用于该 chart。
    * **`token_file`：** 用于任何其他 Vault 登录方法。在服务器旁运行 Vault Agent，并让其将令牌写入此文件。

    使用服务器 Helm chart 时，请通过 Secret 向服务器提供 secret ID：

    1. 将 secret ID 放入服务器命名空间中的某个 Secret，并在 chart 的 `server.extensionFiles.secretName` 中指定该 Secret 的名称。
    2. 将 `approle.secret_id_file` 设置为 secret ID 的路径。chart 会将该 Secret 的每个键作为文件挂载到 `/run/cma-server/extension-files/` 下，因此名为 `secret-id` 的键位于 `/run/cma-server/extension-files/secret-id`。

    对于 `kubernetes` 方法，请将 Vault Kubernetes 角色绑定到服务器的命名空间和服务账户，而不是沙箱的命名空间 `cma`。
  </Tab>
</Tabs>

### 授予访问权限

为服务器提供一个身份，并将其访问范围限制在该前缀。归档的 `iam/` 目录为每种密钥管理器提供了一个文件：

* **AWS Secrets Manager：** 替换 `aws-secrets-manager-policy.json` 中的占位符，并将策略附加到服务器运行时所用的身份。
* **Google Cloud Secret Manager：** 运行 `gcp-secret-manager-role.yaml` 注释中的命令。
* **Azure Key Vault：** Azure 角色作用于整个密钥保管库，而不是某个前缀。因此，请为该部署提供一个专属的密钥保管库，并且该密钥保管库需使用 Azure 基于角色的访问控制。替换 `azure-key-vault-role.json` 中的 `SUBSCRIPTION_ID`，并据此创建一个自定义角色。在该密钥保管库上将此角色分配给服务器运行时所用的身份。
* **HashiCorp Vault：** 替换 `hashicorp-vault-policy.hcl` 中的占位符，并将策略附加到服务器运行时所用的身份。

对于 AWS 和 Google Cloud，服务器的身份验证方式与推理时相同。对于 Azure Key Vault，它使用 Microsoft Entra ID 进行身份验证，方式与 [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider#microsoft-foundry) 相同。

### 检查设置

请在首次启动之前以及任何策略变更之后运行 `validate-store`。它会在该前缀下创建、读取并删除一个测试密钥，而无需启动服务器。请在服务器运行的位置、使用服务器的环境运行该命令：

* **Docker Compose：** 在服务器的容器中运行。在 Arm 上，请将此命令中的 `linux_amd64` 替换为 `linux_arm64`：

  ```bash
  docker compose exec cma-server /opt/cma-server/bin/linux_amd64/cma-server validate-store -config /etc/cma-server/config.yaml
  ```

* **在主机上：**

  ```bash
  ./cma-server validate-store -config /etc/cma-server/config.yaml
  ```

* **Kubernetes：** 以 Job 形式在服务器的服务账户下运行相同的命令。

通过的运行会以一行 `pass` 结尾，并以代码 `0` 退出。

使用 Azure Key Vault 时，请在依赖它之前针对您的密钥保管库运行此检查。

### 密钥管理器限制

* **设置只选择一次。** 进行以下任一更改后，服务器将无法再找到其在更改之前写入的密钥：

  * 您移除了 `secret_store` 部分。
  * 您更改了 `type`、`prefix`、`region`、`project`、`vault_url`、`mount` 或 `namespace`。后两者是可选的 HashiCorp Vault 设置。
  * 您将 `address` 指向另一个 Vault。

  只有在您恢复旧值后，服务器才会再次找到这些密钥。

* **一个身份可以读取整个前缀。** 如果两个团队不能访问彼此的凭据，请为每个团队提供各自的部署。每个部署都需要各自的前缀或密钥保管库以及各自的身份。

## 环境变量凭据

借助[环境变量凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults#add-a-credential)，智能体的命令可以调用某个服务，而沙箱只持有该密钥的占位符。"egress gateway"（出站网关）会将真实值放入发往该凭据所列主机的请求中。

该网关 `cma-egress` 是发布归档中的一个代理，位于您的沙箱和互联网之间。它默认处于关闭状态，您的服务器仅在启用网关时才接受这些凭据。

启用网关后，您的沙箱会受到两项限制：

* 沙箱只能通过网关访问互联网。
* 网关只连接您在服务器的 `egress.allowed_hostnames` 设置中列出的主机。

请列出确切的公共主机名，不要使用通配符。请包含您的智能体命令会从中下载内容的每个主机。

如果您构建了自己的沙箱镜像，请通过 `cma-with-egress` 启动 worker，如[构建您自己的沙箱镜像](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#what-an-image-of-your-own-needs)中所示。

### 在 Compose 栈中启用网关

在包含 `docker-compose.yml` 的目录中，向 `.env` 添加两行。第一行列出网关所连接的主机，以逗号分隔。第二行加载网关的 Compose 文件。然后启动栈：

```bash
echo 'CMA_EGRESS__ALLOWED_HOSTNAMES=api.github.com' >> .env
echo 'COMPOSE_FILE=docker-compose.yml:docker-compose.egress.yml' >> .env
docker compose up --build -d
```

如果您还使用另一个 Compose 文件，请将其添加到 `COMPOSE_FILE` 行。shell 中的 `COMPOSE_FILE` 变量会覆盖 `.env` 中的该变量，因此请取消设置 shell 中的变量。

网关的文件会设置服务器的其他 `egress` 设置，因此不要在 `compose.config.yaml` 中添加 `egress` 部分。

### 在 Kubernetes 上启用网关

以下步骤以[在 Kubernetes 上部署](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes)中的部署为起点。它们使用该页面的 `REGISTRY`、`TAG` 和 `PLATFORM` 变量及其两个 values 文件。请在归档的顶层目录中，使用 OpenSSL 3.0 或更高版本运行这些命令。在 Azure Kubernetes Service (AKS) 上，沙箱 chart 不提供该网关。

这些步骤中服务器 chart 的升级还会在服务器的 pod 上添加一个网络策略。开始之前，请检查该策略涵盖的其中两个端口：

* **端口 9464，即指标端口：** 该策略会关闭此端口。如果 Prometheus 抓取该端口，请在 `cma-server-values.yaml` 中将 `server.telemetry.prometheus.scraper.namespace` 设置为 Prometheus 所运行的命名空间。
* **端口 8400，即 API 的端口：** 该策略允许所有来源。如果您自己的某个策略限制了谁可以访问端口 8400，该限制将失效。网络策略是叠加生效的，因此只要任意一个策略允许某个连接，该连接即可通过。

<Steps>
  <Step title="构建并推送网关的镜像">
    ```bash
    docker buildx build --platform "$PLATFORM" -f kubernetes/egress/Dockerfile -t "$REGISTRY/cma-egress:$TAG" --push .
    ```
  </Step>

  <Step title="创建证书颁发机构和网关密钥">
    ```bash
    (umask 077 && openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:P-256 -nodes -days 825 \
      -subj "/CN=cma-server egress gateway" \
      -addext "basicConstraints=critical,CA:TRUE,pathlen:0" \
      -addext "keyUsage=critical,keyCertSign,cRLSign" \
      -keyout egress-ca.key -out egress-ca.crt)
    kubectl -n cma create secret tls cma-egress-ca --cert=egress-ca.crt --key=egress-ca.key
    kubectl -n cma-server create configmap cma-egress-ca --from-file=ca.crt=egress-ca.crt
    GATEWAY_KEY=$(openssl rand -hex 32)
    printf %s "$GATEWAY_KEY" | kubectl -n cma create secret generic cma-egress-gateway-key --from-file=gateway-key=/dev/stdin
    printf %s "$GATEWAY_KEY" | kubectl -n cma-server create secret generic cma-egress-gateway-key --from-file=gateway-key=/dev/stdin
    unset GATEWAY_KEY
    ```

    然后删除 `egress-ca.key`，即证书颁发机构的私钥。任何持有该私钥的人，都可以对您的沙箱冒充任何网站。该证书的有效期为 825 天。请参阅[续订证书](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#renew-the-certificate)。
  </Step>

  <Step title="移除早期会话的沙箱">
    如果该环境尚未运行过任何会话，请跳过此步骤。变更之前创建的沙箱在变更之后无法启动其 worker。以下命令还会删除每个沙箱的工作区：

    ```bash
    kubectl -n cma scale deployment cma-dispatcher --replicas=0
    kubectl -n cma delete sandboxwarmpool cma-sandbox
    kubectl -n cma delete sandboxclaims --all
    ```

    在 Google Kubernetes Engine (GKE) 上，还需运行以下命令：

    ```bash
    kubectl -n cma delete sandboxes --all
    ```

    重复运行 `kubectl -n cma get sandboxes,pods,pvc`，直到找不到任何内容。"启用沙箱端"步骤中的 `helm upgrade` 会再次启动 dispatcher。
  </Step>

  <Step title="启用服务器端">
    在集群的 Service 地址范围内为网关选择一个空闲的 IPv4 地址。`kubectl get svc -A` 的 `CLUSTER-IP` 列会显示该范围内的地址。将以下内容添加到 `cma-server-values.yaml`。如果该文件已有 `server` 块，请将这些行合并到其中：

    ```yaml
    egressGateway:
      enabled: true
      proxyAddress: 34.118.224.53:3128   # the address that you chose, and port 3128
    server:
      config:
        egress:
          allowed_hostnames:
            - api.github.com
    ```

    然后运行升级：

    ```bash
    helm upgrade cma-server kubernetes/server/chart --namespace cma-server -f cma-server-values.yaml --wait
    ```
  </Step>

  <Step title="启用沙箱端">
    运行 `kubectl -n cma-server get svc` 以列出服务器的两个 Service。在新安装中，它们的名称为 `cma-server` 和 `cma-server-egress`。

    在 `cma-values.yaml` 中，将 `egress.mode` 改为 `gateway`，并在 `egress` 下添加一个 `gateway` 块。在该块中填入该命令列出的两个名称：

    ```yaml
    egress:
      mode: gateway
      gateway:
        serviceIP: 34.118.224.53   # the address that you chose
        serverService: cma-server
        serverEgressService: cma-server-egress
        apiServerCIDRs:
          - 203.0.113.10/32        # the cluster's API endpoint
    ```

    保留该文件中其他的 `egress` 值。若要打印 API 端点的地址和端口，请运行 `kubectl get endpoints kubernetes -n default`。如果端口不是 443，还需设置 `egress.gateway.apiServerPort`。

    ```bash
    helm upgrade cma kubernetes/chart --namespace cma -f cma-values.yaml --wait
    ```

    如果升级因 `.spec.replicas` 上的冲突而停止，请运行 `kubectl -n cma scale deployment cma-dispatcher --replicas=1`，然后再次运行升级。
  </Step>
</Steps>

### 添加凭据

按照[添加凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults#add-a-credential)中的说明创建环境变量凭据。托管服务的指南称这些凭据不支持自托管沙箱。该说法描述的是托管服务，而不是您的服务器。

您的服务器会添加自己的规则。它会以 `400` 错误拒绝违反其中某条规则的请求，该错误会指明相应字段。其中两条规则如下：

* `networking` (csharp, go: `Networking`) 的类型必须为 `limited`，并且 `allowed_hosts` (java, php: `allowedHosts`; csharp, go: `AllowedHosts`) 中的每个主机都必须在服务器的 `egress.allowed_hostnames` 列表中。
* `injection_location` (java, php: `injectionLocation`; csharp, go: `InjectionLocation`) 在 JSON 中必须为 `{"header": true}`，且不能省略。

向服务器的列表中添加主机后，请应用该更改。在 Docker Compose 上，请运行 `docker compose up -d`。在 Kubernetes 上，请运行服务器的 `helm upgrade` 命令和 `kubectl -n cma-server rollout restart deployment -l anthropic.com/role=server`。

### 检查出站网关

使用该凭据的 vault 创建一个会话，并让智能体运行两条命令。此示例针对的是名为 `GITHUB_TOKEN`、列出了 `api.github.com` 的凭据：

```bash
printenv GITHUB_TOKEN
curl -sS -H "Authorization: Bearer $GITHUB_TOKEN" https://api.github.com/user
```

第一条命令会打印一个以 `cma_ph_` 开头的占位符。第二条命令会打印该令牌对应的 GitHub 账户的配置文件，因为网关已用令牌替换了占位符。对于发往不在您列表中的主机的请求，网关会以 `403` 错误拒绝。

在 Kubernetes 上，如果每个请求都收到 `503` 错误，很可能是您集群的网络改写了 pod 的地址，例如 kube-proxy 的 `--masquerade-all` 选项就会这样做。网关通过 pod 的地址来区分各个沙箱，因此这些地址必须原样到达网关。

### 续订证书

在 Kubernetes 上，请在没有会话运行时续订证书。重复[在 Kubernetes 上启用网关](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#turn-on-the-gateway-on-kubernetes)中的整个"创建证书颁发机构和网关密钥"步骤。该步骤会重新创建证书颁发机构和网关密钥，因此您不需要已删除的 `egress-ca.key` 文件。只需做一处更改：在运行 `kubectl create` 的四行中，每行末尾都加上 `--dry-run=client -o yaml | kubectl apply -f -`。例如，以 `--from-file=gateway-key=/dev/stdin` 结尾的行，随后会以 `--from-file=gateway-key=/dev/stdin --dry-run=client -o yaml | kubectl apply -f -` 结尾。

然后重启服务器和网关：

```bash
kubectl -n cma-server rollout restart deployment -l anthropic.com/role=server
kubectl -n cma rollout restart deployment/cma-egress
```

### 出站网关限制

* **标头：** 网关只会将值放入请求标头中，绝不会放入 URL 或请求体中。默认情况下，唯一的标头是 `Authorization`。若要允许其他标头，请将它们与 `Authorization` 一起列在 `egress.swap_header_names` 中；对于 Compose 栈，则列在 `.env` 的 `CMA_EGRESS__SWAP_HEADER_NAMES` 中。
* **忽略代理变量的程序：** 沙箱通过 `HTTPS_PROXY` 及相关变量找到网关。Maven 和 Gradle 等 Java 工具不会读取这些变量，因此这些工具无法访问任何内容。
