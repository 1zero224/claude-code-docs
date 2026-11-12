# SSO/SCIM 电子邮件不匹配 — Okta

**摘要：** Claude 使用电子邮件作为主要标识符来匹配 SSO 登录与已配置的席位。在 Okta 中，SCIM 配置和 SSO 在应用的不同部分进行配置，可能从不同的 Okta 用户配置文件字段中提取电子邮件。本指南说明如何识别和修复不匹配问题。

# 症状

用户在尝试通过 SSO 访问 Claude for Enterprise 时可能会遇到以下一个或多个问题：

- **"账户创建被阻止"** — 用户通过 SSO 进行身份验证，但 Claude 找不到匹配的已配置账户。

- **登陆到免费个人账户** — 如果未限制组织创建，用户会绕过企业组织。

- **"请确认您的电子邮件"不匹配** — SSO 回调显示的电子邮件与用户在登录时输入的电子邮件不同。

- **Claude Code 身份验证失败** — Claude Code CLI 显示电子邮件不匹配错误。

# 这是如何发生的

Okta 用户配置文件包含多个代表身份的字段。SCIM 配置（在 **Provisioning → To App** 下）和 SAML/OIDC SSO（在 **Sign On** 下）是独立配置的：

| Okta 属性         | 典型值                                   | 常用于                            |
| --------------- | ------------------------------------- | ------------------------------ |
| `user.login`    | `testuser1` 或 `testuser1@example.com` | 默认 SCIM userName 映射；有时为 NameID |
| `user.email`    | `test.user.one@example.com`           | SAML/OIDC 电子邮件声明（推荐）           |
| `appuser.email` | 用户电子邮件的应用级覆盖                          | 自定义应用级属性映射                     |

常见的不匹配：SCIM 使用 `user.login` 而 SAML 发送 `user.email`。Claude 需要 **完全字符串匹配**。

**常见混淆：** Okta 的 SCIM 属性映射和 SAML 属性语句位于两个不同的选项卡中 — SCIM 的 **Provisioning → To App** 和 SSO 的 **Sign On**。您必须验证两者。

# 诊断步骤

# 步骤 1 — 确认不匹配

1. **检查 SCIM 电子邮件：** 在 Okta 管理控制台中，转到 **Applications → [Claude App] → Provisioning → To App → Attribute Mappings**。找到 `email`（或 `userName`，如果该字段在 Claude 端映射到电子邮件）的行。**Value** 列显示正在发送的 Okta 表达式或字段。

2. **检查 SSO 电子邮件 (SAML)：** 转到 **Applications → [Claude App] → Sign On → SAML Settings → Edit → Attribute Statements**。找到名为 `email` 的属性。**Value** 列显示在登录时声明的 Okta 字段。

3. **检查 SSO 电子邮件 (OIDC)：** 转到 **Applications → [Claude App] → Sign On → OpenID Connect ID Token** 并检查 **Claims** 部分中的电子邮件声明。

4. 如果 SCIM 和 SSO 值引用不同的 Okta 字段，您已确认不匹配。

# 步骤 2 — 确定问题的范围

确定这是影响一个用户还是系统性问题：

- 如果 **大多数或所有** 已配置的用户共享相同的电子邮件格式不匹配，这是一个 **系统性属性映射问题**，影响您的整个目录。修复方法在您的 IdP 的 SCIM 属性映射中。

- 如果只有 **一个或两个用户** 受影响，而其他用户工作正常，问题可能特定于这些用户账户。直接检查他们的用户配置文件。

# 步骤 3 — 直接验证 Okta 用户配置文件

对于受影响的个别用户，检查其实际字段值：

1. 转到 **Directory → People → [User] → Profile**。

2. 比较 `Login` 和 `Email` 字段值。如果它们不同，任何使用 `Login` 与 `Email` 的映射都会产生不匹配。

# 解决方案

# 将两个映射对齐到同一 Okta 字段

最安全的修复是对 SCIM 和 SSO 都使用 `user.email`，因为此字段包含 Okta 中的规范电子邮件地址。

1. **更新 SCIM 映射：** 在 **Provisioning → To App → Attribute Mappings** 中，将 `email`（或 `userName`）源更改为 `user.email`。

2. **更新 SSO 声明 (SAML)：** 在 **Sign On → SAML Settings → Attribute Statements** 中，将 `email` 属性值更改为 `user.email`。

