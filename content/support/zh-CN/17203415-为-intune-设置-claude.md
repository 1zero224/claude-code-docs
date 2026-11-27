# 为 Intune 设置 Claude

Claude for Intune 是适用于 iOS 的 Claude 的托管版本，专为使用 Microsoft Intune 的组织设计。本文介绍了 Intune 管理员如何在 Microsoft Intune 管理中心添加 Claude for Intune，以及应告知员工安装哪个应用。

## 什么是 Claude for Intune？

Claude for Intune 专为使用 Microsoft Intune 管理移动设备的组织而构建，包括允许员工从个人 iPhone 和 iPad 工作的组织。

- Claude for Intune 是独立于标准 Claude 应用的应用。两者都在 App Store 中列出。

- IT 团队可以在公司拥有和员工拥有的 iPhone 和 iPad 上对 Claude for Intune 应用 Intune 应用保护策略。

- 它仅使用 Microsoft 登录。

- 它支持 Intune 应用保护策略和 Microsoft Entra 条件访问。

## 要求

在设置 Claude for Intune 之前，请检查您是否满足以下要求：

- 您拥有企业计划。

- 您的组织使用 Microsoft Intune，且您有权访问 Microsoft Intune 管理中心。

- 员工使用 Microsoft 登录 Claude for Intune。Claude for Intune 中不提供其他登录方法。

- 您使用的是 iOS 或 iPadOS 18.0 或更高版本。

- Anthropic 已为您组织的电子邮件域启用 Microsoft 登录。

- 如果您使用条件访问，Claude for Intune 已在您的 Microsoft Entra 租户中注册。

- 您的 Microsoft Entra 租户中的管理员已为 Claude for Intune 授予管理员同意。这对每个组织都是必需的。

- 您拥有包含 Claude for Intune 的 Intune 应用保护策略，并已分配给所有将使用它的人员。如果没有策略适用于员工，Claude for Intune 会将其注销。

- 将使用 Claude for Intune 的每位员工都拥有 Microsoft Intune 许可证。

## 准备您的组织

### 1. 请求 Anthropic 启用 Microsoft 登录

在员工可以登录 Claude for Intune 之前，Anthropic 需要为您组织的电子邮件域启用 Microsoft 登录。请联系您的 Anthropic 客户团队或支持部门，并告知他们您的员工用于登录的电子邮件域。

### 2. 在您的 Entra 租户中注册 Claude for Intune

在员工可以登录之前，您的 Microsoft Entra 租户中的管理员需要为 Claude for Intune 授予管理员同意。这对每个组织都是必需的。它批准 Claude for Intune 用于与 Microsoft Intune 应用保护配合工作的权限，并在您的租户中注册该应用，以便您可以在条件访问策略中选择它。

**授予管理员同意。**租户管理员打开以下 URL，将 {organization} 替换为您的租户 ID 或域：`https://login.microsoftonline.com/{organization}/adminconsent?client_id=bb747f0e-002b-4882-9960-916fe00a2b90`

选择“接受”后，浏览器可能会显示空白页面。这是正常的。在“权限”页面上确认同意，如下所述。要确认其有效，请在 Microsoft Entra 管理中心转到“企业应用程序”，打开“Claude for Intune (Public)”，然后选择“权限”。“Microsoft Mobile Application Management” 应在“管理员同意”选项卡上列出。如果没有，请在该页面上选择“授予管理员同意”。

**注意：**员工使用“继续使用 Microsoft”登录 Claude for Intune 不会授予此同意；Entra 管理员必须授予它。

