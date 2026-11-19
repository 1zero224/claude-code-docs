# 使用 Google Workspace 连接器

将您的 Gmail、Google 日历和 Google Drive 连接到 Claude，这样您可以搜索和发送电子邮件、管理日历、处理文档和保存文件，所有操作都无需离开对话。

Google Workspace 连接器（Gmail、Google 日历和 Google Drive）适用于 Claude 和 Claude Desktop 上的所有用户。

Claude 可以在 Claude 网页版和 Claude Desktop 上，在聊天旁边的窗格中实时编辑 Google Docs、Sheets 和 Slides。您和 Claude 可以同时编辑同一文件。请参阅**[当前限制](#h_cbd94d5da5)**了解不支持的功能。

对于 Team 和 Enterprise 计划，Owner 或 Primary Owner 必须在组织级别启用这些连接器，然后个人用户才能进行身份验证。有关设置说明，请阅读**[使用连接器扩展 Claude 的功能](https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities#h_17dd443beb)**。

## 您可以做什么

### Gmail

- **使用自然语言查询搜索和阅读电子邮件**

- **起草格式正确且包含上下文的电子邮件**

- **从 Gmail 发送、回复和转发电子邮件**。默认情况下，在非自动模式下，Claude 会在执行这些操作之前请求您的批准。在 Team 和 Enterprise 计划上，所有者决定成员是否可以允许这些操作在不每次都询问的情况下运行。

- **访问电子邮件元数据**，包括附件元数据

- **打开电子邮件附件**，以便 Claude 可以读取或处理文件，每个文件最多约 12 MB。在聊天中，这需要启用代码执行和文件创建。

- **使用标签和线程管理电子邮件组织**

- **列出 Gmail 账户中保存的草稿**

### Google 日历

- **查看事件和日历**，包括您有权访问的共享日历

- **创建、更新和删除事件**，具有完整的自定义选项

- **查找与会者之间的共同可用时间**

- **管理与会者列表**并回复邀请（接受、拒绝或暂定）

- **设置定期会议**

### Google Drive

- **从您的 Drive 中搜索和检索 Google Docs**

- **在 Google Drive 中共享、移动和删除文件**。默认情况下，在非自动模式下，Claude 会在执行这些操作之前请求您的批准。在 Team 和 Enterprise 计划上，所有者决定成员是否可以允许这些操作在不每次都询问的情况下运行。

- **查找文件元数据**并直接预览，无需先搜索。

- **读取** Sheets、Slides、PDF、图像和 MS Office 文件。

- **上传任何文件类型**，可选择自动转换为 Google 格式。

- **创建文件夹**。

- **通过粘贴 URL 或从最近的文档中选择，直接将 Google Docs 添加到聊天和项目**

- **查看文件权限**并**列出对您的 Drive 的最近更改**。

- **将 Claude 生成的文件直接保存到您的 Drive**（需要启用**[代码执行和文件创建](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#h_1c99382190)**）。

**注意：**当 Claude 读取 Google Drive 文件时，它仅提取文本内容。文档中嵌入的图像不会被处理。

### Google Docs、Sheets 和 Slides

- **在聊天旁边的窗格中实时编辑文件。**您和 Claude 可以同时在同一文件中工作。

- **创建新的 Google Docs、Sheets 和 Slides 文件。**

- **在 Google Docs 中读取和处理评论和建议**

这些是与 Google Drive 分开的连接器。Google Drive 仍然处理搜索、上传和共享文件。

---

## 如何使用连接器

向 Claude 提出需要访问您的 Gmail、日历或 Drive 的问题。Claude 会自动检测所需的工具并使用它们来响应。默认情况下，在非自动模式下，Claude 代表您采取的每项操作都需要您的明确批准。在**[自动模式](https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork#h_c274f6c9b5)**中，这些操作可以在不询问您的情况下运行，除非自动检查将其标记为有风险或您的组织需要批准。该检查可能会遗漏有风险的操作。要自己批准它们，请使用手动模式或将工具的权限设置为**[“始终询问”](https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork#h_c274f6c9b5)**（如果该选项可用）。在 Team 和 Enterprise 计划上，所有者决定成员是否可以允许某些操作在不每次都询问的情况下运行。

Claude 的回复包含引用，指示哪些电子邮件、日历事件或文档被用作来源，并在可用时提供指向原始内容的链接。您可以提出后续问题以深入了解任何来源。

### 将 Google Drive 文件添加到聊天

1. 点击聊天界面中的加号。

2. 选择“从 Google Drive 添加。”

3. 搜索您最近访问的文档或粘贴文档的 URL。

4. 发送消息时，Claude 会访问并处理文档以为其响应提供信息。

您可以将多个文件添加到单个聊天中，为 Claude 提供全面的上下文。这些文件必须符合对话的上下文窗口。

### 将 Google Drive 文件添加到项目

Google Drive 连接器仅在添加到私有项目中的**文件**时可用。此选项将对共享项目禁用。

1. 打开您的项目并找到**文件**部分。

2. 点击“+”按钮添加文件。

3. 选择“Drive。”

4. 搜索您最近访问的文档或粘贴文档的URL。

5. 您的文档被添加到项目知识库中，Claude在该项目中聊天时可以访问。

添加到聊天和项目中的Google文档直接从Google Drive同步，因此您始终使用最新版本。

### 实时编辑Google文档、表格和幻灯片

1. 打开文档、表格和幻灯片连接器。点击聊天中的加号，悬停在“连接器”上，然后将其打开。如果在您要求Claude编辑Google文件时连接器未打开，Claude会提示您连接它，然后继续您的请求。

2. 将Google文档、表格或幻灯片链接粘贴到聊天中，或要求Claude按名称创建一个（例如，“根据这些笔记制作Google幻灯片演示文稿”）。要求通用“文档”或“演示文稿”而不指定Google会创建本地文件。

3. 文件在对话旁的窗格中打开，您可以在Claude编辑时继续在其中工作。

  1. 该窗格适用于Chrome上的Claude网页版和打开了**[内置浏览器](https://support.claude.com/en/articles/16607400)**的Claude桌面版。如果窗格不可用，请在文件卡上选择“在Google中打开”以在常规浏览器中打开文件。

4. 使用窗格标题复制文件的Google链接或直接在Google中打开它。

Claude只能打开和更改您的Google账户有权访问的文件。您现有的Google共享权限适用。

Claude创建或编辑的每个文件都在聊天中显示为卡片。选择卡片在窗格中打开文件，或选择“在Google中打开”在Google中打开它。在窗格中，您可以复制文件的Google链接或使用共享对话框。

在窗格中登录Google：窗格需要Google会话来显示您的文件。在Claude网页版上，它使用已登录到您浏览器的Google账户。在Claude桌面版上，Claude要求您在窗格中登录Google。如果窗格中的账户与您连接到Claude的账户不同，Claude会告诉您并提供切换选项。

### 管理单个连接器

实时编辑使用单独的Google文档、Google表格和Google幻灯片连接器，以及Gmail、Google日历和Google Drive。您可以从聊天界面下方启用或禁用特定连接器：

1. 点击聊天界面中的加号。

2. 悬停在“连接器”上。

3. 单独打开或关闭连接器。

---

## 隐私和数据处理

- 使用这些连接器之前，您必须直接使用Google账户进行身份验证。

- 在Team和Enterprise计划上，所有者或主要所有者必须在账户级别启用连接器，然后个人用户才能进行身份验证。

- Claude只能访问您已连接的Google账户的Gmail、日历和Drive数据。

- Claude仅在您明确提出问题或请求需要此信息的操作时访问您的数据，并检索所需的最少信息。

- Claude反映您现有的权限——您无法访问您在Google Workspace中已无权访问的信息。

- 使用连接器检索的数据存储在Anthropic服务器上，受Anthropic安全基础设施保护（详见我们的**[信任中心](https://trust.anthropic.com/controls#organizational-security)**）。此数据与其关联的聊天一起保留，因此您可以通过删除聊天来删除任何检索的数据。

- 我们不会在您的Gmail、Drive或日历连接器数据上训练我们的模型，确保您的私人信息保持私密。

  - **注意**：如果您使用我们的消费者产品（例如Claude Free、Pro、Max、Ultra，以及使用这些账户的Claude Code），并且您已选择允许我们使用您的聊天和编码会话进行模型训练，那么您从Gmail、Drive、日历复制/粘贴的任何内容或Claude的响应中包含来自这些连接器的特定信息可能会被用于改进我们的模型。有关更多信息，请参阅**[我的数据是否用于模型训练？](https://privacy.claude.com/en/articles/10023580-is-my-data-used-for-model-training)**

- 数据在静止和传输中都经过加密。

- Claude会议（测试版）也使用您的Google日历连接，工作方式与上述不同。请参阅**[使用Claude会议](https://support.claude.com/en/articles/18325154-use-claude-meetings)**。

### 关于Gmail权限的说明

在身份验证期间，Google的OAuth屏幕会提及电子邮件发送权限。Claude可以发送、回复和转发电子邮件，但在自动模式之外，默认情况下仅在您明确批准时才这样做。在Team和Enterprise计划上，所有者决定成员是否可以允许这些操作在不每次询问的情况下运行。

---

## 当前限制

- Claude无法向Google幻灯片添加图表，幻灯片编辑可能不会在窗格中实时显示。

- 窗格中的实时编辑适用于Chrome上的Claude网页版和打开了内置浏览器的Claude桌面版。在其他情况下，请在文件卡上使用“在Google中打开”。

- Claude无法打开大约12 MB以上的Gmail附件。并非在每个组织中都可以打开附件，或者当组织设置阅读电子邮件需要批准或阻止时。

- 某些高级Gmail过滤器可能不受支持。

- 复杂查询可能需要多个API调用。

- 速率限制按Google的API配额应用。

- 大型邮箱的性能可能会有所不同。

---

## 故障排除

### 重新连接您的Google账户

如果您看到错误“*请重试。您可能需要使用Google Drive账户重新连接”*，请按照以下步骤操作：

1. 导航到**[自定义 > 连接器](https://claude.ai/customize/connectors)**。

2. 找到您的Google连接器并点击它以查看详细信息。

3. 点击“断开连接”按钮。

4. 在确认消息中点击“断开连接”。

5. 下次在聊天中使用Google Workspace功能时，您将被重定向到Google进行重新身份验证。

如果您仍然遇到问题，请从您的Google账户设置中断开连接，地址为[**myaccount.google.com/connections**](https://myaccount.google.com/connections)。搜索Claude连接，点击设置，然后选择“删除所有连接”。

### Google Workspace管理员配置

如果您的组织使用Google Workspace且连接器不工作（您可能会看到消息*“访问被阻止：您的机构管理员需要审查Claude for Google Drive”*），您的Google Workspace管理员可能需要允许Claude作为受信任的应用程序：

1. 转到**[admin.google.com](https://admin.google.com)**

2. 导航至**安全 > 访问和数据控制 > API 控制 > 管理第三方应用访问**。

3. 点击“添加应用，”然后选择“OAuth 应用名称。”

4. 搜索“Claude”并将其设置为“受信任。”

5. 等待约 15 分钟以便 Google 的政策传播，然后尝试重新连接。

Docs、Sheets 和 Slides 的实时编辑使用相同的 Claude for Google Drive 应用。管理员只需批准一次，无需为每种文件类型单独批准。

---

## 常见问题

### 将 Google 文档添加到聊天后，如果我更新它会发生什么？

该文档将继续与 Google Drive 中的最新版本同步。

### 我可以将多个文件添加到单个聊天中吗？

可以。您可以添加多个 Google Drive 文件为 Claude 提供全面的上下文。这些文档必须符合对话的上下文窗口限制。

### 如果我失去对文档的访问权限会发生什么？

您将无法在之前添加该文档的对话中查看其内容。文档预览将被移除，但对话历史记录会保留。

### Claude 能否访问 Google Docs 中的图像、评论或建议？

Claude 可以看到 Google Docs 中的评论和建议，但看不到图像。

### 我如何让 Claude 创建 Google 文件而不是本地文件？

按名称请求 Google 文档、Google 表格或 Google 幻灯片，或粘贴现有 Google 文件的链接。如果您请求文档时未提及 Google，Claude 将创建一个您可以下载的文件。

### Claude 编辑文件时我可以编辑它吗？

可以。您和 Claude 可以同时在同一文件中工作。Claude 在每次更改前重新读取当前内容，但在繁忙的文件中，请检查结果。

### 为什么 Claude 要求我在窗格中登录 Google？

连接器允许 Claude 更改您的文件，窗格需要自己的 Google 会话来显示它。请使用您连接到 Claude 的同一 Google 账户登录。

### Claude 可以代表我发送电子邮件吗？

可以。Claude 可以从 Gmail 发送、回复和转发电子邮件，在自动模式外，默认在执行这些操作前请求您的批准。在**[自动模式](https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork#h_c274f6c9b5)**中，除非自动检查将其标记为风险或您的组织要求批准，否则这些操作可以在不询问您的情况下运行。该检查可能会遗漏风险操作。要自己批准它们，请使用手动模式或将工具权限设置为**[“始终询问”](https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork#h_c274f6c9b5)**（如果该选项可用）。在 Team 和 Enterprise 计划中，所有者决定成员是否可以允许这些操作在不每次都询问的情况下运行。

### 我可以选择不使用这些连接器吗？

在 Team 和 Enterprise 计划中，所有者和主要所有者可以在**[组织设置 > 连接器](https://claude.ai/admin-settings/connectors)**中为其组织禁用 Google Workspace 连接器。个人用户也可以从聊天界面禁用特定工具。

在**[连接器目录](https://claude.ai/directory)**中浏览所有可用连接器。