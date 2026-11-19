---
title: 在您的服务器上使用控制台
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console
description: 为自托管 Claude Managed Agents 服务器上的控制台设置登录，然后在浏览器中创建智能体并运行会话。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

服务器上的控制台是一个网页，由自托管的 Claude Managed Agents 服务器在与其 API 相同的地址上提供。在控制台中，您无需 SDK 或 CLI，即可创建智能体、运行会话，以及管理环境、凭据保管库（credential vault）、记忆存储（memory store）和技能。

它并不是位于 platform.claude.com 的 Claude Console。它只显示您服务器上的内容。

## 设置登录

管理员只需设置一次登录。如果只想在一台机器上试用控制台，请改为按照[在您服务器上试用控制台](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/quickstart#try-the-console-on-your-server)操作，该方法使用发行版中的模拟登录提供商。

<Steps>
  <Step title="设置单点登录和终端登录">
    请按照[单点登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#single-sign-on)操作，包括[让开发者从终端登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#let-developers-log-in-from-a-terminal)。控制台通过您为终端登录注册的应用程序让用户登录。它会根据颁发者条目中 `audiences` 的第一个条目检查每个令牌，因此请将该应用程序的受众（audience）列在首位。
  </Step>

  <Step title="通过 HTTPS 提供控制台">
    在端口 8400 前放置一个终止 TLS 的代理。[在服务器前放置 TLS](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/kubernetes#put-tls-in-front-of-the-server) 列出了代理必须执行的操作。如果使用纯 HTTP，登录仅在 `localhost` 或 `127.0.0.1` 上有效。
  </Step>

  <Step title="将控制台的地址添加到应用程序">
    在您的身份提供商中，将服务器的 HTTPS 地址（带尾部斜杠）添加为该应用程序的第二个重定向 URI：`https://cma.example.com/`。同时允许其源 `https://cma.example.com` 进行跨源（CORS）请求。
  </Step>
</Steps>

要检查设置，请[打开控制台](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console#open-the-console)并登录。

## 打开控制台

如果您的应用程序调用 `https://cma.example.com`，请在浏览器中打开 `https://cma.example.com/`，然后点击 **Continue with SSO**。

您通过身份提供商登录。控制台不接受服务器 API 密钥。如果控制台提示此服务器上未设置登录，则需要管理员[设置登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console#set-up-sign-in)。

## 从控制台运行智能体

空白智能体的内置工具（例如 `bash`）在工作器（worker）上运行，因此请选择一个有工作器正在轮询的环境。[各组件的运行位置](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted#what-runs-where)说明了工作器是什么。如果没有工作器，第一次工具调用会一直等待，直到有工作器执行它。环境由管理员创建，管理员可以在环境的 **Workers** 选项卡上查看其工作器。

<Steps>
  <Step title="创建智能体">
    在侧边栏中选择 **Agents**，然后点击 **Create agent**。选择一个起点，例如 **Blank agent**，然后输入名称。

    点击 **Create agent**。随后会打开该智能体的页面。
  </Step>

  <Step title="启动会话">
    在智能体页面上，点击 **Start session**。选择一个环境，然后点击 **Create session**。随后会打开该会话的页面。
  </Step>

  <Step title="发送消息">
    在页面底部的输入框中输入消息并发送。页面会随着智能体的工作而更新。如果某次工具调用需要等待您处理，其卡片会显示 **Approve** 和 **Deny**。
  </Step>
</Steps>

当某次工具调用正在等待您处理时，会话的徽章会显示 **Idle**。

要打开任何其他会话（例如应用程序通过 API 启动的会话），请在侧边栏中选择 **Sessions**，然后选择该会话。

## 谁能看到什么

与 API 中一样，您的角色决定了控制台显示的内容：

* **管理员**可以查看和更改服务器上的所有内容。
* **具有 `developer` 或 `user` 角色的用户**可以看到所有环境。在智能体、会话、保管库、记忆存储和技能中，他们只能看到自己以自己的登录身份创建的内容。

使用服务器 API 密钥创建的智能体或会话（例如快速入门中创建的）仅对管理员可见。有关完整表格，请参阅[各角色可执行的操作](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#what-each-role-can-do)。

## 关闭控制台

服务器默认提供控制台。要关闭它，请在[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)中设置 `console.serve: false`，然后重启服务器。之后服务器将停止提供控制台。智能体和会话仍可通过 API 继续工作。

## 后续步骤

* [在自托管服务器上调用 API](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/call-the-api)：通过您的代码或 `ant` CLI 完成相同的工作。
* [自托管 Claude Managed Agents 中的数据和访问](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-data-and-access#who-can-reach-the-server)：了解谁可以访问服务器及其控制台。