如果员工在登录后几秒内被注销，请参阅下面的**[员工登录后立即被注销](#h_d28e61fd3f)**。

要在登录时要求应用保护，请将条件访问规则定位到“所有资源”，其中“授予”= 要求应用保护策略。仅命名 Office 365 或 Claude SSO 的规则不涵盖 Claude for Intune。设备上必须安装 Microsoft Authenticator。

## 在 Microsoft Intune 管理中心添加 Claude for Intune

### 1. 在员工设备上安装 Claude for Intune

在第 2 步中创建应用保护策略，然后再让任何人登录。没有策略的登录员工会立即被注销。

员工如何获取 Claude for Intune 取决于其 iPhone 或 iPad 的管理方式：

- **未在 Intune 中注册的个人设备（仅应用保护）：**您无需在 Intune 中添加或分配应用。员工从 App Store 安装 Claude for Intune（在那里显示为“**[Claude Intune](https://apps.apple.com/us/app/claude-intune/id6812855318)**”），并使用“继续使用 Microsoft”登录。您的应用保护策略（第 2 步）在他们登录后应用。

  - 可选：要在公司门户中也列出它，请将其添加为 iOS 商店应用，并将用户组分配为“可用，无论是否注册。”

- **使用 Apple 用户注册注册的设备：**员工自己从 App Store 安装 Claude for Intune。Intune 无法在这些设备上安装添加为“iOS 商店应用”的应用，公司门户显示“需要管理此设备才能安装应用。”

- **在 Intune 中注册的公司拥有的设备：**将 Claude for Intune 添加为 iOS 商店应用，并使用以下步骤将其分配给用户或设备组。员工从公司门户安装它，或者您可以将分配设为“必需”以将其推送到他们的设备。

要为已注册的公司拥有的设备添加 Claude for Intune：

1. 在 Microsoft Intune 管理中心，从左侧导航面板选择“应用”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700192269/ce2bf95a18aba11042da50f4c1ad/dda0f9a2-d585-4b09-ae4f-a1ff70b1e010?expires=1795793400&amp;signature=4dfba1707f4bfa11ad26b28d7ff8dab679770a9a8ebd6d54abe3194fcf0a0c84&amp;req=dicnFsh3n4NZUPMW1HO4ze7khjTW62hkAMLV2Uu0R4VGpWP48qiNAYAw2oMw%0AWUzC%0A)

2. 在“平台”下，选择“iOS/iPadOS”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700195036/b70c6c0bdeff5f2b72568ef4943b/67cc834c-5e91-42da-9d3b-1f32c8dd8c05?expires=1795793400&amp;signature=02e1fcc06d071522cf0c12c92487c6cc22b847132d2f471a874ceffbc54d6337&amp;req=dicnFsh3mIFcX%2FMW1HO4zeH1oUjguHCQQ7hR7Wh%2Bi731gA1O039VYLG7WVd7%0AGIgU%0A)

3. 单击“+ 创建。”对于应用类型，选择“iOS 商店应用，”然后单击底部的“选择”按钮：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700195791/9a01290b4189e6a39c473af0a448/cfed06da-f35a-4b22-b687-286ea48864c1?expires=1795793400&amp;signature=97a11f5e88f6bc60ec5777516fe9a03d490c54b3af69e853369f9f6f88ce2d80&amp;req=dicnFsh3mIZWWPMW1HO4zTni3cMua50o2SRx6amjj9FZjLhHW9%2BBD0%2FaKWho%0ASLjt%0A)

4. 搜索“Claude Intune”并单击底部的“选择”按钮。

5. 在**应用信息**中，将**最低操作系统**设置为 iOS 18，然后单击“下一步”按钮：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700196458/588327ab209e9524c45111a244a1/258697c2-2f82-471b-9104-2bccb1b01614?expires=1795793400&amp;signature=6861d7fff045f31a62e11ba7777d2198e254ec0174489cf3e0df85244502653d&amp;req=dicnFsh3m4VaUfMW1HO4zbxl5Y1QfjEqDNNSLAHCCFrePbqJ8nFKTfj5DkC2%0AGAzg%0A)

6. 对于**分配**，添加应该获取应用的组：在“必需”下自动安装，或在“可用”下用于已注册的设备，以便员工可以从公司门户安装。您可以使用用户组或设备组。

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2741107114/8f15b0d179a54c2ca4effaad8577/55bed25a-4b20-47b0-9e89-7f8adc86c04f?expires=1795793400&amp;signature=21fb565a46ec7cb437b9af7fb2320638a8861ffdefce18e88c6c81f17df27d06&amp;req=dicjF8h%2BmoBeXfMW1HO4zYaZMe27cPCamN%2B6UXepdg9a2Xbw06X0LKqNWSN9%0Ap04E%0A)

