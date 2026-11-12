# 在 Claude Desktop 上的 Claude Cowork 中运行端点检测和响应 \(EDR\) 传感器

当 Claude Cowork 在用户的计算机上运行时，无论是独立运行还是作为将聊天和 Cowork 结合在一起的新统一 Claude 体验的一部分，它都会在隔离的虚拟机 (VM) 中运行 shell 命令和代码。计算机上已有的端点检测和响应 (EDR) 传感器无法看到虚拟机内部。本文向 IT 和安全管理员展示如何在虚拟机内运行传感器，以便虚拟机中的活动报告到您的安全团队已在使用的控制台。

在 Cowork 中运行 EDR 传感器目前以测试版形式提供给 Team 和 Enterprise 计划。它适用于 macOS 和 Windows 上的 Claude Desktop，以及连接到 Amazon Bedrock、Google Cloud Vertex AI 或 Microsoft Foundry 的 Claude Desktop。

**重要提示：**这仅涵盖用户计算机上的虚拟机。云中的 Cowork 会话在 Anthropic 的服务器上运行，您无法在那里安装传感器。

## 工作原理

1. 每次 Cowork 虚拟机启动时，Claude Desktop 都会将您的供应商的传感器包复制到虚拟机中，并根据您指定的供应商的批准密钥检查其签名。

2. 如果签名有效，Claude 会在虚拟机内运行供应商的安装脚本，每次虚拟机启动最多运行一次。如果未设置该设置，则不会安装任何内容。

3. 传感器连接到您的供应商的云并报告活动，方式与从计算机其余部分报告的方式相同。传感器可以将遥测数据和数据发送出虚拟机，超出 Cowork 网络允许列表。

虚拟机在第一次需要时启动（通常是当用户打开 Cowork 时），并一直运行到 Claude 退出或用户注销。

## 支持的供应商

Claude Desktop 仅安装使用批准的 EDR 供应商密钥签名的传感器包。

批准的供应商：

- CrowdStrike

- Evoke Security

- Neo

- Palo Alto Networks

- Pluto Security

- SentinelOne

- Tanium

如果您的供应商未列出，请要求他们使用下面**[EDR 供应商](#h_ae012dc926)**部分中的表单申请。如果您有 Anthropic 账户团队，请告知他们您需要此功能。

## 开始前

您需要：

- **macOS：**Claude Desktop 4.4212.0 或更高版本

- **Windows：**Claude Desktop 的 .msix 版本，4.4212.0 或更高版本，在 Windows 10 或 11 上运行，并启用硬件虚拟化和虚拟机平台功能

- **设备管理：**macOS 的 MDM 工具，或在 Windows 上设置注册表策略的方法，例如组策略或 Intune

然后按照您的 EDR 供应商的设置指南进行操作。每个供应商都提供自己的文件和说明。

## 常见问题

### 该集成是否涵盖云中的 Cowork 会话？

否。某些 Cowork 任务在 Anthropic 托管的云虚拟机上运行，您的 EDR 传感器无法在那里安装。该集成仅涵盖用户计算机上的本地虚拟机。

### 传感器可以看到什么？

传感器可以看到虚拟机内的活动和内容，其中可能包括提示、文件和 Claude 的响应。它将数据发送到您的 EDR 供应商。Anthropic 不接收或控制该数据。

### 该集成是否需要额外费用？

在测试版期间，Anthropic 不对该集成收取额外费用。您的 EDR 供应商可能会对传感器收费，请联系他们了解定价详情。

### 如果我删除 MDM 设置会发生什么？

删除设置不会删除已安装的传感器。Claude 会在后续虚拟机启动时停止运行该包。

### 传感器更新如何工作？

您的供应商维护该包。签名仅涵盖包文件，不涵盖传感器稍后下载的更新。签名在两年内过期，因此您需要从供应商部署重新签名的包。

## EDR 供应商

如果您是 EDR 供应商，并希望您的传感器在 Cowork 虚拟机中运行，请**[申请加入该集成](https://forms.gle/9CPwYUcdXoRVtqWE9)**。

## 条款

您的组织选择安全供应商。您的组织根据与该供应商的自己的协议使用 EDR 软件。

EDR 软件是您与 Anthropic 协议下的第三方功能。它不是 Anthropic 服务的一部分。

EDR 软件可以看到虚拟机内的活动和内容。这可能包括提示、文件和 Claude 的响应。它将数据发送给供应商。Anthropic 不接收或控制该数据，供应商也不代表 Anthropic 处理该数据。

您的组织负责 EDR 软件。这包括在部署前测试它，以及在 Claude 或 EDR 软件的任何更新后再次测试。Anthropic 不对因未进行测试而产生的索赔负责。

您的组织负责因 EDR 软件而产生的索赔，如您与 Anthropic 的协议中所述。