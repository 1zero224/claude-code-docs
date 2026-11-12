# 我们对 Claude API 速率限制的方法

您的速率限制取决于您的使用层级，目前通过三个关键指标进行衡量：

1. 每分钟请求数 (RPM)

2. 每分钟输入令牌数 (ITPM)

3. 每分钟输出令牌数 (OTPM)

如果您超过任何这些速率限制，您将收到一个 429 错误，说明超过了哪个速率限制，以及一个 `retry-after` 标头，指示需要等待多长时间。

速率限制在组织级别设置，取决于您的使用层级。有三个标准使用层级：Start、Build 和 Scale。新组织和使用历史有限的组织可能从 Evaluation 层级开始，其限制低于标准层级，直到建立账户历史。限制由账户团队管理的账户位于单独的 Custom 层级。更高的层级具有更高的速率限制。

您可以在 **[Claude Console](https://platform.claude.com)** 中查看您的组织当前的层级和限制。如果您需要更高的限制，一旦您使用了当前限制的至少 50%，您可以在那里请求。

有关使用层级和速率限制的更多信息，请参阅 Claude Platform Docs 中的 **[速率限制](https://platform.claude.com/docs/en/api/rate-limits)**。