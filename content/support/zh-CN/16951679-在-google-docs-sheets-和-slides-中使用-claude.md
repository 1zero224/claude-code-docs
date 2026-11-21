# 在 Google Docs、Sheets 和 Slides 中使用 Claude

Claude for Google Workspace 在 Google Docs、Google Sheets 和 Google Slides 中添加了 Claude 侧边栏。Claude 会读取您打开的文件并直接编辑它，因此您可以进行草稿编写、分析、重组和重新设计，而无需来回复制内容。

Claude for Google Workspace 目前以测试版形式提供给 Pro、Max、Ultra、Team 和 Enterprise 计划用户。您还需要一个被允许安装 Google Workspace Marketplace 应用的 Google 账户。

使用 Claude for Google Workspace，您可以：

- 提出关于您打开的文档、电子表格或演示文稿的问题，并获得引用来源的答案。

- 让 Claude 为您进行编辑，所有更改都在版本历史中可见。

- 在 Sheets 中构建电子表格模型、编写和修复公式以及清理数据。

- 在 Docs 中草稿编写、重写和重组长文档。

- 在 Slides 中创建新幻灯片或改进和重新设计现有幻灯片。

- 使用您的组织已在 Claude 中设置的连接器，从侧边栏内访问。

这与 Claude 中的 **[Google Workspace 连接器](https://support.claude.com/en/articles/10166901)**不同，后者允许 Claude 在您在 claude.ai 聊天时搜索和引用您的 Drive、Gmail 和 Calendar。该扩展的工作方式相反：Claude 来到您已经打开的文件。

## 快速开始

如果您的组织已允许 Marketplace 应用，这大约需要两分钟。

1. 从 **[Google Workspace Marketplace 列表](https://workspace.google.com/marketplace/app/claude/12459801340)**安装 Claude 并点击“安装”。如果管理员已为您安装，请跳过此步骤。

2. 打开 Google Docs、Sheets 或 Slides 文件，然后转到“扩展程序”> Claude > “打开 Claude”：

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2707281468/abc0497defca04936c9a4b97d147/8b17d05f-3fb1-4e49-8d1f-65364ac3f496?expires=1795269600&amp;signature=b755774a4927601113b136eec49adf1842b7770d277b88911c95744443de8fd9&amp;req=dicnEct2nIVZUfMW1HO4zRwmJTluUu1JaAKM49NdBLMfElpNQus8hQ6P7%2FXj%0A4hsQ%0A)

3. 第一次时，Google 会要求您允许两个权限。查看它们并点击“允许”：

  ![](https://downloads.intercomcdn.com/i/o/lupk8zyo/2707282342/f583a396098ff16e3cbd156d4a2a/112c6459-79c6-4fbb-9bc0-748fda6f592f?expires=1795269600&amp;signature=ccaf0a93523dae37be20d7055cb19f13eedf2bf571698d21aad36f18f449f372&amp;req=dicnEct2n4JbW%2FMW1HO4zeKY3pkEhgf2eBfHlrCfDWfBkxSQrG8wDpm%2BwpRq%0A%2BiFE%0A)

4. 在侧边栏中使用您的 Claude 账户登录，并可选择启用您的连接器。

5. 然后向 Claude 提出关于该文件的问题。尝试“用五个要点总结此文档”或“F 列计算什么？”

如果安装按钮呈灰色或您看到“此应用程序不被您的管理员允许”，您的 Google Workspace 管理员需要先允许或安装 Claude。请参阅下面的 **[为您的组织安装](#h_2118c53c4b)**。

## 安装 Claude for Google Workspace

有两种安装方式：为自己安装或为整个组织安装。您可以使用哪一种取决于您的 Google Workspace 管理员的 Marketplace 设置，而不是您的 Claude 计划。

### 为自己安装

1. 在 Google Workspace Marketplace 中打开 **[Claude 列表](https://workspace.google.com/marketplace/app/claude/12459801340)**。

2. 点击“安装”，选择您用于 Docs、Sheets 和 Slides 的 Google 账户，然后点击“继续”。

3. 查看权限并点击“允许”。

4. 重新加载您已打开的任何 Docs、Sheets 或 Slides 文件。Claude 现在出现在所有三个应用的“扩展程序”菜单下。

一次安装涵盖 Docs、Sheets 和 Slides。您无需安装三个单独的扩展程序。

### 为您的组织安装

Google Workspace 超级管理员可以为所有人或特定群组或组织单位安装 Claude，这样用户就无需自己安装。

1. 以超级管理员身份登录时，打开 **[Claude 列表](https://workspace.google.com/marketplace/app/claude/12459801340)**。

2. 点击“管理员安装”，然后点击“继续”。

3. 选择“您组织中的所有人”或“特定群组或组织单位”，并选择谁应该获得 Claude。

4. 查看数据访问和条款，勾选协议框，然后点击“完成”。

5. 要求用户重新加载任何打开的文件。Claude 在**扩展程序**下出现，无需用户采取任何操作。

要稍后删除 Claude，请转到 Google 管理控制台，应用，Google Workspace Marketplace 应用，应用列表，选择“Claude”，然后点击“卸载应用”。删除在所有用户下次重新加载文件时生效。

### 如果您的组织阻止 Marketplace 应用

许多组织默认阻止 Marketplace 安装。管理员在 Google 管理控制台的“应用”、“Google Workspace Marketplace 应用”、“设置”下有两个选项：

- **将 Claude 加入允许列表**，以便用户可以自己安装：将 Marketplace 访问权限设置为“仅允许用户安装和运行允许列表中的应用”，然后在“应用列表”、“允许列表应用”下将 Claude 添加到允许列表。

- **管理员使用上述步骤安装**。即使用户安装被阻止，管理员安装也能正常工作。

尝试安装被阻止应用的用户会看到“此应用程序不被您的管理员允许”。该消息来自 Google，而不是 Claude，只有 Workspace 管理员才能清除它。

## 您可以要求 Claude 做什么

当您告诉 Claude 您想要的结果并让它找出步骤时，Claude 效果最好。您可以先选择文本、单元格或幻灯片，以将 Claude 指向文件的特定部分。

### 在 Google Docs 中

- “重写执行摘要，使其以建议开头并适合一个段落。”

- “这份简报有 14 页。将其缩减到 6 页，不丢失任何编号的要求。”

- “将“后续步骤”下的注释转换为包含所有者、操作和截止日期的表格。”

- “检查此合同中定义但未使用的术语。”

- “将每个标题改为句首大写，并修复第 4 部分中的编号。”

### 在 Google Sheets 中

- “向我说明 H42 中的总计是如何计算的，并标记任何硬编码的内容。”

- “在新标签页上根据“输入”中的假设构建三表模型。”

- “D列包含三种不同格式的日期。将其规范化为YYYY-MM-DD格式。”

- “查找摘要标签页中的VLOOKUP函数为某些行返回#N/A的原因并修复。”

- “添加价格和销量的敏感性分析表，以及按季度划分的收入图表。”

### 在Google幻灯片中

- “根据第1张幻灯片演讲者备注中的要点起草五张项目更新幻灯片。”

- “第7张幻灯片文字过多。将其分成两张幻灯片并保持演示文稿的样式。”

- “使整个演示文稿中的标题格式一致，每个标题不超过八个单词。”

- “在第4张幻灯片上添加区域数据的柱状图，与演示文稿的颜色相匹配。”

- “重新设置此演示文稿的样式，使用标题幻灯片中的字体和颜色。”

Claude还可以回答问题而无需编辑任何内容：“Q2和Q3标签页之间有什么变化？”或“哪些幻灯片提到了定价？”

## Claude在您的文件中的工作原理

### Claude读取您打开的文件，仅限该文件

当您发送消息时，Claude会读取打开文件的相关部分，以及您选择的任何内容。除了您启用的任何连接器中的内容外，它无法看到您Drive中的其他文件、您的电子邮件或日历。如果您需要Claude使用其他地方的内容，请将其粘贴到聊天中或附加文件。

### 编辑会实时进行

Claude直接在文档、电子表格或演示文稿中进行更改。Google会以您的名义记录这些更改，因此它们会像您自己进行的任何编辑一样出现在“文件”>“版本历史记录”中。要撤销Claude所做的操作，请使用“撤销”来撤销最后一项更改，或从版本历史记录中恢复较早的版本。

## 您可以选择Claude询问的程度

侧边栏底部的模式选择器有两个设置：

| **模式**    | **发生的情况**                                                      |
| --------- | -------------------------------------------------------------- |
| 编辑前询问（默认） | Claude用简洁的语言向您展示它将要做什么，并等待您点击“允许”。                             |
| 接受所有编辑    | Claude进行普通内容编辑而无需停止。对于涉及链接、外部图像、网址或某人对文件的访问权限的任何操作，它仍会暂停以获得批准。 |

在Google文档中，您还可以提示Claude将文本更改建议为卡片，您可以逐个接受或拒绝，类似于建议功能。

### 某些操作始终需要您的批准，某些操作永远不被允许

无论模式如何，Claude在从网址插入图像、添加链接或移除任何人的访问权限之前都会停止并询问。这些操作会显示“安全审查”标签，且没有“始终允许”选项。

Claude无法共享文件、授予任何人访问权限或更改所有者。它也无法创建菜单或计划自动化，或将任意网页加载到您的文件中。即使您要求，这些请求也会被拒绝。

### 侧边栏中还有什么

- **模型选择器。**为对话选择Claude模型。

- **连接器。**您的组织在Claude中启用的连接器会出现在+菜单中。在claude.ai中添加新连接器，而不是在侧边栏中。

- **文件和图像。**附加参考文件（每条消息最多20个，每个文档30 MB，每个图像10 MB），供Claude与打开的文件一起阅读。

- **网络搜索。**当您希望Claude查找信息时，切换“搜索网络”。默认关闭。

- **聊天历史记录。**对话按文件保存在您的浏览器中，当您在该文件中重新打开Claude时会返回。它们不会跨设备或浏览器同步，文件副本会重新开始。

---

## 当前限制

Claude for Google Workspace处于测试阶段。这些是人们最常询问的限制。其中几个是有意的，因为解除这些限制的权限也会扩大扩展程序可以访问的范围。

### 所有三个应用

- 仅适用于Google文档、表格和幻灯片文件。Google不支持Word、Excel或PowerPoint文件中的扩展程序，即使您在文档、表格或幻灯片中打开它们也是如此。要在其中使用Claude，请先转换它（**文件 > 另存为Google文档**、**另存为Google表格**或**另存为Google幻灯片**）并在新文件中打开Claude。

- 仅适用于打开它的文件。Claude无法打开、创建或复制Drive中的其他文件，也无法将文件导出为PDF或下载。

- 在Claude工作时保持侧边栏打开。关闭它会停止当前任务。

- 使用Chrome、Edge或Safari。此测试版不支持Firefox；侧边栏会加载但操作无法完成。

- 侧边栏中不提供语音听写功能。

- 单个长时间操作最多可运行六分钟，然后Google会停止它。对于非常大的任务，请要求Claude分阶段工作。

- “跨应用工作”（在Excel、Word和PowerPoint之间传递任务）是Microsoft 365功能，在Google Workspace中不可用。

### Google文档

- Claude无法读取、回复或解决评论。如果您希望Claude对其进行操作，请将评论粘贴到聊天中。

- Claude无法接受或拒绝现有建议，其自己的编辑会作为直接编辑而不是建议出现。如果您想先审查每项更改，请使用“编辑前询问”。

- Claude无法创建、重命名或重新排序文档标签页。它可以读取和编辑每个标签页的内容。

### Google表格

- 连接的表格标签页（实时BigQuery数据）无法读取或编辑。使用“数据”>“数据连接器”>“提取”将行提取到常规标签页中。

- Claude 无法创建触发器、自定义菜单或定时刷新。

### Google Slides

- Claude 创建的图表作为图像插入，而不是链接的 Sheets 图表，因此当数字变化时不会更新。请要求 Claude 重新生成图表。

- Claude 无法刷新现有的链接图表。更改源工作表后，请自己点击图表上的“更新”。

- Claude 无法在没有您每次批准的情况下从网址插入图像。

---

## 安全性、隐私和管理员控制

Claude 仅查看您打开的文件以及您启用的任何连接器，您的内容根据现有 Claude 条款处理，不用于训练，Google Workspace 管理员控制谁可以使用该扩展程序。

### Claude 请求的 Google 权限

Claude 请求两个 Google 权限，这是显示侧边栏和编辑打开的文件所需的最少权限。它不请求访问 Drive、Gmail、Calendar、您的联系人或从您的文件发出网络请求的权限。

| **Google 向您显示的权限**                 | **Claude 需要它的原因**                                      |
| ---------------------------------- | ------------------------------------------------------ |
| 在 Google 应用程序中的提示和侧边栏中显示和运行第三方网络内容 | 在 Docs、Sheets 和 Slides 中显示 Claude 侧边栏                  |
| 查看和管理已安装此应用程序的文档、电子表格或演示文稿         | 让 Claude 读取和编辑您打开的一个文件。Google 强制执行此操作：该扩展程序无法打开任何其他文件。 |

### 您的数据如何处理

- 来自打开文件的内容、您的消息和您附加的任何文件都会发送到 Claude 以生成响应。对于 Team 和 Enterprise 计划，这由您组织的商业条款和数据处理附录管理。对于 Pro、Max 和 Ultra，这由消费者条款和隐私政策管理。

- Anthropic 不使用此内容来训练其模型。从 Google Workspace API 接收的数据的使用遵守**[Google API 服务用户数据政策](https://developers.google.com/terms/api-services-user-data-policy)**，包括有限使用要求。

- 输入和输出在 30 天内从 Anthropic 的系统中删除，除非在**[您存储我的组织数据多长时间？](https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data)中另有说明。该扩展程序不继承您的组织可能设置的自定义数据保留设置。**

- 聊天历史记录存储在您的浏览器中本地，按文件存储，不会跨设备同步。从侧边栏的设置中清除它。

- 在 Enterprise 计划上，Google 扩展程序的会话可通过合规性 API 获得，包括提示、响应、工具调用和 Claude 所做的编辑。

### Enterprise 安全控制

标准 Claude Enterprise 控制今天如何应用于 Google Workspace 扩展程序：

- **合规性 API：**在 Enterprise 计划上受支持。会话通过与 Claude for Microsoft 365 相同的端点和合规性访问密钥捕获和返回，包括提示、响应、工具调用和编辑。

- **客户管理的加密密钥 (CMEK)：**支持 Enterprise。Anthropic 存储的对话数据使用客户的密钥加密，无需额外配置。

- **OpenTelemetry 审计导出：**受支持。如果 Claude 管理员在组织设置中为 Office 代理配置了自定义 OpenTelemetry 收集器，来自 Docs、Sheets 和 Slides 的会话会将完整审计跟踪发送到它，包括提示和工具输入和输出。

- **第三方推理（Amazon Bedrock、Google Cloud Vertex AI、Azure AI Foundry 或 LLM 网关）：**在此测试版中不受支持。该扩展程序需要 Claude 账户登录。Claude for Microsoft 365 支持这些平台；Google Workspace 扩展程序尚不支持。

- **零数据保留 (ZDR)：**不可用。标准保留适用，如上所述。由于不支持第三方推理，目前扩展程序没有零保留部署路径。

- **HIPAA 就绪组织：**不涵盖。请勿将该扩展程序用于受保护的健康信息。

### 管理员控制

- **Google Workspace 管理员**通过 Marketplace 允许列表和管理员安装决定谁可以安装或使用 Claude，范围限定为整个域、组或组织单位。从管理控制台卸载会为所有人删除它。

- **Claude 管理员**控制哪些连接器、技能和模型可供成员使用。侧边栏从您的 Claude 组织继承这些设置。

- **用户使用其 Claude 账户登录**，如果您的组织使用 SSO，则包括 SSO。Google 扩展程序中不支持 API 密钥和云提供商登录。

### 提示注入

文件可能包含他人编写的指令，有时隐藏在白色文本或极小的字体中。Claude 将文档内容视为数据而非命令，标记它找到的隐藏文本，并设计为不遵循文档的指令来共享文件、获取链接或联系任何人。您仍应在批准 Claude 的建议操作之前进行审查，特别是在您从组织外部收到的文件中。

---

## 故障排除

### Claude 未出现在扩展程序菜单中

扩展程序仅在安装后打开的文件中显示，因此如果 Claude 未出现在菜单中，请重新加载文件。如果仍然缺失，请检查您是否使用安装 Claude 的账户登录到 Google，然后要求您的 Google Workspace 管理员确认为您安装或允许列表了 Claude。

**提示：**按 Alt+/（Mac 上为 Option+/）并输入"Claude"以快速找到菜单项。如果文件是 Word、Excel 或 PowerPoint 文件，Claude 不会出现。首先使用**文件 > 另存为 Google Docs**、**Sheets**或**Slides**进行转换。

### “您的管理员不允许此应用程序”

您的 Google Workspace 组织阻止未被允许列表的 Marketplace 应用程序。这是 Google 设置。请要求您的 Workspace 管理员允许列表 Claude 或管理员安装它。请参阅如果您的组织阻止 Marketplace 应用程序。

### “Claude for Google Workspace 需要 Claude 账户”

您之前使用 API 密钥、Amazon Bedrock、Google Vertex AI 或网关登录到 Claude 加载项。Google 扩展程序仅支持 Claude 账户登录。

### 侧边栏打开，但发送消息时没有任何反应

这可能是由于使用 Firefox 或具有严格跟踪保护的浏览器导致的，该浏览器阻止侧边栏与文档通信。切换到 Chrome、Edge 或 Safari。如果您已在其中一个上，请重新加载文件并重新打开 Claude。

### Claude 无法读取或更改您的文件，或每个操作都失败

这通常发生在您的浏览器登录到多个 Google 账户时。在仅登录一个 Google 账户的浏览器窗口中打开文件，例如单独的 Chrome 配置文件。然后从**扩展程序**菜单重新打开 Claude。

### “此文档的 Claude 加载项脚本已过期”

Google 仍在向此文件提供较旧版本的扩展程序。重新加载文件并重新打开 Claude。如果持续超过几分钟，请稍后重试。

### “超过最大执行时间”或任务中途停止

Google 限制单个扩展程序操作的时间为六分钟。请要求 Claude 继续，或将请求分解为更小的步骤，例如一次一个标签页或十张幻灯片。

### Claude 表示无法读取 Sheets 中的标签页

该标签页是由 BigQuery 支持的关联工作表。使用“数据”、“数据连接器”、“提取”将行复制到常规标签页，然后重新尝试。

### 我的对话消失了

聊天记录按文件保存在您的浏览器中。在不同的浏览器、隐私窗口、清除网站数据后或使用“文件”、“制作副本”创建的文件副本中将不会显示。

---

## 常见问题

### 我需要为 Docs、Sheets 和 Slides 分别安装 Claude 吗？

不需要。从 Marketplace 进行一次安装会将 Claude 添加到所有三个应用中。

### Claude 能看到我在 Google Drive 中的其他文件吗？

默认情况下不能。Google 将扩展程序限制在打开它的单个文件中。但是，如果您连接了 Google Drive MCP，Claude 可以使用它来查看其他文件。此功能默认关闭。

### 文件中的其他人能看到我与 Claude 的对话吗？

不能。对话存在于您的浏览器中。协作者可以在文件和版本历史记录中看到 Claude 的编辑（归属于您），但看不到您的聊天内容。

### 如果我关闭侧边栏或标签页，Claude 会继续工作吗？

不会。请保持侧边栏打开，直到 Claude 完成。您可以在它工作时切换到其他浏览器标签页。

### 我如何撤销 Claude 所做的操作？

对最近的更改使用“撤销”，或使用“文件”、“版本历史记录”、“查看版本历史记录”来恢复较早的版本。每个 Claude 编辑都是文件历史记录中的常规编辑。

### 我可以使用哪些 Claude 模型？

与您的 Claude 计划相同的可用模型。从侧边栏底部的模型选择器中选择一个。

### 这与 Chrome 中的 Claude 或 Google Drive 连接器相同吗？

不同。Chrome 中的 Claude 可在您浏览器中的网页上工作；Google Workspace 连接器让 Claude 从 claude.ai 搜索您的 Drive、Gmail 和 Calendar。此扩展程序将 Claude 放在特定的 Docs、Sheets 或 Slides 文件中，并让它编辑该文件。

### 我也使用 Microsoft 365。有等效的产品吗？

有。请参阅 **[Claude for Microsoft 365](https://claude.com/claude-for-microsoft-365)**。