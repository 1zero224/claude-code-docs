---
title: CMEK 在 Claude Platform 上涵盖的内容
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform
description: 查看您的密钥加密哪些 Claude Platform 数据、哪些功能会关闭或发生变化，以及哪些数据仍使用 Anthropic 的密钥加密。
---

有关 Claude Enterprise，请参阅 [CMEK 在 Claude Enterprise 上涵盖的内容](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-enterprise)。

## 使用您的密钥加密

* 消息内容。
* 文件和附件，无论是随请求内联发送，还是通过 Files API 上传。
* MCP 和工具配置。
* [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)（测试版）数据，包括智能体配置、环境、webhook、会话及其事件、[记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory)（测试版）及其记忆和记忆版本，以及[梦境](https://platform.claude.com/docs/zh-CN/managed-agents/dreams)（研究预览）。
* 静态存储的 Batch API 和 Skills API 数据。
* 由网络搜索、网络获取、代码执行、bash、文本编辑器、MCP 连接器（测试版）、结构化输出、顾问工具（测试版）、计算机使用、浏览器使用和上下文管理（测试版）静态存储的工具数据。
* 备份和快照，它们继承该密钥。

## 关闭或发生变化的功能

本页未列出的测试版和研究预览功能可能不在客户管理的加密密钥（CMEK）的保护范围内。

此列表并非详尽无遗。请在设置 CMEK 之前与您的团队一起审阅。

| 功能                                                                                                   | 变化内容                                                                                                                                                     |
| ---------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claude Console 中的 Playground                                                                         | 关闭。                                                                                                                                                      |
| 结构化输出，包括工具上的 `strict: true`，搭配 Claude Fable 或 Claude Mythos 模型                                       | 在 CMEK 组织中关闭。使用其他模型时，结构化输出静态存储的数据[使用您的密钥加密](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-coverage-claude-platform#encrypted-with-your-key)。 |
| [组织 webhook](https://platform.claude.com/docs/zh-CN/manage-claude/organization-webhooks#errors)（测试版） | 对于默认工作区使用密钥的组织关闭。请求返回 400 错误。                                                                                                                            |
| 零数据保留（ZDR）                                                                                           | 使用您的密钥的工作区会保留数据。请参阅 [CMEK、数据保留和 ZDR](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention)。                                          |

## 使用 Anthropic 的密钥加密

存储这些数据的功能仍然可用，但这些数据不使用您的密钥加密。您可以在**设置**中关闭其中部分功能。

| 数据                                                                                                                                       | 详细信息                                                                                                                                                                                                                             |
| ---------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 密钥设置之前的现有数据                                                                                                                              | 在[您的密钥生效](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)之前写入。                                                                                                                |
| 非静态存储的数据，以及 TTL 短于 24 小时的数据                                                                                                              | 例如缓存。                                                                                                                                                                                                                            |
| 活动源、审计日志以及 OpenTelemetry（测试版）等遥测网络流量                                                                                                     | 保留在 Anthropic 的密钥下，以便即使您的密钥无法使用，您也能保持合规。                                                                                                                                                                                         |
| Claude Managed Agents（测试版）[保管库凭据](https://platform.claude.com/docs/zh-CN/managed-agents/vaults)值，例如 OAuth 令牌和客户端密钥                       | 只写，绝不会在 API 响应中返回。                                                                                                                                                                                                               |
| [用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)                                                             | `name` 和 `metadata` 等配置文件字段。请勿在配置文件 `metadata` 中存储敏感个人数据。                                                                                                                                                                        |
| Claude Console 反馈                                                                                                                        | 成员选择通过**反馈**选项发送的评分和评论。没有任何设置可以将其关闭。                                                                                                                                                                                             |
| [OpenTelemetry 目标](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#add-a-destination)（测试版）请求头，例如您的可观测性平台的 API 密钥头 | 只写，绝不会在 API 响应中返回。                                                                                                                                                                                                               |
| 您组织中用户的账户数据                                                                                                                              | 包括姓名、电子邮件地址和头像。                                                                                                                                                                                                                  |
| 保存的记录                                                                                                                                    | Anthropic 可以[在三种有限的情况下](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#limited-preservation-outside-your-key)保存特定记录。                                                                                   |
| Claude Code 遥测                                                                                                                           | 当 Claude Code 通过 LLM 网关或代理（自定义的 `ANTHROPIC_BASE_URL`）连接时，CMEK 不适用于 Claude Code 的运营遥测数据。要关闭此遥测，请将 `DISABLE_TELEMETRY` 环境变量设置为 `1`，如 Claude Code 文档中的[遥测服务](https://code.claude.com/docs/zh-CN/data-usage#telemetry-services)部分所述。 |
