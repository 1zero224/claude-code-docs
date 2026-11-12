---
title: MCP 隧道快速入门
url: https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/quickstart
description: 使用本地 Docker Compose 部署将 Claude 连接到私有 MCP 服务器。
---

<Note>
  “MCP tunnels”（MCP 隧道）目前处于"research preview"（研究预览）阶段。[申请访问权限](https://claude.com/form/mcp-tunnels)即可试用。
</Note>

本快速入门将引导您从创建隧道开始，直到 Claude 通过该隧道调用私有 MCP 服务器。它使用 Docker Compose 和[手动](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#credential-provisioning)凭据配置（credential provisioning），这是进行本地测试的最短路径。对于生产部署，请参阅[使用 Helm 部署](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm)或[使用 Docker Compose 部署](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-compose)。

## 您将构建的内容

一个由两个容器组成的 “tunnel stack”（[隧道栈](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)）（[代理](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)和 [cloudflared](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)），以及一个与之一起运行的示例 MCP 服务器。当一切运行起来后，即使没有任何服务在公共端口上监听，Claude 也可以通过 `https://echo.<your-tunnel-domain>/mcp` 访问该示例服务器。

## 您需要准备的内容

* 一台具有出站互联网访问权限的机器，并安装 [Docker 和 Docker Compose](https://docs.docker.com/get-docker/)。
* 在 [Claude Console](https://platform.claude.com/) 中拥有可以管理 MCP 隧道的角色。请参阅 [Console 指南的前提条件](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#prerequisites)。
* [OpenSSL](https://openssl-library.org/source/) 1.1.1 或更高版本。macOS 和大多数 Linux 发行版已预装。在 Windows 上，需要单独安装（`openssl` 二进制文件必须位于您的 `PATH` 中）。

<Steps>
  <Step title="创建隧道">
    在 Claude Console 中，前往 [MCP 隧道](https://platform.claude.com/settings/workspaces/default/mcp-tunnels)页面（`platform.claude.com/settings/workspaces/default/mcp-tunnels`）。每个隧道属于一个工作区。在侧边栏的工作区选择器中选择您需要的工作区。点击**新建隧道**并为其命名。保持**设置程序化访问**关闭；本快速入门使用手动凭据配置。

    创建完成后，打开该隧道。从**连接**部分复制两个值：

    * **域名**（形如 `abcd1234.tunnel.anthropic.com`）
    * **令牌**（点击眼睛图标，然后复制）
  </Step>

  <Step title="设置部署目录">
    <Tabs>
      <Tab title="macOS / Linux">
        ```bash
        mkdir -p mcp-tunnel/{config,data}
        cd mcp-tunnel
        export TUNNEL_DOMAIN=YOUR_TUNNEL_DOMAIN_HERE   # from step 1
        export TUNNEL_TOKEN='eyJ...'            # from step 1
        ```
      </Tab>

      <Tab title="Windows (PowerShell)">
        ```powershell
        New-Item -ItemType Directory -Force -Path mcp-tunnel/config, mcp-tunnel/data | Out-Null
        Set-Location mcp-tunnel
        $env:TUNNEL_DOMAIN = "YOUR_TUNNEL_DOMAIN_HERE"   # from step 1
        $env:TUNNEL_TOKEN  = "eyJ..."             # from step 1
        ```
      </Tab>
    </Tabs>
  </Step>

  <Step title="生成 CA 和服务器证书">
    代理使用由您控制的 CA 签名的证书来终止 “inner TLS”（[内层 TLS](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/concepts#components)）。生成这两者：

    <Tabs>
      <Tab title="macOS / Linux">
        ```bash
        openssl req -x509 -newkey rsa:2048 -nodes \
          -keyout data/ca.key -out data/ca.crt \
          -days 3650 -subj "/CN=mcp-tunnel-ca" \
          -addext "basicConstraints=critical,CA:TRUE" \
          -addext "keyUsage=critical,keyCertSign,cRLSign" \
          -addext "subjectKeyIdentifier=hash"

        cat > data/tls.ext <<EOF
        subjectAltName = DNS:${TUNNEL_DOMAIN},DNS:*.${TUNNEL_DOMAIN}
        authorityKeyIdentifier = keyid,issuer
        extendedKeyUsage = serverAuth
        EOF

        openssl req -newkey rsa:2048 -nodes \
          -keyout data/tls.key -out /tmp/server.csr \
          -subj "/CN=${TUNNEL_DOMAIN}"
        openssl x509 -req -in /tmp/server.csr \
          -CA data/ca.crt -CAkey data/ca.key -CAcreateserial \
          -out data/tls.crt -days 90 -extfile data/tls.ext

        chmod 644 data/tls.key
        ```
      </Tab>

      <Tab title="Windows (PowerShell)">
        ```powershell
        openssl req -x509 -newkey rsa:2048 -nodes `
          -keyout data/ca.key -out data/ca.crt `
          -days 3650 -subj "/CN=mcp-tunnel-ca" `
          -addext "basicConstraints=critical,CA:TRUE" `
          -addext "keyUsage=critical,keyCertSign,cRLSign" `
          -addext "subjectKeyIdentifier=hash"

        @"
        subjectAltName = DNS:$env:TUNNEL_DOMAIN,DNS:*.$env:TUNNEL_DOMAIN
        authorityKeyIdentifier = keyid,issuer
        extendedKeyUsage = serverAuth
        "@ | Set-Content -NoNewline -Encoding ascii -Path data/tls.ext

        openssl req -newkey rsa:2048 -nodes `
          -keyout data/tls.key -out data/server.csr `
          -subj "/CN=$env:TUNNEL_DOMAIN"
        openssl x509 -req -in data/server.csr `
          -CA data/ca.crt -CAkey data/ca.key -CAcreateserial `
          -out data/tls.crt -days 90 -extfile data/tls.ext
        ```
      </Tab>
    </Tabs>

    回到 Console，在隧道详情页面上点击**添加证书**并上传 `data/ca.crt`（或粘贴其内容）。隧道名称旁边的**需要证书**标记随即消失。
  </Step>

  <Step title="编写示例 MCP 服务器">
    <Tabs>
      <Tab title="macOS / Linux">
        ```bash
        cat > hello_server.py <<'EOF'
        from mcp.server.fastmcp import FastMCP

        mcp = FastMCP("hello-server", host="0.0.0.0", port=9000)


        @mcp.tool()
        def hello(name: str = "world") -> str:
            """Say hello to someone."""
            return f"Hello, {name}!"


        if __name__ == "__main__":
            mcp.run(transport="streamable-http")
        EOF
        ```
      </Tab>

      <Tab title="Windows (PowerShell)">
        ```powershell
        @'
        from mcp.server.fastmcp import FastMCP

        mcp = FastMCP("hello-server", host="0.0.0.0", port=9000)


        @mcp.tool()
        def hello(name: str = "world") -> str:
            """Say hello to someone."""
            return f"Hello, {name}!"


        if __name__ == "__main__":
            mcp.run(transport="streamable-http")
        '@ | Set-Content -NoNewline -Encoding ascii -Path hello_server.py
        ```
      </Tab>
    </Tabs>
  </Step>

  <Step title="编写代理配置和 compose 文件">
    <Tabs>
      <Tab title="macOS / Linux">
        ```bash
        cat > config/mcp-proxy.yaml <<EOF
        listen_addr: ":8080"
        tunnel_domain: ${TUNNEL_DOMAIN}
        tls:
          cert_file: /data/tls.crt
          key_file: /data/tls.key
        routes:
          echo: http://hello-mcp:9000
        EOF

        cat > docker-compose.yaml <<'EOF'
        services:
          mcp-proxy:
            image: us-docker.pkg.dev/anthropic-public-registry/images/mcp-proxy@sha256:d243d30642d39857a0579e465cb3b9bf3ae7c1141e83143bb6856777a31467a5
            volumes:
              - ./config/mcp-proxy.yaml:/etc/mcp-gateway/config.yaml:ro
              - ./data:/data:ro
            restart: unless-stopped

          cloudflared:
            image: cloudflare/cloudflared@sha256:0e56c13c7058fcef9c4f0e7d83372f82330548dbf8f8b366f3d8b506526d93b2
            command: tunnel --no-autoupdate run --url http://localhost:8080
            environment:
              - TUNNEL_TOKEN
            network_mode: "service:mcp-proxy"
            restart: unless-stopped

          hello-mcp:
            image: python:3.13-slim
            working_dir: /app
            volumes:
              - ./hello_server.py:/app/hello_server.py:ro
            command: sh -c "pip install --quiet mcp && python hello_server.py"
            restart: unless-stopped
        EOF
        ```
      </Tab>

      <Tab title="Windows (PowerShell)">
        ```powershell
        @"
        listen_addr: ":8080"
        tunnel_domain: $env:TUNNEL_DOMAIN
        tls:
          cert_file: /data/tls.crt
          key_file: /data/tls.key
        routes:
          echo: http://hello-mcp:9000
        "@ | Set-Content -NoNewline -Encoding ascii -Path config/mcp-proxy.yaml

        @'
        services:
          mcp-proxy:
            image: us-docker.pkg.dev/anthropic-public-registry/images/mcp-proxy@sha256:d243d30642d39857a0579e465cb3b9bf3ae7c1141e83143bb6856777a31467a5
            volumes:
              - ./config/mcp-proxy.yaml:/etc/mcp-gateway/config.yaml:ro
              - ./data:/data:ro
            restart: unless-stopped

          cloudflared:
            image: cloudflare/cloudflared@sha256:0e56c13c7058fcef9c4f0e7d83372f82330548dbf8f8b366f3d8b506526d93b2
            command: tunnel --no-autoupdate run --url http://localhost:8080
            environment:
              - TUNNEL_TOKEN
            network_mode: "service:mcp-proxy"
            restart: unless-stopped

          hello-mcp:
            image: python:3.13-slim
            working_dir: /app
            volumes:
              - ./hello_server.py:/app/hello_server.py:ro
            command: sh -c "pip install --quiet mcp && python hello_server.py"
            restart: unless-stopped
        '@ | Set-Content -NoNewline -Encoding ascii -Path docker-compose.yaml
        ```
      </Tab>
    </Tabs>
  </Step>

  <Step title="启动">
    <Tabs>
      <Tab title="macOS / Linux">
        ```bash
        docker compose up -d
        docker compose logs mcp-proxy | grep "route configured"
        docker compose logs cloudflared | grep "Registered tunnel connection"
        ```
      </Tab>

      <Tab title="Windows (PowerShell)">
        ```powershell
        docker compose up -d
        docker compose logs mcp-proxy | Select-String "route configured"
        docker compose logs cloudflared | Select-String "Registered tunnel connection"
        ```
      </Tab>
    </Tabs>

    您应该会看到一行针对 `echo` 的 `route configured`，以及四行 `Registered tunnel connection`。容器需要几秒钟才能启动；如果日志命令返回为空，请重新运行。
  </Step>

  <Step title="从 Claude 调用">
    在 Console 中，前往[会话](https://platform.claude.com/sessions)页面（`platform.claude.com/sessions`）并点击**创建会话**。在**智能体**下拉菜单中，选择**新建智能体**，为代理命名，并保留预填的模型。在 **MCP 服务器**下，点击 **MCP 服务器**，选择您的隧道，并将**子域名**设置为 `echo`，将**路径**设置为 `mcp`。示例服务器不需要身份验证，因此请忽略卡片上的**连接**或 **Needs vault**。选择一个环境。如果出现警告，提示该环境阻止 MCP 服务器，请参阅[在智能体中使用隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/console#use-the-tunnel-in-an-agent)。点击**创建会话**，然后提问：

    > Use the hello tool to greet tunnel.

    您应该会看到一次工具调用及其结果。
  </Step>
</Steps>

## 后续步骤

隧道已完成端到端验证。要换用您自己的 MCP 服务器，请将其添加到 `docker-compose.yaml`（或在同一 Docker 网络上运行它），在 `config/mcp-proxy.yaml` 中为其添加路由，然后重启代理（`docker compose restart mcp-proxy`）。

对于生产部署：

<CardGroup cols={2}>
  <Card title="使用 Docker Compose 部署" icon="cube" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-compose">
    经过加固的单主机部署，可选择是否启用编程访问。
  </Card>

  <Card title="使用 Helm 部署" icon="stack" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/deploy-helm">
    具有自动凭据管理功能的 Kubernetes 部署。
  </Card>
</CardGroup>
