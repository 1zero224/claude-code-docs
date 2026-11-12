# 为 Intune 设置 Claude

Claude for Intune 是适用于 iOS 的 Claude 的托管版本，供使用 Microsoft Intune 的组织使用。本文介绍了 Intune 管理员如何在 Microsoft Intune 管理中心添加 Claude for Intune，以及应告知员工安装哪个应用。

## 什么是 Claude for Intune？

Claude for Intune 专为使用 Microsoft Intune 管理移动设备的组织而构建，包括允许员工从个人 iPhone 和 iPad 工作的组织。

- Claude for Intune 是与标准 Claude 应用不同的应用。两者都在 App Store 中列出。

- IT 团队可以在公司拥有和员工拥有的 iPhone 和 iPad 上对 Claude for Intune 应用 Intune 应用保护策略。

- 它仅使用 Microsoft 登录。

- 它支持 Intune 应用保护策略和 Microsoft Entra 条件访问。

## 要求

在设置 Claude for Intune 之前，请检查您是否满足这些要求：

- 您拥有企业计划。

- 您的组织使用 Microsoft Intune，并且您有权访问 Microsoft Intune 管理中心。

- 员工使用 Microsoft 登录 Claude for Intune。Claude for Intune 中不提供其他登录方法。

- 您使用的是 iOS 或 iPadOS 版本 18.0 或更高版本。

- Anthropic 已为您组织的电子邮件域启用 Microsoft 登录。

- 如果您使用条件访问，Claude for Intune 已在您的 Microsoft Entra 租户中注册。

- 您的 Microsoft Entra 租户中的管理员已为 Claude for Intune 授予管理员同意。这是每个组织都需要的。

## 准备您的组织

### 1. 请求 Anthropic 启用 Microsoft 登录

在员工可以登录 Claude for Intune 之前，Anthropic 需要为您组织的电子邮件域启用 Microsoft 登录。请联系您的 Anthropic 客户团队或支持部门，并告诉他们您的员工用于登录的电子邮件域。

### 2. 在您的 Entra 租户中注册 Claude for Intune

在员工可以登录之前，您的 Microsoft Entra 租户中的管理员需要为 Claude for Intune 授予管理员同意。这是每个组织都需要的。它批准 Claude for Intune 用于与 Microsoft Intune 应用保护配合工作的权限，并在您的租户中注册该应用，以便您可以在条件访问策略中选择它。

**授予管理员同意。**租户管理员打开以下 URL，将 {organization} 替换为您的租户 ID 或域：`https://login.microsoftonline.com/{organization}/adminconsent?client_id=bb747f0e-002b-4882-9960-916fe00a2b90`

要确认其有效，请在 Microsoft Entra 管理中心转到企业应用程序，打开 “Claude for Intune (Public)”，然后选择权限。“Microsoft Mobile Application Management” 应在 “管理员同意” 选项卡上列出。如果没有，请在该页面上选择 “授予管理员同意”。

**注意：**员工使用 “继续使用 Microsoft” 登录 Claude for Intune 不会授予此同意；Entra 管理员必须授予它。

如果员工在登录后几秒钟内被登出，则很可能未完成此步骤。授予管理员同意后，请要求他们删除 Claude for Intune，重新安装它，然后再次登录。

要在登录时要求应用保护，请将条件访问规则定位到 “所有资源”，其中授予 = 需要应用保护策略。仅命名 Office 365 或 Claude SSO 的规则不涵盖 Claude for Intune。设备上必须安装 Microsoft Authenticator。

## 在 Microsoft Intune 管理中心添加 Claude for Intune

### 1. 在员工设备上安装 Claude for Intune

员工获取 Claude for Intune 的方式取决于其 iPhone 或 iPad 的管理方式：

