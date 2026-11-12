---
title: 在 Anthropic SDK 中设置 OpenTelemetry
url: https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk
description: 追踪 Anthropic SDK 在 Python、TypeScript、C#、Go、Java、PHP 或 Ruby 中的请求和工具使用，并将 span 发送到您的可观测性平台。
featureMetadata:
  topic:
    title: OpenTelemetry
    url: https://platform.claude.com/docs/en/manage-claude/opentelemetry
  status: beta
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
    Claude API: beta
    Amazon Bedrock: beta
    Amazon Bedrock (bedrock-runtime): beta
    Claude Platform on AWS: beta
    Claude Models on Google Cloud (Google managed): beta
    Microsoft Foundry: beta
---

借助 OpenTelemetry，Anthropic SDK 可以为其发送的每个请求以及 [tool runner](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner)（工具运行器）运行的每次工具使用发出一个 “span”（跨度）。您可以在可观测性平台中查看这些 span，它们会嵌套在您自己的 “trace”（追踪）中。

SDK 在您的应用程序中发出 span。您应用程序的 OpenTelemetry “exporter”（导出器）会将它们发送到您的可观测性平台。

<Note>
  在 beta 期间，SDK 选项和 span 架构可能会以破坏您的代码、查询或仪表板的方式发生变化。例如，新版本的 SDK 可能会重命名某个 span、属性、选项或环境变量。请在依赖项中固定 SDK 版本，以便自行选择何时采用这些变更。
</Note>

## 1. 安装 SDK

安装或升级到最新版本的 Anthropic SDK：