3. **更新 SSO 声明 (OIDC)：** 在 **Sign On → OpenID Connect ID Token → Claims** 中，更新电子邮件声明以映射到 `user.email`。

4. 点击 **Save**。

# 强制完全重新同步

**关键 — 需要完全同步：** 增量同步 *不会* 在您更改属性映射后更新现有用户。您必须触发配置周期的 **完全重启**。

1. 在 **Provisioning → To App** 中，点击 **Force Sync** 以触发所有已分配用户的完全重新评估。

2. 或者，对于目标用户：**Provisioning → Push Users** → 选择受影响的用户 → **Push Now**。

3. 检查 Okta 系统日志（**Reports → System Log**）中是否有任何配置错误。

4. 在要求用户重试登录之前，验证更新的电子邮件值是否出现在配置日志中。

# 修复后清理

在更正属性映射并完成完全同步后，根据您的情况，您可能需要进行额外的清理：

- **流氓免费账户：** 如果在修复前未限制组织创建，某些用户可能无意中创建了免费个人 Claude 账户。联系 [Anthropic 支持](mailto:support@anthropic.com) 以删除这些账户。

- **幽灵账户（错误电子邮件席位）：** 最初配置的账户（使用不正确的电子邮件）可能仍然存在于您的企业组织中，占用永远无法使用的席位。联系 [Anthropic 支持](mailto:support@anthropic.com) 以取消配置这些幽灵账户。

- **席位可用性：** 如果幽灵账户占用了所有合同席位，即使映射已修复，新登录也会因席位不足错误而失败。如果您遇到这种情况，请联系支持。

- **重新添加受影响的用户：** 删除幽灵账户后，具有更正电子邮件的用户可能需要重新邀请或重新配置。

**防止将来发生：** 在您的企业设置中启用"限制组织创建"。这可以防止尚未配置的用户意外创建免费个人账户。

# 验证

完成修复和任何清理后，验证以下内容：

1. 检查已配置用户的样本 — 确认他们在 IdP 的配置日志中的电子邮件与 SSO 发送的电子邮件格式匹配。

2. 要求受影响的用户清除 `claude.ai` 的浏览器 Cookie，然后通过 SSO 登录。他们应该直接登陆到企业工作区，不会出现任何错误。

3. 确认用户没有意外创建免费账户 — 如果限制了组织创建，被阻止的用户将看到清晰的错误，而不是登陆到个人账户。

4. 如果 Claude Code 受到影响，让用户重新运行 `claude auth login --enterprise` 并确认电子邮件与其企业席位匹配。

# 常见陷阱

| 陷阱                                                    | 解决方案                                                                                 |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------ |
| 管理员更新 SAML 声明但忘记 SCIM 属性映射（或反之）                       | 两者都必须更新。SCIM 映射在 **Provisioning → To App** 下；SAML 声明在 **Sign On → SAML Settings** 下。 |
| `user.login` 对某些用户包含用户名（不是电子邮件）                       | 切换到 `user.email` 并验证所有用户的电子邮件字段已填充。                                                  |
| 增量同步运行但电子邮件未更新                                        | 在属性映射更改后需要强制同步或推送用户。                                                                 |
| 应用级配置文件属性 (`appuser.email`) 与用户配置文件 (`user.email`) 不同 | 检查应用级属性映射是否覆盖用户配置文件值。                                                                |
| 电子邮件已更新但用户仍无法登录                                       | 查找流氓免费组织或幽灵账户。清除浏览器 Cookie 并重试。如果问题仍然存在，请联系 Anthropic 支持。                            |
| OIDC 和 SAML 应用是 Claude 的单独 Okta 应用                    | 某些组织配置两者。确保在两个应用中都检查属性对齐。                                                            |

# 何时联系 Anthropic 支持

- SCIM 和 SSO 属性看起来相同，但用户仍然无法访问其席位。

- 您需要确认 Claude 在 SCIM 配置期间为特定用户记录的电子邮件。

- 您需要帮助清理幽灵账户或流氓免费组织。

- 用户遇到席位不足错误，尽管您的合同有可用席位。

联系 **<support@anthropic.com>**，并提供您组织的域、受影响用户的电子邮件地址以及您的属性映射的屏幕截图。