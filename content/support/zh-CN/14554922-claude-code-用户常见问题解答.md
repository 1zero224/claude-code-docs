# Claude Code 用户常见问题解答

办公时间常见问题的简短解答，附带深入了解的链接。按照你在前几周的进度组织。

## 如何使用本指南

五个部分遵循开发者前几周的发展轨迹：入门、日常使用、进阶、常见陷阱、隐私和信任。浏览与你当前阶段相符的部分，或搜索特定问题。

## 1. 入门

| **问题**                                     | **答案**                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **1.1 我如何安装它？**                            | **最简单的方法：**下载 Claude Desktop，用你的计划登录，然后打开“Code”标签页。Claude Code 已内置，无需安装其他内容。<br>**终端：**<br>**macOS/Linux：** `curl -fsSL <https://claude.ai/install.sh> | bash`
​**Windows PowerShell：** `irm <https://claude.ai/install.ps1> | iex`
​**Homebrew：** `brew install --cask claude-code`
​**WinGet：** `winget install Anthropic.ClaudeCode`
然后在任何仓库中运行 `claude`。<br>
参考：**[快速开始](https://code.claude.com/docs/en/quickstart)** |
| **1.2 已安装，但显示“claude: command not found”** | 原生安装程序将二进制文件放在 `~/.local/bin/claude`（Windows：`%USERPROFILE%\.local\bin`）。将该目录添加到你的 PATH，例如在 `~/.zshrc` 或 `~/.bashrc` 中运行 `export PATH="$PATH:$HOME/.local/bin"`，然后重启终端。<br>
参考：**[故障排除：PATH](https://code.claude.com/docs/en/troubleshooting)**                                                                                                                                                                                    |
| **1.3 登录在错误的机器上打开浏览器 / 我在 SSH 上**          | 在登录提示符处按 `c` 复制认证 URL。在本地浏览器中打开它，然后将代码粘贴回终端。<br>
参考：**[故障排除：认证](https://code.claude.com/docs/en/troubleshooting)**                                                                                                                                                                                                                                                                                                                 |
| **1.4 登录后立即出现认证错误，但我有访问权限**                | **400 “organization disabled”：**一个杂散的 `ANTHROPIC_API_KEY` 环境变量正在覆盖你的登录。取消设置它，从 shell 配置文件中删除，重启。运行 `/status` 确认哪个认证处于活跃状态。
​**403 Forbidden：**你的管理员还未为你的工作区启用 Claude Code，或者公司代理正在干扰。<br>
参考：**[故障排除：认证](https://code.claude.com/docs/en/troubleshooting)**                                                                                                                                                                      |
| **1.5 Claude Code 包含在我的计划中吗？**             | 是的。它包含在付费 Claude 计划和 Console（API）访问中。用你的工作账户登录；SSO 会自动处理。登录后出现 403 通常意味着你的管理员仍需为工作区启用它。<br>
参考：**[认证](https://code.claude.com/docs/en/authentication)**                                                                                                                                                                                                                                                                            |
| **1.6 Claude Code 与 Claude 中的聊天有什么区别？**    | **Claude Code：**一个编码代理，可以读取你的仓库、编辑文件和运行命令。它是 Claude Desktop 中的“Code”标签页，也可以在你的终端或 IDE 中运行。
**Claude 中的聊天**（网页、Claude Desktop 或移动设备）：用于对话和一次性问题的聊天。
底层使用相同的模型，工具形式不同。<br>
参考：**[概述](https://code.claude.com/docs/en/overview)**                                                                                                                                                                                                 |
| **1.7 它能在我的 IDE 中工作吗？**                    | 可以。VS Code 和 JetBrains IDE（IntelliJ、PyCharm 等）都有扩展可用。功能相同，嵌入在编辑器中而不是单独的终端。<br>
参考：**[VS Code](https://code.claude.com/docs/en/vs-code)** · **[JetBrains](https://code.claude.com/docs/en/jetbrains)**                                                                                                                                                                                                                              |
| **1.8 这与 Copilot/Cursor 自动完成有什么不同？**       | 自动完成建议接下来的几行。Claude Code 是一个代理：给它一个任务（“修复失败的测试”），它会读取文件、运行命令并进行多文件编辑直到完成。不是“完成我的句子”，而是“这是一个问题，去处理它”。<br>
参考：**[概述](https://code.claude.com/docs/en/overview)**                                                                                                                                                                                                                                                                     |
| **1.9 我应该先尝试什么？**                          | 指向一个你一直在推迟的繁琐但不难的 bug。例如：*“文件中的测试不稳定，找出原因。”*让它读取代码，而不是你解释代码。<br>
参考：**[常见用例](https://code.claude.com/docs/en/common-workflows)**                                                                                                                                                                                                                                                                                                   |
| **1.10 我如何更新它？**                           | 原生安装会在后台自动更新。要立即强制更新，运行 `claude update`。
Homebrew/WinGet 不会自动更新：定期运行 `brew upgrade claude-code` 或 `winget upgrade Anthropic.ClaudeCode`。<br>
参考：**[设置：更新](https://code.claude.com/docs/en/setup)**                                                                                                                                                                                                                               |

---

## 2. 日常使用

| **问题**                          | **答案**                                                                                                                                                                                                                                                       |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **2.1 它一直要求相同命令的权限**            | 默认情况下，批准仅在当前会话期间有效。要使其持久化：
• 在提示符处选择“始终允许”
• 将模式添加到 `.claude/settings.json` 中的 `permissions.allow`
• 或运行 `/permissions` 以交互方式管理<br>
参考：**[权限](https://code.claude.com/docs/en/permissions)**                                                         |
| **2.2 权限模式及如何切换**               | 按 **Shift+Tab** 循环切换模式：<br>     **auto** · 自动批准并进行后台安全检查。除非你的组织在托管设置中关闭了它，且在支持的模型上可用。<br>**manual** · 在进行风险编辑或命令前询问<br>**acceptEdits** · 文件编辑通过；运行命令前仍会询问<br>**plan** · 只读；提议一个计划并等待批准<br>
参考：**[权限模式](https://code.claude.com/docs/en/permission-modes)** |
| **2.3 /init 是什么，我何时运行它？**       | 在任何你会多次使用的仓库中，早期运行一次。它会扫描项目并写入 `CLAUDE.md`，包含构建命令、架构和约定。每个后续会话都会自动加载它，所以 Claude 开始时就有上下文。<br>
参考：**[内存和 CLAUDE.md](https://code.claude.com/docs/en/memory)**                                                                                               |
| **2.4 CLAUDE.md 中应该包含什么？**      | 工具无法强制执行的内容，新队友第一天会搞错的东西：“从 `release` 部署，不是 `main`”、“所有 ID 都是字符串”、“永远不要从路由处理程序直接调用数据库。”保持在两屏以内；更长的内容会被跳过。<br>
参考：**[内存和 CLAUDE.md](https://code.claude.com/docs/en/memory)**                                                                               |
| **2.5 Claude 没有遵循我的 CLAUDE.md** | • 太长或太模糊：精简到真正重要的规则
• 埋在散文中：把硬性规则放在顶部，使用祈使语气（“永远不要 X。总是 Y。”）<br>
参考：**[最佳实践](https://code.claude.com/docs/en/best-practices)**                                                                                                                           |
| **2.6 指向特定文件而不粘贴它**             | 输入 `@` 然后是路径（制表符自动完成）。提及的文件在 Claude 响应前被读取。<br>
参考：**[常见工作流](https://code.claude.com/docs/en/common-workflows)**                                                                                                                                           |
| **2.7 将截图粘贴到提示中**               | 将图像拖入终端，或按 **Ctrl+V**。在 Mac 上是 Ctrl，不是 Cmd（Cmd+V 粘贴文本）。适用于错误对话框、UI 模型、白板照片。<br>
参考：**[使用图像](https://code.claude.com/docs/en/common-workflows#work-with-images)**                                                                                           |
| **2.8 从终端复制 Claude 的响应**        | `/copy` 将最后的响应放到你的剪贴板。`/export` 将整个对话写入文件。                                                                                                                                                                                                                   |
| **2.9 恢复之前的会话**                 | `claude --continue` 恢复最近的一个。`claude --resume` 打开列表供选择。会话按项目目录本地存储。<br>
参考：**[常见工作流：恢复](https://code.claude.com/docs/en/common-workflows#resume-previous-conversations)**                                                                                   |
| **2.10 切换模型**                   | `/model` 打开选择器。如果你想每个会话都用同一个模型，在 `.claude/settings.json` 中设置默认值。<br>
参考：**[模型配置](https://code.claude.com/docs/en/model-config)**                                                                                                                           |
| **2.11 扩展思考**                   | 默认启用。推理本身在普通视图中隐藏；按 **Ctrl+O** 切换到详细记录（如果你想读它）。使用 `/effort` 调整深度。对于棘手的调试或架构决策值得额外延迟。<br>
参考：**[扩展思考](https://code.claude.com/docs/en/common-workflows#use-extended-thinking-thinking-mode)**                                                               |
| **2.12 在中途停止它**                 | 按 **Ctrl+C** 取消当前生成，然后告诉它改做什么。无需重新开始对话。                                                                                                                                                                                                                      |

---

## 3. 进阶

| **问题**                  | **答案**                                                                                                                                                                                                   |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **3.1 什么是 MCP？**        | MCP 将 Claude Code 连接到你的外部工具：GitHub、Linear、Slack、你的数据库、你的可观测性堆栈。一个 `.mcp.json` 配置，Claude 就能读取你的问题、查询你的数据、使用你使用的相同工具。常见的第一个连接器：你的问题跟踪器。<br>
参考：**[MCP](https://code.claude.com/docs/en/mcp)**            |
| **3.2 连接你的第一个 MCP 服务器** | • 在项目根目录添加 `.mcp.json`，或使用 `claude mcp add`
• 每个条目命名一个服务器包加上任何环境变量（通常是认证令牌）
• 重启 Claude Code 并运行 `/mcp` 确认已连接<br>
参考：**[MCP 设置](https://code.claude.com/docs/en/mcp)**                             |
| **3.3 钩子有什么用？**         | 在事件上触发的 Shell 脚本（工具运行前、文件编辑后、Claude 等待你时）。常见的第一个钩子：一个**通知**钩子，在 Claude 需要输入时 ping 你的桌面。相同的机制可以在每次编辑后运行你的 linter、发布到 Slack 或阻止对受保护路径的编辑。<br>
参考：**[钩子指南](https://code.claude.com/docs/en/hooks-guide)** |
| **3.4 制作可重用的提示 / 技能**   | 创建 `.claude/skills/ship/SKILL.md`，文件夹名称就变成命令：`/ship`。纯英文，无特殊语法。最简单的方法：让 Claude 为你写。（旧版 `.claude/commands/ship.md` 路径仍然有效。）<br>
参考：**[技能](https://code.claude.com/docs/en/skills)**                     |
| **3.5 技能 vs. 旧版命令**     | 相同的机制；命令已合并到技能中。`.claude/commands/foo.md` 和 `.claude/skills/foo/SKILL.md` 都创建 `/foo`。技能形式是首选的，为你提供一个支持文件的文件夹（参考文档、模板、辅助脚本）。<br>
参考：**[技能](https://code.claude.com/docs/en/skills)**                    |
| **3.6 子代理有什么用？**        | 并行工作：搜索代码库的不同部分、从不同维度审查差异或同时生成竞争实现。主会话聚合结果。<br>
参考：**[子代理](https://code.claude.com/docs/en/sub-agents)**                                                                                               |
| **3.7 非交互式运行（CI / 脚本）** | `claude -p "your prompt"` 运行一次并打印结果。适合 CI 钩子、预提交检查或管道传输到其他工具。通过已登录的会话或 `ANTHROPIC_API_KEY` 进行身份验证。<br>
参考：**[Unix 风格用法](https://code.claude.com/docs/en/common-workflows)**                            |
| **3.8 撤销其操作**           | `/rewind` 回滚到较早的检查点。每次发送提示时都会自动创建检查点。对于已提交的任何内容，请使用常规 `git revert`。<br>
参考：**[检查点](https://code.claude.com/docs/en/checkpointing)**                                                                    |
| **3.9 与团队共享您的设置**       | 将 `.claude/` 检入仓库（CLAUDE.md、命令、MCP 配置）。克隆仓库的任何人都会自动获得相同的设置。技能也可以打包为 **插件**，团队可以通过 `/plugin` 安装。<br>
参考：**[插件](https://code.claude.com/docs/en/plugins)**                                               |

---

## 4. 常见陷阱

| **问题**                             | **答案**                                                                                                                                                                                                                                               |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **4.1 找不到文件 / 搜索返回空结果**            | Claude Code 附带了 ripgrep 的捆绑副本，因此您无需安装它。捆绑的二进制文件在 Alpine/musl 系统上可能会失败；在这种情况下，安装系统副本（`apk add ripgrep`）并设置 `USE_BUILTIN_RIPGREP=0` 以便 Claude 使用它。<br>
参考：**[故障排除：搜索](https://code.claude.com/docs/en/troubleshooting#search-and-discovery-issues)** |
| **4.2 通过 SSH 或在 tmux 中复制/粘贴和滚动损坏** | 终端 UI 捕获鼠标事件。选择时按住 Shift 以绕过它，或配置 tmux 以传递鼠标事件。`/copy` 和 `/export` 完全避免了这个问题。                                                                                                                                                                        |
| **4.3 WSL 上速度缓慢**                  | 通过 `/mnt/c/` 读取 Windows 文件是已知的性能损失。将仓库移到 WSL 文件系统（`~/` 而不是 `/mnt/c/...`）。速度差异很明显。<br>
参考：**[故障排除：WSL](https://code.claude.com/docs/en/troubleshooting)**                                                                                           |
| **4.4 Mac 上的图像粘贴不起作用**             | 使用 **Ctrl+V**，而不是 Cmd+V。Cmd+V 粘贴文本；Ctrl+V 是从剪贴板粘贴图像的路径。                                                                                                                                                                                              |
| **4.5 通配符权限规则不匹配**                 | 逐步构建规则：首先以交互方式批准命令，检查写入设置的内容，然后进行泛化。<br>
参考：**[权限模式](https://code.claude.com/docs/en/permissions)**                                                                                                                                                |
| **4.6 非交互式 -p 模式行为不同**             | 需要 OAuth 的 MCP 服务器无法在非交互式模式下提示<br>交互式批准不会结转<br>对于非交互式/CI 运行，优先使用 API 密钥身份验证和配置了环境变量令牌的 MCP 服务器。                                                                                                                                                      |
| **4.7 任务中途上下文用尽**                  | `/compact` 总结早期对话以释放空间。`/clear` 在保持 `CLAUDE.md` 和设置加载的情况下重新开始。对于长任务，使用 `/clear` 在各个阶段之间分解步骤。<br>
参考：**[管理上下文](https://code.claude.com/docs/en/common-workflows)**                                                                                  |

---

## 5. 隐私和信任

| **问题**                                  | **答案**                                                                                                                                                                                 |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **5.1 Anthropic 会在我的代码上进行训练吗？**         | 不会。根据您组织的团队/企业条款，您的代码和对话不用于训练模型。<br>
参考：**[数据使用](https://code.claude.com/docs/en/data-usage)**                                                                                       |
| **5.2 我的代码实际上去哪里了？**                    | Claude Code 在您的机器上运行。源文件在本地读取，只有当前任务所需的部分才会发送到 API 以生成响应。没有任何内容被索引、作为整个仓库上传或用于训练。<br>
参考：**[数据使用](https://code.claude.com/docs/en/data-usage)**                                      |
| **5.3 其他人可以看到我的对话吗？**                   | 不会。会话存储在您的机器上，按项目目录存储，不与队友共享，也不在任何仪表板中可见。如果您*想*共享对话，请使用 `/export`。<br>
参考：**[数据使用](https://code.claude.com/docs/en/data-usage)**                                                     |
| **5.4 我如何将秘密和 .env 文件排除在对话之外？**         | Claude 只读取任务所需的文件；它不会扫描您的整个仓库。要硬阻止特定文件，请在 `.claude/settings.json` 中添加读取拒绝规则（例如 `"Read(.env*)"`）。即使您不小心要求读取被拒绝的文件，也无法读取。<br>
参考：**[权限](https://code.claude.com/docs/en/permissions)** |
| **5.5 “acceptEdits” 模式在不询问我的情况下可以做什么？** | 文件编辑无需提示即可进行。它仍然在运行 shell 命令、进行网络调用或触及工作目录外的任何内容之前询问。为了更严格的控制，请保持默认模式。<br>
参考：**[权限](https://code.claude.com/docs/en/permissions)**                                                  |

---

## 附录：仍然卡住了？

| **资源**                                   | **用途**              |
| ---------------------------------------- | ------------------- |
| `/help`                                  | 内置命令，列出您的会话中可用的功能   |
| `/feedback`                              | 从终端提交问题（`/bug` 的别名） |
| **[完整文档](https://code.claude.com/docs)** | 此处的所有内容，详细说明        |
| 您团队的 `#claude-code` 频道                   | 小成功和奇怪的错误都可以在那里讨论   |

---

## 附录：资源目录

| **页面**        | **链接**                                                                                         |
| ------------- | ---------------------------------------------------------------------------------------------- |
| 快速入门          | **[code.claude.com/docs/en/quickstart](https://code.claude.com/docs/en/quickstart)**           |
| 故障排除          | **[code.claude.com/docs/en/troubleshooting](https://code.claude.com/docs/en/troubleshooting)** |
| 权限            | **[code.claude.com/docs/en/permissions](https://code.claude.com/docs/en/permissions)**         |
| 内存和 CLAUDE.md | **[code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)**                   |
| MCP           | **[code.claude.com/docs/en/mcp](https://code.claude.com/docs/en/mcp)**                         |
| 数据使用          | **[code.claude.com/docs/en/data-usage](https://code.claude.com/docs/en/data-usage)**           |

Claude Code 更新频繁。在内部分发前，请根据 **[code.claude.com/docs](https://code.claude.com/docs)** 验证版本特定的详细信息。