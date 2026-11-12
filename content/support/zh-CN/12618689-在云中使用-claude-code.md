# 在云中使用 Claude Code

云会话是在云基础设施上运行的 Claude Code 会话，而不是在您的机器上运行，可以处理来自 GitHub 存储库的代码。您可以从 Claude Desktop、claude.ai/code 上的 Claude Code 网页版、Claude Mobile 或终端中使用 `claude --cloud` 命令启动云会话。本文介绍了云会话的工作原理、何时使用云会话而不是 Claude Desktop、终端或 IDE 中的本地会话，以及它们支持的工作流程。

## 云会话提供的功能

云会话让您可以将任务委托给 Claude，这些任务无需您的主动监督即可运行。您选择一个 GitHub 存储库，描述您想要完成的工作，Claude 在云中处理该任务。Claude Code 开始处理任务后，您可以完全离开页面；Claude 将继续其工作。完成后，Claude 会自动创建一个拉取请求，其中包含供您审查的更改。

此功能适用于您本地机器上可能没有的存储库。您可以在任何有权访问的 GitHub 存储库上启动任务，无需在本地克隆或设置开发环境。这对于您偶尔贡献的项目或您仍在学习的代码库很有用。

云会话支持异步开发工作流程。在终端或编辑器中的本地会话中，您通常同步工作：您发出请求，等待 Claude 响应，审查更改，然后发出另一个请求。这种同步工作方式让您能够进行细粒度控制，但需要您在整个过程中保持关注。云会话的处理方式不同：您可以分配一个更大的任务，让 Claude 独立工作，然后稍后返回审查已完成的工作。

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/1786446157/07ec74cd46317f8278083a317841/6448f3ee-c6df-4417-8a13-90d8c2ca3d55?expires=1794465000&amp;signature=868266a7a56b4c19d8a9ba0dd2ab9ee6743ef157e917114dea00bad74d5dca5c&amp;req=dScvEM16m4BaXvMW1HO4zR8%2BAVeERJh27XrRA1YwWGvuwJkB0KlLerWC837w%0AuPP2Zhdio3SDlixQWtc%3D%0A)

您还可以并行运行多个任务。由于每个任务在自己的隔离环境中运行，您可以让 Claude 同时处理多个不同的问题或存储库。每个任务独立进行，完成时创建自己的拉取请求。多个任务可以同时在同一存储库上工作。

---

## 工作原理

启动云会话时，Claude Code 会为您的工作创建一个隔离的虚拟机。您的 GitHub 存储库被克隆到此环境中，该环境预配置了常见的开发工具和语言生态系统。

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/1786446158/c092f1383826cb871493f74169d4/97b7cb98-5da2-438e-a920-e170b8b9790e?expires=1794465000&amp;signature=89272b0cb261b5446999914eed4e3abdc1ec7950c12dc6c63c94d9047b62fc8b&amp;req=dScvEM16m4BaUfMW1HO4zcR0rJIwiezA7DtpMiX%2FBYl8xe8qgVf66c%2FLSqlP%0AtvohaoueybK13J0FwjU%3D%0A)

Claude 通过运行您在存储库配置中定义的任何设置命令来准备环境。这包括安装依赖项、设置数据库或运行项目需要的其他初始化步骤。如果您的任务需要网络访问（例如安装包或获取数据），您可以配置环境具有的互联网访问级别。

环境准备就绪后，Claude 开始处理您的任务。Claude 读取您的代码、进行更改、编写测试并运行命令来验证工作。如果需要，您可以通过网页界面监控进度并提供指导。

