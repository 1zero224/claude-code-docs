# 使用 Google Workspace 连接器

将您的 Gmail、Google 日历和 Google Drive 连接到 Claude，这样您就可以搜索和发送电子邮件、管理日历、处理文档和保存文件，所有操作都无需离开对话。

Google Workspace 连接器（Gmail、Google 日历和 Google Drive）适用于 Claude 和 Claude Desktop 上的所有用户。

Claude 可以在 Claude 网页版和 Claude Desktop 上的聊天旁边的窗格中实时编辑 Google Docs、Sheets 和 Slides。您和 Claude 可以同时编辑同一文件。请参阅**[当前限制](#h_cbd94d5da5)**了解不支持的功能。

对于 Team 和 Enterprise 计划，所有者或主要所有者必须在组织级别启用这些连接器，然后个人用户才能进行身份验证。有关设置说明，请阅读**[使用连接器扩展 Claude 的功能](https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities#h_17dd443beb)**。

## 您可以执行的操作

### Gmail

- **使用自然语言查询搜索和阅读电子邮件**

- **起草格式正确且包含上下文的电子邮件**

- **从 Gmail 发送、回复和转发电子邮件**。默认情况下，Claude 会在执行这些操作之前请求您的批准。在 Team 和 Enterprise 计划上，所有者决定成员是否可以允许这些操作在不每次都询问的情况下运行。

- **访问电子邮件元数据**，包括附件元数据（不包括附件内容）

- **使用标签和线程管理电子邮件组织**

- **列出 Gmail 账户中保存的草稿**

### Google 日历

- **查看事件和日历**，包括您有权访问的共享日历

- **创建、更新和删除事件**，具有完整的自定义选项

- **查找与会者之间的共同可用时间**

- **管理与会者列表**并回复邀请（接受、拒绝或暂定）

- **设置定期会议**

### Google Drive

- **从您的 Drive 中搜索和检索 Google Docs**。

- **在 Google Drive 中共享、移动和删除文件**。默认情况下，Claude 会在执行这些操作之前请求您的批准。在 Team 和 Enterprise 计划上，所有者决定成员是否可以允许这些操作在不每次都询问的情况下运行。

- **查找文件元数据**并直接预览，无需先搜索。

- **阅读** Sheets、Slides、PDF、图像和 MS Office 文件。

- **上传任何文件类型**，可选择自动转换为 Google 格式。

- **创建文件夹**。

- **通过粘贴 URL 或从最近的文档中选择，直接将 Google Docs 添加到聊天和项目**。

- **查看文件权限**并**列出对您的 Drive 的最近更改**。

- **直接将 Claude 生成的文件保存到您的 Drive**（需要启用**[代码执行和文件创建](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#h_1c99382190)**）。

**注意：**当 Claude 读取 Google Drive 文件时，它仅提取文本内容。文档中嵌入的图像不会被处理。

### Google Docs、Sheets 和 Slides

- **在聊天旁边的窗格中实时编辑文件。**您和 Claude 可以同时在同一文件中工作。

- **创建新的 Google Docs、Sheets 和 Slides 文件。**

- **在 Google Docs 中阅读和处理评论和建议**。

这些是与 Google Drive 分开的连接器。Google Drive 仍然处理文件的搜索、上传和共享。

---

## 如何使用连接器

向 Claude 提出需要访问您的 Gmail、日历或 Drive 的问题。Claude 会自动检测所需的工具并使用它们来响应。默认情况下，Claude 代表您执行的每个操作都需要您的明确批准。在 Team 和 Enterprise 计划上，所有者决定成员是否可以允许某些操作在不每次都询问的情况下运行。

Claude 的响应包括引用，指示哪些电子邮件、日历事件或文档被用作来源，并在可用时提供指向原始内容的链接。您可以提出后续问题以深入了解任何来源。

### 将 Google Drive 文件添加到聊天

1. 单击聊天界面中的加号。

2. 选择"从 Google Drive 添加"。

3. 搜索您最近访问的文档或粘贴文档的 URL。

4. 发送消息时，Claude 会访问并处理文档以为其响应提供信息。

您可以将多个文件添加到单个聊天中，为 Claude 提供全面的上下文。这些文件必须符合对话的上下文窗口。

### 将 Google Drive 文件添加到项目

Google Drive 连接器仅在向私有项目中的**文件**添加时可用。此选项对于共享项目将被禁用。

1. 打开您的项目并找到**文件**部分。

2. 单击"+"按钮添加文件。

3. 选择"Drive"。

4. 搜索您最近访问的文档或粘贴文档的 URL。

5. 您的文档已添加到项目知识库中，Claude 在该项目中聊天时可以访问。

添加到聊天和项目的 Google Docs 直接从 Google Drive 同步，因此您始终使用最新版本。

### 实时编辑 Google Docs、Sheets 和 Slides

1. 打开 Docs、Sheets 和 Slides 连接器。点击聊天中的加号，悬停在"连接器"上，然后将其打开。如果在您要求 Claude 编辑 Google 文件时连接器未打开，Claude 会提示您连接它，然后继续您的请求。

2. 将 Google Docs、Sheets 或 Slides 链接粘贴到聊天中，或要求 Claude 按名称创建一个（例如，"从这些笔记制作 Google Slides 演示文稿"）。要求通用的"文档"或"演示文稿"而不指定 Google 会创建本地文件。

3. 文件在对话旁的窗格中打开，您可以在 Claude 编辑时继续在其中工作。

  1. 该窗格适用于 Chrome 上的 Claude 网页版和打开了**[内置浏览器](https://support.claude.com/en/articles/16607400)**的 Claude Desktop。如果窗格不可用，请在文件卡上选择"在 Google 中打开"以在常规浏览器中打开文件。

4. 使用窗格标题复制文件的 Google 链接或直接在 Google 中打开它。

Claude 只能打开和更改您的 Google 账户有权访问的文件。您现有的 Google 共享权限适用。

Claude 创建或编辑的每个文件都在聊天中显示为卡片。选择卡片在窗格中打开文件，或选择"在 Google 中打开"在 Google 中打开它。在窗格中，您可以复制文件的 Google 链接或使用共享对话框。

在窗格中登录 Google：窗格需要 Google 会话来显示您的文件。在 Claude 网页版上，它使用已登录到您浏览器的 Google 账户。在 Claude Desktop 上，Claude 要求您在窗格中登录 Google。如果窗格中的账户与您连接到 Claude 的账户不同，Claude 会告诉您并提供切换选项。

### 管理单个连接器

实时编辑使用单独的 Google Docs、Google Sheets 和 Google Slides 连接器，以及 Gmail、Google Calendar 和 Google Drive。您可以从聊天界面下方启用或禁用特定连接器：

1. 点击聊天界面中的加号。

2. 悬停在"连接器"上。

3. 打开或关闭单个连接器。

---

## 隐私和数据处理

- 使用这些连接器前，您必须直接使用 Google 账户进行身份验证。

- 在 Team 和 Enterprise 计划中，所有者或主要所有者必须在账户级别启用连接器，然后个人用户才能进行身份验证。

- Claude 只能访问您已连接的 Google 账户的 Gmail、Calendar 和 Drive 数据。

- Claude 仅在您明确提出问题或请求需要此信息的操作时才访问您的数据，并检索所需的最少信息。

- Claude 镜像您现有的权限——您无法访问在 Google Workspace 中您已无权访问的信息。

- 使用连接器检索的数据存储在 Anthropic 服务器上，受 Anthropic 安全基础设施保护（详见我们的**[信任中心](https://trust.anthropic.com/controls#organizational-security)**）。此数据与其关联的聊天一起保留，因此您可以通过删除聊天来删除任何检索的数据。

- 我们不会在您的 Gmail、Drive 或 Calendar 连接器数据上训练我们的模型，确保您的私人信息保持私密。

  - **注意**：如果您使用我们的消费者产品（例如 Claude Free、Pro、Max、Ultra，以及使用这些账户的 Claude Code），并且您已选择允许我们使用您的聊天和编码会话进行模型训练，那么您从 Gmail、Drive、Calendar 复制/粘贴的任何内容或 Claude 的响应（包含来自这些连接器的特定信息）可能被用于改进我们的模型。有关更多信息，请参阅**[我的数据是否用于模型训练？](https://privacy.claude.com/en/articles/10023580-is-my-data-used-for-model-training)**

- 数据在静止和传输中都经过加密。

### 关于 Gmail 权限的说明

在身份验证期间，Google 的 OAuth 屏幕会提及电子邮件发送权限。Claude 可以发送、回复和转发电子邮件，但默认情况下仅在您明确批准时才这样做。在 Team 和 Enterprise 计划中，所有者决定成员是否可以允许这些操作在不每次询问的情况下运行。

---

## 当前限制

- Claude 无法向 Google Slides 添加图表，Slides 编辑可能不会在窗格中实时显示。

- 窗格中的实时编辑适用于 Chrome 上的 Claude 网页版和打开了内置浏览器的 Claude Desktop。在其他情况下，请使用文件卡上的"在 Google 中打开"。

- 附件内容无法通过 Gmail 直接访问（仅元数据）。

- 某些高级 Gmail 过滤器可能不受支持。

- 复杂查询可能需要多个 API 调用。

- 根据 Google 的 API 配额应用速率限制。

- 大型邮箱的性能可能会有所不同。

---

## 故障排除

### 重新连接您的 Google 账户

如果您看到错误"*请重试。您可能需要使用 Google Drive 账户重新连接*"，请按照以下步骤操作：

1. 导航到**[自定义 > 连接器](https://claude.ai/customize/connectors)**。

2. 找到您的 Google 连接器并点击它以查看详细信息。

3. 点击"断开连接"按钮。

4. 在确认消息中点击"断开连接"。

5. 下次在聊天中使用 Google Workspace 功能时，您将被重定向到 Google 进行重新身份验证。

如果您仍有问题，请从 Google 账户设置中断开连接，访问[**myaccount.google.com/connections**](https://myaccount.google.com/connections)。搜索 Claude 连接，点击进入设置，然后选择"删除所有连接"。

### Google Workspace 管理员配置

如果您的组织使用 Google Workspace 且连接器不工作（您可能会看到消息*"访问被阻止：您的机构管理员需要审查 Claude for Google Drive"*），您的 Google Workspace 管理员可能需要将 Claude 允许为受信任的应用程序：

1. 转到**[admin.google.com](https://admin.google.com)**

2. 导航到**安全 > 访问和数据控制 > API 控制 > 管理第三方应用访问**。

3. 点击"添加应用"，然后选择"OAuth 应用名称"。

4. 搜索"Claude"并将其设置为"受信任"。

5. 等待约15分钟以便Google的政策生效，然后尝试重新连接。

Docs、Sheets和Slides的实时编辑使用相同的Claude for Google Drive应用。管理员只需批准一次，无需为每种文件类型单独批准。

---

## 常见问题

### 将Google文档添加到聊天后更新它会发生什么？

该文档将继续与Google Drive中的最新版本同步。

### 我可以将多个文件添加到单个聊天中吗？

可以。您可以添加多个Google Drive文件为Claude提供全面的上下文。这些文档必须符合对话的上下文窗口限制。

### 如果我失去对文档的访问权限会发生什么？

您将无法在之前添加该文档的对话中查看其内容。文档预览将被移除，但对话历史记录会保留。

### Claude能否访问Google Docs中的图像、评论或建议？

Claude可以看到Google Docs中的评论和建议，但看不到图像。

### 我如何让Claude创建Google文件而不是本地文件？

按名称请求Google文档、Google表格或Google幻灯片，或粘贴现有Google文件的链接。如果您请求文档时未提及Google，Claude会创建一个您可以下载的文件。

### Claude编辑文件时我可以编辑该文件吗？

可以。您和Claude可以同时在同一文件中工作。Claude在每次更改前重新读取当前内容，但在繁忙的文件中，请检查结果。

### Claude为什么要求我在窗格中登录Google？

连接器允许Claude更改您的文件，窗格需要自己的Google会话来显示它。请使用您连接到Claude的同一Google账户登录。

### Claude可以代表我发送电子邮件吗？

可以。Claude可以从Gmail发送、回复和转发电子邮件，默认情况下在执行这些操作前会要求您批准。在Team和Enterprise计划中，所有者可以决定成员是否可以允许这些操作在不每次询问的情况下运行。

### 我可以选择不使用这些连接器吗？

在Team和Enterprise计划中，所有者和主要所有者可以在**[组织设置 > 连接器](https://claude.ai/admin-settings/connectors)**中为其组织禁用Google Workspace连接器。个人用户也可以从聊天界面禁用特定工具。

在**[连接器目录](https://claude.ai/directory)**中浏览所有可用连接器。