# Claude Sheets 入门

Claude Sheets 是内置于 Claude 的电子表格。您可以要求 Claude 创建跟踪器、预算表或模型，Claude 会生成一个可工作的表格，您、您的团队和 Claude 都可以一起编辑。本指南涵盖创建表格、与 Claude 协作编辑、导入和导出数据以及共享表格。

Claude Sheets 在 Pro、Max、Ultra、Team 和 Enterprise 计划中以测试版形式提供。在 Team 计划中默认启用。在 Enterprise 计划中，默认关闭，直到在**[组织设置 > 工件](https://claude.ai/admin-settings/artifacts)**中启用。

## Claude Sheets 的工作原理

表格是您和 Claude 在同一单元格中编辑的工件。Claude 可以添加标签页、编写跨标签页的公式、应用格式和条件格式，以及添加下拉菜单、复选框和图表。公式保留在单元格中，因此您可以查看和检查每个数字的计算方式。

当 Claude 更改表格时，每次编辑都会在进行时突出显示，因此您可以看到发生了什么变化。当其他人在表格中时，您都在同一时间在同一单元格中工作。

您创建的每个表格都保存在**工件**标签中。了解更多关于**[工件及其使用方法](https://support.claude.com/en/articles/17153992)**。

## Claude Sheets 与 Claude for Excel 的区别

Claude Sheets 将电子表格放在 Claude 内部。Claude for Excel 和 Claude for Google Workspace 将 Claude 引入您已在这些应用中保存的文件。

使用 Claude Sheets 创建轻量级共享表格，与 Claude 一起构建。当您的文件已存在于 Excel 或 Google Sheets 中，或您需要这些应用中的全套功能时，请使用 Claude for Excel 或 Claude for Google Workspace，或导出您的表格。

## 创建表格

### 从聊天开始

在任何聊天中向 Claude 要求创建表格。例如：

- “创建一个包含负责人、日期和状态的活动跟踪器。”

- “按团队和季度制作招聘预算。”

- “根据这些交易预测下一季度，并标记有风险的项目。”

### 从工件标签

1. 转到**工件**标签。

2. 选择“表格”磁贴。

3. 描述您想要的表格，或从空白表格开始。

### 获得更好结果的提示

说明表格的用途和您想要跟踪的内容。如果您已有数据，请将其提供给 Claude，以便从您自己的数据构建表格。

## 在表格中工作

- **询问 Claude：**要求进行更改，例如“添加预算余额列”或“突出显示任何逾期项目。”

- **询问范围：**选择单元格并询问 Claude 相关问题，例如“为什么这个总数比上个月高？”

- **直接编辑：**在单元格中输入、编写公式并自己格式化数据。

- **评论：**在单元格上为您的团队添加评论。

## 撤销 Claude 的更改

版本历史显示表格的早期版本，可以恢复早期版本，这也会删除之后所做的编辑。

## 导入和导出

要将数据导入表格，您可以：

- 导入 CSV 文件

- 从 Google Sheets 导入

- 从 Excel 导入

- 从空白表格开始

也可以从 Excel 或 Google Sheets 粘贴。要下载表格，请将其导出为 .xlsx 或 CSV 文件。要在其他工具中继续工作，请使用“发送至。”

## 共享表格

表格默认仅对您私密。要共享表格，请打开它并点击“共享。”您与之共享且具有“可编辑”权限的人可以与您和 Claude 一起处理它。了解更多关于**[共享工件](https://support.claude.com/en/articles/9547008)**。

## 使用情况

Claude Sheets 计入您计划的使用限制，与您在 Claude 中的其他工作一样。了解更多关于**[使用情况和长度限制如何工作](https://support.claude.com/en/articles/11647753)**。

## 为您的组织启用 Claude Sheets

本部分适用于 Team 和 Enterprise 计划的所有者和主要所有者。

在**[组织设置 > 工件](https://claude.ai/admin-settings/artifacts)**中的**模板**下启用或关闭 Claude Sheets。在 Team 计划中默认启用，在 Enterprise 计划中默认关闭。您的组织也必须启用工件。在 Enterprise 计划中，启用 Claude Sheets 后，自定义角色可以将其限制为特定组。在**[Team 和 Enterprise 计划的工件管理员指南](https://support.claude.com/en/articles/16994751)**中了解更多信息。

## 当前限制

Claude Sheets 处于测试阶段，涵盖常见的电子表格工作。我们会随着时间的推移添加更多功能。

- **大小：**表格至少可以容纳 200,000 个填充单元格。

- **图表：**可用条形图、折线图和饼图，以及 Claude 可以生成的多种其他图表类型。

- **尚不可用的功能：**数据透视表、受保护的范围、共享筛选视图、多选下拉菜单和 PDF 导出。

- **不支持的 Excel 功能：**宏、VBA、Power Query、DAX 和打印布局。

- **实时数据：**表格无法从连接的应用中提取实时数据，文档、演示文稿和仪表板无法使用表格作为数据源。

- **平台：**支持网页和桌面，移动设备上的访问受限。

- **数据控制：**Claude Sheets 是测试版产品，因此尚不支持 HIPAA 和 ZDR。