![](https://downloads.intercomcdn.com/i/o/lupk8zyo/1786446156/83ecf0a5b98eddc9ffc9694c50f7/353589ce-b678-441d-8909-71b45fa2d065?expires=1794465000&amp;signature=987fe2acf8e1b3b623ed1289227a70426c3c961842017beb903800ea8cb59b3d&amp;req=dScvEM16m4BaX%2FMW1HO4zVbcTWeH5cHOUQl3YqgIJdbZyTNBZTWjzZVpNVRf%0Ax7x%2F82qeI37RAH49X8k%3D%0A)

Claude 完成任务后，会将更改推送到 GitHub 存储库中的新分支。您会收到通知，可以审查更改，然后直接从界面创建拉取请求。拉取请求包含 Claude 的所有工作，可供您审查和进行任何其他您想要进行的更改。

每个任务在完全隔离的环境中运行。虚拟机仅针对该特定任务存在，并包括安全控制，如受限网络访问和受保护的凭证处理。您的 GitHub 身份验证通过安全代理进行管理，因此凭证永远不会直接存在于 Claude 工作的环境中。

---

## 何时使用云会话与本地会话

某些任务适合在云中异步运行，而其他任务最好在 Claude Desktop、终端或 IDE 中本地运行。

### 使用云会话的情况：

- **明确定义的任务，具有清晰的要求：**当您可以准确描述需要完成的工作，并且不期望需要在任务中途引导 Claude 时，云会话让您启动工作并在完成时返回。

- **对错误积压的后台工作：**您可以从积压中为 Claude 分配多个问题，让它们并行运行。每个任务独立进行，允许您同时处理多个修复，而无需单独监控每一个。

- **您本地没有的存储库：**如果您需要对未克隆或不想在机器上设置的存储库进行更改，云会话会为您处理环境设置。

- **您想要排队的任务：**当您有一个更改列表但现在不想处理它们时，您可以在云中启动它们并稍后审查结果。这让您可以批量处理类似的工作或在您专注于其他事情的时候委托任务。

### 在 Claude Desktop、终端或 IDE 中使用本地会话的情况：

- **需要频繁纠正方向的任务：**当您不确定正确的方法是什么，或期望您需要根据看到的内容重定向 Claude 时，本地会话会给您即时反馈。您可以在 Claude 工作时调整方向，而不是等待完整的结果。

- **需求不明确的探索性工作：**如果您正在弄清楚如何解决问题或调查不同的方法，本地会话让您可以在学习时改进您的请求。来回的讨论有助于澄清一开始不明显的需求。

- **具有未提交更改的本地开发：**当您正在积极开发并在本地存储库中有未提交的工作时，在本地使用 Claude Code 会将所有内容保持在一个地方。您可以快速迭代更改，而无需提交或推送尚未准备好的工作。

- **需要即时反馈的任务：**如果您需要快速看到结果并想要快速迭代，本地会话提供更低的延迟。您可以实时观看 Claude 工作，如果过程早期出现问题，可以停止或重定向。

---

## 示例用例

### 使用测试驱动开发的后端更改

让 Claude 编写定义预期行为的测试，然后实现代码以使这些测试通过。这对于可以通过自动化测试验证行为的后端更改特别有效。

**示例提示：**

```
Add rate limiting to the /api/search endpoint.

The rate limiter should:
- Allow 100 requests per minute per API key
- Return 429 status when limit exceeded
- Reset limits after 60 seconds
- Track different API keys independently

Use a TDD approach: write comprehensive tests first, then implement the rate limiting logic to pass them.
```

**何时使用此方法：**这在云会话中效果很好，因为测试为 Claude 提供了明确的验证标准。您无需监控 Claude 的进度，因为测试会捕获问题并指导迭代以获得工作解决方案。任务的自包含性质（Claude 编写测试然后使其通过）不需要您在启动后的输入。

**这为什么有效：**Claude 可以在没有您监督的情况下迭代实现，使用测试失败来识别和修复问题。任务运行时间比简单的代码更改要长，但您可以让它在后台完成。当您审查拉取请求时，测试和实现都已准备好，您对解决方案有信心，因为测试通过了。

### 文档更新

生成或更新技术文档，如 README 文件、API 文档、代码注释或用户指南。

**示例提示：**

```
Update CHANGELOG.md with all changes since the v2.3.0 release:
  - Review commits on main branch since that tag.
  - Categorize changes into "Added, "Changed, "Fixed", and "Removed" sections.
  - Include the commit hash for each entry.
```

**何时使用此方法：**更新日志更新非常适合云会话，因为 Claude 可以独立审查提交历史并格式化条目而无需指导。该任务手动执行很繁琐，但足够简单，Claude 可以完成而无需关于包含哪些提交或如何分类的问题。

**这为什么有效：**您可以委托整个更新日志更新并在完成时审查结果。Claude 读取提交、提取有意义的更改并遵循您现有的更新日志格式。

### 具有明确范围的重构

当您可以定义明确的更改边界时，重构代码以改进组织或可读性。这包括提取代码、拆分大文件或组织模块结构。

**示例提示：**

```
The UserService class in /src/services/user.go has 800 lines long.
Split it into three focused services:
  - UserAuthService (login/logout/sessions)
  - UserProfileService (profile CRUD operations)
  - UserPreferencesService (settings/preferences)

Ensure all tests still pass.
```

**何时使用此方法：**具有明确约束的重构在云会话中效果很好，因为您可以为 Claude 设置明确的边界。测试套件可以提供验证，允许Claude 验证重构没有破坏任何现有功能。

**这为什么有效：**该任务需要时间，但一旦定义了结构，就不需要您的主动输入。您可以启动重构并稍后审查组织的结果，而不是在 Claude 处理任务时监控它。明确的范围意味着 Claude 不太可能需要任务中途的指导。

---

## 有效使用的提示

- 考虑向您的存储库添加测试套件，以便 Claude 更容易验证它已成功完成任务

- 指定成功标准，而不是模糊的目标，如"改进"或"修复"

- 在您的提示中定义应该更改的内容和应该保持不变的内容

- 使用明确的边界来确定任务范围，以便 Claude 不需要任务中途的指导

- 如果您在想"我需要先看看这是如何进行的"，请考虑使用本地会话

- 如果您在任务中途意识到需要提供指导，请使用"在 CLI 中打开"