# 开始使用 Claude 合规 API 集成

安全和合规平台已在 Claude 合规 API 基础上构建了集成，因此您的团队可以在您已使用的工具中监控 Claude 活动。集成可从跨越企业依赖的多个类别的安全和合规提供商获得，包括 DLP、SASE、数据安全、SIEM 和安全运营、身份、电子数据发现、AI 安全态势管理以及 AI 可观测性和遥测基础设施。

本文列出了可用的合规 API 集成及其设置文档的链接。如果您是已构建合规 API 集成的安全、合规或 IT 平台，**[请在此申请加入该计划](https://forms.gle/1rkBcdwwR5bLLD82A)**。

Claude 合规 API 可供 Claude Enterprise 计划和 Claude Platform 客户使用。

## 您可以访问的数据

合规 API 根据产品提供对不同数据的访问权限：

- **Claude Enterprise：**

  - **对话内容**，包括聊天、上传的文件和项目

  - **来自 Cowork（通过 Claude、Claude Desktop 和 Claude Mobile）和 Claude Code（通过 CLI 和 Claude Desktop）的会话内容。**包括提示和响应、工具调用内容以及作为文本记录捕获的技能和工件内容。

  - **来自 Claude for Microsoft 365 加载项（Excel、Word、PowerPoint 和 Outlook）和 Claude Science 的会话内容**（测试版）。包括提示和响应，以及来自 Microsoft 365 加载项的草稿电子邮件、公式和文档文本。

  - **活动源事件**，涵盖用户登录、管理员操作和配置更改

- **Claude Platform：**

  - **活动源事件**，包括管理员和系统事件（如成员和工作区更改、API 密钥创建和账户设置）以及资源事件（如文件创建、下载和技能更改）

  - **注意：**对话内容（提示和模型响应）不可通过 Claude Platform 的合规 API 获得。

## 如何设置集成

对于 Claude Enterprise 客户，使用集成非常简单：

1. 请您组织的主要所有者启用合规 API（仅主要所有者可以执行此操作），然后创建访问密钥——所有者可以创建仅限于其自己组织的密钥。了解如何**[访问合规 API](https://support.claude.com/en/articles/13015708-access-the-compliance-api)**。

2. 将您的实例连接到受支持的安全平台。设置指南可在**[可用集成部分](#h_7ff9e34f8e)**中获得。

Claude 活动将流入您的团队用于所有其他应用程序的相同仪表板和工作流。

**重要**：对于 Claude Enterprise 组织，仅您组织的主要所有者可以从**[组织设置 > API](https://claude.ai/admin-settings/api-access)** 启用合规 API。您可以通过在**密钥**下单击"+ 创建密钥"在同一页面上创建密钥：主要所有者可以创建涵盖所有关联组织的密钥，所有者可以创建仅限于其自己组织的密钥。所有者可以看到此页面，但看不到**合规 API** 切换——仅主要所有者可以打开或关闭 API。管理员根本看不到此页面。如果您的组织关联到父组织，则父组织的主要所有者启用它，该设置适用于所有关联的组织。

对于 Claude Platform，请联系您的 Anthropic 销售团队。查看 Claude API 文档上的**[合规 API 文档](https://platform.claude.com/docs/en/manage-claude/compliance-api)**。

## 可用集成

- **Above Security：**Above Security 通过 Claude 合规 API 将企业 Claude 活动——claude.ai 聊天和 Claude Code、Cowork 以及远程 Cowork 会话——引入其 AI 驱动的内部风险平台。只读且隐私保护，它为安全团队提供了证据充分、自动化的调查，了解组织内人员如何使用 Claude。要设置此集成，请参阅**[Above Security 设置指南](https://www.above.security/integrations)**。

- **Air：**Air 将 Claude 合规 API 用作其代理安全平台的核心部分，为安全团队提供统一的态势管理、实时运行时保护以及跨所有端点、云和 SaaS 代理的供应链治理，从单一控制平面进行管理。要设置此集成，请参阅**[Air 设置指南](https://www.air.security/claude-air-integration)**。

- **Airia：**Airia 与合规 API 的集成为安全和风险团队提供了策略执行、自动合规报告以及跨 Claude 使用的连续审计跟踪，因此企业可以检测风险行为、执行治理策略并自信地展示监管合规性。要设置此集成，请参阅**[Airia 设置指南](https://explore.airia.com/integrations/Tools/anthropic-compliance-api)**。

- **aizome：**aizome 发现并清点 Claude 在您的组织中的使用方式——用户、代理、项目、技能、MCP 服务器和连接器——浮现治理风险、生成审计报告，并决定将哪些代理载入具有范围权限、即时访问和完全可审计性的身份感知控制平面。了解更多关于**[aizome 与 Claude 合规 API 的集成](https://www.aizome.ai/integrations)**。

- **Akeyless Security：**Akeyless 为 Claude 代理提供无密钥、即时访问权限，以访问它们作用的系统——数据库、云服务和 SaaS API——消除了常驻凭证，并确保每个代理仅执行其允许的任务，具有每个操作的完整审计跟踪。要设置此集成，请参阅**[Akeyless Security 设置指南](https://www.akeyless.io/blog/securing-ai-agents-in-claude-demo-and-setup-guide/)**。

- **Akto：**Akto 的 Claude 合规 API 集成为安全团队提供了对 Claude Enterprise 聊天、文件、连接器和项目的可见性，跨 AI 代理、MCP 服务器和人类用户——检测提示注入、敏感数据泄露和策略违规。要设置此集成，请参阅**[Akto 设置指南](https://ai-security-docs.akto.io/akto-atlas-agentic-ai-security-for-employee-endpoints/endpoints-discovery-agents/anthropic-connector)**。

- **Alignmt AI：**Alignmt AI 将合规 API 活动转换为每个 AI 资源的风险评分清单，浮现自主自动化、可写连接器和敏感数据泄露，具有每个事实的来源和人类对证据无法解决的内容的证明。了解更多关于**[Alignmt AI 与 Claude 合规 API 的集成](https://www.alignmt.ai/post/alignmt-ai-integrates-with-anthropic-s-compliance-api)**。

- **Apiiro：**Apiiro 使用合规 API 活动源为安全团队提供对组织中活跃的 AI 编码代理、MCP 服务器、技能和插件的可见性，将该活动与 Apiiro 的软件供应链风险模型相关联，以浮现不安全或未授权的组件。了解更多关于**[Apiiro 与 Claude 合规 API 的集成](https://apiiro.com/blog/claude-compliance-api-now-in-the-apiiro-software-graph)**。

- **Artemis Security：**Artemis 摄取 Claude 合规 API 数据以检测和调查恶意行为，将 Claude 活动与环境的其余部分相关联，以实现机器速度的检测和响应。要设置此集成，请参阅**[Artemis 设置指南](https://artemissecurity.com/company-news/secure-your-enterprise-ai-artemis-now-integrates-with-anthropic-compliance-api-and-telemetry/)**。

- **Axonius：**Axonius 将 Claude Enterprise 引入 Axonius Asset Cloud，将目录和访问工件与堆栈中的所有其他资产和暴露进行协调。决策级资产智能浮现覆盖范围差距、漂移和环境中的身份风险，映射到资产和所有者。要设置此集成，请参阅**[Axonius 设置指南](https://docs.axonius.com/docs/anthropic)**。

- **Bay：**Bay 在企业中保护 AI 代理——为跨所有端点、容器和浏览器的 Claude 提供实时可见性和执行。要设置此集成，请参阅**[Bay 设置指南](https://bay.security/use-cases/platform/claude)**。

- **Beacon Security：**Beacon 是 AI 原生 SecOps 平台，充当安全团队的数据和上下文层。其 Claude 连接器摄取 Claude Enterprise 活动和合规数据，对其进行规范化和丰富，并使其可供安全代理、SIEM、数据湖和合规存档使用。了解更多关于**[Beacon 与 Claude 合规 API 的集成](https://beacon.security/beacon-for-anthropic-compliance-api)**。

- **BigID：**BigID 通过为安全团队提供对与 Claude 共享的每个文件的可见性来保护 Claude 使用，精确指出在上传风险文件时存在的敏感属性。了解更多关于**[BigID 与 Claude 合规 API 的集成](https://bigid.com/blog/securing-claude/)**。

- **Bloom Security：**Bloom 是 AI 原生时代的端点安全平台，为安全团队提供对组织中每个 AI 工具、插件和 MCP 服务器的可见性和治理。Claude 合规 API 集成深化了该覆盖范围——进入每个用户的每个会话、文件和配置更改——因此安全团队可以大规模启用 Claude 而不失去控制。要设置此集成，请参阅**[Bloom Security 设置指南](https://bloom.security/anthropic-compliance-api-integration)**。

- **Bluebear Security：**Bluebear 使用合规 API 持续检查 Claude Enterprise 组织的安全设置——包括 SSO 执行、受信任设备、权限绕过、网络出口、数据保留和连接器自动批准——与安全基线相比，当控制漂移时打开事件并提供补救步骤，并将 Claude 活动与 AI 编码代理在开发人员机器上执行的操作联系起来。要设置此集成，请参阅**[Bluebear Security 设置指南](https://bluebear.io/integrations/claude/)**。

- **Bold Security：**通过直接在端点上的实时保护，防止用户或 AI 代理滥用敏感数据。要设置此集成，请参阅**[Bold Security 设置指南](https://www.bold.security/blog/bold-security-extends-ai-data-protection-through-claudes-compliance-api)**。

- **Brava Security：**Brava 通过合规 API 摄取 Claude 活动源，并将其转换为充分利用的安全遥测——路由到您现有的 SIEM、数据湖和存档，并可搜索以进行调查——因此一个源为监控、内部风险、保留和治理提供动力，无需自定义收集器。要设置此集成，请参阅**[Brava Security 设置指南](https://brava.security/integrations/anthropic-claude)**。

- **C1.ai：**C1.ai 与 Claude 合规 API 集成，为 Claude Enterprise 和 Claude Platform 带来使用情况知情的访问审查、休眠账户检测和自动化生命周期工作流——扩展客户已在其堆栈其余部分运行的相同身份治理。要设置此集成，请参阅**[C1.ai 设置指南](https://www.c1.ai/docs/baton/claude-enterprise)**。

- **Capsule Security：**Capsule 连接到 Claude 合规 API，将您组织的成员、对话和活动转换为可审计的 AI 清单，为安全和合规团队提供持续可见性、数据泄露检测、恶意代理检测以及对 Claude 使用的审计就绪治理。要设置此集成，请参阅**[Capsule Security 设置指南](https://docs.capsulesecurity.io/guides/claude-enterprise)**。

- **Cato Networks：**Cato AI Security 与合规 API 集成，以保护、治理和审计所有 Claude 交互，执行实时策略、防止数据泄露并维护全面的合规可见性。要设置此集成，请参阅**[Cato Networks 设置指南](https://www.catonetworks.com/integrations/ai-activity-monitoring/anthropic-compliance-api/)**。

- **Check Point：**Check Point 的 Workforce AI Security 与 Claude 合规 API 的集成为企业提供了对跨表面和组织级 MCP 服务器的 Claude 使用的审计级可见性——每用户和每表面使用分析，加上跨聊天和上传文件的内容级暴露分析，标记 PII、凭证和受管制数据以用于现有安全工作流。要设置此集成，请参阅**[Check Point 设置指南](https://sc1.checkpoint.com/documents/Infinity_Portal/WebAdminGuides/EN/Workforce-AI-Security-Admin-Guide/Topics-Workforce-AI-Security-AG/Integrations.htm?tocpath=Integrations%7C_____0)**。

- **CloudEagle.ai：**CloudEagle.ai 与合规 API 集成，为 IT 和安全团队提供对谁使用 Claude 以及其访问权限是否最新的第一方可见性，将活动数据与使用和支出分析以及自动化配置和取消配置配对——因此 Claude 与所有其他 SaaS 和 AI 工具一起受到治理。要设置此集成，请参阅**[CloudEagle.ai 设置指南](https://www.cloudeagle.ai/app-integration/claude)**。

- **Cloudflare：**Cloudflare 的 CASB 集成为安全团队提供了对组织使用 Claude 的态势、数据和合规风险的无代理可见性。要设置此集成，请参阅**[Cloudflare 设置指南](https://developers.cloudflare.com/changelog/post/2026-05-19-casb-claude-compliance-api/)**。

- **Clover Security：**Clover 使用合规 API 为安全团队提供对 Claude Code 使用的组织范围可见性——谁在使用 Claude Code 构建、已有多少受 Clover 保护，以及差距在哪里——然后将 Clover 的安全控制扩展到每个开发人员。了解更多关于**[Clover Security 与 Claude 合规 API 的集成](https://clover.security/blog/see-what-your-coding-agents-build-clover-integrates-with-anthropics-claude-compliance-api-to-give-security-real-time-visibility/)**。

- **Concentric AI：**Concentric AI Semantic DLP 为组织提供对员工使用的 AI 工具（包括影子 AI）的可见性。通过与合规 API 的集成，它捕获每个提示、响应和附件——包括跨 Claude 浏览器和桌面应用的历史活动——以加强治理、加快调查并展示合规性。了解更多关于**[Concentric AI 与 Claude 合规 API 的集成](https://concentric.ai/integrations/anthropic-compliance/)**。

- **Cribl：**Claude 合规 API 允许您通过 HTTPS 将来自 Claude Enterprise 和 Claude Platform 的审计活动摄取到 Cribl Stream 中，作为一流的遥测流。它提取用户活动，包括身份验证、聊天交互、文件上传和管理操作，因此您可以规范化、丰富、掩盖和路由该数据以及您的其余可观测性和安全管道。要设置此集成，请参阅**[Cribl 设置指南](https://docs.cribl.io/stream/sources-anthropic-compliance/)**。

- **CrowdStrike：**CrowdStrike 与 Claude 合规 API 构建了一个集成，将 Claude 活动引入 Falcon 平台，通过 Falcon Next-Gen SIEM 和 CrowdStrike 的代理 SOAR 提供实时可见性、检测和自动化响应。它将进一步通过 Falcon AIDR 和 Falcon Shield 扩展保护，以持续检测、治理和大规模保护 AI 使用。要设置此集成，请参阅**[CrowdStrike 设置指南](https://marketplace.crowdstrike.com/listings/anthropic-data-connector/)**。

- **Cyberhaven：**Cyberhaven 与合规 API 集成，为企业安全团队提供对员工与 Claude 共享的数据的可见性，将 Cyberhaven 的数据分类引擎应用于聊天、文件和项目，因此客户可以自信地采用 Claude，同时满足其安全和合规要求。要设置此集成，请参阅**[Cyberhaven 设置指南](https://www.cyberhaven.com/support/anthropic-claude-enterprise-connector)**。

- **Cyera:** Cyera 通过 Claude Compliance API 与 Claude Enterprise 集成，为安全团队提供对对话、文件和平台用户活动的全面可见性。每次交互都通过 OmniDLP 进行分析，并通过 Cyera 的 95% 精度数据分类和风险评分进行增强，帮助团队识别敏感数据泄露、检测风险或未经授权的行为，并从第一天起维护审计就绪的监督。部署只需几分钟。**[了解有关集成的更多信息](https://www.cyera.com/blog/cyera-and-anthropic-integrate-to-bring-real-time-ai-security-to-claude-enterprise)**并联系您的 Cyera 销售代表以开始使用。

- **Dash Security:** Dash Security 对 Claude Enterprise 对话提供实时、意图感知的监控和策略执行，检测敏感数据泄露、提示注入和风险行为。通过提取对话意图，Dash 了解用户实际上在用 AI 做什么，并区分真实威胁、误报和良性活动。要设置此集成，请参阅 **[Dash Security 设置指南](https://dash.security/integrations/claude-enterprise)**。

- **Datadog:** Datadog 的 Claude Compliance 将来自 Claude Platform 的审计日志摄取到 Datadog 中，为安全团队提供对其 Claude 组织中的管理员活动、API 密钥生命周期事件和身份验证事件的可见性，用于 SIEM 和合规用例。要设置此集成，请参阅 **[Datadog 设置指南](https://docs.datadoghq.com/integrations/anthropic-compliance-logs/)**。

- **Daylight Security:** 监控组织中的 Claude 使用情况以进行安全调查和合规目的。要设置此集成，请参阅 **[Daylight Security 设置指南](https://daylight.ai/claude-enterprise-detection)**。

- **Elastic:** 收集 Claude 审计和使用活动以检测安全风险、支持合规要求并调查与 AI 相关的事件。要设置此集成，请参阅 **[Elastic 设置指南](https://www.elastic.co/docs/reference/integrations/anthropic)**。

- **Endor Labs:** Endor Labs AI Governance 使用 Compliance API 提取 Claude Enterprise 用户、角色和组，将 Claude Code 活动归属于真实开发者身份，以便安全团队可以跨其 AI 编码代理应用角色和组感知的护栏。要设置此集成，请参阅 **[Endor Labs 设置指南](https://docs.endorlabs.com/agent-governance/claude-code/compliance-api)**。

- **Eon:** Eon 通过自动数据分类和自然语言安全查询为 Claude 提供 AI 治理，帮助企业保持对与 AI 工具共享的敏感数据的可见性和控制。了解更多关于 **[Eon 与 Claude Compliance API 的集成](https://www.eon.io/integrations/claude-compliance-api)**。

- **eSentire:** eSentire Atlas 与 Compliance API 集成，为 Claude 使用提供 24/7 托管检测和响应。eSentire 摄取活动事件和对话内容，将其与端点、身份和云遥测关联，并代表客户调查和响应威胁和策略违规。了解更多关于 **[eSentire 与 Claude Compliance API 的集成](https://www.esentire.com/what-we-do/mdr-and-platform-integrations)**。

- **Evoke Security:** Evoke Security 为 AI 代理提供运行时威胁检测和响应，涵盖 Claude Code 和 Cowork。其 Compliance API 集成将该覆盖范围扩展到 Claude Enterprise 对话层，将聊天、文件、项目和活动事件与 Evoke 检测和响应平台中的端点遥测相结合。要设置此集成，请参阅 **[Evoke Security 设置指南](https://www.evokesecurity.com/blogs/evoke-anthropic-compliance-api)**。

- **Exaforce:** Exaforce 将 Claude Compliance API 活动、聊天和文件数据摄取到 Exaforce Agentic SOC Platform 中，为安全团队提供 Claude 使用的受管、SOC 就绪视图，以监控、检测和调查风险活动以及更广泛的安全信号。要设置此集成，请参阅 **[Exaforce 设置指南](https://www.exaforce.com/blogs/anthropic-integration)**。

- **ForceAI Security:** ForceAI PineSMITH 与 Anthropic Claude Compliance API 集成，为 Claude 活动、用户、聊天、文件、项目、治理事件和审计就绪的合规数据提供企业可见性。要设置此集成，请参阅 **[ForceAI Security 设置指南](https://forceaisecurity.com/documents/claude-compliance-integration)**。

- **Forcepoint:** Forcepoint 安全启用 AI 的解决方案现已通过 Claude Compliance API 扩展到 Claude Enterprise，将每次对话、文件上传、生成的内容和用户活动摄取到统一的 AI 治理仪表板中。安全团队可以跨所有 Claude 交互应用现有的 DLP 和 DSPM 策略，检测敏感数据泄露、执行合规性并维护完整的审计跟踪，在部署连接器时立即回填历史活动。要设置此集成，请参阅 **[Forcepoint 设置指南](https://www.forcepoint.com/use-case/securely-enable-ai)**。

- **Forever:** Forever 使用 Compliance API 将 Claude Enterprise 对话和活动日志纳入其代理活动记录中，为安全团队提供对 Claude 代理和聊天正在做什么的可见性，以便他们可以应用其安全标准并调查风险操作。了解更多关于 **[Forever 与 Claude Compliance API 的集成](https://forever.security/blog/forever-claude-enterprise-integration/)**。

- **Fortinet:** Fortinet 数据安全态势管理与 Claude Compliance API 的集成使组织能够监控和分析 Claude Enterprise 中的用户交互，包括提示和上传的文件，以检测 PII、PHI、CHD 和身份验证数据等敏感数据的潜在泄露。该集成为 AI 使用提供可见性、警报和可审计性，帮助安全团队加强企业 AI 工作流的治理、合规性和数据保护。要设置此集成，请参阅 **[Fortinet 设置指南](https://www.fortinet.com/content/dam/fortinet/assets/alliances/fortidata-claude-setup-guide.pdf)**。

- **Geordie AI:** Geordie 是代理安全和治理平台，通过 Claude Compliance API 将其对 Claude 套件（包括 Claude Code 和 Claude Cowork）的现有覆盖范围扩展到整个 Claude Enterprise 部署表面。这为安全和合规团队提供了对企业中每个 Claude 用例和活动的单一、一致的视图，以及对其进行治理的可见性和控制。要设置此集成，请参阅 **[Geordie AI 设置指南](https://www.geordie.ai/resources/geordie-extends-agentic-ai-security-and-governance-coverage-through-claude-compliance-api/)**。

- **Global Relay:** Global Relay 使用 Compliance API 在合规存档中捕获 Claude Enterprise 对话、文件、工件和项目，支持 SEC Rule 17a-4 和 FINRA 规则等法规下的电子发现、监督和保留要求。要设置此集成，请参阅 **[Global Relay 设置指南](https://www.globalrelay.com/connector/claude/)**。

- **Glow:** Glow 摄取 Compliance API 活动源以完善其对组织 Claude 工作区中 AI 采用和安全态势的了解，在从个人用户到托管部署和代理集成的每种 Claude 采用形式中实现一致的治理和安全控制。要设置此集成，请参阅 **[Glow 设置指南](https://docs.glow.io/integrations/anthropic)**。

- **Grip Security:** Grip 与 Anthropic Compliance API 集成，为 Claude 提供 SSPM 检查、补救和 AI 代理可见性。要设置此集成，请参阅 **[Grip Security 设置指南](https://help.grip.security/docs/anthropic-claude-compliance-api-integration-with-grip-security-1)**。

- **Hadrius:** Hadrius 是 SEC 和 FINRA 监管公司的代理合规基础设施。其 Claude 集成将对话和活动捕获到具有自动监督的合规账簿和记录存档中，以便顾问和经纪交易商可以采用 Claude，同时满足其记录保留和监督义务。要设置此集成，请参阅 **[Hadrius 设置指南](https://support.hadrius.com/articles/8197408781-claude-integration?lang=en)**。

- **Harmonic Security:** Harmonic Security 使用 Anthropic Compliance API 在每个表面（桌面、网络、移动、Claude Code、Cowork）上管理 Claude 使用，使用专有检测模型对每次交互进行分类，以显示敏感数据泄露、提示注入以及按团队、部门和位置分解的 AI 使用和采用见解。要设置此集成，请参阅 **[Harmonic Security 设置指南](https://docs.harmonicsecurity.app/integration-guides/configure-anthropic-compliance-api-connector)**。

- **IBM Guardium:** IBM Guardium 帮助组织监控代理 AI 系统的数据访问和使用，并帮助确保自主活动保持在批准的安全边界内。通过现成的模板、可自定义的策略、持续的审计工作流和与 EU AI Act、AIDA、PCI、SOX、HIPAA、GDPR 和 CPRA 等框架一致的合规报告，Guardium 帮助安全和合规团队自信地展示对 AI 驱动活动的监督。了解更多关于 **[IBM Guardium](https://www.ibm.com/products/guardium)**。

- **Island:** Island 的 Claude Compliance API 集成为安全和合规团队提供对每个 Claude 会话（用户驱动或自主）的可见性，涵盖对话主题、工具/模型/技能使用，以及对敏感数据泄露和策略违规的持续丰富检测，用于态势评估和异常检测。了解更多关于 **[Island 与 Claude Compliance API 的集成](https://www.island.io/blog/island-integrates-with-the-claude-compliance-api)**。

- **Kapalins:** Kapalins 使用 Compliance API 将 Claude Enterprise 活动纳入与组织使用的所有其他 AI 工具的同一时间线中，将每个事件归属于用户，并使用相同的策略引擎和检测规则对其进行评估，以提出由证据链支持的事件。要设置此集成，请参阅 **[Kapalins 设置指南](https://kapalins.com/integrations/claude-enterprise/)**。

- **KnowBe4:** KnowBe4 Agent Risk Manager 使用 Compliance API 来清点 AI 代理和 Claude 使用情况，将活动映射回其人类、系统或代理源，并检测风险，如提示注入、敏感信息泄露、过度代理和无限消费——无需部署任何设备上软件。 可用性说明：计划于 2026 年 11 月正式推出。要设置此集成，请参阅 **[KnowBe4 设置指南](https://support.knowbe4.com/hc/en-us/articles/55662933177875-Claude-Compliance-Integration-Guide-for-Agent-Risk-Manager)**。

- **Lasso Security:** Lasso 与 Compliance API 集成，为安全团队提供对 Claude 活动的集中可见性——谁在使用 Claude、他们在做什么以及他们调用哪些 MCP 服务器和技能——根据客户定义的策略对每次交互进行分类，并提供完整的审计跟踪以实现合规性。要设置此集成，请参阅 **[Lasso Security 设置指南](https://www.lasso.security/use-cases/claude-enterprise)**。

- **Linx Security:** Linx 与 Claude Compliance API 集成，以在 Claude 中跨人类、非人类和代理身份管理访问——提供可见性、风险检测、访问审查和最小权限执行。要设置此集成，请参阅 **[Linx Security 设置指南](https://docs.linxsecurity.io/connectors/connectors-library/anthropic/claude.ai)**。

- **Lumia Security:** Lumia Security 使用 Compliance API 根据用自然语言编写的组织特定策略分析 Claude Enterprise 对话、文件和会话记录，检测敏感数据泄露、风险行为和策略违规，以支持安全调查和合规监督。了解更多关于 **[Lumia Security 与 Claude Compliance API 的集成](https://www.lumia.security/blog/claude-compliance-api)**。

  - **可用性说明：**此集成处于测试版，计划于 2026 年 12 月正式推出。

- **Microsoft Purview:** Purview 与 Claude Compliance API 的集成通过显示来自 Claude 活动源和聊天对话的见解来扩展 Purview 的企业安全和合规覆盖范围，在 Purview Data Security Posture Management 和 Audit 中启用场景。它提供对 Claude Enterprise 和 Claude Platform 中活动的集中可见性，使组织能够了解数据如何在其 Claude 系统中被访问和使用。在 Purview 中显示此信号有助于关闭一个关键的治理差距，因为企业正在跨多个平台快速采用 AI。Purview 为 Copilot 和更广泛的 Microsoft AI 生态系统提供的相同全面可见性现已可用于 Claude Enterprise 用户。要设置此集成，请参阅 **[Microsoft Purview 设置指南](https://learn.microsoft.com/en-us/purview/ai-microsoft-purview)**。

- **Mimecast:** Mimecast 的 Governance、Compliance & Insights (GCI) 平台现已与 Claude Compliance API 集成，将 Claude Enterprise 对话纳入组织已用于电子邮件和协作数据的相同统一治理和合规体验中，无需新的界面或工作流。要设置此集成，请参阅 **[Mimecast 设置指南](https://www.mimecast.com/use-cases/ai-governance/)**。

- **Mind Security Inc:** MIND 与 Compliance API 集成，以发现整个组织中的 Claude 使用情况，将其与用户身份关联，并对聊天和文件中共享的数据进行分类，以便安全团队可以检测 Claude 中的数据风险以及 SaaS、端点和 GenAI 活动。了解更多关于 **[Mind Security Inc 与 Claude Compliance API 的集成](https://mind.io/blog/claude-compliance-governance)**。

- **Mint Security:** Mint 为 AI 代理提供上下文运行时安全，使用 Claude Compliance API 对团队如何使用 Claude 应用组织特定的可见性、上下文和策略控制。要设置此集成，请参阅 **[Mint Security 设置指南](https://docs.mint.security/integrations/claude/claude)**。

- **Neo Security:** Neo 使用 Compliance API 将 Claude Enterprise 对话内容和审计日志事件纳入其控制台，为安全团队提供对 Claude、Claude Code 和 Cowork 如何与 Neo 的 AI 代理、MCP 服务器和扩展的端点视图一起使用的用户归属视图。要设置此集成，请参阅 **[Neo Security 设置指南](https://www.neo.ai/integrations/claude-enterprise/)**。

- **Netskope:** Netskope 与 Claude Compliance API 的集成使组织能够将 Claude Enterprise 直接连接到 Netskope One 平台的增强 AI 安全功能中，在安全和合规团队已依赖的工作流中提供完整的资产、身份和活动可见性、强大的策略执行、高级数据安全和全面的安全态势管理。**说明：**在私有预览中可用。要设置此集成，请参阅 **[Netskope 设置指南](https://docs.netskope.com/en/onboard-claude-for-ai-discovery)**。

- **Nightfall AI:** Nightfall 与 Claude Compliance API 集成，以检测 Claude Enterprise 聊天和文件上传中的敏感数据（PII、PHI、PCI、机密、源代码）。仅监控，具有管理员警报、最终用户通知和与其他 SaaS 连接器相同工作流中的审计跟踪。要设置此集成，请参阅 **[Nightfall AI 设置指南](https://help.nightfall.ai/nightfall-for-anthropic-claude/nightfall-for-claude-enterprise-compliance-api)**。

- **Noma Security:** Noma 的 AI Detection and Response (AI-DR) 与 Compliance API 的集成为安全团队提供对 Claude Enterprise 使用的持续可见性，检测敏感数据泄露、风险提示和策略违规，具有完整的行为上下文——因此团队可以在实施治理的情况下对 Claude 说

- **Obsidian Security:** Obsidian 从 Compliance API 和 200 多个其他应用程序中提取态势和活动。该数据被规范化并由企业客户用于持续治理、漂移检测以及识别和调查潜在威胁。了解更多关于 **[Obsidian Security 与 Claude Compliance API 的集成](https://www.obsidiansecurity.com/integrations/claude)**。

- **Okta:** Okta 与 Claude Compliance API 的集成将 Claude 使用纳入 Okta Identity Security Posture Management，使安全和合规团队能够将 Claude 用户活动与来自其他身份平台、SaaS 工具和云提供商的信号进行上下文关联，并从统一视图中显示身份盲点。这使组织能够持续评估身份风险态势、检测错误配置和风险（如权限过高的用户、休眠账户和未使用的管理员密钥）、按严重程度补救问题，并安全地大规模部署 AI。了解更多关于 **[Okta 与 Claude Compliance API 的集成](https://www.okta.com/newsroom/articles/okta-identity-security-posture-management-integrates-with-anthropics-new-compliance-api/)**。

  - **可用性说明：**此产品将很快以测试版形式提供给使用 Okta Identity Security Posture Management 以及 Anthropic 的 Claude Enterprise 或 Claude Platform 的选定客户。

- **Onyx Security:** Onyx 是安全 AI 控制平面，为企业提供对 SaaS、云、端点和代码中的 AI 代理的可观测性、安全性和治理。此集成将 Onyx 的护栏和审计跟踪扩展到 Claude 部署——让安全团队监控使用情况、执行策略、防止数据泄露并满足合规要求，而不会减缓 Claude 采用。要设置此集成，请参阅 **[Onyx Security 设置指南](https://onyx.security/integrations/anthropic)**。

- **Opal Security:** Opal 使用 Compliance API 向安全和 IAM 团队展示其组织如何使用 Claude——包括 MCP 服务器连接和 API 密钥——并通过调整连接系统中的权限来收紧 Claude 代理可以访问的内容，在过度宽泛的访问成为事件之前。要设置此集成，请参阅 **[Opal Security 设置指南](https://docs.opal.dev/docs/agent-inventory)**。

- **Opsin:** Claude Enterprise 的 AI 风险和代理治理：Opsin 的 Compliance API 集成在 24 小时内显示数据过度共享、风险用户行为和代理蔓延，使用连接身份、数据和模型的上下文图进行根本原因补救。要设置此集成，请参阅 **[Opsin 设置指南](https://www.opsinsecurity.com/claude-integration)**。

- **Orca Security:** Orca Security 将其统一平台扩展到 Claude Enterprise，为安全团队提供对 Claude 身份、访问、AI 工作区和其中敏感数据的完整可见性——以显示风险、执行最小权限并跟踪合规性。要设置此集成，请参阅 **[Orca Security 设置指南](https://claude-doc.orca.security/)**。

- **Origin:** Origin 是代理 AI 的端点可观测性：您的每个 AI 代理、它做了什么以及您的 AI 支出去向。Compliance API 添加了来自 claude.ai 和 Claude Desktop 的 Claude Enterprise 使用情况，归属于每个用户并折叠到 Origin 观察的代理的相同记录中。要设置此集成，请参阅 **[Origin 设置指南](https://support.originhq.com/docs/claude-enterprise)**。

- **Palo Alto Networks:** Claude Compliance API 集成使组织能够了解与 Claude 共享的数据、检测敏感内容和策略违规、识别威胁和异常行为，并指导用户采取更安全的 AI 使用方式。要设置此集成，请参阅 **[Palo Alto Networks 设置指南](https://www.paloaltonetworks.com/blog/cloud-security/claude-security-integration-ai-governance)**。

- **Pi Security:** Pi Security 使用 Compliance API 了解工程团队如何使用 Claude 构建——他们委派的任务、他们遵循的工作流以及安全相关决策发生的地方——因此其代理应用安全指导反映真实的开发实践并更早地捕获风险在 SDLC 中。了解更多关于 **[Pi Security 与 Claude Compliance API 的集成](https://www.pi.security/blog/powering-every-builder-with-security-context-pi-integrates-with-anthropics-compliance-api)**。

- **Pluto Security:** Pluto 在整个 Anthropic 生态系统（Claude Code、Cowork 和 Office Agents）中提供实时可见性和执行。它监控代理活动、检测提示注入和敏感数据泄露等风险、识别恶意生态系统组件，并对代理操作、MCP、连接器和工具使用执行策略。了解更多关于 **[Pluto Security 与 Claude Compliance API 的集成](https://pluto.security/blog/claude-enterprise-meets-ai-security-platform/)**。

- **Proofpoint:** Proofpoint 与 Claude Compliance API 的集成将安全和治理结合在一起，跨 AI 交互。提示、响应和文件中的敏感数据通过 DLP、AI 感知分类器和内部威胁信号进行保护，而活动被捕获并保留用于电子发现、监督和合规目的。结果是对利用 Claude 的组织的全面可见性、控制和保留，与保护其其余技术堆栈的控制一致。要设置此集成，请参阅 **[Proofpoint 设置指南](https://support.nuclei.ai/hc/en-us/articles/39717913412759-Compliance-Archiving-for-Anthropic-Claude-Enterprise)**。

- **Radware:** Radware Agentic AI Protection 与 Compliance API 集成，以检查 Claude Enterprise 聊天交互中的敏感数据泄露、OWASP LLM 和代理 AI 违规以及其他代理安全威胁，帮助安全团队评估代理和工具的态势并优先考虑风险。了解更多关于 **[Radware 与 Claude Compliance API 的集成](https://www.radware.com/blog/agentic-ai-protection-for-claude-enterprise/)**。

- **Rapid7:** Rapid7 Surface Command 使用 Compliance API 将 Claude Enterprise 组织、用户、角色、组和设置纳入其统一的资产和身份清单中，将每个 Claude 账户与身份提供商、端点和云数据关联，以便安全团队可以看到谁有权访问 Claude 以及特权访问在哪里造成风险。要设置此集成，请参阅 **[Rapid7 设置指南](https://extensions.rapid7.com/extension/anthropic_app)**。

- **Reco:** Reco 监控 Claude Enterprise 以防范 AI 安全威胁、态势风险和身份异常。通过 Compliance API，它摄取活动日志和对话数据，以检测 MCP 服务器滥用、权限提升和未授权访问，并将违规行为映射到 MITRE ATT&CK、OWASP LLM Top 10 等。详细了解 **[Reco 与 Claude Compliance API 的集成](https://www.reco.ai/blog/claude-security-governance)**。

- **RelativityOne:** RelativityOne 使法律、合规和安全团队能够通过 Claude Compliance API 直接从源头收集 Claude Enterprise 数据。这包括对话提示和响应数据（如聊天、上传的文件和项目）。收集的数据会自动映射、保留并处理为可审查的格式，包括规范化为 Relativity Short Message Format (RSMF)。这使团队能够快速分析对话证据并保持完整上下文，同时维护防御所需的元数据和透明度。要设置此集成，请参阅 **[RelativityOne 设置指南](https://help.relativity.com/RelativityOne/Content/Collect/Data_sources/Claude.htm)**。

- **ReliaQuest:** ReliaQuest GreyMatter 平台现已与 Claude Compliance API 集成，将 Anthropic 的 Claude Enterprise 活动数据引入 GreyMatter，为安全和 IT 团队提供活动日志的可见性，以检测威胁或不需要的行为。要设置此集成，请参阅 **[ReliaQuest 设置指南](https://greymatter.myreliaquest.com/resources/articles/anthropic-claude-compliance)**。

- **Reva.ai:** Reva 为 Claude Code 和 Claude Enterprise 代理提供运行时授权，通过预工具钩子和推理钩子在操作时评估策略，并根据代理的目的和正常模式检查每个请求，以捕捉仅权限检查无法发现的漂移和滥用。详细了解 **[Reva.ai 与 Claude Compliance API 的集成](https://codingagent.dev.reva.ai/register)**。

- **Rubrik:** Rubrik Agent Cloud 与 Claude Compliance API 集成，帮助组织控制 Claude 在工作中的使用方式。团队可以查看谁在使用 Claude、在需要时审查活动、标记风险行为，并使用 Rubrik SAGE 创建自然语言策略，将警报路由到其安全工具。要设置此集成，请参阅 **[Rubrik 设置指南](https://www.rubrik.com/products/rubrik-agent-cloud)**。

- **Safeguard:** Safeguard 与 Claude Compliance API 集成，以监控 Claude 使用情况，作为 AI 原生软件供应链安全的一部分，通过 AI 驱动的补救措施在整个生命周期中浮现漏洞和风险。要设置此集成，请参阅 **[Safeguard 设置指南](https://safeguard.sh/mcp/claude)**。

- **SailPoint:** SailPoint Claude Compliance API 集成通过聚合组织用户、组和角色来简化身份和访问管理。要设置此集成，请参阅 **[SailPoint 设置指南](https://documentation.sailpoint.com/connectors/saas/claude_enterprise/help/)**。

- **Salt Security:** Salt Security 的 Claude Compliance API 集成为安全团队提供了连接到 Claude Enterprise 的每个 MCP 服务器的连续、权威清单，通过单个只读范围跟踪已添加、已删除和活跃的服务器，该范围永远不会接触提示或对话内容。要设置此集成，请参阅 **[Salt Security 设置指南](https://salt.security/blog/introducing-the-salt-claude-connect-continuous-mcp-server-visibility-for-claude-enterprise)**。

- **Saviynt:** Saviynt Claude Compliance API 集成通过聚合用户、组和角色来简化 Claude Enterprise 的身份治理，以实现集中可见性、自动化访问认证和无缝生命周期管理。要设置此集成，请参阅 **[Saviynt 设置指南](https://exchange.saviynt.com/products/claude-enterprise)**。

- **SentinelOne:** 通过 SentinelOne 的 Anthropic Compliance API 集成，为安全团队提供可见性和控制，以在整个企业中采用 Claude。Prompt Security 根据您的策略扫描每个提示和响应，实时标记违规。Singularity AI SIEM 在您的安全遥测中关联 Claude 活动，以加快调查、检测和响应。详细了解 **[SentinelOne 与 Claude Compliance API 的集成](https://sentinelone.com/blog/sentinelone-and-claude-integrations-for-ai-visibility-governance-and-defense)**。

- **Sentra:** 发现项目、聊天、提示和文件中的敏感数据，识别数据泄露风险，并将数据安全和合规控制扩展到您的 AI 环境。要设置此集成，请参阅 **[Sentra 设置指南](https://sentra.io/guides/sentra-claude-enterprise-integration-guide)**。

- **Shield:** Shield 使用 Compliance API 将 Claude Enterprise 聊天、文件、项目和工件捕获到合规的长期存储中，通过行为分析和基于 NLP 的风险检测进行监控，为受监管的金融机构提供针对其 Claude 使用情况的审查就绪的记录保留和电子发现层。要设置此集成，请参阅 **[Shield 设置指南](https://hs.shieldfc.com/claude-connector)**。

- **Silverfort:** Silverfort 使用 Compliance API 将 Claude 身份和活动映射到其统一身份图中，帮助安全团队调查访问并浮现风险，如休眠特权用户、不完整的离职、SSO 绕过、公共工件、暴露的隧道密钥和不受管理的 MCP 服务器。详细了解 **[Silverfort 与 Claude Compliance API 的集成](https://www.silverfort.com/blog/silverfort-integration-with-anthropic-compliance-api/)**。

- **Smarsh:** Smarsh Capture 帮助组织监控和保留 Claude Enterprise 交互，以便您可以自信地采用和治理 AI，并支持合规工作流。通过直接连接到 Claude Compliance API，您可以获得对 Claude 对话的结构化访问、完整的一致记录，以及对审计和监管响应的支持。这使公司范围内的 Claude Enterprise 采用和治理成为可能，同时领先不断发展的法规。要设置此集成，请参阅 **[Smarsh 设置指南](https://central.smarsh.com/s/article/From-which-networks-does-cloudcapture-gather-data-from)**。

- **Snyk:** 这项新集成为安全和合规团队提供了其 Claude Enterprise 环境的完整清单，包括模型、已批准的 MCP 服务器和工具级权限，都在他们已用于治理堆栈其余部分的平台中。要设置此集成，请参阅 **[Snyk 设置指南](https://docs.snyk.io/integrations/cloud-ai-platforms/anthropic-claude-enterprise)**。

- **Sola Security:** Sola 连接到 Compliance API，将 Claude Enterprise 活动、用户访问和对话数据引入 Sola 的 AI 驱动安全平台，安全团队可以用纯语言查询其 Claude 环境，将其与身份工具关联，并浮现治理风险，如权限过高的用户和孤立账户。要设置此集成，请参阅 **[Sola Security 设置指南](https://docs.sola.security/integrations/data-sources/claude-enterprise)**。

- **Splunk，思科公司：** Splunk 通过 Compliance API 和相关 Claude API 摄取 Claude Enterprise 审计、使用、成本和支出限制数据，具有 CIM 规范化事件、现成检测和仪表板，用于安全审计、治理以及使用和支出分析。要设置此集成，请参阅 **[Splunk 设置指南](https://lantern.splunk.com/Security_Use_Cases/Compliance/Monitoring_and_governing_enterprise_AI_platforms)**。

- **Strac:** Strac 通过使用 Compliance API 检测敏感数据泄露、监控使用情况和生成审查就绪的安全见解，帮助企业安全地采用 Claude。要设置此集成，请参阅 **[Strac 设置指南](https://www.strac.io/integrations/claude-compliance-api)**。

- **Straiker:** Straiker 为 Claude Enterprise 提供实时安全，检测 Claude 聊天和 Claude Code 中的提示注入、数据泄露和代理滥用。安全团队在单个平台中获得所有 Claude 表面的统一可见性、执行和取证调查。详细了解 **[Straiker 与 Claude Compliance API 的集成](https://www.straiker.ai/claude-code)**。

- **Sumo Logic:** Sumo Logic 与 Claude Compliance API 的集成将 Claude Enterprise 活动日志和 Claude 平台活动日志引入 Sumo Logic Intelligence Operations Platform，使团队能够将 AI 治理转变为透明的、企业就绪的操作，通过 Sumo Logic 内的实时监控和自动化检测、调查和响应功能来降低风险。要设置此集成，请参阅 **[Sumo Logic 设置指南](https://www.sumologic.com/help/docs/integrations/saas-cloud/claude-compliance)**。

- **Sun Security:** Sun Security 使用 Compliance API 发现和清点在 Claude、Claude Code 和 Cowork 中使用的代理、MCP 服务器、技能和插件，通过端点、SaaS、云和身份上下文丰富该活动，以浮现敏感数据泄露和风险代理行为。要设置此集成，请参阅 **[Sun Security 设置指南](https://www.sun.security/integrations/claude-enterprise)**。

- **Surf AI:** Surf 从 Compliance API 读取 Claude 使用情况，以检测未经批准的 MCP 连接，将每个连接与可问责的所有者关联，并推动补救。详细了解 **[Surf AI 与 Claude Compliance API 的集成](https://www.surf.ai/technology-partners/anthropic)**。

- **Sysdig:** Sysdig 检测可疑的 Claude 凭证活动，并将其与同一机器上的运行时和安全事件关联，将孤立的 API 警报转变为明确的滥用证据，使安全团队能够以完整上下文响应真实威胁。要设置此集成，请参阅 **[Sysdig 设置指南](https://www.sysdig.com/integrations/anthropic)**。

- **Teleskope:** Teleskope 使用 Compliance API 扫描 Claude Enterprise 对话和文件中已存在的敏感数据，帮助安全团队查找和调查泄露并执行其数据安全策略。要设置此集成，请参阅 **[Teleskope 设置指南](https://docs.teleskope.ai/connectors/saas/ai/claude)**。

- **Tenable:** 与 Claude Compliance API 的集成将利用对用户交互的可见性来检测恶意和可疑活动。此功能使组织能够在整个组织中安全地采用 Claude Enterprise，而不会失去对用户如何与其交互的可见性。要设置此集成，请参阅 **[Tenable 设置指南](https://docs.tenable.com/early-access/ai-exposure/Content/settings/integrations/claude.htm)**。

- **Theta Lake:** 当企业在整个员工队伍中部署 Claude 以大规模解锁生产力和投资回报率时，AI 和人类之间以及 AI 代理之间的 AI 交互正在引起全新的用户行为，并产生新类别的通信。安全、合规和法律团队需要完全了解 Claude 的使用方式。此集成使组织能够使用 Theta Lake 的调查、分析和监管合规功能，其中每一项都在 Gartner® 关键功能中获得了最高分。要设置此集成，请参阅 **[Theta Lake 设置指南](https://docs.thetalake.com/docs/en-US/integrations/anthropic.html?Highlight=anthropic%20)**。

- **Token Security:** Token Security 与 Anthropic 的 Compliance API 集成，以发现、治理和保护为 Claude 使用情况提供动力的非人类身份，在统一的 NHI 安全平台内浮现陈旧密钥、所有权差距和代理活动。详细了解 **[Token Security 与 Claude Compliance API 的集成](https://www.token.security/blog/token-security-brings-anthropic-claude-compliance-api-into-its-identity-control-plane)**。

- **Torch Security:** Torch 将 Claude Enterprise 纳入身份治理：它通过 Compliance API 读取 Claude 组织目录和活动源，将每个账户关联到真实身份，并运行访问审查，标记带外（非 SCIM）组授予、孤立账户和休眠访问，只读，永远不会接触对话内容。要设置此集成，请参阅 **[Torch Security 设置指南](https://www.torch.security/integrations/claude/)**。

- **Trellix:** Trellix Claude Compliance API 集成使用其数据安全引擎根据客户定义的隐私策略检查所有聊天内容、工件和响应，并在 Trellix Helix 中创建检测和审计事件。这包括为其端点、服务器和协作软件定义的客户特定模式和策略，并将其应用于 Claude。要设置此集成，请参阅 **[Trellix 设置指南](https://www.trellix.com/platform/data-security/)**。

- **TrendAI:** TrendAI 与 Claude Compliance API 的集成自动收集和监控整个组织的 Claude 聊天活动，扫描提示注入、有害内容和敏感数据泄露，并通过 SIEM 和 XDR 将发现与更广泛的安全信号关联，用于调查和审查就绪的合规和治理。要设置此集成，请参阅 **[TrendAI 设置指南](https://www.trendmicro.com/en_us/research/26/f/governing-claude-enterprise.html)**。

- **Twine Security:** Twine 的 AI 数字员工 Alex 使用 Compliance API 为 Claude Enterprise 带来身份治理，映射谁有访问权限、他们拥有哪些角色、组和连接器，并标记风险，如在身份提供商之外配置的账户，并将补救路由以供人工批准。要设置此集成，请参阅 **[Twine Security 设置指南](https://www.twinesecurity.com/resources/twine-anthropic-governing-claude-enterprise)**。

- **Valence Security:** Valence 与 Compliance API 集成，为安全团队提供对 Claude Enterprise 和 Claude Platform 使用方式的可见性，包括用户、代理、MCP 服务器、连接器和工具。Valence 将此活动与环境其余部分的身份和配置关联，帮助团队检测风险行为和补救泄露。要设置此集成，请参阅 **[Valence Security 设置指南](https://www.valencesecurity.com/resources/blogs/securing-agentic-identities-in-claude-the-valence-integration-with-the-compliance-api)**。

- **Vanta:** Vanta 使用 Compliance API 在其 AI 治理产品中从 Claude Enterprise 聊天和 Claude Code 和 Cowork 会话构建 AI 清单，使合规团队能够监控 Claude 使用情况并针对其策略和控制生成审查就绪的报告。可用性说明：此集成以测试版形式提供给使用 Claude Enterprise 的选定 Vanta AI 治理客户，计划于 2027 年 1 月正式推出。要设置此集成，请参阅 **[Vanta 设置指南](https://www.vanta.com/integrations/claudes-compliance-api)**。

- **Varonis:** Compliance API 集成深化了 Varonis 对 Claude 的支持，使安全和治理团队能够监控使用情况、调查整个会话中的滥用，并通过数据上下文评估 AI 相关风险。要设置此集成，请参阅 **[Varonis 设置指南](https://info.varonis.com/hubfs/documentation/varonis-atlas-anthropic-compliance-api-documentation.pdf)**。

- **Vega:** Vega 摄取您组织的 Compliance API 活动源，将其规范化为通用安全架构，并让您的团队搜索、检测、警报和调查 Claude 审计事件以及其余安全遥测。详细了解 **[Vega 与 Claude Compliance API 的集成](https://vega.io/blog/vega-anthropic-claude-compliance-api)**。

- **Venice:** Venice 使用 Compliance API 发现 Claude Enterprise 用户并将 AI 代理与其所有者的身份关联，对每个代理的爆炸半径进行评分并标记常设权限，以便安全团队可以将其移至即时访问。详细了解 **[Venice 与 Claude Compliance API 的集成](https://www.venice.io/blog/claude-compliance-api)**。

- **Vero Security:** Vero Security 与 Compliance API 集成，为企业提供安全见解，为治理和策略执行提供完整上下文，使团队能够监控、管理和安全地扩展整个组织的 Claude 采用。详细了解 **[Vero Security 与 Claude Compliance API 的集成](https://www.vero.security/blog/vero-antrpc-integration)**。

- **Willow:** Willow 为每个 AI 代理提供身份和对正确工具和技能的许可访问，具有完整的审计跟踪。通过与 Compliance API 的集成，Willow 将该治理扩展到 Claude 使用情况，使安全团队能够查看和控制整个组织的代理活动。要设置此集成，请参阅 **[Willow 设置指南](https://docs.withwillow.ai/docs/admin/settings/claude-compliance-sync)**。

- **WithWings:** WithWings 使用 Compliance API 发现 Claude Enterprise 环境中的 AI 代理和会话，映射其身份、权限和集成，并评估每个代理的操作是否与其原始目的保持一致，检测意图和行为之间的漂移。要设置此集成，请参阅 **[WithWings 设置指南](https://withwings.ai/integrations/claude)**。

- **Wiz:** Wiz 的 Claude Compliance API 连接器使组织能够发现 AI 项目、模型和用户，同时识别安全风险，如提升的角色或知识库中的敏感数据。它通过与 Claude Compliance API 集成来映射身份关系并跟踪整个企业的模型使用情况。详细了解 **[Wiz 与 Claude Compliance API 的集成](https://www.wiz.io/blog/claude-wiz-integration)**。

- **Zenity:** Zenity 自动发现和清点所有 Claude 代理、MCP 服务器、连接的资源和非人类身份，从会话开始时起映射其关系和安全态势。要设置此集成，请参阅 **[Zenity 设置指南](https://zenity.io/use-cases/platform/claude-enterprise)**。

- **Zscaler:** 通过与 Claude Compliance API 集成，Zscaler 现在能够提供对企业用户创建的每个 Claude 提示的深度可见性和精细控制。Zscaler 数据安全和 DLP 工作流帮助管理员通过从 Claude 提取的组织数据和活动日志的详细分析来加强其数据保护计划。此解决方案使更快的执行操作成为可能，以防止跨多个泄露渠道的数据丢失。要设置此集成，请参阅 **[Zscaler 设置指南](https://www.zscaler.com/products-and-solutions/data-security)**。