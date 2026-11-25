---
title: 为每个客户使用单独的凭据
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer
description: 使用每个客户自己的 OAuth 访问令牌调用 Claude，这样您无法完全控制的代码只能向该客户计费。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：根据您与 Anthropic 之间的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在 Beta 测试期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

您通过 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing) 关联的每个客户都带有自己的凭据：一个只能向该客户计费的 OAuth 访问令牌。当您无法完全控制的代码为某一个客户调用 Claude 时（例如在代理的沙箱或单租户部署中），请使用它。这样，该代码中的缺陷或泄露只能向该客户计费。

如果每个向客户计费的请求都来自您自己的后端，则您不需要这些凭据。请[以您的操作员服务账户向客户计费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#bill-a-request)，并跳过本页。

## 开始之前

* **客户的 OAuth 令牌和连接 ID：**&#x5728;您[关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#save-the-connection-and-its-tokens)时，授权码交换已返回这些内容。连接是将客户的 Claude 组织与您的某个[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)绑定在一起的记录。
* **刷新令牌：**&#x60A8;的应用已开启 [**Offline access**](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#choose-whether-to-get-refresh-tokens)，并且客户已批准 `billing:attribution offline_access`。未批准 `offline_access` 的客户可以通过[再次批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#if-a-customer-approves-again)获得刷新令牌，这会替换其连接而无需取消关联。
* **对于刷新示例：**&#x4E00;个 client assertion（客户端断言）：即由您的身份提供商为您应用的 **Expected audience** 签发的 JWT，并已设置为您应用的[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#set-up-client-authentication)。示例会从 `CLIENT_ASSERTION_FILE` 环境变量指定的文件或从 `CLIENT_ASSERTION` shell 变量中读取它。

## 对 OAuth 令牌保密

将每个客户的访问令牌、刷新令牌及其过期时间存储在只有负责刷新的服务才能读取的密钥存储中。任何持有访问令牌的人都可以在令牌过期前向该客户计费。如果令牌泄露，请先[取消关联该客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#unlink-a-customer)，然后再次关联。如果您丢失了刷新令牌，请[再次关联该客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)。

## 使用客户的访问令牌调用 Claude

发送客户的访问令牌来代替您操作员服务账户的凭据：

* **在 Anthropic SDK 中：**&#x6784;建客户端时，将客户的访问令牌作为客户端的身份验证令牌传入。
* **在 `ant` CLI 中：**&#x4E3A;该命令将 `ANTHROPIC_AUTH_TOKEN` 设置为该令牌。
* **在 HTTP 请求中：**&#x5728; `Authorization` 标头中将其作为 bearer 令牌发送。不要发送 `x-api-key` 标头。

<Warning>
  在任何使用客户访问令牌的地方，都要取消设置 `ANTHROPIC_API_KEY`：某些客户端会改为发送该密钥，Anthropic 可能会向您计费。
</Warning>

这些客户端是 `ant` CLI，以及从 `ANTHROPIC_AUTH_TOKEN` 读取令牌的 SDK 客户端。有关原因和防范措施，请参阅[如果 API 密钥可能进入进程](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#if-an-api-key-can-reach-the-process)。如果您的查找没有为该客户找到令牌，请在构建客户端之前停止。如果令牌为 null，SDK 可能会回退到您自己的凭据并向您计费。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "authorization: Bearer $CUSTOMER_ACCESS_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}]
    }'
  ```

  ```bash CLI
  ANTHROPIC_AUTH_TOKEN="$CUSTOMER_ACCESS_TOKEN" ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  client = anthropic.Anthropic(auth_token=customer_access_token)

  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )

  for block in message.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic({ authToken: customerAccessToken });

  const message = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  });

  for (const block of message.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new() { AuthToken = customerAccessToken };

  var message = await client.Messages.Create(
      new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new UserMessageParam { Content = "Hello, Claude" }],
      }
  );

  foreach (var block in message.Content)
  {
      if (block.TryPickText(out var textBlock))
      {
          Console.WriteLine(textBlock.Text);
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient(option.WithAuthToken(customerAccessToken))

  message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  for _, block := range message.Content {
  	if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.builder()
      .authToken(customerAccessToken)
      .build();
  // ...

  var params = MessageCreateParams.builder()
      .model(Model.CLAUDE_OPUS_5_5)
      .maxTokens(1024)
      .addUserMessage("Hello, Claude")
      .build();
  var message = client.messages().create(params);

  for (var block : message.content()) {
      block.text().ifPresent(textBlock -> IO.println(textBlock.text()));
  }
  ```

  ```php PHP
  $client = new Client(authToken: $customerAccessToken);

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_OPUS_5_5,
  );

  foreach ($message->content as $block) {
      if ($block instanceof \Anthropic\Messages\TextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new(auth_token: customer_access_token)

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  )

  message.content.each do |block|
    puts(block.text) if block.is_a?(Anthropic::Models::TextBlock)
  end
  ```
</CodeGroup>

客户的访问令牌仅适用于以下端点：

* `/v1/messages` 和 `/v1/chat/completions`
* `/v1/messages/count_tokens`
* Models 端点

在 `/v1/messages` 和 `/v1/chat/completions` 上，仅凭访问令牌即可向客户计费。您不需要 `anthropic-user-profile-id` 标头或 [`billing_policy`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#bill-a-request)。

所有其他请求以及所有由您付费的请求，都应以您的操作员服务账户发送。这包括使用计费代码执行的请求，此类请求[无法向客户计费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#if-you-use-code-execution)。

使用该令牌的请求在您设置为应用默认值的工作区中运行。归档该工作区会撤销其中的所有令牌：请参阅[如果您迁移到其他工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-move-to-another-workspace)。如果使用该令牌的请求被拒绝，请参阅 [Linked billing 错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors)。

## 刷新 OAuth 令牌

客户的访问令牌最多持续 1 小时。在其过期之前，将客户的刷新令牌发送到令牌端点以获取一对新令牌。每个刷新令牌只能使用一次，但允许在 30 秒内重复一次，以恢复您丢失的响应。

<Warning>
  如果在首次使用超过 30 秒后再次发送刷新令牌，会终止该客户的连接，您必须重新关联该客户。
</Warning>

为确保您永远不会延迟发送已使用过的令牌：

* 在您的 HTTP 客户端和任何代理服务器中，关闭刷新请求的自动重试。
* 从一个服务刷新每个客户的 OAuth 令牌，每个客户一次只进行一次刷新。如果该服务有副本，请为它们提供共享锁。从读取客户存储的令牌开始持有该锁，直到新令牌存储完毕。
* 在使用新的访问令牌之前，先存储新的刷新令牌。

该请求是发往 `auth.claude.com` 的 `refresh_token` 授权。它携带刷新令牌和您应用的[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#set-up-client-authentication)：`client_id` 和客户端断言。没有 SDK 支持 `auth.claude.com`，因此示例通过 HTTP 调用它。示例展示的是没有故障处理的普通调用：

<CodeGroup exclude="shell:CLI">
  ```bash cURL
  curl https://auth.claude.com/v1/oauth/token \
    -H "content-type: application/json" \
    -d @- <<JSON
  {
    "grant_type": "refresh_token",
    "refresh_token": "$CUSTOMER_REFRESH_TOKEN",
    "client_id": "clid_01ABcDeFgHiJkLmNoPqRsTuV",
    "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
    "client_assertion": "$CLIENT_ASSERTION"
  }
  JSON
  ```

  <CodeGroupItem language="CLI">
    `ant` CLI 没有用于 `auth.claude.com` 的命令。请使用 cURL 或您应用程序的语言发送刷新请求。
  </CodeGroupItem>

  ```python Python
  import os
  from pathlib import Path

  import httpx2  # comes with the Anthropic SDK


  def refresh_tokens(refresh_token: str) -> dict:
      assertion_file = Path(os.environ["CLIENT_ASSERTION_FILE"])

      response = httpx2.post(
          "https://auth.claude.com/v1/oauth/token",
          json={
              "grant_type": "refresh_token",
              "refresh_token": refresh_token,
              "client_id": "clid_01ABcDeFgHiJkLmNoPqRsTuV",
              "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
              "client_assertion": assertion_file.read_text().strip(),
          },
          timeout=10,
      )
      if not response.is_success:
          raise RuntimeError(f"Refresh failed: {response.status_code} {response.text}")
      return response.json()  # the new access_token and refresh_token
  ```

  ```typescript TypeScript
  import { readFile } from "node:fs/promises";

  async function refreshTokens(refreshToken: string) {
    const jwt = await readFile(process.env.CLIENT_ASSERTION_FILE!, "utf8");

    const response = await fetch("https://auth.claude.com/v1/oauth/token", {
      method: "POST",
      redirect: "error",
      signal: AbortSignal.timeout(10_000),
      headers: { "content-type": "application/json" },
      body: JSON.stringify({
        grant_type: "refresh_token",
        refresh_token: refreshToken,
        client_id: "clid_01ABcDeFgHiJkLmNoPqRsTuV",
        client_assertion_type: "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
        client_assertion: jwt.trim()
      })
    });
    if (!response.ok) {
      throw new Error(`Refresh failed: ${response.status} ${await response.text()}`);
    }
    return response.json(); // the new access_token and refresh_token
  }
  ```

  ```csharp C#
  using System.Net.Http.Json;
  using System.Text.Json;

  using var http = new HttpClient { Timeout = TimeSpan.FromSeconds(10) };

  async Task<JsonElement> RefreshTokens(string refreshToken)
  {
      var assertionFile = Environment.GetEnvironmentVariable("CLIENT_ASSERTION_FILE")!;
      var jwt = await File.ReadAllTextAsync(assertionFile);

      using var response = await http.PostAsJsonAsync(
          "https://auth.claude.com/v1/oauth/token",
          new Dictionary<string, string>
          {
              ["grant_type"] = "refresh_token",
              ["refresh_token"] = refreshToken,
              ["client_id"] = "clid_01ABcDeFgHiJkLmNoPqRsTuV",
              ["client_assertion_type"] = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
              ["client_assertion"] = jwt.Trim(),
          }
      );
      if (!response.IsSuccessStatusCode)
      {
          var details = await response.Content.ReadAsStringAsync();
          throw new HttpRequestException($"Refresh failed: {(int)response.StatusCode} {details}");
      }

      // 新的 access_token 和 refresh_token
      return await response.Content.ReadFromJsonAsync<JsonElement>();
  }
  ```

  ```go Go
  import (
  	"bytes"
  	"encoding/json"
  	"fmt"
  	"io"
  	"net/http"
  	"os"
  	"strings"
  	"time"
  )

  var httpClient = &http.Client{Timeout: 10 * time.Second}

  func refreshTokens(refreshToken string) (map[string]any, error) {
  	jwt, err := os.ReadFile(os.Getenv("CLIENT_ASSERTION_FILE"))
  	if err != nil {
  		return nil, err
  	}

  	body, err := json.Marshal(map[string]string{
  		"grant_type":            "refresh_token",
  		"refresh_token":         refreshToken,
  		"client_id":             "clid_01ABcDeFgHiJkLmNoPqRsTuV",
  		"client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
  		"client_assertion":      strings.TrimSpace(string(jwt)),
  	})
  	if err != nil {
  		return nil, err
  	}

  	endpoint := "https://auth.claude.com/v1/oauth/token"
  	response, err := httpClient.Post(endpoint, "application/json", bytes.NewReader(body))
  	if err != nil {
  		return nil, err
  	}
  	defer response.Body.Close()
  	if response.StatusCode/100 != 2 {
  		details, _ := io.ReadAll(response.Body)
  		return nil, fmt.Errorf("refresh failed: %d %s", response.StatusCode, details)
  	}

  	var tokens map[string]any // the new access_token and refresh_token
  	err = json.NewDecoder(response.Body).Decode(&tokens)
  	return tokens, err
  }
  ```

  ```java Java
  import module java.base;
  import module java.net.http;
  import com.fasterxml.jackson.databind.JsonNode;
  import com.fasterxml.jackson.databind.ObjectMapper; // Jackson comes with the Anthropic SDK

  final HttpClient http = HttpClient.newHttpClient();
  final ObjectMapper mapper = new ObjectMapper();

  JsonNode refreshTokens(String refreshToken) throws Exception {
      var jwt = Files.readString(Path.of(System.getenv("CLIENT_ASSERTION_FILE")));

      var body = mapper.createObjectNode()
          .put("grant_type", "refresh_token")
          .put("refresh_token", refreshToken)
          .put("client_id", "clid_01ABcDeFgHiJkLmNoPqRsTuV")
          .put("client_assertion_type", "urn:ietf:params:oauth:client-assertion-type:jwt-bearer")
          .put("client_assertion", jwt.strip());

      var request = HttpRequest.newBuilder(URI.create("https://auth.claude.com/v1/oauth/token"))
          .header("content-type", "application/json")
          .timeout(Duration.ofSeconds(10))
          .POST(HttpRequest.BodyPublishers.ofString(body.toString()))
          .build();
      var response = http.send(request, HttpResponse.BodyHandlers.ofString());
      if (response.statusCode() / 100 != 2) {
          throw new IOException("Refresh failed: " + response.statusCode() + " " + response.body());
      }
      return mapper.readTree(response.body()); // the new access_token and refresh_token
  }
  ```

  ```php PHP
  function refreshTokens(string $refreshToken): mixed
  {
      $jwt = file_get_contents((string) getenv('CLIENT_ASSERTION_FILE'))
          ?: throw new \RuntimeException('Cannot read the client assertion file');

      $body = json_encode([
          'grant_type' => 'refresh_token',
          'refresh_token' => $refreshToken,
          'client_id' => 'clid_01ABcDeFgHiJkLmNoPqRsTuV',
          'client_assertion_type' => 'urn:ietf:params:oauth:client-assertion-type:jwt-bearer',
          'client_assertion' => trim($jwt),
      ], JSON_THROW_ON_ERROR);

      $context = stream_context_create(['http' => [
          'method' => 'POST',
          'header' => 'content-type: application/json',
          'content' => $body,
          'timeout' => 10,
          'ignore_errors' => true, // keep the body of an error response
      ]]);
      $response = file_get_contents('https://auth.claude.com/v1/oauth/token', context: $context);
      $status = http_get_last_response_headers()[0] ?? '';
      if ($response === false || !preg_match('{^HTTP/\S+ 2}', $status)) {
          throw new \RuntimeException("Refresh failed: {$status} {$response}");
      }

      // 新的 access_token 和 refresh_token
      return json_decode($response, associative: true, flags: JSON_THROW_ON_ERROR);
  }
  ```

  ```ruby Ruby
  require "json"
  require "net/http"

  def refresh_tokens(refresh_token)
    jwt = File.read(ENV.fetch("CLIENT_ASSERTION_FILE"))

    body = {
      grant_type: "refresh_token",
      refresh_token:,
      client_id: "clid_01ABcDeFgHiJkLmNoPqRsTuV",
      client_assertion_type: "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
      client_assertion: jwt.strip
    }
    response = Net::HTTP.start("auth.claude.com", 443, use_ssl: true, read_timeout: 10) do |http|
      http.post("/v1/oauth/token", body.to_json, "content-type" => "application/json")
    end
    unless response.is_a?(Net::HTTPSuccess)
      raise "Refresh failed: #{response.code} #{response.body}"
    end

    JSON.parse(response.body) # the new access_token and refresh_token
  end
  ```
</CodeGroup>

响应包含一个新的访问令牌和一个新的刷新令牌。`expires_in` 和 `refresh_token_expires_in` 以秒为单位给出它们的有效期。新的刷新令牌会替换您发送的那个，之前的访问令牌会在几分钟内失效。

在刷新失败后再次发送令牌之前，请阅读[刷新失败时该怎么办](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#if-a-refresh-fails)。有些重试会终止连接，有些失败则意味着您需要暂停所有客户的刷新。[完整刷新示例](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#full-refresh-sample)会处理这些失败。

刷新令牌自授权码交换起最多持续 1 年，刷新不会延长该期限。在其过期之前，请[再次关联该客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)。

## 在沙箱中运行代理

在使用本节之前，请阅读 [Linked Billing Beta Terms](https://www.anthropic.com/legal/linked-billing-terms) 的第 4.1(b) 节。如果您对此有疑问，请咨询您的 Anthropic 客户团队。

当代理在沙箱中为某一个客户工作时，请遵循以下规则：

* 只向沙箱提供访问令牌，切勿提供刷新令牌。由您的后端刷新令牌，并将每个新的访问令牌发送到沙箱。
* 根据沙箱自身的身份（例如其 pod 身份）来决定沙箱服务于哪个客户。切勿根据沙箱发送的值来决定。
* 只向沙箱发送其所服务客户的令牌。
* 不要为沙箱挂载身份令牌，也不要让其访问云元数据服务器。

最后一条规则可防止沙箱获取与您应用的客户端身份验证或您的某条[联合规则](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation)相匹配的 JWT。借助这样的 JWT，沙箱中的代码可能会以您的应用或您的某个服务账户的身份进行身份验证。

Anthropic SDK、Claude Code 和 [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) 可以从配置文件（profile）中的凭据文件读取访问令牌。有关该文件的格式，请参阅[配置文件（profile）格式](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#profile-configuration-file)。要设置沙箱：

<Steps>
  <Step title="添加 profile">
    在沙箱中创建一个配置目录。在其中的 `configs/customer.json` 处添加一个身份验证类型为 `user_oauth` 的 profile。不要为该 profile 设置 `client_id`：如果设置了，SDK 会期望文件中有刷新令牌并自行刷新令牌。

    <File filename="configs/customer.json">
      ```json
      {
        "version": "1.0",
        "authentication": {
          "type": "user_oauth"
        }
      }
      ```
    </File>
  </Step>

  <Step title="写入访问令牌">
    在代理启动之前，将访问令牌写入同一配置目录中的 `credentials/customer.json`，使 `configs` 和 `credentials` 并列存放。每次刷新后立即再次写入。`expires_at` 是令牌的过期时间，以 Unix 秒表示。有关安全写入的代码，请参阅[将每个新的访问令牌写入沙箱时](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#when-you-write-each-new-token-to-a-sandbox)。

    ```json credentials/customer.json
    {
      "type": "oauth_token",
      "access_token": "sk-ant-oat01-...",
      "expires_at": 1793635200
    }
    ```
  </Step>

  <Step title="将客户端指向该 profile">
    将 `ANTHROPIC_CONFIG_DIR` 设置为配置目录，并将 `ANTHROPIC_PROFILE` 设置为 `customer`。
  </Step>

  <Step title="移除所有其他凭据">
    取消设置沙箱中的所有其他凭据，因为每个凭据都可能优先于该 profile。这些凭据包括：

    * `ANTHROPIC_API_KEY` 和 `ANTHROPIC_AUTH_TOKEN` 环境变量
    * Claude Code 的 `CLAUDE_CODE_OAUTH_TOKEN` 环境变量和 `apiKeyHelper` 设置
  </Step>
</Steps>

SDK 无需重启即可获取每个新令牌：请参阅[将每个新的访问令牌写入沙箱时](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#when-you-write-each-new-token-to-a-sandbox)。

## 检查结果

要确认某个请求已向客户计费，请参阅[计费请求的显示位置](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#check-the-result)。要确认令牌仅限于其所属客户：

* 使用该令牌且指定了您另一个用户配置文件的请求会收到 `400 invalid_request_error`。
* 刷新几分钟后，使用之前访问令牌的请求会收到 `401 authentication_error`。

## 决定是否关闭仅操作员身份验证

默认情况下，您的操作员服务账户无需客户的访问令牌即可向任何已关联客户计费。它会指定客户的用户配置文件，并在 [`billing_policy`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#bill-a-request) 中设置 `bill_to: "user_profile"`。此默认行为称为 operator-only authentication（仅操作员身份验证），并且在客户的访问令牌过期后仍然有效。

一旦每个客户的请求都使用其自己的访问令牌，您就可以关闭仅操作员身份验证。此后，只有客户自己的访问令牌才能向其计费，因此指定了错误用户配置文件的缺陷不会向错误的客户计费。

此后，每个客户的请求都依赖于刷新。如果您丢失了某个客户的刷新令牌，则在其[再次关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)之前，您无法向其计费。任何停止刷新的操作（例如关闭 **Offline access** 或清除[默认工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#default-workspace)），都会导致每个客户的请求在其访问令牌过期时失败。

Claude Console 仅检查您的应用是否具有 `billing:attribution`、`offline_access` 和默认工作区。在关闭仅操作员身份验证之前，请自行检查以下内容：

* 您的存储中为每个已关联客户保存了刷新令牌。
* 您使用其访问令牌发出的请求能够正常工作。

要关闭仅操作员身份验证：

1. 在 Console 中打开应用的 **Client authentication** 选项卡。
2. 在操作员服务账户下，开启 **Disable operator-only tokenless authentication** 并在对话框中确认。它在 API 中的名称是 `disable_operator_only_auth`。

此后适用以下规则：

* 仅由服务账户发出、为已关联客户设置 `bill_to: "user_profile"` 的请求会收到 `403 user_profile_not_billable`。该客户仍显示为 **Linked**，因此请在日志中留意此错误。
* 服务账户仍可管理用户配置文件，并发送由您付费的请求。
* 要恢复默认设置，请关闭同一设置。

## 实现细节、规则和特殊情况

本页其余部分是步骤背后的详细信息。内容涵盖刷新失败、完整刷新示例，以及 API 密钥或刷新如何影响正在运行的客户端。仅批准了已弃用作用域的客户没有自己的访问令牌：请参阅[如果您仍在使用已弃用的作用域](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-still-use-the-deprecated-scope)。

### 如果刷新失败

刷新会替换您发送的刷新令牌。当被替换的令牌再次到达时会发生什么，取决于它到达的时间：

* **30 秒内，第一次：**&#x8FD4;回一对新令牌，并撤销您从未收到的那对令牌。这使您可以重复一次丢失了响应的刷新。
* **30 秒内，第一次，但您发送的令牌在被替换时存在时间不足 30 秒：**&#x6536;到 `invalid_grant`，连接保留。
* **30 秒内，之后的任何一次：**&#x6536;到 `invalid_grant`，连接保留。
* **30 秒后：**&#x6536;到 `invalid_grant` 并终止连接。所有令牌都会被撤销，客户显示为 **Unlinked**，在其再次关联之前，任何方式都无法向其计费。

刷新失败后，该怎么做取决于刷新令牌是否到达 `auth.claude.com` 并被使用：

| 刷新得到的结果 | 含义 | 应对方式 |
| ----- | ----- | ----- |
| 请求无法到达 `auth.claude.com`，包括连接超时 | 令牌未被发送 | 稍后再次发送 |
| `429` | 没有任何内容被撤销 | 等待，然后再次发送 |
| `503` 拒绝：带有 `retry-after` 标头，且响应体顶层为 `"type": "error"` | 没有任何内容被撤销 | 等待，然后再次发送 |
| 除 `invalid_grant` 之外的任何其他 `4xx` | 没有任何内容被撤销 | 修复原因，然后再次发送 |
| 任何其他 `5xx`，或请求发出后网络中断 | 可能已经成功 | 立即再发送一次 |
| 未及时收到响应（读取超时） | 可能仍在运行 | 不要再次发送 |
| `invalid_grant` | 该令牌现在无法使用 | 停止发送 |

**等待，然后再次发送：**&#x81F3;少等待 `retry-after` 所指示的时长。如果 `429` 没有 `retry-after` 标头，请先退避再重新发送。缺少其中任一标记的 `503` 视为任何其他 `5xx`。

**修复原因，然后再次发送：**

* `unauthorized_client`：`error_description` 通常会指明原因，例如缺少默认工作区或您的应用已不再启用 `offline_access`。
* `invalid_client`：您的客户端断言失败。请对照[客户端断言声明](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#client-assertion-claims)进行检查。
* `invalid_scope`：请求发送了 `scope` 参数，或者您的应用已不再具有 `billing:attribution`。
* 任何其他 `4xx`，例如 `invalid_request`：更正请求。

如果您更改或归档了操作员服务账户，则对有效令牌的每次刷新都会收到 `unauthorized_client`，再次发送也无济于事。请参阅[如果您更改服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-change-the-service-account)。

**立即再发送一次：**`auth.claude.com` 必须在替换令牌后的 30 秒内收到第二个请求。您无法得知替换发生的时间，因此请从您发送第一个请求时开始计时，并为第二个请求的到达留出时间。然后：

* 如果第二个请求返回了新令牌，请存储它们并只使用它们。
* 如果它返回其他任何结果、没有得到响应或无法及时发出，请不要再次发送该令牌。

**不要再次发送：**&#x5728;读取超时或丢失了无法恢复的响应后，请停止刷新该客户，并[再次关联该客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)。

**停止发送：**&#x505C;止刷新该客户。`invalid_grant` 通常意味着以下情况之一，而响应不会说明是哪一种，因此在重新关联任何人之前，请先检查前两种情况：

* 客户的组织已被暂停。不要取消关联该客户，并保留刷新令牌：暂停结束后它会重新生效。要了解情况，请咨询 [Anthropic 支持](https://support.claude.com/en/articles/9015913-how-to-get-support)或您的 Anthropic 客户团队（如果有）。
* 所有客户的刷新同时收到了 `invalid_grant`。请在您这一侧查找原因，例如[已归档的工作区](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#if-you-move-to-another-workspace)。
* 令牌已过期或已被使用、连接已终止，或其工作区已被归档。[再次关联该客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)。

网络连接丢失、`5xx` 响应或超时可能意味着 `auth.claude.com` 出现了故障。如果其中某种情况导致您停止刷新某个客户，请防止该故障让您损失更多连接：

1. 暂停其他客户的刷新。
2. 刷新一个客户。
3. 只有在该刷新成功或收到 `4xx` 响应后，才恢复其余客户的刷新。

### 完整刷新示例

此版本的刷新遵循[如果刷新失败](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#if-a-refresh-fails)。它在普通调用的基础上增加了以下保护措施：

* 当访问令牌剩余时间少于 10 分钟时进行刷新。
* 从读取存储的令牌开始为每个客户持有锁，直到新令牌存储完毕。
* 在刷新令牌离开进程之前，将存储的令牌标记为 `on_hold` (typescript, java, php: `onHold`; csharp, go: `OnHold`)，这样崩溃不会导致第二次发送。
* 最多等待 2 秒连接 `auth.claude.com`，最多等待 10 秒获取响应。
* 仅在第一个请求发出后 4 秒内发送第二个请求。这为第二个请求在 30 秒内到达留出了时间。同时也使第二个请求处于 10 秒的等待时间内，从而超时的请求永远不会被再次发送。
* 如果只是写入密钥存储失败，它会重试写入，而不会再次刷新。

该示例将以下三件事留给您处理：

* 所有副本共享的锁，因为示例自身的锁无法覆盖您的其他副本
* 在收到 `429` 或 `503` 拒绝后等待并再次发送，示例会将其作为错误报告
* 在丢失刷新后，在每个副本上暂停其他客户的刷新

请以高于每 10 分钟一次的频率为每个客户调用刷新函数，以确保它始终在该余量内运行。

当刷新令牌不得再次发送时，`refresh_connection()` (typescript, go, java, php: `refreshConnection()`; csharp: `Connections.RefreshConnection()`) 会以 `RefreshOnHoldError` (csharp, java, php: `RefreshOnHoldException`) 失败。之后针对该客户的每次调用也都会失败。存储的记录会保留 `on_hold` (typescript, java, php: `onHold`; csharp, go: `OnHold`) 和 `hold_reason` (typescript, java, php: `holdReason`; csharp, go: `HoldReason`)，以便您日后了解设置搁置的原因。当只是写入失败时，错误的 `new_tokens` (typescript, java, php: `newTokens`; csharp, go: `NewTokens`) 包含需要存储的令牌。

以下设置定义了共享名称。您需要为自己的密钥存储编写 `load_tokens()` (typescript, go, java, php: `loadTokens()`; csharp: `LoadTokens()`) 和 `save_tokens()` (typescript, go, java, php: `saveTokens()`; csharp: `SaveTokens()`)。

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此设置与后面的刷新函数配套使用，该函数需要的逻辑超出单个 shell 命令所能承载的范围。请使用您应用程序的语言编写两者。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此设置与后面的刷新函数配套使用，该函数需要的逻辑超出单个 shell 命令所能承载的范围。请使用您应用程序的语言编写两者。
  </CodeGroupItem>

  ```python Python
  import os
  import threading
  import time
  from collections import defaultdict
  from dataclasses import dataclass, field, replace
  from pathlib import Path

  import httpx2  # comes with the Anthropic SDK

  TOKEN_URL = "https://auth.claude.com/v1/oauth/token"
  CLIENT_ID = "clid_01ABcDeFgHiJkLmNoPqRsTuV"
  JWT_BEARER = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer"
  REFRESH_MARGIN_SECONDS = 600
  REQUEST_TIMEOUT = httpx2.Timeout(10, connect=2)
  SECOND_REQUEST_DEADLINE_SECONDS = 4  # keep this under the 10-second read timeout


  @dataclass
  class ConnectionTokens:
      refresh_token: str = field(repr=False)
      access_token: str = field(repr=False)
      expires_at: int  # Unix seconds
      refresh_expires_at: int  # Unix seconds
      on_hold: bool = False  # True: the refresh token might have been used
      hold_reason: str | None = None  # "lost" or "invalid_grant"


  class RefreshOnHoldError(Exception):
      """The refresh token must not be sent again. Don't catch this to try again.

      If new_tokens is set, only the write failed: store those tokens.
      """

      new_tokens: ConnectionTokens | None = None


  def client_assertion() -> str:
      """Reads the JWT that authenticates your app to auth.claude.com."""
      return Path(os.environ["CLIENT_ASSERTION_FILE"]).read_text().strip()


  def load_tokens(connection_id: str) -> ConnectionTokens:
      """Reads the connection's newest tokens from your secret store."""
      raise NotImplementedError("Read the tokens from your secret store.")


  def save_tokens(connection_id: str, tokens: ConnectionTokens) -> None:
      """Writes the connection's newest tokens to your secret store."""
      raise NotImplementedError("Write the tokens to your secret store.")
  ```

  ```typescript TypeScript
  import { readFile } from "node:fs/promises";
  import type { IncomingHttpHeaders } from "node:http";
  import { request } from "node:https";

  const TOKEN_URL = "https://auth.claude.com/v1/oauth/token";
  const CLIENT_ID = "clid_01ABcDeFgHiJkLmNoPqRsTuV";
  const JWT_BEARER = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer";
  const REFRESH_MARGIN_SECONDS = 600;
  const SECOND_REQUEST_DEADLINE_MS = 4_000; // keep this under the 10-second timeout

  interface ConnectionTokens {
    refreshToken: string;
    accessToken: string;
    expiresAt: number; // Unix seconds
    refreshExpiresAt: number; // Unix seconds
    onHold?: boolean; // true: the refresh token might have been used
    holdReason?: "lost" | "invalid_grant";
  }

  /** The refresh token must not be sent again. Don't catch this to try again. */
  class RefreshOnHoldError extends Error {
    override name = "RefreshOnHoldError";
    readonly #newTokens?: ConnectionTokens; // private, so that printing the error doesn't show them

    constructor(message: string, newTokens?: ConnectionTokens) {
      super(message);
      this.#newTokens = newTokens;
    }

    /** If set, only the write failed: store these tokens. */
    get newTokens() {
      return this.#newTokens;
    }
  }

  /** Reads the JWT that authenticates your app to auth.claude.com. */
  async function clientAssertion(): Promise<string> {
    return (await readFile(process.env.CLIENT_ASSERTION_FILE!, "utf8")).trim();
  }

  /** Reads the connection's newest tokens from your secret store. */
  async function loadTokens(connectionId: string): Promise<ConnectionTokens> {
    throw new Error("Read the tokens from your secret store.");
  }

  /** Writes the connection's newest tokens to your secret store. */
  async function saveTokens(connectionId: string, tokens: ConnectionTokens): Promise<void> {
    throw new Error("Write the tokens to your secret store.");
  }
  ```

  ```csharp C#
  using System.Collections.Concurrent;
  using System.Diagnostics;
  using System.Text;
  using System.Text.Json;

  record ConnectionTokens(
      string RefreshToken,
      string AccessToken,
      long ExpiresAt, // Unix seconds
      long RefreshExpiresAt, // Unix seconds
      bool OnHold = false, // true: the refresh token might have been used
      string? HoldReason = null // "lost" or "invalid_grant"
  )
  {
      // 避免令牌出现在日志中。
      public override string ToString() =>
          $"ConnectionTokens {{ ExpiresAt = {ExpiresAt}, OnHold = {OnHold}, HoldReason = {HoldReason} }}";
  }

  /// <summary>不得再次发送该刷新令牌。请勿捕获此异常以重试。</summary>
  class RefreshOnHoldException(string message) : Exception(message)
  {
      /// <summary>如已设置，说明仅写入失败：请存储这些令牌。</summary>
      public ConnectionTokens? NewTokens { get; init; }
  }

  static partial class Connections
  {
      const string TokenUrl = "https://auth.claude.com/v1/oauth/token";
      const string ClientId = "clid_01ABcDeFgHiJkLmNoPqRsTuV";
      const string JwtBearer = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer";
      const int RefreshMarginSeconds = 600;
      static readonly TimeSpan SecondRequestDeadline = TimeSpan.FromSeconds(4); // keep this under the 10-second timeout

      /// <summary>读取用于向 auth.claude.com 验证您应用的 JWT。</summary>
      static string ClientAssertion() =>
          File.ReadAllText(Environment.GetEnvironmentVariable("CLIENT_ASSERTION_FILE")!).Trim();

      /// <summary>从您的密钥存储中读取该连接的最新令牌。</summary>
      static Task<ConnectionTokens> LoadTokens(string connectionId) =>
          throw new NotImplementedException("Read the tokens from your secret store.");

      /// <summary>将该连接的最新令牌写入您的密钥存储。</summary>
      static Task SaveTokens(string connectionId, ConnectionTokens tokens) =>
          throw new NotImplementedException("Write the tokens to your secret store.");
  }
  ```

  ```go Go
  import (
  	"bytes"
  	"encoding/json"
  	"errors"
  	"fmt"
  	"io"
  	"net"
  	"net/http"
  	"net/http/httptrace"
  	"os"
  	"strings"
  	"sync"
  	"time"
  )

  const (
  	tokenURL              = "https://auth.claude.com/v1/oauth/token"
  	clientID              = "clid_01ABcDeFgHiJkLmNoPqRsTuV"
  	jwtBearer             = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer"
  	refreshMargin         = 10 * time.Minute
  	secondRequestDeadline = 4 * time.Second // keep this under the 10-second timeout
  )

  var httpClient = &http.Client{
  	Timeout: 10 * time.Second,
  	Transport: &http.Transport{
  		Proxy:               http.ProxyFromEnvironment,
  		DialContext:         (&net.Dialer{Timeout: 2 * time.Second}).DialContext,
  		TLSHandshakeTimeout: 2 * time.Second,
  	},
  	CheckRedirect: func(*http.Request, []*http.Request) error { return http.ErrUseLastResponse },
  }

  type ConnectionTokens struct {
  	RefreshToken     string
  	AccessToken      string
  	ExpiresAt        int64  // Unix seconds
  	RefreshExpiresAt int64  // Unix seconds
  	OnHold           bool   // true: the refresh token might have been used
  	HoldReason       string // "lost" or "invalid_grant"
  }

  // String 可防止令牌出现在日志中。
  func (t ConnectionTokens) String() string {
  	return fmt.Sprintf("ConnectionTokens{ExpiresAt: %d, OnHold: %t, HoldReason: %q}", t.ExpiresAt, t.OnHold, t.HoldReason)
  }

  // RefreshOnHoldError 表示不得再次发送该刷新令牌。请勿重试。
  type RefreshOnHoldError struct {
  	Message   string
  	NewTokens *ConnectionTokens // if set, only the write failed: store these tokens
  }

  func (e *RefreshOnHoldError) Error() string { return e.Message }

  // clientAssertion 读取用于向 auth.claude.com 验证您应用的 JWT。
  func clientAssertion() (string, error) {
  	jwt, err := os.ReadFile(os.Getenv("CLIENT_ASSERTION_FILE"))
  	return strings.TrimSpace(string(jwt)), err
  }

  // loadTokens 从您的密钥存储中读取该连接的最新令牌。
  func loadTokens(connectionID string) (ConnectionTokens, error) {
  	return ConnectionTokens{}, errors.New("read the tokens from your secret store")
  }

  // saveTokens 将该连接的最新令牌写入您的密钥存储。
  func saveTokens(connectionID string, tokens ConnectionTokens) error {
  	return errors.New("write the tokens to your secret store")
  }
  ```

  ```java Java
  import module java.base;
  import module java.net.http;
  import com.fasterxml.jackson.databind.JsonNode;
  import com.fasterxml.jackson.databind.ObjectMapper; // Jackson comes with the Anthropic SDK

  static final URI TOKEN_URL = URI.create("https://auth.claude.com/v1/oauth/token");
  static final String CLIENT_ID = "clid_01ABcDeFgHiJkLmNoPqRsTuV";
  static final String JWT_BEARER = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer";
  static final long REFRESH_MARGIN_SECONDS = 600;
  static final Duration SECOND_REQUEST_DEADLINE = Duration.ofSeconds(4); // keep this under the 10-second read timeout

  record ConnectionTokens(
      String refreshToken,
      String accessToken,
      long expiresAt, // Unix seconds
      long refreshExpiresAt, // Unix seconds
      boolean onHold, // true: the refresh token might have been used
      String holdReason // "lost" or "invalid_grant"
  ) {
      @Override // keeps the tokens out of logs
      public String toString() {
          return "ConnectionTokens[expiresAt=" + expiresAt + ", onHold=" + onHold + ", holdReason=" + holdReason + "]";
      }

      ConnectionTokens withHold(String reason) {
          return new ConnectionTokens(refreshToken, accessToken, expiresAt, refreshExpiresAt, true, reason);
      }
  }

  record AccessToken(String value, long expiresAt /* Unix seconds */) {
      @Override // keeps the token out of logs
      public String toString() {
          return "AccessToken[expiresAt=" + expiresAt + "]";
      }
  }

  /** The refresh token must not be sent again. Don't catch this to try again. */
  static class RefreshOnHoldException extends RuntimeException {
      /** If set, only the write failed: store these tokens. */
      ConnectionTokens newTokens;

      RefreshOnHoldException(String message) {
          super(message);
      }
  }

  /** Reads the JWT that authenticates your app to auth.claude.com. */
  String clientAssertion() throws IOException {
      return Files.readString(Path.of(System.getenv("CLIENT_ASSERTION_FILE"))).strip();
  }

  /** Reads the connection's newest tokens from your secret store. */
  ConnectionTokens loadTokens(String connectionId) throws IOException {
      throw new UnsupportedOperationException("Read the tokens from your secret store.");
  }

  /** Writes the connection's newest tokens to your secret store. */
  void saveTokens(String connectionId, ConnectionTokens tokens) throws IOException {
      throw new UnsupportedOperationException("Write the tokens to your secret store.");
  }
  ```

  ```php PHP
  const TOKEN_URL = 'https://auth.claude.com/v1/oauth/token';
  const CLIENT_ID = 'clid_01ABcDeFgHiJkLmNoPqRsTuV';
  const JWT_BEARER = 'urn:ietf:params:oauth:client-assertion-type:jwt-bearer';
  const REFRESH_MARGIN_SECONDS = 600;
  const SECOND_REQUEST_DEADLINE_SECONDS = 4; // keep this under the 10-second timeout

  final class ConnectionTokens
  {
      public function __construct(
          public readonly string $refreshToken,
          public readonly string $accessToken,
          public readonly int $expiresAt, // Unix seconds
          public readonly int $refreshExpiresAt, // Unix seconds
          public readonly bool $onHold = false, // true: the refresh token might have been used
          public readonly ?string $holdReason = null, // 'lost' or 'invalid_grant'
      ) {}

      /** Keeps the tokens out of var_dump and print_r. */
      public function __debugInfo(): array
      {
          return ['expiresAt' => $this->expiresAt, 'onHold' => $this->onHold, 'holdReason' => $this->holdReason];
      }

      public function withHold(string $reason): self
      {
          return new self(
              refreshToken: $this->refreshToken,
              accessToken: $this->accessToken,
              expiresAt: $this->expiresAt,
              refreshExpiresAt: $this->refreshExpiresAt,
              onHold: true,
              holdReason: $reason,
          );
      }
  }

  /** The refresh token must not be sent again. Don't catch this to try again. */
  final class RefreshOnHoldException extends \RuntimeException
  {
      /** If set, only the write failed: store these tokens. */
      public ?ConnectionTokens $newTokens = null;
  }

  /** Reads the JWT that authenticates your app to auth.claude.com. */
  function clientAssertion(): string
  {
      $jwt = file_get_contents((string) getenv('CLIENT_ASSERTION_FILE'))
          ?: throw new \RuntimeException('Cannot read the client assertion file');
      return trim($jwt);
  }

  /** Reads the connection's newest tokens from your secret store. */
  function loadTokens(string $connectionId): ConnectionTokens
  {
      throw new \LogicException('Read the tokens from your secret store.');
  }

  /** Writes the connection's newest tokens to your secret store. */
  function saveTokens(string $connectionId, ConnectionTokens $tokens): void
  {
      throw new \LogicException('Write the tokens to your secret store.');
  }
  ```

  ```ruby Ruby
  require "json"
  require "net/http"

  TOKEN_URL = URI("https://auth.claude.com/v1/oauth/token")
  CLIENT_ID = "clid_01ABcDeFgHiJkLmNoPqRsTuV"
  JWT_BEARER = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer"
  REFRESH_MARGIN_SECONDS = 600
  SECOND_REQUEST_DEADLINE_SECONDS = 4 # keep this under the 10-second read timeout

  ConnectionTokens = Data.define(
    :refresh_token,
    :access_token,
    :expires_at, # Unix seconds
    :refresh_expires_at, # Unix seconds
    :on_hold, # true: the refresh token might have been used
    :hold_reason # "lost" or "invalid_grant"
  ) do
    def initialize(on_hold: false, hold_reason: nil, **) = super

    # 防止令牌出现在日志中。
    def inspect = "#<data ConnectionTokens expires_at=#{expires_at} on_hold=#{on_hold} hold_reason=#{hold_reason.inspect}>"
    alias_method :to_s, :inspect
  end

  # 刷新令牌不得再次发送。不要捕获此异常来重试。
  # 如果设置了 new_tokens，则仅写入失败：请存储这些令牌。
  class RefreshOnHoldError < StandardError
    attr_accessor :new_tokens
  end

  # 读取用于向 auth.claude.com 验证您应用的 JWT。
  def client_assertion = File.read(ENV.fetch("CLIENT_ASSERTION_FILE")).strip

  # 从您的密钥存储中读取该连接的最新令牌。
  def load_tokens(connection_id)
    raise NotImplementedError, "Read the tokens from your secret store."
  end

  # 将该连接的最新令牌写入您的密钥存储。
  def save_tokens(connection_id, tokens)
    raise NotImplementedError, "Write the tokens to your secret store."
  end
  ```
</CodeGroup>

以下刷新函数使用上述设置：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此刷新需要的逻辑超出单个 shell 命令所能承载的范围。请使用您应用程序的语言编写。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此刷新需要的逻辑超出单个 shell 命令所能承载的范围。请使用您应用程序的语言编写。
  </CodeGroupItem>

  ```python Python
  connection_locks: defaultdict[str, threading.Lock] = defaultdict(threading.Lock)


  def error_body(response: httpx2.Response) -> dict:
      try:
          body = response.json()
      except ValueError:
          return {}
      return body if isinstance(body, dict) else {}


  def is_lost(response: httpx2.Response) -> bool:
      """True for a 5xx response that can follow a refresh that took effect."""
      if response.status_code < 500:
          return False
      refused = (
          response.status_code == 503
          and error_body(response).get("type") == "error"
          and "retry-after" in response.headers
      )
      return not refused


  def hold_error(connection_id: str, reason: str) -> RefreshOnHoldError:
      return RefreshOnHoldError(
          f"Connection {connection_id} is on hold. {reason} Don't send its refresh token"
          " again: that can end the customer's link. See"
          ' "If a refresh fails" in the Linked billing docs.'
      )


  def refresh_connection(connection_id: str, *, force: bool = False) -> tuple[str, int]:
      """Returns a current access token and its expiry time in Unix seconds.

      Call this from your schedule. Pass force=True to refresh a token that isn't
      close to its expiry. Raises RefreshOnHoldError when the refresh token must not be
      sent again, and httpx2.HTTPStatusError for a refusal that keeps it.
      """
      # 此锁仅在单个进程内有效。
      with connection_locks[connection_id]:
          tokens = load_tokens(connection_id)
          if tokens.on_hold:
              raise hold_error(connection_id, f"Reason: {tokens.hold_reason}.")
          if not force and tokens.expires_at - time.time() > REFRESH_MARGIN_SECONDS:
              return tokens.access_token, tokens.expires_at
          request = {
              "grant_type": "refresh_token",
              "refresh_token": tokens.refresh_token,
              "client_id": CLIENT_ID,
              "client_assertion_type": JWT_BEARER,
              "client_assertion": client_assertion(),
          }
          # 先设置搁置标记，这样即使进程在刷新中途停止，搁置标记也会保留。
          held = replace(tokens, on_hold=True, hold_reason="lost")
          save_tokens(connection_id, held)
          issued_at = int(time.time())
          started = time.monotonic()
          # 不要添加重试：延迟的重试会终止客户的关联。
          try:
              response = httpx2.post(TOKEN_URL, json=request, timeout=REQUEST_TIMEOUT)
              lost = is_lost(response)
          except (httpx2.ConnectError, httpx2.ConnectTimeout):
              # 刷新令牌从未离开此进程，因此解除搁置标记。
              save_tokens(connection_id, tokens)
              raise
          except httpx2.TransportError:
              lost = True
          if lost:
              if time.monotonic() - started > SECOND_REQUEST_DEADLINE_SECONDS:
                  raise hold_error(connection_id, "A refresh got no usable answer.")
              try:
                  response = httpx2.post(TOKEN_URL, json=request, timeout=REQUEST_TIMEOUT)
              except httpx2.TransportError:
                  raise hold_error(
                      connection_id, "Two refreshes got no answer."
                  ) from None
              if response.status_code != 200:
                  raise hold_error(connection_id, "The second refresh failed.")
          if error_body(response).get("error") == "invalid_grant":
              save_tokens(connection_id, replace(held, hold_reason="invalid_grant"))
              raise hold_error(connection_id, "The refresh returned invalid_grant.")
          if response.status_code != 200:
              # 这是一次保留刷新令牌的拒绝，因此解除搁置标记。
              save_tokens(connection_id, tokens)
              response.raise_for_status()
          body = response.json()
          tokens = ConnectionTokens(
              refresh_token=body["refresh_token"],
              access_token=body["access_token"],
              expires_at=issued_at + body["expires_in"],
              refresh_expires_at=issued_at + body["refresh_token_expires_in"],
          )
          # 仅重试写入操作，不要再次刷新。
          for attempt in range(5):
              try:
                  save_tokens(connection_id, tokens)
                  return tokens.access_token, tokens.expires_at
              except OSError:  # your secret store's error type
                  if attempt < 4:
                      time.sleep(2**attempt)
          error = hold_error(
              connection_id,
              "The refresh succeeded, but the new tokens could not be saved.",
          )
          error.new_tokens = tokens
          raise error
  ```

  ```typescript TypeScript
  const connectionLocks = new Map<string, Promise<unknown>>();

  function withConnectionLock<T>(connectionId: string, work: () => Promise<T>): Promise<T> {
    const result = (connectionLocks.get(connectionId) ?? Promise.resolve()).then(work, work);
    connectionLocks.set(
      connectionId,
      result.catch(() => {})
    );
    return result;
  }

  class NotSent extends Error {}

  interface TokenResponse {
    status: number;
    headers: IncomingHttpHeaders;
    text: string;
  }

  function post(body: string): Promise<TokenResponse> {
    return new Promise((resolve, reject) => {
      let connected = false;
      const req = request(
        TOKEN_URL,
        {
          method: "POST",
          agent: false, // a new connection for each request
          headers: { "content-type": "application/json" },
          signal: AbortSignal.timeout(10_000)
        },
        (response) => {
          let text = "";
          response.setEncoding("utf8");
          response.on("data", (chunk: string) => (text += chunk));
          response.on("end", () =>
            resolve({ status: response.statusCode!, headers: response.headers, text })
          );
          response.on("error", reject);
        }
      );
      const connectTimer = setTimeout(
        () => req.destroy(new Error("No connection in 2 seconds")),
        2_000
      );
      req.on("socket", (socket) =>
        socket.once("secureConnect", () => {
          connected = true;
          clearTimeout(connectTimer);
        })
      );
      req.on("error", (error) => {
        clearTimeout(connectTimer);
        reject(connected ? error : new NotSent(error.message, { cause: error }));
      });
      req.end(body);
    });
  }

  function errorBody(response: TokenResponse): Record<string, unknown> {
    try {
      const body: unknown = JSON.parse(response.text);
      return typeof body === "object" && body !== null ? (body as Record<string, unknown>) : {};
    } catch {
      return {};
    }
  }

  /** True for a 5xx response that can follow a refresh that took effect. */
  function isLost(response: TokenResponse): boolean {
    if (response.status < 500) {
      return false;
    }
    const refused =
      response.status === 503 &&
      errorBody(response).type === "error" &&
      "retry-after" in response.headers;
    return !refused;
  }

  function holdError(
    connectionId: string,
    reason: string,
    newTokens?: ConnectionTokens
  ): RefreshOnHoldError {
    return new RefreshOnHoldError(
      `Connection ${connectionId} is on hold. ${reason} Don't send its refresh token again: ` +
        'that can end the customer\'s link. See "If a refresh fails" in the Linked billing docs.',
      newTokens
    );
  }

  /**
   * Returns a current access token and its expiry time in Unix seconds.
   *
   * Call this from your schedule. Pass force: true to refresh a token that isn't
   * close to its expiry. Throws RefreshOnHoldError when the refresh token must not be
   * sent again, and an Error for a refusal that keeps it.
   */
  function refreshConnection(connectionId: string, { force = false } = {}) {
    // 此锁仅在单个进程内有效。
    return withConnectionLock(connectionId, async () => {
      const tokens = await loadTokens(connectionId);
      if (tokens.onHold) {
        throw holdError(connectionId, `Reason: ${tokens.holdReason}.`);
      }
      if (!force && tokens.expiresAt - Date.now() / 1000 > REFRESH_MARGIN_SECONDS) {
        return { accessToken: tokens.accessToken, expiresAt: tokens.expiresAt };
      }
      const body = JSON.stringify({
        grant_type: "refresh_token",
        refresh_token: tokens.refreshToken,
        client_id: CLIENT_ID,
        client_assertion_type: JWT_BEARER,
        client_assertion: await clientAssertion()
      });
      // 先设置搁置标记，这样即使进程在刷新中途停止，搁置标记也会保留。
      const held: ConnectionTokens = { ...tokens, onHold: true, holdReason: "lost" };
      await saveTokens(connectionId, held);
      const issuedAt = Math.floor(Date.now() / 1000);
      const started = performance.now();
      // 不要添加重试：延迟的重试会终止客户的关联。
      let response: TokenResponse | undefined;
      try {
        response = await post(body);
      } catch (error) {
        if (error instanceof NotSent) {
          // 刷新令牌从未离开此进程，因此解除搁置标记。
          await saveTokens(connectionId, tokens);
          throw error;
        }
      }
      if (!response || isLost(response)) {
        if (performance.now() - started > SECOND_REQUEST_DEADLINE_MS) {
          throw holdError(connectionId, "A refresh got no usable answer.");
        }
        try {
          response = await post(body);
        } catch {
          throw holdError(connectionId, "Two refreshes got no answer.");
        }
        if (response.status !== 200) {
          throw holdError(connectionId, "The second refresh failed.");
        }
      }
      if (errorBody(response).error === "invalid_grant") {
        await saveTokens(connectionId, { ...held, holdReason: "invalid_grant" });
        throw holdError(connectionId, "The refresh returned invalid_grant.");
      }
      if (response.status !== 200) {
        // 这是一次保留刷新令牌的拒绝，因此解除搁置标记。
        await saveTokens(connectionId, tokens);
        throw new Error(`Refresh failed: ${response.status} ${response.text}`);
      }
      const answer = JSON.parse(response.text);
      const newTokens: ConnectionTokens = {
        refreshToken: answer.refresh_token,
        accessToken: answer.access_token,
        expiresAt: issuedAt + answer.expires_in,
        refreshExpiresAt: issuedAt + answer.refresh_token_expires_in
      };
      // 仅重试写入，不要再次刷新。
      for (let attempt = 0; attempt < 5; attempt++) {
        try {
          await saveTokens(connectionId, newTokens);
          return { accessToken: newTokens.accessToken, expiresAt: newTokens.expiresAt };
        } catch {
          // 写入失败
          if (attempt < 4) {
            await new Promise((wake) => setTimeout(wake, 2 ** attempt * 1000));
          }
        }
      }
      throw holdError(
        connectionId,
        "The refresh succeeded, but the new tokens could not be saved.",
        newTokens
      );
    });
  }
  ```

  ```csharp C#
  static partial class Connections
  {
      static readonly ConcurrentDictionary<string, SemaphoreSlim> ConnectionLocks = new();

      static readonly HttpClient Http = new(new SocketsHttpHandler
      {
          ConnectTimeout = TimeSpan.FromSeconds(2),
          AllowAutoRedirect = false,
          PooledConnectionLifetime = TimeSpan.Zero, // a new connection for each request
      })
      { Timeout = Timeout.InfiniteTimeSpan }; // Post sets the 10-second timeout

      static async Task<(HttpResponseMessage? Response, string Body, Exception? Error, bool Connected)> Post(
          string body)
      {
          using var request = new HttpRequestMessage(HttpMethod.Post, TokenUrl)
          {
              Content = new StringContent(body, Encoding.UTF8, "application/json"),
          };
          using var timeout = new CancellationTokenSource(TimeSpan.FromSeconds(10));
          try
          {
              var response = await Http.SendAsync(request, timeout.Token);
              return (response, await response.Content.ReadAsStringAsync(timeout.Token), null, true);
          }
          catch (HttpRequestException e) when (e.HttpRequestError is HttpRequestError.ConnectionError
              or HttpRequestError.NameResolutionError or HttpRequestError.SecureConnectionError)
          {
              return (null, "", e, false);
          }
          catch (TaskCanceledException e) when (!timeout.IsCancellationRequested)
          {
              return (null, "", e, false); // the 2-second connect timeout
          }
          catch (Exception e) when (e is HttpRequestException or TaskCanceledException)
          {
              return (null, "", e, true); // a dropped connection or a timeout
          }
      }

      static JsonElement ErrorBody(string body)
      {
          try
          {
              using var document = JsonDocument.Parse(body);
              var parsed = document.RootElement;
              return parsed.ValueKind == JsonValueKind.Object ? parsed.Clone() : default;
          }
          catch (JsonException)
          {
              return default;
          }
      }

      static string? ErrorField(string body, string name) =>
          ErrorBody(body) is { ValueKind: JsonValueKind.Object } error && error.TryGetProperty(name, out var value)
              ? value.ToString()
              : null;

      /// <summary>对于可能在刷新已生效后出现的 5xx 响应，返回 true。</summary>
      static bool IsLost(HttpResponseMessage response, string body)
      {
          if ((int)response.StatusCode < 500) return false;
          var refused = (int)response.StatusCode == 503
              && ErrorField(body, "type") == "error"
              && response.Headers.Contains("retry-after");
          return !refused;
      }

      static RefreshOnHoldException HoldError(string connectionId, string reason) => new(
          $"Connection {connectionId} is on hold. {reason} Don't send its refresh token again: " +
          "that can end the customer's link. See \"If a refresh fails\" in the Linked billing docs.");

      /// <summary>
      /// 返回当前访问令牌及其过期时间（Unix 秒）。
      /// 请从您的定时任务中调用此方法。传入 force: true 可刷新尚未
      /// 临近过期的令牌。当不得再次发送刷新令牌时抛出 RefreshOnHoldException，
      /// 当请求被拒绝但刷新令牌仍有效时抛出 HttpRequestException。
      /// </summary>
      public static async Task<(string AccessToken, long ExpiresAt)> RefreshConnection(
          string connectionId, bool force = false)
      {
          // 此锁仅在单个进程内有效。
          var connectionLock = ConnectionLocks.GetOrAdd(connectionId, _ => new SemaphoreSlim(1, 1));
          await connectionLock.WaitAsync();
          try
          {
              var tokens = await LoadTokens(connectionId);
              if (tokens.OnHold) throw HoldError(connectionId, $"Reason: {tokens.HoldReason}.");
              if (!force && tokens.ExpiresAt - DateTimeOffset.UtcNow.ToUnixTimeSeconds() > RefreshMarginSeconds)
              {
                  return (tokens.AccessToken, tokens.ExpiresAt);
              }
              var request = JsonSerializer.Serialize(new Dictionary<string, string>
              {
                  ["grant_type"] = "refresh_token",
                  ["refresh_token"] = tokens.RefreshToken,
                  ["client_id"] = ClientId,
                  ["client_assertion_type"] = JwtBearer,
                  ["client_assertion"] = ClientAssertion(),
              });
              // 先设置搁置标记，这样即使进程在刷新中途停止，搁置标记也会保留。
              var held = tokens with { OnHold = true, HoldReason = "lost" };
              await SaveTokens(connectionId, held);
              var issuedAt = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
              var started = Stopwatch.StartNew();
              // 请勿添加重试：延迟的重试会断开客户的关联。
              var (response, body, error, connected) = await Post(request);
              if (!connected)
              {
                  // 刷新令牌从未离开此进程，因此解除搁置标记。
                  await SaveTokens(connectionId, tokens);
                  throw new HttpRequestException("Couldn't connect to auth.claude.com.", error);
              }
              if (response is null || IsLost(response, body))
              {
                  if (started.Elapsed > SecondRequestDeadline)
                      throw HoldError(connectionId, "A refresh got no usable answer.");
                  (response, body, _, _) = await Post(request);
                  if (response is null) throw HoldError(connectionId, "Two refreshes got no answer.");
                  if ((int)response.StatusCode != 200) throw HoldError(connectionId, "The second refresh failed.");
              }
              if (ErrorField(body, "error") == "invalid_grant")
              {
                  await SaveTokens(connectionId, held with { HoldReason = "invalid_grant" });
                  throw HoldError(connectionId, "The refresh returned invalid_grant.");
              }
              if ((int)response.StatusCode != 200)
              {
                  // 此拒绝不会使刷新令牌失效，因此解除搁置标记。
                  await SaveTokens(connectionId, tokens);
                  throw new HttpRequestException($"Refresh failed: {(int)response.StatusCode} {body}",
                      null, response.StatusCode);
              }
              using var document = JsonDocument.Parse(body);
              var answer = document.RootElement;
              var newTokens = new ConnectionTokens(
                  RefreshToken: answer.GetProperty("refresh_token").GetString()!,
                  AccessToken: answer.GetProperty("access_token").GetString()!,
                  ExpiresAt: issuedAt + answer.GetProperty("expires_in").GetInt64(),
                  RefreshExpiresAt: issuedAt + answer.GetProperty("refresh_token_expires_in").GetInt64());
              // 仅重试写入操作，不要再次刷新。
              for (var attempt = 0; attempt < 5; attempt++)
              {
                  try
                  {
                      await SaveTokens(connectionId, newTokens);
                      return (newTokens.AccessToken, newTokens.ExpiresAt);
                  }
                  catch (IOException) // your secret store's error type
                  {
                      if (attempt < 4) await Task.Delay(TimeSpan.FromSeconds(1 << attempt));
                  }
              }
              throw new RefreshOnHoldException(HoldError(connectionId,
                  "The refresh succeeded, but the new tokens could not be saved.").Message)
              { NewTokens = newTokens };
          }
          finally
          {
              connectionLock.Release();
          }
      }
  }
  ```

  ```go Go
  var connectionLocks sync.Map

  type tokenResponse struct {
  	status int
  	header http.Header
  	body   []byte
  }

  // post 发送刷新请求。若请求从未发出，connected 为 false。
  func post(body []byte) (response *tokenResponse, connected bool, err error) {
  	request, err := http.NewRequest(http.MethodPost, tokenURL, bytes.NewReader(body))
  	if err != nil {
  		return nil, false, err
  	}
  	request.Header.Set("content-type", "application/json")
  	trace := &httptrace.ClientTrace{GotConn: func(httptrace.GotConnInfo) { connected = true }}
  	request = request.WithContext(httptrace.WithClientTrace(request.Context(), trace))
  	answer, err := httpClient.Do(request)
  	if err != nil {
  		return nil, connected, err
  	}
  	defer answer.Body.Close()
  	data, err := io.ReadAll(answer.Body)
  	if err != nil {
  		return nil, true, err
  	}
  	return &tokenResponse{answer.StatusCode, answer.Header, data}, true, nil
  }

  func errorBody(response *tokenResponse) map[string]any {
  	var body map[string]any
  	if json.Unmarshal(response.body, &body) != nil {
  		return map[string]any{}
  	}
  	return body
  }

  // 对于可能出现在已生效刷新之后的 5xx 响应，isLost 为 true。
  func isLost(response *tokenResponse) bool {
  	if response.status < 500 {
  		return false
  	}
  	refused := response.status == 503 &&
  		errorBody(response)["type"] == "error" &&
  		response.header.Get("retry-after") != ""
  	return !refused
  }

  func holdError(connectionID, reason string) *RefreshOnHoldError {
  	return &RefreshOnHoldError{Message: fmt.Sprintf("connection %s is on hold. %s Don't send its refresh token "+
  		"again: that can end the customer's link. See \"If a refresh fails\" in the Linked billing docs.",
  		connectionID, reason)}
  }

  // refreshConnection 返回当前访问令牌及其过期时间（Unix
  // 秒）。请在您的计划任务中调用它。传入 force=true 可刷新尚未
  // 临近过期的令牌。当刷新令牌不得再次发送时，它返回 *RefreshOnHoldError，
  // 对于仍保留该令牌的拒绝，则返回其他错误。
  func refreshConnection(connectionID string, force bool) (string, int64, error) {
  	// 此锁仅在单个进程内有效。
  	lock, _ := connectionLocks.LoadOrStore(connectionID, &sync.Mutex{})
  	lock.(*sync.Mutex).Lock()
  	defer lock.(*sync.Mutex).Unlock()

  	tokens, err := loadTokens(connectionID)
  	if err != nil {
  		return "", 0, err
  	}
  	if tokens.OnHold {
  		return "", 0, holdError(connectionID, "Reason: "+tokens.HoldReason+".")
  	}
  	if !force && time.Until(time.Unix(tokens.ExpiresAt, 0)) > refreshMargin {
  		return tokens.AccessToken, tokens.ExpiresAt, nil
  	}
  	assertion, err := clientAssertion()
  	if err != nil {
  		return "", 0, err
  	}
  	body, err := json.Marshal(map[string]string{
  		"grant_type":            "refresh_token",
  		"refresh_token":         tokens.RefreshToken,
  		"client_id":             clientID,
  		"client_assertion_type": jwtBearer,
  		"client_assertion":      assertion,
  	})
  	if err != nil {
  		return "", 0, err
  	}
  	// 先设置搁置标记，这样即使进程在刷新中途停止，搁置标记也会保留。
  	held := tokens
  	held.OnHold, held.HoldReason = true, "lost"
  	if err := saveTokens(connectionID, held); err != nil {
  		return "", 0, err
  	}
  	issuedAt := time.Now().Unix()
  	started := time.Now()
  	// 不要添加重试：延迟的重试会断开客户的关联。
  	response, connected, err := post(body)
  	if err != nil && !connected {
  		// 刷新令牌从未离开此进程，因此解除搁置标记。
  		if saveErr := saveTokens(connectionID, tokens); saveErr != nil {
  			return "", 0, saveErr
  		}
  		return "", 0, err
  	}
  	if err != nil || isLost(response) {
  		if time.Since(started) > secondRequestDeadline {
  			return "", 0, holdError(connectionID, "A refresh got no usable answer.")
  		}
  		if response, _, err = post(body); err != nil {
  			return "", 0, holdError(connectionID, "Two refreshes got no answer.")
  		}
  		if response.status != 200 {
  			return "", 0, holdError(connectionID, "The second refresh failed.")
  		}
  	}
  	if errorBody(response)["error"] == "invalid_grant" {
  		held.HoldReason = "invalid_grant"
  		if err := saveTokens(connectionID, held); err != nil {
  			return "", 0, err
  		}
  		return "", 0, holdError(connectionID, "The refresh returned invalid_grant.")
  	}
  	if response.status != 200 {
  		// 这是一次保留刷新令牌的拒绝，因此解除搁置标记。
  		if err := saveTokens(connectionID, tokens); err != nil {
  			return "", 0, err
  		}
  		return "", 0, fmt.Errorf("refresh failed: %d %s", response.status, response.body)
  	}
  	var answer struct {
  		AccessToken           string `json:"access_token"`
  		RefreshToken          string `json:"refresh_token"`
  		ExpiresIn             int64  `json:"expires_in"`
  		RefreshTokenExpiresIn int64  `json:"refresh_token_expires_in"`
  	}
  	if err := json.Unmarshal(response.body, &answer); err != nil {
  		return "", 0, holdError(connectionID, "The refresh answer could not be read.")
  	}
  	tokens = ConnectionTokens{
  		RefreshToken:     answer.RefreshToken,
  		AccessToken:      answer.AccessToken,
  		ExpiresAt:        issuedAt + answer.ExpiresIn,
  		RefreshExpiresAt: issuedAt + answer.RefreshTokenExpiresIn,
  	}
  	// 仅重试写入操作。不要再次刷新。
  	for attempt := range 5 {
  		if err := saveTokens(connectionID, tokens); err == nil {
  			return tokens.AccessToken, tokens.ExpiresAt, nil
  		}
  		if attempt < 4 {
  			time.Sleep(time.Second << attempt)
  		}
  	}
  	hold := holdError(connectionID, "The refresh succeeded, but the new tokens could not be saved.")
  	hold.NewTokens = &tokens
  	return "", 0, hold
  }
  ```

  ```java Java
  final Map<String, ReentrantLock> connectionLocks = new ConcurrentHashMap<>();
  final ObjectMapper mapper = new ObjectMapper();
  final HttpClient http = HttpClient.newBuilder()
      .connectTimeout(Duration.ofSeconds(2))
      .followRedirects(HttpClient.Redirect.NEVER)
      .build();

  HttpResponse<String> post(String body) throws IOException, InterruptedException {
      var request = HttpRequest.newBuilder(TOKEN_URL)
          .header("content-type", "application/json")
          .timeout(Duration.ofSeconds(10))
          .POST(HttpRequest.BodyPublishers.ofString(body))
          .build();
      return http.send(request, HttpResponse.BodyHandlers.ofString());
  }

  JsonNode errorBody(HttpResponse<String> response) {
      try {
          return mapper.readTree(response.body());
      } catch (IOException _) {
          return mapper.createObjectNode();
      }
  }

  /** True for a 5xx response that can follow a refresh that took effect. */
  boolean isLost(HttpResponse<String> response) {
      if (response.statusCode() < 500) {
          return false;
      }
      boolean refused = response.statusCode() == 503
          && errorBody(response).path("type").asText().equals("error")
          && response.headers().firstValue("retry-after").isPresent();
      return !refused;
  }

  RefreshOnHoldException holdError(String connectionId, String reason) {
      return new RefreshOnHoldException("Connection " + connectionId + " is on hold. " + reason
          + " Don't send its refresh token again: that can end the customer's link."
          + " See \"If a refresh fails\" in the Linked billing docs.");
  }

  /**
   * Returns a current access token and its expiry time in Unix seconds.
   * Call this from your schedule. Pass force = true to refresh a token that isn't
   * close to its expiry. Throws RefreshOnHoldException when the refresh token must not be
   * sent again, and IOException for a refusal that keeps it.
   */
  AccessToken refreshConnection(String connectionId, boolean force)
          throws IOException, InterruptedException {
      // 此锁仅在单个进程内有效。
      var lock = connectionLocks.computeIfAbsent(connectionId, id -> new ReentrantLock());
      lock.lock();
      try {
          var tokens = loadTokens(connectionId);
          if (tokens.onHold()) {
              throw holdError(connectionId, "Reason: " + tokens.holdReason() + ".");
          }
          if (!force && tokens.expiresAt() - Instant.now().getEpochSecond() > REFRESH_MARGIN_SECONDS) {
              return new AccessToken(tokens.accessToken(), tokens.expiresAt());
          }
          var request = mapper.createObjectNode()
              .put("grant_type", "refresh_token")
              .put("refresh_token", tokens.refreshToken())
              .put("client_id", CLIENT_ID)
              .put("client_assertion_type", JWT_BEARER)
              .put("client_assertion", clientAssertion())
              .toString();
          // 先设置搁置标记，这样即使进程在刷新中途停止，搁置标记也会保留。
          var held = tokens.withHold("lost");
          saveTokens(connectionId, held);
          long issuedAt = Instant.now().getEpochSecond();
          long started = System.nanoTime();
          // 不要添加重试：迟到的重试会终止客户的关联。
          HttpResponse<String> response = null;
          try {
              response = post(request);
          } catch (ConnectException | HttpConnectTimeoutException | SSLHandshakeException e) {
              // 刷新令牌从未离开此进程，因此解除搁置标记。
              saveTokens(connectionId, tokens);
              throw e;
          } catch (IOException _) {
              // 连接已断开，或未能及时收到响应。
          }
          if (response == null || isLost(response)) {
              if (Duration.ofNanos(System.nanoTime() - started).compareTo(SECOND_REQUEST_DEADLINE) > 0) {
                  throw holdError(connectionId, "A refresh got no usable answer.");
              }
              try {
                  response = post(request);
              } catch (IOException _) {
                  throw holdError(connectionId, "Two refreshes got no answer.");
              }
              if (response.statusCode() != 200) {
                  throw holdError(connectionId, "The second refresh failed.");
              }
          }
          if (errorBody(response).path("error").asText().equals("invalid_grant")) {
              saveTokens(connectionId, held.withHold("invalid_grant"));
              throw holdError(connectionId, "The refresh returned invalid_grant.");
          }
          if (response.statusCode() != 200) {
              // 这是一次保留刷新令牌的拒绝，因此解除搁置标记。
              saveTokens(connectionId, tokens);
              throw new IOException("Refresh failed: " + response.statusCode() + " " + response.body());
          }
          var answer = mapper.readTree(response.body());
          var newTokens = new ConnectionTokens(
              answer.get("refresh_token").asText(),
              answer.get("access_token").asText(),
              issuedAt + answer.get("expires_in").asLong(),
              issuedAt + answer.get("refresh_token_expires_in").asLong(),
              false,
              null);
          // 仅重试写入操作，不要再次刷新。
          for (int attempt = 0; attempt < 5; attempt++) {
              try {
                  saveTokens(connectionId, newTokens);
                  return new AccessToken(newTokens.accessToken(), newTokens.expiresAt());
              } catch (IOException _) { // your secret store's error type
                  if (attempt < 4) {
                      Thread.sleep(Duration.ofSeconds(1L << attempt));
                  }
              }
          }
          var error = holdError(connectionId, "The refresh succeeded, but the new tokens could not be saved.");
          error.newTokens = newTokens;
          throw error;
      } finally {
          lock.unlock();
      }
  }
  ```

  ```php PHP
  /**
   * Sends the refresh. Returns a string, curl's error, when the request never left this process.
   *
   * @return array{status: int, headers: list<string>, body: string}|false|string false: no answer
   */
  function post(string $body): array|false|string
  {
      $headers = [];
      $curl = curl_init(TOKEN_URL);
      curl_setopt_array($curl, [
          CURLOPT_POST => true,
          CURLOPT_POSTFIELDS => $body,
          CURLOPT_HTTPHEADER => ['content-type: application/json'],
          CURLOPT_RETURNTRANSFER => true,
          CURLOPT_CONNECTTIMEOUT => 2,
          CURLOPT_TIMEOUT => 10,
          CURLOPT_FOLLOWLOCATION => false,
          CURLOPT_FORBID_REUSE => true, // a new connection for each request
          CURLOPT_HEADERFUNCTION => function ($curl, string $line) use (&$headers): int {
              $headers[] = strtolower(trim($line));
              return strlen($line);
          },
      ]);
      $text = curl_exec($curl);
      if (!is_string($text)) {
          // 尚未建立 TLS 连接，说明未发送任何内容。
          return curl_getinfo($curl, CURLINFO_APPCONNECT_TIME_T) > 0 ? false : curl_error($curl);
      }
      return ['status' => curl_getinfo($curl, CURLINFO_RESPONSE_CODE), 'headers' => $headers, 'body' => $text];
  }

  /**
   * @param array{status: int, headers: list<string>, body: string} $response
   * @return array<mixed>
   */
  function errorBody(array $response): array
  {
      $body = json_decode($response['body'], associative: true);
      return is_array($body) ? $body : [];
  }

  /**
   * True for a 5xx response that can follow a refresh that took effect.
   *
   * @param array{status: int, headers: list<string>, body: string} $response
   */
  function isLost(array $response): bool
  {
      if ($response['status'] < 500) {
          return false;
      }
      $refused = $response['status'] === 503
          && (errorBody($response)['type'] ?? null) === 'error'
          && preg_grep('/^retry-after:/', $response['headers']);
      return !$refused;
  }

  function holdError(string $connectionId, string $reason): RefreshOnHoldException
  {
      return new RefreshOnHoldException(
          "Connection {$connectionId} is on hold. {$reason} Don't send its refresh token again: "
          . "that can end the customer's link. See \"If a refresh fails\" in the Linked billing docs."
      );
  }

  /**
   * Returns a current access token and its expiry time in Unix seconds.
   *
   * Call this from your schedule. Pass force: true to refresh a token that isn't
   * close to its expiry. Throws RefreshOnHoldException when the refresh token must not be
   * sent again, and \RuntimeException for a refusal that keeps it.
   *
   * @return array{string, int}
   */
  function refreshConnection(string $connectionId, bool $force = false): array
  {
      // 此锁仅在单台机器上有效。
      $lock = fopen(sys_get_temp_dir() . '/refresh-' . hash('sha256', $connectionId) . '.lock', 'c')
          ?: throw new \RuntimeException('Cannot open the lock file');
      flock($lock, LOCK_EX);
      try {
          $tokens = loadTokens($connectionId);
          if ($tokens->onHold) {
              throw holdError($connectionId, "Reason: {$tokens->holdReason}.");
          }
          if (!$force && $tokens->expiresAt - time() > REFRESH_MARGIN_SECONDS) {
              return [$tokens->accessToken, $tokens->expiresAt];
          }
          $request = json_encode([
              'grant_type' => 'refresh_token',
              'refresh_token' => $tokens->refreshToken,
              'client_id' => CLIENT_ID,
              'client_assertion_type' => JWT_BEARER,
              'client_assertion' => clientAssertion(),
          ], JSON_THROW_ON_ERROR);
          // 先设置搁置标记，这样即使进程在刷新中途停止，搁置标记也会保留。
          $held = $tokens->withHold('lost');
          saveTokens($connectionId, $held);
          $issuedAt = time();
          $started = hrtime(true);
          // 不要添加重试：延迟的重试会终止客户的关联。
          $response = post($request);
          if (is_string($response)) {
              // 刷新令牌从未离开此进程，因此解除搁置标记。
              saveTokens($connectionId, $tokens);
              throw new \RuntimeException("Could not connect to auth.claude.com: {$response}");
          }
          if ($response === false || isLost($response)) {
              if ((hrtime(true) - $started) / 1e9 > SECOND_REQUEST_DEADLINE_SECONDS) {
                  throw holdError($connectionId, 'A refresh got no usable answer.');
              }
              $response = post($request);
              if (!is_array($response)) {
                  throw holdError($connectionId, 'Two refreshes got no answer.');
              }
              if ($response['status'] !== 200) {
                  throw holdError($connectionId, 'The second refresh failed.');
              }
          }
          if ((errorBody($response)['error'] ?? null) === 'invalid_grant') {
              saveTokens($connectionId, $held->withHold('invalid_grant'));
              throw holdError($connectionId, 'The refresh returned invalid_grant.');
          }
          if ($response['status'] !== 200) {
              // 这是一次保留刷新令牌的拒绝，因此解除搁置标记。
              saveTokens($connectionId, $tokens);
              throw new \RuntimeException("Refresh failed: {$response['status']} {$response['body']}");
          }
          /**
           * @var array{
           *     access_token: string,
           *     refresh_token: string,
           *     expires_in: int,
           *     refresh_token_expires_in: int,
           * } $answer
           */
          $answer = json_decode($response['body'], associative: true, flags: JSON_THROW_ON_ERROR);
          $newTokens = new ConnectionTokens(
              refreshToken: $answer['refresh_token'],
              accessToken: $answer['access_token'],
              expiresAt: $issuedAt + $answer['expires_in'],
              refreshExpiresAt: $issuedAt + $answer['refresh_token_expires_in'],
          );
          // 仅重试写入操作，不要再次刷新。
          for ($attempt = 0; $attempt < 5; $attempt++) {
              try {
                  saveTokens($connectionId, $newTokens);
                  return [$newTokens->accessToken, $newTokens->expiresAt];
              } catch (\RuntimeException) { // your secret store's error type
                  if ($attempt < 4) {
                      sleep(2 ** $attempt);
                  }
              }
          }
          $error = holdError($connectionId, 'The refresh succeeded, but the new tokens could not be saved.');
          $error->newTokens = $newTokens;
          throw $error;
      } finally {
          flock($lock, LOCK_UN);
          fclose($lock);
      }
  }
  ```

  ```ruby Ruby
  CONNECTION_LOCKS = Hash.new { |locks, connection_id| locks[connection_id] = Mutex.new }
  LOCKS_GUARD = Mutex.new

  # 如果无法连接，则在请求发出前抛出异常。
  def open_connection
    Net::HTTP.start(TOKEN_URL.host, TOKEN_URL.port, use_ssl: true,
      open_timeout: 2, read_timeout: 10, write_timeout: 10, max_retries: 0)
  end

  def post(http, body)
    http.post(TOKEN_URL.path, body, "content-type" => "application/json")
  ensure
    http.finish
  end

  def error_body(response)
    body = JSON.parse(response.body.to_s)
    body.is_a?(Hash) ? body : {}
  rescue JSON::ParserError
    {}
  end

  # 对于可能出现在已生效刷新之后的 5xx 响应，返回 True。
  def lost?(response)
    return false if response.code.to_i < 500

    refused = response.code == "503" &&
      error_body(response)["type"] == "error" &&
      response.key?("retry-after")
    !refused
  end

  def hold_error(connection_id, reason)
    RefreshOnHoldError.new(
      "Connection #{connection_id} is on hold. #{reason} Don't send its refresh token again: " \
      "that can end the customer's link. See \"If a refresh fails\" in the Linked billing docs."
    )
  end

  # 返回当前访问令牌及其过期时间（Unix 秒）。
  #
  # 请从您的定时任务中调用此方法。传入 force: true 可刷新尚未
  # 临近过期的令牌。当刷新令牌不得再次发送时抛出 RefreshOnHoldError，
  # 对于保留该令牌的拒绝则抛出 RuntimeError。
  def refresh_connection(connection_id, force: false)
    # 此锁仅在单个进程内有效。
    LOCKS_GUARD.synchronize { CONNECTION_LOCKS[connection_id] }.synchronize do
      tokens = load_tokens(connection_id)
      raise hold_error(connection_id, "Reason: #{tokens.hold_reason}.") if tokens.on_hold
      if !force && tokens.expires_at - Time.now.to_i > REFRESH_MARGIN_SECONDS
        return tokens.access_token, tokens.expires_at
      end

      request = {
        grant_type: "refresh_token",
        refresh_token: tokens.refresh_token,
        client_id: CLIENT_ID,
        client_assertion_type: JWT_BEARER,
        client_assertion:
      }.to_json
      # 先设置搁置标记，这样即使进程在刷新中途停止，搁置标记也会保留。
      held = tokens.with(on_hold: true, hold_reason: "lost")
      save_tokens(connection_id, held)
      issued_at = Time.now.to_i
      started = Process.clock_gettime(Process::CLOCK_MONOTONIC)
      # 不要添加重试：延迟的重试会终止客户的关联。
      begin
        http = open_connection
      rescue
        # 刷新令牌从未离开此进程，因此解除搁置标记。
        save_tokens(connection_id, tokens)
        raise
      end
      begin
        response = post(http, request)
        lost = lost?(response)
      rescue
        lost = true
      end
      if lost
        if Process.clock_gettime(Process::CLOCK_MONOTONIC) - started > SECOND_REQUEST_DEADLINE_SECONDS
          raise hold_error(connection_id, "A refresh got no usable answer.")
        end
        begin
          response = post(open_connection, request)
        rescue
          raise hold_error(connection_id, "Two refreshes got no answer.")
        end
        raise hold_error(connection_id, "The second refresh failed.") unless response.code == "200"
      end
      if error_body(response)["error"] == "invalid_grant"
        save_tokens(connection_id, held.with(hold_reason: "invalid_grant"))
        raise hold_error(connection_id, "The refresh returned invalid_grant.")
      end
      unless response.code == "200"
        # 这是保留刷新令牌的拒绝，因此解除搁置标记。
        save_tokens(connection_id, tokens)
        raise "Refresh failed: #{response.code} #{response.body}"
      end
      body = JSON.parse(response.body)
      tokens = ConnectionTokens.new(
        refresh_token: body.fetch("refresh_token"),
        access_token: body.fetch("access_token"),
        expires_at: issued_at + body.fetch("expires_in"),
        refresh_expires_at: issued_at + body.fetch("refresh_token_expires_in")
      )
      # 仅重试写入操作。不要再次刷新。
      5.times do |attempt|
        save_tokens(connection_id, tokens)
        return tokens.access_token, tokens.expires_at
      rescue IOError, SystemCallError # your secret store's error types
        sleep(2**attempt) if attempt < 4
      end
      error = hold_error(connection_id, "The refresh succeeded, but the new tokens could not be saved.")
      error.new_tokens = tokens
      raise error
    end
  end
  ```
</CodeGroup>

### 如果 API 密钥可能进入进程

本节仅适用于 `ANTHROPIC_API_KEY` 可能与客户访问令牌同时设置的进程。以下客户端会发送该密钥来代替令牌：

* `ant` CLI
* 从 `ANTHROPIC_AUTH_TOKEN` 读取令牌的 SDK 客户端

在构建时传入令牌的 SDK 客户端只会发送令牌。其他客户端可能会同时发送两者。此时仍以密钥为准，但在 Models 端点上请求可能会被拒绝。当密钥被发送出去时，Claude API 会使用它对请求进行身份验证，因此该请求不再仅限于该客户。

作为此类进程中的防护措施，请在每个请求中发送 `anthropic-user-profile-id` 标头、带有 `bill_to: "user_profile"` 的 `billing_policy`，以及 `user-profiles-billing-2026-10-01` beta 标头。这样，携带密钥发出的请求会产生以下结果之一：

* **带有该标头和 `bill_to: "user_profile"`：**&#x8BF7;求失败，或向该标头指定的客户计费。
* **仅带有该标头，或两者都没有：**&#x41;nthropic 会向您计费。

### 将每个新的访问令牌写入沙箱时

您的后端通过您构建的通道，将每个新令牌发送到沙箱中的代码。该代码以代理的用户身份运行，并将令牌写入模式为 `0700` 的 `credentials` 目录。它会写入一个模式为 `0600` 的新文件，并将其重命名以覆盖旧文件，这样代理永远不会读到不完整的文件。以下函数实现了这一点：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    使用您应用程序的代码写入该文件，并在沙箱中以代理的用户身份运行。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    使用您应用程序的代码写入该文件，并在沙箱中以代理的用户身份运行。
  </CodeGroupItem>

  ```python Python
  import json
  import os
  import tempfile
  from pathlib import Path


  def write_credentials_file(
      config_dir: Path, profile: str, token: str, expires_at: int
  ) -> None:
      """Run this in the sandbox, as the agent's user."""
      credentials_dir = config_dir / "credentials"
      credentials_dir.mkdir(mode=0o700, parents=True, exist_ok=True)
      body = {"type": "oauth_token", "access_token": token, "expires_at": expires_at}
      handle, temporary_path = tempfile.mkstemp(dir=credentials_dir)  # mode 0600
      with os.fdopen(handle, "w") as temporary_file:
          json.dump(body, temporary_file)
      os.replace(temporary_path, credentials_dir / f"{profile}.json")
  ```

  ```typescript TypeScript
  import { mkdir, rename, writeFile } from "node:fs/promises";
  import { join } from "node:path";

  /** Run this in the sandbox, as the agent's user. */
  async function writeCredentialsFile(
    configDir: string,
    profile: string,
    token: string,
    expiresAt: number
  ) {
    const credentialsDir = join(configDir, "credentials");
    await mkdir(credentialsDir, { mode: 0o700, recursive: true });
    const body = { type: "oauth_token", access_token: token, expires_at: expiresAt };
    const temporaryPath = join(credentialsDir, `.${crypto.randomUUID()}.tmp`);
    await writeFile(temporaryPath, JSON.stringify(body), { mode: 0o600, flag: "wx" });
    await rename(temporaryPath, join(credentialsDir, `${profile}.json`));
  }
  ```

  ```csharp C#
  using System.Text.Json;

  /// <summary>以代理的用户身份在沙箱中运行此命令。</summary>
  static void WriteCredentialsFile(string configDir, string profile, string token, long expiresAt)
  {
      var credentialsDir = Path.Combine(configDir, "credentials");
      Directory.CreateDirectory(credentialsDir, UnixFileMode.UserRead | UnixFileMode.UserWrite | UnixFileMode.UserExecute);
      var body = new Dictionary<string, object>
      {
          ["type"] = "oauth_token",
          ["access_token"] = token,
          ["expires_at"] = expiresAt,
      };
      var temporaryPath = Path.Combine(credentialsDir, $".{Guid.NewGuid()}.tmp");
      using (var temporaryFile = new FileStream(temporaryPath, new FileStreamOptions
      {
          Mode = FileMode.CreateNew,
          Access = FileAccess.Write,
          UnixCreateMode = UnixFileMode.UserRead | UnixFileMode.UserWrite,
      }))
      {
          JsonSerializer.Serialize(temporaryFile, body);
      }
      File.Move(temporaryPath, Path.Combine(credentialsDir, $"{profile}.json"), overwrite: true);
  }
  ```

  ```go Go
  import (
  	"encoding/json"
  	"os"
  	"path/filepath"
  )

  // writeCredentialsFile 在沙箱中以代理的用户身份运行。
  func writeCredentialsFile(configDir, profile, token string, expiresAt int64) error {
  	credentialsDir := filepath.Join(configDir, "credentials")
  	if err := os.MkdirAll(credentialsDir, 0o700); err != nil {
  		return err
  	}
  	body, err := json.Marshal(map[string]any{"type": "oauth_token", "access_token": token, "expires_at": expiresAt})
  	if err != nil {
  		return err
  	}
  	temporaryFile, err := os.CreateTemp(credentialsDir, ".tmp-*") // mode 0600
  	if err != nil {
  		return err
  	}
  	defer os.Remove(temporaryFile.Name()) // does nothing once the rename is done
  	if _, err := temporaryFile.Write(body); err != nil {
  		temporaryFile.Close()
  		return err
  	}
  	if err := temporaryFile.Close(); err != nil {
  		return err
  	}
  	return os.Rename(temporaryFile.Name(), filepath.Join(credentialsDir, profile+".json"))
  }
  ```

  ```java Java
  import module java.base;
  import com.fasterxml.jackson.databind.ObjectMapper; // Jackson comes with the Anthropic SDK

  /** Run this in the sandbox, as the agent's user. */
  void writeCredentialsFile(Path configDir, String profile, String token, long expiresAt) throws IOException {
      var credentialsDir = Files.createDirectories(configDir.resolve("credentials"),
          PosixFilePermissions.asFileAttribute(PosixFilePermissions.fromString("rwx------")));
      var body = Map.of("type", "oauth_token", "access_token", token, "expires_at", expiresAt);
      var temporaryPath = Files.createTempFile(credentialsDir, ".", ".tmp"); // mode 0600
      Files.writeString(temporaryPath, new ObjectMapper().writeValueAsString(body));
      Files.move(temporaryPath, credentialsDir.resolve(profile + ".json"),
          StandardCopyOption.ATOMIC_MOVE, StandardCopyOption.REPLACE_EXISTING);
  }
  ```

  ```php PHP
  /** Run this in the sandbox, as the agent's user. */
  function writeCredentialsFile(string $configDir, string $profile, string $token, int $expiresAt): void
  {
      $credentialsDir = "{$configDir}/credentials";
      if (!is_dir($credentialsDir) && !mkdir($credentialsDir, 0700, recursive: true)) {
          throw new \RuntimeException("Cannot create {$credentialsDir}");
      }
      $credentials = ['type' => 'oauth_token', 'access_token' => $token, 'expires_at' => $expiresAt];
      $body = json_encode($credentials, JSON_THROW_ON_ERROR);
      $temporaryPath = tempnam($credentialsDir, '.tmp'); // mode 0600
      if ($temporaryPath === false || dirname($temporaryPath) !== realpath($credentialsDir)) {
          throw new \RuntimeException("Cannot create a file in {$credentialsDir}"); // tempnam can fall back to /tmp
      }
      $finalPath = "{$credentialsDir}/{$profile}.json";
      if (file_put_contents($temporaryPath, $body) === false || !rename($temporaryPath, $finalPath)) {
          throw new \RuntimeException("Cannot write the credentials for {$profile}");
      }
  }
  ```

  ```ruby Ruby
  require "fileutils"
  require "json"
  require "tempfile"

  # 在沙箱中以代理的用户身份运行此命令。
  def write_credentials_file(config_dir, profile, token, expires_at)
    credentials_dir = File.join(config_dir, "credentials")
    FileUtils.mkdir_p(credentials_dir, mode: 0o700)
    body = { type: "oauth_token", access_token: token, expires_at: }
    temporary_file = Tempfile.create(".tmp", credentials_dir) # mode 0600
    temporary_file.write(body.to_json)
    temporary_file.close
    File.rename(temporary_file.path, File.join(credentials_dir, "#{profile}.json"))
  end
  ```
</CodeGroup>

刷新后几分钟内，使用之前令牌的请求会收到 `401`。客户端随后可以重新读取文件，并使用新令牌发送请求。如果单独使用 TypeScript 或 C# SDK，该请求会直接失败。请再发送一次：下一个请求会使用新令牌。SDK 还会在 `expires_at` 之前的最后 2 分钟内（在 Java 中为最后 30 秒内）重新读取该文件。

### 面向智能体

* 刷新时，不要发送 `scope` 参数。
* 每次刷新时，都从文件中重新读取客户端断言。
* 存储新令牌时，使用 `expires_in` 和 `refresh_token_expires_in`，从您发送刷新请求的时间开始计算其过期时间。
* 暂停结束时，仅当 `hold_reason` (typescript, java, php: `holdReason`; csharp, go: `HoldReason`) 为 `invalid_grant` 时才清除 `on_hold` (typescript, java, php: `onHold`; csharp, go: `OnHold`)。
* 写入沙箱的凭据文件时，写入距其 `expires_at` 剩余时间超过 30 秒的令牌。

## 后续步骤

<CardGroup cols={2}>
  <Card title="测试并发布您的应用" icon="rocket" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish">
    添加测试用户、发布您的应用，以及更改或删除应用
  </Card>

  <Card title="您的客户会看到什么" icon="browser" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see">
    您的客户看到的批准页面、支出上限和电子邮件，以便您为他们提供支持
  </Card>
</CardGroup>
