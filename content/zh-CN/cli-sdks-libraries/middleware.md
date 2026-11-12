---
title: SDK 中间件
url: https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/middleware
description: 拦截并修改 SDK 的请求和响应。
featureMetadata:
  supportedModels:
    - claude-fable-5-2
    - claude-mythos-5-2
    - claude-fable-5-1
    - claude-mythos-5-1
    - claude-fable-5
    - claude-mythos-5
    - claude-mythos-preview
    - claude-opus-5-5
    - claude-opus-5
    - claude-sonnet-5-5
    - claude-sonnet-5
    - claude-haiku-5-5
    - claude-haiku-4-5-20251001
    - claude-opus-4-8
    - claude-opus-4-7
    - claude-opus-4-6
    - claude-sonnet-4-6
    - claude-opus-4-5-20251101
    - claude-sonnet-4-5-20250929
  supportedPlatforms:
    Claude API: stable
    Amazon Bedrock: stable
    Amazon Bedrock (bedrock-runtime): stable
    Claude Platform on AWS: stable
    Claude Models on Google Cloud (Anthropic managed): stable
    Claude Models on Google Cloud (Google managed): stable
    Microsoft Foundry: stable
---

Anthropic SDK 提供了一个 middleware（中间件，也称 interceptor，即拦截器）钩子，会在 SDK 发送请求之前以及收到响应之后运行您的代码。可将中间件用于横切关注点，例如日志记录、自定义重试、请求注解，以及在[回退模型](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/middleware#built-in-middleware)上重试被拒绝的请求。

```mermaid
sequenceDiagram
    accTitle: How a request and its response pass through middleware
    accDescr: Your code sends the request to Middleware A. Middleware A calls next(request) to pass it to Middleware B, and Middleware B calls next(request) to pass it to the SDK core. The SDK core sends the HTTP request to the Claude API and receives the HTTP response. The response returns through Middleware B, then Middleware A, to your code.
    autonumber
    participant App as Your code
    participant M1 as Middleware A
    participant M2 as Middleware B
    participant Core as SDK core
    participant API as Claude API
    App->>M1: request
    M1->>M2: next(request)
    M2->>Core: next(request)
    Core->>API: HTTP request
    API-->>Core: HTTP response
    Core-->>M2: response
    M2-->>M1: response
    M1-->>App: response
```

每个中间件都可以在调用 `next()` 之前检查或替换请求，并在 `next()` 返回之后检查或替换响应。

## 注册中间件

每个中间件都是一个函数，它接收传出的请求和下一个处理程序。调用 `call_next(request)` (typescript: `next(request)`; csharp: `next(request, cancellationToken)`; go: `next(req)`; java: `nextClient.execute(request, requestOptions)`; php: `$next($request)`; ruby: `call_next.call(request)`) 将请求转发到链中的其余部分（如果这是最后一个中间件，则直接转发到 SDK 核心），并返回其响应。该调用之前的任何代码在请求发出时运行；之后的任何代码在响应返回时运行。

<CodeGroup exclude="shell">
  ```python Python
  def logging_middleware(request: APIRequest, call_next: CallNext) -> APIResponse[Any]:
      # 请求之前
      print(f"-> {request.method} {request.url}")

      # 将请求转发给链的其余部分
      response = call_next(request)

      # 请求之后
      print(f"<- {response.status_code}")

      return response


  client = Anthropic(middleware=[logging_middleware])
  ```

  ```typescript TypeScript
  import type { Middleware } from "@anthropic-ai/sdk";

  const loggingMiddleware: Middleware = async (request, next, ctx) => {
    // 请求之前
    ctx.logger.debug("->", request.method, request.url);

    // 将请求转发给链的其余部分
    const response = await next(request);

    // 请求之后
    ctx.logger.debug("<-", response.status, request.url);

    return response;
  };

  const client = new Anthropic({ middleware: [loggingMiddleware] });
  ```

  ```csharp C#
  AnthropicClient client = new()
  {
      Handlers =
      [
          Handler.Create(async (request, next, cancellationToken) =>
          {
              // 请求之前
              Console.WriteLine($"Sending {request.Method} {request.RequestUri}");

              // 将请求转发给下一个处理程序
              var response = await next(request, cancellationToken);

              // 请求之后
              Console.WriteLine($"Received {(int)response.StatusCode}");

              return response;
          }),
      ],
  };
  ```

  ```go Go
  client := anthropic.NewClient(
  	option.WithMiddleware(func(req *http.Request, next option.MiddlewareNext) (*http.Response, error) {
  		// 请求之前
  		start := time.Now()
  		slog.Info("sending request", "method", req.Method, "url", req.URL)

  		// 将请求转发给链的其余部分
  		res, err := next(req)
  		if err != nil {
  			return nil, err
  		}

  		// 请求之后
  		slog.Info("received response", "status", res.StatusCode, "duration", time.Since(start))

  		return res, nil
  	}),
  )
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.builder()
      .fromEnv()
      .addInterceptor(Interceptor.syncOnly((nextClient, request, requestOptions) -> {
          // 请求之前
          IO.println(request.method() + " /" + String.join("/", request.pathSegments()));

          // 将请求转发给下一个处理程序
          HttpResponse response = nextClient.execute(request, requestOptions);

          // 请求之后
          IO.println(response.statusCode());

          return response;
      }))
      .build();
  ```

  <CodeGroupItem>
    ```php PHP
    $loggingMiddleware = function (RequestInterface $request, callable $next): ResponseInterface {
        // 请求之前
        error_log("-> {$request->getMethod()} {$request->getUri()}");

        // 将请求转发给链的其余部分
        $response = $next($request);

        // 请求之后
        error_log("<- {$response->getStatusCode()}");

        return $response;
    };

    $client = new Client(requestOptions: ['middleware' => [$loggingMiddleware]]);
    ```

    有关 middleware（中间件）类、请求选项参数，以及中间件如何与重试、错误和请求签名交互，请参阅 [PHP SDK 中的中间件](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/php#middleware)。
  </CodeGroupItem>

  ```ruby Ruby
  logging_middleware = lambda do |request, call_next|
    # 请求之前
    puts "-> #{request.method.upcase} #{request.url}"

    # 将请求转发给链的其余部分
    response = call_next.call(request)

    # 请求之后
    puts "<- #{response.status}"

    response
  end

  client = Anthropic::Client.new(middleware: [logging_middleware])
  ```
</CodeGroup>

## 中间件顺序

当您注册多个中间件时，它们按给定的顺序应用：第一个中间件的"前置"代码最先运行，其"后置"代码最后运行。在客户端上注册的中间件先于作为单次请求选项传入的中间件运行。

要注册多个中间件，请将它们全部传入，例如 `middleware=[first, second]` (typescript, ruby: `middleware: [first, second]`; csharp: `Handlers = [first, second]`; go: `option.WithMiddleware(first, second)`; java: `.addInterceptor(first).addInterceptor(second)`; php: `'middleware' => [$first, $second]`)。

## 替换 HTTP 客户端

SDK 还接受自定义 HTTP 客户端（用于代理配置、自定义 TLS 或连接池）。每个 SDK 客户端只使用一个 HTTP 客户端；设置它会替换默认客户端。自定义 HTTP 客户端在所有中间件运行完毕之后接收请求。

## 内置中间件

SDK 附带一个拒绝回退中间件，当 Claude 拒绝某个请求（`stop_reason: "refusal"`）时，它会自动在回退模型上重试该请求。[拒绝与回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)列出了其安全分类器可能拒绝请求的模型，[检测并在回退模型上重试](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#client-side-fallback)介绍了设置方法以及各语言的示例。

## 可用性

SDK 中间件可用或处于 beta 阶段，具体取决于API。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态 | 云平台     |
| -- | ------- |
| 可用 | 所有支持的平台 |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用
* 用法：在您的 SDK 客户端对 Managed Agents API 的调用上运行。它看不到智能体在其循环内发出的模型调用。

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |
