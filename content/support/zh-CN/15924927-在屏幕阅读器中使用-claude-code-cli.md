# 在屏幕阅读器中使用 Claude Code CLI

屏幕阅读器模式将 Claude Code 恢复到基本的终端体验：纯文本、顺序显示，并添加了标签和提示，没有视觉装饰。它是为屏幕阅读器用户构建的，对于需要纯文本输出的盲文显示、低速连接或转录的任何人都很有用。

## 打开屏幕阅读器模式

屏幕阅读器模式需要 Claude Code 版本 2.1.181 或更高版本，可以通过多种方式启用：

- 对于单个会话：运行 `claude --ax-screen-reader`

- 对于终端中的每个会话：设置环境变量 `CLAUDE_AX_SCREEN_READER=1`

- 永久设置：将 `"axScreenReader": true` 添加到您的 Claude Code 设置文件（`~/.claude/settings.json`）

会话的第一行宣布屏幕阅读器模式已打开。在 Claude Code v2.1.206 及更高版本中，它宣布：`[Screen Reader Mode: on via flag]`（或 `via env` / `via settings`）。

## 预期效果

- **您的提示和 Claude 的回复都有标签。**您的消息以 `you:` 开头，Claude 的回复以 `claude:` 开头。工具活动以 `tool:` 开头，权限提示以 `Permission Required:` 开头。这些标签也是可搜索的，因此您可以使用终端的搜索功能在对话的各个部分之间跳转。

- **菜单变成编号列表。**您不再用箭头键移动高亮，而是听到每个选项作为编号行，然后输入您想要的选项的编号并按 Enter。对于是/否提示，输入 `y` 或 `n` 并按 Enter。

- **视觉装饰被移除。**加载动画、进度动画和原地重绘变成按顺序读取的纯文本，装饰性字符（如制表符）被移除。

- **终端铃声告诉您何时 Claude 需要您。**当 Claude 完成回复、出现权限提示或长时间运行的工具完成时，您会听到终端的警报声。如果您听不到，请检查终端应用程序中的铃声设置——某些终端默认关闭警报声。Claude Code 自己的通知频道可以通过 `/config` 在通知下设置。

要了解有关行标记、列表导航和现有限制的全面文档，请访问 **[Claude Code 指南的无障碍部分](https://code.claude.com/docs/en/accessibility)**。

## 报告问题

如果屏幕阅读器或终端出现问题，请在 **[Claude Code GitHub 存储库](https://github.com/anthropics/claude-code/issues)**中打开问题，并在标题中提及您的屏幕阅读器。请包括您的操作系统、终端应用程序以及屏幕阅读器的名称和版本。

**注意：**一个容易混淆的设置：`CLAUDE_CODE_ACCESSIBILITY=1` 是一个不同的、无关的设置——它保持终端光标对屏幕放大镜可见，不会打开屏幕阅读器模式。对于屏幕阅读器，请使用上述方法。