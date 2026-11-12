---
title: 常见用例指南
url: https://platform.claude.com/docs/zh-CN/about-claude/use-case-guides/overview
description: 探索用于构建常见 Claude 用例的生产指南：工单路由、客户支持智能体、内容审核、法律文书摘要和商务智能体。
---

Claude 旨在出色地完成各种任务。探索这些深入的生产指南，了解如何使用 Claude 构建常见用例。

<CardGroup cols={2}>
  <Card title="工单路由" icon="headset" href="https://platform.claude.com/docs/zh-CN/about-claude/use-case-guides/ticket-routing">
    使用 Claude 大规模分类和路由客户支持工单的最佳实践。
  </Card>

  <Card title="客户支持智能体" icon="robot" href="https://platform.claude.com/docs/zh-CN/about-claude/use-case-guides/customer-support-chat">
    使用 Claude 构建智能的、具备上下文感知能力的聊天机器人，以提升客户支持互动体验。
  </Card>

  <Card title="内容审核" icon="verified" href="https://platform.claude.com/docs/zh-CN/about-claude/use-case-guides/content-moderation">
    使用 Claude 执行内容过滤和一般内容审核的技术与最佳实践。
  </Card>

  <Card title="法律文书摘要" icon="book" href="https://platform.claude.com/docs/zh-CN/about-claude/use-case-guides/legal-summarization">
    使用 Claude 总结法律文件，以提取关键信息并加快研究进度。
  </Card>

  <Card title="商务智能体" icon="building" href="https://platform.claude.com/docs/zh-CN/about-claude/use-case-guides/commerce-agents">
    基于一个开源蓝图构建购物和商家智能体，该蓝图可在 Messages API、Claude Agent SDK 和 Claude Managed Agents 上运行。
  </Card>
</CardGroup>

## 选择构建路径

使用 Claude 进行构建的三种路径，区别在于您保留多少控制权，以及将多少实现工作交给 Anthropic。[Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages) 为您提供最大的控制权：您需要自行编写 “agent loop”（智能体循环），并运行您自己的工具和基础设施。[Claude Agent SDK](https://code.claude.com/docs/zh-CN/agent-sdk/overview) 介于两者之间，在由您运维的进程中提供智能体循环和工具执行。使用 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)，您可以交出最多的工作：Anthropic 会为您托管智能体循环、工具执行和运行时。
