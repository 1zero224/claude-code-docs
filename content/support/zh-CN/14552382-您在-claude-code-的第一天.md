# 您在 Claude Code 的第一天

**目标：**在大约 15 分钟内，您将打开 Claude Code、登录并完成您的第一个 AI 辅助更改。本指南仅涵盖您第一天需要的内容；更深入的材料链接出现在末尾。

---

## 第 1 步：打开 Claude Code（2 分钟）

**推荐：**下载 **Claude Desktop**，使用您的付费计划登录，然后打开"Code"选项卡。Claude Desktop 包含 Claude Code，因此无需安装其他任何内容。按照 Claude Code 文档中的 **desktop quickstart** 进行第一次会话，然后跳到第 3 步。

要改用终端，请选择与您的计算机匹配的命令并将其粘贴到终端中。

| **您的设置**             | **命令**                                     |
| -------------------- | ------------------------------------------ |
| macOS / Linux / WSL  | `curl -fsSL <https://claude.ai/install.sh> |
| Windows (PowerShell) | `irm <https://claude.ai/install.ps1>       |
| Homebrew             | `brew install --cask claude-code`          |
| npm (Node 18+)       | `npm install -g @anthropic-ai/claude-code` |

**提示：**不要使用 `sudo` 运行 npm install，因为这会导致后续文件权限问题。

验证安装：

```
claude --version
```

---

## 第 2 步：登录（2 分钟）

如果您使用的是 Claude Desktop，您已经登录，Claude Code 使用相同的账户。跳到第 3 步。

在终端中，您的身份验证方式取决于您的组织如何配置访问权限。选择以下其中一项：

| **如果您的组织使用…**            | **执行此操作**                                                        |
| ------------------------ | ---------------------------------------------------------------- |
| 付费 Claude 计划             | 运行 `claude`，然后运行 `/login` 并在浏览器中批准请求（如果提示，包括 SSO）。这是最常见的方式。      |
| Anthropic Console API 密钥 | 运行 `export ANTHROPIC_API_KEY=sk-ant-…`，然后运行 `claude`。            |
| Amazon Bedrock           | 运行 `export CLAUDE_CODE_USE_BEDROCK=1`，配置您的 AWS 凭证，然后运行 `claude`。 |
| Google Vertex AI         | 运行 `export CLAUDE_CODE_USE_VERTEX=1`，配置您的 GCP 凭证，然后运行 `claude`。  |

如果您不确定适用哪种方法，请咨询您的管理员或先尝试 `/login`，这是大多数组织的默认方式。

---

## 第 3 步：打开项目（1 分钟）

在 Claude Desktop 中，在"Code"选项卡中启动新会话，然后选择要处理的存储库的文件夹。在终端中，从存储库内启动 Claude Code：

```
cd path/to/your/project
claude
```

Claude Code 按需读取您的文件，因此无需上传或附加任何内容。

---

## 第 4 步：了解唯一的安全规则（1 分钟）

Claude 将 **始终在更改文件或运行命令前询问。**当它提议更改时，您将看到一个 diff 和三个选择：

- **是** — 应用此单个更改。

- **是，以后不再询问编辑** — 在本次会话的剩余时间内自动批准文件编辑。

- **否** — 拒绝更改并解释原因。

您可以随时按 **Shift+Tab** 在模式之间循环（Plan → Accept Edits → default）。如果更改产生意外结果，您的 git 历史记录在您提交之前保持不变，因此 `git checkout` 将撤销它。

---

## 第 5 步：首先尝试的五件事

将这些逐个复制并粘贴到提示中。每个示例演示了一个不同的核心功能。

### 1. 了解情况

```
Give me a 5-bullet summary of what this codebase does and where the entry point is.
```

*为什么这很有用：*它展示了 Claude 如何自行探索文件，而无需指向特定路径。

### 2. 查找内容

```
Where is user authentication handled? Show me the file and the key function.
```

*为什么这很有用：*当您不知道确切的符号或文件名时，它通常比 `grep` 更快。

### 3. 进行安全编辑

```
Add a docstring to the function <name> in <file>. Keep it to 2 lines.
```

*为什么这很有用：*它让您可以在低风险更改上练习审查和批准 diff。

### 4. 修复真实问题

```
This test is failing: <paste the error>. Find the cause and fix it.
```

*为什么这很有用：*这是主要工作流程 — 您描述症状，Claude 调查并提议修复。

### 5. 让它处理 git

```
Stage my changes and write a commit message that follows our existing style.
```

*为什么这很有用：*Claude 可以代表您运行 `git`（需要批准），并将匹配您存储库的现有提交约定。

---

## 第 6 步：在您结束今天的工作之前

每个项目运行一次：

```
/init
```

这会在您的项目根目录生成一个 `CLAUDE.md` 文件，用于捕获您的代码库约定。Claude 在每次会话开始时会自动读取它，因此未来的响应将已针对您的项目进行定制。这是您可以采取的最高价值的设置步骤。

---

## 后续步骤

您现在已打开 Claude Code、登录、进行了编辑并设置了项目记忆。当您准备进一步了解时，官方文档深入涵盖了 **[memory 和 CLAUDE.md](https://code.claude.com/docs/en/memory)**、**[built-in commands](https://code.claude.com/docs/en/commands)** 和 **[interactive-mode shortcuts](https://code.claude.com/docs/en/interactive-mode)**。

如果您在安装或登录期间遇到问题，请参阅 **[troubleshooting guide](https://code.claude.com/docs/en/troubleshooting)**。