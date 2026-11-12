# 为 Claude Code 设置代码审查

代码审查分析您的 GitHub 拉取请求，并在发现问题的代码行上发布内联注释。一队专门的代理在完整代码库的背景下检查代码更改，寻找逻辑错误、安全漏洞、损坏的边界情况和回归。

本文介绍如何启用代码审查、配置审查触发器、自定义标记内容以及排查常见设置问题。

**注意：**代码审查仅在团队和企业计划中可用。启用零数据保留的组织无法使用。代码审查使用情况通过使用额度单独计费，不计入您计划的包含使用量。

---

## 代码审查的工作原理

组织启用代码审查后，可以在拉取请求打开时、每次推送时或仅在有人手动请求审查时自动触发。审查运行时，多个代理并行分析差异和周围代码。每个代理查找不同类别的问题，然后验证步骤根据实际代码行为检查结果，以过滤掉误报。

发现结果被去重、按严重程度排序，并作为内联注释发布在发现问题的特定行上。如果未发现问题，Claude 会在 PR 上发布简短的确认注释。审查不会批准或阻止您的 PR，因此现有审查工作流保持不变。

审查成本随 PR 大小和复杂性而变化，平均完成时间为 20 分钟。

### 严重程度级别

每个发现结果都标记有严重程度级别：

| **标记** | **严重程度** | **含义**               |
| ------ | -------- | -------------------- |
| 🔴      | 正常       | 应在合并前修复的错误           |
| 🟡      | 小问题      | 轻微问题，值得修复但不阻止        |
| 🟣      | 已存在      | 代码库中存在但不是由此 PR 引入的错误 |

发现结果包括可折叠的扩展推理部分，您可以展开以查看 Claude 为什么标记该问题以及它如何验证问题。

### 代码审查检查的内容

默认情况下，代码审查专注于正确性：会破坏生产的错误，而不是格式偏好或缺失的测试覆盖。您可以通过向存储库添加指导文件来扩展其检查范围。

---

## 设置代码审查

