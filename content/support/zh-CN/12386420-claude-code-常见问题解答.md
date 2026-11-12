# Claude Code 常见问题解答

本文汇总了关于 Claude Code 的常见问题，涵盖身份验证、集成、配置等多个方面。如果您想了解更多关于 Claude Code 的信息，请参阅我们的 Claude 文档：**[Claude Code 概览](https://docs.claude.com/en/docs/claude-code/overview)**。

## 如何为 Claude Code 设置单点登录 (SSO)？

我们为团队、企业或控制台组织提供了详细的单点登录设置说明，请参阅：**[设置单点登录 (SSO)](https://support.claude.com/en/articles/13132885-set-up-single-sign-on-sso)**。

## 是否有办法在整个组织中禁用 Claude Code 中的 Opus 模型访问权限？

如果您是 Claude 控制台用户，可以通过控制台组织中的速率限制来配置此功能。如果您使用 Bedrock 或 Vertex，请在 Vertex/Bedrock 项目设置中将 Opus 速率限制设置为 0。请注意，即使在 Vertex 中禁用，用户仍可能能够在 Claude Code 中切换模型，因此速率限制是最有效的方法。

## 什么是 Claude Code 动态工作流，我们如何控制访问权限？

动态工作流（适用于 Claude Enterprise 计划）允许 Claude 在单个 Claude Code 会话中从头到尾运行大型工程任务——迁移、审计、代码库范围的错误搜索。它们将在 2026 年 6 月 8 日为整个企业组织默认启用。由于单次运行可能持续数小时并使用比典型会话更多的令牌，管理员应在该日期之前决定谁有权访问。

您有三种方式来控制访问权限：

1. **在角色级别（具有自定义角色的企业计划）：**按组授予或限制 Claude Code 动态工作流功能。这仅影响角色设置为

2. **通过 `managed-settings.json` 禁用组织范围：**将 `"disableWorkflows": true` 添加到您的托管设置。这在 6 月 8 日之前和之后都有效。

3. **组织范围：**6 月 8 日之后，所有者可以通过转到**[组织设置 > Claude Code](https://claude.ai/admin-settings/claude-code)** 并关闭**工作流**来为所有人禁用动态工作流。

我们在每个用户的基础上应用更严格的托管设置和自定义角色权限。如果其中任何一个为给定用户禁用工作流，该用户将无法访问。

## Claude Code 是否支持 Microsoft Visual Studio IDE 集成（不是 VS Code）？

目前不存在 Visual Studio 2022 集成。Claude Code 目前支持 VS Code、Cursor（及其他 VS Code 分支）、Intellij、Pycharm（及其他 Jetbrains IDE）。

## 我们如何使用 Claude Code 实现 PR 审查自动化？

虽然还没有现成的 PR 审查解决方案，但您可以使用**[Claude Code GitHub Actions 集成](https://support.claude.com/en/articles/11932705-automated-security-reviews-in-claude-code)**进行自动审查。目前，您可以使用安全审查操作作为模板并为常规 PR 审查自定义它。这也是**[Claude Code SDK](https://docs.claude.com/en/docs/claude-code/sdk/sdk-overview)** 的一个很好的用例。

## 我收到一条错误消息，说需要 Claude Max、Ultra 或 Pro 才能连接到 Claude Code，但我应该能够通过我的组织的团队或企业计划访问。我该如何排查故障？

这表示您从 Claude Code 设置屏幕中选择了错误的登录方法。尝试再次运行 /login 并选择与您的主要工作电子邮件地址关联的帐户。如果您仍然无法连接，请参阅**[使用您的团队或企业帐户访问 Claude Code 时遇到问题？](https://support.claude.com/en/articles/11845131-using-claude-code-with-your-team-or-enterprise-plan#h_540f9e65d8)**

## 使用 Claude Code 与 Bedrock/Vertex API 密钥时，哪些数据会发送到 Anthropic？

配置 Bedrock/Vertex 并设置 CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC 时，仅发送必要的遥测数据。所有模型 API 请求直接转到您的 Bedrock/Vertex 端点。请查看**[数据流文档](https://docs.claude.com/en/docs/claude-code/security)**了解完整详情。

## 是否有办法通过 Bedrock/Vertex 访问 Claude Code 而不暴露密钥/访问密钥？

可以。我们的**[Bedrock](https://docs.claude.com/en/docs/claude-code/amazon-bedrock)** 和**[Vertex](https://docs.claude.com/en/docs/claude-code/google-vertex-ai)** 设置指南展示了如何启用此功能。例如，在 Bedrock 的情况下，您可以在添加必要的**[配置](https://docs.claude.com/en/docs/claude-code/amazon-bedrock#3-configure-claude-code)**并使用 Bedrock 运行 Claude Code 之前运行 `aws configure` 来配置 AWS CLI，或者您可以使用 Bedrock API 密钥，这是 AWS 的一项新功能，可为 Bedrock 使用启用 API 密钥，而无需完整的 AWS 凭证。

## 我们如何在整个组织中使用自定义环境变量和权限部署 Claude Code？

创建在运行 Claude Code 之前设置环境变量的包装脚本。对于权限，使用带有允许/拒绝列表的 .claude/settings.json 文件。请注意，通配符模式 (*) 并不总是按预期匹配——请彻底测试权限。企业团队通常注入标准化的 Claude.md 文件以实现一致的配置。

## Claude Code 的路线图上是否有公共代码过滤或归属功能？

不，公共代码过滤和归属功能目前不在路线图上。一些客户使用 BlackDuck 进行代码扫描，但关于成本和误报的反馈不一。我们意识到这是扩展 Claude Code 用户的障碍，正在寻找解决方案。

## Claude Code SDK 和 GitHub Actions 中是否提供子代理？

子代理可通过**[Claude Code SDK](https://docs.claude.com/en/docs/claude-code/sdk/sdk-overview)** 获得。它们尚未集成到 GitHub Actions 中，但我们正在考虑这一点。当超过三个子代理并行运行时，UX 会折叠输出以管理复杂性。

## 是否可以将子代理配置为使用特定的 MCP 工具？

可以，创建子代理时，您可以使用配置中的 `tools` 字段指定它有权访问的工具。在子代理配置文件中，您可以省略 tools 字段以继承主线程中的所有工具，或者可以指定单个工具作为逗号分隔列表以获得更精细的控制。在我们的 Claude 文档中了解更多信息：**[子代理 - 可用工具](https://docs.claude.com/en/docs/claude-code/sub-agents#available-tools)**。

## 我们如何管理 Claude Code 成本，特别是对于自动化工作流？

对于安全审查等自动化工作流，使用**[claude --model 配置选项](https://docs.claude.com/en/docs/claude-code/model-config)**从 Opus 切换到 Sonnet 以节省成本。您还可以通过控制台仪表板监控使用情况并设置适当的速率限制。请注意，您可以使用工作区为不同的用户组设置更精细的支出限制。在此处了解有关工作区的更多信息：**[在 Claude 控制台中创建和管理工作区](https://support.claude.com/en/articles/9796807-creating-and-managing-workspaces-in-the-claude-console)**。我们还允许您在控制台中查看每个 API 密钥的支出。有关更多信息，请参阅本文：**[Claude 控制台中的成本和使用情况报告](https://support.claude.com/en/articles/9534590-cost-and-usage-reporting-in-the-claude-console)**。

## Claude Code 是否可以通过 LiteLLM 等企业代理工作？

可以，Claude Code 支持企业代理配置，只要它们支持 Anthropic API 规范。请按照我们 Claude 文档中的代理设置说明进行操作：**[代理配置](https://docs.claude.com/en/docs/claude-code/network-config#proxy-configuration)**。常见问题源于受限环境中的端口限制。

## 使用控制台帐户时，我们如何向 Claude Code 添加用户？

直接将用户添加到您的控制台组织，具有 Claude Code 用户或开发者角色——这就是所需的全部。然后用户从 Claude Code 中运行 /login 并选择预期的控制台帐户。不要尝试在 Claude Code 工作区中手动创建 API 密钥。

## 除了 Claude.md 文件之外，是否有基于团队的内存或知识共享？

目前，Claude.md 文件是主要机制。IT 团队可以将标准化的 Claude.md 文件注入到每台机器的 .claude 目录中以实现组织范围的配置。更高级的团队内存功能正在探索中，但尚不可用。

## Claude Code 中的权限如何工作，为什么我的允许列表没有被遵守？

权限在 .claude/settings.json 或 settings.local.json 中使用模式匹配。通配符语法可能很复杂——"Bash(atlassian-api:*)"应该有效，但可能需要精确的命令匹配。使用"是的，以后不要再问类似的命令"来逐步构建权限。检查全局 (~/.claude/settings.json) 和本地设置文件。

## Claude Code 是否索引我的整个代码库或使用向量数据库来存储有关我的代码库的信息？

不。Claude Code 可以访问系统提示和一系列工具，可以用来按需导航您的代码库。例如，如果 Claude Code 需要了解有关您的代码库的信息，它将使用搜索工具来搜索您的代码库并按需读取文件。我们发现这比完整的代码库索引更有效和灵活：Claude Code *真的*很擅长知道如何筛选代码库以动态收集所需的上下文！

## Claude Code 是否可以与 CI/CD、版本控制和可观测性平台集成？

可以，Claude Code 与 GitHub Actions 集成用于 CI/CD，支持 git 操作，并可以通过 MCP 服务器连接到各种平台。有关更多信息，请参阅我们的 Claude 文档：

- **[Claude Code GitHub Actions](https://docs.claude.com/en/docs/claude-code/github-actions)**

- **[Claude Code GitLab CI/CD](https://docs.claude.com/en/docs/claude-code/gitlab-ci-cd)**

## 为什么我在 GitHub Actions 中看到"工作流验证失败"错误？

这通常发生在可重用工作流中。检查您的工作流语法是否正确以及是否传递了所有必需的参数。如果错误仍然存在，请在此处提交问题，并附上您的工作流配置：**[github.com/anthropics/claude-code-action](https://github.com/anthropics/claude-code-action)**。