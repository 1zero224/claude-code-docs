# 在Claude Desktop上开始使用本地MCP服务器

模型上下文协议(MCP)是一个开放协议,可实现LLM应用与外部数据源和工具的无缝集成。随着桌面扩展的推出,安装和管理本地MCP服务器变得显著更容易。

## 桌面扩展

桌面扩展提供了一种简化的方式来通过单击可安装的软件包安装和管理本地MCP服务器。无需手动配置JSON文件和管理依赖项,您现在可以像安装浏览器扩展一样轻松地在计算机上安装本地MCP服务器。

**注意:**如果您是想在目录中列出桌面扩展(Claude Desktop的本地MCP服务器)的开发者,请通过**[桌面扩展提交表单](https://clau.de/desktop-extention-submission)**提交。要提交插件或远程连接器,请使用**[开发者门户](https://claude.ai/directory/manage)**。请参阅**[提交您的插件](https://claude.com/docs/plugins/submit)**了解每条路线接受的内容。

## 从目录安装桌面扩展

1. 在Claude Desktop上导航至**设置 > 桌面应用**,并在**扩展**旁选择“管理”(或在较旧的设置布局中选择**设置 > 扩展**)。

2. 点击“浏览扩展”查看我们的目录,并点击您想使用的任何Anthropic审核的工具。在您所需的扩展上点击“安装”。

3. 通过用户友好的界面配置任何必需的设置(如API密钥)。

4. 该扩展将自动在您的对话中可用。

## 安装自定义桌面扩展

1. 在Claude Desktop上导航至**设置 > 桌面应用**,并在**扩展**旁选择“管理”(或在较旧的设置布局中选择**设置 > 扩展**)。

2. 点击“高级设置”并找到**扩展开发者**部分。

3. 点击“安装扩展…”

4. 选择.mcpb文件并按照提示进行安装。

## 桌面扩展的管理员控制

团队和企业计划的所有者和主要所有者可以使用两个控制来管理团队对桌面扩展的访问:

1. 根据您组织的安全标准启用或禁用公共桌面扩展。

2. 上传自定义桌面扩展,使其可供您的团队进行一键安装。

这些控制允许组织通过仅添加团队需要访问的扩展并删除其他扩展来完全自定义其注册表。

### 在团队和企业计划上启用/禁用特定扩展

团队和企业组织的所有者和主要所有者可以管理为您的组织启用哪些桌面扩展,以及其他成员可通过Claude Desktop访问哪些扩展。请参阅[启用和使用桌面扩展允许列表](https://support.claude.com/en/articles/12592343-enabling-and-using-the-desktop-extension-allowlist)

了解具体说明。

### 为您的组织创建和上传自定义扩展

自定义桌面扩展上传允许团队和企业计划利用公共目录中不可用的组织特定工作流。创建自定义桌面扩展后,所有者和主要所有者可以导航至**设置 > 桌面应用**,在**扩展**旁选择“管理”(或在较旧的设置布局中选择**设置 > 扩展**),并点击“高级设置”以访问**扩展开发者**部分:

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/1681607607/ba6e379d2769d190f0970a0adaed/AD_4nXd4aZkqjJFpiXMPF28Pih7HmSJ9pPsnoWAfVgiLdFRFiTkO92YtXteIjvDHaPl7T0tjfpRTBOlyrMbQ_aciCNDgfIuEvV3szmKvt72x5O51DMSClXOYWk1JIRIzylwkj3joXqZcLw?expires=1794465900&amp;signature=7a695af47a66686477c98f7b41cd53ec66f30baeaa2dec9fe38ab36e36073cff&amp;req=dSYvF89%2BmodfXvMW1HO4zWbPxUR7MTg7Hn9K2IaIG2LmHpKOZ7ZgG9A5zjWW%0Ar5Qb7ml61xmCgZZ%2B1as%3D%0A)

点击“安装扩展…”并选择.mcpb文件。按照提示安装和配置您的自定义桌面扩展。如需更深入的信息,请参阅我们的[桌面扩展开发者文档](https://github.com/anthropics/mcpb)。

## 企业策略控制

**注意:**用户机器级别的企业策略控制将覆盖任何应用内控制(黑名单和白名单)。如果您想使用应用内控制,请确保`isDesktopExtensionEnabled`和`isDesktopExtensionDirectoryEnabled`未设置为“false”,以便白名单可以填充可用注册表。

有关通过系统策略控制桌面扩展,请参阅我们的[桌面企业配置文档](https://support.claude.com/en/articles/12622667-enterprise-configuration)。

## 桌面扩展安装问题故障排除

### 扩展无法安装

- 确保您运行的是最新版本的Claude Desktop。

- 通过重新下载来检查扩展文件是否损坏。

- 验证您有足够的磁盘空间进行安装。

### 扩展显示已安装但工具不可用

- 重启Claude Desktop以刷新扩展注册表。

- 检查扩展的配置设置是否缺少必需字段。

- 验证任何API密钥或身份验证凭据输入正确。

### 扩展配置问题

- 在Claude Desktop上导航至**设置 > 桌面应用**,在**扩展**旁选择“管理”(或在较旧的设置布局中选择**设置 > 扩展**),然后点击扩展以查看其设置。

- 确保所有必需的配置字段都已完成。

- 检查文件路径(如适用)是否指向您有权访问的现有目录。

### 权限或安全错误

- 在macOS上,如果收到安全警告,请检查系统偏好设置 > 安全与隐私。

- 在Windows和Linux上,确保Claude Desktop具有访问所需目录的必要权限。

- 对于企业环境,请验证桌面扩展是否通过您组织的策略启用。

## 开发者常见问题

### 我如何检查我的MCP服务器是否在Claude Desktop中正确连接?

点击Claude Desktop聊天框底部的“+”按钮,然后选择“连接器。”这将显示您连接的MCP服务器及其工具。或者,转到扩展页面的**本地MCP服务器**部分(**设置 > 桌面应用**,然后在**扩展**旁点击“管理”),或在较旧的设置布局中转到**设置 > 开发者**下的**此计算机**,以查看连接状态并查看任何MCP服务器的日志。

### 我如何将现有MCP服务器转换为桌面扩展?

1. 将manifest.json文件添加到您的MCP服务器目录,其中包含必需的元数据。

2. 使用`mcpb pack`命令打包它。

3. 请参阅[MCPB文档](https://github.com/anthropics/mcpb)了解详细说明。

### 桌面扩展支持哪些编程语言?

桌面扩展支持Node.js、Python和二进制MCP服务器。Claude Desktop包含内置的Node.js环境,因此不需要安装Node.js。

### 我如何处理API密钥等敏感配置?

在manifest.json中将配置字段标记为“sensitive”: true。Claude Desktop将使用操作系统的安全存储(macOS上的Keychain、Windows上的凭据管理器和Linux上的发行版密钥链管理器)自动加密这些值。

### 我可以私下分发我的自定义桌面扩展吗?

是的,您可以直接与特定用户或团队共享.mcpb文件。如需更广泛的分发,您可以向官方目录提交扩展。

### 我如何调试我的桌面扩展的问题?

在Claude Desktop设置中启用调试日志,检查扩展设置面板中的扩展日志,并参考MCP调试指南了解协议特定的故障排除。

### 当我的扩展需要更新时会发生什么?

来自官方目录的扩展默认自动更新。对于私下分发的扩展,用户需要手动安装更新的.mcpb文件。

## 有兴趣了解更多关于MCP的信息?

如需了解有关构建您自己的MCP客户端和服务器的更深入信息,我们建议查看以下资源:

- [MCP快速入门指南](https://modelcontextprotocol.io/docs/getting-started/intro) - 一个完美的分步教程,适合开始基本MCP集成。

- [模型上下文协议Github](https://github.com/modelcontextprotocol) - 包含完整的技术文档、代码示例和实现指南。

- [MCP集成调试指南](https://modelcontextprotocol.io/docs/tools/debugging) - 常见实现挑战的故障排除提示和解决方案。

如果您需要进一步的指导,请访问我们的[如何获得支持](https://support.claude.com/en/articles/9015913-how-to-get-support)指南,了解其他支持选项。