---
title: 获取您的 Claude API 密钥
url: https://platform.claude.com/docs/zh-CN/get-api-key
description: 在 Claude Console 中查找、创建和管理用于 Claude API 的 API 密钥。
---

Claude API 的 API 密钥（也称为 Anthropic API 密钥）位于 Claude Console 中。要查看您现有的密钥或创建新密钥，请前往 Console 中的 [API 密钥](https://platform.claude.com/settings/keys)页面（`platform.claude.com/settings/keys`）。

## 选择密钥类型

创建密钥时，您需要选择其类型，类型决定了密钥可以执行的操作、可以在何处使用以及何时失效。**个人密钥**（personal key）代表您本人执行操作，如果您离开某个组织，该密钥将失效。**服务账户密钥**（service account key）代表一个[服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#service-accounts)，可供 CI 流水线、生产服务或智能体等工作负载使用。请将个人密钥用于您自己的开发工作，将服务账户密钥用于任何共享场景。

您还可以创建**工作区密钥**（workspace key），这是一种没有所有者的旧版密钥：它属于您创建它时所在的工作区，并在其创建者离开后继续有效。建议优先使用个人密钥或服务账户密钥，因为当其关联账户从组织中移除时，这些密钥会自动停止工作。

## 创建 API 密钥

<Steps>
  <Step title="登录 Claude Console">
    前往 [platform.claude.com](https://platform.claude.com/) 上的 Claude Console 并登录，如果您还没有账号，请创建一个。
  </Step>

  <Step title="打开 API 密钥页面">
    在 Claude Console 中，前往 [API 密钥](https://platform.claude.com/settings/keys)页面（`platform.claude.com/settings/keys`）。
  </Step>

  <Step title="创建密钥">
    点击**创建密钥**，为密钥命名，选择[过期时间](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-expiration)，并将**已关联账户**设置为您自己或某个服务账号。您还可以在 Console 中选择一个[工作区](https://platform.claude.com/settings/workspaces)，将密钥的作用范围限定于该工作区。
  </Step>

  <Step title="复制并保存密钥">
    Console 仅在创建时显示一次完整密钥（以 `sk-ant-` 开头）。请复制它并将其保存在安全的地方，例如密钥管理器中。如果您丢失了密钥，将无法在 Console 中再次查看它。请改为创建一个新密钥。
  </Step>
</Steps>

如果 API 密钥页面上的**创建密钥**按钮被禁用，则您的角色可能不允许您在此处创建密钥。请联系组织管理员更改您的角色，或为您的工作负载创建服务账号密钥。

## 使用您的 API 密钥

将密钥设置为环境变量：

```bash
export ANTHROPIC_API_KEY="sk-ant-api03-..."
```

[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 会自动读取 `ANTHROPIC_API_KEY`。直接发送的 HTTP 请求以 `Authorization: Bearer <key>` 的形式发送密钥。如果您的 API 密钥未限定到某个工作区，您还必须在每个 Claude API 请求中发送 `anthropic-workspace-id` 标头，如[选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)中所示。有关 Admin API，请参阅 [API 密钥与 Admin API](https://platform.claude.com/docs/zh-CN/get-api-key#api-keys-and-the-admin-api)。

要发出您的第一个请求，请按照[快速入门](https://platform.claude.com/docs/zh-CN/get-started)操作，并参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)了解完整详情，包括使用工作负载身份联合的短期凭据。

## API 密钥与 Admin API

[Admin API](https://platform.claude.com/docs/zh-CN/api/organization) 包含用于以编程方式管理组织 API 密钥的端点，例如[检索 API 密钥](https://platform.claude.com/docs/zh-CN/api/organization/api_keys/retrieve)和[列出 API 密钥](https://platform.claude.com/docs/zh-CN/api/organization/api_keys/list)。这些端点供组织管理员自动化密钥管理使用。它们接受 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)、具有 `org:admin` 作用域的 OAuth 令牌，或未限定到特定工作区的个人密钥或服务账户密钥；工作区密钥在此处不可用。它们从不返回密钥的秘密值，只返回部分隐去的提示。

<Note>
  Admin API 无法恢复丢失的密钥，也无法为您提供用于调用 Claude API 的密钥。要获取可用的 API 密钥，请在 Claude Console 的 [API 密钥](https://platform.claude.com/settings/keys)页面（`platform.claude.com/settings/keys`）上创建一个。
</Note>