下面的步骤涵盖 github.com 上的存储库设置。如果您的存储库在自托管 GitHub Enterprise Server (GHES) 实例上，请参阅 **[Claude Code 与 GitHub Enterprise Server](https://code.claude.com/docs/en/github-enterprise-server)** 了解完整设置指南。

团队和企业计划的所有者和主要所有者可以为组织启用一次代码审查，并选择要包含的存储库。除了在 Claude 组织内拥有所有者角色外，您还需要在 GitHub 组织中安装 GitHub 应用的权限。

1. 转到 **[组织设置 > Claude Code](https://claude.ai/admin-settings/claude-code)** 并找到 **代码审查** 部分。

2. 点击"配置"开始 GitHub 应用安装流程。

3. 按照提示将 Claude GitHub 应用安装到您的 GitHub 组织。该应用请求内容、问题和拉取请求的读写权限。

4. 选择要为代码审查启用的存储库。如果您看不到存储库，请确认您在安装期间向 Claude GitHub 应用授予了访问权限。

5. 使用 **审查行为** 下拉菜单为每个存储库设置审查触发器（请参阅下一部分了解每个选项的详细信息）。

要验证设置，请打开测试 PR。如果您选择了自动触发器，名为 **Claude Code Review** 的检查运行应在几分钟内出现。如果您选择了手动，请在 PR 上评论"@claude review"以开始第一次审查。

---

## 选择审查触发器

设置后，**代码审查** 部分在表格中显示您的存储库。对于每个存储库，选择审查运行的时间：

- **PR 创建后一次：**当 PR 打开或标记为准备审查时，审查运行一次。

- **每次推送后：**审查在每次推送到 PR 分支时运行，在 PR 演变时捕获新问题。当您修复之前标记的问题时，Claude 会自动解决线程。这运行最多的审查并花费最多。

- **手动：**仅当有人在 PR 上评论"@claude review"时才开始审查。对于高流量存储库很有用，您可以选择审查哪些 PR。

存储库表还显示每个存储库基于最近活动的平均审查成本。

---

## 手动触发审查

在拉取请求上评论"@claude review"以开始审查，并将该 PR 选择加入后续推送触发的审查。无论存储库的配置触发器如何，这都有效。

要使注释触发审查：

- 将其作为顶级 PR 注释发布，而不是差异行上的内联注释。

- 在注释的开头放置"@claude review"。

- 您必须对存储库具有所有者、成员或协作者访问权限。

- PR 必须打开且不是草稿。

如果审查已在运行，请求将排队等待进行中的审查完成。

---

## 自定义审查

代码审查从您的存储库读取两个文件来指导它标记的内容。两者都是在默认正确性检查之上的附加内容。

### CLAUDE.md

Code Review 读取您的代码库中的 CLAUDE.md 文件，并将新引入的违规视为细微级别的发现。如果您的 PR 以某种方式更改代码，使得 CLAUDE.md 声明过时，Claude 会标记文档也需要更新。

Claude 在目录层次结构的每个级别读取 CLAUDE.md 文件，因此子目录中的 CLAUDE.md 中的规则仅适用于该路径下的文件。

### REVIEW.md

在您的代码库根目录添加 REVIEW.md 文件以获取特定于审查的规则。使用它来编码：

- 公司或团队风格指南

- linter 未涵盖的特定于语言或框架的约定

- Claude 应始终标记的内容（例如，"任何新的 API 路由必须有集成测试"）

- Claude 应跳过的内容（例如，"不要对生成的代码进行评论"）

Claude 自动发现代码库根目录中的 REVIEW.md。无需配置。

---

## 定价和使用

Code Review 根据令牌使用情况计费。每次审查的平均成本为 $15–25，根据 PR 大小、代码库复杂性和需要验证的问题数量而变化。

Code Review 使用情况通过使用额度单独计费，不计入您计划的包含使用情况。您选择的审查触发器会影响总成本：

- **PR 创建后一次**每个 PR 运行一次。

- **每次推送后**在每次推送时运行，成本乘以推送次数。

- **手动**在有人评论"@claude review"之前不产生成本。在该评论之后，对 PR 的其他推送会自动触发审查。

无论您的组织是否为其他 Claude Code 功能使用 AWS Bedrock 或 Google Vertex AI，成本都会显示在您的 Anthropic 账单上。

要设置每月支出上限，请转到 **[组织设置 > 使用情况](https://claude.ai/admin-settings/usage)**并为 Claude Code Review 服务配置限制。

通过分析仪表板中的每周成本图表或管理员设置中的每个代码库平均成本列来监控支出。

### 查看使用情况

转到 **[Code Review 分析仪表板](https://claude.ai/analytics/code-review)**查看您的组织中的活动。仪表板显示：

- **已审查的 PR：**在选定时间范围内审查的拉取请求的每日计数。

- **每周成本：**Code Review 的每周支出。

- **反馈：**因有人解决问题而自动解决的审查评论计数。

- **代码库分解：**每个代码库已审查的 PR 计数和已解决的评论。

---

## 故障排除

### 安装 GitHub App 后代码库未出现

如果您已安装 Claude GitHub App，但您的代码库未出现在管理面板中：

1. 确认 Claude GitHub App 有权访问您期望的代码库。转到您的 GitHub 组织设置，在 **已安装的 GitHub App** 下找到 Claude GitHub App，并检查它是否有权访问所有代码库或仅选定的代码库。

2. 如果您的组织使用带有企业托管用户 (EMU) 的 GitHub Enterprise Cloud，请在使用您的托管账户登录并拥有活跃 SSO 会话时授权 Claude。没有单独的企业级批准步骤。

3. 尝试在 Claude 中断开并重新连接您的 GitHub 账户。转到 **[自定义 > 连接器](https://claude.ai/customize/connectors)**，断开 GitHub，然后重新连接。App 可以访问哪些代码库由 GitHub 端管理，在第 1 步中介绍。

4. 如果问题仍然存在，请 **[联系我们的支持团队](https://support.claude.com/en/articles/9015913-how-to-get-support)**，并提供您的组织名称和 GitHub 组织名称，以便我们进行调查。

### Code Review 不在新 PR 上启动

如果打开 PR 后未出现检查运行：

- 确认代码库已列出并在您的管理员设置中启用。

- 检查审查触发器设置。如果设置为"手动"，您需要在 PR 上评论"@claude review"以启动审查。

- 确保 PR 不是草稿。Code Review 不在草稿 PR 上运行。

- 验证 Claude GitHub App 在您的 GitHub 组织设置中仍然有权访问代码库。

### GitHub Enterprise Cloud 与 IP 限制

如果您的 GitHub Enterprise Cloud 组织按 IP 地址限制访问，Claude GitHub App 可能无法访问您的代码库。要解决此问题，请在您的 GitHub 企业设置中 **[为已安装的 GitHub App 启用 IP 允许列表继承](https://docs.github.com/en/enterprise-cloud@latest/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/managing-allowed-ip-addresses-for-your-organization#allowing-access-by-github-apps)**。Claude GitHub App 注册了其 IP 范围，因此启用此设置允许访问而无需手动配置。要 **[改为手动将范围添加到您的允许列表](https://docs.github.com/en/enterprise-cloud@latest/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/managing-allowed-ip-addresses-for-your-organization#adding-an-allowed-ip-address)**，或配置其他防火墙，请参阅 **[Anthropic API IP 地址](https://platform.claude.com/docs/en/api/ip-addresses)**。

### GitHub Enterprise Server（自托管）

自托管 GitHub Enterprise Server (GHES) 通过单独的设置流程支持。有关完整说明，请参阅 **[Claude Code 与 GitHub Enterprise Server](https://code.claude.com/docs/en/github-enterprise-server)**。

常见 GHES 设置问题：

- **"Public cannot be private"设置期间出错：**您的 GHES 主机名必须解析为可公开路由的 IP 地址。Code Review 通过互联网访问您的服务器，因此内部或专用网络地址不起作用。更新 DNS 或您的网络配置，使主机名解析为公共 IP，然后重试。

- **GitHub Enterprise Cloud 与数据驻留是不同的产品。**GitHub Enterprise Cloud 与数据驻留 (.ghe.com) 也受支持。所有者遵循 GitHub Enterprise Server 管理员设置并输入 `ghe.com` 主机名。

---

## 常见问题

### 创建自定义角色时 Code Review 是否可用？

否，Code Review 目前不可用于添加到 **[自定义角色](https://support.claude.com/en/articles/13930452-manage-custom-roles-on-enterprise-plans)**。

---

## 相关资源

- **[插件](https://code.claude.com/docs/en/discover-plugins)** — 浏览插件市场，包括用于在推送前本地运行按需审查的代码审查插件。

- **[GitHub Actions](https://code.claude.com/docs/en/github-actions)** — 在您自己的 GitHub Actions 工作流中运行 Claude，实现代码审查之外的自定义自动化。

- **[GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd)** — GitLab 管道的自托管 Claude 集成。