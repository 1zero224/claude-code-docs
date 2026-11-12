---
title: 订阅 webhook
url: https://platform.claude.com/docs/zh-CN/managed-agents/webhooks
description: 在发生重大事件时获得通知，无需轮询。
featureMetadata:
  topic:
    title: Managed Agents
    url: https://platform.claude.com/docs/en/managed-agents/overview
  status: beta
  betaHeader: managed-agents-2026-04-01
  zdr:
    eligibility: not-eligible
    note: 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。
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

会话是长时间运行的交互。虽然大多数实时交互通过 [SSE 事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)进行，但 webhook 会在发生重大状态变化时通知您。

Webhook 事件返回事件的 `type` 和 `id`，而不是完整对象。当您收到 webhook 事件时，需要通过 `GET` 调用直接获取该对象。这样可以避免在重试时传递过时的数据，并使每次投递都保持较小的体积。

## 支持的事件类型

<Tabs>
  <Tab title="会话事件">
    其中一些事件的名称与会话[事件流](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming)上对应事件的名称不同。例如，事件流中的 `session.status_idle` 和 `session.status_running` 分别对应 `session.status_idled` 和 `session.status_run_started` webhook 事件。

    | 事件                                 | 触发条件                                                                                                                                                                                                                                                                                                                                                            |
    | ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `session.status_run_started`       | 智能体开始运行。每次会话状态转换为 `running` 时都会触发。                                                                                                                                                                                                                                                                                                                              |
    | `session.status_idled`             | 智能体正在等待输入，例如工具权限批准或新的用户消息。                                                                                                                                                                                                                                                                                                                                      |
    | `session.budget_reached`           | 会话达到其[预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)并已暂停。对于您设置的每个预算值最多触发一次；更改预算会重新启用该事件。                                                                                                                                                                                                                                                    |
    | `session.status_rescheduled`       | 发生了暂时性错误，会话正在自动重试。                                                                                                                                                                                                                                                                                                                                              |
    | `session.status_terminated`        | 会话已终止，原因可能是发生了不可恢复的错误，也可能是会话已被归档。                                                                                                                                                                                                                                                                                                                               |
    | `session.thread_created`           | 打开了新的[多智能体线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)：会话所运行的智能体调用的另一个智能体开始工作，或者正在咨询会话的[顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)。它还会针对[工作流运行](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs#a-runs-threads)的每个线程触发，本表中的另外两个线程事件也是如此。 |
    | `session.thread_idled`             | [多智能体交互](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)中的某个智能体正在等待输入。                                                                                                                                                                                                                                                                   |
    | `session.thread_terminated`        | 某个[多智能体线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads)已终止，原因可能是该线程已被归档，也可能是其重试次数已用尽。完成工作的子智能体会进入 `idle` 状态，而不是 `terminated`（顾问线程在其咨询完成后终止，工作流运行的线程在服务器将其归档时终止，即在运行结束时）。仅针对子线程触发；主线程的结束（包括归档整个会话）仅以 `session.status_terminated` 的形式呈现。                                                                                                |
    | `session.outcome_evaluation_ended` | 单次迭代的[结果评估](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes)已完成。                                                                                                                                                                                                                                                                          |
    | `session.updated`                  | 会话属性已更改（例如，其名称或配置已更新）。                                                                                                                                                                                                                                                                                                                                          |
    | `session.deleted`                  | 会话已被永久删除。已没有可获取的对象，因此请将该事件本身视为最终结果。                                                                                                                                                                                                                                                                                                                             |
  </Tab>

  <Tab title="保管库事件">
    | 事件                                | 触发条件                                                                            |
    | --------------------------------- | ------------------------------------------------------------------------------- |
    | `vault.created`                   | 保管库已创建。                                                                         |
    | `vault.archived`                  | 保管库已归档。同时还会为每个底层凭据发出一个 `vault_credential.archived` 事件。                          |
    | `vault.deleted`                   | 保管库已删除。同时还会为每个底层凭据发出一个 `vault_credential.deleted` 事件。已没有可获取的对象，因此请将该事件本身视为最终结果。 |
    | `vault_credential.created`        | 凭据已创建。                                                                          |
    | `vault_credential.archived`       | 凭据已归档，可能是直接归档，也可能是由于保管库归档所致。                                                    |
    | `vault_credential.deleted`        | 凭据已删除，可能是直接删除，也可能是由于保管库删除所致。已没有可获取的对象，因此请将该事件本身视为最终结果。                          |
    | `vault_credential.refresh_failed` | 无法刷新 `mcp_oauth` 凭据（刷新令牌无效，或 OAuth 服务器返回不可恢复的错误）。                               |
  </Tab>

  <Tab title="智能体事件">
    这些事件跟踪您工作区中智能体资源的生命周期，与在会话事件流上传递的智能体事件不同。

    | 事件               | 触发条件                                                                                                              |
    | ---------------- | ----------------------------------------------------------------------------------------------------------------- |
    | `agent.created`  | 智能体已创建。                                                                                                           |
    | `agent.updated`  | 发布了[智能体的新版本](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup#update-an-agent)。不创建新版本的更新不会触发此事件。 |
    | `agent.archived` | 智能体已归档。                                                                                                           |
    | `agent.deleted`  | 智能体已被永久删除。已没有可获取的对象，因此请将该事件本身视为最终结果。                                                                              |
  </Tab>

  <Tab title="部署事件">
    | 事件                    | 触发条件                                                                                                                                                                                    |
    | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `deployment.created`  | [计划部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments)已创建。                                                                                                 |
    | `deployment.updated`  | 部署属性已更改（例如，其计划已更新）。                                                                                                                                                                     |
    | `deployment.paused`   | 部署已暂停，可能是应请求暂停，也可能是在计划运行因不可恢复的错误（例如子智能体已归档或环境已归档）而失败时自动暂停。可恢复的失败（包括速率限制）不会暂停部署。请参阅[失败行为](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments#failure-behavior)。 |
    | `deployment.unpaused` | 部署已取消暂停，恢复其计划。                                                                                                                                                                          |
    | `deployment.archived` | 部署已归档，可能是直接归档，也可能是因为其智能体已被归档。如果智能体被删除，计划部署会在其下一次计划运行时被归档；没有计划的部署不会被自动归档。                                                                                                                |
    | `deployment.deleted`  | 部署已被永久删除。已没有可获取的对象，因此请将该事件本身视为最终结果。                                                                                                                                                     |
  </Tab>

  <Tab title="部署运行事件">
    | 事件                         | 触发条件                                                                                                                                                                                                                                                                                         |
    | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `deployment_run.started`   | 计划运行已开始。只有计划运行会发出 `deployment_run` 事件；[手动运行](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments#trigger-a-manual-run)不会。                                                                                                                                            |
    | `deployment_run.succeeded` | 计划运行已创建其会话。该事件携带的 `data.id`（运行 ID）与该运行的 `deployment_run.started` 事件相同。要跟踪会话的工作，请订阅其会话事件（**会话事件**选项卡），或获取[部署运行](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments#deployment-runs)以得到其 `session_id` (java: `sessionId`; php: `sessionID`; csharp, go: `SessionID`)。 |
    | `deployment_run.failed`    | 计划运行未创建会话。该事件携带的 `data.id` 与该运行的 `deployment_run.started` 事件相同。获取[部署运行](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments#deployment-runs)以查看错误详情。                                                                                                                 |
  </Tab>

  <Tab title="环境事件">
    | 事件                     | 触发条件                                        |
    | ---------------------- | ------------------------------------------- |
    | `environment.created`  | 环境已创建。                                      |
    | `environment.updated`  | 环境已更新，且至少有一个字段发生了更改。无实际变更的更新不会发出任何事件。       |
    | `environment.archived` | 环境已归档。重新归档已归档的环境不会发出任何事件。                   |
    | `environment.deleted`  | 环境已删除，包括删除已归档的环境。已没有可获取的对象，因此请将该事件本身视为最终结果。 |

    环境的[工作项](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes)不会发出任何 webhook 事件。
  </Tab>

  <Tab title="记忆存储事件">
    | 事件                      | 触发条件                                                                                                          |
    | ----------------------- | ------------------------------------------------------------------------------------------------------------- |
    | `memory_store.created`  | 记忆存储已创建，可能由您创建，也可能由 Anthropic 运营的、克隆您现有存储之一的进程创建。                                                             |
    | `memory_store.archived` | 记忆存储已归档。重新归档已归档的存储不会发出任何事件。                                                                                   |
    | `memory_store.deleted`  | 记忆存储已删除，包括删除已归档的存储。删除存储会级联删除其记忆和记忆版本，但不会为每条记忆发出事件；单个 `memory_store.deleted` 事件即为信号。已没有可获取的对象，因此请将该事件本身视为最终结果。 |

    单个[记忆](https://platform.claude.com/docs/zh-CN/managed-agents/memory)和记忆版本不会发出任何 webhook 事件。
  </Tab>
</Tabs>

## 注册端点

在 Claude Console 中，前往 [Webhook](https://platform.claude.com/settings/workspaces/default/webhooks) 页面（`platform.claude.com/settings/workspaces/default/webhooks`）。每个端点属于一个工作区。请在侧边栏的工作区选择器中选择您需要的工作区。

Webhook 端点包括：

* **URL：** 必须是 443 端口上的 HTTPS，且主机名可公开解析。
* **事件类型：** 此端点接收的 `data.type` 值列表。端点只接收其已订阅的事件。
* **签名密钥：** 创建时生成的 32 字节、以 `whsec_` 为前缀的密钥。它只显示一次，因此请妥善保存，以便验证 webhook 投递。

## 验证签名

每次投递都带有 `webhook-id`、`webhook-timestamp` 和 `webhook-signature` 标头。在除 C# 以外的所有 SDK 中，使用 `unwrap()` (csharp, go: `Unwrap()`) 辅助函数，一步完成签名验证和事件解析。如果签名无效或负载超过 5 分钟，它会抛出异常。

将 `ANTHROPIC_WEBHOOK_SIGNING_KEY` 设置为创建端点时显示的、以 `whsec_` 为前缀的密钥。

<CodeGroup exclude="shell:cURL, shell:CLI, csharp">
  ```python Python
  from flask import Flask, request
  import anthropic
  import standardwebhooks  # pip install "anthropic[webhooks]"

  client = anthropic.Anthropic()  # reads ANTHROPIC_WEBHOOK_SIGNING_KEY from env
  app = Flask(__name__)


  @app.route("/webhook", methods=["POST"])
  def webhook():
      try:
          # 如果签名无效或负载已过期，unwrap() 会抛出异常
          event = client.beta.webhooks.unwrap(
              request.get_data(as_text=True),
              headers=dict(request.headers),
          )
      except standardwebhooks.WebhookVerificationError:
          return "invalid signature", 400

      if event.data.type == "session.status_idled":
          print("session idled:", event.data.id)
      # 处理其他事件类型

      return "", 200
  ```

  ```typescript TypeScript
  import express from "express";
  import Anthropic from "@anthropic-ai/sdk";

  const client = new Anthropic(); // reads ANTHROPIC_WEBHOOK_SIGNING_KEY from env
  const app = express();

  // 重要：请使用 express.raw()，而非 express.json()。签名是基于原始字节计算的。
  app.post("/webhook", express.raw({ type: "application/json" }), (req, res) => {
    let event;
    try {
      // 如果签名无效或负载已过期，unwrap() 会抛出异常
      event = client.beta.webhooks.unwrap(req.body.toString("utf8"), {
        headers: req.headers as Record<string, string>
      });
    } catch {
      return res.status(400).send("invalid signature");
    }

    switch (event.data.type) {
      case "session.status_idled":
        console.log("session idled:", event.data.id);
        break;
      // 处理其他事件类型
    }

    res.sendStatus(200);
  });
  ```

  <CodeGroupItem language="C#">
    C# SDK 没有 `Unwrap()`。请使用 [Standard Webhooks](https://www.standardwebhooks.com/) 库验证签名，然后将请求体反序列化为 `UnwrapWebhookEvent`。
  </CodeGroupItem>

  ```go Go
  package main

  import (
  	"fmt"
  	"io"
  	"net/http"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  )

  var client = anthropic.NewClient() // reads ANTHROPIC_WEBHOOK_SIGNING_KEY from env

  func webhook(w http.ResponseWriter, r *http.Request) {
  	body, err := io.ReadAll(r.Body)
  	if err != nil {
  		http.Error(w, "could not read body", http.StatusBadRequest)
  		return
  	}

  	// 如果签名无效或负载已过期，Unwrap 会返回错误
  	event, err := client.Beta.Webhooks.Unwrap(body, r.Header)
  	if err != nil {
  		http.Error(w, "invalid signature", http.StatusBadRequest)
  		return
  	}

  	switch event.Data.Type {
  	case "session.status_idled":
  		fmt.Println("session idled:", event.Data.ID)
  		// 处理其他事件类型
  	}

  	w.WriteHeader(http.StatusOK)
  }

  func main() {
  	http.HandleFunc("/webhook", webhook)
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.core.UnwrapWebhookParams;
  import com.anthropic.core.http.Headers;
  import com.sun.net.httpserver.HttpServer;

  // 从环境变量读取 ANTHROPIC_WEBHOOK_SIGNING_KEY
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  void main() throws Exception {
      var server = HttpServer.create(new InetSocketAddress(8000), 0);
      server.createContext("/webhook", exchange -> {
          var body = new String(exchange.getRequestBody().readAllBytes());
          var headers = Headers.builder();
          exchange.getRequestHeaders().forEach(headers::put);

          try {
              // 如果签名无效或负载已过期，unwrap() 会抛出异常
              var event = client.beta().webhooks().unwrap(
                  UnwrapWebhookParams.builder()
                      .body(body)
                      .headers(headers.build())
                      .build());

              event.data().sessionStatusIdled().ifPresent(idled ->
                  IO.println("session idled: " + idled.id()));
              // 处理其他事件类型

              exchange.sendResponseHeaders(200, -1);
          } catch (Exception _) {
              exchange.sendResponseHeaders(400, -1);
          }
          exchange.close();
      });
  }
  ```

  ```php PHP
  use Anthropic\Client;
  use Anthropic\Core\Exceptions\WebhookException;

  $client = new Client(); // reads ANTHROPIC_WEBHOOK_SIGNING_KEY from env

  $body = file_get_contents('php://input');
  $headers = getallheaders();

  try {
      // 如果签名无效或负载已过期，unwrap() 会抛出异常
      $event = $client->beta->webhooks->unwrap($body, headers: $headers);
  } catch (WebhookException) {
      http_response_code(400);
      exit('invalid signature');
  }

  match ($event->data->type) {
      'session.status_idled' => print "session idled: {$event->data->id}\n",
      // 处理其他事件类型
      default => null,
  };

  http_response_code(200);
  ```

  ```ruby Ruby
  require "sinatra"
  require "anthropic"

  client = Anthropic::Client.new # reads ANTHROPIC_WEBHOOK_SIGNING_KEY from env

  post "/webhook" do
    headers = request.env
      .select { |key, _| key.start_with?("HTTP_") }
      .transform_keys { it.delete_prefix("HTTP_").downcase.tr("_", "-") }

    begin
      # 如果签名无效或负载已过期，unwrap 会抛出异常
      event = client.beta.webhooks.unwrap(request.body.read, headers: headers)
    rescue StandardError
      halt 400, "invalid signature"
    end

    if event.data.type == :"session.status_idled"
      puts "session idled: #{event.data.id}"
    end
    # 处理其他事件类型

    status 200
  end
  ```
</CodeGroup>

## 处理事件

解析请求体，根据 `data.type` 进行分支处理，并按 ID 获取资源。返回任意 `2xx` 以确认接收。任何其他响应都会计入该端点的失败：`3xx` 会立即禁用该端点（从不跟随重定向），而其他失败会被重试；有关重试和自动禁用规则，请参阅[投递行为](https://platform.claude.com/docs/zh-CN/managed-agents/webhooks#delivery-behavior)。

每个事件负载都具有相同的结构，包括事件类型、标识符以及事件发生时的时间戳。

```json
{
  "type": "event",
  "id": "whe_9d5c1f7e...",
  "created_at": "2026-03-18T14:05:22Z",
  "data": {
    "type": "session.status_idled",
    "id": "sesn_01XYZ...",
    "organization_id": "8a3d2f1e-...",
    "workspace_id": "c7b0e4d9-..."
  }
}
```

<CodeGroup exclude="shell:cURL, shell:CLI">
  ```python Python
  if event.data.type == "session.status_idled":
      session = client.beta.sessions.retrieve(event.data.id)
      notify_user(session)
  return "", 204
  ```

  ```typescript TypeScript
  if (event.data.type === "session.status_idled") {
    const session = await client.beta.sessions.retrieve(event.data.id);
    notifyUser(session);
  }
  res.sendStatus(204);
  ```

  ```csharp C#
  if (webhookEvent.Data.TryPickSessionStatusIdled(out var idled))
  {
      var session = await client.Beta.Sessions.Retrieve(idled.ID);
      NotifyUser(session);
  }
  return Results.StatusCode(204);
  ```

  ```go Go
  if event.Data.Type == "session.status_idled" {
  	session, err := client.Beta.Sessions.Get(r.Context(), event.Data.ID, anthropic.BetaSessionGetParams{})
  	if err != nil {
  		panic(err)
  	}
  	notifyUser(session)
  }
  w.WriteHeader(http.StatusNoContent)
  ```

  ```java Java
  event.data().sessionStatusIdled().ifPresent(idled -> {
      var session = client.beta().sessions().retrieve(idled.id());
      notifyUser(session);
  });
  exchange.sendResponseHeaders(204, -1);
  ```

  ```php PHP
  if ($event->data->type === 'session.status_idled') {
      $session = $client->beta->sessions->retrieve($event->data->id);
      notifyUser($session);
  }
  http_response_code(204);
  ```

  ```ruby Ruby
  if event.data.type == :"session.status_idled"
    session = client.beta.sessions.retrieve(event.data.id)
    notify_user(session)
  end
  status 204
  ```
</CodeGroup>

顶层的 `event.id` 对每个事件是唯一的，而不是对每次投递唯一。如果您两次收到相同的 `event.id`，那就是一次重试，您可以将其丢弃。

## 投递行为

* **重复：** 一个端点可能会多次收到同一事件，并且每次尝试都会投递相同的顶层 `event.id`（与 `webhook-id` 标头的值相同）。请据此进行去重。

* **订阅范围：** 事件仅投递给在其发出时已订阅该类型的端点。在没有任何端点订阅其类型时发出的事件永远不会被投递，之后再订阅也不会补发，因此请在需要之前订阅相应的事件类型。

* **顺序：** 不保证事件按其发生的顺序投递：即使结果先产生，`session.status_idled` 也可能在 `session.outcome_evaluation_ended` 之前到达，并且同一资源的 `.deleted` 事件可能在 `.archived` 事件之前到达。请根据您获取的资源来驱动状态，而不是根据事件到达的顺序。

* **重试：** 对于每个端点和事件，Anthropic 最多进行三次投递尝试（触发自动禁用的响应（本节后文会介绍）永远不会被重试），尝试之间采用 5 到 120 秒之间带抖动的指数退避。每次尝试都会投递相同的 `event.id`。最后一次尝试失败后，该事件将被丢弃：它不会排队等待稍后投递，也不会有任何信号表明它已丢失。Webhook 不是持久化日志，因此如果您需要观察每一次状态转换，请通过 API 列出或获取资源来进行核对。

* **时间戳：** `webhook-timestamp` 标头在投递尝试被签名时生成，并在每次重试时重新生成，因此重试不会被 SDK 的新鲜度检查拒绝。它是投递尝试的时钟，而不是事件的时钟：请使用事件负载中的 `created_at` (java, php: `createdAt`; csharp, go: `CreatedAt`) 来确定事件发生的时间。

* **自动禁用：** 在以下三种情况下，端点会被自动设置为 `disabled`，并附带机器可读的 `disabled_reason`：

  * 端点返回 `3xx` 响应。从不跟随重定向；这会在第一次尝试时立即禁用该端点，原因为 `auto-disabled: endpoint URL returned a redirect (3xx)`。如果您的端点地址发生变化，请使用新 URL 和相同的事件类型添加一个新端点，然后删除旧端点。新端点有其自己的签名密钥，因此请更新 `ANTHROPIC_WEBHOOK_SIGNING_KEY`。
  * Anthropic 连接时，端点的 URL 解析为非公共 IP 地址。这会立即禁用该端点，原因为 `auto-disabled: endpoint URL resolved to an invalid address`。
  * 在一段持续的时间内，对该端点的投递持续失败，原因为 `auto-disabled after sustained delivery failures`。触发条件是端点连续失败的时长，而不是投递次数。单个 `2xx` 即可重置该时间窗口，因此一个间歇性失败的事件不会导致端点被禁用。

  这三种情况都是可逆的：解决问题后，在 Console 中重新启用该端点即可，除非您已用新端点替换了它。端点被禁用期间发出的事件不会被重放。

## 可用性

Webhook在所有模型上处于 beta 阶段。

### API 支持

#### [Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview)

* 可用性：可用

| 状态   | 云平台                                                                                                                   |
| ---- | --------------------------------------------------------------------------------------------------------------------- |
| Beta | Claude API, [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws) |

### 所需的 beta 请求头

[Beta 请求头概览](https://platform.claude.com/docs/zh-CN/api/beta-headers)

Beta 请求头让您可以在实验性功能和新模型能力成为标准 API 的一部分之前访问它们。

* `managed-agents-2026-04-01`

### 数据处理

[数据处理概览](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention#feature-eligibility)

| 项目   | 零数据保留                                                         | HIPAA                             |
| ---- | ------------------------------------------------------------- | --------------------------------- |
| 状态   | 不具备资格                                                         | 不具备资格                             |
| 详细信息 | 会话是有状态的资源；对话记录会一直保留，直到您将其删除。适用于所有 Managed Agents 子功能，包括自托管沙箱。 | 适用于所有 Managed Agents 子功能，包括自托管沙箱。 |

适用的端点：

* `/v1/agents`
* `/v1/sessions`
* `/v1/environments`
