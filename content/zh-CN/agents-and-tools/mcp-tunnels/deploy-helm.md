---
title: 使用 Helm 部署 MCP 隧道
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm
description: 使用 Anthropic Helm chart 在 Kubernetes 集群上安装隧道栈。
---

<Note>
  “MCP tunnels”（MCP 隧道）目前处于"research preview"（研究预览）阶段。[申请访问权限](https://claude.com/form/mcp-tunnels)即可试用。
</Note>

Anthropic Helm chart 将[隧道栈](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)作为单个 Deployment 安装，并将其附加到您的隧道上：可以是 chart 的设置钩子为您创建的隧道，也可以是您在 [Claude Console](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#create-a-tunnel) 中创建的现有隧道。

## 开始之前

您需要：

* **一个隧道。**&#x4F7F;用程序化访问时，如果您未提供隧道 ID，chart 的设置钩子会为您创建一个；如果要改为附加到现有隧道，请[在 Console 中创建隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#create-a-tunnel)并记录隧道 ID（`tnl_...`）。手动配置始终从在 Console 中创建的隧道开始；您还需要其隧道令牌和隧道域名。

* **chart 向 Tunnels API 进行身份验证的方式。**

  * **[程序化访问](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#credential-provisioning)（推荐）。**[设置组件](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)通过 Workload Identity Federation（工作负载身份联合）进行身份验证，获取隧道令牌，生成 CA，将其注册到 Anthropic，并将所有内容存储在 Secret 中。您需要一个作用域为 `workspace:manage_tunnels` 的联合规则。
  * **[手动](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#credential-provisioning)。** 跳过编程访问。您需要[从 Console 获取隧道令牌](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#get-the-connection-details)，自行生成 CA 和服务器证书，[在 Console 中注册 CA](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#add-a-ca-certificate)，并将凭据作为 Secret 提供给集群。

* **一个 Kubernetes 集群**，您可以使用 `helm` 和 `kubectl` 向其部署。**不使用编程访问**选项卡还会用到 `openssl`（1.1.1 或更高版本）。

* 从集群到 `api.anthropic.com`（443 TCP）和[隧道边缘](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)（7844 TCP 和 UDP）的**出站网络连接**。请参阅完整的[网络要求](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview#network-requirements)。

* **一个或多个 MCP 服务器**，正在运行且可从集群通过您将在 `gateway.config.routes` 下配置的地址访问。如果您还没有，请[使用示例服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm#optional-use-a-sample-mcp-server)。

## 可选：使用示例 MCP 服务器

如果您没有可用于测试的 MCP 服务器，请使用这个最小化的服务器：

```bash
kubectl create namespace mcp-tunnel --dry-run=client -o yaml | kubectl apply -f -
kubectl -n mcp-tunnel apply -f - <<'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-mcp-src
data:
  hello_server.py: |
    from mcp.server.fastmcp import FastMCP

    mcp = FastMCP("hello-server", host="0.0.0.0", port=9000)


    @mcp.tool()
    def hello(name: str = "world") -> str:
        """Say hello to someone."""
        return f"Hello, {name}!"


    if __name__ == "__main__":
        mcp.run(transport="streamable-http")
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-mcp
spec:
  replicas: 1
  selector:
    matchLabels: { app: hello-mcp }
  template:
    metadata:
      labels: { app: hello-mcp }
    spec:
      containers:
        - name: hello-mcp
          image: python:3.13-slim
          command: ["sh", "-c", "pip install --quiet mcp && python /app/hello_server.py"]
          volumeMounts:
            - { name: src, mountPath: /app }
          ports:
            - { containerPort: 9000 }
      volumes:
        - name: src
          configMap: { name: hello-mcp-src }
---
apiVersion: v1
kind: Service
metadata:
  name: hello-mcp
spec:
  selector: { app: hello-mcp }
  ports:
    - { port: 9000, targetPort: 9000 }
EOF
```

后续的安装步骤会注明在何处添加相应的路由。

## 安装

<Tabs>
  <Tab title="使用编程访问">
    设置组件通过您的联合规则交换集群的投射 ServiceAccount 令牌，获取隧道令牌，生成 CA 和服务器证书，并将 CA 注册到 Anthropic。每日运行的 CronJob 会根据需要续订服务器证书，因此您无需手动处理任何密钥。

    <Steps>
      <Step title="为集群设置工作负载身份联合">
        按照[在 Kubernetes 中使用 WIF](https://platform.claude.com/docs/zh-CN/manage-claude/wif-providers/kubernetes) 注册集群的 OIDC 颁发者并创建联合规则。设置组件在发布命名空间中以其自己的 ServiceAccount 运行；确切名称遵循 Helm 的 `fullname` 约定，因此对于 `mcp-tunnel` 以外的任何发布名称，请在创建规则之前运行 `helm template <release> ... | grep -A2 'kind: ServiceAccount'` 进行确认。本指南的其余部分假定发布名称为 `mcp-tunnel`，命名空间为 `mcp-tunnel`，此时 ServiceAccount 为 `mcp-tunnel-setup`。

        | 字段  | 值                                                   |
        | --- | --------------------------------------------------- |
        | 主体  | `system:serviceaccount:mcp-tunnel:mcp-tunnel-setup` |
        | 受众  | `api.anthropic.com`（chart 的默认值；不含协议）                |
        | 作用域 | `workspace:manage_tunnels`                          |

        <Note>
          chart 的默认受众是不含协议的 `api.anthropic.com`，但 Console 的联合规则表单建议使用 `https://api.anthropic.com`。两者必须逐字节匹配，否则身份验证会失败。请将规则的受众设置为 `api.anthropic.com`，或在 `values.yaml` 中将 `api.wif.audience` 设置为 `https://api.anthropic.com`。
        </Note>

        如果隧道位于组织默认工作区以外的工作区中，还需[将规则的服务账户添加到该工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#role-inheritance)（Tunnels API 根据服务账户的工作区成员资格进行授权）。

        记下规则的 ID（`fdrl_...`）；您将把它设置为 `api.wif.federationRuleId`。

        <Note>
          每日证书续订 CronJob 使用单独的 ServiceAccount（同样派生自 Helm `fullname`），但不调用 Tunnels API；它在本地续订证书，只需要 chart 授予的 Kubernetes RBAC。联合规则无需覆盖它。
        </Note>
      </Step>

      <Step title="获取默认值">
        ```bash
        helm show values \
          oci://us-docker.pkg.dev/anthropic-public-registry/charts/mcp-tunnel \
          --version 2.1.0 > values.yaml
        ```
      </Step>

      <Step title="配置隧道附加和路由">
        编辑 `values.yaml`，使用联合规则 ID 和组织 ID 设置 `api.wif.*` 键，并为每个[上游 MCP 服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)添加一个 `routes` 条目：

        ```yaml values.yaml
        api:
          wif:
            federationRuleId: "fdrl_..."
            organizationId: "00000000-0000-0000-0000-000000000000"
            # Set when the tunnel is in a non-default workspace and the
            # rule's service account is a member of that workspace.
            # workspaceId: "wrkspc_..."

        tunnel:
          # Leave empty to have the setup hook create a tunnel during install.
          # Set to attach to an existing tunnel from the Console.
          id: ""
          # Increment to rotate the tunnel token on the next upgrade.
          # See the "Rotate the tunnel token" section.
          tokenVersion: "1"

        gateway:
          config:
            routes:
              docs: http://docs-mcp.internal:8080
              search: http://search-mcp.internal:8080
        ```

        使用这些路由后，Claude 通过 `docs.<your-tunnel-domain>` 和 `search.<your-tunnel-domain>` 访问服务器。某些托管 Kubernetes 发行版会在标准私有地址范围之外分配 Service CIDR；如果您的路由指向集群内的 Service，请按照[上游 IP 验证](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/troubleshooting#upstream-ip-validation)中的说明在此处添加 `gateway.config.upstream.allowed_ips`。

        <Note>
          如果您使用的是[示例 MCP 服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm#optional-use-a-sample-mcp-server)，请改为将 `routes` 设置为 `echo: http://hello-mcp:9000`。
        </Note>
      </Step>

      <Step title="审查渲染后的清单">
        渲染 chart，并按照您组织的审查惯例检查输出：

        ```bash
        helm template mcp-tunnel \
          oci://us-docker.pkg.dev/anthropic-public-registry/charts/mcp-tunnel \
          --version 2.1.0 \
          -n mcp-tunnel \
          -f values.yaml > rendered.yaml
        ```
      </Step>

      <Step title="安装">
        ```bash
        helm install mcp-tunnel \
          oci://us-docker.pkg.dev/anthropic-public-registry/charts/mcp-tunnel \
          --version 2.1.0 \
          --namespace mcp-tunnel --create-namespace \
          -f values.yaml
        ```

        设置组件作为 Helm pre-install 钩子 Job 运行，因此 `helm install` 会阻塞直到其完成。成功后，Helm 会自动删除该 Job。如果 `helm install` 因钩子错误而失败，请参阅[设置组件身份验证失败](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/troubleshooting#setup-component-authentication-failures)。

        当 `tunnel.id` 为空时，设置组件会在您的联合规则所针对的工作区中创建隧道（除非您设置了 `api.wif.workspaceId`，否则为组织的默认工作区），并将其 ID 和域名存储在 `mcp-tunnel` Secret 中。您可以在隧道的详情页面上找到[验证](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm#verify-the-deployment)所需的域名，该页面可从 Console 中的 [MCP 隧道](https://platform.claude.com/settings/workspaces/default/mcp-tunnels)页面（`platform.claude.com/settings/workspaces/default/mcp-tunnels`）打开。在侧边栏的工作区选择器中，选择包含该隧道的工作区。您也可以从 Secret 中读取域名：

        ```bash
        kubectl -n mcp-tunnel get secret mcp-tunnel \
          -o jsonpath='{.data.tunnel-domain}' | base64 -d
        ```

        重新运行设置组件（在[升级](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm#upgrades)或[令牌轮换](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm#rotate-the-tunnel-token)期间）会复用存储在此 Secret 中的隧道 ID；它永远不会创建第二个隧道。

        <Warning>
          `api.wif.*` 值是标识符而非密钥，因此将它们存储在 Helm 发布历史 Secret 中不存在风险。静态存储的敏感数据是设置组件创建的 `mcp-tunnel` Secret，其中包含隧道令牌和 TLS 私钥。请对此命名空间应用您组织保护 Kubernetes Secret 的标准做法。
        </Warning>
      </Step>
    </Steps>
  </Tab>

  <Tab title="不使用编程访问">
    在此模式下（`setup.enabled: false`），chart 不进行任何 API 调用；设置组件不会运行，也没有 cert-renew CronJob。如果您不想设置工作负载身份联合，请使用此方式。

    <Steps>
      <Step title="获取隧道令牌和域名">
        [创建隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#create-a-tunnel)并[从 Console 获取隧道令牌](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#get-the-connection-details)。

        <Note>
          从详情页面记录隧道域名。您将把它设置为 `gateway.config.tunnel_domain`。
        </Note>
      </Step>

      <Step title="生成 CA 和服务器证书">
        代理监听普通 WebSocket，[内层 TLS](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components) 使用您在此处生成的证书承载在该数据流中。服务器证书的 SAN 必须包含 `*.<tunnel-domain>`，详见[证书要求](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/reference#certificate-requirements)。

        ```bash
        export TUNNEL_DOMAIN=YOUR_TUNNEL_DOMAIN_HERE
        mkdir -p mcp-tunnel/data
        cd mcp-tunnel

        # 自签名 CA。显式指定扩展，使其满足证书
        # 要求，而不受发行版 openssl.cnf 默认值影响。
        openssl req -x509 -newkey rsa:2048 -nodes \
          -keyout data/ca.key -out data/ca.crt \
          -days 3650 -subj "/CN=mcp-tunnel-ca" \
          -addext "basicConstraints=critical,CA:TRUE" \
          -addext "keyUsage=critical,keyCertSign,cRLSign" \
          -addext "subjectKeyIdentifier=hash"

        # 服务器证书的扩展文件。使用 -extfile（而非
        # 仅 OpenSSL 3.0+ 支持的 -copy_extensions）可确保在
        # OpenSSL 1.1.x 上也能正常工作。
        cat > data/tls.ext <<EOF
        subjectAltName = DNS:${TUNNEL_DOMAIN},DNS:*.${TUNNEL_DOMAIN}
        authorityKeyIdentifier = keyid,issuer
        extendedKeyUsage = serverAuth
        EOF

        # 由 CA 签名的服务器证书
        openssl req -newkey rsa:2048 -nodes \
          -keyout data/tls.key -out /tmp/server.csr \
          -subj "/CN=${TUNNEL_DOMAIN}"
        openssl x509 -req -in /tmp/server.csr \
          -CA data/ca.crt -CAkey data/ca.key -CAcreateserial \
          -out data/tls.crt -days 90 \
          -extfile data/tls.ext
        ```

        [在 Console 中注册 `data/ca.crt`](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#add-a-ca-certificate)。将 `data/ca.key` 保存在持久且安全的位置；续订时您需要用它签署新的服务器证书。
      </Step>

      <Step title="创建两个 Secret">
        chart 读取特定的键；Secret 名称可配置，但键不可配置。如果命名空间已存在（例如，在[示例 MCP 服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm#optional-use-a-sample-mcp-server)步骤中已创建），以下命名空间创建命令不会执行任何操作。

        ```bash
        kubectl create namespace mcp-tunnel --dry-run=client -o yaml | kubectl apply -f -
        kubectl -n mcp-tunnel create secret generic mcp-tunnel-token \
          --from-literal=tunnel-token='eyJ...'
        kubectl -n mcp-tunnel create secret generic mcp-tunnel-cert \
          --from-file=tls.crt=data/tls.crt \
          --from-file=tls.key=data/tls.key
        ```
      </Step>

      <Step title="获取默认值">
        ```bash
        helm show values \
          oci://us-docker.pkg.dev/anthropic-public-registry/charts/mcp-tunnel \
          --version 2.1.0 > values.yaml
        ```
      </Step>

      <Step title="为手动配置设置值">
        编辑 `values.yaml` 并设置以下键：

        ```yaml values.yaml
        setup:
          enabled: false

        external:
          tunnelTokenSecretName: mcp-tunnel-token   # must contain key: tunnel-token
          serverCertSecretName: mcp-tunnel-cert     # must contain keys: tls.crt, tls.key

        gateway:
          config:
            # Required when setup.enabled is false. Replace the placeholder with
            # the $TUNNEL_DOMAIN value you exported earlier. When setup.enabled
            # is true the chart injects this from the Secret as a -tunnel-domain
            # flag instead.
            tunnel_domain: YOUR_TUNNEL_DOMAIN_HERE
            routes:
              docs: http://docs-mcp.internal:8080
              search: http://search-mcp.internal:8080
        ```

        某些托管 Kubernetes 发行版会在标准私有地址范围之外分配 Service CIDR；如果您的路由指向集群内的 Service，请按照[上游 IP 验证](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/troubleshooting#upstream-ip-validation)中的说明在此处添加 `gateway.config.upstream.allowed_ips`。

        <Note>
          如果您使用的是[示例 MCP 服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm#optional-use-a-sample-mcp-server)，请改为将 `routes` 设置为 `echo: http://hello-mcp:9000`。
        </Note>
      </Step>

      <Step title="审查渲染后的清单">
        ```bash
        helm template mcp-tunnel \
          oci://us-docker.pkg.dev/anthropic-public-registry/charts/mcp-tunnel \
          --version 2.1.0 \
          -n mcp-tunnel \
          -f values.yaml > rendered.yaml
        ```
      </Step>

      <Step title="安装">
        ```bash
        helm install mcp-tunnel \
          oci://us-docker.pkg.dev/anthropic-public-registry/charts/mcp-tunnel \
          --version 2.1.0 \
          --namespace mcp-tunnel --create-namespace \
          -f values.yaml
        ```
      </Step>
    </Steps>
  </Tab>
</Tabs>

## 验证部署

从 Anthropic 一侧进行端到端验证：在 Managed Agent 会话或 Messages API 请求中使用 `https://<route>.<your-tunnel-domain>/<path>`，其中 `<route>` 是 `gateway.config.routes` 中的一个键，`<path>` 是上游 MCP 服务器提供服务的路径。使用[示例 MCP 服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm#optional-use-a-sample-mcp-server)时，该地址为 `https://echo.<your-tunnel-domain>/mcp`。有关请求格式，请参阅[使用隧道化的 MCP 服务器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview#use-the-tunneled-mcp-servers)。

如果验证失败，请检查 pod 日志（`kubectl -n mcp-tunnel logs deploy/mcp-tunnel -c mcp-proxy` 和 `-c cloudflared`）并查阅[故障排除](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/troubleshooting)。

## 可选配置

### 使用 NetworkPolicy 限制出站流量

默认情况下，拒绝到代理 pod 的入站流量（`networkPolicy.ingress.enabled: true`）。要进一步限制 pod 出站流量，请设置 `networkPolicy.egress.enabled: true`，并在 `networkPolicy.egress.mcpServers` 中填入覆盖您上游 MCP 服务器的 pod 标签选择器或 CIDR 范围。从 cloudflared 到隧道边缘的出站流量通过 `networkPolicy.egress.cloudflaredEgressCIDRs` 单独放行。

### 调整代理

`gateway.config.*` 下的字段会传递到代理配置文件。常见的调整包括 `upstream.allowed_ips`、`log_level` 和 `upstream.tls`。有关完整的字段列表，请参阅[代理配置](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/reference#proxy-configuration)参考。chart 始终会设置 `listen_addr`、`tls.cert_file` 和 `tls.key_file`；在 `gateway.config` 中设置它们不会生效。

### 提供您自己的 OIDC 令牌

默认情况下，chart 会为设置组件投射一个 Kubernetes ServiceAccount 令牌。要使用来自其他身份提供商（例如 [SPIFFE](https://platform.claude.com/docs/zh-CN/manage-claude/wif-providers/spiffe)、Vault 或云 SDK sidecar）的令牌，请使用 `setup.extraVolumes` 和 `setup.extraVolumeMounts` 挂载它。然后将 `api.wif.tokenFile` 指向挂载路径。chart 会将 `ANTHROPIC_IDENTITY_TOKEN_FILE` 设置为该路径，设置组件会从那里读取令牌。

## 升级

始终向 `helm upgrade` 传递 `--version`，以免意外拉取更新的 chart。

### 从 chart 1.x 升级

Chart 2.0.0 将隧道 ID 从 `api.wif.tunnelId` 移至 `tunnel.id`。升级前，请编辑您的 `values.yaml`：将 `tnl_...` 值移至 `tunnel.id` 并删除 `api.wif.tunnelId`。不设置 `tunnel.id` 也是安全的（设置组件在重新运行时会复用已存储在 `mcp-tunnel` Secret 中的隧道 ID），但显式迁移可以使您的 `values.yaml` 保持准确。此外，还需在 Console 中将联合规则的范围从 `org:manage_tunnels` 更新为 `workspace:manage_tunnels`。

### 更改配置

对于路由、副本数或 NetworkPolicy 等常规更改：

```bash
helm upgrade mcp-tunnel \
  oci://us-docker.pkg.dev/anthropic-public-registry/charts/mcp-tunnel \
  --version 2.1.0 \
  -n mcp-tunnel \
  -f values.yaml
```

<Warning>
  请维护一个完整的 `values.yaml`，而不是依赖 `--reuse-values`。Helm 的深度合并行为可能会在无提示的情况下无法移除已删除的路由。
</Warning>

### 轮换隧道令牌

使用编程访问时，在 `values.yaml` 中递增 `tunnel.tokenVersion`，并使用 `--set setup.force=true` 进行升级。设置组件仅在强制时才会在升级期间重新运行：

```bash
helm upgrade mcp-tunnel \
  oci://us-docker.pkg.dev/anthropic-public-registry/charts/mcp-tunnel \
  --version 2.1.0 \
  -n mcp-tunnel \
  -f values.yaml \
  --set setup.force=true
```

设置组件使用工作负载身份联合进行身份验证；没有需要撤销的 API 令牌。

不使用编程访问时，在 Console 的隧道详情页面上点击**轮换令牌**，然后更新 `mcp-tunnel-token` Secret：

```bash
kubectl -n mcp-tunnel create secret generic mcp-tunnel-token \
  --from-literal=tunnel-token='eyJ...' --dry-run=client -o yaml | kubectl apply -f -
kubectl -n mcp-tunnel rollout restart deploy/mcp-tunnel
```

<Warning>
  点击**轮换令牌**会立即使当前令牌失效。在 Secret 更新且滚动发布完成之前，任何使用旧令牌重启的 pod（驱逐、节点排空、OOM）都无法重新连接。请在轮换后及时更新 Secret；如果有更严格的可用性要求，请使用编程访问，以便 chart 以原子方式处理轮换。
</Warning>

### 证书续订

chart 提供了自动化功能，但您仍需负责监控证书到期情况并确认续订完成。

使用编程访问时，证书续订是自动的。chart 会部署一个 CronJob（以 Helm `fullname` 命名，后缀为 `-cert-renew`），每天运行 `setup renew-cert`（在 `serverCert.cronSchedule`，默认为 UTC `0 0 * * *`）。除非证书距到期时间在 `serverCert.renewBefore`（默认 30 天）以内，否则该作业不执行任何操作。续订在本地进行：该作业使用已存储在 Secret 中的 CA 签署新证书，不进行任何 API 调用，只需要 chart 授予的 Kubernetes RBAC。代理会从 Secret 挂载中热重载证书，因此无需重启 Deployment。

不使用编程访问时，没有 CronJob。在安装后保留的 `mcp-tunnel/` 目录中，使用现有 CA 签署新的服务器证书（不要重新生成 CA）：

```bash
export TUNNEL_DOMAIN=YOUR_TUNNEL_DOMAIN_HERE
openssl req -new -key data/tls.key -out /tmp/server.csr \
  -subj "/CN=${TUNNEL_DOMAIN}"
openssl x509 -req -in /tmp/server.csr \
  -CA data/ca.crt -CAkey data/ca.key -CAcreateserial \
  -out data/tls.crt -days 90 -extfile data/tls.ext

kubectl -n mcp-tunnel create secret generic mcp-tunnel-cert \
  --from-file=tls.crt=data/tls.crt --from-file=tls.key=data/tls.key \
  --dry-run=client -o yaml | kubectl apply -f -
```

代理会从 Secret 挂载中热重载证书。

## 后续步骤

<CardGroup cols={2}>
  <Card title="使用隧道化的 MCP 服务器" icon="link" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview#use-the-tunneled-mcp-servers">
    将上游 MCP 服务器附加到 Managed Agent 或 Messages API。
  </Card>

  <Card title="安全" icon="lock" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/security">
    加固指南、凭据轮换和安全事件响应。
  </Card>

  <Card title="故障排除" icon="wrench" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/troubleshooting">
    诊断连接、TLS 和路由问题。
  </Card>
</CardGroup>
