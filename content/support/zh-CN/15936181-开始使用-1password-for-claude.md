# 开始使用 1Password for Claude

1Password for Claude 让 Claude 能够完成需要登录的浏览器任务，使用您存储在 1Password 中的登录凭证。1Password 直接在页面上填充凭证，因此 Claude 永远看不到您的密码或一次性代码。

1Password for Claude 目前处于测试阶段，适用于在 macOS 上使用 Claude Desktop 的付费计划（Pro、Max、Ultra、Team、Enterprise）。它可供个人、家庭和商业计划的 1Password 客户使用，并需要在 Chrome 中使用 Claude。对于 Team 和 Enterprise 计划，1Password for Claude 默认处于关闭状态。

## 工作原理

当 Claude 在执行任务期间到达登录页面时，它会从 1Password 请求凭证，而不是要求您输入。1Password 会准确显示 Claude 请求的项目，您可以在单个提示中批准、更换或拒绝该请求，并通过生物识别确认。

批准后，1Password 直接在页面上填充登录信息。Claude 知道它使用了哪个登录，但密码和任何一次性代码永远不会进入 Claude 的上下文、内存或 Anthropic 的系统。访问权限仅限于当前任务，任务完成时结束。

填充登录后，1Password 会检查页面上是否暴露了任何凭证。如果提交失败，1Password 会在返回控制权前清除填充的值。

## 要求

要使用 1Password for Claude，您需要：

- Mac

- **[Claude Desktop](https://support.claude.com/en/articles/10065433)**

- **[Chrome 中的 Claude](https://support.claude.com/en/articles/12012173)**

- 1Password 账户（个人、家庭或商业计划）

- **[1Password 桌面应用](https://1password.com/downloads/mac)**

- **[1Password 浏览器扩展](https://1password.com/downloads/browser-extension)**

如果您使用的是 Team 或 Enterprise 计划，所有者还必须为您的组织启用此集成。请参阅下面的**[为您的组织启用 1Password for Claude](#h_f7c663d5a2)**。

## 将 1Password 连接到 Claude

满足要求后，您可以从 Claude 中的几个位置设置 1Password：

- 检测到 1Password 时 Claude Desktop 中的横幅

- 任务需要登录时出现的聊天内提示

- 通过在 Claude Desktop 中导航到**设置 > 连接器**（在**自定义**下）并点击 1Password 旁边的"连接"。

## 为您的组织启用 1Password for Claude

对于 Team 和 Enterprise 组织，1Password for Claude 默认处于关闭状态。要启用它：

1. 使用所有者或主所有者账户登录 Claude。

2. 导航到**[组织设置 > Chrome 中的 Claude](https://claude.ai/admin-settings/browser-extension)**。

3. 如果尚未启用，请打开**为您的团队启用**。

4. 打开**密码管理器**：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2546126596/ba71ca47e2df21cec62c243831f8/5b1c67e1-607d-4c73-8f61-d1ceb081082a?expires=1794465900&amp;signature=b0b36a7d93859ad30112dca4dd280fb63204181fffc9a4090347648a2211767b&amp;req=diUjEMh8m4RWX%2FMW1HO4zU5ln29qq8BqGkiu4hEpcPWBQ%2F6fmuCPe%2FLXi%2BqS%0A2g6YKo83yHjAil2uYTI%3D%0A)

启用后，符合条件的用户将看到上述发现选项。用户仍需要自己安装和设置所需的应用和扩展。

## 批准凭证请求

每次 Claude 在任务期间需要登录时：

1. 1Password 显示 Claude 请求的保险库项目。

2. 审查请求。您可以批准、更换为不同的登录或拒绝。

3. 使用生物识别确认您的选择。

1Password 随后直接在页面上填充凭证。您按任务批准凭证使用，因此 Claude 永远无法持续访问您的保险库。

## 代理模式

代理模式是 1Password 浏览器扩展的一项功能，可在 AI 代理控制浏览器时保护您的保险库。当 Claude 接管时，1Password 扩展会自动锁定。扩展的界面被隐藏，Claude 只能使用您为当前任务批准的登录和一次性代码。您保险库的其余部分保持不可访问。

代理模式在扩展级别默认启用，即使您尚未设置 1Password for Claude 也能工作。随时打开 1Password 扩展以确认其处于活动状态或取消当前任务。

## Claude 能看到和看不到什么

Claude 可以看到它用来完成任务的登录。Claude 看不到：

- 您的密码

- 您的一次性代码

- 您的 1Password 保险库的内容

凭证通过 1Password 处理的安全通道注入，在 Claude 的视图之外。它们在 1Password 中保持加密并受您的控制。

## 支持的项目类型

1Password for Claude 支持登录和一次性代码 (TOTP)。其他项目类型（如信用卡和身份）不受支持。

## 保持安全

1Password for Claude 减少了凭证暴露，但基于浏览器的 AI 仍然存在风险。检查每个批准提示，以便在使用前了解 Claude 请求的凭证，并在处理敏感账户前查看**[安全使用 Chrome 中的 Claude](https://support.claude.com/en/articles/12902428)**。