7. 单击“下一步，”审查并创建。

8. 如果您将应用分配为“必需”，它会自动安装；如果为“可用”，员工从公司门户安装。

### 2. 对 Claude for Intune 应用应用保护策略

1. 在 Microsoft Intune 管理中心，从左侧导航面板选择“应用”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700190760/9f6412fe68632cffee29826f7d30/42fdeb30-64af-4089-a976-5094bb6b8292?expires=1795793400&amp;signature=f1de4124e75ed7c500f1e8ca8db345f40bb33c63e0b36a490112b91ebfde05ff&amp;req=dicnFsh3nYZZWfMW1HO4zQwrZI8FWtE4LtTCoSp4D6pS0Cp20v3xI%2F%2BndjO2%0AV%2FVt%0A)

2. 在**管理应用**下，选择“保护”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700200858/a7ddd4265aeb6d61a7a096297cd5/fc9a1fe2-3d0e-4038-9b2c-f96bd19e4020?expires=1795793400&amp;signature=8ae4c0ccdc273464244794525175a5529c64dd909181cc9a4da3c742e2e55c8d&amp;req=dicnFst%2BnYlaUfMW1HO4zeBHXBo6%2FP7pnJWY0xu8hzIYwZ4jueR2zO1ced41%0A6ilW%0A)

3. 单击“+ 创建”，然后选择“iOS/iPadOS”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700201382/6240e257b2b498b5fa5ac1111561/11ef4f59-aba1-4d2b-ac58-9aec1dd60a4d?expires=1795793400&amp;signature=8dfe5acb98dbab92173b2224883d7d99fc8803a69f3f178fca130583b0f8b6b4&amp;req=dicnFst%2BnIJXW%2FMW1HO4zc89y05RC3lKV2VWh%2B9zzEo5eE6Lg1ki%2FKaL6rb3%0ANMfq%0A)

4. 在**基础**选项卡中输入名称，然后单击“下一步”到**应用**。将**目标策略设置为**“选定的应用。”单击“+ 选择自定义应用”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700201929/2edf45cedc0bc03204f86d01a49e/4d96c12d-962b-43f6-8b3f-668e3247d4c1?expires=1795793400&amp;signature=6d6af6b89e94cc53d45fd8efed5d1843c46bda71e7366fc5c7881c7ce2a3b719&amp;req=dicnFst%2BnIhdUPMW1HO4zVmMRJQTWj4MKETwku7gMss8LDxUc2eMPeyA51p6%0AHHAH%0A)

5. 输入捆绑 ID `com.anthropic.claudeforintune`。选择它以使其显示在“选择”之前的**选定应用**下：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700202816/3565aea2d1b9c22a5f8d105825c5/3de6d241-6259-463e-a503-a6ca9f2095e0?expires=1795793400&amp;signature=09d0b3f0107bfda06dab3d1d02cc6f1a6e966ce9cac2739a9e8fe93527e07b8d&amp;req=dicnFst%2Bn4leX%2FMW1HO4zf6tYiYvwalz12LT9vQL0W7%2FX9n8WcLEbX%2B5e7dj%0Amrc7%0A)

6. 在应用中，确认捆绑 ID 现在显示在**自定义应用**下，然后单击“下一步”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700203780/24ed354ef0de87bf0a0c1bc129a1/46eed662-f488-48ee-a4d1-8a9a76079884?expires=1795793400&amp;signature=674a745f965ab65ada36e27fb97b1de1d0ad32fab4291d4ba49979f4ce70effb&amp;req=dicnFst%2BnoZXWfMW1HO4zQd95NCN0svb8F6Cfy2R7u9oArkxF5EeOV%2FVVhX0%0ApuTm%0A)

