---
title: Java SDK
url: https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java
description: 安装和配置 Anthropic Java SDK，支持构建器模式和异步操作
---

Anthropic Java SDK 为使用 Java 编写的应用程序提供了便捷的 Claude API 访问方式。它使用 “builder pattern”（构建器模式）来创建请求，并同时支持同步和异步操作。

<Info>
  有关包含代码示例的 API 功能文档，请参阅 [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)。本页介绍 Java 特有的 SDK 功能和配置。
</Info>

## 安装

<Tabs>
  <Tab title="Gradle">
    ```kotlin
    implementation("com.anthropic:anthropic-java:3.8.0")
    ```
  </Tab>

  <Tab title="Maven">
    ```xml
    <dependency>
        <groupId>com.anthropic</groupId>
        <artifactId>anthropic-java</artifactId>
        <version>3.8.0</version>
    </dependency>
    ```
  </Tab>
</Tabs>

### 附加组件

云平台集成和客户端 MCP 辅助函数以独立的附加组件构件形式发布。请将附加组件与 `anthropic-java`（提供 `AnthropicOkHttpClient`）一起安装，并为两者使用相同的版本。此示例添加客户端 MCP 辅助函数：

<Tabs>
  <Tab title="Gradle">
    ```kotlin
    implementation("com.anthropic:anthropic-java:3.8.0")
    implementation("com.anthropic:anthropic-java-mcp:3.8.0")
    ```
  </Tab>

  <Tab title="Maven">
    ```xml
    <dependency>
        <groupId>com.anthropic</groupId>
        <artifactId>anthropic-java</artifactId>
        <version>3.8.0</version>
    </dependency>
    <dependency>
        <groupId>com.anthropic</groupId>
        <artifactId>anthropic-java-mcp</artifactId>
        <version>3.8.0</version>
    </dependency>
    ```
  </Tab>
</Tabs>

如需使用其他附加组件，请将 `anthropic-java-mcp` 替换为相应的构件名称。[平台集成](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#platform-integrations)列出了每个云平台对应的构件。[客户端 MCP 辅助函数](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#client-side-mcp-helpers)介绍了 `anthropic-java-mcp` 及其所需的 Java 版本。

## 要求

此库需要 Java 8 或更高版本。

<Note>
  SDK 支持 Java 8 及更高版本。本文档中的代码示例以 [JDK 25 紧凑源文件](https://openjdk.org/jeps/512)的形式编写，使用简单的 `void main()` 入口点，并使用 `IO.println()` 进行输出。API 调用本身在所有受支持的 JDK 上都是相同的；要在较早版本上编译示例，请将 `IO.println(...)` 替换为 `System.out.println(...)`，并将代码主体放入某个类中的 `public static void main(String[] args)` 内。
</Note>

## 快速开始

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;
import com.anthropic.models.messages.Message;
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Model;

// 使用 `anthropic.apiKey`、`anthropic.authToken` 和 `anthropic.baseUrl` 系统属性进行配置
// 或使用 `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN` 和 `ANTHROPIC_BASE_URL` 环境变量进行配置
AnthropicClient client = AnthropicOkHttpClient.fromEnv();

MessageCreateParams params = MessageCreateParams.builder()
    .maxTokens(1024L)
    .addUserMessage("Hello, Claude")
    .model(Model.CLAUDE_OPUS_5_5)
    .build();

Message message = client.messages().create(params);
```

## 客户端配置

### API 密钥设置

使用系统属性或环境变量配置客户端：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

// 使用 `anthropic.apiKey`、`anthropic.authToken` 和 `anthropic.baseUrl` 系统属性进行配置
// 或使用 `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN` 和 `ANTHROPIC_BASE_URL` 环境变量进行配置
AnthropicClient client = AnthropicOkHttpClient.fromEnv();
```

或者手动配置：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

AnthropicClient client = AnthropicOkHttpClient.builder()
    .apiKey("my-anthropic-api-key")
    .build();
```

或者结合使用这两种方式：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

AnthropicClient client = AnthropicOkHttpClient.builder()
    // 使用系统属性或环境变量进行配置
    .fromEnv()
    .apiKey("my-anthropic-api-key")
    .build();
```

有关包括 Workload Identity Federation（工作负载身份联合）在内的身份验证选项，请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)。如果您的 API 密钥是未限定到某个工作区的[个人或服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)，请在 `anthropic-workspace-id` 请求标头中设置工作区 ID；[选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)介绍了此 SDK 的按请求设置选项。

### 配置选项

| Setter 方法   | 系统属性                  | 环境变量                   | 是否必需  | 默认值                           |
| ----------- | --------------------- | ---------------------- | ----- | ----------------------------- |
| `apiKey`    | `anthropic.apiKey`    | `ANTHROPIC_API_KEY`    | false | -                             |
| `authToken` | `anthropic.authToken` | `ANTHROPIC_AUTH_TOKEN` | false | -                             |
| `baseUrl`   | `anthropic.baseUrl`   | `ANTHROPIC_BASE_URL`   | true  | `"https://api.anthropic.com"` |

系统属性优先于环境变量。

<Tip>
  不要在同一个应用程序中创建多个客户端。每个客户端都有一个连接池和多个线程池，在请求之间共享它们会更高效。
</Tip>

### 修改配置

要在复用相同连接池和线程池的同时临时使用修改后的客户端配置，请在任意客户端或服务上调用 `withOptions()`：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

AnthropicClient client = AnthropicOkHttpClient.fromEnv();

AnthropicClient clientWithOptions = client.withOptions(optionsBuilder -> {
    optionsBuilder.baseUrl("https://example.com");
    optionsBuilder.maxRetries(42);
});
```

`withOptions()` 方法不会影响原始客户端或服务。

## 异步用法

默认客户端是同步的。要切换到异步执行，请调用 `async()` 方法：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;
import com.anthropic.models.messages.Message;
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Model;

AnthropicClient client = AnthropicOkHttpClient.fromEnv();

MessageCreateParams params = MessageCreateParams.builder()
    .maxTokens(1024L)
    .addUserMessage("Hello, Claude")
    .model(Model.CLAUDE_OPUS_5_5)
    .build();

CompletableFuture<Message> message = client.async().messages().create(params);
```

或者从一开始就创建异步客户端：

```java
import com.anthropic.client.AnthropicClientAsync;
import com.anthropic.client.okhttp.AnthropicOkHttpClientAsync;
import com.anthropic.models.messages.Message;
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Model;

AnthropicClientAsync client = AnthropicOkHttpClientAsync.fromEnv();

MessageCreateParams params = MessageCreateParams.builder()
    .maxTokens(1024L)
    .addUserMessage("Hello, Claude")
    .model(Model.CLAUDE_OPUS_5_5)
    .build();

CompletableFuture<Message> message = client.messages().create(params);
```

异步客户端支持与同步客户端相同的选项，只是大多数方法返回 `CompletableFuture`。

## 流式传输

SDK 定义了返回响应"块"（chunk）流的方法，每个块在到达后即可单独处理，而无需等待完整响应。

### 同步流式传输

对于同步客户端，这些流式传输方法返回 `StreamResponse`：

```java
import com.anthropic.core.http.StreamResponse;
import com.anthropic.models.messages.RawMessageStreamEvent;

try (StreamResponse<RawMessageStreamEvent> streamResponse = client.messages().createStreaming(params)) {
    streamResponse.stream().forEach(chunk -> {
        IO.println(chunk);
    });
    IO.println("No more chunks!");
}
```

### 异步流式传输

对于异步客户端，该方法返回 `AsyncStreamResponse`：

```java
import com.anthropic.core.http.AsyncStreamResponse;
import com.anthropic.models.messages.RawMessageStreamEvent;

client.async().messages().createStreaming(params).subscribe(chunk -> {
    IO.println(chunk);
});

// 如果您需要处理流的错误或完成事件
client.async().messages().createStreaming(params).subscribe(new AsyncStreamResponse.Handler<>() {
    @Override
    public void onNext(RawMessageStreamEvent chunk) {
        IO.println(chunk);
    }

    @Override
    public void onComplete(Optional<Throwable> error) {
        if (error.isPresent()) {
            IO.println("Something went wrong!");
            throw new RuntimeException(error.get());
        } else {
            IO.println("No more chunks!");
        }
    }
});

// 或者使用 futures
client.async().messages().createStreaming(params)
    .subscribe(chunk -> {
        IO.println(chunk);
    })
    .onCompleteFuture()
    .whenComplete((unused, error) -> {
        if (error != null) {
            IO.println("Something went wrong!");
            throw new RuntimeException(error);
        } else {
            IO.println("No more chunks!");
        }
    });
```

异步流式传输使用每个客户端专用的缓存线程池 `Executor` 进行流式传输，而不会阻塞当前线程。要使用其他 `Executor`：

```java
Executor executor = Executors.newFixedThreadPool(4);
client.async().messages().createStreaming(params).subscribe(
    chunk -> IO.println(chunk), executor
);
```

或者使用 `streamHandlerExecutor` 方法全局配置客户端：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

AnthropicClient client = AnthropicOkHttpClient.builder()
    .fromEnv()
    .streamHandlerExecutor(Executors.newFixedThreadPool(4))
    .build();
```

### 使用消息累加器进行流式传输

`MessageAccumulator` 可以在处理响应中的事件流时记录这些事件，并累积出一个 `Message` 对象，该对象与非流式 API 返回的对象类似。

对于同步响应，请在流管道中添加 `Stream.peek()` 调用以累积每个事件：

```java
import com.anthropic.core.http.StreamResponse;
import com.anthropic.helpers.MessageAccumulator;
import com.anthropic.models.messages.Message;
import com.anthropic.models.messages.RawMessageStreamEvent;

MessageAccumulator messageAccumulator = MessageAccumulator.create();

try (StreamResponse<RawMessageStreamEvent> streamResponse =
         client.messages().createStreaming(createParams)) {
    streamResponse.stream()
        .peek(messageAccumulator::accumulate)
        .flatMap(event -> event.contentBlockDelta().stream())
        .flatMap(deltaEvent -> deltaEvent.delta().text().stream())
        .forEach(textDelta -> IO.print(textDelta.text()));
}

Message message = messageAccumulator.message();
```

对于异步响应，请将 `MessageAccumulator` 添加到 `subscribe()` 调用中：

```java
import com.anthropic.helpers.MessageAccumulator;
import com.anthropic.models.messages.Message;

MessageAccumulator messageAccumulator = MessageAccumulator.create();

client.async().messages()
    .createStreaming(createParams)
    .subscribe(event -> messageAccumulator.accumulate(event).contentBlockDelta().stream()
        .flatMap(deltaEvent -> deltaEvent.delta().text().stream())
        .forEach(textDelta -> IO.print(textDelta.text())))
    .onCompleteFuture()
    .join();

Message message = messageAccumulator.message();
```

还提供了 `BetaMessageAccumulator`，用于累积 `BetaMessage` 对象。其使用方式与 `MessageAccumulator` 相同。

## 结构化输出

有关包含 Java 示例的完整结构化输出文档，请参阅[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)。

## 工具使用

[Claude 的工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)让您可以将外部工具和函数直接集成到 AI 模型的响应中。模型可以在适当时输出调用工具或函数的指令（包含参数），而不仅仅是生成纯文本。您为工具定义 JSON schema，模型使用这些 schema 来确定何时以及如何使用这些工具。

工具使用功能支持"strict"（严格）模式，该模式保证 AI 模型的 JSON 输出符合您在输入参数中提供的 JSON schema。

SDK 可以根据任意 Java 类的结构自动推导出工具及其参数：类名（转换为 “snake case”（蛇形命名））提供工具名称，类的字段定义工具的参数。

<Note>
  请将您的工具类声明为顶级类或 `static` 嵌套类。此要求来自 Jackson Databind 库（`com.fasterxml.jackson.databind`），SDK 使用该库将工具输入反序列化为您的类实例，而该库无法实例化非静态内部类。
</Note>

### 使用注解定义工具

```java
import com.fasterxml.jackson.annotation.JsonClassDescription;
import com.fasterxml.jackson.annotation.JsonPropertyDescription;

enum Unit {
    CELSIUS,
    FAHRENHEIT;

    public String toString() {
        return switch (this) {
            case CELSIUS -> "C";
            case FAHRENHEIT -> "F";
        };
    }

    public double fromKelvin(double temperatureK) {
        return switch (this) {
            case CELSIUS -> temperatureK - 273.15;
            case FAHRENHEIT -> (temperatureK - 273.15) * 1.8 + 32.0;
        };
    }
}

@JsonClassDescription("Get the weather in a given location")
static class GetWeather {

    @JsonPropertyDescription("The city and state, e.g. San Francisco, CA")
    public String location;

    @JsonPropertyDescription("The unit of temperature")
    public Unit unit;

    public Weather execute() {
        double temperatureK = switch (location) {
            case "San Francisco, CA" -> 300.0;
            case "New York, NY" -> 310.0;
            case "Dallas, TX" -> 305.0;
            default -> 295;
        };
        return new Weather(String.format("%.0f%s", unit.fromKelvin(temperatureK), unit));
    }
}

static class Weather {

    public String temperature;

    public Weather(String temperature) {
        this.temperature = temperature;
    }
}
```

### 调用工具

定义工具类后，使用 `MessageCreateParams.Builder.addTool(Class<T>)` 将它们添加到消息参数中，然后在 AI 模型的响应请求调用时调用它们。`BetaToolUseBlock.input(Class<T>)` 可用于将 JSON 形式的工具参数解析为您的工具定义类的实例。

调用工具后，使用 `BetaToolResultBlockParam.Builder.contentAsJson(Object)` 将工具的结果传回 AI 模型：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;
import com.anthropic.models.beta.messages.*;
import com.anthropic.models.messages.Model;

AnthropicClient client = AnthropicOkHttpClient.fromEnv();

MessageCreateParams.Builder createParamsBuilder = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_5_5)
    .maxTokens(2048)
    .addTool(GetWeather.class)
    .addUserMessage("What's the temperature in New York?");

client.beta().messages().create(createParamsBuilder.build()).content().stream()
    .flatMap(contentBlock -> contentBlock.toolUse().stream())
    .forEach(toolUseBlock -> createParamsBuilder
        // 添加一条消息，表明已请求工具使用。
        .addAssistantMessageOfContentBlockParams(
            List.of(BetaContentBlockParam.ofToolUse(BetaToolUseBlockParam.builder()
                .name(toolUseBlock.name())
                .id(toolUseBlock.id())
                .input(toolUseBlock._input())
                .build())))
        // 添加一条消息，包含所请求工具使用的结果。
        .addUserMessageOfContentBlockParams(
            List.of(BetaContentBlockParam.ofToolResult(BetaToolResultBlockParam.builder()
                .toolUseId(toolUseBlock.id())
                .contentAsJson(callTool(toolUseBlock))
                .build()))));

client.beta().messages().create(createParamsBuilder.build()).content().stream()
    .flatMap(contentBlock -> contentBlock.text().stream())
    .forEach(textBlock -> IO.println(textBlock.text()));

private static Object callTool(BetaToolUseBlock toolUseBlock) {
    if (!"get_weather".equals(toolUseBlock.name())) {
        throw new IllegalArgumentException("Unknown tool: " + toolUseBlock.name());
    }

    GetWeather tool = toolUseBlock.input(GetWeather.class);
    return tool != null ? tool.execute() : new Weather("unknown");
}
```

### 工具名称转换

工具名称派生自驼峰命名的工具类名（例如 `GetWeather`），并转换为蛇形命名（例如 `get_weather`）。当当前字符不是第一个字符、是大写字母，并且前一个字符是小写字母或后一个字符是小写字母时，即开始一个新的单词边界。例如，`MyJSONParser` 变为 `my_json_parser`，`ParseJSON` 变为 `parse_json`。可以使用 `@JsonTypeName` 注解覆盖此转换。

### 本地工具 JSON schema 验证

您可以执行本地验证，以检查从工具类推导出的 JSON schema 是否符合 Anthropic 的限制。本地验证默认启用，但可以禁用：

```java
MessageCreateParams.Builder createParamsBuilder = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_5_5)
    .maxTokens(2048)
    .addTool(GetWeather.class, JsonSchemaLocalValidation.NO)
    .addUserMessage("What's the temperature in New York?");
```

### 为工具类添加注解

您可以使用注解向 JSON schema 添加有关工具的更多信息：

* `@JsonClassDescription` - 为工具类添加描述，详细说明何时以及如何使用该工具。
* `@JsonTypeName` - 将工具名称设置为类的简单名称转换为蛇形命名之外的其他名称。
* `@JsonPropertyDescription` - 为工具参数添加详细描述。
* `@JsonIgnore` - 从为工具参数生成的 JSON schema 中排除 `public` 字段或 getter 方法。
* `@JsonProperty` - 在为工具参数生成的 JSON schema 中包含非 `public` 字段或 getter 方法。

## 消息批处理

SDK 在 `client.messages().batches()` 命名空间下提供对[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)的支持。有关如何列出批次并对其进行分页，请参阅[分页](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#pagination)。

## 文件上传

SDK 定义了通过 `MultipartField` 类接受文件的方法：

```java
import com.anthropic.core.MultipartField;
import com.anthropic.models.files.FileMetadata;
import com.anthropic.models.files.FileUploadParams;

FileUploadParams params = FileUploadParams.builder()
    .file(
        MultipartField.<InputStream>builder()
            .value(Files.newInputStream(Paths.get("/path/to/file.pdf")))
            .contentType("application/pdf")
            .build()
    )
    .build();

FileMetadata fileMetadata = client.files().upload(params);
```

或者从 `InputStream` 上传：

```java
import com.anthropic.core.MultipartField;
import com.anthropic.models.files.FileMetadata;
import com.anthropic.models.files.FileUploadParams;

FileUploadParams params = FileUploadParams.builder()
    .file(
        MultipartField.<InputStream>builder()
            .value(URI.create("https://example.com/path/to/file").toURL().openStream())
            .filename("document.pdf")
            .contentType("application/pdf")
            .build()
    )
    .build();

FileMetadata fileMetadata = client.files().upload(params);
```

或者从内存中的字节上传：

```java
import com.anthropic.core.MultipartField;
import com.anthropic.models.files.FileMetadata;
import com.anthropic.models.files.FileUploadParams;

FileUploadParams params = FileUploadParams.builder()
    .file(
        MultipartField.<InputStream>builder()
            .value(new ByteArrayInputStream("content".getBytes()))
            .filename("document.txt")
            .contentType("text/plain")
            .build()
    )
    .build();

FileMetadata fileMetadata = client.files().upload(params);
```

### 二进制响应

对于不一定解析为 JSON 的 API 响应，SDK 定义了返回二进制响应的方法：

```java
import com.anthropic.core.http.HttpResponse;

HttpResponse response = client.files().download("file_abc123");
```

要将响应内容保存到文件：

```java
import com.anthropic.core.http.HttpResponse;

try (HttpResponse response = client.files().download(params)) {
    Files.copy(
        response.body(),
        Paths.get(path),
        StandardCopyOption.REPLACE_EXISTING
    );
} catch (Exception e) {
    IO.println("Something went wrong!");
    throw new RuntimeException(e);
}
```

或者将响应内容传输到任意 `OutputStream`：

```java
import com.anthropic.core.http.HttpResponse;

try (HttpResponse response = client.files().download(params)) {
    response.body().transferTo(Files.newOutputStream(Paths.get(path)));
} catch (Exception e) {
    IO.println("Something went wrong!");
    throw new RuntimeException(e);
}
```

## 错误处理

SDK 会抛出自定义的非受检异常类型：

* `AnthropicServiceException` - HTTP 错误的基类
* `AnthropicIoException` - I/O 网络错误
* `AnthropicRetryableException` - 表示可重试失败的通用错误
* `AnthropicInvalidDataException` - 无法解释已成功解析的数据（例如，访问本应必需但 API 意外省略的属性时）
* `AnthropicException` - 所有异常的基类

### 状态码映射

| 状态码 | 异常                              |
| --- | ------------------------------- |
| 400 | `BadRequestException`           |
| 401 | `UnauthorizedException`         |
| 403 | `PermissionDeniedException`     |
| 404 | `NotFoundException`             |
| 422 | `UnprocessableEntityException`  |
| 429 | `RateLimitException`            |
| 5xx | `InternalServerException`       |
| 其他  | `UnexpectedStatusCodeException` |

在初始 HTTP 响应成功后，如果在 SSE 流式传输期间遇到错误，则会抛出 `SseException`。

```java
import com.anthropic.errors.*;

try {
    Message message = client.messages().create(params);
} catch (RateLimitException e) {
    IO.println("Rate limited, retry after: " + e.headers());
} catch (UnauthorizedException e) {
    IO.println("Invalid API key");
} catch (AnthropicServiceException e) {
    IO.println("API error: " + e.statusCode());
} catch (AnthropicIoException e) {
    IO.println("Network error: " + e.getMessage());
}
```

## 请求 ID

使用[原始响应](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#raw-response-access)时，您可以使用 `requestId()` 方法访问 `request-id` 响应头：

```java
import com.anthropic.core.http.HttpResponseFor;
import com.anthropic.models.messages.Message;

HttpResponseFor<Message> message = client.messages().withRawResponse().create(params);

Optional<String> requestId = message.requestId();
```

这可用于记录失败的请求并将其报告给 Anthropic。有关调试请求的更多信息，请参阅[请求 ID](https://platform.claude.com/docs/zh-CN/api/errors#request-id)。

## 重试

SDK 默认自动重试 2 次，请求之间采用较短的 “exponential backoff”（指数退避）。

仅以下错误类型会被重试：

* 连接错误（例如，由于网络连接问题）
* 408 Request Timeout
* 409 Conflict
* 429 Rate Limit
* 5xx Internal

API 也可能明确指示 SDK 重试或不重试某个请求。

要设置自定义重试次数，请使用 `maxRetries` 方法配置客户端：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

AnthropicClient client = AnthropicOkHttpClient.builder()
    .fromEnv()
    .maxRetries(4)
    .build();
```

## 超时

请求默认在 10 分钟后超时。

但是，对于接受 `maxTokens` 的方法，如果您指定了较大的 `maxTokens` 值并且正在进行流式传输，则默认超时将使用以下公式动态计算：

```java
Duration.ofSeconds(
    Math.min(
        60 * 60, // 1 hour max
        Math.max(
            10 * 60, // 10 minute minimum
            60 * 60 * maxTokens / 128_000
        )
    )
)
```

除非被覆盖，否则这将产生最长 60 分钟的超时，并根据 `maxTokens` 参数进行缩放。

对于非流式请求，动态超时会根据 `maxTokens` 在最短 30 秒到最长 10 分钟之间缩放。

要为每个请求设置自定义超时：

```java
import com.anthropic.models.messages.Message;

Message message = client
    .messages()
    .create(params, RequestOptions.builder()
        .timeout(Duration.ofSeconds(30))
        .build());
```

或者在客户端级别为所有方法调用配置默认值：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

AnthropicClient client = AnthropicOkHttpClient.builder()
    .fromEnv()
    .timeout(Duration.ofSeconds(30))
    .build();
```

## 长时间请求

<Warning>
  对于运行时间较长的请求，请考虑使用[流式传输](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#streaming)。
</Warning>

避免在不使用流式传输的情况下设置较大的 `maxTokens` 值。某些网络可能会在一段时间后断开空闲连接，这可能导致请求失败或[超时](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#timeouts)而未收到 Anthropic 的响应。SDK 会定期 ping API 以保持连接活跃，从而减少这些网络的影响。

如果预计非流式请求耗时超过 10 分钟，SDK 会抛出错误。使用[流式传输方法](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#streaming)或在客户端或请求级别[覆盖超时](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#timeouts)可以禁用该错误。

## 分页

SDK 提供了便捷的方式来访问分页结果，既可以一次访问一页，也可以跨所有页面逐项访问。

### 自动分页

要遍历所有页面中的所有结果，请使用 `autoPager()` 方法，该方法会根据需要自动获取更多页面。

```java
import com.anthropic.models.messages.batches.BatchListPage;
import com.anthropic.models.messages.batches.MessageBatch;

BatchListPage page = client.messages().batches().list();

// 作为 Iterable 处理
for (MessageBatch batch : page.autoPager()) {
    IO.println(batch);
}

// 作为 Stream 处理
page.autoPager()
    .stream()
    .limit(50)
    .forEach(batch -> IO.println(batch));
```

使用异步客户端时，该方法返回 `AsyncStreamResponse`：

```java
import com.anthropic.core.http.AsyncStreamResponse;
import com.anthropic.models.messages.batches.BatchListPageAsync;
import com.anthropic.models.messages.batches.MessageBatch;

CompletableFuture<BatchListPageAsync> pageFuture = client.async().messages().batches().list();

pageFuture.thenAccept(page -> page.autoPager().subscribe(batch -> {
    IO.println(batch);
}));

// 如果您需要处理流的错误或完成事件
pageFuture.thenAccept(page -> page.autoPager().subscribe(new AsyncStreamResponse.Handler<>() {
    @Override
    public void onNext(MessageBatch batch) {
        IO.println(batch);
    }

    @Override
    public void onComplete(Optional<Throwable> error) {
        if (error.isPresent()) {
            IO.println("Something went wrong!");
            throw new RuntimeException(error.get());
        } else {
            IO.println("No more!");
        }
    }
}));

// 或者使用 futures
pageFuture.thenAccept(page -> page.autoPager()
    .subscribe(batch -> {
        IO.println(batch);
    })
    .onCompleteFuture()
    .whenComplete((unused, error) -> {
        if (error != null) {
            IO.println("Something went wrong!");
            throw new RuntimeException(error);
        } else {
            IO.println("No more!");
        }
    }));
```

### 手动分页

要访问单个页面的项目并手动请求下一页：

```java
import com.anthropic.models.messages.batches.BatchListPage;
import com.anthropic.models.messages.batches.MessageBatch;

BatchListPage page = client.messages().batches().list();
while (true) {
    for (MessageBatch batch : page.items()) {
        IO.println(batch);
    }

    if (!page.hasNextPage()) {
        break;
    }

    page = page.nextPage();
}
```

## 类型系统

### 不可变性和构建器

SDK 中的每个类都有一个关联的构建器用于构造它。每个类在构造后都是不可变的。如果类有关联的构建器，那么它就有一个 `toBuilder()` 方法，可用于将其转换回构建器，以便创建修改后的副本。

```java
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Model;

MessageCreateParams params = MessageCreateParams.builder()
    .maxTokens(1024L)
    .addUserMessage("Hello, Claude")
    .model(Model.CLAUDE_OPUS_5_5)
    .build();

// 使用 toBuilder() 创建修改后的副本
MessageCreateParams modified = params.toBuilder()
    .maxTokens(2048L)
    .build();
```

由于每个类都是不可变的，修改构建器永远不会影响已构建的类实例。

### 请求和响应

要向 Claude API 发送请求，请构建某个 `Params` 类的实例并将其传递给相应的客户端方法。收到响应后，它会被反序列化为一个 Java 类的实例。

例如，`client.messages().create(...)` 接受 `MessageCreateParams` 的实例并返回 `Message` 的实例。

### 未记录的参数

要设置未记录的参数，请在任意 `Params` 类上调用 `putAdditionalHeader`、`putAdditionalQueryParam` 或 `putAdditionalBodyProperty` 方法：

```java
import com.anthropic.core.JsonValue;
import com.anthropic.models.messages.MessageCreateParams;

MessageCreateParams params = MessageCreateParams.builder()
    .putAdditionalHeader("Secret-Header", "42")
    .putAdditionalQueryParam("secret_query_param", "42")
    .putAdditionalBodyProperty("secretProperty", JsonValue.from("42"))
    .build();
```

之后可以使用 `_additionalHeaders()`、`_additionalQueryParams()` 和 `_additionalBodyProperties()` 方法在构建的对象上访问这些参数。

<Warning>
  传递给这些方法的值会覆盖传递给先前方法的值。出于安全原因，请仅对可信的输入数据使用这些方法。
</Warning>

要在嵌套的请求头、查询参数或请求体类上设置未记录的参数：

```java
import com.anthropic.core.JsonValue;
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Metadata;

MessageCreateParams params = MessageCreateParams.builder()
    .metadata(
        Metadata.builder()
            .putAdditionalProperty("secretProperty", JsonValue.from("42"))
            .build()
    )
    .build();
```

之后可以使用 `_additionalProperties()` 方法在嵌套的构建对象上访问这些属性。

要将已记录的参数或属性设置为未记录或尚不支持的值，请将 `JsonValue` 对象传递给其 setter：

```java
import com.anthropic.core.JsonValue;
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Model;

MessageCreateParams params = MessageCreateParams.builder()
    .maxTokens(JsonValue.from(3.14))
    .addUserMessage("Hello, Claude")
    .model(Model.CLAUDE_OPUS_5_5)
    .build();
```

### 创建 JsonValue

创建 `JsonValue` 最直接的方式是使用其 `from(...)` 方法：

```java
import com.anthropic.core.JsonValue;

// 创建基本类型的 JSON 值
JsonValue nullValue = JsonValue.from(null);

JsonValue booleanValue = JsonValue.from(true);

JsonValue numberValue = JsonValue.from(42);

JsonValue stringValue = JsonValue.from("Hello World!");

// 创建一个等价于 `["Hello", "World"]` 的 JSON 数组值
JsonValue arrayValue = JsonValue.from(List.of("Hello", "World"));

// 创建一个等价于 `{ "a": 1, "b": 2 }` 的 JSON 对象值
JsonValue objectValue = JsonValue.from(Map.of("a", 1, "b", 2));

// 创建一个任意嵌套的 JSON，等价于：
// { "a": [1, 2], "b": [3, 4] }
JsonValue complexValue = JsonValue.from(Map.of("a", List.of(1, 2), "b", List.of(3, 4)));
```

### 强制省略必需参数

通常，如果任何必需的参数或属性未设置，`Builder` 类的 `build` 方法将抛出 `IllegalStateException`。要强制省略必需的参数或属性，请传递 `JsonMissing`：

```java
import com.anthropic.core.JsonMissing;
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Model;

MessageCreateParams params = MessageCreateParams.builder()
    .addUserMessage("Hello, world")
    .model(Model.CLAUDE_OPUS_5_5)
    .maxTokens(JsonMissing.of())
    .build();
```

### 响应属性

要访问未记录的响应属性，请调用 `_additionalProperties()` 方法：

```java
import com.anthropic.core.JsonValue;

Map<String, JsonValue> additionalProperties = client
    .messages()
    .create(params)
    ._additionalProperties();

JsonValue secretPropertyValue = additionalProperties.get("secretProperty");

String result = secretPropertyValue.accept(new JsonValue.Visitor<>() {
    @Override
    public String visitNull() {
        return "It's null!";
    }

    @Override
    public String visitBoolean(boolean value) {
        return "It's a boolean!";
    }

    @Override
    public String visitNumber(Number value) {
        return "It's a number!";
    }

    // 其他方法包括 `visitMissing`、`visitString`、`visitArray` 和 `visitObject`
    // 每个未实现方法的默认实现都会委托给 `visitDefault`，
    // 该方法默认会抛出异常，但也可以被重写
});
```

要访问属性的原始 JSON 值，请调用其带 `_` 前缀的方法：

```java
import com.anthropic.core.JsonField;
import com.anthropic.models.messages.StopReason;

JsonField<StopReason> stopReason = client.messages().create(params)._stopReason();

if (stopReason.isMissing()) {
    // JSON 响应中不存在该属性
} else if (stopReason.isNull()) {
    // 该属性被设置为字面量 null
} else {
    // 检查值是否以字符串形式提供
    // 其他方法包括 `asNumber()` 和 `asBoolean()`。
    Optional<String> jsonString = stopReason.asString();

    // 尝试反序列化为自定义类型
    MyClass myObject = stopReason.asUnknown().orElseThrow().convert(MyClass.class);
}
```

### 响应验证

默认情况下，当 API 返回的响应与预期类型不匹配时，SDK 不会抛出异常。只有在您直接访问该属性时，它才会抛出 `AnthropicInvalidDataException`。

要预先检查响应的类型是否完全正确，请调用 `validate()`：

```java
import com.anthropic.models.messages.Message;

Message message = client.messages().create(params).validate();
```

或者按请求配置：

```java
import com.anthropic.models.messages.Message;

Message message = client
    .messages()
    .create(params, RequestOptions.builder()
        .responseValidation(true)
        .build());
```

或者在客户端级别为所有方法调用配置默认值：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

AnthropicClient client = AnthropicOkHttpClient.builder()
    .fromEnv()
    .responseValidation(true)
    .build();
```

## HTTP 客户端自定义

### 代理配置

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;
import java.net.Proxy;

AnthropicClient client = AnthropicOkHttpClient.builder()
    .fromEnv()
    .proxy(new Proxy(Proxy.Type.HTTP, new InetSocketAddress("https://example.com", 8080)))
    .build();
```

### HTTPS / SSL 配置

<Note>
  大多数应用程序不应调用这些方法，而应使用系统默认值。默认值包含一些特殊优化，如果修改了实现，这些优化可能会丢失。
</Note>

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

AnthropicClient client = AnthropicOkHttpClient.builder()
    .fromEnv()
    .sslSocketFactory(yourSSLSocketFactory)
    .trustManager(yourTrustManager)
    .hostnameVerifier(yourHostnameVerifier)
    .build();
```

### 自定义 HTTP 客户端

SDK 由三个构件组成：

* `anthropic-java-core` - 包含核心 SDK 逻辑，不依赖 OkHttp。公开 `AnthropicClient`、`AnthropicClientAsync` 及其实现类，所有这些都可以与任何 HTTP 客户端配合使用。
* `anthropic-java-client-okhttp` - 依赖 OkHttp。公开 `AnthropicOkHttpClient` 和 `AnthropicOkHttpClientAsync`。
* `anthropic-java` - 依赖并公开 `anthropic-java-core` 和 `anthropic-java-client-okhttp` 两者的 API。自身不包含任何逻辑。

这种结构允许替换 SDK 的默认 HTTP 客户端，而无需引入不必要的依赖。

#### 自定义 OkHttpClient

<Tip>
  在替换默认客户端之前，请先尝试可用的[网络选项](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#retries)。
</Tip>

要使用自定义的 `OkHttpClient`：

1. 将您的 `anthropic-java` 依赖替换为 `anthropic-java-core`。
2. 将 `anthropic-java-client-okhttp` 的 `OkHttpClient` 类复制到您的代码中并进行自定义。
3. 使用您自定义的客户端构造 `AnthropicClientImpl` 或 `AnthropicClientAsyncImpl`。

#### 完全自定义的 HTTP 客户端

要使用完全自定义的 HTTP 客户端：

1. 将您的 `anthropic-java` 依赖替换为 `anthropic-java-core`。
2. 编写一个实现 `HttpClient` 接口的类。
3. 使用您的新客户端类构造 `AnthropicClientImpl` 或 `AnthropicClientAsyncImpl`。

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

Java SDK 通过提供特定平台 `Backend` 实现的独立依赖来支持以下平台：

* **Claude Models on Google Cloud (Google managed)：** `com.anthropic:anthropic-java-vertex`：使用 `VertexBackend.builder()` 或 `VertexBackend.fromEnv()`。`fromEnv()` 读取 `CLOUD_ML_REGION` 和 `ANTHROPIC_VERTEX_PROJECT_ID`，并使用应用默认凭据（Application Default Credentials）。
* **Bedrock：** `com.anthropic:anthropic-java-bedrock`：对于 `bedrock-mantle` 端点，使用 `BedrockMantleBackend.fromEnv()` 或 `BedrockMantleBackend.builder()`；对于 `bedrock-runtime` 端点上的 `InvokeModel` API，使用 `BedrockBackend.fromEnv()` 或 `BedrockBackend.builder()`。
* **Claude Platform on AWS：** `com.anthropic:anthropic-java-aws`：使用 `AwsBackend.fromEnv()`（读取 `ANTHROPIC_AWS_WORKSPACE_ID` 以及 AWS 默认区域/凭据链）或 `AwsBackend.builder()`。以 beta 形式提供。
* **Foundry：** `com.anthropic:anthropic-java-foundry`：使用 `FoundryBackend.fromEnv()` 或 `FoundryBackend.builder()`。

Java SDK 还通过 `com.anthropic:anthropic-java-google-cloud` 依赖项中的 `GoogleCloudBackend` 支持 Claude Models on Google Cloud (Anthropic managed)。此后端目前处于 beta 阶段。请使用 `GoogleCloudBackend.builder()` 或 `GoogleCloudBackend.fromEnv()` 创建它。`fromEnv()` 会读取 `ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID`、`ANTHROPIC_GOOGLE_CLOUD_PROJECT` 以及 Google 应用默认凭据（Application Default Credentials）。请参阅[项目、位置和工作区解析](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)。

请将每个平台构件与基础 `anthropic-java` 依赖一起安装，如[附加组件](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#add-ons)中所示。每个 `Backend` 实现都通过 `AnthropicOkHttpClient.builder()` 上的 `.backend()` 传递给客户端。

## 高级用法

### 原始响应访问

要访问 HTTP 头、状态码和原始响应体，请在任何 HTTP 方法调用前加上 `withRawResponse()`：

```java
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;
import com.anthropic.core.http.Headers;
import com.anthropic.core.http.HttpResponseFor;
import com.anthropic.models.messages.Message;
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Model;

AnthropicClient client = AnthropicOkHttpClient.fromEnv();

MessageCreateParams params = MessageCreateParams.builder()
    .maxTokens(1024L)
    .addUserMessage("Hello, Claude")
    .model(Model.CLAUDE_OPUS_5_5)
    .build();

HttpResponseFor<Message> message = client.messages().withRawResponse().create(params);

int statusCode = message.statusCode();

Headers headers = message.headers();
```

如果需要，您仍然可以将响应反序列化为 Java 类的实例：

```java
import com.anthropic.models.messages.Message;

Message parsedMessage = message.parse();
```

### 日志记录

SDK 使用标准的 OkHttp 日志拦截器。

通过将 `ANTHROPIC_LOG` 环境变量设置为 `info` 来启用日志记录：

```bash
export ANTHROPIC_LOG=info
```

或者设置为 `debug` 以获得更详细的日志：

```bash
export ANTHROPIC_LOG=debug
```

<Accordion title="Jackson 兼容性">
  SDK 依赖 Jackson 进行 JSON 序列化/反序列化。它兼容 2.13.4 或更高版本，但默认依赖 2.19.4 版本。

  如果 SDK 在运行时检测到不兼容的 Jackson 版本（例如，默认版本在您的 Maven 或 Gradle 配置中被覆盖），则会抛出异常。

  如果 SDK 抛出了异常，但您确定该版本是兼容的，则可以在 `AnthropicOkHttpClient` 或 `AnthropicOkHttpClientAsync` 上使用 `checkJacksonVersionCompatibility` 禁用版本检查。

  <Warning>
    禁用 Jackson 版本检查后，无法保证 SDK 能够正常工作。
  </Warning>

  较旧的 Jackson 版本中也存在一些可能影响 SDK 的 bug。SDK 不会规避所有 Jackson bug，而是期望用户为此升级 Jackson。
</Accordion>

<Accordion title="ProGuard/R8 配置">
  尽管 SDK 使用了反射，但它仍然可以与 ProGuard 和 R8 一起使用，因为 `anthropic-java-core` 发布时附带了一个包含 keep 规则的配置文件。

  ProGuard 和 R8 应该会自动检测并使用已发布的规则，但如有必要，您也可以手动复制这些 keep 规则。
</Accordion>

### OpenTelemetry 追踪

当您的应用程序注册 OpenTelemetry tracer provider 时，SDK 会为其发送的每个请求发出一个 span。要将这些 span 发送到您的可观测性平台，请参阅[在 Anthropic SDK 中设置 OpenTelemetry](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)。

### 未记录的 API 功能

SDK 的类型设计便于使用已记录的 API。不过，它也支持使用 API 中未记录或尚不支持的部分。

#### 未记录的请求参数

要设置未记录的请求参数，请按照[未记录的参数](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#undocumented-parameters)中的说明使用 `putAdditionalHeader`、`putAdditionalQueryParam` 或 `putAdditionalBodyProperty` 方法。

#### 未记录的响应属性

要访问未记录的响应属性，请按照[响应属性](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#response-properties)中的说明使用 `_additionalProperties()` 方法。

#### 新的或未发布的枚举值

SDK 中的类枚举类（例如 `Model` 和 `AnthropicBeta`）并不是封闭的 Java `enum` 类型。每个类都提供一个接受任意字符串的 `of(String)` 工厂方法，因此您可以使用尚未添加到 SDK 中的值，例如在您的 SDK 版本之后发布的模型或 beta 头：

```java
import com.anthropic.models.beta.AnthropicBeta;
import com.anthropic.models.messages.Model;

Model model = Model.of("some-new-model");
AnthropicBeta beta = AnthropicBeta.of("some-new-beta-2026-01-01");
```

接受这些类型的构建器方法通常还提供一个 `String` 重载，它会为您调用 `of(...)`：

```java
import com.anthropic.models.messages.MessageCreateParams;

MessageCreateParams params = MessageCreateParams.builder()
    .model("some-new-model") // same as .model(Model.of("some-new-model"))
    .maxTokens(1024L)
    .addUserMessage("Hello, Claude")
    .build();
```

请优先使用类型明确的常量（例如 `Model.CLAUDE_OPUS_5`），以便获得自动补全和弃用警告。`String` 重载和 `of(...)` 主要用于在等待包含该值的 SDK 版本发布期间，将字段设置为未记录或尚不支持的值。

## Beta 功能

Beta 功能在正式发布之前提供，以便获取早期反馈并测试新功能。您可以在[功能概览](https://platform.claude.com/docs/zh-CN/build-with-claude/overview)中查看 Claude 所有功能和工具的可用性。

您可以通过客户端上的 `beta()` 方法访问大多数 beta API 功能。要启用特定的 beta 功能，请在构建消息参数时使用 `.addBeta()` 添加相应的 [beta 头](https://platform.claude.com/docs/zh-CN/api/beta-headers)。

例如，要启用[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)：

```java
import com.anthropic.models.beta.AnthropicBeta;
import com.anthropic.models.beta.messages.BetaMessage;
import com.anthropic.models.beta.messages.MessageCreateParams;
// ...
void main() {
    AnthropicClient client = AnthropicOkHttpClient.fromEnv();

    BetaMessage message = client.beta().messages().create(
        MessageCreateParams.builder()
            .model(Model.CLAUDE_OPUS_5_5)
            .maxTokens(1024L)
            .addBeta(AnthropicBeta.CONTEXT_MANAGEMENT_2025_06_27)
            .addUserMessage("Hello, Claude")
            .build());
}
```

## 常见问题

<AccordionGroup>
  <Accordion title="为什么 SDK 不使用普通的枚举类？">
    Java `enum` 类不容易实现向前兼容。如果 API 更新后返回新的枚举值，在 SDK 中使用它们可能会导致运行时异常。

    由于这些类是开放的，您还可以通过它们的 `of(String)` 工厂方法使用任意字符串值来构造它们。如果您需要使用 SDK 版本中尚未包含的值，请参阅[新的或未发布的枚举值](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/java#new-or-unreleased-enum-values)。
  </Accordion>

  <Accordion title="为什么字段使用 JsonField<T> 而不是普通的 T 来表示？">
    使用 `JsonField<T>` 可以实现以下几个功能：

    * 允许使用未记录的 API 功能
    * 根据预期结构对 API 响应进行延迟验证
    * 区分缺失值与显式的 null 值
  </Accordion>

  <Accordion title="为什么 SDK 不使用数据类？">
    向数据类添加新字段不具备向后兼容性，而 SDK 希望避免每次向类添加字段时都引入破坏性变更。
  </Accordion>

  <Accordion title="为什么 SDK 不使用受检异常？">
    受检异常被普遍认为是 Java 编程语言中的一个错误设计。事实上，Kotlin 正是出于这个原因而省略了它们。

    受检异常：

    * 处理起来很繁琐
    * 鼓励在错误的抽象层级进行错误处理，而在该层级对错误无能为力
    * 由于函数着色问题，传播起来很麻烦
    * 与 lambda 配合不佳（同样是由于函数着色问题）
  </Accordion>
</AccordionGroup>

## 语义化版本

此软件包通常遵循 [SemVer](https://semver.org/spec/v2.0.0.html) 约定，但某些向后不兼容的变更可能会作为次版本发布：

1. 对库内部的更改，这些内部在技术上是公开的，但并非供外部使用，也未针对外部使用提供文档。
2. 预计在实践中不会影响绝大多数用户的更改。

对 Claude API 的每个请求都使用一个 API 版本。使用此 SDK 时，您的请求会使用 SDK 主版本发布时的当前 API 版本，因此您需要通过升级到新的主版本来迁移到新的 API 版本。版本 3.0.0 是首个使用 `2026-10-21` 的版本。有关如何升级，请参阅[迁移到 API 版本 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#upgrade)。

## 其他资源

* [GitHub 仓库](https://github.com/anthropics/anthropic-sdk-java)
* [Javadocs](https://javadoc.io/doc/com.anthropic/anthropic-java)
* [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)
* [流式传输消息](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)
* [Claude 的工具使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview)
