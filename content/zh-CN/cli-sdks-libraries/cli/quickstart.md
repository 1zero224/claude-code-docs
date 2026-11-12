---
title: CLI 快速入门
url: https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart
description: 安装 ant 命令行工具，完成身份验证，并向 Claude API 发送您的第一个请求。
---

`ant` CLI 让您可以从终端访问 Claude API。每个 API 资源都以子命令的形式提供，并支持输出格式化、响应过滤以及 YAML 或 JSON 文件输入。

<Frame caption="The ant CLI in action（ant CLI 实际运行效果）。">
  <video aria-label="ant CLI 在终端中运行的屏幕录像。">
    <source src="https://platform.claude.com/docs/videos/ant-cli-demo.webm" type="video/webm" />
    <source src="https://platform.claude.com/docs/videos/ant-cli-demo.mp4" type="video/mp4" />
  </video>
</Frame>

与 `curl` 相比，`ant` 通过类型化的标志或管道传入的 YAML 构建请求体，而不是手写 JSON，并且可以通过 `@path` 引用将文件内容内联到字符串字段中。它使用内置的 `--transform` 查询提取响应字段，因此您无需使用 `jq` 等单独的工具，并且它会自动对列表端点进行分页。

<Info>
  有关特定端点的参数和响应模式，请参阅 [API 参考](https://platform.claude.com/docs/zh-CN/api/cli/messages/create)。本页面帮助您运行第一个可用的命令。有关 CLI 的其他所有功能，请参阅[使用 CLI](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/using) 和 [CLI 脚本编写与自动化](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/scripting)。
</Info>

## 安装

<Tabs noDeepLinks>
  <Tab title="Homebrew (macOS)">
    ```bash
    brew install anthropics/tap/ant
    ```
  </Tab>

  <Tab title="curl (Linux/WSL)">
    对于 Linux 环境，请直接下载发布版二进制文件。

    ```bash
    VERSION=2.9.0
    OS=$(uname -s | tr '[:upper:]' '[:lower:]')
    case $(uname -m) in
      x86_64) ARCH=amd64 ;;
      aarch64) ARCH=arm64 ;;
    esac
    curl -fsSL "https://github.com/anthropics/anthropic-cli/releases/download/v${VERSION}/ant_${VERSION}_${OS}_${ARCH}.tar.gz" \
      | sudo tar -xz -C /usr/local/bin ant
    ```

    您可以在 [GitHub 发布页面](https://github.com/anthropics/anthropic-cli/releases)上找到所有发布版本。
  </Tab>

  <Tab title="go install">
    您也可以使用 `go install` 从源代码安装 CLI，这需要 Go 1.26.10 或更高版本。

    ```bash
    go install github.com/anthropics/anthropic-cli/v2/cmd/ant@latest
    ```

    `go install` 会将二进制文件放置在 `$(go env GOPATH)/bin` 中。如果该目录尚未添加到您的 `PATH` 中，请将其添加进去：

    ```bash
    export PATH="$PATH:$(go env GOPATH)/bin"
    ```
  </Tab>
</Tabs>

检查安装情况：

```bash
ant --version
```

## 身份验证

`ant auth login` 会针对 Claude Console 打开基于浏览器的 OAuth 流程，并将生成的凭据存储在本地，因此您无需创建或管理 API 密钥即可调用 API。

```bash CLI
ant auth login
```

<Note>
  有关其他身份验证方式（API 密钥环境变量、无头主机、多个工作区、命名配置文件以及工作负载身份联合），请参阅 [CLI 身份验证选项](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/authentication)。
</Note>

## 发送您的第一个请求

安装二进制文件并完成身份验证后，调用 [Messages API](https://platform.claude.com/docs/zh-CN/api/cli/messages/create)：

```bash
ant messages create \
  --model claude-opus-5-5 \
  --max-tokens 1024 \
  --message '{role: user, content: "Hello, Claude"}'
```

```text Output wrap
{
  "model": "claude-opus-5-5",
  "id": "msg_01YMmR5XodC5nTqMxLZMKaq6",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello! How are you doing today? Is there something I can help you with?"
    }
  ],
  "stop_reason": "end_turn",
  "usage": { "input_tokens": 27, "output_tokens": 20 /*, ... */ }
}
```

响应是完整的 API 对象，由于 stdout 是终端，因此会以美化格式打印。

## Shell 补全

CLI 附带适用于 bash、zsh、fish 和 PowerShell 的补全脚本。为您的 shell 生成并安装一个补全脚本：

<Tabs>
  <Tab title="zsh">
    ```bash
    ant @completion zsh > "${fpath[1]}/_ant"
    # 重启您的 shell 或运行：autoload -U compinit && compinit
    ```
  </Tab>

  <Tab title="bash">
    ```bash
    ant @completion bash > /etc/bash_completion.d/ant
    ```
  </Tab>

  <Tab title="fish">
    ```bash
    ant @completion fish > ~/.config/fish/completions/ant.fish
    ```
  </Tab>

  <Tab title="PowerShell">
    ```powershell
    ant @completion powershell | Out-String | Invoke-Expression
    # 如需在各会话之间持久保留：
    # ant @completion powershell >> $PROFILE
    ```
  </Tab>
</Tabs>

## 后续步骤

<CardGroup cols={3}>
  <Card title="CLI 身份验证选项" icon="lock" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/authentication">
    API 密钥、无头主机、多个工作区和命名配置文件
  </Card>

  <Card title="使用 CLI" icon="terminal" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/using">
    命令结构、输出格式、GJSON 转换和请求体
  </Card>

  <Card title="CLI 脚本编写与自动化" icon="code" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/scripting">
    API 资源的版本控制、脚本编写模式以及在 Claude Code 中的使用
  </Card>
</CardGroup>
