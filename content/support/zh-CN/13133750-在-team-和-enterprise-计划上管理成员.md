# 在 Team 和 Enterprise 计划上管理成员

本指南介绍如何在 Team 或 Enterprise 计划上添加、删除和管理成员。

**权限说明：**组织管理员可以在 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**中管理成员，但只有所有者和主要所有者才能访问 **[组织设置 > 账单](https://claude.ai/admin-settings/billing)**。有关更多信息，请参阅我们关于 **[角色和权限](https://support.claude.com/en/articles/9267276-roles-and-permissions)**的文章。

有关购买席位或调整计划席位分配的信息，请参阅我们的 **[Team 计划](https://support.claude.com/en/articles/12004354-purchasing-and-managing-seats)**和 **[Enterprise 计划](https://support.claude.com/en/articles/13393991-purchasing-and-managing-seats-on-enterprise-plans)**指南。

---

## 添加成员

### 通过邀请添加成员

**注意：**待处理的邀请会立即占用您的可用席位；新成员无需接受邀请即可占用席位。

管理员及以上权限的用户可以按照以下步骤添加成员：

1. 导航到 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**，然后点击"添加成员"。

2. 输入该人的电子邮件地址（必须使用您的组织的 **[允许的电子邮件域](https://support.claude.com/en/articles/13325567-account-management-faqs#h_b54c41c86c)**之一）。

3. 选择适当的席位类型。

4. 为成员设置角色和权限。

  1. **注意：**在 Enterprise 计划上，您也可以选择"自定义"作为成员的角色。设置为此角色的成员通过组成员身份和自定义角色来控制其访问权限。要查看自定义角色成员可以访问的内容，请打开其行右侧的"**⋮**"菜单，然后选择

5. 点击"添加成员"。

这会向该人发送电子邮件邀请。邀请在 21 天后过期，因此如果他们在该时间段内未接受，您需要重新邀请他们。

**一次添加多个成员：**您可以点击"批量添加"并输入或粘贴以逗号或换行符分隔的电子邮件地址来邀请多个成员。

**注意：**席位类型选择器仅显示您的计划已拥有的席位类型。如果所选类型的所有席位都已分配，系统会提示您购买一个。有关更多信息，请参阅我们的 **[Team 计划](https://support.claude.com/en/articles/12004354)**和 **[Enterprise 计划](https://support.claude.com/en/articles/13393991)**指南。

### 通过组织发现添加成员

成员也可以通过组织发现自行加入您的组织。启用可发现性后，具有匹配电子邮件域的同事可以在注册期间找到您的组织并请求加入——无需邀请。您可以配置是自动添加他们还是需要您的批准。有关详细信息，请参阅 **[查找并加入 Team 或 Enterprise 组织](https://support.claude.com/en/articles/13566435-organization-discovery)**。

### 分享邀请链接

管理员及以上权限的用户可以生成可共享的邀请链接，并将其分发给队友——例如，通过在 Slack 频道、电子邮件线程或团队 wiki 中发布——无需输入单个电子邮件地址。

**可用性：**

- **Team 计划：**邀请链接对新组织默认启用。

- **Enterprise 计划（非 SSO）：**邀请链接默认禁用。管理员可以在 **[组织设置 > 组织和访问](https://claude.ai/admin-settings/organization)**中启用它们。

- **SSO 组织：**邀请链接不可用。成员配置通过您的身份提供商进行管理。

要查找并复制您的邀请链接，请导航到 **[组织设置 > 组织和访问](https://claude.ai/admin-settings/organization)**。通过链接加入的新成员被分配到最低可用席位层级，如果没有可用的，则默认为标准席位。

管理员可以随时禁用该链接——这会立即使所有现有链接失效。重新生成链接也会使前一个链接失效。

有关加入流程如何工作的更多详细信息，请参阅 **[通过邀请链接加入组织](https://support.claude.com/en/articles/13776697-join-an-organization-via-invite-link)**。

### 使用 SSO 进行自动配置

配置了单点登录 (SSO) 的组织可以自动化成员配置。了解更多关于 **[设置 SSO](https://support.claude.com/en/articles/13132885-setting-up-single-sign-on-sso-for-claude-and-claude-console)** 的信息。

- **即时 (JIT) 配置：**分配给您的身份提供商中 Anthropic 应用的成员将在首次登录时自动创建账户。在具有多种席位类型的计划上，用户在首次登录时被分配到最高可用席位类型。在单席位 Enterprise 计划上，用户自动分配 Enterprise 席位。管理员及以上权限的用户可以之后在 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**中手动重新分配席位类型。

- **SCIM 配置（仅限 Enterprise 计划）：**启用 SCIM 目录同步后，分配给您的身份提供商中 Anthropic 应用的成员会自动配置，最多达到您计划上的总席位数。在具有多种席位类型的计划上，席位类型从最高到最低可用进行分配。在单席位 Enterprise 计划上，所有用户自动分配 Enterprise 席位。管理员及以上权限的用户可以之后在 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**中手动重新分配席位类型。

**重要：**所有者或主要所有者必须确保在配置新用户之前有可用的席位。我们建议监控您的席位使用情况，并主动添加席位以确保您的团队不间断地访问。您可以 **[使用 JIT 或 SCIM 启用组映射](https://support.claude.com/en/articles/13133195-setting-up-jit-or-scim-provisioning#h_adee31eeba)**以直接将用户配置到特定角色和席位层级。

---

## 成员对成员的邀请

组织成员可以通过电子邮件邀请队友，即使他们不是管理员。这使您的团队可以有机地增长，而无需为每个新成员都需要管理员参与。

### 工作原理

任何成员都可以从 Claude 中的账户选择器访问邀请流程。他们输入队友的电子邮件地址并提交邀请。接下来发生的情况取决于您的组织的新成员批准设置：

- **逐个批准（默认）：**邀请请求转到管理员进行审查。邀请仅在管理员批准后才发送给队友。

- **自动批准：**邀请立即发送，被邀请者可以立即加入。

成员发送的邀请遵循与其他加入方法相同的域限制——被邀请者的电子邮件必须与您的组织的允许域之一匹配。

### 可用性

- **Team 计划：**成员对成员的邀请对新组织默认启用。

- **Enterprise 计划：**成员对成员的邀请默认禁用。管理员可以从管理员设置中启用它们。

### 管理员控制

管理员可以从管理员设置中启用或禁用成员对成员的邀请。禁用时，只有管理员可以发送邀请。通过成员邀请添加的被邀请者被分配默认成员角色并放在最低可用席位层级。

---

## 删除成员

您可以通过导航到 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**，点击成员右侧的菜单按钮，然后选择"从团队中删除"来删除成员。但是，如果您是所有者或主要所有者，您无法从您的组织中删除自己。另一个主要所有者或所有者必须将您从团队中删除。

对于使用 SCIM 配置的 Enterprise 组织，当成员从您的身份提供商中删除时，他们会自动从 Claude 中删除。

删除成员时：

- 他们立即失去对组织的访问权限。

- 他们占用的席位变为可用，可分配给另一个用户。

- 如果您稍后使用相同的电子邮件地址重新添加该成员，他们的账户历史将被保留。

删除成员会释放其席位以供重新分配，但不会自动减少您计划的总席位数。有关减少席位的信息，请参阅我们的 **[Team 计划](https://support.claude.com/en/articles/12004354-purchasing-and-managing-seats)**和 **[Enterprise 计划](https://support.claude.com/en/articles/13393991-purchasing-and-managing-seats-on-enterprise-plans)**指南。

**注意：**如果您使用 Team 计划，取消计划后，从 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**中删除成员的选项将不再可用。有关更多信息，请参阅 **[取消您的组织的 Team 计划订阅](https://support.claude.com/en/articles/9267323)**。

---

## 导出成员数据

管理员及以上权限的用户可以从您的组织设置中导出您的组织当前成员列表的 CSV。

导出包括成员详细信息，如姓名、电子邮件地址、角色和席位类型。这对于审计成员身份、协调席位使用或维护团队的外部记录很有用。

要导出：

1. 导航到 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**。

2. 点击 **成员**部分顶部的"导出 CSV"按钮。

3. CSV 文件将下载到您的设备。

---

## 管理邀请

### 重新发送过期的邀请

您可以从 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**重新发送邀请。点击"待处理"选项卡，找到该成员，然后选择重新发送邀请。

### 撤销待处理邀请

您可以从 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**撤销待处理邀请。点击"待处理"选项卡，找到该成员，然后选择"从团队中删除"。

---

## 在任何地方注销成员（测试版）

如果成员丢失了设备或其账户可能被泄露，您可以立即结束其登录会话，而无需将其从您的组织中删除。有两种方式：

- **在组织设置中：**转到 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**，点击成员右侧的三点"..."菜单，然后选择"在任何地方注销"。

- **通过管理员 API（Claude Enterprise，测试版）：**主要所有者创建具有 `write:members` 范围的 API 密钥，然后调用 `POST /v1/organizations/users/{user_id}/sign_out`。访问 Claude 平台文档以了解更多关于 **[使用管理员 API 在任何地方注销成员](https://platform.claude.com/docs/en/manage-claude/user-management#sign-a-member-out-everywhere)**的信息。

这会结束该成员在每个设备上的会话并撤销其受信任的设备。为您的组织授权的 Claude Code 和 Claude Desktop 会话需要在下次使用时重新登录。该成员保留在您的组织中并保持其角色，其聊天和项目不会被删除。

使用前：

- 会话和受信任的设备属于该人的账户，因此这也会将他们从他们使用的任何其他组织或个人计划中注销。

- 计划的 Claude Code 例程继续运行。它也不会断开连接到 Cowork 的计算机。

- 该成员可以立即重新登录。要将其拒之门外，还需从您的组织或身份提供商中删除他们。

- 您无法对自己或主要所有者使用它，只有所有者可以注销其他所有者。

---

## 常见问题

### 我可以邀请已经使用 Claude 个人账户（使用其工作电子邮件）的人吗？

可以。一旦他们加入您的团队，他们将同时拥有个人账户和 Team 或 Enterprise 计划账户。他们可以通过点击左下角的首字母或名字旁边的菜单在账户之间切换。他们可以保留两个账户（除非您的组织已声称其账户）或关闭个人账户。并非所有内容都会移动。在 **[将您的个人 Claude 账户移动到 Team 或 Enterprise 组织](https://support.claude.com/en/articles/9267400-move-your-personal-claude-account-to-a-team-or-enterprise-organization)**中了解更多。

### 我如何添加之前删除的成员？

要添加之前删除的成员，请按照添加新成员的相同步骤进行。他们的账户历史将被保留。

### 我如何更改主要所有者？

当前主要所有者可以通过以下方式转移所有权：

1. 导航到 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**。

2. 点击新用户旁边的角色下拉菜单，然后选择"主要所有者"。

3. 在模态框中输入新主要所有者的电子邮件地址以确认并转移所有权。

**重要：**每个组织只能有一个主要所有者。按照这些步骤会将角色转移给另一个用户。

### 新 Enterprise 组织的初始邀请会发生什么？

当 Anthropic 提供新的 Enterprise 组织并邀请主要所有者时，相同的 21 天过期期限适用于该初始邀请。如果您的邀请已过期，请联系您的账户经理。

### 我如何检查我的席位是标准还是高级？

只有管理员及以上权限的用户可以在 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**中的 **层级**列中查看席位类型。如果您没有管理员、所有者或主要所有者角色，您无法在 Claude 中查看自己的席位类型。要确认您的席位类型，请询问您的组织中的所有者或管理员。