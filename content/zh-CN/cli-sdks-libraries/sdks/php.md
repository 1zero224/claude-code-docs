---
title: PHP SDK
url: https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/php
description: 安装和配置 Anthropic PHP SDK，使用值对象和构建器模式
---

Anthropic PHP 库为任何 PHP 8.1.0+ 应用程序提供了便捷访问 Claude API 的方式。

<Info>
  有关包含代码示例的 API 功能文档，请参阅 [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)。本页介绍 PHP 特定的 SDK 功能和配置。
</Info>

## 安装

该 SDK 使用 [PSR-18](https://www.php-fig.org/psr/psr-18/) 处理 HTTP，并会自动发现任何已安装的 PSR-18 客户端。推荐使用 [Guzzle](https://docs.guzzlephp.org/)，因为 SDK 会自动将其配置为支持流式传输，无需额外设置：

```bash
composer require "anthropic-ai/sdk" "guzzlehttp/guzzle:^7"
```

## 要求

PHP 8.1.0 或更高版本。

## 用法

该库使用命名参数来指定可选参数。具有默认值的参数必须按名称设置。

```php
$client = new Client();

$message = $client->messages->create(
    maxTokens: 1024,
    messages: [['role' => 'user', 'content' => 'Hello, Claude']],
    model: 'claude-opus-5-5',
);

$textBlock = array_find($message->content, static fn ($block): bool => $block->type === 'text');
echo $textBlock->text;
```

有关包括 Workload Identity Federation（工作负载身份联合）在内的身份验证选项，请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)。如果您的 API 密钥是未限定到某个工作区的[个人或服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)，请在 `anthropic-workspace-id` 请求标头中设置工作区 ID；[选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)介绍了此 SDK 的按请求设置选项。

## 值对象

初始化值对象的推荐方式是使用带命名参数的静态 `with` 构造函数 `Base64ImageSource::with(data: 'U3RhaW5sZXNzIHJvY2tz', ...)`。

不过，也提供了构建器（builder），例如 `(new Base64ImageSource())->withData('U3RhaW5sZXNzIHJvY2tz')`。

## 流式传输

“Streaming”（流式传输）：SDK 支持使用 “Server-Sent Events”（服务器发送事件），即 SSE，来流式传输响应。

```php
$client = new Client();

$stream = $client->messages->createStream(
    maxTokens: 1024,
    messages: [['role' => 'user', 'content' => 'Hello, Claude']],
    model: Model::CLAUDE_OPUS_5_5,
);

foreach ($stream as $event) {
    echo $event->type . PHP_EOL;
}
```

流式传输需要一个能够增量返回响应体的 HTTP 客户端。当发现的 PSR-18 客户端是 Guzzle 时，SDK 会自动将其配置为支持流式传输。如果使用缓冲型客户端，`foreach` 循环会在响应完成时一次性产出所有事件，而不是增量产出；如果您观察到这种现象，请安装 Guzzle，或通过 `streamingTransporter` 请求选项提供一个支持流式传输的 PSR-18 客户端：

```php
$client = new Client(
    requestOptions: RequestOptions::with(streamingTransporter: $myStreamingClient),
);
```

## 错误处理

当库无法连接到 API，或者 API 返回非成功状态码（即 4xx 或 5xx 响应）时，会抛出 `Anthropic\Core\Exceptions\APIException` 的子类：

```php
<?php
// ...
use Anthropic\Core\Exceptions\APIConnectionException;
use Anthropic\Core\Exceptions\APIStatusException;
use Anthropic\Core\Exceptions\RateLimitException;
use Anthropic\Messages\Model;
// ...
try {
    $message = $client->messages->create(
        maxTokens: 1024,
        messages: [['role' => 'user', 'content' => 'Hello, Claude']],
        model: Model::CLAUDE_OPUS_5_5,
    );
} catch (APIConnectionException $e) {
    echo 'The server could not be reached', PHP_EOL;
    echo $e->getPrevious()?->getMessage(), PHP_EOL;
} catch (RateLimitException $_) {
    echo 'A 429 status code was received; we should back off a bit.', PHP_EOL;
} catch (APIStatusException $e) {
    echo 'Another non-200-range status code was received', PHP_EOL;
    echo $e->getMessage();
}
```

错误代码如下：