- **未在 Intune 中注册的个人设备（仅限应用保护）：**您无需在 Intune 中添加或分配应用。员工从 App Store 安装 Claude for Intune（在那里显示为 “**[Claude Intune](https://apps.apple.com/us/app/claude-intune/id6812855318)**”）并使用 “继续使用 Microsoft” 登录。您的应用保护策略（第 2 步）在他们登录后应用。

  - 可选：要在公司门户中也列出它，请将其添加为 iOS 商店应用，并将用户组分配为 “可用，无论是否注册。”

- **使用 Apple 用户注册注册的设备：**员工自己从 App Store 安装 Claude for Intune。Intune 无法在这些设备上安装添加为 “iOS 商店应用” 的应用，公司门户显示 “需要管理此设备才能安装应用。”

- **在 Intune 中注册的公司拥有的设备：**将 Claude for Intune 添加为 iOS 商店应用，并使用以下步骤将其分配给用户或设备组。员工从公司门户安装它，或者您可以将分配设为

要为注册的公司拥有的设备添加 Claude for Intune：

1. 在 Microsoft Intune 管理中心，从左侧导航面板选择 “应用”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700192269/ce2bf95a18aba11042da50f4c1ad/dda0f9a2-d585-4b09-ae4f-a1ff70b1e010?expires=1794465000&amp;signature=08e1446735d082b54c0b9578c221f3491b35b572bdfa1d765d8c647c346d62f6&amp;req=dicnFsh3n4NZUPMW1HO4ze7khjXV5G5gAMLV2Uu0R4XprI%2BTVv0%2ByU5lCkUH%0AdEa8%0A)

2. 在平台下，选择 “iOS/iPadOS”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700195036/b70c6c0bdeff5f2b72568ef4943b/67cc834c-5e91-42da-9d3b-1f32c8dd8c05?expires=1794465000&amp;signature=c5d87fb23c6243fb5b47fc78d437abac578f3e3a6b31d1e1c2bf9fb19a329854&amp;req=dicnFsh3mIFcX%2FMW1HO4zeH1oUnjt3aUQ7hR7Wh%2Bi717dLuimGnk13%2BUOFlS%0ANPX9%0A)

3. 单击 “+ 创建。”对于应用类型，选择 “iOS 商店应用，”然后单击底部的 “选择” 按钮：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700195791/9a01290b4189e6a39c473af0a448/cfed06da-f35a-4b22-b687-286ea48864c1?expires=1794465000&amp;signature=17cb7df1173b6bfc0588f8711fb0798ff96102fb46a887fc29f40a7705ebaede&amp;req=dicnFsh3mIZWWPMW1HO4zTni3cItZJss2SRx6amjj9H6CRt8iBiOBnCgTi3n%0AiTx1%0A)

4. 搜索 “Claude for Intune” 并单击底部的 “选择” 按钮。

5. 在 **应用信息** 中，将 **最低操作系统** 设置为 iOS 18，然后单击 “下一步” 按钮：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700196458/588327ab209e9524c45111a244a1/258697c2-2f82-471b-9104-2bccb1b01614?expires=1794465000&amp;signature=a726b2a966876f013ca93c5139e867325bb07ee4351549329c057bb08e86970e&amp;req=dicnFsh3m4VaUfMW1HO4zbxl5YxTcTcuDNNSLAHCCFqZX9LDMuzo503s6mbT%0AvmCg%0A)

6. 对于 **分配**，添加应该获取应用的组：在

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2741107114/8f15b0d179a54c2ca4effaad8577/55bed25a-4b20-47b0-9e89-7f8adc86c04f?expires=1794465000&amp;signature=80432eeb397d1da2587b50a2ec252f0d056bb21bc0f0fa78e020d432b9190d81&amp;req=dicjF8h%2BmoBeXfMW1HO4zYaZMey4f%2FaemN%2B6UXepdg%2FL8pss6R4qwwB5CyVz%0ApXeY%0A)

7. 单击 “下一步，”审查并创建。

8. 如果您将应用分配为

### 2. 对 Claude for Intune 应用应用保护策略

1. 在 Microsoft Intune 管理中心，从左侧导航面板选择 “应用”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700190760/9f6412fe68632cffee29826f7d30/42fdeb30-64af-4089-a976-5094bb6b8292?expires=1794465000&amp;signature=a3850d01f4ac9953e2c0731a8bc3b3cd589b1e329492110ee2b762f9a96c2151&amp;req=dicnFsh3nYZZWfMW1HO4zQwrZI4GVdc8LtTCoSp4D6qm%2FiMzdLBLJ4Ca0qRr%0A7lSm%0A)

2. 在 **管理应用** 下，选择 “保护”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700200858/a7ddd4265aeb6d61a7a096297cd5/fc9a1fe2-3d0e-4038-9b2c-f96bd19e4020?expires=1794465000&amp;signature=c9b14d6994b9f3d94445d6d16d98d9f65623c1ec0e3dc913c61f0bf075933bff&amp;req=dicnFst%2BnYlaUfMW1HO4zeBHXBs58%2FjtnJWY0xu8hzLCHiwlzomyLOHM2%2BX%2B%0AYtd6%0A)

3. 单击 “+ 创建”，然后选择 “iOS/iPadOS”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700201382/6240e257b2b498b5fa5ac1111561/11ef4f59-aba1-4d2b-ac58-9aec1dd60a4d?expires=1794465000&amp;signature=83fe3612894d56651ce0d10ead6397cbc8810e94f550ea9cde302a2c27c7452b&amp;req=dicnFst%2BnIJXW%2FMW1HO4zc89y09SBH9OV2VWh%2B9zzErG0aNxu0sdFv%2Bj7qeT%0APKN7%0A)

4. 在 **基础** 选项卡中输入名称，然后单击 “下一步” 到 **应用**。将 **目标策略设置为** “选定的应用。”单击 “+ 选择自定义应用”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700201929/2edf45cedc0bc03204f86d01a49e/4d96c12d-962b-43f6-8b3f-668e3247d4c1?expires=1794465000&amp;signature=c121ca1dcd0270257ab8c06aa82ffde7a45984382d118ca4adc09f7034ffc2df&amp;req=dicnFst%2BnIhdUPMW1HO4zVmMRJUQVTgIKETwku7gMsu1s4Oh5v42OI6O%2FWWM%0AtKIO%0A)

5. 输入捆绑 ID `com.anthropic.claudeforintune`。选择它，使其在单击 “选择” 之前显示在 **选定的应用** 下：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700202816/3565aea2d1b9c22a5f8d105825c5/3de6d241-6259-463e-a503-a6ca9f2095e0?expires=1794465000&amp;signature=79a436988e589002910f85c63e557a3b0adf35c919f1536f57a7bfc04e88930b&amp;req=dicnFst%2Bn4leX%2FMW1HO4zf6tYicszq9312LT9vQL0W4UT7k9kjP9jHe2bSaY%0ABARJ%0A)

6. 在应用中，确认捆绑 ID 现在显示在 **自定义应用** 下，然后单击 “下一步”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2700203780/24ed354ef0de87bf0a0c1bc129a1/46eed662-f488-48ee-a4d1-8a9a76079884?expires=1794465000&amp;signature=e6336f8f8febf7258bbd86ae791ffa70e1bfcf2ee87ab162a9b321ed2b9fb2ce&amp;req=dicnFst%2BnoZXWfMW1HO4zQd95NGO3c3f8F6Cfy2R7u%2BwXOodJSEzczxtpJo7%0AP0iB%0A)

7. 根据需要配置数据保护、访问要求和条件启动设置。

8. 在 **分配** 选项卡中，将策略分配给包括所有将使用 Claude for Intune 的用户的用户组——应用保护策略只能针对用户组，不能针对设备组。确保它也适用于非托管设备：不要使用设备类型设置或分配筛选器将其限制为 Intune 托管设备，否则个人手机上的员工会在登录后立即被登出。

9. 审查并创建。

10. 应用仅在用户使用组织凭据登录后安装后才变为 “托管”（可能需要重新启动）。

## 告诉员工要安装哪个应用

Claude for Intune 和标准 Claude 应用是单独的应用。您的 Intune 应用保护策略适用于 Claude for Intune。

在您的组织需要 Intune 保护的任何地方，告诉托管或员工拥有的 iPhone 和 iPad 上的员工安装 **[Claude for Intune](https://apps.apple.com/us/app/claude-intune/id6812855318)**，而不是标准 Claude 应用。

## 在 Microsoft 将 Claude for Intune 列为受保护的应用之前

Claude for Intune 尚未出现在 Microsoft 的受保护应用列表中，因此在创建应用保护策略时无法搜索它。在列出之前，通过手动输入其捆绑 ID 将其添加到您的策略中：

1. 按照 **[对 Claude for Intune 应用应用保护策略](#h_5fda0300e8)** 中的步骤操作。

2. 选择应用时，选择 “选择自定义应用” 并输入捆绑 ID `com.anthropic.claudeforintune`。

在 Microsoft 将 Claude for Intune 添加到其受保护应用列表后，您将从公共应用列表中选择它，而不是使用 “选择自定义应用。”