---
title: 开始使用自托管 Claude Managed Agents
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/quickstart
description: 使用 Docker Compose 在一台机器上启动自托管 Claude Managed Agents，并在沙箱容器中运行第一个会话。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

在本快速入门中，您将使用 Docker Compose 在一台机器上启动自托管 Claude Managed Agents。Compose 栈包括服务器、其 PostgreSQL 数据库，以及一组 worker（即工作进程），每个会话都由 worker 在其专属的沙箱容器中运行。然后，您可以在终端中与智能体聊天，并确认其工具调用是在沙箱容器中运行的。随后，您可以在[您服务器上的控制台](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console)中打开该会话。

<Warning>
  Compose 栈用于开发环境，不适用于生产环境。它提供纯 HTTP 服务，并且默认仅在 Docker 主机的环回地址 `127.0.0.1` 上发布端口 8400。在 28.3.3 之前的某些 Docker Engine 发布版本下，主机所在本地网络中的机器也可以访问该端口。智能体工具在该主机上的容器中运行，因此请使用一台不存放任何需要保护内容的机器。
</Warning>

对于生产环境，Anthropic 建议使用发布版本中的两个 Helm chart。请参阅[在 Kubernetes 上部署](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes)。

让 Claude Code 为您运行本快速入门

```text
按照 https://platform.claude.com/docs/en/managed-agents/self-hosted/quickstart.md 在这台机器上设置自托管 Claude Managed Agents。我已经下载了发布归档。
```

如果您是代替某人运行这些步骤的智能体：

