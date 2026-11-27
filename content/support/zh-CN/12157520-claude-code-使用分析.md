# Claude Code 使用分析

此功能允许 Console 用户以及 Team 和 Enterprise 计划的所有者监控其组织如何使用 Claude Code，跟踪生产力指标和团队间的采用模式。

Claude Code 使用分析可供以下人员访问：

- **Team 计划：**所有者和主要所有者

- **Enterprise 计划：**所有者、主要所有者和管理员（基于使用量的计划需要 Chat + Claude Code 席位，或基于席位的计划需要 Premium 席位）

- **API Console：**管理员、账单和开发者角色

## 访问 Claude Code 分析

### Team 和 Enterprise 计划

1. 使用所有者或主要所有者账户登录。

2. 点击左下角的您的首字母或名称。

3. 导航到 **[分析 > Claude Code](https://claude.ai/analytics/claude-code)**。

4. 选择 “使用情况”或 “价值”  标签页。

### API Console 用户

1. 登录您的 **[Claude Console 账户](https://platform.claude.com)**。

2. 转到 **Claude Code** 页面 (platform.claude.com/claude-code)。

---

## 使用情况标签页

**使用情况**标签页显示您组织的以下指标。此标签页上的数据每天刷新一次。

### 组织级指标

- **接受的代码行数**：您的团队从 Claude Code 建议中接受的代码总行数。

- **建议接受率**：您的团队接受的 Claude Code 建议的百分比。

- **活动趋势**：活跃用户和会话随时间变化的每日视图。

- **接受的代码行数随时间变化**：接受的代码行数的每日分解。

- **热门命令**：整个组织中最常使用的 Claude Code 命令。

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/1717579277/46c512f4b3ed05c359cecd78ed5c/e0ce2c19-39e2-411f-9a1f-cb1d46439a42?expires=1795793400&amp;signature=4fb0e598716a587571dcd3148501af319a509d7d6ae74372aa53b50784cd7323&amp;req=dScmEcx5lINYXvMW1HO4zfiEPqNXhnrKCX9h5MbdDjPvBnB63X%2Bpsku%2Bh87e%0AX3jsbQCTUevBQ4ds2t8%3D%0A)

### 用户级指标

**个人使用情况**：查看每个团队成员的电子邮件地址和当月接受的代码总行数。您可以搜索特定用户或点击 “导出”按钮生成包含成员电子邮件地址和代码总行数的 CSV 文件。

---

## 价值标签页

**价值**标签页汇总使用情况和成本数据，帮助您一目了然地了解 Claude Code 为您的组织带来的回报。

- **预计生产力提升**

- **每次提交的成本**

- **预计年度价值**

此标签页上使用的每个公式都以内联方式显示。点击任何输入以调整其与您组织的假设相匹配，估计值将重新计算。

---

## 贡献指标（测试版）

贡献指标是公开测试版中的新功能，可帮助 Team 和 Enterprise 组织衡量 Claude Code 对工程速度的影响。通过连接到您组织的 GitHub 账户，这些指标跟踪有和没有 Claude Code 的代码发布活动，让您可以看到它在哪里产生了影响。

贡献指标需要 GitHub Cloud。它们不适用于 Console 用户或为 Claude Code 启用了零数据保留 (ZDR) 的组织。

如需更深入地了解贡献指标，请参阅 **[我们的 Claude Code 文档](https://code.claude.com/docs/en/analytics#enable-contribution-metrics)**。

### 设置贡献指标

贡献指标需要在基础分析设置之外执行几个步骤：

1. 在您组织的 GitHub 账户上安装 **[Claude GitHub App](https://github.com/apps/claude)**。

2. 使用所有者或主要所有者账户登录。

3. 导航到 **[组织设置 > Claude Code](https://claude.ai/admin-settings/claude-code)**。

4. 如果您还没有启用 Claude Code 分析功能，请启用它。

5. 打开 **GitHub 分析**。

6. 选择要包含在比较中的 GitHub 组织。

设置完成后，指标将自动开始填充。请允许最多 24 小时的时间让数据显示。仪表板目前每天处理一次数据。

如果您看到 “需要 GitHub app。安装 GitHub app 以查看分析，”表示 GitHub App 尚未安装。如果应用已通过身份验证但没有数据显示，请确认 GitHub App 已安装且您的团队已开始使用 Claude Code。

### 可用的贡献指标

启用后，以下指标将显示在您的 Claude Code 分析仪表板中：

1. **合并的拉取请求**：在有和没有 Claude Code 协助的情况下合并的拉取请求总数，包括组织级和用户级。

2. **提交的代码行数**：在有和没有 Claude Code 协助的情况下提交的代码总行数，包括组织级和用户级。

3. **每个用户打开的拉取请求**：整个团队的个人拉取请求活动。

数据通过关联 Claude Code 会话活动与 GitHub 提交和拉取请求来收集。

---

## 数据重置和可用性

使用情况指标显示当前日历月的数据，并在每个月初重置。历史数据可视化显示每日粒度，用于跟踪一段时间内的趋势。

## 使用分析优化 Claude Code 采用

查看您组织的代码接受率，以了解团队是否认为 Claude Code 的建议有价值。如果接受率低于预期，请考虑提供有关有效提示技巧的额外培训。

监控个人使用模式，以识别可以与更广泛团队分享最佳实践的高级用户，或发现可能需要额外支持的团队成员。

跟踪活动趋势，以了解您的团队何时最有效地使用 Claude Code，并确保在高峰期间有足够的席位分配。

---

## 常见问题

### 我使用的是个人付费计划；如何访问 Claude Code 的使用分析？

Claude Code 使用分析目前不适用于个人 Pro、Max 或 Ultra 计划。

### 我在寻找特定用户，但他们在报告中缺失。

如果您发现特定用户未显示在您的分析中，应让他们将 Claude Code 更新到最新版本。支持此功能的第一个 Claude Code 版本是 **2.0.28 版本**，因此用户应运行 `claude update` 以在需要时手动更新 Claude Code。

### 我在哪里可以找到更多信息？

请参阅我们 Claude Code 文档中的 **[分析](https://code.claude.com/docs/en/analytics)**以获取更多信息。