# 移动 Claude Code CLI 用户

如果您的开发人员使用 Claude Code CLI，请将其与桌面应用分开设置。桌面应用从其 Claude Desktop on 3P 配置获取提供商设置，**[而不是从 `ANTHROPIC_BASE_URL` 或 `settings.json`](https://code.claude.com/docs/en/llm-gateway-connect)**，并且**[将其传递给它启动的 Code 会话](https://claude.com/docs/third-party/claude-desktop/code)**。CLI 读取环境变量、设置文件和**[Claude Code 托管设置](https://code.claude.com/docs/en/managed-settings)**，这些设置使用**[单独的策略密钥](https://code.claude.com/docs/en/admin-setup)**。

本指南适用于 Claude Enterprise 计划组织的管理员。

本文是**[从 Claude Enterprise 迁移到云提供商上的 Claude Desktop](https://support.claude.com/en/articles/18426225)**的一部分，该指南定义了波次、重叠期和截止日期等术语。

**[CLI 和所有本地运行的内容都可在每个提供商上运行](https://code.claude.com/docs/en/feature-availability)**。迁移后，开发人员以不同方式登录，您的策略以不同方式到达他们的计算机，某些功能不可用。

## 将 CLI 连接到您的提供商

为**[Amazon Bedrock](https://code.claude.com/docs/en/amazon-bedrock)**、**[Google Cloud](https://code.claude.com/docs/en/google-vertex-ai)**、**[Microsoft Foundry](https://code.claude.com/docs/en/microsoft-foundry)** 或**[LLM 网关](https://code.claude.com/docs/en/llm-gateway-connect)**设置提供商变量。开发人员**[不需要 claude.ai 浏览器登录](https://code.claude.com/docs/en/authentication)**。使用网关时，设置基础 URL 和网关凭证。仅使用基础 URL 时，**[保存的 claude.ai 登录保持为活跃凭证](https://code.claude.com/docs/en/llm-gateway)**，因此其限制和计费仍然适用。在团队推出前，**[固定您的模型版本](https://code.claude.com/docs/en/third-party-integrations)**。

## 向每台设备传递策略

**[设置了第三方提供商或自定义基础 URL 时，服务器托管设置不适用](https://code.claude.com/docs/en/server-managed-settings)**。将这些策略移至**[设备上的托管设置](https://code.claude.com/docs/en/managed-settings)**。您可以将其作为 `managed-settings.json` 文件、`com.anthropic.claudecode` macOS 配置文件或 `HKLM\SOFTWARE\Policies\ClaudeCode` 注册表项传递。它们优先于用户和项目设置，除了少数安全敏感的密钥，其中来自较低级别的更严格值仍然适用。

- **限制计算机可以使用的提供商：**设置**[`allowedProviders`](https://code.claude.com/docs/en/third-party-integrations)**（Claude Code v2.1.285 或更高版本）。在重叠期间，**[列表如 `["anthropic", "bedrock"]`](https://code.claude.com/docs/en/authentication)** 让开发人员使用其中任一个。迁移后，仅列出新提供商。在用户在 Claude Desktop 中保持登录到 claude.ai 的计算机上保留 “anthropic”，因为他们的 Cowork 会话没有它会被拒绝。对于通过 `ANTHROPIC_BASE_URL` 访问的网关，列出 `"customEndpoint"` 并将网关 URL 放在同一托管源的 `env` 块中。

- **删除旧登录设置：**使用网关凭证，**[删除 `forceLoginMethod` 和 `forceLoginOrgUUID`](https://code.claude.com/docs/en/llm-gateway-rollout)**，因为任一密钥都会在启动时阻止网关凭证。在云提供商上，这些密钥**[仅在某些凭证仍在计算机上时阻止会话](https://code.claude.com/docs/en/authentication)**：`ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`apiKeyHelper` 凭证或由早期 Claude Console 登录保存的 API 密钥。

开发人员可以运行 `/status` 查看活跃提供商及其设置来源。有关遥测、网络主机和更新，请参阅**[数据使用](https://code.claude.com/docs/en/data-usage)**、**[企业网络配置](https://code.claude.com/docs/en/network-config)**和**[高级设置](https://code.claude.com/docs/en/setup)**。

## 跟踪成本和使用情况

在 Bedrock、Google Cloud 和 Foundry 上，使用情况**[按令牌计费到您的云账户](https://code.claude.com/docs/en/costs)**，成本控制在您的云计费控制台中。通过网关，使用情况**[按令牌计费给拥有网关转发的凭证的人](https://code.claude.com/docs/en/llm-gateway)**。claude.ai 分析仪表板和 Claude Code Analytics API 不涵盖此使用情况。对于按用户报告，请使用**[OpenTelemetry](https://code.claude.com/docs/en/monitoring-usage)** 或 LLM 网关。

## 开发人员保留的内容

| **项目**                                       | **迁移后**                                                                                                                                                                                                                                                                                    | **要做的事**                                                                                                                                                                                                                                          |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 设置、`CLAUDE.md`、自动记忆、技能、子代理、命令、钩子、插件和 MCP 服务器 | 它们保留。它们位于**[`~/.claude`、`~/.claude.json` 和项目文件夹](https://code.claude.com/docs/en/claude-directory)**中。                                                                                                                                                                                     | 无需操作，除非您登出（见下文）。在任何清理期间保留这些文件夹。                                                                                                                                                                                                                   |
| 本地会话记录                                       | 它们保留在 `~/.claude/projects/` 中，直到**[`cleanupPeriodDays`](https://code.claude.com/docs/en/claude-directory)** 删除它们。默认值为 30 天。**[`claude --continue` 和 `claude --resume`](https://code.claude.com/docs/en/sessions)** 重新打开它们。                                                                 | 在您的试点中测试恢复会话。在 Bedrock、Google Cloud 和 Foundry 上，**[会话的早期模型不会被恢复](https://code.claude.com/docs/en/model-config)**。如果网关的上游拒绝早期思考签名，Claude Code **[会删除早期思考块并重试](https://code.claude.com/docs/en/llm-gateway-protocol)**。                             |
| 云会话                                          | Anthropic 在 Claude 账户下存储它们。**[当 Claude Code 配置为第三方提供商时，`--cloud` 不可用](https://code.claude.com/docs/en/claude-code-on-the-web)**，`--resume` 不列出云会话。                                                                                                                                         | 在迁移前，使用 `--teleport` 将您需要的每个会话拉入终端。它会创建本地副本。它需要 claude.ai 登录、干净的工作树、同一存储库的检出和推送到远程的会话分支。                                                                                                                                                          |
| 来自 claude.ai 的连接器                            | **[当第三方提供商处于活跃状态时不加载](https://code.claude.com/docs/en/mcp)**。                                                                                                                                                                                                                              | 将它们作为 MCP 服务器添加到用户或项目范围，或通过托管 MCP 配置部署它们。某些 Anthropic 托管的连接器（如 Microsoft 365、Gmail 和 Google Calendar）**[不支持从您自己添加的服务器登录](https://code.claude.com/docs/en/mcp)**，因此请为这些计划不同的服务器。                                                                   |
| 从 claude.ai 同步的技能                            | **[在 Bedrock、Google Cloud 或 Foundry 上的会话中，或当网关凭证替换 claude.ai 登录时，技能同步不运行](https://code.claude.com/docs/en/skills)**。早期同步的技能保留在 `~/.claude/skills/synced/` 中的磁盘上。                                                                                                                           | 将您依赖的任何技能从 `~/.claude/skills/synced/` 复制到 `~/.claude/skills/`。                                                                                                                                                                                    |
| 从 claude.ai 同步的插件                            | **[在 Bedrock、Google Cloud 或 Foundry 上的会话中既不下载也不加载](https://code.claude.com/docs/en/plugins/loading)**，因为这些会话**[不从 Anthropic 获取功能标志](https://code.claude.com/docs/en/env-vars)**。当 `ANTHROPIC_AUTH_TOKEN` 或 `apiKeyHelper` 提供凭证时也是如此。                                                       | 如果市场提供它们，请从市场重新安装。                                                                                                                                                                                                                                |
| 需要 claude.ai 登录的功能                           | **[无法从提供商访问](https://code.claude.com/docs/en/feature-availability)**。这些包括云会话、例程、托管代码审查、Ultrareview、远程控制、Chrome 扩展和语音听写。在 Bedrock、Google Cloud 和 Foundry 上，频道和分析仪表板也不可用。Advisor 和 Code Mode 也是如此。CLI 的网络搜索在 Bedrock 上不可用。快速模式在 Bedrock 和 Foundry 上不可用，在 Google Cloud 上仅限于 Claude Fable 5.2。 | 使用 `/loop` 进行调度，使用 GitHub Actions 或 GitLab CI/CD 进行云运行（仅在 Foundry 上使用 GitHub Actions），以及**[本地 `/code-review`](https://code.claude.com/docs/en/code-review)**。**[决定开发人员是否仍需要 Team 或 Enterprise 席位](https://code.claude.com/docs/en/admin-setup)**。 |

## 在重叠期间要注意的事项

- **在仍使用 claude.ai 进行身份验证的会话中，服务器托管设置优先。**默认情况下，Claude Code **[使用它们并跳过设备上的托管设置](https://code.claude.com/docs/en/server-managed-settings)**，除了 `env` 块和 `allowedProviders` 等少数例外。如果某些开发人员保留在 claude.ai 上，**[为他们使用服务器托管设置，为其他人使用文件或操作系统策略](https://code.claude.com/docs/en/admin-setup)**。

- **保存的 claude.ai 登录保留在计算机上。**云提供商凭证**[优先于它](https://code.claude.com/docs/en/authentication)**，`/logout` **[在云提供商处于活跃状态后不可用](https://code.claude.com/docs/en/amazon-bedrock)**。如果您想删除登录，请在部署提供商设置前让开发人员登出。登出也**[删除保存的 MCP 服务器登录和插件密钥](https://code.claude.com/docs/en/troubleshoot-install)**，因此他们需要再次授权这些。使用网关凭证，`/logout` 保持可用，您可以**[运行它来清除保存的登录](https://code.claude.com/docs/en/llm-gateway-connect)**。

- **CLI 策略可以覆盖您的部分桌面策略。**默认情况下，如果计算机有 Claude Code 托管设置，桌面 Code 会话**[忽略桌面应用提供的托管策略值](https://claude.com/docs/third-party/claude-desktop/code)**。要同时使用两者，请参阅**[部署托管设置](https://code.claude.com/docs/en/managed-settings)**中的 `parentSettingsBehavior`。

## 开发人员检查清单

**迁移前**

- 将 Claude Code 更新到您的管理员指定的版本。

- 使用 `claude --teleport` 将您需要的任何云会话拉入终端。首先确保会话的分支已推送到远程，并清理您的工作树。

- 列出您使用的 claude.ai 连接器，以便将其添加为 MCP 服务器。

- 复制您依赖的任何技能（来自 `~/.claude/skills/synced/`），并列出您需要重新安装的同步插件。

**在您迁移的那天**

1. 按照您的管理员描述的方式登录到您的云提供商，例如使用 `aws sso login`、`gcloud auth application-default login` 或 `az login`。

2. 启动 `claude` 并运行 `/status`。检查提供商、模型和设置来源。

3. 使用 `claude --resume` 重新打开最近的会话并确认它响应。

4. 告诉您的管理员任何缺失的工具或连接器。