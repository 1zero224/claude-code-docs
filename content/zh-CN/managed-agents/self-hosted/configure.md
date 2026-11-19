---
title: 单点登录
url: https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure
description: 让用户和服务使用来自您的身份提供商的令牌调用您自托管的 Claude Managed Agents 服务器。
---

<Note>
  如需申请自托管 Claude Managed Agents 的访问权限，请联系您的 Anthropic 客户团队。如果您所属的 Claude Console 组织已获得访问权限，请[登录 Claude Console](https://platform.claude.com)并切换到该组织。然后重新加载此页面。
</Note>

默认情况下，您的服务器接受一个共享密钥，即[服务器 API 密钥](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted#the-three-credentials)。使用 “single sign-on”（单点登录）时，每个用户或服务还可以使用来自您的 OpenID Connect (OIDC) 身份提供商的令牌调用服务器。服务器根据令牌中的组为调用方分配角色。服务器 API 密钥仍可作为管理员凭据使用。

本页面介绍如何添加您的身份提供商、各角色可执行的操作，以及如何让开发者从终端登录。

## 添加您的身份提供商

<Steps>
  <Step title="在您的身份提供商中注册服务器">
    将服务器注册为一个 API，并使用类似 `api://cma-server` 的标识符，然后让客户端为其请求令牌。
  </Step>

  <Step title="在设置文件中列出身份提供商">
    在[设置文件](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#where-settings-go)的 `auth.issuers` 下添加一个条目：

    ```yaml
    auth:
      issuers:
        - name: okta
          issuer_url: https://login.example.com/oauth2/default
          audiences: [api://cma-server]
          roles:
            - group: cma-admins
              role: admin
            - group: "eng-*"
              role: developer
    ```

    * `name` 用于在日志行中标识该签发方，服务器还会将其记录为每个资源创建者的一部分。一旦有人创建了资源，就不要再更改它。
    * `issuer_url` 必须等于您令牌的 `iss` 声明。
    * `audiences` 列出令牌的 `aud` 声明可以携带的值，例如第一步中的标识符。
    * `roles` 将组映射到 `admin`、`developer` 或 `user`。匹配到的最高角色生效。

    有关其他字段，请参阅[签发方条目](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configuration-reference#issuer-entries)。如果您的提供商是 Microsoft Entra ID，另请参阅 [Microsoft Entra ID](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#microsoft-entra-id)。
  </Step>

  <Step title="重启服务器并检查登录">
    重启服务器。调用方在任何接受服务器 API 密钥的路由上，以 `Authorization: Bearer <token>` 的形式发送令牌。

    对于每个经令牌验证的请求，服务器都会记录 `auth=oidc`、签发方的 `name` 以及调用方的主体（`sub=`）。
  </Step>
</Steps>

### 如果登录失败

| 响应                 | 原因                   | 服务器日志行中包含             |
| ------------------ | -------------------- | --------------------- |
| `401`，与错误 API 密钥相同 | 令牌未通过检查。日志行中有具体原因。   | `oidc token`          |
| `403`              | 令牌的组未匹配任何 `roles` 条目 | `token holds no role` |

### Microsoft Entra ID

Entra ID 在两个方面与其他提供商不同。这两项设置都放在签发方条目中：

* **用户身份：** Entra ID 会在每个客户端应用程序中为同一个人提供不同的 `sub`，因此使用默认值会将一个人记录为多个创建者。请设置 `subject_claim: oid`。
* **组：** 如果某人属于很多组，Entra ID 会在令牌中省略 `groups` 声明。这样该用户就不会匹配任何 `roles` 条目。为避免这种情况，请让 Entra ID 仅发出分配给该应用程序的组。或者在 Entra ID 中为用户分配应用角色，并设置 `groups_claim: roles`，让服务器读取应用角色来代替组。

## 各角色可执行的操作

| 角色                   | 智能体、技能、会话、vault 和记忆存储 | 环境                |
| -------------------- | --------------------- | ----------------- |
| `admin` 以及服务器 API 密钥 | 查看和更改所有内容             | 创建、更改和删除环境，并创建其密钥 |
| `developer` 或 `user` | 仅查看和更改该调用方创建的内容       | 列出、读取环境，并在其中运行会话  |

`developer` 和 `user` 拥有相同的访问权限。拥有这两种角色之一的调用方：

* **无法看到他人创建的资源。** 该资源不会出现在列表中，对其 ID 的请求所得到的响应与对未知 ID 的请求相同。这也包括在您启用单点登录之前使用服务器 API 密钥创建的所有内容。
* **无法与其他调用方或组共享资源。**

请将 `admin` 映射到精确的组名。管理员还可以启动使用任何用户 vault 的会话。因此，请仅将该角色授予您信任其可接触每个用户 MCP 令牌的组。

## 让开发者从终端登录

开发者可以通过您的身份提供商登录，然后以自己的身份使用 `ant` CLI，而无需服务器 API 密钥。前两步由您完成，第三步由每位开发者完成。

<Steps>
  <Step title="在您的身份提供商中注册应用程序">
    注册一个具有以下特征的原生（"public"）应用程序：

    * 不设客户端密钥
    * 使用带 PKCE 的授权码授权方式
    * 重定向 URI 为 `http://127.0.0.1/callback`

    某些提供商需要使用[其他重定向 URI](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#other-redirect-uris) 之一。
  </Step>

  <Step title="将应用程序添加到签发方条目">
    在签发方条目中添加 `login_client_id` 和 `login_scopes`，然后重启服务器：

    ```yaml
    login_client_id: 0oa1b2c3d4e5f6g7h8i9
    login_scopes: [openid, cma-server.access]
    ```

    `login_scopes` 列出您的提供商签发 `aud` 声明位于签发方条目 `audiences` 列表中的令牌所需的作用域。
  </Step>

  <Step title="登录">
    每位开发者从解压后归档的顶层目录运行登录命令：

    ```bash
    ./cma-server-admin login https://cma.example.com
    ```

    服务器的 URL 必须使用 `https://`，或是指向 `localhost`、`127.0.0.1` 或 `::1` 的 `http://`。使用[其他重定向 URI](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#other-redirect-uris) 之一时，请在命令中添加相应的标志。

    该命令会在浏览器中打开提供商的登录页面，然后写入一个名为 `cma-server` 的 `ant` CLI 配置文件。在任何 `ant` 命令中传入 `--profile cma-server`，或运行 `ant profile activate cma-server` 将该配置文件设为默认。
  </Step>
</Steps>

令牌过期后，开发者需再次运行 `login`。要改为让 `ant` CLI 续期令牌，请将 `offline_access` 添加到 `login_scopes`。

### 其他重定向 URI

| 提供商                 | 需注册的重定向 URI                                     | 开发者需在 `login` 命令中添加的标志   |
| ------------------- | ----------------------------------------------- | ------------------------ |
| Microsoft Entra ID  | `http://localhost/callback`                     | `-listen localhost:0`    |
| 要求重定向 URI 中包含端口的提供商 | 使用固定端口的 URI，例如 `http://127.0.0.1:8123/callback` | `-listen 127.0.0.1:8123` |

要列出其他标志，请在 `login` 命令中添加 `-h`。

## 让用户登录您服务器上的控制台

您服务器上的控制台通过您为[终端登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/configure#let-developers-log-in-from-a-terminal)注册的应用程序让用户登录。要完成其设置，请参阅[设置登录](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted/console#set-up-sign-in)。