<Tabs>
  <Tab title="Python">
    ```bash
    pip install --upgrade anthropic
    ```

    如果您的应用程序尚未使用 OpenTelemetry SDK，请将其与控制台导出器一起安装，您将在下一步中使用该导出器打印 span：

    ```bash
    pip install opentelemetry-sdk
    ```
  </Tab>

  <Tab title="TypeScript">
    ```bash
    npm install @anthropic-ai/sdk@latest
    ```

    如果您的应用程序尚未使用 OpenTelemetry SDK，请将其与控制台导出器一起安装，您将在下一步中使用该导出器打印 span：

    ```bash
    npm install @opentelemetry/sdk-trace-node
    ```
  </Tab>

  <Tab title="C#">
    ```bash
    dotnet add package Anthropic
    ```

    如果您的应用程序尚未使用 OpenTelemetry SDK，请将其与控制台导出器一起安装，您将在下一步中使用该导出器打印 span：

    ```bash
    dotnet add package OpenTelemetry.Exporter.Console
    ```
  </Tab>

  <Tab title="Go">
    ```bash
    go get github.com/anthropics/anthropic-sdk-go/v2@latest
    ```

    如果您的应用程序尚未使用 OpenTelemetry SDK，请将其与控制台导出器一起安装，您将在下一步中使用该导出器打印 span：

    ```bash
    go get go.opentelemetry.io/otel/sdk go.opentelemetry.io/otel/exporters/stdout/stdouttrace
    ```
  </Tab>

  <Tab title="Java">
    将 Anthropic SDK 添加到您的依赖项中；如果您的构建文件已声明它，请将其至少更新到以下版本：

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

    如果您的应用程序尚未使用 OpenTelemetry SDK，请将其与控制台导出器一起安装，您将在下一步中使用该导出器打印 span。将 `io.opentelemetry:opentelemetry-sdk` 和 `io.opentelemetry:opentelemetry-exporter-logging` 添加到您的构建中，两者均使用 [Maven Central](https://central.sonatype.com/artifact/io.opentelemetry/opentelemetry-sdk) 上的最新版本。
  </Tab>

  <Tab title="PHP">
    ```bash
    composer require "anthropic-ai/sdk" "guzzlehttp/guzzle:^7"
    ```

    如果您的应用程序尚未使用 OpenTelemetry SDK，请将其与控制台导出器一起安装，您将在下一步中使用该导出器打印 span：

    ```bash
    composer require open-telemetry/sdk
    ```
  </Tab>

  <Tab title="Ruby">
    如果您的 `Gemfile` 尚未列出 `anthropic`，请添加该 gem：

    ```bash
    bundle add anthropic
    ```

    如果您的 `Gemfile` 已列出 `anthropic`，请更新该 gem：

    ```bash
    bundle update anthropic
    ```

    如果您的应用程序尚未使用 OpenTelemetry SDK，请将其与控制台导出器一起安装，您将在下一步中使用该导出器打印 span：

    ```bash
    bundle add opentelemetry-sdk
    ```
  </Tab>
</Tabs>

## 2. 注册 tracer provider

“tracer provider”（追踪器提供程序）是创建 span 并将其传递给导出器的 OpenTelemetry 对象。SDK 会使用您的应用程序注册的 tracer provider，因此您的 SDK 调用保持不变。请在您想要追踪的第一次调用之前注册它。

以下示例注册了一个将每个 span 打印到控制台的 tracer provider，然后进行一次调用。在连接可观测性平台之前，您可以用它来查看 span：

<CodeGroup exclude="shell">
  ```python Python
  from anthropic import Anthropic
  from opentelemetry import trace
  from opentelemetry.sdk.trace import TracerProvider
  from opentelemetry.sdk.trace.export import ConsoleSpanExporter, SimpleSpanProcessor

  # 注册一个 tracer provider，将每个 span 打印到控制台。
  provider = TracerProvider()
  provider.add_span_processor(SimpleSpanProcessor(ConsoleSpanExporter()))
  trace.set_tracer_provider(provider)

  client = Anthropic()

  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )
  print(message.content)
  ```

  ```typescript TypeScript
  import Anthropic from "@anthropic-ai/sdk";
  import {
    ConsoleSpanExporter,
    NodeTracerProvider,
    SimpleSpanProcessor
  } from "@opentelemetry/sdk-trace-node";

  // 注册一个 tracer provider，将每个 span 打印到控制台。
  const provider = new NodeTracerProvider({
    spanProcessors: [new SimpleSpanProcessor(new ConsoleSpanExporter())]
  });
  // register() 还会安装上下文管理器，使 SDK 的 span 能嵌套在您的 span 之下。
  provider.register();

  const client = new Anthropic();

  const message = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  });
  console.log(message.content);
  ```

  ```csharp C#
  using Anthropic;
  using Anthropic.Models.Messages;
  using OpenTelemetry;
  using OpenTelemetry.Trace;

  // 注册一个 tracer provider，将每个 span 打印到控制台。
  using var tracerProvider = Sdk.CreateTracerProviderBuilder()
      .AddSource("com.anthropic.sdk.csharp*")
      .AddConsoleExporter()
      .Build();

  AnthropicClient client = new();

  var message = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }],
  });
  Console.WriteLine(message);
  ```

  ```go Go
  package main

  import (
  	"context"
  	"fmt"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  	"go.opentelemetry.io/otel"
  	"go.opentelemetry.io/otel/exporters/stdout/stdouttrace"
  	"go.opentelemetry.io/otel/propagation"
  	sdktrace "go.opentelemetry.io/otel/sdk/trace"
  )

  func main() {
  	ctx := context.Background()

  	// 注册一个 tracer provider，将每个 span 打印到控制台。
  	exporter, err := stdouttrace.New(stdouttrace.WithPrettyPrint())
  	if err != nil {
  		panic(err)
  	}
  	provider := sdktrace.NewTracerProvider(sdktrace.WithSyncer(exporter))
  	defer provider.Shutdown(ctx)
  	otel.SetTracerProvider(provider)
  	// OpenTelemetry for Go 默认不注册 propagator，因此
  	// 需注册 W3C trace context 和 baggage propagator。
  	otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(propagation.TraceContext{}, propagation.Baggage{}))

  	client := anthropic.NewClient()

  	message, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 1024,
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  		},
  	})
  	if err != nil {
  		panic(err)
  	}
  	fmt.Println(message.Content)
  }
  ```

  ```java Java
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;
  import io.opentelemetry.api.baggage.propagation.W3CBaggagePropagator;
  import io.opentelemetry.api.trace.propagation.W3CTraceContextPropagator;
  import io.opentelemetry.context.propagation.ContextPropagators;
  import io.opentelemetry.context.propagation.TextMapPropagator;
  import io.opentelemetry.exporter.logging.LoggingSpanExporter;
  import io.opentelemetry.sdk.OpenTelemetrySdk;
  import io.opentelemetry.sdk.trace.SdkTracerProvider;
  import io.opentelemetry.sdk.trace.export.SimpleSpanProcessor;

  void main() {
      // 注册一个 tracer provider，将每个 span 写入日志。
      var tracerProvider = SdkTracerProvider.builder()
          .addSpanProcessor(SimpleSpanProcessor.create(LoggingSpanExporter.create()))
          .build();
      OpenTelemetrySdk.builder()
          .setTracerProvider(tracerProvider)
          // 在代码中构建 OpenTelemetry for Java 时，它不会注册 propagator，
          // 因此需注册 W3C trace context 和 baggage propagator。
          .setPropagators(ContextPropagators.create(TextMapPropagator.composite(
              W3CTraceContextPropagator.getInstance(), W3CBaggagePropagator.getInstance())))
          .buildAndRegisterGlobal();

      var client = AnthropicOkHttpClient.fromEnv();

      var message = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .addUserMessage("Hello, Claude")
          .build());
      IO.println(message.content());

      tracerProvider.close();
  }
  ```

  ```php PHP
  use Anthropic\Client;
  use Anthropic\Messages\Model;
  use OpenTelemetry\API\Baggage\Propagation\BaggagePropagator;
  use OpenTelemetry\API\Trace\Propagation\TraceContextPropagator;
  use OpenTelemetry\Context\Propagation\MultiTextMapPropagator;
  use OpenTelemetry\SDK\Sdk;
  use OpenTelemetry\SDK\Trace\SpanExporter\ConsoleSpanExporterFactory;
  use OpenTelemetry\SDK\Trace\SpanProcessor\SimpleSpanProcessor;
  use OpenTelemetry\SDK\Trace\TracerProvider;

  // 注册一个将每个 span 打印到控制台的 tracer provider。
  $tracerProvider = new TracerProvider(
      new SimpleSpanProcessor((new ConsoleSpanExporterFactory())->create()),
  );
  Sdk::builder()
      ->setTracerProvider($tracerProvider)
      // 在代码中构建 OpenTelemetry for PHP 时，它不会注册 propagator，
      // 因此需手动注册 W3C trace context 和 baggage propagator。
      ->setPropagator(new MultiTextMapPropagator([
          TraceContextPropagator::getInstance(),
          BaggagePropagator::getInstance(),
      ]))
      ->buildAndRegisterGlobal();

  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_OPUS_5_5,
  );
  echo $message;
  ```

  ```ruby Ruby
  require "anthropic"
  require "opentelemetry/sdk"

  # 注册一个 tracer provider，将每个 span 打印到控制台。
  OpenTelemetry::SDK.configure do |config|
    config.add_span_processor(
      OpenTelemetry::SDK::Trace::Export::SimpleSpanProcessor.new(
        OpenTelemetry::SDK::Trace::Export::ConsoleSpanExporter.new
      )
    )
  end

  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{role: "user", content: "Hello, Claude"}]
  )
  puts(message.content)
  ```
</CodeGroup>

控制台会显示一个 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) span。

