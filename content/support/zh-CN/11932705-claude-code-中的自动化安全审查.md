# Claude Code 中的自动化安全审查

Claude Code 现已包含自动化安全审查功能，帮助您识别和修复代码中的漏洞。本指南说明如何使用 `/security-review` 命令和 GitHub Actions 来改进代码安全性。

**注意：**虽然自动化安全审查有助于识别许多常见漏洞，但它应该补充而非替代您现有的安全实践和手动代码审查。

## 概述

Claude Code 中的自动化安全审查帮助开发人员在漏洞到达生产环境之前捕获它们。这些功能检查常见的安全问题，包括 SQL 注入风险、跨站脚本 (XSS) 漏洞、身份验证缺陷、不安全的数据处理和依赖项漏洞。

您可以通过两种方式使用安全审查：在 Claude Code 会话中通过 `/security-review` 命令进行按需检查，或通过 GitHub Actions 自动审查拉取请求。

## 可用性

这些功能可供所有 Claude Code 用户使用，包括：

- 使用个人付费计划（Pro、Max 或 Ultra）的用户。

- 拥有按使用量付费 API 控制台账户的个人用户或企业。

## 使用 /security-review 命令

`/security-review` 命令让您在提交代码前直接从 Claude Code 运行安全分析。

### 运行安全审查

要检查代码中的漏洞：

1. 在您的项目中打开 Claude Code 会话，在 Claude Desktop 的"Code"标签中或在终端中的项目目录中运行 `claude`。

2. 运行 `/security-review`。

3. Claude 将分析您的代码库并识别潜在的安全问题。

4. 查看为找到的每个问题提供的详细说明。

### 实施修复

Claude 识别漏洞后，您可以要求它直接实施修复。这使安全审查集成到您的开发工作流中，让您能够在问题最容易解决时解决它们。

### 自定义命令

您可以根据特定需求自定义 `/security-review` 命令。有关配置选项，请参阅[安全审查文档](https://github.com/anthropics/claude-code-security-review/tree/main?tab=readme-ov-file#security-review-slash-command)。

## 为自动化 PR 审查设置 GitHub Actions

安装并配置 GitHub action 后，它将在打开拉取请求时自动审查其中的安全漏洞。

### 安装

要为您的存储库设置自动化安全审查：

1. 导航到您的存储库的 GitHub Actions 设置

2. 按照我们文档中的[分步安装指南](https://github.com/anthropics/claude-code-security-review)进行操作

3. 根据您团队的安全要求配置该 action

### 工作原理

配置后，GitHub action 将：

- 在打开新的拉取请求时自动触发。

- 审查代码更改中的安全漏洞。

- 应用可自定义的过滤规则以减少误报。

- 在 PR 上发布内联注释，说明识别的问题和建议的修复。

这在整个团队中创建了一致的安全审查流程，确保在合并前检查代码中的漏洞。

### 自定义选项

您可以自定义 GitHub action 以匹配您团队的安全策略，包括为代码库设置特定规则和调整不同漏洞类型的敏感度级别。

## 可以检测哪些安全问题？

`/security-review` 命令和 GitHub action 都检查常见的漏洞模式：

- **SQL 注入风险**：识别潜在的数据库查询漏洞。

- **跨站脚本 (XSS)**：检测客户端脚本注入漏洞。

- **身份验证和授权缺陷**：查找访问控制问题。

- **不安全的数据处理**：识别数据验证和清理问题。

- **依赖项漏洞**：检查第三方包中的已知问题。

## 开始使用

要开始使用自动化安全审查：

- **对于 `/security-review` 命令**：将 Claude Code 更新到最新版本（运行），然后在您的项目目录中运行 `/security-review`。

  - Claude Code 会自动保持最新状态以确保您拥有最新功能和安全修复，但您也可以运行 `claude update` 来手动更新。

- **对于 GitHub actions**：访问我们的[文档](https://github.com/anthropics/claude-code-security-review)获取安装和配置说明。

## 最佳实践

为获得最佳效果，我们建议在提交重大更改前运行 `/security-review`，并为所有包含生产代码的存储库配置 GitHub action。考虑根据您团队的特定安全要求和代码库特性调整过滤规则。