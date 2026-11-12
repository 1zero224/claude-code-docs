---
title: CLI 身份验证选项
url: https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/authentication
description: 使用交互式登录、API 密钥、命名配置文件和工作负载身份联合对 ant CLI 进行身份验证。
---

`ant` CLI 支持多种凭据来源。[快速入门](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart#authentication)介绍了一条命令即可完成的登录方式（`ant auth login`）。本页完整介绍每一种选项。

## 交互式登录

`ant auth login` 让您无需创建或管理 API 密钥即可调用 API。它会针对 Claude Console 打开一个基于浏览器的 OAuth 流程，并将生成的凭据存储在 `$ANTHROPIC_CONFIG_DIR` 下（有关特定操作系统的默认值，请参阅[配置目录](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#configuration-directory)）。在远程主机上或任何没有本地浏览器的环境中，传递 `--no-browser` 以打印授权 URL，并将返回的代码粘贴回终端。

```bash CLI
ant auth login

# 在没有浏览器的远程主机上：
ant auth login --no-browser

# 绑定到特定工作区并跳过浏览器选择器：
ant auth login --workspace-id wrkspc_01...

# 如果您通过 --profile 传入的命名配置文件不存在，
# 将使用该名称创建一个新的命名配置文件。
ant auth login --profile work
```

在浏览器流程中，您需要选择一个组织，然后选择一个[工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces)。颁发的令牌[限定于该工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#api-keys-and-resource-scoping)，因此 CLI 只能看到属于该工作区的资源。传递 `--workspace-id` 可直接绑定并跳过选择器。要在多个工作区中工作，请参阅[在工作区之间切换](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/authentication#switch-between-workspaces)。

交互式登录适用于您自己机器上的本地开发和脚本编写。对于 CI、服务器和容器等非交互式工作负载，请改用[工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)。

登录会将凭据写入 `credentials/<profile>.json`。配置文件的首次登录还会创建 `configs/<profile>.json` 并将其设置为活动配置文件。要删除存储的凭据，请运行 `ant auth logout`，或运行 `ant auth logout --all` 以清除每个配置文件。

## 管理员访问

默认情况下，`ant auth login` 请求一个限定于工作区的令牌。要管理 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 页面上记录的资源，请在专用配置文件下请求 `org:admin` 范围：

```bash CLI
ant auth login --profile admin --scope "org:admin"

# 打印用于 Authorization 标头的 bearer 令牌：
ant auth print-credentials --profile admin --access-token
```

`org:admin` 范围仅授予具有管理员、所有者或主要所有者角色的组织成员。颁发的令牌具有组织范围的访问权限，配置文件上的任何工作区绑定都不会对其进行约束。请将管理员配置文件与您的日常配置文件分开，这样常规命令就永远不会以提升的访问权限运行。

## API 密钥

CLI 也会从 `ANTHROPIC_API_KEY` 环境变量中读取您的 API 密钥。请从 [Claude Console](https://platform.claude.com/settings/keys) 获取密钥。

<Tabs>
  <Tab title="zsh">
    ```bash
    echo 'export ANTHROPIC_API_KEY=sk-ant-api03-...' >> ~/.zshrc
    source ~/.zshrc
    ```
  </Tab>

  <Tab title="bash">
    ```bash
    echo 'export ANTHROPIC_API_KEY=sk-ant-api03-...' >> ~/.bashrc
    source ~/.bashrc
    ```
  </Tab>

  <Tab title="Windows">
    ```powershell
    setx ANTHROPIC_API_KEY "sk-ant-api03-..."
    ```

    打开一个新终端以使更改生效。
  </Tab>
</Tabs>

要为单次运行覆盖密钥，请传入 [`--api-key-stdin`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/using#global-flags) 并将密钥通过管道传给命令。要指向不同的 API 主机，请设置 `ANTHROPIC_BASE_URL` 或传入 `--base-url`。

如果您使用的是未限定到某个工作区的[个人或服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)，则必须[指定运行命令所在的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)。为此，可以设置 CLI 会自动读取的 `ANTHROPIC_WORKSPACE_ID` 环境变量，或使用 [`--workspace-id` 标志](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/using#global-flags)。该值必须是 `wrkspc_...` ID。

```bash CLI
ant messages create \
  --workspace-id wrkspc_01... \
  --model claude-opus-5-5 \
  --max-tokens 1024 \
  --message '{role: user, content: "Hello, Claude"}'
```

## 检查身份验证状态

`ant auth status` 会打印 CLI 选择的凭据来源（API 密钥环境变量、OAuth 登录、联合身份验证、Google Cloud 或配置文件）、活动配置文件、活动令牌绑定的工作区以及配置目录路径。使用它来诊断工作负载为何选择了错误的凭据或工作区。

```bash CLI
ant auth status
```

```text
Active profile:  default
Config dir:      ~/.config/anthropic
Profile config:  ~/.config/anthropic/configs/default.json
Credentials:     ~/.config/anthropic/credentials/default.json

Credentials
  (active) * Profile (user_oauth) [via active_config]       sk-ant-oat01-EXA...
...

Workspace
  (active) * Workspace                                      wrkspc_01... (Engineering)
```

阅读 `(active)` 行以查看哪个凭据来源和工作区胜出。该命令报告状态而不是执行健康检查，因此不要针对退出状态编写脚本。有关凭据来源的完整排序，请参阅[凭据优先级](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#credential-precedence)。

## 在工作区之间切换

交互式登录令牌绑定到单个工作区。要针对多个工作区使用 CLI，请在各自的命名配置文件下登录每个工作区，然后在它们之间切换：

```bash CLI
# 1. 创建配置文件（交互式；在浏览器中选择另一个工作区，
#    或传入 --workspace-id 以跳过选择器）：
# ant auth login --profile other-ws

# 2. 将其设为后续命令的默认值：
ant profile activate other-ws

# 3. 或在不更改默认值的情况下为单个命令选择它：
ant --profile other-ws models list
ANTHROPIC_PROFILE=other-ws ant models list
```

运行 [`ant auth status`](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/authentication#check-authentication-status) 以确认哪个配置文件和工作区处于活动状态。

<Note>
  环境中的 `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` 优先于所有配置文件（请参阅[凭据优先级](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#credential-precedence)）。在您取消设置之前，无论您选择哪个配置文件，API 请求都会使用该凭据，并且 `ant` 会向 stderr 打印一条说明此情况的消息。
</Note>

## 管理配置文件

`ant profile` 子命令可直接检查和编辑配置文件状态：

```bash CLI
ant profile list
ant profile get --profile other-ws
ant profile set workspace_id wrkspc_01... --profile other-ws
```

`ant profile set` 的可写键为 `workspace_id`、`base_url`、`organization_id`、`scope`、`client_id` 和 `console_url`。设置 `workspace_id` 会在配置文件配置中记录目标工作区，但不会重新绑定已颁发的凭据；请在该配置文件下再次运行 `ant auth login` 以为新工作区铸造令牌。

有关配置文件的文件架构和联合身份验证块，请参阅[配置文件配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#profile-configuration-file)。有关工作负载身份联合，请参阅[身份验证概述](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)和 [WIF 参考](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference)。

## 后续步骤

<CardGroup cols={3}>
  <Card title="使用 CLI" icon="terminal" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/using">
    命令结构、输出格式、GJSON 转换和请求正文
  </Card>

  <Card title="CLI 脚本编写和自动化" icon="code" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/scripting">
    API 资源的版本控制、脚本编写模式以及在 Claude Code 中的使用
  </Card>

  <Card title="工作负载身份联合" icon="cloud" href="https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation">
    用于 CI、服务器和容器的非交互式身份验证
  </Card>
</CardGroup>
