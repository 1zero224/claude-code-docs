# Claude Code 模型配置

本指南展示了三种方式来更改您在 Claude Code 中使用的 Claude 模型：快速 `/model` 命令用于即时更改，`--model` 标志用于一次性会话更改，以及环境变量用于将您首选的模型设置为永久默认值。

## 最简单的方法：使用 /model 命令

更改模型的最简单方法是直接在 Claude Code 中使用 `/model` 命令。它在 Claude Desktop 和终端中都可用，并立即生效。

1. 启动 Claude Code 会话：在 Claude Desktop 中打开

2. 输入 `/model` 并从交互式菜单中选择您所需的模型。

3. 您的模型更改立即生效。

**注意：**您可以随时通过在 Claude Code 中运行 `/status` 来检查您当前的模型。

## 支持的模型

- Haiku 5.5，`claude-haiku-5-5`

- Sonnet 5.5，`claude-sonnet-5-5`

- Opus 5.5，`claude-opus-5-5`

- Fable 5.1，`claude-fable-5-1`

- Opus 5，`claude-opus-5`

- Sonnet 5，`claude-sonnet-5`

- Fable 5，`claude-fable-5`

- Opus 4.8，`claude-opus-4-8`

- Opus 4.7，`claude-opus-4-7`

- Sonnet 4.6，`claude-sonnet-4-6`

- Opus 4.6，`claude-opus-4-6`

- Opus 4.5，`claude-opus-4-5-20251101`

- Haiku 4.5，`claude-haiku-4-5-20251001`

- Sonnet 4.5，`claude-sonnet-4-5-20250929`

## 仅为当前会话更改模型

启动 Claude Code 时使用 `--model` 标志。`--model` 标志和下一部分中的环境变量适用于终端会话。

1. 启动一个新的终端会话。

2. 输入以下命令（取决于您想为该会话使用的模型）：

  - **对于 Haiku 5.5**：`claude --model claude-haiku-5-5`

  - **对于 Sonnet 5.5**：`claude --model claude-sonnet-5-5`

  - **对于 Opus 5.5**：`claude --model claude-opus-5-5`

  - **对于 Fable 5.1**：`claude --model claude-fable-5-1`

  - **对于 Opus 5**：`claude --model claude-opus-5`

  - **对于 Sonnet 5**：`claude --model claude-sonnet-5`

  - **对于 Fable 5**：`claude --model claude-fable-5`

  - **对于 Opus 4.8**：`claude --model claude-opus-4-8`

  - **对于 Opus 4.7**：`claude --model claude-opus-4-7`

  - **对于 Sonnet 4.6**：`claude --model claude-sonnet-4-6`

  - **对于 Opus 4.6**：`claude --model claude-opus-4-6`

  - **对于 Opus 4.5**：`claude --model claude-opus-4-5-20251101`

  - **对于 Haiku 4.5**：`claude --model claude-haiku-4-5-20251001`

  - **对于 Sonnet 4.5**：`claude --model claude-sonnet-4-5-20250929`

## 为所有未来会话更改默认模型

**步骤 1)** 通过运行以下命令检查您的 shell 类型：`echo $SHELL`

- `/bin/zsh` → 您正在使用 zsh（macOS 默认值）

- `/bin/bash` → 您正在使用 bash（Linux 默认值）

**步骤 2)** 将模型设置添加到您的 shell 配置：

### 对于 ZSH 用户 (macOS)

- Haiku 5.5：`echo 'export ANTHROPIC_MODEL="claude-haiku-5-5"' >> ~/.zshrc`

- Sonnet 5.5：`echo 'export ANTHROPIC_MODEL="claude-sonnet-5-5"' >> ~/.zshrc`

- Opus 5.5：`echo 'export ANTHROPIC_MODEL="claude-opus-5-5"' >> ~/.zshrc`

- Fable 5.1：`echo 'export ANTHROPIC_MODEL="claude-fable-5-1"' >> ~/.zshrc`

- Opus 5：`echo 'export ANTHROPIC_MODEL="claude-opus-5"' >> ~/.zshrc`

- Sonnet 5：`echo 'export ANTHROPIC_MODEL="claude-sonnet-5"' >> ~/.zshrc`

- Fable 5：`echo 'export ANTHROPIC_MODEL="claude-fable-5"' >> ~/.zshrc`

- Opus 4.8：`echo 'export ANTHROPIC_MODEL="claude-opus-4-8"' >> ~/.zshrc`

- Opus 4.7：`echo 'export ANTHROPIC_MODEL="claude-opus-4-7"' >> ~/.zshrc`

- Sonnet 4.6：`echo 'export ANTHROPIC_MODEL="claude-sonnet-4-6"' >> ~/.zshrc`

- Opus 4.6：`echo 'export ANTHROPIC_MODEL="claude-opus-4-6"' >> ~/.zshrc`

- Opus 4.5：`echo 'export ANTHROPIC_MODEL="claude-opus-4-5-20251101"' >> ~/.zshrc`

- Haiku 4.5：`echo 'export ANTHROPIC_MODEL="claude-haiku-4-5-20251001"' >> ~/.zshrc`

- Sonnet 4.5：`echo 'export ANTHROPIC_MODEL="claude-sonnet-4-5-20250929"' >> ~/.zshrc`

### 对于 BASH 用户 (Linux)

- Haiku 5.5：`echo 'export ANTHROPIC_MODEL="claude-haiku-5-5"' >> ~/.bashrc`

- Sonnet 5.5：`echo 'export ANTHROPIC_MODEL="claude-sonnet-5-5"' >> ~/.bashrc`

- Opus 5.5：`echo 'export ANTHROPIC_MODEL="claude-opus-5-5"' >> ~/.bashrc`

- Fable 5.1：`echo 'export ANTHROPIC_MODEL="claude-fable-5-1"' >> ~/.bashrc`

- Opus 5：`echo 'export ANTHROPIC_MODEL="claude-opus-5"' >> ~/.bashrc`

- Sonnet 5：`echo 'export ANTHROPIC_MODEL="claude-sonnet-5"' >> ~/.bashrc`

- Fable 5：`echo 'export ANTHROPIC_MODEL="claude-fable-5"' >> ~/.bashrc`

- Opus 4.8：`echo 'export ANTHROPIC_MODEL="claude-opus-4-8"' >> ~/.bashrc`

- Opus 4.7：`echo 'export ANTHROPIC_MODEL="claude-opus-4-7"' >> ~/.bashrc`

- Sonnet 4.6：`echo 'export ANTHROPIC_MODEL="claude-sonnet-4-6"' >> ~/.bashrc`

- Opus 4.6：`echo 'export ANTHROPIC_MODEL="claude-opus-4-6"' >> ~/.bashrc`

- Opus 4.5：`echo 'export ANTHROPIC_MODEL="claude-opus-4-5-20251101"' >> ~/.bashrc`

- Haiku 4.5：`echo 'export ANTHROPIC_MODEL="claude-haiku-4-5-20251001"' >> ~/.bashrc`

- Sonnet 4.5：`echo 'export ANTHROPIC_MODEL="claude-sonnet-4-5-20250929"' >> ~/.bashrc`

**步骤 3)** 应用更改：

- 对于 ZSH：`source ~/.zshrc`

- 对于 BASH：`source ~/.bashrc`

**步骤 4)** 完全关闭终端，然后重新打开。

**步骤 5)** 在新的终端会话中启动 Claude Code：`claude`。

现在您选择的模型将成为所有未来 Claude Code 会话的默认值。