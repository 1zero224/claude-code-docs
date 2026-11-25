---
title: 管理已关联的客户
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers
description: 查看和管理已关联的客户：查看他们的用量，取消关联或重新关联，以及将他们迁移到另一个组织或从其 API 密钥迁移出来。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：根据您与 Anthropic 之间的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在 Beta 测试期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

客户完成关联后，您可以在 Claude Console 和您的后端中管理他们。每个客户在您的组织中都有一个[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)，这是您为关联他们而创建的。关联会创建一个连接，即将客户的 Claude 组织与该配置文件绑定的记录。

## 开始之前

* **客户的 ID：**&#x60A8;在[关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#save-the-connection-and-its-tokens)时存储的用户配置文件 ID（`uprof_…`）和 `connection_id`（`ocon_…`）。
* **要检查客户的状态或使用取消关联端点：**&#x60A8;应用的 [操作员服务账户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#service-account)，以及来自 [工作负载身份联合](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation) 的访问令牌。该令牌必须来自具有 `workspace:developer` 作用域的联合规则，这是默认设置。
* **要通过 `auth.claude.com` 取消关联：**&#x60A8;应用的[客户端身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#set-up-client-authentication)。示例会从 `CLIENT_ASSERTION_FILE` 环境变量指定的文件中读取客户端断言，或者您可以将其粘贴到 `YOUR_CLIENT_ASSERTION_JWT` 处。它是一个面向您应用的 **Expected audience** 的 JWT。
* **要从后端读取用量：**&#x4E00;个 [Admin API 凭据](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#if-you-read-usage-from-your-back-end)。

## 查看客户和用量