* **发布归档：** 询问对方归档所在的位置。只有对方才能下载它。
* **Claude API 密钥：** 让对方自行编辑 `.env`。不要索要该密钥，也不要打印 `.env`。
* **变量：** 如果您的 shell 不会在命令之间保留变量，请在每条 `ant` 命令中重新设置 `ANTHROPIC_BASE_URL` 和 `ANTHROPIC_API_KEY`。如果没有这些变量，`ant` 会调用托管服务。
* **Compose：** 在每条 `docker compose` 命令前加上 `env -u ANTHROPIC_BASE_URL -u ANTHROPIC_API_KEY -u ANTHROPIC_AUTH_TOKEN`，以免您 shell 中的值传递到服务器。
* **聊天：** `ant beta:sessions connect` 需要交互式终端。如果没有交互式终端，请使用 `ant beta:sessions:events send` 发送消息，并使用 `ant beta:sessions:events list` 读取回复。[发送事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#send-events)和[列出过去的事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#list-past-events)中提供了示例。
* **控制台：** 仅当对方要求时，才执行"在您服务器上试用控制台"这一部分。如果您的 shell 不会在命令之间保留变量，还需从该部分起在每条 `docker compose` 命令前加上 `COMPOSE_FILE=docker-compose.yml:docker-compose.oidc.yml`。

## 前提条件

* **发布归档：** 如需申请下载权限，请联系您的 Anthropic 客户团队。如果您的组织已拥有访问权限，请按照此页面顶部的说明操作。
* **Claude API 密钥：** 服务器使用它进行模型调用。请在 [Claude Console](https://platform.claude.com/settings/keys) 中创建一个。
* **Docker：** [最新版本的 Docker Engine](https://docs.docker.com/engine/install/)，以及 Compose 插件（`docker compose`，而不是旧版的 `docker-compose`）。
* **至少有两个 CPU 的主机：** x86-64 或 Arm。在较小的主机上，请在创建 `.env` 时将 `CMA_SANDBOX_CPUS` 设置为其 CPU 数量。
* **首次启动时的网络访问：** 首次构建会从 Docker Hub、Alpine 软件包仓库和 GitHub releases 拉取内容。
* **工具：** 在运行 Docker 的机器上，您需要 `openssl` 和 [`ant` CLI](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart#installation)。

## 启动栈

<Steps>
  <Step title="解压归档">
    解压归档，并在其 `compose` 目录中操作。在命令中，将 `<archive>` 替换为您的归档名称，不带 `.tar.gz`。

    ```bash
    tar -xzf <archive>.tar.gz && cd <archive>/compose
    ```

    如果您的 umask 严于 `022`，二进制文件会失去其他用户的执行权限，服务器容器将无法启动它们。在这种情况下，请将其设置为 `022` 并重新解压归档。
  </Step>

  <Step title="创建 .env 文件">
    Compose 从 `docker-compose.yml` 旁边的 `.env` 文件读取栈的机密信息。创建该文件：

    ```bash
    cat > .env <<EOF
    CMA_PG_PASSWORD=$(openssl rand -hex 32)
    ANTHROPIC_API_KEY=sk-ant-...
    CMA_SERVER__API_KEY=$(openssl rand -hex 32)
    CMA_STORE__CREDENTIALS_ENCRYPTION_KEY=$(openssl rand -base64 32)
    EOF
    chmod 600 .env
    ```

    在编辑器中打开 `.env`，并将 `sk-ant-...` 替换为您的 Claude API 密钥。该代码块会生成其余内容：数据库密码、您的应用程序发送的服务器 API 密钥，以及用于加密服务器所存储机密信息的密钥。
  </Step>

  <Step title="启动容器">
    构建并启动栈，然后跟踪服务器的日志：

    ```bash
    docker compose up --build -d
    docker compose logs -f cma-server
    ```

    当日志中出现包含 `msg="cma-server listening"` 的行时，按 Ctrl+C 停止跟踪日志。
  </Step>

  <Step title="检查服务器是否就绪">
    ```bash
    curl -s http://127.0.0.1:8400/cma-server/readyz
    ```

    就绪的服务器会返回 `"status":"ready"`。刚启动时，在首次检查运行期间，它可能会在几秒钟内返回 `"status":"degraded"`。如果一直处于 `degraded` 状态或返回 `"status":"not_ready"`，请参阅[故障排除](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/quickstart#troubleshooting)。
  </Step>
</Steps>

## 运行您的第一个会话

使用 `ant` CLI 从终端创建一个智能体并与其聊天。

<Steps>
  <Step title="将 CLI 指向您的服务器">
    在同一目录中打开第二个终端，并将其专用于 `ant`。以下命令会将 `ANTHROPIC_API_KEY` 设置为您的服务器 API 密钥，而不是您的 Claude API 密钥，如[在自托管服务器上调用 API](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api) 中所述：

    ```bash
    export ANTHROPIC_BASE_URL=http://127.0.0.1:8400
    export ANTHROPIC_API_KEY=$(grep '^CMA_SERVER__API_KEY=' .env | cut -d= -f2)
    ```

    请在第一个终端中运行 `docker compose` 命令。如果您在此终端中运行 `docker compose up`，Compose 会将这些变量传递给服务器，服务器将调用自身，而不是 Claude API。
  </Step>

  <Step title="创建智能体">
    将命令后面的文件保存为 `quickstart-agent.md`，然后运行该命令。该文件为智能体提供内置工具，并允许智能体无需先询问您即可运行这些工具。

    <CodeGroup>
      <CodeGroupItem>
        ```bash CLI
        ant apply --yes quickstart-agent.md
        ```

        <File filename="quickstart-agent.md">
          ```markdown
          ---
          name: Quickstart agent
          model: claude-opus-5-5
          tools:
            - type: agent_toolset_20260401
          ---

          You are a helpful assistant.
          ```
        </File>
      </CodeGroupItem>
    </CodeGroup>

    该命令会先打印一条警告，提示其 `could not determine the organization and workspace`，并附带 `404` 错误，然后再创建智能体。在您自己的服务器上，这是预期行为。
  </Step>

  <Step title="启动会话">
    这些命令会查找智能体和栈的环境，启动一个会话，并将会话的 ID 保存在 `SESSION_ID` 中。栈将其环境命名为 `compose-sandbox`。

    ```bash
    AGENT_ID=$(ant beta:agents list --format raw \
      --transform 'data.#(name=="Quickstart agent").id' --raw-output)
    ENVIRONMENT_ID=$(ant beta:environments list --format raw \
      --transform 'data.#(name=="compose-sandbox").id' --raw-output)
    SESSION_ID=$(ant beta:sessions create --agent "$AGENT_ID" \
      --environment-id "$ENVIRONMENT_ID" --transform id --raw-output)
    ```
  </Step>

  <Step title="让智能体运行命令">
    将您的终端连接到该会话：

    ```bash
    ant beta:sessions connect "$SESSION_ID"
    ```

    该命令会打开会话的[聊天视图](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/sessions-connect)。发送以下消息：

    ```text wrap
    Run `hostname` with the bash tool and tell me the output.
    ```

    智能体会回复一个以 `cma-sandbox-` 开头的主机名。这是运行该命令的沙箱容器的名称，证明该工具是在沙箱中运行的，而不是在您的机器上。
  </Step>

  <Step title="查找沙箱容器">
    保持聊天窗口打开。在第一个终端中，列出沙箱容器：

    ```bash
    docker ps --filter label=cma-server.sandbox
    ```

    列表中会显示智能体报告的容器。一轮对话结束后约一分钟，该容器会退出，会话的下一条消息会启动一个新容器。如果列表为空，请让智能体再次运行 `hostname`，并在其回复期间再次列出容器。

    要退出聊天，请在其终端中按 Ctrl+C。会话会继续运行。
  </Step>
</Steps>

## 在您服务器上试用控制台

本部分为可选内容。控制台需要单点登录。如需试用，请使用发布版本中的模拟登录提供商重新启动栈。您机器上的端口 8080 必须处于空闲状态。

<Warning>
  仅在使用可丢弃的数据时使用该模拟提供商。它会向任何能访问它的人提供管理员的令牌。在 Docker Desktop 上，这包括沙箱容器。在 28.3.3 之前的某些 Docker Engine 发布版本下，这包括您网络中的机器。
</Warning>

<Steps>
  <Step title="使用模拟登录提供商重新启动栈">
    在第一个终端中运行以下命令：

    ```bash
    export COMPOSE_FILE=docker-compose.yml:docker-compose.oidc.yml
    docker compose up --build -d
    ```

    `export` 行仅对第一个终端有效，因此请在您为 `docker compose` 打开的任何新终端中再次运行该行。如需更改此栈中服务器的设置，请编辑 `compose.config.oidc.yaml`，而不是 `compose.config.yaml`，然后运行 `docker compose restart cma-server`。
  </Step>

  <Step title="打开会话">
    在同一台机器上的浏览器中打开 `http://127.0.0.1:8400/`，点击 **Continue with SSO**，然后选择演示管理员 **alice**。在侧边栏中选择 **Sessions**，然后选择该会话。

    页面会随着智能体的工作而更新，您也可以从该页面向智能体发送消息。
  </Step>
</Steps>

如需同时在控制台中创建智能体并启动会话，请参阅[从控制台运行智能体](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console#run-an-agent-from-the-console)。

## 停止栈

在第一个终端中，运行以下命令：

```bash
docker compose down --remove-orphans
```

该命令会删除容器并保留数据库。如需同时删除数据库，以及其中的所有智能体、会话和会话历史，请运行 `docker compose down --remove-orphans -v`。

## 故障排除

* **`docker compose up` 停止并显示 `required variable CMA_PG_USER is missing a value`：** 如果是全新安装，请在 `.env` 中添加一行 `CMA_PG_USER=cma_server`，然后再次运行 `docker compose up --build -d`。如果是升级栈，请不要添加该行。请参阅[如果升级失败](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade#if-the-upgrade-fails)。

* **您的归档中没有 `compose` 目录：** 该归档包含的是较早的发布版本，本页面的命令与其不匹配。请向您的 Anthropic 客户团队索取当前发布版本。如需将正在运行的部署迁移到当前发布版本，请参阅[升级自托管 Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/upgrade)。

* **服务器始终无法启动：** 运行 `docker compose logs migrate`。一次性的 `migrate` 容器负责创建数据库表。它必须以状态 `0` 退出，服务器才会启动。

* **就绪检查一直处于 `degraded` 状态，或每一轮都失败：** 如果您在 `ANTHROPIC_BASE_URL` 为 `http://127.0.0.1:8400` 的终端中运行了 `docker compose up`，服务器会调用自身，而不是 Claude API。请在未设置该变量的第一个终端中再次运行 `docker compose up -d`。否则，服务器日志会显示失败的内容。如需查看日志，请再次运行 `docker compose logs -f cma-server`。

* **就绪检查返回 `not_ready`：** 在返回内容中，找到状态为 `failed` 的检查：

  * **`upstream`：** Claude API 拒绝了您的 Claude API 密钥。请更正 `.env` 中的 `ANTHROPIC_API_KEY`，然后在第一个终端中再次运行 `docker compose up -d`。如果您的 shell 也设置了 `ANTHROPIC_API_KEY`，Compose 会使用 shell 中的值，因此请先运行 `unset ANTHROPIC_API_KEY`。
  * **`database`：** 要么是数据库拒绝了服务器的登录（例如因为密码错误），要么是与数据库的已打开连接中断。如需确定是哪种情况，请再次运行 `docker compose logs -f cma-server`。

* **智能体有回复，但其工具调用始终未完成：** 没有 worker 在轮询该环境。使用 `docker compose logs sandbox` 检查 worker 的日志。无法访问 Docker 守护进程套接字的 worker 会退出，并显示一条指明 `CMA_DOCKER_SOCKET` 的消息。在 `.env` 中将该变量设置为您的守护进程套接字路径，然后在第一个终端中再次运行 `docker compose up -d`。

* **worker 正在运行，但工具调用仍始终未完成：** 栈会启动 10 个 worker，每个 worker 一次运行一个会话的容器。会话在创建时就会占用一个空闲 worker，而从未收到消息的会话最多会占用其 worker 一小时。当 10 个 worker 全部被占用时，其他任何会话的工具调用都会等待 worker。如果"查找沙箱容器"步骤中的 `docker ps` 命令列出了 10 个容器，请归档您不使用的会话：

  1. 在第一个终端中，运行 `docker ps --filter label=cma-server.sandbox --format '{{.Label "cma-server.session"}}'`。它会打印每个容器对应的会话 ID。正在等待的会话还没有容器，因此不在列表中。
  2. 在第二个终端中，按 Ctrl+C 退出聊天，并对列表中每个您不使用的会话运行 `ant beta:sessions archive --session-id <session-id>`。
  3. 再次运行 `ant beta:sessions connect "$SESSION_ID"`。当某个 worker 接手其会话时，工具调用就会完成。

## 后续步骤

<CardGroup cols={2}>
  <Card title="调用 API" icon="code-brackets" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api">
    将 SDK 和 `ant` CLI 指向您的服务器。
  </Card>

  <Card title="与托管服务之间的差异" icon="arrows-left-right" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/differences">
    查看哪些功能的行为有所不同或不可用。
  </Card>

  <Card title="您服务器上的控制台" icon="browser" href="https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console">
    设置登录，并在浏览器中创建智能体和运行会话。
  </Card>
</CardGroup>
