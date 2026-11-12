# Claude Code：常见开发者用例

Claude Code 是一个编码代理，可以读取您的代码库、编辑文件、运行命令，并在执行可能具有破坏性的操作前请求确认。它内置于 Claude Desktop 中，也可在您的终端和支持的 IDE 中运行。它旨在协助软件开发生命周期的每个阶段，从探索陌生代码库到发布和维护生产代码。

下表总结了 Claude Code 在开发生命周期中的位置。本文的其余部分将介绍开发者最常使用它的十项任务，每项任务都包含示例提示和预期结果。

| **1. 发现** | **2. 设计** | **3. 构建** | **4. 部署** | **5. 支持和扩展** |
| --------- | --------- | --------- | --------- | ------------ |
| 探索代码库和历史  | 规划项目      | 实现代码      | 自动化 CI/CD | 调试错误         |
| 搜索文档      | 开发技术规范    | 编写并执行测试   | 配置环境      | 大规模重构        |
| 入职和学习     | 定义架构      | 创建提交和 PR  | 管理部署      | 监控使用情况和性能    |

## 如何使用本指南

下面的每个用例都包括何时使用、示例提示和预期结果。除了第 9 项（问题跟踪器集成）需要一次性 MCP 连接器设置外，其他所有项都不需要配置。

用例大致按照大多数开发者遇到它们的早晚顺序排列。选择与您当前任务相匹配的用例。

---

## 用例

### 1. 修复失败的测试

**何时使用：**测试失败且原因不明显。使用此功能让 Claude Code 定位根本原因并提出修复方案，无需您先识别源文件。

**示例提示：**

```
> the tests in tests/auth.test.ts are failing, can you figure out why and fix them
```

**预期结果：**Claude Code 读取测试文件，追踪被测模块的代码路径，识别不匹配之处，并提出编辑建议。您批准更改后，它会重新运行测试套件以确认修复。

### 2. 理解陌生代码

**何时使用：**在进行更改前，您需要了解模块、函数或子系统的工作原理——例如，在继承代码或审查陌生区域时。

**示例提示：**

```
> walk me through how the payment retry logic works
```

**预期结果：**Claude Code 定位相关文件，读取实现，并用简洁语言解释控制流：入口点、决策分支、边界情况以及代码如何与系统其余部分连接。提出后续问题（例如

## 3. 查找某事发生的位置

**何时使用：**您知道代码库中存在某个行为，但无法通过文件名或简单文本搜索找到它。

**示例提示：**

```
> where do we validate email addresses in this codebase? I want to add a new rule
```

**预期结果：**Claude Code 搜索代码库，打开候选文件，并返回文件路径和行号，附带足够的上下文以识别正确的匹配项。如果存在多个位置，它会指出哪个是真实来源。

### 4. 分类错误或堆栈跟踪

**何时使用：**您有运行时错误、堆栈跟踪或日志输出，需要将其映射回相应的代码。

**示例提示：**

```
<table class="prompt"> <colgroup> <col style="width: 100%" /> </colgroup>
<tbody> <tr class="odd"> <td>&gt; getting this in production:<br />
TypeError: Cannot read properties of undefined (reading 'sessionId')<br />
at validateSession (src/auth/session.ts:47)<br />     at middleware
(src/auth/middleware.ts:12)<br /> what's going on?</td> </tr> </tbody>
</table>
```

**预期结果：**Claude Code 读取跟踪中引用的文件，解释未定义的内容及原因，并提出修复方案。对于没有清晰堆栈跟踪的错误，粘贴任何可用的日志输出，Claude Code 将从上下文重建失败。

### 5. 按计划重构

**何时使用：**更改跨越多个文件，您希望在进行任何编辑前审查完整范围。

**前置条件：**按 **Shift+Tab** 直到模式指示器显示

**示例提示：**

```
> refactor the auth module to use the new session store instead of SessionCache
```

**预期结果：**Claude Code 分析代码并生成一个编号计划，列出它将修改的每个文件和每个文件中的更改。您可以按原样批准计划或修改它（例如

