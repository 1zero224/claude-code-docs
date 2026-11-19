# 为您的学校或地区设置 Claude for Teachers

**[Claude for Teachers](https://claude.com/solutions/teachers)** 对经过验证的美国 K-12 教育工作者免费提供至 2028 年 7 月 30 日，无论您是以个人教师身份注册，还是由您的学校或地区为所有人设置组织。

本指南说明授权管理员如何为其教师设置免费 Claude Enterprise 组织及其包含的内容。本指南适用于地区技术负责人和学校 IT 人员。

## 您将获得什么

您的学校员工可获得完整的 Enterprise 产品（网页版、桌面版和移动版 Claude，以及 Claude Code、Cowork 和 Claude Design）、K-12 教学插件、包含州标准的 Learning Commons 连接器、基础学习组件和进度跟踪，以及高质量课程，并享受与个人 Claude for Teachers 账户相同的使用额度，完全免费。管理员可选择启用哪些功能、模型和连接器。

您的组织获得一套 **[K-12 条款和数据处理协议](https://support.claude.com/en/articles/15926041-claude-for-teachers-your-data-and-our-terms)**，为所有人一次性接受，标准 Enterprise 管理员控制（已验证的域、SSO、自动账户创建、角色、使用情况和合规性 API），以及将您域上的现有教师账户纳入组织的方式。

**注意：**Claude for Teachers 面向教育工作者，不面向学生。学生账户不属于此服务范围。

## 谁可以申请

美国 K-12 学校和地区符合条件。申请人必须获得授权为学校或地区设置技术，并且必须使用学校或地区电子邮件地址申请。

**重要提示：**每个电子邮件域一次只能有一个开放的申请，因此请在您的地区内协调，确保由合适的人员申请。这应该是能够代表所有地区员工接受 K-12 条款的管理员。如果您的申请有任何问题，请**[联系支持](https://support.claude.com/en/articles/9015913-how-to-get-support)**。

## 设置您的组织

您需要学校或地区电子邮件、显示您角色的文件（如员工 ID）、访问您域的 DNS 记录的权限，以及访问您的身份提供商的权限。

1. **申请并接受 K-12 条款**。转到 **[claude.ai/k12districts](https://claude.ai/k12districts)**，确认您获得授权，并接受 **[美国 K-12 服务条款](https://www.anthropic.com/legal/k12-terms)**、**[美国 K-12 公共部门附录](https://anthropic.com/legal/k12-addendum)**和 **[美国 K-12 数据处理协议](https://www.anthropic.com/legal/k12-dpa)**。

2. **验证您的学校或地区。**我们的合作伙伴 Goodstack 将确认您的隶属关系。获得批准后，您将收到一封包含新组织链接的电子邮件。

3. **验证您的电子邮件域**。添加您地区的电子邮件域，发布页面上显示的 DNS TXT 记录，然后点击“验证”。席位将开放，免费 Claude for Teachers 计划将自动应用。

4. **设置 SSO 和自动账户创建。**连接您的身份提供商，启用“要求 SSO”，启用即时配置或 SCIM 配置，并启用“限制组织创建”，以便地区电子邮件无法创建个人 Claude 账户。启用自动账户创建后，电子邮件邀请将被禁用。请参阅 **[设置 JIT 或 SCIM 配置](https://support.claude.com/en/articles/13133195-set-up-jit-or-scim-provisioning)**。

5. **引入您的教师。**在开始迁移之前，请确保您有足够的席位来容纳您期望迁移的所有人员。您的组织最初有 500 个席位的限制。要请求更多席位，请转到 **[组织设置 > 成员](https://claude.ai/admin-settings/members)**并点击“请求更多席位。”使用您的域点击“迁移账户”以启动 30 天的时间窗口。您域上的每个现有 Claude 账户（包括个人 Claude for Teachers 账户）都会收到通知，并可选择迁移他们能迁移的内容、重新开始或不采取任何行动。并非所有内容都可以迁移。如果他们不采取任何行动，该账户将在截止日期关闭并被删除，其数据也将根据我们的 **[数据保留政策](https://privacy.claude.com/en/articles/10023548-how-long-do-you-store-my-data)**被删除。请教师在截止日期前从 **[组织设置 > 数据和隐私](https://claude.ai/admin-settings/data-privacy-controls)**导出并下载其数据。如果他们无法导出，应在截止日期前**[联系支持](https://support.claude.com/en/articles/9015913-how-to-get-support#h_d6fc679aa4)**。稍后加入的教师使用 SSO 登录，直接进入您的组织。有关更多信息，请参阅 **[声明和迁移您域上的账户](https://support.claude.com/en/articles/14625619-claim-and-migrate-accounts-on-your-domain)**。教师可以阅读 **[响应您 Claude 账户上的 Enterprise 域声明](https://support.claude.com/en/articles/14625626-respond-to-an-enterprise-domain-claim-on-your-claude-account)**。

## 管理您的组织

您的组织配备标准 Enterprise 管理员控制，全部位于**组织设置**中。您可以管理成员并分配角色来控制每位教师可以使用的功能、模型和连接器；要求地区 SSO 并通过 SCIM 配置成员；验证域并阻止其上的个人账户；选择教师可以访问的连接器（默认情况下不启用任何连接器）；设置默认模型和组织范围的说明；以及跟踪使用情况。

有关完整的管理员文章集，请参阅 **[管理员管理](https://support.claude.com/en/collections/9811449-admin-management)**集合和 **[什么是 Enterprise 计划？](https://support.claude.com/en/articles/9797531-what-is-the-enterprise-plan)**

## 获取帮助

如有设置问题，请**[联系支持](https://support.claude.com/en/articles/9015913-how-to-get-support)**。与 Anthropic 账户团队合作的地区也可以联系其联系人以获取采购文件、安全审查或多组织设置。