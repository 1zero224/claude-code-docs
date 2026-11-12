---
title: 轮换、禁用或删除您的加密密钥时会发生什么
url: https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle
description: 对您的密钥执行的每项操作会对您的数据产生什么影响、多快生效，以及能否撤消。
---

使用 “customer-managed encryption keys”（客户管理的加密密钥），即 CMEK 时，您的密钥位于您的 “key management service”（密钥管理服务），即 KMS 中，Claude 会保存一条指向该密钥的记录。删除密钥和删除记录是两种不同的操作。

## 每项操作的作用

<Frame>
  ![密钥状态：附加是单向的，已禁用的密钥可以重新启用，密钥删除后 Anthropic 无法恢复数据。](https://platform.claude.com/docs/zh-CN/images/cmek/key-states.svg)
</Frame>

| 操作                                              | 会发生什么                                                                                                                                                                                                                                                    | 能否撤消？                                                                                                                            |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| 将密钥附加到工作区（在 Claude Enterprise 上，将其添加到组织）        | 您的密钥保护[其生效后](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-how-it-works#when-your-key-takes-effect)写入的数据。在 Claude Platform 上，附加操作还会锁定该工作区的[数据保留设置](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-data-retention#data-retention)。 | 否。该密钥无法移除或替换。                                                                                                                    |
| 在同一密钥内轮换密钥材料                                    | 您在自己的 KMS 中按自己的计划轮换。轮换无需在 Claude 中执行任何操作。                                                                                                                                                                                                                | 无需撤消。请保持较早的密钥版本处于启用状态。                                                                                                           |
| 禁用密钥，或移除（撤销）Anthropic 对您的密钥的访问权限                | 该更改最多可能需要 1 小时才能应用，在这一小时内进行中的请求可能仍会成功。此后，Claude 无法读取或写入该密钥保护的数据，请求会失败。                                                                                                                                                                                   | 重新启用密钥，或恢复 Anthropic 对您的密钥的访问权限。[有两项限制](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-key-lifecycle#restore-access)。 |
| 在您的 KMS 中删除密钥、安排删除密钥，或销毁其密钥材料                   | Anthropic 无法恢复该密钥保护的数据。                                                                                                                                                                                                                                  | Anthropic 无法撤消。Anthropic 不保留您的密钥的任何副本。                                                                                           |
| 在 Claude 中删除该密钥的记录（Claude Console 中的**删除加密密钥**） | 仅当没有任何工作区（包括已归档的工作区）使用该密钥时，您才能执行此操作。                                                                                                                                                                                                                     | 否。                                                                                                                               |

## 禁用的作用，以及哪些内容可以恢复

密钥处于禁用状态时，使用该密钥的工作区中的 Messages API 请求会返回[代码为 `cmek_key_disabled` 的 400 错误](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-troubleshooting#cmek-key-disabled)。

重新启用密钥，或恢复 Anthropic 对您的密钥的访问权限，即可恢复对该密钥保护的数据的访问。有两项限制：密钥禁用期间失败的请求不会被重试；禁用密钥不会暂停保留期。

## 如果 Anthropic 无法连通您的密钥

例如，您的 KMS 发生中断，可能会使 Anthropic 无法连通您的密钥。出现这种情况时，在您的密钥下存储数据的请求在最长 1 小时内可能仍会成功。此后，这些请求会失败，直到 Anthropic 能够再次连通您的密钥。在 Claude Enterprise 上，这些请求包括聊天和语音模式中的消息。

在 Messages API 上，以这种方式失败的请求会返回代码为 `cmek_key_unreachable` 的 503 错误，您可以重试这些请求。Message Batches API 会自动为您重试这些请求，直到您的密钥可用或批处理过期。

## 更换密钥或停止使用 CMEK

工作区或组织无法切换到其他密钥，也无法恢复为不使用密钥。在 Claude Platform 上，您可以创建一个使用其他密钥或不使用密钥的新工作区，并将您的流量迁移到该工作区。在 Claude Enterprise 上，无法在组织内更换密钥或停止使用 CMEK。
