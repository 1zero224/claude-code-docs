# 将您的组织从 Team 迁移到 Enterprise

从 Team 计划升级到 Enterprise 计划时，我们建议您保留相同的 Team 组织并按照升级路径将其更改为 Enterprise。这样可以保留您的数据（成员身份/角色、对话和项目）以及 Team 计划组织中的某些设置。如果您创建全新的 Enterprise 组织，则需要从头开始设置所有内容。

升级有两种方式：使用自助流程升级，或者如果您正在与 Anthropic 客户经理合作处理发票或合同协议，请按照销售协助流程进行。步骤和时间安排不同，因此请使用与您的情况相符的部分。

## 保留的内容（相同组织升级）

- 所有聊天历史和对话

- 项目和共享内容

- 用户成员身份和角色

## 迁移前

如果您之前在 Team 计划上设置了支出限制，请考虑在上线日期前增加该限制，以防用户被锁定。一旦达到组织级别的支出限制，所有用户将立即失去访问权限，直到支出限制提高或新月份开始。

## 从 Team 迁移到自助 Enterprise

您可以按照以下步骤从 Team 计划迁移到自助 Enterprise 计划：

1. 导航到 **[claude.ai/upgrade](https://claude.ai/upgrade)** 并点击"获取 Enterprise 计划"。

2. 出现提示时，选择"升级现有组织"并选择您的 Team 计划组织。

3. 添加所有团队成员所需的座位数（Enterprise 组织最少需要 20 个座位）。

4. 设置每个用户的支出限制和整个团队的起始使用余额。

5. 您的付款信息将从之前的 Team 计划付款中保存，但如果需要，您可以点击铅笔图标进行更改。

  1. 从 Team 迁移到自助 Enterprise 计划时，我们仅支持信用卡付款。迁移到自助 Enterprise 计划后，您可以选择使用信用卡、借记卡或银行转账。有关更多信息，请参阅 **[自助与销售协助 Enterprise](https://support.claude.com/en/articles/9797531-what-is-the-enterprise-plan#h_3058c781c5)**。

6. 查看您的订单摘要并点击"确认升级"。

**重要提示：**通过此路径将组织从 Team 迁移到 Enterprise 是不可逆的。在启动此更改之前，请确保 Enterprise 计划适合您的组织。

## 从 Team 迁移到销售协助 Enterprise

您可以通过 Anthropic **[销售团队](https://claude.com/contact-sales?utm_source=support&utm_medium=article&utm_content=enterprise-plan_contact-sales_intro)**从 Team 计划迁移到销售协助 Enterprise 计划。如果您的组织正在迁移到销售协助 Enterprise 计划，请在上线日期前查看以下内容：

- **规划激活时间：**与您的客户经理协商在合同副署前同意上线时间。合同副署后，您的组织将在合同约定的开始日期进行配置。激活可能要到当天晚些时候才能完成，因此请计划在下一个工作日上午进行内部推出，以避免工作日中断。

- **预期在切换时出现短暂中断：**在配置进行中时，您的用户将无法访问 Claude。迁移完成后，所有用户（包括管理员和标准用户）应该登出并重新登录以刷新其会话并清除任何缓存的旧版设置。

## 迁移后

自助或销售协助迁移完成后，我们建议查看以下设置。

### 功能

在 Team 计划上默认启用的某些功能在 Enterprise 计划上可能默认关闭，因此值得检查您的配置是否与您的偏好相符。

即使功能关闭，您的基础数据也会保留。例如，如果您的 Team 计划启用了技能，成员创建了自定义技能，这些技能在迁移期间不会被删除——它们会保留在您的组织中。当管理员在 Enterprise 计划上重新启用技能时，任何之前创建的自定义技能都将再次可用。同样适用于其他默认关闭的功能：重新打开设置会恢复对现有内容的访问，而不是从头开始。

以下功能对 Enterprise 计划默认关闭：

- 技能及其依赖项、技能创建和技能共享（公开和组织内）

- 代码执行和文件创建

- 工件中的交互式内容

- Claude Design

- Claude in Chrome

### 每个用户的支出限制

转到 **[组织设置 > 使用情况](https://claude.ai/admin-settings/usage)**并查看所有成员的每个用户支出限制。清除或调整任何不再适合您的 Enterprise 计划配置的限制。

### 座位分配

在迁移期间，某些用户可能显示为**未分配座位**，而不是自动映射到座位层级。管理员应在切换后验证所有用户是否具有正确的座位分配。注意您使用最频繁的成员，并注意没有座位的用户在管理员纠正其座位分配之前将无法访问。

有关详细指导，请参阅 **[在 Enterprise 计划上购买和管理座位](https://support.claude.com/en/articles/13393991)**。

### 基于角色的访问控制

将组织从 Team 计划迁移到 Enterprise 计划后，所有者或主要所有者可以按照说明配置自定义角色、组和组支出限制：**[在 Enterprise 计划上设置基于角色的权限](https://support.claude.com/en/articles/13930458-set-up-role-based-permissions-on-enterprise-plans)**。

详细了解组、组支出限制和自定义角色：

- **[在 Enterprise 计划上管理组和组支出限制](https://support.claude.com/en/articles/13799932-manage-groups-and-group-spend-limits-on-enterprise-plans)**

- **[在 Enterprise 计划上管理自定义角色](https://support.claude.com/en/articles/13930452-manage-custom-roles-on-enterprise-plans)**

---

## 其他信息

### SSO 和身份设置时间表

- **域验证 (DNS)：**允许 24-48 小时让 DNS 更改全球传播，尽管许多更改在 10 分钟内生效。

- **SCIM 配置同步：**Microsoft Entra ID 大约每 40 分钟推送一次更改。Okta 同步更频繁。

有关详细的设置说明，请参阅 **[设置单点登录 (SSO)](https://support.claude.com/en/articles/13132885)** 和 **[设置 JIT 或 SCIM 配置](https://support.claude.com/en/articles/13133195)**。

**注意：**启用 SSO 后，现有用户将被强制登出并重新登录。

### 计费和使用情况配置

对于基于使用情况的 Enterprise 计划，使用情况根据实际消费进行计费。有关更详细的信息，请参阅 **[我的 Enterprise 计划如何计费？](https://support.claude.com/en/articles/11526368-how-am-i-billed-for-my-enterprise-plan)**

如果您为 Team 计划购买了使用额度，任何未使用的余额将结转并在您的新的基于使用情况的 Enterprise 计划上可用。

### 配置过程

在开始日期，您将被配置并能够在当天结束前使用新功能。初始设置后，所有者和主要所有者可以通过导航到 **[组织设置 > 组织和访问](https://claude.ai/admin-settings/organization)**并点击**总座位数**下的"管理"来自助添加额外座位。

---

## 有用的资源

- **[设置单点登录 (SSO)](https://support.claude.com/en/articles/13132885-set-up-single-sign-on-sso)**

- **[设置 JIT 或 SCIM 配置](https://support.claude.com/en/articles/13133195-set-up-jit-or-scim-provisioning)**

- **[在 Enterprise 计划上购买和管理座位](https://support.claude.com/en/articles/13393991-purchase-and-manage-seats-on-enterprise-plans)**

- **[在 Team 和 Enterprise 计划上管理成员](https://support.claude.com/en/articles/13133750-manage-members-on-team-and-enterprise-plans)**

- **[Enterprise 计费信息](https://support.claude.com/en/articles/11526368-how-am-i-billed-for-my-enterprise-plan)**

- **[在 Team 或 Enterprise 计划中使用 Claude Code](https://support.claude.com/en/articles/11845131-use-claude-code-with-your-team-or-enterprise-plan)**