## 3. 将 span 发送到您的可观测性平台

将控制台导出器替换为指向您的可观测性平台的 OTLP 导出器。有关您所用语言的设置，请参阅 OpenTelemetry 文档中的 [Python 导出器](https://opentelemetry.io/docs/languages/python/exporters/)、[TypeScript 导出器](https://opentelemetry.io/docs/languages/js/exporters/)、[.NET 导出器](https://opentelemetry.io/docs/languages/dotnet/exporters/)、[Go 导出器](https://opentelemetry.io/docs/languages/go/exporters/)、[Java SDK 配置](https://opentelemetry.io/docs/languages/java/configuration/)、[PHP 导出器](https://opentelemetry.io/docs/languages/php/exporters/)或 [Ruby 导出器](https://opentelemetry.io/docs/languages/ruby/exporters/)。

## 4. 配置 SDK

使用以下选项来决定 span 中包含哪些内容以及将它们发送到何处。您可以在代码中设置选项，或者（如果该选项有对应的环境变量）使用其名称下方列出的环境变量进行设置。

<Note>
  如果您向客户端传递 `open_telemetry` (typescript, java, php: `openTelemetry`; csharp: `OpenTelemetry`; go: `option.With…()`)，它会整体覆盖该选项的环境变量，包括嵌套键。您省略的任何嵌套键都会使用表中的默认值，即使设置了其环境变量也是如此。
</Note>

| 选项                                                                                                                                                                                                                             | 用途                                                                                                                   | 默认值                                                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `traces.content_mode` (typescript, java, php: `traces.contentMode`; csharp: `Traces.ContentMode`; go: `option.WithOpenTelemetryTracesContentMode()`)<br />`ANTHROPIC_OPEN_TELEMETRY_TRACES_CONTENT_MODE=content`               | 在 span 中包含提示和响应。请参阅[内容模式](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#choose-a-content-mode)。 | `metadata_only`                                                                                                                                |
| `traces.max_content_bytes` (typescript, java, php: `traces.maxContentBytes`; csharp: `Traces.MaxContentBytes`; go: `option.WithOpenTelemetryTracesMaxContentBytes()`)<br />`ANTHROPIC_OPEN_TELEMETRY_TRACES_MAX_CONTENT_BYTES` | 更改每个 span 的内容限制。请参阅[内容限制](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#content-limits)。        | 0.75 MB                                                                                                                                        |
| `conversation_id` (typescript, java, php: `conversationId`; csharp: `ConversationId`; go: `option.WithOpenTelemetryConversationID()`)                                                                                          | 将多次调用关联到一个对话中。SDK 会将该 ID 写入 `gen_ai.conversation.id`。该 ID 在两种内容模式下都会被导出，因此请勿在其中放入个人数据或机密信息。                          | 无                                                                                                                                              |
| `tracer_provider` (typescript, java, php: `tracerProvider`; csharp: `AddSource("com.anthropic.sdk.csharp*")`; go: `option.WithTracerProvider()`)                                                                               | 选择接收 SDK span 的 tracer provider。                                                                                     | 您在[注册 tracer provider](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#register-a-tracer-provider) 中设置的 tracer provider |
| `propagation` (go: `option.WithOpenTelemetryPropagation()`)<br />`ANTHROPIC_OPEN_TELEMETRY_PROPAGATION=false`                                                                                                                  | 停止随请求发送 trace 上下文和 baggage 请求头，例如当您的 HTTP 客户端插桩已经发送它们时。SDK 仍会发出 span。适用于 Python、TypeScript、Go、Java、Ruby 和 PHP SDK。   | 开启                                                                                                                                             |
| `open_telemetry` (typescript, java, php: `openTelemetry`; csharp: `OpenTelemetry`; go: `option.WithoutOpenTelemetry()`)<br />`ANTHROPIC_OPEN_TELEMETRY=false`                                                                  | 为某个客户端关闭 span，例如当另一个库已经在追踪该 SDK 时。                                                                                   | 开启                                                                                                                                             |

以下示例展示了如何使用每个选项：

<CodeGroup exclude="shell">
  ```python Python
  # 使用指定的 tracer provider 并记录内容。
  # 设置 open_telemetry 会覆盖所有 ANTHROPIC_OPEN_TELEMETRY* 环境变量。
  client = Anthropic(
      open_telemetry={"tracer_provider": provider, "traces": {"content_mode": "content"}}
  )

  # 为单个客户端关闭 span。
  quiet_client = Anthropic(open_telemetry=False)

  # 使用 with_options() 设置会话 ID，该方法返回客户端的副本。
  # with_options() 会替换整个选项，因此请以 client.open_telemetry 为基础进行修改。
  message = client.with_options(
      open_telemetry={**client.open_telemetry, "conversation_id": "conv_42"}
  ).messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )
  ```

  ```typescript TypeScript
  // 使用指定的 tracer provider 并记录内容。
  // 设置 openTelemetry 会覆盖所有 ANTHROPIC_OPEN_TELEMETRY* 环境变量。
  const client = new Anthropic({
    openTelemetry: { tracerProvider: provider, traces: { contentMode: "content" } }
  });

  // 为单个客户端关闭 span。
  const quietClient = new Anthropic({ openTelemetry: false });

  // 在单次调用的请求选项中设置会话 ID。
  const message = await client.messages.create(
    {
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello, Claude" }]
    },
    { openTelemetry: { conversationId: "conv_42" } }
  );
  ```

  ```csharp C#
  // 使用指定的 tracer provider：将 SDK 的源添加到其中。
  using var tracerProvider = Sdk.CreateTracerProviderBuilder()
      .AddSource("com.anthropic.sdk.csharp*")
      .AddConsoleExporter()
      .Build();

  // 记录内容。
  // 设置 OpenTelemetry 会覆盖所有 ANTHROPIC_OPEN_TELEMETRY* 环境变量。
  AnthropicClient client = new()
  {
      OpenTelemetry = new() { Traces = new() { ContentMode = OpenTelemetryContentMode.Content } },
  };

  // 为单个客户端关闭 span。
  AnthropicClient quietClient = new() { OpenTelemetry = false };

  // 为同一对话中的调用设置对话 ID。
  var conversation = client.WithOptions(options => options with
  {
      OpenTelemetryRequest = new() { ConversationId = "conv_42" },
  });
  var message = await conversation.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }],
  });
  ```

  ```go Go
  // 使用指定的 tracer provider 并记录内容。
  client := anthropic.NewClient(
  	option.WithTracerProvider(provider),
  	option.WithOpenTelemetryTracesContentMode(option.OpenTelemetryContentModeContent),
  )

  // 为单个客户端关闭 span。
  quietClient := anthropic.NewClient(option.WithoutOpenTelemetry())

  // 在单次调用的请求选项中设置会话 ID。
  message, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  }, option.WithOpenTelemetryConversationID("conv_42"))
  ```

  ```java Java
  // 仅当您使用 fromEnv() 构建客户端时，客户端才会读取环境变量。
  // 系统属性 anthropic.openTelemetry、anthropic.openTelemetryTracesContentMode
  // 和 anthropic.openTelemetryTracesMaxContentBytes 的优先级高于环境
  // 变量。

  // 使用特定的 tracer provider 并记录内容。从
  // OpenTelemetryOptions.fromEnv() 开始，以保留 fromEnv() 读取的选项。
  var envOptions = OpenTelemetryOptions.fromEnv();
  var client = AnthropicOkHttpClient.builder()
      .fromEnv()
      .openTelemetry(envOptions.toBuilder()
          .tracerProvider(tracerProvider)
          .traces(envOptions.traces().toBuilder()
              .contentMode(OpenTelemetryContentMode.CONTENT)
              .build())
          .build())
      .build();

  // 为单个客户端关闭 span。
  var quietClient = AnthropicOkHttpClient.builder()
      .fromEnv()
      .openTelemetry(false)
      .build();

  // 在单次调用的请求选项中设置会话 ID。
  var params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024)
      .addUserMessage("Hello, Claude")
      .build();
  var message = client.messages().create(params, RequestOptions.builder()
      .openTelemetry(OpenTelemetryRequestOptions.builder()
          .conversationId("conv_42")
          .build())
      .build());
  ```

  ```php PHP
  // 使用特定的 tracer provider 并记录内容。
  // 设置 openTelemetry 会覆盖所有 ANTHROPIC_OPEN_TELEMETRY* 环境变量。
  $client = new Client(
      openTelemetry: ['tracerProvider' => $tracerProvider, 'traces' => ['contentMode' => 'content']],
  );

  // 为单个客户端关闭 span。
  $quietClient = new Client(openTelemetry: false);

  // 在单次调用的请求选项中设置会话 ID。
  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_OPUS_5_5,
      requestOptions: ['openTelemetry' => ['conversationId' => 'conv_42']],
  );
  ```

  ```ruby Ruby
  # 使用指定的 tracer provider 并记录内容。
  # 设置 open_telemetry 会覆盖所有 ANTHROPIC_OPEN_TELEMETRY* 环境变量。
  client = Anthropic::Client.new(
    open_telemetry: {
      tracer_provider: OpenTelemetry.tracer_provider,
      traces: {content_mode: :content}
    }
  )

  # 为单个客户端关闭 span。
  quiet_client = Anthropic::Client.new(open_telemetry: false)

  # 在单次调用的请求选项中设置会话 ID。
  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{role: "user", content: "Hello, Claude"}],
    request_options: {open_telemetry: {conversation_id: "conv_42"}}
  )
  ```
</CodeGroup>

## 后续步骤

<CardGroup cols={2}>
  <Card title="设置 API 追踪" icon="cloud" href="https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api">
    获得对 Claude API 的可观测性。
  </Card>

  <Card title="查看示例追踪" icon="sitemap" href="https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#example-traces-and-how-to-read-them">
    跟踪一个请求从您的客户端进入 Claude API 的全过程。
  </Card>
</CardGroup>

## 流追踪

流的 `anthropic.messages.create` span 会在您的代码读取完整个流或关闭流时结束，因此如果您提前停止读取某个流，请将其关闭。在内容模式下，只有当您使用 `messages.stream()` (go: `Messages.NewStreaming()`; java: `messages().createStreaming()`; csharp: `Messages.CreateStreaming()`; php: `messages->createStream()`) 读取流时，span 才会包含模型响应。

您在读取流时启动的 span 是 SDK span 的同级 span，而不是其子 span。要将它们与请求归为一组，请启动您自己的 span，将调用和读取循环都包含在内。

## Managed Agents 自托管沙箱追踪

在 Managed Agents [自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)中，由您的沙箱运行工具，因此工具 span 来自 SDK 而不是 API。Python、TypeScript 和 Go SDK 中的环境工作进程会为其运行的每个工具和发送的每个请求发出一个 span。它会将这些 span 归入每个工作项对应的一个 [`anthropic.environment.work`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-environment-work) span 之下。

要获取这些 span：

1. 在运行工作进程的进程中，按照步骤 1 到 4 [设置 SDK 追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#install-the-sdk)。
2. 允许从沙箱到您的可观测性平台或 OpenTelemetry Collector 的网络流量。

当工作进程发送工具结果时，该请求的 API span 会嵌套在其 SDK span 之下。

## trace 上下文和 baggage

SDK 会在每个请求中使用您应用程序的传播器，通过 `traceparent` 等请求头将 trace 上下文发送给 API。如果该传播器还写入 baggage，SDK 也会发送您的 baggage，您可以借此[识别您的最终用户](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#identify-your-end-users)。若想发送不含 baggage 的 trace 上下文，请只注册 W3C trace 上下文传播器。

若不想让 SDK 发送这些请求头（例如您的 HTTP 客户端插桩已经发送了它们），请关闭 SDK 的传播：

<CodeGroup exclude="shell">
  ```python Python
  client = Anthropic(open_telemetry={"propagation": False})
  ```

  <CodeGroupItem>
    ```typescript TypeScript
    const client = new Anthropic({ openTelemetry: { propagation: false } });
    ```

    <Warning>
      如果您使用 

      `@opentelemetry/instrumentation-undici`

      （

      `@opentelemetry/auto-instrumentations-node`

       包含该库），请设置 

      `propagation: false`

      。否则，一个 

      [undici 插桩错误](https://github.com/open-telemetry/opentelemetry-js-contrib/issues/3711)

      会为每个请求添加第二个 

      `traceparent`

       请求头，从而使 trace 上下文无效。
    </Warning>
  </CodeGroupItem>

  ```csharp C#
  using System.Diagnostics;

  // SDK 使用 .NET 的 DistributedContextPropagator 写入这些请求头。
  // 无输出传播器会让进程中的每个 HttpClient 都不再发送这些请求头。
  // 请在创建任何客户端之前进行设置。
  DistributedContextPropagator.Current = DistributedContextPropagator.CreateNoOutputPropagator();

  AnthropicClient client = new();
  ```

  ```go Go
  client := anthropic.NewClient(option.WithOpenTelemetryPropagation(false))
  ```

  ```java Java
  var envOptions = OpenTelemetryOptions.fromEnv();
  var client = AnthropicOkHttpClient.builder()
      .fromEnv()
      .openTelemetry(envOptions.toBuilder()
          .propagation(false)
          .build())
      .build();
  ```

  ```php PHP
  $client = new Client(openTelemetry: ['propagation' => false]);
  ```

  ```ruby Ruby
  client = Anthropic::Client.new(open_telemetry: {propagation: false})
  ```
</CodeGroup>

SDK 调用 [MCP 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#client-side-mcp-helpers)时，SDK 的工具 span 是当前 span。部分 MCP 客户端库会通过 `_meta` 将 trace 上下文发送到 MCP 服务器，因此读取它的服务器可以将自己的 span 添加到您的 trace 中，位于该工具 span 之下。2.0 及更高版本的 Python `mcp` 包和 C# `ModelContextProtocol` 包会这样做。TypeScript、Go、Java、Ruby 和 PHP 的 MCP SDK 不会自行这样做。

### 识别您的最终用户

[Baggage](https://opentelemetry.io/docs/concepts/signals/baggage/) 是请求通过其 W3C `baggage` 请求头携带的一组键值对。要在某个请求的 API span 上设置 [`user.id`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#user.id) 属性，请在发送该请求时设置一个 `user.id` baggage 条目，其值为您最终用户的 ID。请使用不透明 ID，而不是电子邮件地址：为保护用户隐私，API 会丢弃任何包含电子邮件地址的 `user.id`。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -H "baggage: user.id=4f2a" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [
        {"role": "user", "content": "Hello, Claude"}
      ]
    }'
  ```

  ```bash CLI
  ANTHROPIC_CUSTOM_HEADERS="baggage: user.id=4f2a" ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  from opentelemetry import baggage, context

  client = Anthropic()

  token = context.attach(baggage.set_baggage("user.id", "4f2a"))
  try:
      message = client.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          messages=[{"role": "user", "content": "Hello, Claude"}],
      )
  finally:
      context.detach(token)
  ```

  ```typescript TypeScript
  import { context, propagation } from "@opentelemetry/api";

  const client = new Anthropic();

  const userBaggage = propagation.createBaggage({ "user.id": { value: "4f2a" } });
  const message = await context.with(propagation.setBaggage(context.active(), userBaggage), () =>
    client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello, Claude" }]
    })
  );
  ```

  ```csharp C#
  using System.Diagnostics;
  // ...
  // 请在您的 tracer provider 上通过 AddSource 注册此源，以便其记录该 activity。
  ActivitySource source = new("MyApp");
  AnthropicClient client = new();

  // SDK 会发送当前 activity 的 baggage。
  using var activity = source.StartActivity("chat");
  activity?.SetBaggage("user.id", "4f2a");
  var message = await client.Messages.Create(new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }],
  });
  ```

  ```go Go
  client := anthropic.NewClient()

  member, _ := baggage.NewMember("user.id", "4f2a")
  userBaggage, _ := baggage.New(member)
  ctx := baggage.ContextWithBaggage(context.Background(), userBaggage)
  message, err := client.Messages.New(ctx, anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  })
  ```

  ```java Java
  import io.opentelemetry.api.baggage.Baggage;

  void main() {
      var client = AnthropicOkHttpClient.fromEnv();

      try (var scope = Baggage.current().toBuilder().put("user.id", "4f2a").build().makeCurrent()) {
          var message = client.messages().create(MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024)
              .addUserMessage("Hello, Claude")
              .build());
      }
  }
  ```

  ```php PHP
  use OpenTelemetry\API\Baggage\Baggage;

  $client = new Client();

  $scope = Baggage::getBuilder()->set('user.id', '4f2a')->build()->activate();
  try {
      $message = $client->messages->create(
          maxTokens: 1024,
          messages: [['role' => 'user', 'content' => 'Hello, Claude']],
          model: Model::CLAUDE_OPUS_5_5,
      );
  } finally {
      $scope->detach();
  }
  ```

  ```ruby Ruby
  require "opentelemetry/sdk"

  client = Anthropic::Client.new

  OpenTelemetry::Context.with_current(OpenTelemetry::Baggage.set_value("user.id", "4f2a")) do
    client.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 1024,
      messages: [{role: "user", content: "Hello, Claude"}]
    )
  end
  ```
</CodeGroup>

## 故障排除

| 问题                                                           | 检查内容                                                                                                                                                                                                        |
| ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 没有来自 SDK 的 span                                              | 在第一次调用之前注册 tracer provider。确保您已按[安装 SDK](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#install-the-sdk) 中的说明安装了 SDK，并且 `ANTHROPIC_OPEN_TELEMETRY` 不是 `false`。                      |
| Java SDK 不向已注册的 tracer provider 发送 span，或不向 API 发送 trace 上下文 | 另一个依赖项将 `opentelemetry-api` 设置为低于 1.57.0 的版本。请将 `opentelemetry-api` 升级到 1.57.0 或更高版本。                                                                                                                       |
| SDK span 没有嵌套在您的 span 之下，或者您的工具创建的 span 没有嵌套在工具 span 之下      | 在您自己的某个 span 内部调用 SDK，如[将 API span 连接到您自己的追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#connect-api-spans-to-your-own-traces)中的示例所示。在 Go 中，请从工具运行器传递给工具的 `ctx` 启动您的工具创建的 span。 |
| 来自 SDK 的请求没有 `traceparent` 请求头                               | 向 OpenTelemetry 注册 W3C trace 上下文传播器。SDK 使用您应用程序的传播器写入请求头。                                                                                                                                                   |
| 流的 SDK span 没有结束或不包含响应                                       | 请参阅[流追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk#trace-a-stream)。                                                                                                            |
| 每个请求出现两次                                                     | 请求的 SDK span 和 API span 具有相同的名称。使用 span 类型或 `service.name` 来区分它们。如果另一个库也在追踪该 SDK，请关闭两者之一。                                                                                                                   |

## 可用性

SDK 追踪在所有模型上处于 beta 阶段。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态   | 云平台                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Amazon Bedrock](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock), [Amazon Bedrock (bedrock-runtime)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy), [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws), [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed), [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用
* 用法：SDK 追踪您的代码发送的请求，例如您发送到会话的事件；智能体的模型请求和工具调用的 span 来自 API 追踪。在自托管沙箱中，SDK 的环境 worker 还会追踪其运行的每个工具。

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |
