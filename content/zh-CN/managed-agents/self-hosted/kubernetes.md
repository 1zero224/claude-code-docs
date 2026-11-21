---
title: 在 Kubernetes 上部署
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes
description: 使用发布归档中的两个 Helm chart 在 Kubernetes 集群上安装自托管 Claude Managed Agents 服务器及其沙箱，然后在沙箱 pod 中运行第一个会话。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

本页面将引导您从一个空的 Google Kubernetes Engine (GKE) 集群开始，直到运行一个在沙箱 pod 中执行工具的会话。它使用 “release archive”（发布归档）中的两个 Helm chart，这两个 chart 会在集群上运行整个 “stack”（栈）：

* **`cma-server`：** 应用数据库 schema，并在命名空间 `cma-server` 中的一个 Service 后面运行两个服务器。
* **`cma-sandboxes`：** 在命名空间 `cma` 中运行一个 dispatcher 和沙箱 pod。dispatcher 会轮询服务器以获取工作，并为每个会话分配其专属的沙箱 pod，worker 在其中运行智能体的 shell 和文件工具。

您的 PostgreSQL 数据库位于两者之外。在其他类型的集群上，步骤相同，但集群前提条件和少量 chart 值有所不同。请参阅[其他集群](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#other-clusters)。

## 前提条件

* **一个集群：** Kubernetes 1.30 或更高版本，并具备：

  * Agent Sandbox 控制器（在 GKE 上，即 Agent Sandbox 插件；如[在 GKE 上设置插件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#set-up-the-add-on-on-gke)中所述，版本 0.16.0 需要 GKE 1.35）
  * 运行 gVisor 的节点（在 GKE 上，即 [GKE Sandbox 节点池](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/sandbox-pods)）
  * 强制执行 NetworkPolicy 的网络插件（在 GKE 上，即 Dataplane V2 或 Calico）

* **一个数据库：** 集群可以访问的 PostgreSQL 16 或更高版本。

* **一个镜像仓库：** 集群可以从中拉取镜像的仓库。镜像需要您自行构建。

* **一个 Claude API 密钥：** 服务器使用它进行模型调用。如需使用其他提供商，请参阅[推理提供商](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/inference-provider)。

* **权限：** cluster-admin 权限或同等权限。安装过程会创建命名空间、Role 和一个 ClusterRoleBinding。

* **您机器上的工具：** `helm`、`kubectl`、`docker buildx`、`openssl`，以及 [`ant` CLI](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart#installation)。

* **发布归档：** 如需申请下载访问权限，请联系您的 Anthropic 客户团队。如果您的组织已拥有访问权限，请按照本页面顶部的说明操作。下载归档后，将其解压。本页面上的所有命令都应在其顶层目录中运行。

## 安装栈

<Steps>
  <Step title="构建并推送镜像">
    将这些变量设置为您自己的值。`PLATFORM` 是您节点的架构。`TAG` 是发布版本号，位于归档名称中 `v` 之后。两个 chart 都要求镜像以该版本号作为标签。

    ```bash
    REGISTRY=us-docker.pkg.dev/my-project/cma
    TAG=1.2.3
    PLATFORM=linux/amd64
    ```

    然后构建并推送这三个镜像：

    ```bash
    docker buildx build --platform "$PLATFORM" -f kubernetes/server/Dockerfile -t "$REGISTRY/cma-server-image:$TAG" --push .
    docker buildx build --platform "$PLATFORM" -f kubernetes/dispatcher/Dockerfile -t "$REGISTRY/cma-dispatcher:$TAG" --push .
    docker buildx build --platform "$PLATFORM" -t "$REGISTRY/cma-sandbox:$TAG" --push kubernetes/sandbox
    ```
  </Step>

  <Step title="创建机密文件和数据库">
    创建一个文件，其中包含生成的数据库密码，以及您的 Claude API 密钥的占位符：

    ```bash
    cat > secrets.env <<EOF
    PG_PASSWORD=$(openssl rand -hex 32)
    UPSTREAM_KEY=sk-ant-...
    EOF
    chmod 600 secrets.env
    ```

    在编辑器中打开 `secrets.env`，并将 `sk-ant-...` 替换为您的 Claude API 密钥。然后以管理员身份连接到您的数据库服务器，例如使用 `psql`。运行以下 SQL，使用 `PG_PASSWORD` 的值替换 `...`：

    ```sql
    CREATE ROLE cma_server LOGIN PASSWORD '...';
    GRANT cma_server TO CURRENT_USER;
    CREATE DATABASE cma_server OWNER cma_server;
    ```
  </Step>

  <Step title="创建服务器的命名空间和 Secret">
    创建服务器的命名空间及其 Secret：

    ```bash
    source secrets.env
    ENCRYPTION_KEY=$(openssl rand -base64 32)
    kubectl create namespace cma-server
    kubectl label namespace cma-server pod-security.kubernetes.io/enforce=restricted pod-security.kubernetes.io/enforce-version=latest
    kubectl -n cma-server create secret generic cma-server-db \
      --from-literal=PGPASSWORD="$PG_PASSWORD"
    kubectl -n cma-server create secret generic cma-server-keys \
      --from-literal=CMA_SERVER__API_KEY="$(openssl rand -hex 32)" \
      --from-literal=CMA_STORE__CREDENTIALS_ENCRYPTION_KEY="$ENCRYPTION_KEY"
    kubectl -n cma-server create secret generic cma-server-upstream \
      --from-literal=ANTHROPIC_API_KEY="$UPSTREAM_KEY"
    ```

    使用 `echo "$ENCRYPTION_KEY"` 打印加密密钥，并在集群之外保留一份副本。服务器使用此密钥来加密其存储的凭据，而恢复的数据库需要它。然后删除 `secrets.env`。`cma-server-db` Secret 会保存数据库密码。
  </Step>

  <Step title="创建沙箱的命名空间和证书">
    创建沙箱的命名空间，以及一个由 dispatcher 提供、沙箱信任的证书：

    ```bash
    kubectl create namespace cma
    kubectl label namespace cma pod-security.kubernetes.io/enforce=restricted pod-security.kubernetes.io/enforce-version=latest
    openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:P-256 -nodes -days 825 \
      -subj "/CN=cma-dispatcher.cma.svc" \
      -addext "subjectAltName=DNS:cma-dispatcher.cma.svc,DNS:cma-dispatcher.cma.svc.cluster.local" \
      -keyout tls.key -out tls.crt
    kubectl -n cma create secret tls cma-dispatcher-tls --cert=tls.crt --key=tls.key --dry-run=client -o yaml | kubectl apply -f -
    kubectl -n cma create configmap cma-dispatcher-ca --from-file=ca.crt=tls.crt --dry-run=client -o yaml | kubectl apply -f -
    ```
  </Step>

  <Step title="安装服务器">
    将以下内容保存为 `cma-server-values.yaml`，填入您的镜像仓库和数据库地址：

    ```yaml
    earlierNames: false   # a new install
    image:
      registry: us-docker.pkg.dev/my-project/cma   # REGISTRY in the build commands
    database:
      url: host=10.20.0.5 port=5432 user=cma_server dbname=cma_server sslmode=require   # your database's address
    environments:
      - name: default
        namespace: cma
    ```

    然后安装该 chart：

    ```bash
    helm install cma-server kubernetes/server/chart --namespace cma-server -f cma-server-values.yaml --wait
    kubectl -n cma-server logs job/cma-server-environment-default
    ```

    安装过程会创建环境 `default`，并将其密钥存储在命名空间 `cma` 中的 Secret `cma-environment` 里。第二条命令会打印一行以 `filled Secret cma/cma-environment` 开头的输出。如果首次 `helm install` 失败，请先对该 release 运行 `helm uninstall`，然后再重新安装。
  </Step>

  <Step title="查找要阻止的集群地址">
    此安装允许沙箱访问公共互联网。集群的 API 端点通常具有公共地址，因此沙箱也可能访问到它。要阻止它，您需要在下一步中将其地址列在 `egress.clusterCIDRs` 下。

    要打印该地址，请使用您的集群名称运行 `gcloud container clusters describe my-cluster --format 'value(endpoint)'`。

    chart 本身会阻止私有地址范围 `10.0.0.0/8`、`172.16.0.0/12`、`192.168.0.0/16` 和 `100.64.0.0/10`。如果您的集群有位于这四个范围之外的 pod、service 或节点地址范围，也请将该范围列出。
  </Step>

  <Step title="安装沙箱">
    将以下内容保存为 `cma-values.yaml`，填入您的镜像仓库和上一步中找到的地址：

    ```yaml
    platform: gke
    image:
      registry: us-docker.pkg.dev/my-project/cma   # REGISTRY in the build commands
    egress:
      mode: publicInternet
      clusterCIDRs:
        - 203.0.113.10/32                          # the cluster's API endpoint
      cmaServer:
        pods:
          namespace: cma-server
          matchLabels:
            anthropic.com/role: server
        port: 8400
    ```

    `egress.cmaServer` 会打开通往您服务器的路径。要将 `publicInternet` 与另外两种模式进行比较，请参阅[沙箱可以访问的内容](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#what-sandboxes-can-reach)。

    如果您的集群使用以下 DNS 设置之一，请在该文件中添加一行：

    * **Cloud DNS for GKE（Autopilot 集群即使用它）：** `gke: {dns: cloudDns}`
    * **NodeLocal DNSCache：** `gke: {dns: nodeLocalCache, kubeDnsUpstreamIP: <address>}`。要打印该地址，请运行 `kubectl -n kube-system get svc kube-dns-upstream -o jsonpath='{.spec.clusterIP}'`。

    然后安装该 chart：

    ```bash
    helm install cma kubernetes/chart --namespace cma -f cma-values.yaml --wait
    ```
  </Step>
</Steps>

## 运行会话

这些步骤使用的是新安装的名称。如果您的服务器 values 文件中包含 `earlierNames: true`，请在 [Kubernetes：您的对象名称](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#object-names)中查找应改用的名称。

<Steps>
  <Step title="转发服务器的端口">
    在一个终端中，将服务器的端口转发到您的机器：

    ```bash
    kubectl -n cma-server port-forward svc/cma-server 8400:8400
    ```
  </Step>

  <Step title="将 CLI 指向您的服务器">
    在第二个终端中，将 `ant` CLI 指向转发的端口，并从 `cma-server-keys` Secret 中读取服务器 API 密钥：

    ```bash
    export ANTHROPIC_BASE_URL=http://127.0.0.1:8400
    export ANTHROPIC_API_KEY=$(kubectl -n cma-server get secret cma-server-keys -o jsonpath='{.data.CMA_SERVER__API_KEY}' | base64 -d)
    ```
  </Step>

  <Step title="创建智能体并让其运行命令">
    按照快速入门中[运行您的第一个会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/quickstart#run-your-first-session)的三个步骤操作：创建智能体、启动会话，以及让智能体运行命令。在启动会话步骤中，请将 `compose-sandbox` 替换为 `default`，即 chart 所创建的环境的名称。
  </Step>

  <Step title="查找沙箱 pod">
    智能体会回复运行该命令的 pod 的主机名。在第三个终端中，`kubectl -n cma get pods` 会在 dispatcher 的 pod 旁边列出一个同名的 pod。默认情况下，智能体的轮次结束约 5 分钟后，该 pod 就会消失。如果它不在列表中，请让智能体再次运行 `hostname`。

    要退出聊天，请按 Ctrl+C。
  </Step>
</Steps>

## 在服务器前部署 TLS

服务器在端口 8400 上使用纯 HTTP 通信，而来自您应用程序的请求会携带服务器 API 密钥。在集群外部的客户端连接之前，请在 `cma-server` Service 前面放置一个终止 TLS 的负载均衡器或 ingress。它必须：

* 原样传递请求路径。
* 接受最大 32 MiB 的请求体。
* 以非缓冲方式传递响应。
* 将默认读取超时设置为 2 分钟。对于 Git 请求，为匹配 `^/cma-[a-z]+/git/` 的路径将读取超时设置为 10 分钟。
* 仅将流量发送到已就绪的服务器。`cma-server` Service 已经会这样做。直接访问 pod 的负载均衡器必须在每个 pod 上探测 `/cma-server/readyz`，并跳过返回 `503` 的 pod。
* 绝不重新发送服务器已经接受的 `POST`。
* 绝不缓存响应。

要从集群外部进行检查，请运行 `curl -s https://<host>/cma-server/healthz`。响应为 `{"status":"ok"}`。

## 沙箱可以访问的内容

在 `cma-values.yaml` 中，`egress.mode: publicInternet` 允许沙箱访问公共互联网。

下表将其与另外两种模式进行了比较：

| `egress.mode` | 沙箱可以访问的内容 |
| ----- | ----- |
| `closed`（默认值） | 您的服务器和 dispatcher，且无法访问任何 DNS 服务器 |
| `publicInternet` | 您的服务器、dispatcher、DNS 服务器以及公共互联网 |
| `gateway` | 您的服务器、dispatcher，以及通过出口网关访问您列出的主机。请参阅[在 Kubernetes 上启用网关](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/agent-identity#turn-on-the-gateway-on-kubernetes)。 |

## 在 GKE 上设置插件

版本 0.16.0 需要一个运行 GKE 1.35 的集群以及 Agent Sandbox 插件的 `v1alpha1` API。GKE 无法将集群退回到更早的次要版本。如果您的集群运行的是 1.36 或更高版本，请创建一个新集群。新集群中没有您现有会话的 `/workspace` 卷，因此请在删除旧集群之前复制出您需要的所有文件。

从 GKE 1.36.3-gke.1767000 开始，该插件改用 `v1beta1`，当集群升级到该版本或更高版本时，[现有的 `v1alpha1` 工作负载可能会失败](https://docs.cloud.google.com/kubernetes-engine/docs/troubleshooting/known-issues#agentsandbox-v1alpha1-workload-failures)。

1. 使用 `gcloud beta container clusters update <cluster> --enable-agent-sandbox` 启用该插件。
2. 检查 `kubectl api-versions | grep agents.x-k8s.io` 是否列出了 `agents.x-k8s.io/v1alpha1`。更高的 GKE 版本也可能列出它，因此还请确认集群运行的是 1.35。
3. 由于 GKE 会自动升级集群，请添加一个范围为 `no_minor_upgrades` 的[维护排除项](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/maintenance-windows-and-exclusions#configuring_a_maintenance_exclusion)。

## 其他集群

在 `cma-values.yaml` 中，将 `platform` 设置为 `eks`（用于 Amazon EKS）、`aks`（用于 Azure Kubernetes Service (AKS)）或 `generic`（用于任何其他集群）。这三者都需要上游 Agent Sandbox 控制器及其扩展，以及强制执行 NetworkPolicy 的网络插件。此外还需要：

* **Amazon EKS：** 一个节点组，其中 gVisor 的 `runsc` 已注册为名为 `runsc` 的 containerd 处理程序。为这些节点添加标签 `cma.anthropic.com/sandbox-node=true`，并添加污点 `cma.anthropic.com/sandbox-node=true:NoSchedule`。chart 会针对该处理程序注册 `gvisor` RuntimeClass。
* **AKS：** 一个启用了 Pod Sandboxing 的节点池，其标签和污点与 Amazon EKS 上的相同。将集群网络策略引擎的名称（`cilium`、`calico` 或 `azure`）填入 `aks.networkPolicy`。保持 `egress.mode: publicInternet`，这是该 chart 在 AKS 上唯一接受的模式。在智能体于其上执行实际工作之前，请先在您的集群上测试该 chart。
* **任何其他集群：** 一个用于 gVisor 或其他沙箱化运行时的 RuntimeClass。将其名称填入 `runtimeClass.name`。

归档中的 `kubernetes/chart/examples/` 文件夹为每种类型的集群提供了完整的 values 文件。

## 释放会话的卷

按照 chart 的默认设置，每个会话都有一个用于 `/workspace` 的 10 GiB 卷。会话的 pod 消失后，该卷仍会保留，因此会话的下一条消息可以找到其文件。

| 要释放的内容 | 操作方法 |
| ----- | ----- |
| 单个会话的卷 | 在会话的轮次结束后将其归档。dispatcher 通常会在约 5 分钟内删除该卷。 |
| 无人归档的会话的卷 | 在 `cma-values.yaml` 中设置 `dispatcher.retentionDays`。之后，当会话空闲达到该天数时，dispatcher 会删除其卷，而该会话的下一条消息将以空的 `/workspace` 开始。chart 的默认值 `0` 会保留所有卷。 |

本节中的命令适用于 GKE。在其他集群上，请使用 `sandboxclaims` 代替 `sandboxes`。

删除会话可能不会释放其卷，因此请先归档该会话。仅当 `kubectl -n cma get sandboxes -l anthropic.com/session-id=<session-id>` 没有列出任何内容时，才删除该会话。如 [Kubernetes：已删除会话的卷](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#deleted-sessions-volumes)中所述，升级可能会删除已删除会话的卷。

如果您删除了会话但未将其归档，且其卷仍然存在，请自行释放该卷：

1. 在 dispatcher 的日志中查找会话的 ID。此命令可能会为每个看起来已被删除但仍有卷的会话打印一行，ID 位于 `session_id` 之后：

   ```bash
   kubectl -n cma logs deployment/cma-dispatcher --since=6m \
     | grep "could not check the claim's session" | grep "403 Forbidden"
   ```

   dispatcher 每 5 分钟写入一次这些行。如果该会话未列出，请等待 5 分钟后再次运行该命令。如果它仍未列出，请运行 `kubectl -n cma get sandboxes -l anthropic.com/session-id=<session-id>`。如果没有列出任何内容，则该卷已消失。如果列出了一个沙箱，请使用该 ID 继续执行第 2 步。

2. 确认该会话已被删除。在按照[运行会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#run-a-session)中的方式转发端口并设置 `ANTHROPIC_API_KEY` 后，运行 `ant --base-url http://127.0.0.1:8400 beta:sessions retrieve --session-id <session-id>`。仅当响应为带有 `Session not found` 的 `404 Not Found` 时才继续。如果该命令打印出一个会话，请停止：该会话并未被删除。

3. 删除拥有该卷的 Kubernetes 对象。会话的文件会随之一起删除：

   ```bash
   kubectl -n cma delete sandboxes -l anthropic.com/session-id=<session-id>
   ```

## 续期密钥和证书

环境密钥的有效期为 365 天，dispatcher 的证书有效期为 825 天。请在它们到期之前分别续期。

在环境密钥有效期的最后 30 天内，服务器会在每个使用该密钥的请求上记录一条警告。该警告以 `environment key is within 30 days of expiry` 开头。

### 续期 chart 创建的环境密钥

要查看密钥的到期时间，请运行 `kubectl -n cma describe secret cma-environment`。

请在密钥有效期的最后 60 天内，从已安装的发布版本的归档顶层目录中至少运行一次以下两条命令：

```bash
helm upgrade cma-server kubernetes/server/chart --namespace cma-server -f cma-server-values.yaml --wait
kubectl -n cma rollout restart deployment/cma-dispatcher
```

每次运行服务器 chart 的 `helm upgrade` 命令时，如果密钥将在 60 天内到期，它都会在 Secret 中存储一个新密钥。除此之外，没有任何操作会续期该密钥。第二条命令会重启 dispatcher，使其读取新密钥。

### 续期 dispatcher 的证书

返回"创建沙箱的命名空间和证书"步骤。重新运行其中的 `openssl` 命令及其后的两条 `kubectl` 命令。然后使用 `kubectl -n cma rollout restart deployment/cma-dispatcher` 重启 dispatcher。

### 为您自己的 worker 替换环境密钥

chart 只会续期它们自己的密钥。如果您自行创建了密钥（如[创建环境及其密钥](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api#create-an-environment-and-its-key)中所述），请使用管理 CLI 替换它。该 CLI 位于发布归档的顶层。

请在[运行会话](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#run-a-session)中的第二个终端中运行这些命令，其中 `ANTHROPIC_API_KEY` 保存着服务器 API 密钥。请在第一个终端中保持端口转发运行。如果您通过其他地址访问服务器，请将该地址作为 `-base-url` 传入，此时无需端口转发。

<Steps>
  <Step title="为同一环境创建新密钥">
    对于 `-env`，请传入该环境的 ID：

    ```bash
    echo "$ANTHROPIC_API_KEY" | ./cma-server-admin mint-key \
      -base-url http://127.0.0.1:8400 -env env_01595EKxaaTTGwwY3kyXdtbs -key-file -
    ```

    该命令会打印新密钥，并在标准错误输出中打印该密钥的 ID 和到期时间。
  </Step>

  <Step title="将您的 worker 迁移到新密钥">
    为每个轮询的进程将新密钥设置为 `ANTHROPIC_ENVIRONMENT_KEY`，然后重启该进程。
  </Step>

  <Step title="吊销旧密钥">
    仍在使用已吊销密钥的 worker 会收到 `401` 错误，并放弃其正在服务的会话。要查找旧密钥的 ID，请使用相同的 `-base-url`、`-env` 和 `-key-file` 标志运行 `list-keys`。当不再有 worker 使用旧密钥时，按该 ID 将其吊销：

    ```bash
    echo "$ANTHROPIC_API_KEY" | ./cma-server-admin revoke-key \
      -base-url http://127.0.0.1:8400 -env env_01595EKxaaTTGwwY3kyXdtbs \
      -key-id etok_01AbCdEfGhIjKlMnOpQrStUv -key-file -
    ```
  </Step>
</Steps>

## 卸载栈

请先删除沙箱。`helm uninstall` 会删除限制沙箱的网络策略，但 dispatcher 为会话创建的沙箱会继续运行。

这些步骤不会影响您的数据库，但会删除 Secret。要再次使用该数据库，您需要保留的加密密钥副本。

<Steps>
  <Step title="停止 dispatcher 并删除沙箱">
    ```bash
    kubectl -n cma scale deployment/cma-dispatcher --replicas=0
    kubectl -n cma delete sandboxwarmpool cma-sandbox
    kubectl -n cma delete sandboxclaims --all
    ```

    在 GKE 上，还需运行以下命令：

    ```bash
    kubectl -n cma delete sandboxes --all
    ```
  </Step>

  <Step title="等待沙箱消失">
    重复运行此命令，直到沙箱、其 pod 及其卷都已消失：

    ```bash
    kubectl -n cma get sandboxes,pods,pvc
    ```
  </Step>

  <Step title="卸载 chart 并删除命名空间">
    ```bash
    helm uninstall cma --namespace cma
    helm uninstall cma-server --namespace cma-server
    kubectl delete namespace cma cma-server
    ```
  </Step>
</Steps>

## 后续步骤

* **升级：** [升级自托管 Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade)中提供了 `helm upgrade` 命令。
* **更改其他 chart 值：** 请参阅服务器的 `kubernetes/server/chart/values.yaml` 和沙箱的 `kubernetes/chart/values.yaml` 中的注释。
* **从代码中调用 API：** 请参阅[在自托管服务器上调用 API](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api)。
* **在浏览器中工作：** 请参阅[在您的服务器上使用控制台](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console)。
