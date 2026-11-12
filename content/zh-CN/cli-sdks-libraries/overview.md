---
title: SDK、CLI 和库
url: https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview
description: 用于基于 Claude API 进行构建的官方工具：七种语言的客户端 SDK、ant CLI 以及特定框架的库。
---

Anthropic 提供三类官方工具，用于基于 Claude API 进行构建：

* **CLI：** `ant` 命令行工具，用于 shell 脚本编写和交互式使用。
* **客户端 SDK：** 适用于 Python、TypeScript、C#、Go、Java、PHP 和 Ruby 的通用 Messages API 客户端。它们还涵盖了 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 和大部分 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api)。每个 SDK 都提供符合语言习惯的接口、类型安全，以及对 “streaming”（流式传输）、重试、错误处理和 [OpenTelemetry 追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)的内置支持。
* **库和集成：** 在其他框架的 API 接口中公开 Claude 的软件包和兼容层，而不是直接公开 Messages API。

<Info>
  有关完整的 API 规范，请参阅 [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)。
</Info>

## CLI

<CardGroup cols={3}>
  <Card title="ant CLI" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart">
    Shell 脚本编写、类型化标志、响应转换
  </Card>
</CardGroup>

## 客户端 SDK

<CardGroup cols={3}>
  <Card title="Python" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/python">
    同步和异步客户端、Pydantic 模型
  </Card>

  <Card title="TypeScript" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/typescript">
    支持 Node.js、Deno、Bun 和浏览器
  </Card>

  <Card title="C#" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/csharp">
    .NET Standard 2.0+、IChatClient 集成
  </Card>

  <Card title="Go" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/go">
    基于 Context 的取消、函数式选项
  </Card>

  <Card title="Java" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java">
    构建器模式、CompletableFuture 异步
  </Card>

  <Card title="PHP" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/php">
    值对象、构建器模式
  </Card>

  <Card title="Ruby" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/ruby">
    Sorbet 类型、流式传输辅助工具
  </Card>
</CardGroup>

客户端 SDK 的每个主要版本默认发送一个 [API 版本](https://platform.claude.com/docs/zh-CN/api/versioning)。要迁移到新的 API 版本，请升级到新的主要版本。对于 `2026-10-21`，请参阅[切换到 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#upgrade)。

## 最新版本

| 工具         | 最新版本     | 软件包                                                                                                         |
| ---------- | -------- | ----------------------------------------------------------------------------------------------------------- |
| ant CLI    | `2.9.0`  | [`github.com/anthropics/anthropic-cli`](https://github.com/anthropics/anthropic-cli/releases)               |
| Python     | `2.8.0`  | [PyPI 上的 `anthropic`](https://pypi.org/project/anthropic/)                                                  |
| TypeScript | `1.8.0`  | [`@anthropic-ai/sdk`](https://www.npmjs.com/package/@anthropic-ai/sdk)                                      |
| C#         | `13.8.0` | [`Anthropic`](https://www.nuget.org/packages/Anthropic)                                                     |
| Go         | `2.9.0`  | [`github.com/anthropics/anthropic-sdk-go/v2`](https://pkg.go.dev/github.com/anthropics/anthropic-sdk-go/v2) |
| Java       | `3.8.0`  | [`com.anthropic:anthropic-java`](https://central.sonatype.com/artifact/com.anthropic/anthropic-java)        |
| PHP        | `1.8.0`  | [`anthropic-ai/sdk`](https://packagist.org/packages/anthropic-ai/sdk)                                       |
| Ruby       | `2.8.0`  | [RubyGems 上的 `anthropic`](https://rubygems.org/gems/anthropic)                                              |

相同的版本信息也以 JSON 格式提供，地址为 `https://platform.claude.com/docs/sdk-versions.json`。

## 库和集成

库和集成通过其他框架的 API 接口公开 Claude。它们不是通用的 Messages API 客户端。

<CardGroup cols={3}>
  <Card title="Apple Foundation Models" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/apple-foundation-models">
    适用于 Apple `LanguageModelSession` API 的 Swift 软件包
  </Card>

  <Card title="OpenAI SDK 兼容性" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/openai-sdk">
    通过 OpenAI SDK 接口使用 Claude
  </Card>
</CardGroup>

## 正在构建智能体或使用 Claude Code？

CLI、客户端 SDK 和库用于您自行调用 Claude API。您发送每个请求并处理每个响应。Claude Code、Claude Agent SDK 和 Claude Managed Agents 在更高层级上工作，提供智能体循环、工具执行和运行时。

<CardGroup cols={3}>
  <Card title="Claude Code" href="https://code.claude.com/docs/en/overview">
    用于将编码任务委托给 Claude 的智能体编码工具
  </Card>

  <Card title="Claude Agent SDK" href="https://code.claude.com/docs/en/agent-sdk/overview">
    构建在您自己运营的进程中运行的智能体
  </Card>

  <Card title="Claude Managed Agents" href="https://platform.claude.com/docs/zh-CN/managed-agents/overview">
    在 Anthropic 的托管基础设施中运行智能体
  </Card>
</CardGroup>