**参考：****[权限](https://code.claude.com/docs/en/permissions)**

### 6. 为现有代码编写测试

**何时使用：**源文件缺乏覆盖，或您需要与项目现有测试约定相匹配的额外边界情况测试。

**示例提示：**

```
> add tests for @src/billing/invoice.ts — cover the edge cases around partial refunds. match the style of the existing tests in tests/billing/
```

**预期结果：**Claude Code 读取源文件和参考测试目录，以现有风格生成新的测试文件，并运行它以确认测试通过。如果仍存在明显的覆盖缺口，它会标记这些缺口并询问是否要解决。

### 7. 审查拉取请求

**何时使用：**您需要审查或总结 PR，特别是在您不太熟悉的代码库区域。

**前置条件：**GitHub CLI 已认证（运行 `gh auth login` 一次）。

**示例提示：**

```
> /pr https://github.com/example/api/pull/4471
> is the error handling solid?
```

**预期结果：**Claude Code 获取 diff、审查评论和 CI 状态，然后在其完整的存储库上下文中读取更改的文件。它可以生成有针对性的审查、起草审查评论、验证特定问题或总结 PR。

### 8. 加入新存储库

**何时使用：**您首次在存储库中工作，需要其架构、构建命令和约定的结构化概览。

**示例提示：**

```
<table class="prompt"> <colgroup> <col style="width: 100%" /> </colgroup>
<tbody> <tr class="odd"> <td>&gt; /init<br /> &gt; give me a tour of this
codebase — where's the entry point, how is it structured, what should I
read first</td> </tr> </tbody> </table>
```

**预期结果：**`/init` 扫描项目并写入 `CLAUDE.md` 文件，总结构建命令、架构和约定。后续导览提示会生成项目结构的引导式演练。项目上下文在会话的其余部分保持加载。

**参考：****[内存和 CLAUDE.md](https://code.claude.com/docs/en/memory)**

### 9. 端到端处理问题

**何时使用：**您想在单个对话中读取工单、实施修复并验证它，无需切换工具。

**前置条件：**通过 MCP 连接的问题跟踪器（一次性 `.mcp.json` 配置）。

**示例提示：**

```
<table class="prompt"> <colgroup> <col style="width: 100%" /> </colgroup>
<tbody> <tr class="odd"> <td>&gt; what's the top-priority issue assigned
to me?<br /> &gt; go ahead and fix it</td> </tr> </tbody> </table>
```

**预期结果：**Claude Code 查询问题跟踪器、读取工单、定位相关代码、提出修复方案并运行测试。会话以可审查的 diff 和关闭工单所需的上下文结束。

**参考：****[MCP](https://code.claude.com/docs/en/mcp)**

### 10. 将重复任务转变为技能

**何时使用：**您定期重复相同的多步骤提示（提交前检查、站会总结、PR 描述、发布说明），并希望使其对整个团队可重用。

**示例提示：**

```
> make me a /ship skill that runs the tests, runs the linter, looks at git diff, and drafts a conventional-commit message — then shows me the message without committing
```

**预期结果：**Claude Code 将 `SKILL.md` 文件写入 `.claude/skills/ship/`。`/ship` 命令立即可用，无需重启，并与克隆存储库的任何人共享。（旧版 `.claude/commands/` 路径仍受支持，但技能是推荐的形式。）

**参考：****[技能](https://code.claude.com/docs/en/skills)**

## 附录：快速参考

| **#** | **用例**    | **示例提示**                                                |
| ----- | --------- | ------------------------------------------------------- |
| 1     | 修复失败的测试   | `[file] 中的测试失败 — 找出原因并修复`                               |
| 2     | 理解陌生代码    | `向我讲解 [module] 如何工作`                                    |
| 3     | 找到某事发生的位置 | `我们在这个代码库中的哪里 [action]？`                                |
| 4     | 分类错误      | 粘贴堆栈跟踪，然后询问：`发生了什么？`                                    |
| 5     | 按计划重构     | Shift+Tab → 计划模式 → 描述重构                                 |
| 6     | 编写测试      | `为 @[file] 添加测试 — 匹配 [dir] 中的风格`                        |
| 7     | 审查 PR     | `/pr [URL]` 然后提出您的问题                                    |
| 8     | 加入仓库      | `/init` 然后                                              |
| 9     | 端到端处理问题   | `what's my top-priority issue?` → `go ahead and fix it` |
| 10    | 创建技能      | `make me a /[name] skill that [steps]`                  |

## 附录：资源目录

| **资源**          | **链接**                                                                                           |
| --------------- | ------------------------------------------------------------------------------------------------ |
| 快速开始            | **[code.claude.com/docs/en/quickstart](https://code.claude.com/docs/en/quickstart)**             |
| 常见工作流           | **[code.claude.com/docs/en/common-workflows](https://code.claude.com/docs/en/common-workflows)** |
| 权限              | **[code.claude.com/docs/en/permissions](https://code.claude.com/docs/en/permissions)**           |
| 内存和 `CLAUDE.md` | **[code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)**                     |
| MCP 连接器         | **[code.claude.com/docs/en/mcp](https://code.claude.com/docs/en/mcp)**                           |
| 技能              | **[code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)**                     |
| 内置命令            | **[code.claude.com/docs/en/commands](https://code.claude.com/docs/en/commands)**                 |

有关 `/init`、@-引用、权限模式、技能、MCP 和钩子的详细信息，请参阅 **[code.claude.com/docs](https://code.claude.com/docs)**。Claude Code 频繁更新——在内部分发前，请根据 **[code.claude.com/docs](https://code.claude.com/docs)** 验证版本特定的详细信息。