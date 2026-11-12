---
title: IP 地址
url: https://platform.claude.com/docs/zh-CN/api/ip-addresses
description: Anthropic 服务对入站和出站连接均使用固定 IP 地址。您可以使用这些地址配置防火墙规则，以安全访问 Claude API 和 Claude Console。这些地址不会在未通知的情况下更改。
---

<Note>
  **[Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)：** 入站端点（`aws-external-anthropic.{region}.api.aws`）解析为 AWS IP 范围。出站工具调用（MCP 连接器、网页搜索和网页抓取）源自本页列出的 Anthropic 范围。有关入站允许列表配置，请参阅 [AWS IP 地址范围](https://docs.aws.amazon.com/vpc/latest/userguide/aws-ip-ranges.html)。
</Note>

## 入站 IP 地址

这些是 Anthropic 服务接收传入连接的 IP 地址。

### IPv4

`160.79.104.0/23`

`153.61.224.0/22`

### IPv6

`2607:6bc0::/48`

## 出站 IP 地址

这些是 Anthropic 用于出站请求的稳定 IP 地址（例如，向外部服务器发起 MCP 工具调用时）。

### IPv4

`160.79.104.0/21`

`153.61.208.0/20`

### 已停用的 IP 地址

Anthropic 已不再使用以下 IP 地址。如果您之前已将这些地址加入允许列表，请将它们从防火墙规则中移除。

```text wrap
34.162.46.92/32
34.162.102.82/32
34.162.136.91/32
34.162.142.92/32
34.162.183.95/32
```
