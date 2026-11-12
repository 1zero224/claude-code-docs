---
title: 通过 API 触发例程
url: https://platform.claude.com/docs/zh-CN/api/claude-code/routines-fire
description: 通过发送经过身份验证的 POST 请求，按需启动 Claude Code 例程会话。
---

<Warning>
  这是一个实验性 API。请求和响应的结构、速率限制以及令牌语义可能会发生变化。
</Warning>

[Claude Code](https://code.claude.com/docs) 是 Anthropic 的智能体编码工具。[网页版 Claude Code](https://code.claude.com/docs/zh-CN/claude-code-on-the-web) 在 Anthropic 托管的云基础设施上（位于 claude.ai/code）运行 Claude Code 会话，而 “[routine](https://code.claude.com/docs/zh-CN/routines)”（例程）是保存在那里的一种配置：一个提示、一个或多个代码仓库以及连接器，打包在一起，使其可以按计划、响应 GitHub 事件或在通过 HTTP 调用时无人值守地运行。

此端点是 HTTP 入口点。向其发送 POST 请求会启动现有例程的一次新运行，并返回生成的会话 ID 和 URL。典型的调用方包括告警系统、CI 流水线以及需要以编程方式启动 Claude Code 会话的内部工具。

调用此端点需要一个订阅 Pro、Max、Ultra、Team 或 Enterprise 计划并已启用[网页版 Claude Code](https://code.claude.com/docs/zh-CN/claude-code-on-the-web) 的 claude.ai 账户。请使用在 Claude Code 网页界面中创建的每例程专属 bearer 令牌进行身份验证，而不是使用 Claude API 密钥。

## 与 Claude Platform 的区别

例程触发端点属于 Claude Code 产品层面，它与 Claude Platform 的 API 和 SDK 在以下几个方面有所不同：

| 方面     | 此端点                                                                                                                   | Claude Platform API                                                                 |
| ------ | --------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| 身份验证   | `Authorization: Bearer`，使用在 [claude.ai/code/routines](https://claude.ai/code/routines) 创建的每例程专属令牌（`sk-ant-oat01-...`） | 来自 Claude Console 的 Claude API 密钥，作为 `Authorization: Bearer` 发送                     |
| 令牌作用域  | 仅限一个例程；无读取权限                                                                                                          | 工作区级别                                                                               |
| SDK 支持 | 无                                                                                                                     | 所有[客户端 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 均支持 |
| 计费     | claude.ai 上的 Claude Code 订阅用量                                                                                         | Claude Platform 用量                                                                  |
| 路径命名空间 | `/v1/claude_code/...`                                                                                                 | `/v1/...`                                                                           |
| 稳定性    | 实验性                                                                                                                   | 稳定版或标准 beta 版                                                                       |

## 开始之前

要调用此端点，您需要：

1. 在 [claude.ai/code/routines](https://claude.ai/code/routines) 创建的一个例程。
2. 为该例程生成的 bearer 令牌：打开例程进行编辑，在**选择触发器**下点击**添加另一个触发器**，选择 **API**，然后在模态窗口中点击**生成令牌**。该令牌仅显示一次，之后无法再次获取。

完整的设置流程请参阅 Claude Code 文档中的[添加 API 触发器](https://code.claude.com/docs/zh-CN/routines#add-an-api-trigger)。

## 触发例程

```http
POST https://api.anthropic.com/v1/claude_code/routines/{routine_id}/fire
```

当您添加 API 触发器时，Claude Code 网页界面会在提供令牌的同时提供完整的 URL，因此大多数集成会将两者都存储为密钥并直接调用该端点。以下示例展示了一个 shell 调用，以及一个在 CI 失败时触发例程的 GitHub Actions 步骤。

```bash cURL
curl -X POST https://api.anthropic.com/v1/claude_code/routines/$ROUTINE_ID/fire \
  -H "Authorization: Bearer $ROUTINE_TOKEN" \
  -H "anthropic-version: 2026-10-21" \
  -H "Content-Type: application/json" \
  -d '{"text": "Sentry alert SEN-4521 fired in prod. Stack trace attached."}'
```

```yaml GitHub Actions
- if: failure()
  env:
    ROUTINE_FIRE_URL: ${{ secrets.ROUTINE_FIRE_URL }}
    ROUTINE_FIRE_TOKEN: ${{ secrets.ROUTINE_FIRE_TOKEN }}
  run: |
    curl -X POST "$ROUTINE_FIRE_URL" \
      -H "Authorization: Bearer $ROUTINE_FIRE_TOKEN" \
      -H "anthropic-version: 2026-10-21" \
      -H "Content-Type: application/json" \
      -d "{\"text\": \"CI failed: $GITHUB_WORKFLOW run $GITHUB_RUN_ID on $GITHUB_REF\"}"
```

请求在会话创建后即返回。它不会流式传输会话输出，也不会等待会话完成。

### 标头

| 名称                  | 必需     | 描述                                                                                                                       |
| ------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------ |
| `Authorization`     | 是      | `Bearer <token>`。在 Claude Code 网页界面中创建的每例程专属令牌，前缀为 `sk-ant-oat01-`。                                                      |
| `anthropic-version` | 是      | [API 版本](https://platform.claude.com/docs/zh-CN/api/versioning)，例如 `2026-10-21`。该端点接受 `2023-06-01` 及之后的所有版本，且在各版本中的行为相同。 |
| `Content-Type`      | 存在请求体时 | `application/json`。                                                                                                      |

无论是否包含 `anthropic-beta: experimental-cc-routine-2026-04-01` 标头，端点都会接受请求。

### 路径参数

| 名称           | 类型     | 描述                                                                        |
| ------------ | ------ | ------------------------------------------------------------------------- |
| `routine_id` | string | 例程的标识符。尽管参数名如此，其值的前缀是 `trig_` 而非 `routine_`。包含在您添加 API 触发器时模态窗口显示的 URL 中。 |

### 请求体

| 字段     | 类型     | 必需 | 描述                                                                                                                     |
| ------ | ------ | -- | ---------------------------------------------------------------------------------------------------------------------- |
| `text` | string | 否  | 本次运行的初始上下文，例如告警正文、失败的日志行或 git diff。该值为自由格式文本，不会被解析；如果您发送 JSON 或其他结构化负载，例程会将其作为字面字符串接收。它会与例程保存的提示一起传递给例程。最多 65,536 个字符。 |

请求体是可选的。请求体中的未知字段将被忽略。

### 响应

成功的请求返回 `200 OK` 以及新会话的详细信息：

```json
{
  "type": "routine_fire",
  "claude_code_session_id": "session_01HJKLMNOPQRSTUVWXYZ",
  "claude_code_session_url": "https://claude.ai/code/session_01HJKLMNOPQRSTUVWXYZ"
}
```

| 字段                        | 类型     | 描述                                               |
| ------------------------- | ------ | ------------------------------------------------ |
| `type`                    | string | 始终为 `routine_fire`。                              |
| `claude_code_session_id`  | string | 为本次运行创建的 Claude Code 会话的 ID。                     |
| `claude_code_session_url` | string | 指向 claude.ai 上该会话的链接。在浏览器中打开它即可观看运行过程、审查更改或继续对话。 |

### 错误

错误使用标准的 Anthropic [错误封装格式](https://platform.claude.com/docs/zh-CN/api/errors)：

```json
{
  "type": "error",
  "error": {
    "type": "not_found_error",
    "message": "<string>"
  }
}
```

| HTTP 状态码 | 错误类型                    | 原因                                                                                                                                                 |
| -------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 400      | `invalid_request_error` | 缺少 `anthropic-version` 标头或其值不受支持、`text` 超过 65,536 个字符，或例程已暂停（请参阅[编辑和控制例程](https://code.claude.com/docs/zh-CN/routines#edit-and-control-routines)）。 |
| 401      | `authentication_error`  | `Authorization` 标头中没有 bearer 令牌，或令牌与此例程不匹配。                                                                                                        |
| 403      | `permission_error`      | 该账户或组织无权访问此端点。                                                                                                                                     |
| 404      | `not_found_error`       | 该例程不存在。                                                                                                                                            |
| 429      | `rate_limit_error`      | 已达到例程或账户的每小时触发次数上限。响应中包含 `Retry-After` 标头，指示时间窗口何时重置。                                                                                              |
| 500      | `api_error`             | 意外的服务器错误。请使用指数退避进行重试；如果错误持续存在，请携带请求 ID 联系支持团队。                                                                                                     |
| 503      | `overloaded_error`      | 服务暂时过载。请稍后重试。Claude Platform 对此错误类型返回 529；此端点返回 503。                                                                                               |

## 身份验证

bearer 令牌的作用域限定为单个例程。泄露的令牌只能触发该例程；它不授予任何读取权限，不能访问其他例程，也不能访问账户数据。

请在 [claude.ai/code/routines](https://claude.ai/code/routines) 的例程 API 触发器设置中生成和撤销令牌。没有用于令牌管理的公开 API。生成新令牌会撤销之前的令牌。

## 幂等性

每个成功的请求都会创建一个新会话。没有幂等性键。如果 webhook 调用方进行重试，该端点会创建多个会话。

## 速率限制

API 触发按小时限制：每个例程每小时最多接受 30 次触发（API 触发、网页界面中的**立即运行**按钮以及一次性重新启用共用这一上限），每个账户在所有例程中每小时最多可进行 100 次 API 触发。由此产生的会话与交互式会话消耗相同的 Claude Code 订阅用量。达到限制时，端点会返回 `429 rate_limit_error` 以及 `Retry-After` 标头。

要了解例程用量如何与订阅限制和额外用量计费相互作用，请参阅 Claude Code 文档中的[用量和限制](https://code.claude.com/docs/zh-CN/routines#usage-and-limits)。

## SDK 支持

此端点未包含在 Anthropic SDK 中。其令牌模型与 API 密钥身份验证不同，并且 CI 作业和告警 webhook 等典型调用方会直接发送请求。

## 另请参阅

* Claude Code 文档中的[使用例程自动化工作](https://code.claude.com/docs/zh-CN/routines)
* [Claude API 错误](https://platform.claude.com/docs/zh-CN/api/errors)