在 Console 中打开 [User profiles](https://platform.claude.com/user-profiles) 页面。您应用的 **Overview** 选项卡也会列出其已关联的客户。对于每个用户配置文件，User profiles 页面会列出过去 28 天的 **Requests**、**Tokens** 和 **Your spend**。点击某个配置文件可查看其过去 7、28 或 90 天的用量。

* 具有活动连接的配置文件显示为 **Linked**，连接已结束的显示为 **Unlinked**，从未关联过的显示为 **Not linked**。只有具有 Admin 角色的成员才能看到这些状态和 **Unlink**。
* **Your spend** 是您自己为该配置文件的请求所承担的成本，未扣除 Anthropic 应用的任何额度。
* 由客户付费的请求会计入 **Requests** 和 **Tokens**，但不会计入 **Your spend**。
* 该页面不会显示客户支付的金额或客户的费率。

这些数据大约每小时刷新一次，一个请求可能需要几个小时才会显示。

不要使用这些数据在您选择的金额处停止客户的用量。客户自己的[支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#spend-caps)会实现这一点。

## 检查客户是否已关联

从您的后端，使用客户的用户配置文件 ID 调用 `GET /v1/user_profiles/{id}`。在 `oauth_app_id` 中发送您应用的客户端 ID，并发送 `user-profiles-search-2026-08-30` beta 标头。以您的操作员服务账户身份进行身份验证。SDK 不会发送 `oauth_app_id`，因此示例通过 HTTP 调用该端点。

在此示例中，`ANTHROPIC_AUTH_TOKEN` 保存服务账户的访问令牌，您可以[从您的工作负载获取](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#authenticate-from-your-workload)该令牌。仅在运行 cURL 的地方设置该变量：SDK 和 CLI 会使用它来代替联合身份验证。

<CodeGroup exclude="shell:CLI, python, typescript, csharp, go, java, php, ruby">
  ```bash cURL
  curl "https://api.anthropic.com/v1/user_profiles/uprof_01HV9Z?oauth_app_id=clid_01ABcDeFgHiJkLmNoPqRsTuV" \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: user-profiles-search-2026-08-30"
  ```
</CodeGroup>

在响应中，`connection_status.type` 描述该配置文件与您应用之间的连接：

| `connection_status.type` | 在 Console 中 | 含义 |
| ----- | ----- | ----- |
| `connected` | **Linked** | 与您应用的连接处于活动状态 |
| `disconnected` | **Unlinked** | 该配置文件曾被关联，且其连接已结束 |
| `not_connected` | **Not linked** | 该配置文件从未关联到您的应用 |

`connected` 状态并不保证请求一定会向客户计费。在每个向客户计费的请求中，都要处理两个 `403` 代码和两个 `402` [计费错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#while-billing)。

对于 `disconnected` 配置文件，以下字段会提供更多信息。列出的原因仅为示例：

| 字段 | 含义 |
| ----- | ----- |
| `last_revoked_at` | 最后一个连接结束的时间 |
| `last_revoked_reason.type` | 原因。`revoked_by_admin`：客户组织中具有 Admin 或 Billing 角色的成员取消了与您应用的关联。`revoked_by_app_owner`：您的组织或您的后端取消了与该客户的关联。 |

对于任何状态，`oauth_connections.data` 都会列出该配置文件与您应用之间的连接，最多 100 个；当列表可能不完整时，会设置 `truncated`。

## 取消客户关联

<Warning>
  取消关联会阻止新的向客户计费的请求，直到您交换新一次批准得到的授权码为止。
</Warning>

在 Console 中，您可以在两个位置取消客户关联：

* **您应用的 Overview 选项卡：**&#x5728; **Linked customers** 下，客户所在行菜单中的 **Unlink** 仅结束其与该应用的连接。
* **User profiles 页面：** **Unlink** 位于配置文件的行菜单中以及其页面的右上角。它会结束该配置文件与您所有应用（包括测试应用）的连接。

从您的后端，向 `auth.claude.com` 发送撤销请求，附带您应用的客户端身份验证以及以下其中一项：

* **`connection_id`：**&#x7ED3;束该单个连接。如果您有此 ID，请使用它。
* **`user_profile_id`：**&#x4EE3;替 `connection_id` 发送，它会结束您的应用在该配置文件上的所有连接，包括客户在您存储 ID 之后建立的连接。

<CodeGroup exclude="shell:CLI">
  ```bash cURL
  curl https://auth.claude.com/v1/oauth/revoke \
    -H "content-type: application/json" \
    -d @- <<'JSON'
  {
    "connection_id": "ocon_01HV9Z",
    "client_id": "clid_01ABcDeFgHiJkLmNoPqRsTuV",
    "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
    "client_assertion": "YOUR_CLIENT_ASSERTION_JWT"
  }
  JSON
  ```

  <CodeGroupItem language="CLI">
    `ant` CLI 没有针对 `auth.claude.com` 的命令。请使用 cURL 或您应用程序的语言发送请求。
  </CodeGroupItem>

  ```python Python
  import os
  from pathlib import Path

  import httpx2  # comes with the Anthropic SDK

  # 您在关联该客户时存储的连接 ID
  connection_id = "ocon_01HV9Z"

  assertion_file = Path(os.environ["CLIENT_ASSERTION_FILE"])

  response = httpx2.post(
      "https://auth.claude.com/v1/oauth/revoke",
      json={
          "connection_id": connection_id,
          "client_id": "clid_01ABcDeFgHiJkLmNoPqRsTuV",
          "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
          "client_assertion": assertion_file.read_text().strip(),
      },
      timeout=10,
  )
  if not response.is_success:
      raise RuntimeError(f"Revoke failed: {response.status_code} {response.text}")

  print(response.json()["revoked_connection_ids"])
  ```

  ```typescript TypeScript
  import { readFile } from "node:fs/promises";

  // 您在关联客户时存储的连接 ID
  const connectionId = "ocon_01HV9Z";

  const jwt = await readFile(process.env.CLIENT_ASSERTION_FILE!, "utf8");

  const response = await fetch("https://auth.claude.com/v1/oauth/revoke", {
    method: "POST",
    redirect: "error",
    signal: AbortSignal.timeout(10_000),
    headers: { "content-type": "application/json" },
    body: JSON.stringify({
      connection_id: connectionId,
      client_id: "clid_01ABcDeFgHiJkLmNoPqRsTuV",
      client_assertion_type: "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
      client_assertion: jwt.trim()
    })
  });
  if (!response.ok) {
    throw new Error(`Revoke failed: ${response.status} ${await response.text()}`);
  }

  const result = (await response.json()) as { revoked_connection_ids: string[] };
  console.log(result.revoked_connection_ids);
  ```

  ```csharp C#
  using System.Net.Http.Json;
  using System.Text.Json;

  // 您在关联该客户时存储的连接 ID
  var connectionId = "ocon_01HV9Z";

  var assertionFile = Environment.GetEnvironmentVariable("CLIENT_ASSERTION_FILE")!;
  var jwt = await File.ReadAllTextAsync(assertionFile);

  using var http = new HttpClient { Timeout = TimeSpan.FromSeconds(10) };
  using var response = await http.PostAsJsonAsync(
      "https://auth.claude.com/v1/oauth/revoke",
      new Dictionary<string, string>
      {
          ["connection_id"] = connectionId,
          ["client_id"] = "clid_01ABcDeFgHiJkLmNoPqRsTuV",
          ["client_assertion_type"] = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
          ["client_assertion"] = jwt.Trim(),
      }
  );
  if (!response.IsSuccessStatusCode)
  {
      var details = await response.Content.ReadAsStringAsync();
      throw new HttpRequestException($"Revoke failed: {(int)response.StatusCode} {details}");
  }

  var result = await response.Content.ReadFromJsonAsync<JsonElement>();
  Console.WriteLine(result.GetProperty("revoked_connection_ids"));
  ```

  ```go Go
  import (
  	"bytes"
  	"encoding/json"
  	"fmt"
  	"io"
  	"log"
  	"net/http"
  	"os"
  	"strings"
  	"time"
  )

  func main() {
  	// 您在关联该客户时存储的连接 ID
  	connectionID := "ocon_01HV9Z"

  	jwt, err := os.ReadFile(os.Getenv("CLIENT_ASSERTION_FILE"))
  	if err != nil {
  		log.Fatal(err)
  	}

  	body, err := json.Marshal(map[string]string{
  		"connection_id":         connectionID,
  		"client_id":             "clid_01ABcDeFgHiJkLmNoPqRsTuV",
  		"client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
  		"client_assertion":      strings.TrimSpace(string(jwt)),
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	httpClient := &http.Client{Timeout: 10 * time.Second}
  	endpoint := "https://auth.claude.com/v1/oauth/revoke"
  	response, err := httpClient.Post(endpoint, "application/json", bytes.NewReader(body))
  	if err != nil {
  		log.Fatal(err)
  	}
  	defer response.Body.Close()
  	if response.StatusCode/100 != 2 {
  		details, _ := io.ReadAll(response.Body)
  		log.Fatalf("revoke failed: %d %s", response.StatusCode, details)
  	}

  	var result struct {
  		RevokedConnectionIDs []string `json:"revoked_connection_ids"`
  	}
  	if err := json.NewDecoder(response.Body).Decode(&result); err != nil {
  		log.Fatal(err)
  	}
  	fmt.Println(result.RevokedConnectionIDs)
  }
  ```

  ```java Java
  import module java.base;
  import module java.net.http;
  import com.fasterxml.jackson.databind.ObjectMapper; // Jackson comes with the Anthropic SDK

  void main() throws Exception {
      // 您在关联客户时存储的连接 ID
      var connectionId = "ocon_01HV9Z";

      var jwt = Files.readString(Path.of(System.getenv("CLIENT_ASSERTION_FILE")));

      var mapper = new ObjectMapper();
      var body = mapper.createObjectNode()
          .put("connection_id", connectionId)
          .put("client_id", "clid_01ABcDeFgHiJkLmNoPqRsTuV")
          .put("client_assertion_type", "urn:ietf:params:oauth:client-assertion-type:jwt-bearer")
          .put("client_assertion", jwt.strip());

      var request = HttpRequest.newBuilder(URI.create("https://auth.claude.com/v1/oauth/revoke"))
          .header("content-type", "application/json")
          .timeout(Duration.ofSeconds(10))
          .POST(HttpRequest.BodyPublishers.ofString(body.toString()))
          .build();
      var response = HttpClient.newHttpClient().send(request, HttpResponse.BodyHandlers.ofString());
      if (response.statusCode() / 100 != 2) {
          throw new IOException("Revoke failed: " + response.statusCode() + " " + response.body());
      }

      IO.println(mapper.readTree(response.body()).get("revoked_connection_ids"));
  }
  ```

  ```php PHP
  // 您在关联该客户时存储的连接 ID
  $connectionId = 'ocon_01HV9Z';

  $jwt = file_get_contents((string) getenv('CLIENT_ASSERTION_FILE'))
      ?: throw new \RuntimeException('Cannot read the client assertion file');

  $body = json_encode([
      'connection_id' => $connectionId,
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
  $response = file_get_contents('https://auth.claude.com/v1/oauth/revoke', context: $context);
  $status = http_get_last_response_headers()[0] ?? '';
  if ($response === false || !preg_match('{^HTTP/\S+ 2}', $status)) {
      throw new \RuntimeException("Revoke failed: {$status} {$response}");
  }

  /** @var array{revoked_connection_ids: list<string>} $result */
  $result = json_decode($response, associative: true, flags: JSON_THROW_ON_ERROR);
  echo implode(', ', $result['revoked_connection_ids']), PHP_EOL;
  ```

  ```ruby Ruby
  require "json"
  require "net/http"

  # 您在关联该客户时存储的连接 ID
  connection_id = "ocon_01HV9Z"

  jwt = File.read(ENV.fetch("CLIENT_ASSERTION_FILE"))

  body = {
    connection_id:,
    client_id: "clid_01ABcDeFgHiJkLmNoPqRsTuV",
    client_assertion_type: "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
    client_assertion: jwt.strip
  }
  response = Net::HTTP.start("auth.claude.com", 443, use_ssl: true, read_timeout: 10) do |http|
    http.post("/v1/oauth/revoke", body.to_json, "content-type" => "application/json")
  end
  raise "Revoke failed: #{response.code} #{response.body}" unless response.is_a?(Net::HTTPSuccess)

  puts(JSON.parse(response.body).fetch("revoked_connection_ids"))
  ```
</CodeGroup>

响应中的 `revoked_connection_ids` 列出了该请求结束的连接。空列表表示没有找到需要结束的连接：通常是连接已经结束，或者 ID 未知。这意味着您可以安全地重复发送带有 `connection_id` 的请求。要进行确认，请[检查客户是否已关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#check-a-links-status)。如果要改为通过 Claude API 取消关联，请参阅[如果您通过 Claude API 取消关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#if-you-unlink-through-the-claude-api)。

取消关联后，会发生以下情况：

* 该配置文件显示为 **Unlinked**。取消关联不会删除它。
* 该连接的 OAuth 令牌停止生效，因此您可以从密钥存储中删除它们。
* 无论是谁取消的关联，新的向客户计费的请求通常会在大约 10 秒内停止。已在运行的请求会完成，并向客户计费。
* 向客户计费的请求会收到 [`403 user_profile_billing_not_authorized`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-billing-not-authorized)。使用客户访问令牌发送的请求可能会改为收到 [`401 authentication_error`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#authentication-error)。
* 当您通过任一途径从后端取消关联时，Anthropic 会向客户的 Admin 和 Billing 成员发送电子邮件。当您在 Console 中取消关联时，不会向他们发送电子邮件。请参阅[您的客户会收到哪些电子邮件](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#emails)。

您仍然可以使用 [`bill_to: "self"`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#you-pay) 以您自己的成本为客户提供服务。每当您取消客户关联时，请亲自告知客户，因为 Anthropic 不保证电子邮件会被发送或送达。

## 重新关联客户

如果客户的连接已结束（**Unlinked**），请使用其现有的用户配置文件再次[引导他们进行批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#send-them-to-approve)。无需取消关联，而新建配置文件会拆分其用量历史记录。

当连接处于活动状态时，您的操作取决于该连接是否具有 刷新令牌；如果客户批准了 `offline_access`，则连接具有刷新令牌：

* **连接没有刷新令牌：**&#x4E0D;要取消关联。使用相同的 `scope` 再次引导他们进行批准，这会[为您提供新的访问令牌](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#if-a-customer-approves-again)。如果访问令牌已泄露，请先[取消客户关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#unlink-a-customer)，因为再次批准不会撤销该令牌。
* **连接具有刷新令牌，即使您已丢失该令牌：**&#x53D6;消客户关联，然后让他们再次批准，如以下步骤所述。

当客户的连接具有刷新令牌时，在以下情况下重新关联客户：

* 刷新返回了 `invalid_grant`，或者刷新失败导致您停止为该客户刷新。首先，[排除组织被暂停和您这一方的原因](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer#if-a-refresh-fails)。
* 您丢失了刷新令牌、刷新令牌已泄露，或者您丢失了授权码交换的响应。
* 刷新令牌即将达到其 1 年的有效期。
* 您更换了操作员服务账户，并且您使用该客户的 OAuth 令牌。

从取消关联开始，直到您新的授权码交换成功为止，通过该连接的计费会停止。首先，与客户组织中具有 Admin 或 Billing 角色的人员约定一个时间。然后：

1. 如果配置文件显示为 **Linked**，请[取消客户关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#unlink-a-customer)。
2. 使用其现有的用户配置文件和相同的作用域（包括 `offline_access`）再次[引导客户进行批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#send-them-to-approve)。
3. 存储新的 `connection_id` 和新的 OAuth 令牌，替换之前的值。之后该配置文件会再次显示为 **Linked**。

## 将客户迁移到另一个组织

要通过客户的另一个 Claude 组织向其计费，请取消客户关联，然后重新关联。在您取消关联之前，批准页面只会提供已关联的组织。从取消关联开始，直到您交换针对另一个组织的批准所得的授权码为止，向客户计费的请求都会被拒绝。

当另一个组织中具有 Admin 或 Billing 角色的人员准备好批准时开始操作。然后：

1. 除非客户已在其 [Linked platforms](https://platform.claude.com/linked-platforms) 页面上取消了与您应用的关联，否则请[取消客户关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#unlink-a-customer)。
2. 使用相同的用户配置文件和作用域再次[引导他们进行批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#send-them-to-approve)，并让该人员为另一个组织批准。
3. 存储新的 `connection_id` 以替换之前的值，如果您使用 OAuth 令牌，也请存储新的令牌。之后该配置文件会通过另一个组织再次显示为 **Linked**。

第一个组织为您的应用设置的支出上限不会沿用。关于适用于另一个组织的上限，请参阅["您的客户会看到什么"中的支出上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see#spend-caps)。

## 将客户从其 API 密钥迁移到 Linked billing

在将客户从其自己的 API 密钥迁移出来之前，请权衡两项成本：

* **无法向客户计费的部分由您支付：**[Message Batches](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)、Claude Managed Agents 以及计费的代码执行。
* **请求在您的组织中运行：**&#x9002;用您的速率限制和数据条款。客户的零数据保留、业务伙伴协议（BAA）、客户管理的加密密钥（CMEK）和 IP 允许列表不适用。请参阅[适用谁的条款和限制](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#whose-data-terms-and-rate-limits-apply)。

要迁移客户：

1. [关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer)，并存储其 `connection_id`。
2. 不再使用其密钥，而是以您的操作员服务账户身份为其请求计费，并在 [`billing_policy`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#bill-a-request) 中设置 `bill_to: "user_profile"`。
3. 删除存储的 API 密钥，并告知客户您已删除。

之后，他们的请求会计入您 [User profiles](https://platform.claude.com/user-profiles) 页面上其配置文件的 **Requests**，而不会计入 **Your spend**。

## 决定检查客户是否已关联的频率

客户可以随时在其 [Linked platforms](https://platform.claude.com/linked-platforms) 页面上取消与您应用的关联，而 Anthropic 不会通知您。请决定您的产品何时获知此情况：

* **当请求被拒绝时：**&#x67E5;找[拒绝的原因](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#while-billing)，并[检查客户是否仍处于关联状态](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#check-a-links-status)。
* **在计费之前：**&#x5F53;客户在您的产品中启动会向其计费的操作时进行检查。
* **按计划：**&#x68C0;查每个客户，并使您产品中关于哪些客户已关联的视图保持最新。

检查得越频繁，就能越早发现取消关联，代价是更多的请求。

## 实现细节、规则和特殊情况

本页其余部分介绍这些任务背后的 API 细节，还介绍一些特殊情况，例如取消关联失败。

### 如果您通过 Claude API 取消关联

您也可以改为通过 Claude API 的取消关联端点取消关联：在 `oauth_app_id` 中发送您应用的客户端 ID 到 `POST /v1/user_profiles/{id}/unlink`。它结束的连接与带有 `user_profile_id` 的撤销请求相同。

| 细节 | 撤销请求 | 取消关联端点 |
| ----- | ----- | ----- |
| 主机 | `auth.claude.com` | `api.anthropic.com` |
| 身份验证 | 您应用的客户端身份验证 | 您的操作员服务账户 |
| Beta 标头 | 无 | `user-profiles-search-2026-08-30` |
| 结束的内容 | 一个连接，或您的应用在某个配置文件上的所有连接 | 您的应用在某个配置文件上的所有连接 |

在此示例中，`ANTHROPIC_AUTH_TOKEN` 保存您操作员服务账户的访问令牌，与[检查客户是否已关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#check-a-links-status)中相同。

<CodeGroup exclude="shell:CLI, python, typescript, csharp, go, java, php, ruby">
  ```bash cURL
  curl https://api.anthropic.com/v1/user_profiles/uprof_01HV9Z/unlink \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: user-profiles-search-2026-08-30" \
    -H "content-type: application/json" \
    -d '{"oauth_app_id": "clid_01ABcDeFgHiJkLmNoPqRsTuV"}'
  ```
</CodeGroup>

响应包含 `revoked_connection_ids`，如果没有活动的连接，则该字段为空。

### 如果取消关联端点返回 503

取消关联端点返回 `503` 意味着它可能只结束了部分连接。请为您在失败请求之前存储的每个 `connection_id` 发送撤销请求。此方法随时有效。它会结束剩余的连接，且不会影响客户在此之后建立的连接。

在第一次请求后一天内，使用退避策略重复取消关联也有效，但它也会结束客户在此之后建立的连接。超过一天后，重复请求可能会返回 `200` 和空列表。这种较晚的重复请求也不会撤销失败请求所结束连接的剩余部分，因此它们可能仍保留 OAuth 令牌或计费访问权限。

### 如果状态检查或取消关联端点拒绝请求

| 状态 | 状态检查 | 取消关联端点 |
| ----- | ----- | ----- |
| `400` | `oauth_app_id` 不是客户端 ID 的格式，或缺少 beta 标头 | 缺少 `oauth_app_id`，或其不是客户端 ID 的格式 |
| `401` | 凭据缺失或无效，或其联合规则未授予 `workspace:developer` | 相同 |
| `403` | 调用方不是 `oauth_app_id` 中应用的操作员服务账户，或者它使用的是该账户仅限于一个会话的令牌 | 相同 |
| `404` | 用户配置文件不存在 | 用户配置文件不存在，或缺少 beta 标头 |
| `503` | 使用退避策略重试 | 请参阅[如果取消关联端点返回 503](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#if-an-unlink-returns-a-503) |

撤销请求本身的错误请参阅[撤销端点的错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#errors-from-the-revocation-endpoint)。

### 如果您从后端读取用量

[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 会报告每个用户配置文件的用量。与 Console 一样，它会同时统计您付费的请求和您的客户付费的请求。

请使用 Admin API 凭据调用它，例如具有 `org:admin` 作用域的 OAuth 访问令牌或 [Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api#authentication)。不要使用您的操作员服务账户：其联合规则默认授予 `workspace:developer`，而不是 `org:admin`。也不要为其添加 `org:admin`，因为该作用域涵盖整个 Admin API。

要为每个用户配置文件获取一行数据，请调用[用量端点](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#usage-endpoint)或[成本端点](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#cost-endpoint)。仅按 `user_profile_id` 分组，并省略 `time_bucket_width`。只有在时间范围内有用量的配置文件才会有对应的行。`user_profile_id` 为 null 的行包含未指定任何配置文件的用量，以及在请求之外计量的用量，例如代码执行容器时间。

在成本端点上，您可以通过两个字段判断由谁付费：

| 字段 | 您付费的请求 | 您的客户付费的请求 |
| ----- | ----- | ----- |
| `amount` | 其成本，与 **Your spend** 的计量方式相同 | 0 |
| `list_amount` | 其标价 | 其标价 |

API 没有用于表示由谁付费的字段或筛选条件。`amount` 为 0 且 `list_amount` 大于 0 的行通常意味着客户支付了全部费用。关于数据可能延迟多久，请参阅[数据新鲜度](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#data-freshness)。

### 面向智能体

* 将 `connection_status` 为 `null` 或您无法识别的 `type` 视为未知。
* 将您无法识别的 `last_revoked_reason.type` 视为 `other`。

## 后续步骤

<CardGroup cols={1}>
  <Card title="测试并发布您的应用" icon="rocket" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish">
    添加测试用户、发布您的应用，以及更改或删除应用
  </Card>
</CardGroup>
