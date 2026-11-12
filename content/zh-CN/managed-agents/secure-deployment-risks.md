---
title: 安全风险与沙箱边界
url: https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-risks
description: 为什么智能体的设置对安全至关重要，以及 Managed Agents 沙箱能做什么、不能做什么。
featureMetadata:
  topic:
    title: Managed Agents
    url: https://platform.claude.com/docs/en/managed-agents/overview
  status: beta
  betaHeader: managed-agents-2026-04-01
  zdr:
    eligibility: not-eligible
    note: 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-fable-5-1
    - claude-mythos-5-1
    - claude-fable-5
    - claude-mythos-5
    - claude-mythos-preview
    - claude-opus-5-5
    - claude-opus-5
    - claude-sonnet-5-5
    - claude-sonnet-5
    - claude-haiku-5-5
    - claude-haiku-4-5-20251001
    - claude-opus-4-8
    - claude-opus-4-7
    - claude-opus-4-6
    - claude-sonnet-4-6
    - claude-opus-4-5-20251101
    - claude-sonnet-4-5-20250929
  supportedPlatforms:
    Claude API: beta
    Claude Platform on AWS: beta
    Claude Models on Google Cloud (Anthropic managed): beta
---

本页说明为什么[保护您的 Managed Agents 部署](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment)中的设置很重要，以及沙箱能做什么、不能做什么。

## 为什么这很重要

智能体读取的内容中隐藏的指令可能会操纵智能体。这被称为 “[prompt injection](https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)”（提示注入）。Claude 本身对此具有抵御能力，但模型行为并不是一种安全控制措施。[保护您的 Managed Agents 部署](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment)中的设置可以在攻击尝试成功时限制其影响。

Claude 在持续努力完成您交给它的任务时，也可能采取您未预料到的步骤。例如，它可能会使用它能访问到的工具或凭据来绕过障碍。同样的设置会限制此类步骤能够触及的范围，并且[离线审查者](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-monitoring#optionally-have-an-offline-reviewer-read-every-transcript)可以在事后标记它。

| 可能出现的问题            | 示例                                        | 控制措施                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------ | ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 不受信任的输入操纵智能体       | 一张工单要求智能体取消所有未完成的订单                       | [最小工具集](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#give-the-agent-the-smallest-tool-set)、[让不受信任的输入远离强大的工具](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#keep-untrusted-input-away-from-powerful-tools)、[对有风险的操作进行审批](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#approve-risky-actions) |
| 沙箱中的代码可以使用凭据       | 智能体被诱骗运行的脚本将智能体的密钥发送到外部主机，或使用该密钥读取所有客户的订单 | [将凭据保存在保管库中](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#keep-credentials-in-vaults)、[受限网络](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#limit-the-network)、[对有风险的操作进行审批](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#approve-risky-actions)                                             |
| 开放的网络导致数据外泄        | 一条 bash 命令将文件发送到外部主机                      | [受限网络](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#limit-the-network)、[`web_fetch` 域名列表](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment#restrict-web-search-and-web-fetch)                                                                                                                                             |
| Claude 采取了您未预料到的步骤 | 为了绕过障碍，它使用了其能够访问的工具或凭据                    | [保护您的 Managed Agents 部署](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment)中的设置。[离线审查器](https://platform.claude.com/docs/zh-CN/managed-agents/secure-deployment-monitoring#optionally-have-an-offline-reviewer-read-every-transcript)可以在事后标记该步骤                                                                                                       |

这些设置可以降低风险，但无法消除风险，因此不要将绝不能离开沙箱的数据放入沙箱中。智能体仍然可以将其被允许读取的数据放入自己的回复或工具输出中。请在签发密钥的服务处限制该密钥的权限，因为这些设置不会限制密钥权限。这些设置不会保护您的自定义工具、您的 MCP 服务器或您的自托管基础设施。

## 沙箱能做什么、不能做什么

Managed Agents 在设计上将智能体循环、会话历史和保管库密钥保存在沙箱之外。有关设计背景，请参阅[扩展 Managed Agents](https://www.anthropic.com/engineering/managed-agents)。

| 组成部分                       | 运行位置                                     | 限制因素                                                                                                   |
| -------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| 智能体循环                      | Anthropic 的服务器                           | 您设置的工具集、权限策略和预算                                                                                        |
| `bash` 和文件工具               | 沙箱。在云环境中，每个会话都有自己的隔离 Linux 容器            | 环境网络、工具集和权限策略                                                                                          |
| `web_search` 和 `web_fetch` | Anthropic 的服务器                           | 工具自身的域名列表、您的组织在 Claude Console 中的网络搜索和网页获取设置，以及在 `limited` 云环境中该环境允许的主机。`web_fetch` 还只会打开已在会话中出现过的 URL |
| 保管库密钥                      | 沙箱之外。MCP 凭据永远不会进入沙箱。对于环境变量凭据，代码看到的是一个占位符 | 对于 MCP 凭据，是其所对应的 MCP 服务器 URL。对于环境变量凭据，是该凭据允许的主机、其注入位置以及环境允许的主机                                         |
| 会话历史                       | Anthropic 的服务器，直到您删除该会话                  | 工作区的 API 密钥、Claude Console 中的 Developer 和 Admin 角色，以及会话删除                                              |
| 自定义工具                      | 您的应用程序                                   | 您的代码                                                                                                   |

在云环境中，会话之间不共享文件系统状态。附加到多个会话的记忆存储会在这些会话之间共享。

这些限制不能相互替代。例如，限制环境的主机并不能阻止智能体在您允许的主机上使用凭据。
