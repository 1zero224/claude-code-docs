---
title: 在 Claude API 中设置 OpenTelemetry
url: https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api
description: 在 Claude Console 中添加目标，将您的 Messages API 请求和 Managed Agents 会话的 OpenTelemetry 追踪数据发送到您的可观测性平台。
featureMetadata:
  topic:
    title: OpenTelemetry
    url: https://platform.claude.com/docs/en/manage-claude/opentelemetry
  status: beta
  zdr:
    eligibility: eligible
    note: Excludes [Covered Models](https://platform.claude.com/docs/en/manage-claude/api-and-data-retention#model-specific-data-retention-requirements). Beta 版。span 的 ZDR 资格与生成它们的功能相同。HIPAA 组织可以导出元数据。SDK span 直接从您的环境发送到您的导出器。送达后，数据由您选择的服务（位于 Anthropic 之外）持有，并受该服务的保留和访问控制约束。
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
    Claude Platform on AWS: beta
---

借助 OpenTelemetry，Claude API 可以为其运行的每个模型请求和每次工具使用发出一个 “span”（跨度）。您无需更改代码，即可在您的可观测性平台中查看这些 span。

API 会为您的 Messages API 请求和 Managed Agents 会话发出 span。要接收它们，请在 Claude Console 中添加一个"destination"（目标）。目标会告诉 API 您的可观测性平台的 OTLP 端点以及如何向其进行身份验证。

## 您需要准备

* **管理员角色**：在 Claude Console 组织中拥有管理员角色，才能添加和更改目标。拥有开发者角色时，您可以查看目标。如果您的组织使用 Claude Platform on AWS，请按照[登录](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#signing-in)中的说明，通过 AWS Console 登录 Console。
* **OTLP/HTTP 端点**：在 443 端口的 HTTPS URL 上接受 protobuf 编码。如果您运行自己的 [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)，其 OTLP/HTTP 接收器默认监听 4318 端口。请将接收器配置为在 443 端口接受 HTTPS，或在 Collector 前面放置负载均衡器或反向代理。
* **端点凭据**，例如 API 密钥。

## 1. 添加目标

<Info>
  当您添加遥测目标时，Anthropic 将根据您的提供商条款和访问控制，把您组织中 Claude Platform 活动的追踪发送到您指定的第三方提供商的 OTLP/HTTP 端点。交付后，数据由您选择的服务在 Anthropic 之外持有，并受该服务的保留和访问控制约束。
</Info>

1. 在 Console 中，打开您组织的 [OpenTelemetry 设置页面](https://platform.claude.com/settings/telemetry-destinations)，然后点击**添加目标位置**。

2. 配置目标：

   | 字段                | 填写内容                                                                                                                                                                                                                        |
   | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | **提供商**           | 您的可观测性平台；如果未列出，则选择**自定义**。如果您选择已列出的平台，Console 会要求您提供其详细信息（例如 API 密钥），而不是 **OTLP 端点 URL**、**请求标头**和 **OTLP 追踪端点 URL**。                                                                                                       |
   | **名称**            | 可选。例如，"Acme 的可观测性平台"。                                                                                                                                                                                                       |
   | **OTLP 端点 URL**   | 您的可观测性平台 OTLP/HTTP 端点的基础 HTTPS URL，需使用主机名而非 IP 地址。它可以带有路径前缀，例如 `https://gateway.example.com/otlp`。不要添加 `/v1/traces`，因为 API 会自动添加它。如果您设置了 **OTLP 追踪端点 URL**，API 会改用该 URL。                                                    |
   | **请求标头**          | API 随每个请求发送的标头，例如您的可观测性平台的 API 密钥标头。请仅将凭据放在标头值中。保存后，Console 不会再次显示这些值。                                                                                                                                                      |
   | **内容模式**          | **仅元数据**或**完整内容**。请参阅[内容模式的区别](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#choose-a-content-mode)。                                                                                                   |
   | **工作区**           | 目标接收其 span 的工作区：**全部**、**所选**或**除所选外的全部**。请参阅[为目标选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#choose-workspaces-for-a-destination)。                                                        |
   | **OTLP 追踪端点 URL** | 可选，位于**高级设置**下。您的可观测性平台追踪端点的完整 URL，例如 `https://gateway.example.com/custom/traces`。当该平台在 OTLP 端点 URL 加 `/v1/traces` 以外的 URL 接收追踪时，请设置此项。它必须与 OTLP 端点 URL 使用相同的主机。                                                            |
   | **最大内容大小（MB）**    | 可选，当内容模式为**完整内容**时位于**高级设置**下。API 在每个 span 中发送的最大内容量。Console 会根据您选择的**提供商**自动填写该值。请根据您的可观测性平台可接受的大小进行调整。将其留空即可发送 API 允许的最大内容量。请参阅[内容限制](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#content-limits)。 |

3. 点击**添加目标位置**保存。

4. 在目标的**更多操作**菜单中点击**测试**。随后会显示一条消息，说明测试 span 已投递到您的收集器，或投递失败的原因（请参阅 [投递错误](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#delivery-errors)）。

5. 检查 `anthropic.test` span 是否已到达您的可观测性平台。

### Admin API 和 CLI

您也可以通过 [Admin API](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api) 添加目标，无需使用 Console。请以管理员身份登录 CLI，或使用其他 [Admin API 凭证](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#authentication)，例如在 `ANTHROPIC_API_KEY` 中使用 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys)或服务账户密钥。在 Claude Platform on AWS 上，请改为将请求发送到 `https://aws-external-anthropic.{region}.api.aws`，并按照[发出请求](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws#making-requests)中的说明使用您的 AWS 凭据签名。在该平台上，您的 [IAM 策略](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions)决定谁可以管理目标。请求只能包含您的 AWS 账户在该区域中的工作区。

<CodeGroup>
  ```bash cURL
  curl --fail-with-body -sS "https://api.anthropic.com/v1/organizations/telemetry_destinations" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: telemetry-destinations-2026-08-11" \
    -H "content-type: application/json" \
    -d '{
      "endpoint_url": "https://gateway.example.com/otlp",
      "headers": {"authorization": "Bearer YOUR_OBSERVABILITY_API_KEY"},
      "traces": {"content_mode": "metadata_only"},
      "workspaces": {"type": "all"}
    }'
  ```

  ```bash CLI
  # 以管理员身份登录一次，此操作会打开浏览器。
  ant auth login --profile admin --scope "org:admin"
  ant --profile admin beta:organization:telemetry-destinations create \
    --endpoint-url https://gateway.example.com/otlp \
    --headers '{authorization: "Bearer YOUR_OBSERVABILITY_API_KEY"}' \
    --traces.content-mode metadata_only \
    --workspaces.type all
  ```

  ```python Python
  client = anthropic.Anthropic()

  destination = client.beta.organization.telemetry_destinations.create(
      endpoint_url="https://gateway.example.com/otlp",
      headers={"authorization": "Bearer YOUR_OBSERVABILITY_API_KEY"},
      traces={"content_mode": "metadata_only"},
      workspaces={"type": "all"},
  )

  print(destination.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const destination = await client.beta.organization.telemetryDestinations.create({
    endpoint_url: "https://gateway.example.com/otlp",
    headers: { authorization: "Bearer YOUR_OBSERVABILITY_API_KEY" },
    traces: { content_mode: "metadata_only" },
    workspaces: { type: "all" }
  });

  console.log(destination.id);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.TelemetryDestinations;

  AnthropicClient client = new();

  var destination = await client.Beta.Organization.TelemetryDestinations.Create(
      new()
      {
          EndpointUrl = "https://gateway.example.com/otlp",
          Headers = new Dictionary<string, string>
          {
              ["authorization"] = "Bearer YOUR_OBSERVABILITY_API_KEY",
          },
          Traces = new() { ContentMode = BetaTelemetryDestinationContentMode.MetadataOnly },
          Workspaces = new BetaTelemetryDestinationWorkspacesAllParams(),
      }
  );

  Console.WriteLine(destination.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  destination, err := client.Beta.Organization.TelemetryDestinations.New(
  	context.Background(),
  	anthropic.BetaOrganizationTelemetryDestinationNewParams{
  		EndpointURL: "https://gateway.example.com/otlp",
  		Headers: map[string]string{
  			"authorization": "Bearer YOUR_OBSERVABILITY_API_KEY",
  		},
  		Traces: anthropic.BetaTelemetryDestinationTracesParams{
  			ContentMode: anthropic.BetaTelemetryDestinationContentModeMetadataOnly,
  		},
  		Workspaces: anthropic.BetaTelemetryDestinationWorkspacesParamsUnion{
  			OfAll: &anthropic.BetaTelemetryDestinationWorkspacesAllParams{},
  		},
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(destination.ID)
  ```

  ```java Java
  import com.anthropic.core.JsonValue;
  import com.anthropic.models.beta.organization.telemetrydestinations.BetaTelemetryDestinationContentMode;
  import com.anthropic.models.beta.organization.telemetrydestinations.BetaTelemetryDestinationTracesParams;
  import com.anthropic.models.beta.organization.telemetrydestinations.BetaTelemetryDestinationWorkspacesAllParams;
  import com.anthropic.models.beta.organization.telemetrydestinations.TelemetryDestinationCreateParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = TelemetryDestinationCreateParams.builder()
          .endpointUrl("https://gateway.example.com/otlp")
          .headers(TelemetryDestinationCreateParams.DestinationHeaders.builder()
              .putAdditionalProperty("authorization", JsonValue.from("Bearer YOUR_OBSERVABILITY_API_KEY"))
              .build())
          .traces(BetaTelemetryDestinationTracesParams.builder()
              .contentMode(BetaTelemetryDestinationContentMode.METADATA_ONLY)
              .build())
          .workspaces(BetaTelemetryDestinationWorkspacesAllParams.builder().build())
          .build();
      var destination = client.beta().organization().telemetryDestinations().create(params);

      IO.println(destination.id());
  }
  ```

  ```php PHP
  $client = new Client();

  $destination = $client->beta->organization->telemetryDestinations->create(
      endpointURL: 'https://gateway.example.com/otlp',
      headers: ['authorization' => 'Bearer YOUR_OBSERVABILITY_API_KEY'],
      traces: ['contentMode' => 'metadata_only'],
      workspaces: ['type' => 'all'],
  );

  echo "{$destination->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  destination = client.beta.organization.telemetry_destinations.create(
    endpoint_url: "https://gateway.example.com/otlp",
    headers: {authorization: "Bearer YOUR_OBSERVABILITY_API_KEY"},
    traces: {content_mode: :metadata_only},
    workspaces: {type: :all}
  )

  puts destination.id
  ```
</CodeGroup>

创建请求接受 Console 中除**提供商**之外的设置，因此请自行输入您可观测性平台的端点 URL 和标头。设置以字节为单位的 `traces.max_content_bytes` (csharp, go: `Traces.MaxContentBytes`; java, php: `traces.maxContentBytes`)，以适应您可观测性平台可接受的大小（请参阅[内容限制](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#content-limits)）。要列出、更改或删除目标，或者添加或移除单个工作区，请使用[遥测目标 API](https://platform.claude.com/docs/zh-CN/api/beta/organization/telemetry_destinations) 的其他方法。

## 2. 为目标选择工作区

目标会接收其覆盖的工作区的 span。请在添加目标时选择这些工作区。对于**工作区**，请选择以下选项之一：

* **全部：** 组织中的每个工作区，包括[默认工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#how-workspaces-work)和之后创建的工作区。
* **所选：** 仅您选择的工作区。
* **除所选外的全部：** 与**全部**相同，但不包括您选择的工作区。

对于**所选**或**除所选外的全部**，请在选项下方出现的搜索框中选择工作区。要在之后更改目标的工作区，请在**工作区**筛选器显示**全部**时，在 Console 的 [OpenTelemetry 设置页面](https://platform.claude.com/settings/telemetry-destinations)上打开该目标的**更多操作**菜单。点击**编辑**，然后选择另一个**工作区**选项或在搜索框中更改工作区。

如果您在组织中拥有开发者角色，并在某个工作区中拥有工作区管理员角色，则可以将该工作区添加到**所选**目标，或将其从该目标中移除。请将**工作区**筛选器设置为该工作区，然后打开该目标的**更多操作**菜单。在 Claude Platform on AWS 上，Console 没有工作区管理员角色。在该平台上，将由您的 [IAM 策略](https://platform.claude.com/docs/zh-CN/api/claude-platform-on-aws-iam-actions)决定谁可以将工作区添加到目标或将其从该目标中移除。

API 会将 span 发送到覆盖[请求或会话运行所在工作区](https://platform.claude.com/docs/zh-CN/manage-claude/workspaces#api-keys-and-resource-scoping)的每个已启用目标。

## 3. 检查 span 是否到达

1. 在 Console 中，选择一个该目标所覆盖的工作区，并在 [Playground](https://platform.claude.com/playground) 中发送一个 Messages API 请求。您也可以从您的应用程序发送 Messages API 请求，或在该工作区中启动一个 Managed Agents 会话。要启动会话，您可以按照 [Claude Managed Agents 入门](https://platform.claude.com/docs/zh-CN/managed-agents/quickstart)中的步骤操作。
2. 在 Console 的 [OpenTelemetry 设置页面](https://platform.claude.com/settings/telemetry-destinations)上，检查该目标的**投递**列是否显示**正常**。这可能需要大约一分钟。
3. 在您的可观测性平台中，按 `service.name` = `claude-platform` 进行筛选，以找到该请求的 span。

如果该列显示**失败**，请将指针悬停在其上以查看最后一个错误。要查找可能的原因，请参阅[投递错误](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#delivery-errors)。

## 4. 将 API span 连接到您自己的追踪

默认情况下，Messages API 请求或 Managed Agents 会话的 API span 位于它们自己的追踪中。将它们连接到您应用程序的追踪，使其嵌套在您自己的 span 之下。

API 从 W3C [`traceparent` 标头](https://www.w3.org/TR/trace-context/)中读取每个请求的追踪上下文。如果标头中的采样标志处于关闭状态，API 不会发出该请求的 span 或其下的 span。

<Tabs>
  <Tab title="使用 Anthropic SDK">
    要连接 API span，请在您的应用程序中[设置 SDK 追踪](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)，并在您自己的某个 span 内部调用 SDK。这样，您的 span 会包含 SDK span，而 SDK span 又包含 API span。请将所有这些 span 发送到同一个可观测性平台，以便它能将它们显示为一个追踪。请参阅[Messages API 请求](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#a-messages-api-request)和 [Managed Agents 会话](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#a-managed-agents-session)的示例追踪。

    以下示例在您自己的 span 内部向会话发送一条用户消息：

    <CodeGroup exclude="shell">
      ```python Python
      from opentelemetry import trace

      tracer = trace.get_tracer("my-service")

      with tracer.start_as_current_span("summarize_issues"):
          client.beta.sessions.events.send(
              session.id,
              events=[
                  {
                      "type": "user.message",
                      "content": [{"type": "text", "text": "Summarize the open issues"}],
                  },
              ],
          )
      ```

      ```typescript TypeScript
      import { trace } from "@opentelemetry/api";

      const tracer = trace.getTracer("my-service");

      await tracer.startActiveSpan("summarize_issues", async (span) => {
        try {
          await client.beta.sessions.events.send(session.id, {
            events: [
              {
                type: "user.message",
                content: [{ type: "text", text: "Summarize the open issues" }]
              }
            ]
          });
        } finally {
          span.end();
        }
      });
      ```

      ```csharp C#
      using System.Diagnostics;
      // ...

      // 将此源添加到您的 tracer provider 中，与 com.anthropic.sdk.csharp* 并列。
      var source = new ActivitySource("MyService");

      using (source.StartActivity("summarize_issues"))
      {
          // parameters 包含一个 user.message 事件。
          await client.Beta.Sessions.Events.Send(session.ID, parameters);
      }
      ```

      ```go Go
      // 传入包含您的 span 的 context。
      ctx, span := otel.Tracer("my-service").Start(ctx, "summarize_issues")
      defer span.End()

      // params 包含一个 user.message 事件。
      _, err := client.Beta.Sessions.Events.Send(ctx, session.ID, params)
      ```

      ```java Java
      Tracer tracer = GlobalOpenTelemetry.getTracer("my-service");
      Span span = tracer.spanBuilder("summarize_issues").startSpan();

      try (Scope scope = span.makeCurrent()) {
          // params 包含一个 user.message 事件。
          client.beta().sessions().events().send(session.id(), params);
      } finally {
          span.end();
      }
      ```

      ```php PHP
      use OpenTelemetry\API\Globals;

      $tracer = Globals::tracerProvider()->getTracer('my-service');
      $span = $tracer->spanBuilder('summarize_issues')->startSpan();
      $scope = $span->activate();

      try {
          $client->beta->sessions->events->send(
              $session->id,
              events: [
                  [
                      'type' => 'user.message',
                      'content' => [
                          ['type' => 'text', 'text' => 'Summarize the open issues'],
                      ],
                  ],
              ],
          );
      } finally {
          $scope->detach();
          $span->end();
      }
      ```

      ```ruby Ruby
      tracer = OpenTelemetry.tracer_provider.tracer("my-service")

      tracer.in_span("summarize_issues") do
        client.beta.sessions.events.send_(
          session.id,
          events: [
            {
              type: "user.message",
              content: [{type: "text", text: "Summarize the open issues"}]
            }
          ]
        )
      end
      ```
    </CodeGroup>

    SDK 使用您应用程序的传播器写入 `traceparent` 标头，其中的采样标志遵循您的采样器。如果您的应用程序使用 W3C trace context 以外的传播器（例如 B3），请同时添加 W3C trace context 传播器。
  </Tab>

  <Tab title="使用其他客户端">
    将当前 span 的追踪上下文复制到请求标头中，例如在您创建 Managed Agents 会话或发送用户消息时：

    <CodeGroup exclude="shell:CLI">
      ```bash cURL
      # TRACEPARENT 是您自己服务中当前活动 span 的 traceparent，
      # 例如 00-<trace-id>-<span-id>-01。
      curl -fsSL "https://api.anthropic.com/v1/sessions/$SESSION_ID/events" \
        -H "x-api-key: $ANTHROPIC_API_KEY" \
        -H "anthropic-version: 2026-10-21" \
        -H "anthropic-beta: managed-agents-2026-04-01" \
        -H "content-type: application/json" \
        -H "traceparent: $TRACEPARENT" \
        -d '{
          "events": [
            {
              "type": "user.message",
              "content": [{"type": "text", "text": "Summarize the open issues"}]
            }
          ]
        }'
      ```

      ```python Python
      import os
      import httpx2
      from opentelemetry.trace.propagation.tracecontext import TraceContextTextMapPropagator

      headers = {
          "x-api-key": os.environ["ANTHROPIC_API_KEY"],
          "anthropic-version": "2026-10-21",
          "anthropic-beta": "managed-agents-2026-04-01",
          "content-type": "application/json",
      }
      # 将当前活动 span 的 trace context（追踪上下文）复制到请求标头中。
      TraceContextTextMapPropagator().inject(headers)

      httpx2.post(
          f"https://api.anthropic.com/v1/sessions/{os.environ['SESSION_ID']}/events",
          headers=headers,
          json={
              "events": [
                  {
                      "type": "user.message",
                      "content": [{"type": "text", "text": "Summarize the open issues"}],
                  },
              ],
          },
      )
      ```

      ```typescript TypeScript
      import { context, defaultTextMapSetter } from "@opentelemetry/api";
      import { W3CTraceContextPropagator } from "@opentelemetry/core";

      const headers: Record<string, string> = {
        "x-api-key": process.env.ANTHROPIC_API_KEY!,
        "anthropic-version": "2026-10-21",
        "anthropic-beta": "managed-agents-2026-04-01",
        "content-type": "application/json"
      };
      // 将活动 span 的 trace context（追踪上下文）复制到请求头中。
      new W3CTraceContextPropagator().inject(context.active(), headers, defaultTextMapSetter);

      await fetch(`https://api.anthropic.com/v1/sessions/${process.env.SESSION_ID}/events`, {
        method: "POST",
        headers,
        body: JSON.stringify({
          events: [
            {
              type: "user.message",
              content: [{ type: "text", text: "Summarize the open issues" }]
            }
          ]
        })
      });
      ```

      ```csharp C#
      using System.Diagnostics;
      using System.Net.Http.Json;
      using OpenTelemetry;
      using OpenTelemetry.Context.Propagation;

      using var httpClient = new HttpClient();
      using var request = new HttpRequestMessage(
          HttpMethod.Post,
          $"https://api.anthropic.com/v1/sessions/{Environment.GetEnvironmentVariable("SESSION_ID")}/events")
      {
          // JsonContent 会设置 content-type 标头。
          Content = JsonContent.Create(new
          {
              events = new[]
              {
                  new
                  {
                      type = "user.message",
                      content = new[] { new { type = "text", text = "Summarize the open issues" } },
                  },
              },
          }),
      };
      request.Headers.Add("x-api-key", Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY"));
      request.Headers.Add("anthropic-version", "2026-10-21");
      request.Headers.Add("anthropic-beta", "managed-agents-2026-04-01");

      // 将活动 span 的 trace context（追踪上下文）复制到请求标头中。
      new TraceContextPropagator().Inject(
          new PropagationContext(Activity.Current?.Context ?? default, Baggage.Current),
          request,
          (carrier, key, value) => carrier.Headers.Add(key, value));

      await httpClient.SendAsync(request);
      ```

      ```go Go
      import (
      	"context"
      	"net/http"
      	"os"
      	"strings"

      	"go.opentelemetry.io/otel/propagation"
      )

      func sendMessage(ctx context.Context) error {
      	body := strings.NewReader(`{
      		"events": [
      			{
      				"type": "user.message",
      				"content": [{"type": "text", "text": "Summarize the open issues"}]
      			}
      		]
      	}`)
      	req, err := http.NewRequestWithContext(ctx, http.MethodPost,
      		"https://api.anthropic.com/v1/sessions/"+os.Getenv("SESSION_ID")+"/events", body)
      	if err != nil {
      		return err
      	}
      	req.Header.Set("x-api-key", os.Getenv("ANTHROPIC_API_KEY"))
      	req.Header.Set("anthropic-version", "2026-10-21")
      	req.Header.Set("anthropic-beta", "managed-agents-2026-04-01")
      	req.Header.Set("content-type", "application/json")

      	// 将活动 span 的 trace context（追踪上下文）复制到请求头中。
      	propagation.TraceContext{}.Inject(ctx, propagation.HeaderCarrier(req.Header))

      	resp, err := http.DefaultClient.Do(req)
      	if err != nil {
      		return err
      	}
      	return resp.Body.Close()
      }
      ```

      ```java Java
      import io.opentelemetry.api.trace.propagation.W3CTraceContextPropagator;
      import io.opentelemetry.context.Context;
      import java.net.URI;
      import java.net.http.HttpClient;
      import java.net.http.HttpRequest;
      import java.net.http.HttpResponse;

      void main() throws Exception {
          var request = HttpRequest.newBuilder(URI.create(
                  "https://api.anthropic.com/v1/sessions/" + System.getenv("SESSION_ID") + "/events"))
              .header("x-api-key", System.getenv("ANTHROPIC_API_KEY"))
              .header("anthropic-version", "2026-10-21")
              .header("anthropic-beta", "managed-agents-2026-04-01")
              .header("content-type", "application/json")
              .POST(HttpRequest.BodyPublishers.ofString("""
                  {
                    "events": [
                      {
                        "type": "user.message",
                        "content": [{"type": "text", "text": "Summarize the open issues"}]
                      }
                    ]
                  }
                  """));

          // 将活动 span 的 trace context（追踪上下文）复制到请求头中。
          W3CTraceContextPropagator.getInstance()
              .inject(Context.current(), request, HttpRequest.Builder::header);

          HttpClient.newHttpClient().send(request.build(), HttpResponse.BodyHandlers.ofString());
      }
      ```

      ```php PHP
      use OpenTelemetry\API\Trace\Propagation\TraceContextPropagator;

      $headers = [
          'x-api-key' => getenv('ANTHROPIC_API_KEY'),
          'anthropic-version' => '2026-10-21',
          'anthropic-beta' => 'managed-agents-2026-04-01',
          'content-type' => 'application/json',
      ];
      // 将活动 span 的 trace context（追踪上下文）复制到请求头中。
      TraceContextPropagator::getInstance()->inject($headers);

      $ch = curl_init('https://api.anthropic.com/v1/sessions/' . getenv('SESSION_ID') . '/events');
      curl_setopt_array($ch, [
          CURLOPT_RETURNTRANSFER => true,
          CURLOPT_HTTPHEADER => array_map(
              static fn ($name, $value) => "$name: $value",
              array_keys($headers),
              $headers,
          ),
          CURLOPT_POSTFIELDS => json_encode([
              'events' => [
                  [
                      'type' => 'user.message',
                      'content' => [
                          ['type' => 'text', 'text' => 'Summarize the open issues'],
                      ],
                  ],
              ],
          ]),
      ]);
      curl_exec($ch);
      ```

      ```ruby Ruby
      require "json"
      require "net/http"
      require "opentelemetry"

      headers = {
        "x-api-key" => ENV.fetch("ANTHROPIC_API_KEY"),
        "anthropic-version" => "2026-10-21",
        "anthropic-beta" => "managed-agents-2026-04-01",
        "content-type" => "application/json"
      }
      # 将当前活动 span 的 trace context（追踪上下文）复制到请求标头中。
      OpenTelemetry::Trace::Propagation::TraceContext.text_map_propagator.inject(headers)

      Net::HTTP.post(
        URI("https://api.anthropic.com/v1/sessions/#{ENV.fetch("SESSION_ID")}/events"),
        {
          events: [
            {
              type: "user.message",
              content: [{type: "text", text: "Summarize the open issues"}]
            }
          ]
        }.to_json,
        headers
      )
      ```
    </CodeGroup>
  </Tab>
</Tabs>

当您通过 [Messages API 的 MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)或 [Managed Agents MCP 连接器](https://platform.claude.com/docs/zh-CN/managed-agents/mcp-connector)连接您的 MCP 服务器时，API 还会将追踪传递给您的服务器。这样，该服务器就可以将其自己的 span 放在 API span 之下。在 `tools/call` 请求的 `params._meta` 中，API 会发送一个 W3C `traceparent`，它引用该次工具使用的 [`anthropic.mcp_tool_use`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-tool-use-anthropic-mcp-tool-use-and-anthropic-custom-tool-use) span。使用 Messages API 的 MCP 连接器时，发送给您的 MCP 服务器的其他请求和通知也带有 `traceparent`，它引用 [`anthropic.messages.create`](https://platform.claude.com/docs/zh-CN/api/opentelemetry/spans#anthropic-messages-create) span。如果 API 不追踪您的 Messages API 请求（例如该请求的 `traceparent` 标头中采样标志关闭时），您的 MCP 服务器不会收到 `traceparent`。

## 后续步骤

<CardGroup cols={2}>
  <Card title="设置 SDK 追踪" icon="code-brackets" href="https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk">
    获得对客户端的可观测性。
  </Card>

  <Card title="查看示例追踪" icon="sitemap" href="https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#example-traces-and-how-to-read-them">
    跟踪一个请求从您的客户端进入 Claude API 的全过程。
  </Card>
</CardGroup>

## 交付与健康状态

以下规则适用于 API 发送到您目标的 span。

* **至少一次：** 目标可能会多次收到同一个 span。要去除重复项，请使用追踪 ID 和 span ID。
* **尽力而为：** span 可能在交付前丢失。API 无法重新发送丢失的 span。
* **压缩：** API 会使用 gzip 压缩请求，并设置标头 `Content-Encoding: gzip`。如果您的端点对压缩的请求返回 `400`、`406`、`415` 或 `422` 响应，API 会立即以不压缩的方式再次发送相同的 span。之后在大约一天内，API 通常以不压缩的方式向该目标发送后续请求。
* **重试：** 在超时、连接、TLS 或 DNS 失败、地址无效，或收到 `408`、`429` 或 `5xx` 响应后，API 会在稍后重试交付，次数有限。它不会重试其他失败。
* **超时：** 对于每个请求，您的端点必须在 5 秒内完成 TLS 握手，并在 5 秒内发送响应标头。每次交付总共在 10 秒后超时，包括重新发送。
* **用户代理：** 每个请求都带有标头 `User-Agent: Claude-Platform`。

**投递**列显示每个目标的状态。该状态最多可能延迟一分钟。

| 交付状态     | 含义                                 |
| -------- | ---------------------------------- |
| **正常**   | 最后一次交付成功。                          |
| **失败**   | 最后一次交付失败。要查看错误，请将指针悬停在**失败**上。     |
| **暂无数据** | API 尚未尝试交付。                        |
| **已禁用**  | 该目标不接收 span。要查看原因，请将指针悬停在**已禁用**上。 |

在以下情况下，API 会禁用目标：

* 端点发送 `3xx` 重定向。
* 所有交付在 24 小时内均失败。

目标被禁用期间，API 会丢弃其 span，即使之后重新启用，这些 span 也不会被发送。要恢复导出，请修复原因，并在该目标的**更多操作**菜单中使用**测试**确认修复。然后，在 Console 的 [OpenTelemetry 设置页面](https://platform.claude.com/settings/telemetry-destinations)上的**工作区**筛选器显示**全部**时，在同一菜单中点击**启用**。导出会在大约一分钟内重新开始。在下一次投递之前，**投递**列可能仍显示**失败**和之前的错误。

要更改目标，请在**工作区**筛选器显示**全部**时打开其**更多操作**菜单。然后点击**编辑**、**禁用**、**启用**或**删除**。禁用或删除目标后，投递会在大约一分钟内停止。如果您更改了 **OTLP 端点 URL** 的主机，请重新输入标头值。如果该目标有 **OTLP 追踪端点 URL**，请为其设置相同的新主机，或将其清空。

## 限制

| 限制                  | 值         |
| ------------------- | --------- |
| 每个组织的目标数量（包括已禁用的目标） | 10        |
| 您为一个目标选择的工作区数量      | 1,000     |
| 每个目标的请求头数量          | 20        |
| 请求头名称的长度            | 128 字节    |
| 请求头值的长度             | 4,096 字节  |
| 所有请求头名称和值的总长度       | 16,384 字节 |
| 端点 URL 的长度          | 2,048 个字符 |

## 零数据保留

Span（跨度）的 ZDR 资格与生成它们的功能相同。Claude Managed Agents 不符合 ZDR 资格。请参阅[功能资格](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)。

## 故障排除

| 问题                            | 检查内容                                                                                                                                                                                                                                                                                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 没有 API span                   | 工作区需要一个覆盖它的已启用目标。请参阅[为目标选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#choose-workspaces-for-a-destination)。目标开始覆盖某个工作区后，来自该工作区的 span 可能需要最多一分钟才会到达。对于 `traceparent` 标头中采样标志关闭的请求，或发往 [OpenAI 兼容端点](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/libraries/openai-sdk)的请求，API 不会发出 span。 |
| **投递**列显示**失败**               | 将指针悬停在其上以查看最后一个错误。要找出可能的原因，请参阅[投递错误](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#delivery-errors)。                                                                                                                                                                                                        |
| **投递**列显示**已禁用**              | 将指针悬停在其上以查看原因。有关原因，请参阅[API 何时禁用目标](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#delivery-and-health)。                                                                                                                                                                                                      |
| span 中没有提示或响应，但目标已设置为**完整内容** | 您的组织可能与 Anthropic 之间存在 HIPAA、FERPA 或 K-12 数据处理限制。请参阅[内容模式](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry#choose-a-content-mode)。                                                                                                                                                                                |

### 投递错误

当投递失败时，**投递**列会显示**失败**。将指针悬停在其上以查看错误，然后找出可能的原因：

| 错误                                                | 可能的原因                                                                                                         |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| HTTP error (status 401) 或 HTTP error (status 403) | 身份验证请求头错误                                                                                                     |
| HTTP error (status 404)                           | **OTLP 端点 URL** 或 **OTLP 追踪端点 URL** 中的路径错误                                                                    |
| Invalid address                                   | 端点的主机名不存在，或无法解析为公共 IP 地址                                                                                      |
| Temporary DNS failure                             | 端点的主机名 DNS 查询失败，例如因为其 DNS 服务器返回了错误                                                                            |
| TLS failure                                       | TLS 问题，例如端点的证书问题                                                                                              |
| Timeout                                           | 端点未及时响应。请参阅[投递超时](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/api#delivery-and-health) |
| Connection failure                                | API 无法连接到端点                                                                                                   |
| Redirect (status 3xx)                             | 端点重定向了请求。API 不会跟随重定向，因此请使用它重定向到的 URL                                                                          |
| HTTP error (status 408、429 或 5xx)                 | 端点过载、限制了请求速率或发生了错误                                                                                            |
| HTTP error (其他 status 4xx)                        | 端点拒绝了请求，例如因为请求的大小或格式                                                                                          |

## 可用性

API 追踪在所有模型上处于 beta 阶段，并具备零数据保留资格。

### API 支持

#### [Messages API](https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages)

* 可用性：可用

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                                                                                                     | HIPAA            |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| 状态   | 具备资格（不包括 [涵盖的模型](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#model-specific-data-retention-requirements)） | 不具备资格            |
| 详细信息 | Beta 版。span 的 ZDR 资格与生成它们的功能相同。HIPAA 组织可以导出元数据。SDK span 直接从您的环境发送到您的导出器。送达后，数据由您选择的服务（位于 Anthropic 之外）持有，并受该服务的保留和访问控制约束。                 | HIPAA 组织可以导出元数据。 |

适用的端点：

* `/v1/organizations/telemetry_destinations`