7. 根据需要配置数据保护、访问要求和条件启动设置。

8. 在**分配**选项卡中，将策略分配给包含所有将使用 Claude for Intune 的人员的用户组——应用保护策略只能针对用户组，不能针对设备组。确保它也适用于非托管设备：不要使用设备类型设置或分配筛选器将其限制为仅 Intune 托管设备，即使您的所有手机都是公司拥有的。从 App Store 安装的 Claude for Intune 副本在已注册的手机上可能被视为非托管，因此策略会跳过它，员工会在登录后立即被注销。还要检查没有排除的组会遗漏任何将使用该应用的人*。*

9. 审查并创建。

10. 应用仅在用户使用组织凭据登录后安装后才变为“托管”（可能需要重新启动）。

11. 等待约 30 分钟以使策略到达您的员工，然后让一名员工登录，然后再告知其他所有人。

12. 在 Microsoft Intune 管理中心，转到“应用，”然后“监视，”然后“应用保护状态。”Claude for Intune 应为该员工列出，并应用了您的策略。

## 告知员工要安装哪个应用

Claude for Intune 和标准 Claude 应用是独立的应用。您的 Intune 应用保护策略适用于 Claude for Intune。

在您的组织需要 Intune 保护的任何地方，告知托管或员工拥有的 iPhone 和 iPad 上的员工安装**[Claude for Intune](https://apps.apple.com/us/app/claude-intune/id6812855318)**，而不是标准 Claude 应用。

## 在 Microsoft 将 Claude for Intune 列为受保护应用之前

Claude for Intune 尚未出现在 Microsoft 的受保护应用列表中，因此在创建应用保护策略时无法搜索它。在列出之前，通过手动输入其捆绑 ID 将其添加到您的策略：

1. 按照**[对 Claude for Intune 应用应用保护策略](#h_4ca21beece)**中的步骤操作。

2. 选择应用时，选择“选择自定义应用”并输入捆绑 ID `com.anthropic.claudeforintune`。

在 Microsoft 将 Claude for Intune 添加到其受保护应用列表后，您将从公共应用列表中选择它，而不是使用“选择自定义应用。”

## 员工登录后立即被注销

登录似乎有效，然后 Claude for Intune 返回到登录屏幕，显示消息“无法使用 Microsoft 登录。出了问题。”

如果只发生一次，让员工在稳定的连接上重试。如果每次都发生，修复方法在您的 Intune 或 Entra 设置中。检查下面的两个原因，因为修复一个可能会揭示另一个。

通常，没有应用保护策略适用于该员工。检查：

1. iOS/iPadOS 应用保护策略包括自定义应用 `com.anthropic.claudeforintune`。

2. 该策略已分配给包含该员工的用户组。

3. 没有排除的组或分配筛选器会遗漏该员工。

4. 该策略适用于所有设备类型，而不仅仅是 Intune 托管设备。

5. 该员工拥有 Microsoft Intune 许可证。

6. 自上次更改以来已过去约 30 分钟。

较少情况下，缺少管理员同意。按照**[在您的 Entra 租户中注册 Claude for Intune](#h_4889891bcc)**中的步骤确认。授予管理员同意后，要求员工删除 Claude for Intune，重新安装它，然后再次登录。

要区分两者，请打开 Microsoft Entra 管理中心并转到“登录日志，”然后“用户登录（非交互式）。”按应用程序“Claude for Intune (Public)”筛选。资源“Microsoft Mobile Application Management”的失败条目指向管理员同意或条件访问策略。如果每个条目都成功，请检查应用保护策略。

如果仅在您的公司网络或 VPN 上发生，请检查网络是否允许 Microsoft Intune 的端点，并且不检查其 TLS 流量。

如果员工仍然被注销，请联系 Anthropic 支持，提供员工的电子邮件地址、他们尝试登录的时间（带时区）和应用版本。我们可以查找原因。支持可能会通过其 Intune 错误代码来引用它：225（无策略）、203（同意）或 200（无许可证）。