| 原因          | 错误类型                           |
| ----------- | ------------------------------ |
| HTTP 400    | `BadRequestException`          |
| HTTP 401    | `AuthenticationException`      |
| HTTP 403    | `PermissionDeniedException`    |
| HTTP 404    | `NotFoundException`            |
| HTTP 409    | `ConflictException`            |
| HTTP 422    | `UnprocessableEntityException` |
| HTTP 429    | `RateLimitException`           |
| HTTP >= 500 | `InternalServerException`      |
| 其他 HTTP 错误  | `APIStatusException`           |
| 超时          | `APITimeoutException`          |
| 网络错误        | `APIConnectionException`       |

## 重试

默认情况下，某些错误会自动重试两次，并采用短暂的指数退避。

连接错误（例如由于网络连接问题）、408 Request Timeout、409 Conflict、429 Rate Limit、>=500 内部错误以及超时，默认都会重试。

您可以使用 `maxRetries` 选项来配置或禁用此行为：

```php
use Anthropic\Messages\Model;
use Anthropic\RequestOptions;
// ...
// 为所有请求配置默认值：
$client = new Client(requestOptions: RequestOptions::with(maxRetries: 0));

// 或者，按请求单独配置：
$result = $client->messages->create(
    maxTokens: 1024,
    messages: [['role' => 'user', 'content' => 'Hello, Claude']],
    model: Model::CLAUDE_OPUS_5_5,
    requestOptions: RequestOptions::with(maxRetries: 5),
);
```

## 分页

Claude API 中的列表方法是分页的。

该库为每个列表响应提供自动分页迭代器，因此您无需手动请求后续页面：

```php
$client = new Client();

$page = $client->beta->messages->batches->list(limit: 20);

// 获取当前页的条目
foreach ($page->getItems() as $item) {
    echo $item->id, PHP_EOL;
}
// 发起额外的网络请求，获取当前页及其后所有页的条目
// ...
foreach ($page->pagingEachItem() as $item) {
    echo $item->id, PHP_EOL;
// ...
}
```

## 高级用法

### OpenTelemetry 追踪

当您的应用程序注册 OpenTelemetry tracer provider 时，SDK 会为其发送的每个请求发出一个 span。要将这些 span 发送到您的可观测性平台，请参阅[在 Anthropic SDK 中设置 OpenTelemetry](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)。

### 未记录的属性

您可以向任何端点发送未记录的参数，并读取未记录的响应属性，如下所示：

<Note>
  同名的 `extra*` 参数会覆盖已记录的参数。
</Note>

```php
<?php
// ...
use Anthropic\RequestOptions;
// ...
$message = $client->messages->create(
    maxTokens: 1024,
    messages: [['role' => 'user', 'content' => 'Hello, Claude']],
    model: 'claude-opus-5-5',
    requestOptions: RequestOptions::with(
        extraQueryParams: ['my_query_parameter' => 'value'],
        extraBodyParams: ['my_body_parameter' => 'value'],
        extraHeaders: ['my-header' => 'value'],
    ),
);
```

### 未记录的请求参数

如果您想显式发送额外参数，可以在发出请求时使用 `RequestOptions::with()` 下的 `extraQueryParams`、`extraBodyParams` 和 `extraHeaders` 选项，如前面的示例所示。

### 未记录的端点

要向未记录的端点发出请求，同时保留身份验证、重试和其他客户端功能的优势，您可以使用 `client->request` 发出请求，如下所示：

```php
$client = new Client();

$response = $client->request(
    method: 'post',
    path: '/undocumented/endpoint',
    query: ['dog' => 'woof'],
    headers: ['useful-header' => 'interesting-value'],
    body: ['hello' => 'world']
);
```

### 中间件

“Middleware”（中间件）会在客户端发送的每个 HTTP 请求前后运行您的代码。[SDK 中间件](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/middleware)介绍了如何注册中间件以及多个中间件的运行顺序。本节介绍 PHP SDK 特有的细节。

中间件是一个接收请求和 `$next` 的可调用对象，或者是一个实现了 `Anthropic\Middleware` 的类的实例，其 `handle()` 方法具有相同的签名。`middleware` 请求选项同时接受这两种形式：

```php
use Anthropic\Middleware;
use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;

final class TracingMiddleware implements Middleware
{
    public function handle(RequestInterface $request, \Closure $next): ResponseInterface
    {
        return $next($request->withHeader('x-trace-id', bin2hex(random_bytes(8))));
    }
}

$client = new Client(requestOptions: ['middleware' => [new TracingMiddleware()]]);
```

`$next` 会运行链中的其余部分并发送请求。调用一次即可转发请求；跳过调用则可返回您自己的响应（例如缓存的响应）；多次调用则可发送后续请求（例如自定义重试）。

SDK 还会向每个中间件传递第三个参数：该请求的 `Anthropic\RequestOptions`，它合并了客户端的选项和本次调用的选项。要读取诸如 `fallbackState` 之类的选项，请声明一个可选的第三个参数 `?RequestOptions $options = null`。请将该对象视为只读，因为每个中间件以及该请求的每次重试都共享它。如果您的可调用对象已经有一个用于其他用途的第三个参数，请重写它，因为 SDK 在每次调用时都会填充该参数。

运行时：

* **重试：** 中间件在 SDK 的重试循环内部为每次 HTTP 尝试运行一次，因此每次重试时都会再次运行。
* **错误响应：** 对于每个 HTTP 状态，`$next` 都会返回一个响应。SDK 仅在整个链返回之后才抛出其[类型化异常](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/php#error-handling)，因此中间件可以检查或替换 4xx 或 5xx 响应。
* **异常：** 中间件抛出的异常会原样到达您的代码，但有两种情况例外。SDK 会将 PSR-18 `ClientExceptionInterface` 作为连接错误处理，并以 `APIConnectionException` 的形式呈现。在出现 `Anthropic\Core\Exceptions\RetryableException` 后，SDK 会重试该尝试，并在没有剩余重试次数时原样抛出该异常。
* **身份验证：** SDK 在最内层的 `$next` 中刷新 OAuth 令牌，并对 Amazon Bedrock 和 Google Cloud 请求进行签名或授权。因此，您的中间件修改过的请求会在修改之后被签名，并且每次调用 `$next` 时都会重新签名。
* **响应体：** 读取响应体的中间件必须将其倒回（rewind），或者返回带有新响应体的 `$response->withBody(...)`，以便 SDK 仍能对其进行解码。流式响应体无法倒回，因此请不要读取，直接传递。

## 平台集成

<Note>
  有关包含代码示例的详细平台设置指南，请参阅：

  * [Amazon Bedrock（`bedrock-mantle`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)
  * [Amazon Bedrock（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)
  * [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)
  * [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)
  * [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)

  另请参阅 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)。
</Note>

PHP SDK 支持以下平台：

* **Claude Models on Google Cloud (Google managed)：** `Anthropic\Vertex\Client`。使用 `::fromEnvironment()`。
* **Amazon Bedrock（`bedrock-mantle`）：** `Anthropic\Bedrock\MantleClient`。使用 `new MantleClient(awsRegion: ...)`。
* **Amazon Bedrock（`bedrock-runtime`）：** `Anthropic\Bedrock\Client`，用于 `InvokeModel` API。使用 `::fromEnvironment()` 或 `::withCredentials()`。
* **Claude Platform on AWS：** `Anthropic\Aws\Client`（需要 `aws/aws-sdk-php` 作为软依赖）。使用 `new Anthropic\Aws\Client(workspaceId: ...)` 或设置 `ANTHROPIC_AWS_WORKSPACE_ID`。以 beta 形式提供。
* **Foundry：** `Anthropic\Foundry\Client`。使用 `::withCredentials()`。

PHP SDK 还通过 `Anthropic\GoogleCloud\Client` 支持 Claude Models on Google Cloud (Anthropic managed)。此客户端目前处于 beta 阶段。`Anthropic\GoogleCloud\Client` 需要 `google/auth` 作为软依赖（soft dependency）。请使用 `new Anthropic\GoogleCloud\Client(workspaceId: ...)` 或设置 `ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID` 环境变量。请参阅[项目、位置和工作区解析](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)。

## 语义化版本

此软件包遵循 [SemVer](https://semver.org/spec/v2.0.0.html) 约定。

此软件包将对（非运行时）PHPDoc 类型定义的改进视为非破坏性变更。

对 Claude API 的每个请求都使用一个 API 版本。使用此 SDK 时，您的请求会使用 SDK 主版本发布时的当前 API 版本，因此您需要通过升级到新的主版本来迁移到新的 API 版本。版本 1.0.0 是首个使用 `2026-10-21` 的版本。有关如何升级，请参阅[迁移到 API 版本 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#upgrade)。

## 其他资源

* [GitHub 仓库](https://github.com/anthropics/anthropic-sdk-php)
* [Packagist](https://packagist.org/packages/anthropic-ai/sdk)
* [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)
* [流式传输消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)
