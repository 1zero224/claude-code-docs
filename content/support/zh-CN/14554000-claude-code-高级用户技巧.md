# Claude Code 高级用户技巧

本文汇集了 Anthropic 公司 Claude Code 团队的工作流技巧。这些实践涵盖并行执行、规划、自动化、验证和自定义——团队每天用来更快交付代码的模式。每个人的设置都不同，所以请尝试找到适合你的方法。

**重要提示：**本指南中最有影响力的单一技巧是**验证**——让 Claude 能够检查自己的输出。如果你只采用一个实践，就选择这个。请参阅下面的**[验证](#h_7dd53c5c29)**部分。

**开始前：本指南的范围**

这些是从 Claude Code 团队的个别工程师那里收集的高级用户模式。因此：

- 此处显示的大多数命令都随 Claude Code 一起提供：/color 和 /btw 是**内置命令**，/simplify 和 /loop 是随 CLI 一起提供的**捆绑技能**。请参阅**[命令参考](https://code.claude.com/docs/en/commands)**和**[技能](https://code.claude.com/docs/en/skills)**。你可以通过在 .claude/skills/<name>/ 下添加 SKILL.md 文件来构建自己的技能。

- iMessage 插件在官方 claude-plugins-official 市场中提供。社区插件（例如"ralph-wiggum"插件）未经 Anthropic 审查或认可——在托管环境中安装第三方插件前请咨询你的管理员。

- 某些功能——沙箱、远程控制、计划的云任务、语音——**默认关闭**，可能被你的组织政策禁用。如果此处的命令或标志返回"不可用"，你的管理员可能未为你的工作区启用它。

本指南中的其他所有内容都适用于标准 Claude Code 安装。如有疑问，请运行 `/help` 查看你的会话中实际可用的内容。

## 目录

| **部分**        | **涵盖内容**                        |
| ------------- | ------------------------------- |
| 并行工作          | 工作树、子代理隔离、`/batch`              |
| 构建前规划         | 规划模式、模型选择、工作量级别                 |
| 有效提示          | 反推提示、`/btw`                     |
| 与 Claude 一起学习 | 解释性输出、图表、间隔重复                   |
| CLAUDE.md 和内存 | `/init`、PR 中的 @claude、自动内存、自动梦想 |
| 验证            | Chrome 扩展、桌面应用、`/simplify`      |
| 命令、技能和子代理     | 自定义命令、代理定义、代码审查代理               |
| 钩子            | 生命周期事件和模式                       |
| 权限和安全         | 预批准、自动模式、沙箱、长时间运行的任务            |
| 计划和重复任务       | `/loop`、`/schedule`             |
| 移动和远程控制       | 移动应用、传送、远程控制、Dispatch           |
| 工具集成 (MCP)    | 数据分析、错误修复、插件                    |
| 自定义你的环境       | 终端、状态行、语音、输出样式                  |
| SDK 和多仓库工作    | `--bare`、`--add-dir`、分叉、设置脚本    |

---

## 并行工作

### 同时运行多个会话

最大的生产力提升是同时运行 3-5 个 Claude 会话，每个都在自己的 git 工作树中。Claude Code 内置了原生工作树支持。

- 从 CLI 运行 `claude --worktree`（或 `claude --worktree my_worktree`）在隔离的工作树中启动会话。添加 `--tmux` 在自己的 Tmux 会话中启动。

- 从桌面应用，打开代码选项卡并勾选工作树复选框。

- 对于非 git VCS（Mercurial、Perforce、SVN），在 `settings.json` 中定义 `WorktreeCreate` 和 `WorktreeRemove` 钩子以获得相同的隔离。

要在多个会话中保持方向感，请命名你的工作树，设置 shell 别名（`za`、`zb`、`zc`）在它们之间跳转，对终端选项卡进行颜色编码，并启用终端通知，以便在任何 Claude 需要你的注意时收到通知。许多工程师保留一个专用的"分析"工作树，仅用于读取日志和运行查询。

### 具有工作树隔离的子代理

子代理也可以在隔离的工作树中运行，这对大型批量更改特别强大。将 `isolation: worktree` 添加到你的代理的前置内容中：

```
# .claude/agents/worktree-worker.md
---
name: worktree-worker
model: haiku
isolation: worktree
---
```

然后自然地提示：*"将所有同步 IO 迁移到异步。批量处理更改并启动 10 个具有工作树隔离的并行代理。每个代理应该端到端测试其更改，然后提交 PR。"*

### /batch 用于大规模迁移

`/batch` 命令会询问您关于迁移的信息，然后将工作分配给尽可能多的工作树代理——数十个、数百个或更多。每个代理独立工作、测试自己的更改，并独立创建 PR。

```
> /batch migrate src/ from JavaScript to TypeScript
```

---

## 构建前先规划

### 在规划模式下启动复杂任务

按 **Shift+Tab** 进入规划模式。投入精力完善计划，这样 Claude 可以一次性完成实现。典型流程是：进入规划模式 → 完善计划 → 切换到自动接受编辑 → Claude 执行。

团队的一些模式：

- 让一个 Claude 编写计划，然后启动第二个 Claude 作为首席工程师进行审查。

- 一旦出现问题，立即切换回规划模式并重新规划，而不是在过程中进行纠正。

- 规划模式后，Claude **会根据您正在处理的内容自动命名您的会话**——您也可以使用 `claude --name "auth-refactor"` 提前设置名称。

### 对所有工作使用 Opus 和思考功能

Claude Code 团队的理由：*"这是我用过的最好的编码模型，虽然它比 Sonnet 更大、更慢，但由于您需要引导它的次数更少，它在工具使用方面更好，最终使用它几乎总是比使用较小的模型更快。"*

**数学原理：**更少的引导 + 更好的工具使用 = 更快的整体结果，即使使用更大的模型。

### 努力程度

运行 /effort 选择您的努力程度。可用的级别有 **low**（更少的令牌，更快）、**medium**、**high**（更多令牌，更高智能）、**xhigh**、**max** 和 **auto**（Claude 按请求选择）。对于复杂的编码和代理工作，切换到 xhigh 以获得比 high 更深入的推理，而不需要 max 的完整令牌成本。对于困难的调试或架构决策，切换到 max，让 Claude 根据需要进行尽可能长的推理。Max 会更快地消耗使用限额，因此请按会话激活它。

---

## 有效提示

不要接受第一个解决方案——推动 Claude 做得更好。一些效果很好的提示：

- **"审查这些更改，在我通过您的测试之前不要创建 PR。"**强制 Claude 在发布前验证您的理解。

- **"向我证明这有效。"**让 Claude 比较 `main` 和您的功能分支之间的行为差异。

- **"考虑到您现在所知的一切，放弃这个并实现优雅的解决方案。"**在第一次尝试不理想后很有用。

编写详细的规范以减少歧义，然后再交付工作。您越具体，输出就越好。

### /btw 用于附加问题

当 Claude 正在积极工作时，使用 `/btw` 提出快速问题而不中断它。这是单轮的，没有工具调用，但具有完整的对话上下文。

```
> /btw what does the retry logic do?
```

---

## 与 Claude 一起学习

Claude Code 不仅用于编写代码——当您将其配置为解释和教学时，它是一个强大的学习工具。

- **在 `/config` 中启用"解释性"或"学习"输出样式**，让 Claude 解释更改背后的*原因*。

- **生成可视化 HTML 演示文稿**解释不熟悉的代码。

- **请求 ASCII 图表**说明新协议和代码库。

- **建立间隔重复技能：**解释您的理解，Claude 提出后续问题以填补空白。

---

## CLAUDE.md 和记忆

### 投资您的 CLAUDE.md

在您的仓库根目录共享一个 `CLAUDE.md` 文件，检入 git，整个团队贡献。关键做法：**每当 Claude 做错事时，将其添加到 CLAUDE.md**，这样它就知道不要重复这个错误。

每次更正后，以以下内容结尾：*"更新您的 CLAUDE.md，这样您就不会再犯那个错误。"*Claude 非常擅长为自己编写规则。

### 代码审查中的 @claude

使用 `/install-github-app` 安装 GitHub Action，然后在 PR 评论中标记 `@claude` 以在审查过程中将学习内容添加到 `CLAUDE.md`：

```
nit: use a string literal, not ts enum  @claude add to CLAUDE.md to never use enums, always prefer literal unions
```

这是"复合工程"——每次更正都会使未来的每个会话变得更好。

### 自动记忆

运行 `/memory` 配置 Claude Code 的内置记忆系统。

**自动记忆**在会话之间自动保存偏好、更正和模式。记忆写入 ~/.claude/projects/<project>/memory/（每个 git 仓库根目录一个目录）。这与您手动维护的用户级 ~/.claude/CLAUDE.md 和项目级 ./CLAUDE.md 文件分开。

命名映射到 REM 睡眠如何将短期记忆巩固为长期记忆。

### 高级：笔记目录

团队中的一位工程师让 Claude 为每个任务和项目维护一个笔记目录，在每个 PR 后更新——然后将 `CLAUDE.md` 指向它。

---

## 验证——第一大技巧

给 Claude 一种验证其工作的方式将显著提高最终结果的质量。如果 Claude 可以自己关闭反馈循环，它将迭代直到输出正确。

验证在不同领域看起来不同——bash 命令、测试套件、模拟器、浏览器测试——但原则是相同的。投资于特定领域的验证。

### Chrome 扩展

对于前端工作，安装 Claude Code Chrome 扩展。把它想象成任何其他工程师：如果您要求某人构建网站但不让他们使用浏览器，它会看起来不错吗？可能不会。有了浏览器，他们会迭代直到看起来不错。

团队每次处理网络代码时都使用 Chrome 扩展。在 **[code.claude.com/docs/en/chrome](https://code.claude.com/docs/en/chrome)** 下载 Chrome 或 Edge 版本。

### 用于网络服务器的桌面应用

Claude 桌面应用捆绑了在内置浏览器中**自动启动和测试网络服务器**的能力。您可以在 CLI 或 VS Code 中使用 Chrome 扩展设置类似的东西，或直接使用桌面应用。

### /simplify 用于代码质量

在进行更改后，在任何提示后附加 `/simplify`。它运行并行代理来审查更改的代码以实现重用、质量、效率和 `CLAUDE.md` 合规性——一次完成。

```
> hey claude make this code change then run /simplify
```

---

## 命令、技能和子代理

### 重复工作流的技能

如果你每天做某件事不止一次，就把它变成一个技能。技能被检入 `.claude/skills/<name>/SKILL.md` 并与团队共享（旧的 `.claude/commands/` 路径仍然有效，但技能是推荐的方法）。一些想法：

- 一个 `/techdebt` 命令，在每个会话结束时运行以查找重复的代码。

- 一个命令，将 7 天的 Slack、GDrive、Asana 和 GitHub 同步到一个上下文转储中。

- 分析工程师代理，编写 dbt 模型、审查代码并在开发中测试。

斜杠命令可以包含 **内联 Bash** 来预计算信息（如 `git status`），无需额外的模型调用。

### 用于 PR 工作流的子代理

将子代理视为最常见 PR 工作流的自动化。将 `.md` 文件放入 `.claude/agents/`：

```
.claude/agents/   build-validator.md   code-architect.md   code-simplifier.md   verify-app.md
```

每个代理可以有自定义名称、颜色、工具集、允许/禁止的工具、权限模式和模型。通过将 `"agent"` 添加到 `settings.json` 或使用 `claude --agent <name>` 来设置 **主对话的默认代理**。运行 `/agents` 开始。

### --agent 用于自定义系统提示

自定义代理是一个经常被忽视的强大原语。在 `.claude/agents` 中定义一个新代理，然后运行 `claude --agent=<name>`。只读代理的示例：

```
# .claude/agents/ReadOnly.md --- name: ReadOnly description: Read-only agent restricted to the Read tool only color: blue tools: Read ---  You are a read-only agent that cannot edit files or run bash.
```

### 在运行时利用子代理

- **在任何希望 Claude 投入更多计算资源的请求中附加"使用子代理"**。

- **将单个任务卸载到子代理**以保持主代理的上下文窗口清洁和专注。

- **通过钩子将权限请求路由到 Opus** — 让它扫描攻击并自动批准安全的请求。

### 代码审查代理

当 PR 打开时，Claude 可以调度一个代理团队，每个代理专注于不同的问题 — 逻辑错误、安全问题、性能回归 — 并发布内联注释。Anthropic 团队首先为自己构建了这个；每个工程师的代码输出显著增加，审查是瓶颈。

---

## 钩子

钩子让你在代理生命周期中的特定点确定性地运行逻辑。要求 Claude 添加钩子以开始。

### 常见钩子模式

| **事件**              | **用例**                              |
| ------------------- | ----------------------------------- |
| `SessionStart`      | 每次启动 Claude 时动态加载上下文                |
| `PreToolUse`        | 记录模型运行的每个 bash 命令                   |
| `PostToolUse`       | 在写入/编辑后自动格式化代码以防止 CI 失败             |
| `PermissionRequest` | 将权限提示路由到 Slack、WhatsApp 或 Opus 进行审查 |
| `Stop`              | 对长任务运行确定性检查，或推动 Claude 继续进行         |
| `PostCompact`       | 在上下文压缩后重新注入关键指令                     |

自动格式化的 `PostToolUse` 钩子示例：

```
"PostToolUse": [
  {
    "matcher": "Write|Edit",
    "hooks": [{ "type": "command", "command": "bun run format || true" }]
  }
]
```

---

## 权限和安全

### 预批准常见命令

运行 `/permissions` 来预允许常见的安全命令并将其检入团队的 `.claude/settings.json`。这是 **跳过权限的推荐替代方案** — 你会收到更少的提示，同时保持可审计的允许列表。**完全支持通配符语法** — 尝试 `"Bash(bun run *)"` 或 `"Edit(/docs/**)"`。

Claude Code 的权限系统分层提示注入检测、静态分析、沙箱和人工监督。一小组安全命令是开箱即用的预批准；你通过 `/permissions` 添加的所有内容都是对该基线的补充。

### 自动模式

自动模式让 Claude 代表你做出权限决定。分类器在每个操作运行前评估它 — 安全操作会自动批准，风险操作仍会被标记。你可以在会话期间随时使用 **Shift+Tab** 切换权限模式。

### 沙箱

运行 `/sandbox` 以选择加入 Claude Code 的开源沙箱运行时。它在你的机器上运行并支持 **文件和网络隔离**，提高安全性同时减少权限提示。有三种模式可用：

- 沙箱 BashTool，带自动允许

- 沙箱 BashTool，带常规权限

- 无沙箱

### 长时间运行的任务

对于非常长时间运行的任务，确保 Claude 可以不间断地工作。推荐的方法：

- 提示 Claude 在完成时与后台代理进行验证。

- 使用代理 `Stop` 钩子进行确定性检查（对于可审计的工作流程首选）。

- 使用"ralph-wiggum"社区插件。

对于沙箱环境，使用 `--permission-mode=dontAsk` 或 `--dangerously-skip-permissions` 以避免阻止。

---

## 计划和重复任务

### /loop 用于本地重复任务

`/loop` 在本地安排重复任务，最长可达 3 天。Claude Code 团队运行的几个示例：

```
/loop 5m /babysit         # auto-address review, rebase, shepherd PRs /loop 30m /slack-feedback # auto put up PRs for Slack feedback /loop 1h /pr-pruner       # close out stale PRs
```

### /schedule 用于云作业

与 `/loop` 不同，计划作业在**云**中运行——即使您的笔记本电脑关闭，它们也会继续工作。

```
> /schedule a daily job that looks at all PRs shipped since   yesterday and updates our docs based on the changes. Use   the Slack MCP to message #docs-update with the changes
```

**注意：**尝试将您最常见的工作流程转换为技能 + 循环。这很强大。

---

## 移动和远程控制

### 从您的手机工作

Claude Code 有一个**移动应用**——下载 iOS/Android 的 Claude 应用并点击代码选项卡。还提供了**iMessage 插件**（`/plugin install imessage@claude-plugins-official`）以从任何 Apple 设备发送任务。

### 在设备之间传送会话

在移动、网络、桌面和终端之间来回移动会话：

- `claude --teleport`（或从会话内部使用 `/teleport`）在您的机器上继续云会话。

- `/remote-control` 让您从手机或网络控制本地会话。

- `claude remote-control` 让您从移动应用生成新的本地会话。*可用性：Pro、Max、Ultra、Team 和 Enterprise 计划在 CLI v2.1.51+ 上。*

您也可以在 `/config` 中启用**"为所有会话启用远程控制"**。

### Claude Cowork Dispatch

Dispatch 是 Claude Desktop 应用的安全远程控制。它可以在您的许可下使用您的 MCP、浏览器和计算机——对于赶上 Slack 和电子邮件、管理文件以及在您不在笔记本电脑时在其上执行操作很有用。

---

## 工具集成 (MCP)

将 Claude 连接到您现有的工具，以便它可以搜索 Slack、运行 BigQuery、获取 Sentry 日志等。通过 claude mcp add 或 settings.json 中的

### 数据和分析

要求 Claude 使用 `bq` CLI 即时拉取和分析指标——在您的代码库中检查 BigQuery 技能。Claude Code 团队的看法："个人而言，我已经 6 个多月没有写过一行 SQL 了。"这适用于任何具有 CLI、MCP 或 API 的数据库。

### 错误修复

启用 Slack MCP，将错误线程粘贴到 Claude 中，然后说**"修复"**——零上下文切换。或者说**"去修复失败的 CI 测试"**而不进行微观管理。将 Claude 指向**docker logs** 以排查分布式系统故障——它在这方面的能力令人惊讶。

### 插件

插件捆绑 LSP（适用于每种主要语言）、MCP、技能、代理和自定义钩子。从官方 Anthropic 插件市场安装，或为您的组织建立内部市场——然后将市场参考检查到 `settings.json` 中，以便为每个开发人员自动添加。运行 `/plugin` 以开始。

---

## 自定义您的环境

### 终端设置

运行 `/config` 设置浅色/深色模式，运行 /terminal-setup 在 IDE 终端、Warp 或 Alacritty 中启用**Shift+Enter** 换行（Apple Terminal 不支持 `/terminal-setup`）。对于 Vim 键绑定，打开 `/config` → 编辑器模式。团队推荐**Ghostty** 以实现同步渲染和 24 位颜色。

### 状态行、颜色和键绑定

- `/statusline` 根据您的 `.bashrc`/`.zshrc` 生成自定义状态行——显示模型、目录、剩余上下文、成本或其他任何内容。

- `/color` 更改提示输入颜色——当您打开 3-5 个会话并需要一目了然地区分它们时很有用。

- `/keybindings` 重新映射任何键。设置实时重新加载并存储在 `~/.claude/keybindings.json` 中。

### 语音输入

语音模式对所有用户可用，包括 Claude Code Desktop 和 Cowork。Claude Code 团队的大部分编码都是通过语音完成的——您说话的速度大约是打字速度的 3 倍，因此您的提示会变得更详细。

- **CLI：**运行 `/voice` 然后按住空格键

- **桌面：**按语音按钮（麦克风图标）

- **iOS：**在系统设置中启用听写

- **macOS 原生：**在任何终端中按 fn×2 进行系统听写

### 云会话

除了终端外，从 claude.ai/code 在云中运行其他会话。使用 `&` 命令将会话发送到云，或使用 `--teleport` 标志在本地和云会话之间切换上下文。

### 输出样式

运行 `/config` 并设置输出样式。**解释性**让 Claude 在工作时解释框架和模式（非常适合新代码库）。**学习**让 Claude 指导您完成更改。您也可以创建**自定义**样式来调整 Claude 的语音。

### 微调动词

正是这些小事情使 Claude Code 感到个性化。要求 Claude 自定义您的微调动词以添加到或替换默认列表。将 `settings.json` 检查到源代码控制中以与您的团队共享动词。

### 自定义一切

Claude Code 开箱即用效果很好，但当您进行自定义时，**将 `settings.json` 检查到 git** 以便您的团队也能受益。配置支持按代码库、按子文件夹、按用户或通过企业范围的策略。

**按数字统计：**数十个设置和环境变量—请参阅**[设置参考](https://code.claude.com/docs/en/settings)**。在`settings.json`中使用`"env"`字段以避免包装脚本。

---

## SDK 和多仓库工作

### --bare 用于更快的 SDK 启动

默认情况下，`claude -p`（以及 TypeScript/Python SDK）会搜索本地`CLAUDE.md`文件、设置和 MCP。对于非交互式使用，通常需要通过`--system-prompt`、`--mcp-config`、`--settings`等显式指定这些。添加`--bare`以获得大约 10 倍的启动速度提升：

```
claude -p "summarize this codebase" \
    --output-format=stream-json \
    --verbose \
    --bare
```

**注意：**这是 SDK 首次构建时的设计疏漏。在未来版本中，默认值将切换为`--bare`。目前，请使用该标志选择加入。

### --add-dir 用于多仓库工作

在跨仓库工作时，使用`--add-dir`（或`/add-dir`）为 Claude 提供对其他文件夹的访问权限和权限。或在团队的`settings.json`中添加`"additionalDirectories"`以始终包含它们。

### 分支会话

要分支现有会话，请从其内部运行`/branch`，或从 CLI 运行`claude --resume <session-id> --fork-session`。

### 云环境的设置脚本

在 Claude Code 云会话和 Claude Desktop 中，添加在每个新云会话之前运行的设置脚本—安装依赖项、配置设置、设置环境变量。恢复时跳过该脚本。

---

## 附录：快速参考

| **区域**   | **关键命令**                                                 |
| -------- | -------------------------------------------------------- |
| 并行工作     | `claude --worktree`、`/batch`、`isolation: worktree`       |
| 规划       | Shift+Tab、`/effort max`、`claude --name`                  |
| 内存       | `CLAUDE.md`、`/memory`、`/dream`、PR 中的`@claude`            |
| 验证       | Chrome 扩展程序、`/simplify`、Desktop 应用                       |
| 自动化      | `.claude/skills/`、`.claude/agents/`、`--agent`            |
| 钩子       | `PostToolUse`、`Stop`、`PostCompact`、`PermissionRequest`   |
| 权限       | `/permissions`、自动模式、`/sandbox`                           |
| 调度       | `/loop`、`/schedule`                                      |
| 远程       | `--teleport`、`/remote-control`、移动应用、iMessage             |
| 自定义      | `/statusline`、`/color`、`/voice`、`/keybindings`、`/config` |
| SDK 和多仓库 | `--bare`、`--add-dir`、`/branch`                           |

---

## 附录：相关文章

| **资源**      | **链接**                                                                                         |
| ----------- | ---------------------------------------------------------------------------------------------- |
| 钩子参考        | **[code.claude.com/docs/en/hooks](https://code.claude.com/docs/en/hooks)**                     |
| 子代理和自定义代理   | **[code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents)**           |
| 计划任务        | **[code.claude.com/docs/en/scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks)** |
| Chrome 扩展程序 | **[code.claude.com/docs/en/chrome](https://code.claude.com/docs/en/chrome)**                   |

Claude Code 频繁发布。在内部分发前，请根据**[code.claude.com/docs](https://code.claude.com/docs)**验证版本特定的详细信息。