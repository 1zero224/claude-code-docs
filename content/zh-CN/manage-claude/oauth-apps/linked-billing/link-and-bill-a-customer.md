---
title: 关联客户并向其计费
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer
description: 引导客户批准您的 OAuth 应用，保存连接，并将 Claude API 请求计费到其 Claude 组织。
featureMetadata:
  status: beta
  betaHeader: user-profiles-billing-2026-10-01
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
    Claude API:
      availability: beta
      note: 只有发送 `billing_policy` 的请求才需要 beta 标头。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：根据您与 Anthropic 之间的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在 Beta 测试期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

本页介绍如何在您的产品中构建 Linked billing。客户只需批准您的应用一次，即可在其 Claude 组织与您的某个用户配置文件之间创建连接。之后，您的后端以您自己的 [operator service account](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#service-account)（操作员服务账户）身份发送请求，并指明客户的[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)，从而向客户计费。您首先在一个测试客户（即您自己的第二个组织）上试用此流程。有关该功能的工作原理以及它是否适合您的产品，请参阅 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing)。

## 开始之前

您需要以下内容：

* **已为 Linked billing 设置好的应用：** 它需要客户端身份验证、操作员服务账户和默认工作区：请参阅[为 Linked billing 设置您的应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app)。
* **一个测试客户：** 您的第二个 Claude 组织，因为拥有您应用的组织无法关联到该应用。它必须直接向 Anthropic 付款，可通过发票计费或[可用于支付这些请求的额度](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#which-credits-can-pay)。
* **一个测试用户：** 测试客户组织中拥有 Admin 或 Billing 角色、代表该组织进行批准的人员。在您的应用上线之前，只有其 **Test users** 列表中的人员才能关联它。请将[此人的 Claude 电子邮件地址添加](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#add-test-users)到该列表中。

### 示例设置

示例会读取以下值：

| 示例 | 所需内容 | 来源 |
| ----- | ----- | ----- |
| SDK 和 CLI | 您的操作员服务账户的联合设置，位于环境变量中，其中包括 `ANTHROPIC_IDENTITY_TOKEN_FILE` 中的联合 JWT 文件 | [构造 SDK 客户端](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#construct-the-sdk-client) |
| 发往 Claude API 的 cURL 请求 | 您的服务账户的访问令牌，位于 `ANTHROPIC_AUTH_TOKEN` 中 | [从您的工作负载进行身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#authenticate-from-your-workload) |
| 发往 `auth.claude.com` 的请求 | 一个 client assertion（客户端断言）：面向您应用的 **Expected audience** 的 JWT，位于 `CLIENT_ASSERTION_FILE` 指定的文件中，或替换 `YOUR_CLIENT_ASSERTION_JWT` | 您的身份提供商 |
| 全部 | 您应用的客户端 ID 和重定向 URI，用于替换示例中的客户端 ID 和重定向 URI | Claude Console 中您应用的页面 |

仅在运行 cURL 示例的环境中设置 `ANTHROPIC_AUTH_TOKEN`。在运行 SDK 和 CLI 示例的环境中，请取消设置它以及 `ANTHROPIC_API_KEY`，因为[它们各自都优先于联合身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/workload-identity-federation#credential-precedence)。

客户端断言和联合 JWT 通常需要不同的受众，因此请将它们保存在不同的文件中：请参阅[两个受众不同时](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#if-the-audiences-differ)。使用 `CLIENT_ASSERTION_FILE` 的示例会在每次请求时读取该文件，因为其中的 JWT 会轮换。

## 选择用户配置文件

[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)是您自己组织中代表您某位客户的一条记录。向客户计费的请求会携带该配置文件的 ID，以告知 Anthropic 向谁计费。您可以关联已有的配置文件（已归档的除外），关联不会改变其 ID 或使用历史。

在同一个应用中，客户的 Claude 组织只能关联到您的一个配置文件，而一个配置文件也只能关联到一个组织。在关联第一个客户之前，请确定哪个配置文件代表哪个组织。组织是客户在批准时选择的那个，因此请询问客户其团队如何使用 Claude 组织：

* **多个团队共用一个 Claude 组织：** 为所有这些团队使用一个配置文件。
* **每个团队都有自己的 Claude 组织：** 为每个组织使用一个配置文件。

如果客户选择的组织已关联到您的另一个配置文件，则批准会被拒绝：请参阅 [Linked billing 错误中的 `user_profile_already_linked`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-already-linked)。

有两种情况会改变您创建的配置文件：[如果您为每个人都有一个配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#if-you-have-a-profile-for-each-person)和[如果您转售 API 访问权限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#if-you-resell-api-access)。

## 关联客户

<Frame>
  ![时序图：客户在 auth.claude.com 进行批准，然后您的后端获取连接和 OAuth 令牌](https://platform.claude.com/docs/zh-CN/images/linked-billing-link-flow.svg)
</Frame>

### 创建用户配置文件

按照您在[选择用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#choose-the-user-profile)中的映射，为每个将要关联的 Claude 组织创建一个用户配置文件。对于已有的配置文件，请跳过此步骤。发送一个名称以及您自己为该客户设定的 ID（`reference_id`）。配置文件只需创建一次，并将其 ID 与客户记录一起存储：每个创建请求都会生成一个新的配置文件，即使 `reference_id` 相同也是如此。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/user_profiles \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "content-type: application/json" \
    -d '{
      "name": "Example Customer Inc.",
      "external_user_details": {"reference_id": "cust_2abc3def"}
    }'
  ```

  ```bash CLI
  ant user-profiles create \
    --name "Example Customer Inc." \
    --external-user-details.reference-id cust_2abc3def
  ```

  ```python Python
  client = anthropic.Anthropic()

  profile = client.user_profiles.create(
      name="Example Customer Inc.",
      external_user_details={"reference_id": "cust_2abc3def"},
  )

  print(profile.id)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const profile = await client.userProfiles.create({
    name: "Example Customer Inc.",
    external_user_details: { reference_id: "cust_2abc3def" }
  });

  console.log(profile.id);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var profile = await client.UserProfiles.Create(
      new()
      {
          Name = "Example Customer Inc.",
          ExternalUserDetails = new() { ReferenceID = "cust_2abc3def" },
      }
  );

  Console.WriteLine(profile.ID);
  ```

  ```go Go
  client := anthropic.NewClient()

  profile, err := client.UserProfiles.New(context.Background(), anthropic.UserProfileNewParams{
  	Name: anthropic.String("Example Customer Inc."),
  	ExternalUserDetails: anthropic.UserProfileExternalUserDetailsParams{
  		ReferenceID: anthropic.String("cust_2abc3def"),
  	},
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Println(profile.ID)
  ```

  ```java Java
  import com.anthropic.models.userprofiles.UserProfileCreateParams;
  import com.anthropic.models.userprofiles.UserProfileExternalUserDetailsParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = UserProfileCreateParams.builder()
          .name("Example Customer Inc.")
          .externalUserDetails(UserProfileExternalUserDetailsParams.builder()
              .referenceId("cust_2abc3def")
              .build())
          .build();
      var profile = client.userProfiles().create(params);

      IO.println(profile.id());
  }
  ```

  ```php PHP
  $client = new Client();

  $profile = $client->userProfiles->create(
      name: 'Example Customer Inc.',
      externalUserDetails: ['referenceID' => 'cust_2abc3def'],
  );

  echo $profile->id, PHP_EOL;
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  profile = client.user_profiles.create(
    name: "Example Customer Inc.",
    external_user_details: { reference_id: "cust_2abc3def" }
  )

  puts(profile.id)
  ```
</CodeGroup>

响应中包含配置文件的 `id`，例如 `uprof_01HV9Z`。

### 引导客户进行批准

在您的产品中客户管理计费或集成的位置，添加一个 [Link your Claude organization 按钮](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-button)。其 HTML、颜色和徽标均可直接复制。

批准页面仅提供当前登录 Claude 账户的组织，由客户选择的组织付款。有关在按钮旁显示的内容，请参阅[告知客户的内容](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#what-to-tell-your-customers)。

当客户点击按钮时，在您的后端执行以下三件事：

1. **创建一个 `state` 值和一个 PKCE 代码验证器，** 即一个用于证明此次批准由您的后端发起的随机密钥。将两者存储在您的后端，并与浏览器会话和客户记录相关联。
2. **向 `auth.claude.com` 发送 pushed authorization request（推送授权请求）**，它从您的后端传递批准的详细信息，而不是通过浏览器传递。
3. **将浏览器重定向**到 `https://auth.claude.com/oauth/authorize`，并附上您的 `client_id` 以及响应中的 `request_uri`。`request_uri` 只能使用一次，并在 10 分钟后过期，因此每次点击都要发送一个新的推送授权请求。

没有 SDK 涵盖 `auth.claude.com`，因此这些步骤的示例通过 HTTP 调用它：

<CodeGroup exclude="shell:CLI">
  ```bash cURL
  curl https://auth.claude.com/v1/oauth/par \
    -H "content-type: application/json" \
    -d @- <<'JSON'
  {
    "response_type": "code",
    "client_id": "clid_01ABcDeFgHiJkLmNoPqRsTuV",
    "redirect_uri": "https://app.example.com/claude/billing/callback",
    "scope": "billing:attribution",
    "connection_type": "organization_shared",
    "connection_auth_target": "platform",
    "user_profile_id": "uprof_01HV9Z",
    "state": "RANDOM_STATE",
    "code_challenge": "BASE64URL_SHA256_OF_VERIFIER",
    "code_challenge_method": "S256",
    "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
    "client_assertion": "YOUR_CLIENT_ASSERTION_JWT"
  }
  JSON
  ```

  <CodeGroupItem language="CLI">
    `ant` CLI 没有用于 `auth.claude.com` 的命令。请使用 cURL 或您应用程序所用的语言发送请求。
  </CodeGroupItem>

  ```python Python
  import base64
  import hashlib
  import os
  import secrets
  from pathlib import Path
  from urllib.parse import urlencode

  import httpx2  # comes with the Anthropic SDK

  CLIENT_ID = "clid_01ABcDeFgHiJkLmNoPqRsTuV"


  def push_authorization_request(user_profile_id: str) -> dict[str, str]:
      state = secrets.token_urlsafe(32)
      code_verifier = secrets.token_urlsafe(64)
      digest = hashlib.sha256(code_verifier.encode()).digest()
      assertion_file = Path(os.environ["CLIENT_ASSERTION_FILE"])

      response = httpx2.post(
          "https://auth.claude.com/v1/oauth/par",
          json={
              "response_type": "code",
              "client_id": CLIENT_ID,
              "redirect_uri": "https://app.example.com/claude/billing/callback",
              "scope": "billing:attribution",
              "connection_type": "organization_shared",
              "connection_auth_target": "platform",
              "user_profile_id": user_profile_id,
              "state": state,
              "code_challenge": base64.urlsafe_b64encode(digest).rstrip(b"=").decode(),
              "code_challenge_method": "S256",
              "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
              "client_assertion": assertion_file.read_text().strip(),
          },
          timeout=10,
      )
      if not response.is_success:
          raise RuntimeError(f"Request failed: {response.status_code} {response.text}")

      query = {"client_id": CLIENT_ID, "request_uri": response.json()["request_uri"]}
      return {
          "approval_url": f"https://auth.claude.com/oauth/authorize?{urlencode(query)}",
          "state": state,
          "code_verifier": code_verifier,
      }
  ```

  ```typescript TypeScript
  import { createHash, randomBytes } from "node:crypto";
  import { readFile } from "node:fs/promises";

  const CLIENT_ID = "clid_01ABcDeFgHiJkLmNoPqRsTuV";

  async function pushAuthorizationRequest(userProfileId: string) {
    const state = randomBytes(32).toString("base64url");
    const codeVerifier = randomBytes(64).toString("base64url");
    const jwt = await readFile(process.env.CLIENT_ASSERTION_FILE!, "utf8");

    const response = await fetch("https://auth.claude.com/v1/oauth/par", {
      method: "POST",
      redirect: "error",
      signal: AbortSignal.timeout(10_000),
      headers: { "content-type": "application/json" },
      body: JSON.stringify({
        response_type: "code",
        client_id: CLIENT_ID,
        redirect_uri: "https://app.example.com/claude/billing/callback",
        scope: "billing:attribution",
        connection_type: "organization_shared",
        connection_auth_target: "platform",
        user_profile_id: userProfileId,
        state,
        code_challenge: createHash("sha256").update(codeVerifier).digest("base64url"),
        code_challenge_method: "S256",
        client_assertion_type: "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
        client_assertion: jwt.trim()
      })
    });
    if (!response.ok) {
      throw new Error(`Request failed: ${response.status} ${await response.text()}`);
    }

    const { request_uri } = (await response.json()) as { request_uri: string };
    const query = new URLSearchParams({ client_id: CLIENT_ID, request_uri });
    return {
      approvalUrl: `https://auth.claude.com/oauth/authorize?${query}`,
      state,
      codeVerifier
    };
  }
  ```

  ```csharp C#
  using System.Buffers.Text;
  using System.Net.Http.Json;
  using System.Security.Cryptography;
  using System.Text;
  using System.Text.Json;

  const string ClientId = "clid_01ABcDeFgHiJkLmNoPqRsTuV";
  using var http = new HttpClient { Timeout = TimeSpan.FromSeconds(10) };

  async Task<(string ApprovalUrl, string State, string CodeVerifier)> PushAuthorizationRequest(
      string userProfileId
  )
  {
      var state = Base64Url.EncodeToString(RandomNumberGenerator.GetBytes(32));
      var codeVerifier = Base64Url.EncodeToString(RandomNumberGenerator.GetBytes(64));
      var digest = SHA256.HashData(Encoding.ASCII.GetBytes(codeVerifier));
      var assertionFile = Environment.GetEnvironmentVariable("CLIENT_ASSERTION_FILE")!;
      var jwt = await File.ReadAllTextAsync(assertionFile);

      using var response = await http.PostAsJsonAsync(
          "https://auth.claude.com/v1/oauth/par",
          new Dictionary<string, string>
          {
              ["response_type"] = "code",
              ["client_id"] = ClientId,
              ["redirect_uri"] = "https://app.example.com/claude/billing/callback",
              ["scope"] = "billing:attribution",
              ["connection_type"] = "organization_shared",
              ["connection_auth_target"] = "platform",
              ["user_profile_id"] = userProfileId,
              ["state"] = state,
              ["code_challenge"] = Base64Url.EncodeToString(digest),
              ["code_challenge_method"] = "S256",
              ["client_assertion_type"] = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
              ["client_assertion"] = jwt.Trim(),
          }
      );
      if (!response.IsSuccessStatusCode)
      {
          var details = await response.Content.ReadAsStringAsync();
          throw new HttpRequestException($"Request failed: {(int)response.StatusCode} {details}");
      }

      var result = await response.Content.ReadFromJsonAsync<JsonElement>();
      var requestUri = Uri.EscapeDataString(result.GetProperty("request_uri").GetString()!);
      var query = $"client_id={ClientId}&request_uri={requestUri}";
      return ($"https://auth.claude.com/oauth/authorize?{query}", state, codeVerifier);
  }
  ```

  ```go Go
  import (
  	"bytes"
  	"crypto/rand"
  	"crypto/sha256"
  	"encoding/base64"
  	"encoding/json"
  	"fmt"
  	"io"
  	"net/http"
  	"net/url"
  	"os"
  	"strings"
  	"time"
  )

  const clientID = "clid_01ABcDeFgHiJkLmNoPqRsTuV"

  var httpClient = &http.Client{Timeout: 10 * time.Second}

  type approval struct{ URL, State, CodeVerifier string }

  func pushAuthorizationRequest(userProfileID string) (*approval, error) {
  	state := rand.Text()
  	codeVerifier := rand.Text() + rand.Text() // PKCE needs at least 43 characters
  	digest := sha256.Sum256([]byte(codeVerifier))
  	jwt, err := os.ReadFile(os.Getenv("CLIENT_ASSERTION_FILE"))
  	if err != nil {
  		return nil, err
  	}

  	body, err := json.Marshal(map[string]string{
  		"response_type":          "code",
  		"client_id":              clientID,
  		"redirect_uri":           "https://app.example.com/claude/billing/callback",
  		"scope":                  "billing:attribution",
  		"connection_type":        "organization_shared",
  		"connection_auth_target": "platform",
  		"user_profile_id":        userProfileID,
  		"state":                  state,
  		"code_challenge":         base64.RawURLEncoding.EncodeToString(digest[:]),
  		"code_challenge_method":  "S256",
  		"client_assertion_type":  "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
  		"client_assertion":       strings.TrimSpace(string(jwt)),
  	})
  	if err != nil {
  		return nil, err
  	}

  	endpoint := "https://auth.claude.com/v1/oauth/par"
  	response, err := httpClient.Post(endpoint, "application/json", bytes.NewReader(body))
  	if err != nil {
  		return nil, err
  	}
  	defer response.Body.Close()
  	if response.StatusCode/100 != 2 {
  		details, _ := io.ReadAll(response.Body)
  		return nil, fmt.Errorf("request failed: %d %s", response.StatusCode, details)
  	}

  	var result struct {
  		RequestURI string `json:"request_uri"`
  	}
  	if err := json.NewDecoder(response.Body).Decode(&result); err != nil {
  		return nil, err
  	}
  	query := url.Values{"client_id": {clientID}, "request_uri": {result.RequestURI}}
  	approvalURL := "https://auth.claude.com/oauth/authorize?" + query.Encode()
  	return &approval{approvalURL, state, codeVerifier}, nil
  }
  ```

  ```java Java
  import module java.base;
  import module java.net.http;
  import com.fasterxml.jackson.databind.ObjectMapper; // Jackson comes with the Anthropic SDK

  static final String CLIENT_ID = "clid_01ABcDeFgHiJkLmNoPqRsTuV";
  static final Base64.Encoder BASE64_URL = Base64.getUrlEncoder().withoutPadding();
  final HttpClient http = HttpClient.newHttpClient();
  final ObjectMapper mapper = new ObjectMapper();

  record Approval(String url, String state, String codeVerifier) {}

  String randomValue(int size) {
      var bytes = new byte[size];
      new SecureRandom().nextBytes(bytes);
      return BASE64_URL.encodeToString(bytes);
  }

  Approval pushAuthorizationRequest(String userProfileId) throws Exception {
      var state = randomValue(32);
      var codeVerifier = randomValue(64);
      var digest = MessageDigest.getInstance("SHA-256")
          .digest(codeVerifier.getBytes(StandardCharsets.US_ASCII));
      var jwt = Files.readString(Path.of(System.getenv("CLIENT_ASSERTION_FILE")));

      var body = mapper.createObjectNode()
          .put("response_type", "code")
          .put("client_id", CLIENT_ID)
          .put("redirect_uri", "https://app.example.com/claude/billing/callback")
          .put("scope", "billing:attribution")
          .put("connection_type", "organization_shared")
          .put("connection_auth_target", "platform")
          .put("user_profile_id", userProfileId)
          .put("state", state)
          .put("code_challenge", BASE64_URL.encodeToString(digest))
          .put("code_challenge_method", "S256")
          .put("client_assertion_type", "urn:ietf:params:oauth:client-assertion-type:jwt-bearer")
          .put("client_assertion", jwt.strip());

      var request = HttpRequest.newBuilder(URI.create("https://auth.claude.com/v1/oauth/par"))
          .header("content-type", "application/json")
          .timeout(Duration.ofSeconds(10))
          .POST(HttpRequest.BodyPublishers.ofString(body.toString()))
          .build();
      var response = http.send(request, HttpResponse.BodyHandlers.ofString());
      if (response.statusCode() / 100 != 2) {
          throw new IOException("Request failed: " + response.statusCode() + " " + response.body());
      }

      var requestUri = mapper.readTree(response.body()).get("request_uri").asText();
      var query = "client_id=%s&request_uri=%s"
          .formatted(CLIENT_ID, URLEncoder.encode(requestUri, StandardCharsets.UTF_8));
      return new Approval("https://auth.claude.com/oauth/authorize?" + query, state, codeVerifier);
  }
  ```

  ```php PHP
  const CLIENT_ID = 'clid_01ABcDeFgHiJkLmNoPqRsTuV';

  function base64Url(string $bytes): string
  {
      return rtrim(strtr(base64_encode($bytes), '+/', '-_'), '=');
  }

  /** @return array{approvalUrl: string, state: string, codeVerifier: string} */
  function pushAuthorizationRequest(string $userProfileId): array
  {
      $state = base64Url(random_bytes(32));
      $codeVerifier = base64Url(random_bytes(64));
      $jwt = file_get_contents((string) getenv('CLIENT_ASSERTION_FILE'))
          ?: throw new \RuntimeException('Cannot read the client assertion file');

      $body = json_encode([
          'response_type' => 'code',
          'client_id' => CLIENT_ID,
          'redirect_uri' => 'https://app.example.com/claude/billing/callback',
          'scope' => 'billing:attribution',
          'connection_type' => 'organization_shared',
          'connection_auth_target' => 'platform',
          'user_profile_id' => $userProfileId,
          'state' => $state,
          'code_challenge' => base64Url(hash('sha256', $codeVerifier, binary: true)),
          'code_challenge_method' => 'S256',
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
      $response = file_get_contents('https://auth.claude.com/v1/oauth/par', context: $context);
      $status = http_get_last_response_headers()[0] ?? '';
      if ($response === false || !preg_match('{^HTTP/\S+ 2}', $status)) {
          throw new \RuntimeException("Request failed: {$status} {$response}");
      }

      /** @var array{request_uri: string} $result */
      $result = json_decode($response, associative: true, flags: JSON_THROW_ON_ERROR);
      $query = http_build_query(['client_id' => CLIENT_ID, 'request_uri' => $result['request_uri']]);

      return [
          'approvalUrl' => "https://auth.claude.com/oauth/authorize?{$query}",
          'state' => $state,
          'codeVerifier' => $codeVerifier,
      ];
  }
  ```

  ```ruby Ruby
  require "base64"
  require "digest"
  require "json"
  require "net/http"
  require "securerandom"

  CLIENT_ID = "clid_01ABcDeFgHiJkLmNoPqRsTuV"

  def push_authorization_request(user_profile_id)
    state = SecureRandom.urlsafe_base64(32)
    code_verifier = SecureRandom.urlsafe_base64(64)
    digest = Digest::SHA256.digest(code_verifier)
    jwt = File.read(ENV.fetch("CLIENT_ASSERTION_FILE"))

    body = {
      response_type: "code",
      client_id: CLIENT_ID,
      redirect_uri: "https://app.example.com/claude/billing/callback",
      scope: "billing:attribution",
      connection_type: "organization_shared",
      connection_auth_target: "platform",
      user_profile_id:,
      state:,
      code_challenge: Base64.urlsafe_encode64(digest, padding: false),
      code_challenge_method: "S256",
      client_assertion_type: "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
      client_assertion: jwt.strip
    }
    response = Net::HTTP.start("auth.claude.com", 443, use_ssl: true, read_timeout: 10) do |http|
      http.post("/v1/oauth/par", body.to_json, "content-type" => "application/json")
    end
    unless response.is_a?(Net::HTTPSuccess)
      raise "Request failed: #{response.code} #{response.body}"
    end

    request_uri = JSON.parse(response.body).fetch("request_uri")
    query = URI.encode_www_form(client_id: CLIENT_ID, request_uri:)
    { approval_url: "https://auth.claude.com/oauth/authorize?#{query}", state:, code_verifier: }
  end
  ```
</CodeGroup>

推送授权请求携带您的客户端 ID 和一个客户端断言、客户的用户配置文件 ID 以及作用域。它还携带您已注册的重定向 URI、`state`，以及由验证器生成的代码质询。`connection_type` 和 `connection_auth_target` 将其标记为 Linked billing 请求。有关每个字段及其值，请参阅 [OAuth 端点和参数](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/endpoints-and-parameters#authorization-requests)。

作用域为 `billing:attribution`。若还要获取刷新令牌，请请求 `billing:attribution offline_access`，这需要为您的应用开启 **Offline access**。是否需要刷新令牌取决于[您的后端如何计费](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/set-up-your-app#choose-whether-to-get-refresh-tokens)。为已关联的客户添加 `offline_access` 会创建一个新连接来替换先前的连接：请参阅[如果客户再次批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#if-a-customer-approves-again)。

在批准页面上，客户会在需要时登录 Claude，选择要关联的组织并批准。在您的应用上线之前，批准者还必须为您的应用设置支出上限：请参阅[测试期间的上限](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#caps-while-testing)。

### 保存连接及其 OAuth 令牌

客户批准后，`auth.claude.com` 会将浏览器发送到您的重定向 URI。然后，在您的后端执行以下操作：

1. **检查回调。** 检查其 `state` 是否与您存储的一致，这表明返回的是同一个浏览器。检查其 `iss` 是否为 `https://identity.anthropic.com/users`，这表明回调来自 Claude。如果任一项缺失或不匹配，请停止。
2. **检查错误。** 如果回调携带的是 `error` 参数而不是授权码，则表示未建立连接。有关原因，请参阅 [Linked billing 错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#while-linking)。
3. **立即交换授权码。** 将其与 PKCE 代码验证器、推送授权请求中的重定向 URI 以及一个客户端断言一起发送。授权码在 5 分钟后过期。
4. **存储响应返回的内容。** 交换只会返回一次 OAuth 令牌。

以下示例通过 HTTP 向 `auth.claude.com` 发送授权码交换请求：

<CodeGroup exclude="shell:CLI">
  ```bash cURL
  curl https://auth.claude.com/v1/oauth/token \
    -H "content-type: application/json" \
    -d @- <<'JSON'
  {
    "grant_type": "authorization_code",
    "code": "AUTH_CODE",
    "code_verifier": "ORIGINAL_CODE_VERIFIER",
    "redirect_uri": "https://app.example.com/claude/billing/callback",
    "client_id": "clid_01ABcDeFgHiJkLmNoPqRsTuV",
    "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
    "client_assertion": "YOUR_CLIENT_ASSERTION_JWT"
  }
  JSON
  ```

  <CodeGroupItem language="CLI">
    `ant` CLI 没有用于 `auth.claude.com` 的命令。请使用 cURL 或您应用程序所用的语言发送请求。
  </CodeGroupItem>

  ```python Python
  import os
  import secrets
  from pathlib import Path

  import httpx2  # comes with the Anthropic SDK


  def handle_callback(params: dict[str, str], state: str, code_verifier: str) -> dict:
      if not secrets.compare_digest(params.get("state", "").encode(), state.encode()):
          raise ValueError("state does not match")
      if params.get("iss") != "https://identity.anthropic.com/users":
          raise ValueError("iss does not match")
      if "error" in params:
          raise PermissionError(params["error"])

      assertion_file = Path(os.environ["CLIENT_ASSERTION_FILE"])

      response = httpx2.post(
          "https://auth.claude.com/v1/oauth/token",
          json={
              "grant_type": "authorization_code",
              "code": params["code"],
              "code_verifier": code_verifier,
              "redirect_uri": "https://app.example.com/claude/billing/callback",
              "client_id": "clid_01ABcDeFgHiJkLmNoPqRsTuV",
              "client_assertion_type": "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
              "client_assertion": assertion_file.read_text().strip(),
          },
          timeout=10,
      )
      if not response.is_success:
          raise RuntimeError(f"Exchange failed: {response.status_code} {response.text}")
      return response.json()  # connection_id and the tokens
  ```

  ```typescript TypeScript
  import { timingSafeEqual } from "node:crypto";
  import { readFile } from "node:fs/promises";

  async function handleCallback(params: URLSearchParams, state: string, codeVerifier: string) {
    const received = new TextEncoder().encode(params.get("state") ?? "");
    const expected = new TextEncoder().encode(state);
    if (received.length !== expected.length || !timingSafeEqual(received, expected)) {
      throw new Error("state does not match");
    }
    if (params.get("iss") !== "https://identity.anthropic.com/users") {
      throw new Error("iss does not match");
    }
    if (params.has("error")) {
      throw new Error(`Linking failed: ${params.get("error")}`);
    }

    const jwt = await readFile(process.env.CLIENT_ASSERTION_FILE!, "utf8");

    const response = await fetch("https://auth.claude.com/v1/oauth/token", {
      method: "POST",
      redirect: "error",
      signal: AbortSignal.timeout(10_000),
      headers: { "content-type": "application/json" },
      body: JSON.stringify({
        grant_type: "authorization_code",
        code: params.get("code"),
        code_verifier: codeVerifier,
        redirect_uri: "https://app.example.com/claude/billing/callback",
        client_id: "clid_01ABcDeFgHiJkLmNoPqRsTuV",
        client_assertion_type: "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
        client_assertion: jwt.trim()
      })
    });
    if (!response.ok) {
      throw new Error(`Exchange failed: ${response.status} ${await response.text()}`);
    }
    return response.json(); // connection_id and the tokens
  }
  ```

  ```csharp C#
  using System.Net.Http.Json;
  using System.Security.Cryptography;
  using System.Text;
  using System.Text.Json;

  using var http = new HttpClient { Timeout = TimeSpan.FromSeconds(10) };

  async Task<JsonElement> HandleCallback(
      Dictionary<string, string> parameters,
      string state,
      string codeVerifier
  )
  {
      var received = Encoding.UTF8.GetBytes(parameters.GetValueOrDefault("state", ""));
      if (!CryptographicOperations.FixedTimeEquals(received, Encoding.UTF8.GetBytes(state)))
      {
          throw new InvalidOperationException("state does not match");
      }
      if (parameters.GetValueOrDefault("iss") != "https://identity.anthropic.com/users")
      {
          throw new InvalidOperationException("iss does not match");
      }
      if (parameters.TryGetValue("error", out var error))
      {
          throw new InvalidOperationException($"Linking failed: {error}");
      }

      var assertionFile = Environment.GetEnvironmentVariable("CLIENT_ASSERTION_FILE")!;
      var jwt = await File.ReadAllTextAsync(assertionFile);

      using var response = await http.PostAsJsonAsync(
          "https://auth.claude.com/v1/oauth/token",
          new Dictionary<string, string>
          {
              ["grant_type"] = "authorization_code",
              ["code"] = parameters["code"],
              ["code_verifier"] = codeVerifier,
              ["redirect_uri"] = "https://app.example.com/claude/billing/callback",
              ["client_id"] = "clid_01ABcDeFgHiJkLmNoPqRsTuV",
              ["client_assertion_type"] = "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
              ["client_assertion"] = jwt.Trim(),
          }
      );
      if (!response.IsSuccessStatusCode)
      {
          var details = await response.Content.ReadAsStringAsync();
          throw new HttpRequestException($"Exchange failed: {(int)response.StatusCode} {details}");
      }

      // connection_id 和令牌
      return await response.Content.ReadFromJsonAsync<JsonElement>();
  }
  ```

  ```go Go
  import (
  	"bytes"
  	"crypto/subtle"
  	"encoding/json"
  	"errors"
  	"fmt"
  	"io"
  	"net/http"
  	"net/url"
  	"os"
  	"strings"
  	"time"
  )

  var httpClient = &http.Client{Timeout: 10 * time.Second}

  func handleCallback(params url.Values, state, codeVerifier string) (map[string]any, error) {
  	if subtle.ConstantTimeCompare([]byte(params.Get("state")), []byte(state)) != 1 {
  		return nil, errors.New("state does not match")
  	}
  	if params.Get("iss") != "https://identity.anthropic.com/users" {
  		return nil, errors.New("iss does not match")
  	}
  	if params.Has("error") {
  		return nil, fmt.Errorf("linking failed: %s", params.Get("error"))
  	}

  	jwt, err := os.ReadFile(os.Getenv("CLIENT_ASSERTION_FILE"))
  	if err != nil {
  		return nil, err
  	}

  	body, err := json.Marshal(map[string]string{
  		"grant_type":            "authorization_code",
  		"code":                  params.Get("code"),
  		"code_verifier":         codeVerifier,
  		"redirect_uri":          "https://app.example.com/claude/billing/callback",
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
  		return nil, fmt.Errorf("exchange failed: %d %s", response.StatusCode, details)
  	}

  	var connection map[string]any // connection_id and the tokens
  	err = json.NewDecoder(response.Body).Decode(&connection)
  	return connection, err
  }
  ```

  ```java Java
  import module java.base;
  import module java.net.http;
  import com.fasterxml.jackson.databind.JsonNode;
  import com.fasterxml.jackson.databind.ObjectMapper; // Jackson comes with the Anthropic SDK

  final HttpClient http = HttpClient.newHttpClient();
  final ObjectMapper mapper = new ObjectMapper();

  JsonNode handleCallback(Map<String, String> params, String state, String codeVerifier)
          throws Exception {
      var received = params.getOrDefault("state", "").getBytes(StandardCharsets.UTF_8);
      if (!MessageDigest.isEqual(received, state.getBytes(StandardCharsets.UTF_8))) {
          throw new IllegalStateException("state does not match");
      }
      if (!"https://identity.anthropic.com/users".equals(params.get("iss"))) {
          throw new IllegalStateException("iss does not match");
      }
      if (params.containsKey("error")) {
          throw new IllegalStateException("Linking failed: " + params.get("error"));
      }

      var jwt = Files.readString(Path.of(System.getenv("CLIENT_ASSERTION_FILE")));

      var body = mapper.createObjectNode()
          .put("grant_type", "authorization_code")
          .put("code", params.get("code"))
          .put("code_verifier", codeVerifier)
          .put("redirect_uri", "https://app.example.com/claude/billing/callback")
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
          throw new IOException("Exchange failed: " + response.statusCode() + " " + response.body());
      }
      return mapper.readTree(response.body()); // connection_id and the tokens
  }
  ```

  ```php PHP
  /** @param array<string, string> $params */
  function handleCallback(array $params, string $state, string $codeVerifier): mixed
  {
      if (!hash_equals($state, $params['state'] ?? '')) {
          throw new \RuntimeException('state does not match');
      }
      if (($params['iss'] ?? '') !== 'https://identity.anthropic.com/users') {
          throw new \RuntimeException('iss does not match');
      }
      if (isset($params['error'])) {
          throw new \RuntimeException("Linking failed: {$params['error']}");
      }

      $jwt = file_get_contents((string) getenv('CLIENT_ASSERTION_FILE'))
          ?: throw new \RuntimeException('Cannot read the client assertion file');

      $body = json_encode([
          'grant_type' => 'authorization_code',
          'code' => $params['code'],
          'code_verifier' => $codeVerifier,
          'redirect_uri' => 'https://app.example.com/claude/billing/callback',
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
          throw new \RuntimeException("Exchange failed: {$status} {$response}");
      }

      // connection_id 和令牌
      return json_decode($response, associative: true, flags: JSON_THROW_ON_ERROR);
  }
  ```

  ```ruby Ruby
  require "json"
  require "net/http"
  require "openssl"

  def handle_callback(params, state, code_verifier)
    raise "state does not match" unless OpenSSL.secure_compare(params["state"].to_s, state)
    raise "iss does not match" unless params["iss"] == "https://identity.anthropic.com/users"
    raise "Linking failed: #{params["error"]}" if params.key?("error")

    jwt = File.read(ENV.fetch("CLIENT_ASSERTION_FILE"))

    body = {
      grant_type: "authorization_code",
      code: params.fetch("code"),
      code_verifier:,
      redirect_uri: "https://app.example.com/claude/billing/callback",
      client_id: "clid_01ABcDeFgHiJkLmNoPqRsTuV",
      client_assertion_type: "urn:ietf:params:oauth:client-assertion-type:jwt-bearer",
      client_assertion: jwt.strip
    }
    response = Net::HTTP.start("auth.claude.com", 443, use_ssl: true, read_timeout: 10) do |http|
      http.post("/v1/oauth/token", body.to_json, "content-type" => "application/json")
    end
    unless response.is_a?(Net::HTTPSuccess)
      raise "Exchange failed: #{response.code} #{response.body}"
    end

    JSON.parse(response.body) # connection_id and the tokens
  end
  ```
</CodeGroup>

响应包含以下值：

| 值 | 含义 | 处理方式 |
| ----- | ----- | ----- |
| `connection_id` | 连接的 ID | 将其与客户记录一起存储，以便日后从您的后端终止连接 |
| `access_token` | 客户自己的令牌，与您服务账户的令牌分开：它只能向此客户计费，有效期最长 1 小时 | 将其作为机密存储 |
| `refresh_token` | 如果您请求了 `offline_access`，则为用于获取新访问令牌的令牌 | 将其作为机密存储 |

响应不会说明客户关联的是哪个组织。

本页中的请求以您的服务账户身份计费，因此不需要 OAuth 令牌。但仍请存储它们：您需要它们来[为每个客户使用单独的凭据](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/credential-for-each-customer)。如果您请求了刷新令牌，除非您取消关联该客户并由其重新关联，否则无法再次获取它。

如果交换失败，处理方式取决于失败的情形：

* **交换返回 `unauthorized_client`：** 通常是因为您的应用没有操作员服务账户或默认工作区，或者该工作区已归档。请参阅[如何修复 `unauthorized_client`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#code-exchange-unauthorized-client)。
* **交换返回 `invalid_grant`：** 重定向 URI 或验证器有误，或者授权码已过期或已被使用。错误的重定向 URI 或验证器会消耗掉该授权码，因此客户必须再次批准。对于已关联的客户，请参阅[如果客户再次批准](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#if-a-customer-approves-again)。
* **交换超时：** [再发送一次相同的授权码](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#if-the-code-exchange-times-out)。

## 为请求计费

以您的操作员服务账户身份为此客户发送每个请求，并在 `anthropic-user-profile-id` 标头中放入客户的用户配置文件 ID。SDK 会根据其用户配置文件参数为您发送该标头。无论由谁付款，该标头都会使请求计入该配置文件的用量。然后，针对每个请求选择由客户付款还是由您付款。

### 由客户付款

要将请求计费给客户，请在该标头中指明持有其连接的配置文件，并同时发送：

* 请求正文中的 [`billing_policy`](https://platform.claude.com/docs/zh-CN/api/beta/messages/create)，设置为 `{"type": "billing_policy", "bill_to": "user_profile"}`
* `user-profiles-billing-2026-10-01` beta 标头

<Warning>
  如果请求指明了另一个已关联客户的配置文件，Anthropic 会向该客户的组织计费，并且不会有任何错误提示您。
</Warning>

请检查每个请求的配置文件 ID。与任何客户一样，您的测试客户会为这些请求支付真实的费用。

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: user-profiles-billing-2026-10-01" \
    -H "anthropic-user-profile-id: uprof_01HV9Z" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}],
      "billing_policy": {"type": "billing_policy", "bill_to": "user_profile"}
    }'
  ```

  ```bash CLI
  ant beta:messages create \
    --beta user-profiles-billing-2026-10-01 \
    --user-profile-id uprof_01HV9Z \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}' \
    --billing-policy '{type: billing_policy, bill_to: user_profile}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  message = client.beta.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
      betas=["user-profiles-billing-2026-10-01"],
      user_profile_id="uprof_01HV9Z",
      billing_policy={"type": "billing_policy", "bill_to": "user_profile"},
  )

  for block in message.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.beta.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    betas: ["user-profiles-billing-2026-10-01"],
    user_profile_id: "uprof_01HV9Z",
    billing_policy: { type: "billing_policy", bill_to: "user_profile" }
  });

  for (const block of message.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Messages;
  using Model = Anthropic.Models.Messages.Model;

  AnthropicClient client = new();

  var message = await client.Beta.Messages.Create(
      new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new BetaUserMessageParam { Content = "Hello, Claude" }],
          Betas = ["user-profiles-billing-2026-10-01"],
          UserProfileID = "uprof_01HV9Z",
          BillingPolicy = new() { BillTo = BetaBillTo.UserProfile },
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
  client := anthropic.NewClient()

  message, err := client.Beta.Messages.New(context.Background(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude")),
  	},
  	Betas:         []anthropic.AnthropicBeta{"user-profiles-billing-2026-10-01"},
  	UserProfileID: anthropic.String("uprof_01HV9Z"),
  	BillingPolicy: anthropic.BetaBillingPolicyParam{BillTo: anthropic.BetaBillToUserProfile},
  })
  if err != nil {
  	log.Fatal(err)
  }

  for _, block := range message.Content {
  	if textBlock, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaBillTo;
  import com.anthropic.models.beta.messages.BetaBillingPolicy;
  import com.anthropic.models.beta.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .addUserMessage("Hello, Claude")
          .addBeta("user-profiles-billing-2026-10-01")
          .userProfileId("uprof_01HV9Z")
          .billingPolicy(BetaBillingPolicy.builder()
              .billTo(BetaBillTo.USER_PROFILE)
              .build())
          .build();
      var message = client.beta().messages().create(params);

      for (var block : message.content()) {
          block.text().ifPresent(textBlock -> IO.println(textBlock.text()));
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Messages\BetaBillingPolicy;
  use Anthropic\Beta\Messages\BetaBillTo;
  use Anthropic\Beta\Messages\BetaTextBlock;
  use Anthropic\Messages\Model;

  $client = new Client();

  $message = $client->beta->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_OPUS_5_5,
      betas: ['user-profiles-billing-2026-10-01'],
      userProfileID: 'uprof_01HV9Z',
      billingPolicy: BetaBillingPolicy::with(billTo: BetaBillTo::USER_PROFILE),
  );

  foreach ($message->content as $block) {
      if ($block instanceof BetaTextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.beta.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    betas: ["user-profiles-billing-2026-10-01"],
    user_profile_id: "uprof_01HV9Z",
    billing_policy: { type: :billing_policy, bill_to: :user_profile }
  )

  message.content.each do |block|
    puts(block.text) if block.is_a?(Anthropic::Beta::BetaTextBlock)
  end
  ```
</CodeGroup>

Claude API 要么向客户计费，要么拒绝该请求。它绝不会转而向您计费。在 `/v1/messages` 上，被拒绝的请求会在 `error.details.error_code` 中携带其错误代码。可能的原因如下：

* **新连接：** 新连接在授权码交换后可能需要大约 10 秒才能生效。如果您的第一个请求在这 10 秒内因 `user_profile_billing_not_authorized` 被拒绝，请在 10 秒过后再次发送。如果仍被拒绝，请参阅[如何修复 `user_profile_billing_not_authorized`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-billing-not-authorized)。
* **无法计费的客户：** `user_profile_not_billable` 可能表示客户的组织无法被计费，例如因为它尚未设置计费。请参阅[如何修复 `user_profile_not_billable`](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#user-profile-not-billable)。
* **达到支出限额或没有额度：** 当客户通过您的应用产生的支出达到其上限，或其组织达到自身的月度支出限额时，请求会因 `linked_organization_spend_limit_reached` 被拒绝。当其账户上没有可用于支付的额度时，也会发生同样的情况。
* **其他情况：** 请参阅[计费过程中的 Linked billing 错误](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/errors#while-billing)。

### 由您付款

若要自己为请求付款，请继续在标头中发送客户的用户配置文件 ID。然后，省略 `billing_policy`，或将其 `bill_to` 设置为 `"self"`。

不使用 `billing_policy` 时，beta 标头是可选的：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-user-profile-id: uprof_01HV9Z" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}]
    }'
  ```

  ```bash CLI
  ant messages create \
    --user-profile-id uprof_01HV9Z \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
      user_profile_id="uprof_01HV9Z",
  )

  for block in message.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    user_profile_id: "uprof_01HV9Z"
  });

  for (const block of message.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  using Anthropic.Models.Messages;

  AnthropicClient client = new();

  var message = await client.Messages.Create(
      new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new UserMessageParam { Content = "Hello, Claude" }],
          UserProfileID = "uprof_01HV9Z",
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
  client := anthropic.NewClient()

  message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  	UserProfileID: anthropic.String("uprof_01HV9Z"),
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
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .addUserMessage("Hello, Claude")
          .userProfileId("uprof_01HV9Z")
          .build();
      var message = client.messages().create(params);

      for (var block : message.content()) {
          block.text().ifPresent(textBlock -> IO.println(textBlock.text()));
      }
  }
  ```

  ```php PHP
  use Anthropic\Messages\Model;
  use Anthropic\Messages\TextBlock;

  $client = new Client();

  $message = $client->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_OPUS_5_5,
      userProfileID: 'uprof_01HV9Z',
  );

  foreach ($message->content as $block) {
      if ($block instanceof TextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    user_profile_id: "uprof_01HV9Z"
  )

  message.content.each do |block|
    puts(block.text) if block.is_a?(Anthropic::Models::TextBlock)
  end
  ```
</CodeGroup>

将 `bill_to` 设置为 `"self"` 时，请求需要 beta 标头：

<CodeGroup>
  ```bash cURL
  curl https://api.anthropic.com/v1/messages \
    -H "authorization: Bearer $ANTHROPIC_AUTH_TOKEN" \
    -H "anthropic-version: 2026-10-21" \
    -H "anthropic-beta: user-profiles-billing-2026-10-01" \
    -H "anthropic-user-profile-id: uprof_01HV9Z" \
    -H "content-type: application/json" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [{"role": "user", "content": "Hello, Claude"}],
      "billing_policy": {"type": "billing_policy", "bill_to": "self"}
    }'
  ```

  ```bash CLI
  ant beta:messages create \
    --beta user-profiles-billing-2026-10-01 \
    --user-profile-id uprof_01HV9Z \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}' \
    --billing-policy '{type: billing_policy, bill_to: self}'
  ```

  ```python Python
  client = anthropic.Anthropic()

  message = client.beta.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
      betas=["user-profiles-billing-2026-10-01"],
      user_profile_id="uprof_01HV9Z",
      billing_policy={"type": "billing_policy", "bill_to": "self"},
  )

  for block in message.content:
      if block.type == "text":
          print(block.text)
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const message = await client.beta.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    betas: ["user-profiles-billing-2026-10-01"],
    user_profile_id: "uprof_01HV9Z",
    billing_policy: { type: "billing_policy", bill_to: "self" }
  });

  for (const block of message.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Messages;
  using Model = Anthropic.Models.Messages.Model;

  AnthropicClient client = new();

  var message = await client.Beta.Messages.Create(
      new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new BetaUserMessageParam { Content = "Hello, Claude" }],
          Betas = ["user-profiles-billing-2026-10-01"],
          UserProfileID = "uprof_01HV9Z",
          BillingPolicy = new() { BillTo = BetaBillTo.Self },
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
  client := anthropic.NewClient()

  message, err := client.Beta.Messages.New(context.Background(), anthropic.BetaMessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.BetaMessageParam{
  		anthropic.NewBetaUserMessage(anthropic.NewBetaTextBlock("Hello, Claude")),
  	},
  	Betas:         []anthropic.AnthropicBeta{"user-profiles-billing-2026-10-01"},
  	UserProfileID: anthropic.String("uprof_01HV9Z"),
  	BillingPolicy: anthropic.BetaBillingPolicyParam{BillTo: anthropic.BetaBillToSelf},
  })
  if err != nil {
  	log.Fatal(err)
  }

  for _, block := range message.Content {
  	if textBlock, ok := block.AsAny().(anthropic.BetaTextBlock); ok {
  		fmt.Println(textBlock.Text)
  	}
  }
  ```

  ```java Java
  import com.anthropic.models.beta.messages.BetaBillTo;
  import com.anthropic.models.beta.messages.BetaBillingPolicy;
  import com.anthropic.models.beta.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .addUserMessage("Hello, Claude")
          .addBeta("user-profiles-billing-2026-10-01")
          .userProfileId("uprof_01HV9Z")
          .billingPolicy(BetaBillingPolicy.builder()
              .billTo(BetaBillTo.SELF)
              .build())
          .build();
      var message = client.beta().messages().create(params);

      for (var block : message.content()) {
          block.text().ifPresent(textBlock -> IO.println(textBlock.text()));
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Messages\BetaBillingPolicy;
  use Anthropic\Beta\Messages\BetaBillTo;
  use Anthropic\Beta\Messages\BetaTextBlock;
  use Anthropic\Messages\Model;

  $client = new Client();

  $message = $client->beta->messages->create(
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      model: Model::CLAUDE_OPUS_5_5,
      betas: ['user-profiles-billing-2026-10-01'],
      userProfileID: 'uprof_01HV9Z',
      billingPolicy: BetaBillingPolicy::with(billTo: BetaBillTo::SELF),
  );

  foreach ($message->content as $block) {
      if ($block instanceof BetaTextBlock) {
          echo $block->text, PHP_EOL;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  message = client.beta.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }],
    betas: ["user-profiles-billing-2026-10-01"],
    user_profile_id: "uprof_01HV9Z",
    billing_policy: { type: :billing_policy, bill_to: :self }
  )

  message.content.each do |block|
    puts(block.text) if block.is_a?(Anthropic::Beta::BetaTextBlock)
  end
  ```
</CodeGroup>

## 检查结果

首先，在 Claude Console 中检查客户是否已关联：

* 您的 User profiles 页面将客户的配置文件显示为 **Linked**。只有拥有 Admin 角色的成员才能看到此状态。
* 您应用的 **Overview** 选项卡在 **Linked customers** 下列出该客户，并附有您存储的连接 ID。

然后，在 Console 中检查账单，因为成功计费的请求会返回正常响应，不会说明由谁付款。测试客户的 [Linked platforms](https://platform.claude.com/linked-platforms) 页面向该组织中拥有 Admin 或 Billing 角色的成员开放。

| 位置 | 显示内容 | 更新时间 |
| ----- | ----- | ----- |
| 测试客户的 [Linked platforms](https://platform.claude.com/linked-platforms) 页面 | 您的应用本月的支出，以及对应的上限 | 大约一分钟内 |
| 测试客户的 Linked platforms 页面 | 您的应用本月的成本和用量 | 每隔几小时 |
| 您的 [User profiles](https://platform.claude.com/user-profiles) 页面 | **Requests** 和 **Tokens** 会计入该请求。**Your spend** 不会计入。 | 大约每小时一次，但请求可能需要几个小时才会显示 |

测试成功后，[发布您的应用](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish#publish-the-app)，以便您的客户可以关联它。

## 告知客户的内容

在真实客户关联之前，请在您的产品中告知他们以下内容：

* **谁可以批准：** 其 Claude 组织中拥有 Admin 或 Billing 角色的人员。
* **他们关联的是您产品中的哪个账户：** 在按钮旁显示此信息。
* **他们使用的是哪个 Claude 账户：** 请他们核对批准页面显示的账户。
* **他们已关联：** 授权码交换完成后，说明客户已关联，而不是说计费已生效。客户的组织可能在 Anthropic 能够向其计费之前就已关联，例如在其设置计费之前。
* **您向他们收取哪些费用：** 仅限他们及其用户在您的产品中产生的 Claude 用量。有关完整规则，请参阅[您可以计费的内容](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing#what-you-may-bill)。

## 规则和特殊情况

以下各节说明了前述步骤背后的规则，以及在特定情况下应如何处理。

### 如果您的联合规则和客户端身份验证期望不同的受众

您的后端会在两个地方发送来自身份提供商的 JWT：

* **发送给联合身份验证，用于 Claude API：** 该 JWT 必须具有您的联合规则的受众。受众留空的规则期望 `https://api.anthropic.com`。
* **作为客户端断言发送给 `auth.claude.com`：** 该 JWT 必须恰好具有一个受众，即您应用 **Client authentication** 选项卡中的 **Expected audience**。

当规则的受众留空时，两者通常不同，因此请从您的身份提供商为每个受众各获取一个 JWT。SDK 和 CLI 从 `ANTHROPIC_IDENTITY_TOKEN_FILE` 指定的文件中读取联合 JWT。客户端断言则放在 `CLIENT_ASSERTION_FILE` 指定的文件中。

### 如果授权码交换超时

授权码只能使用一次，但超时后重试是安全的。如果第一次尝试从未到达 `auth.claude.com`，则在授权码的 5 分钟有效期内重试会成功。如果第一次尝试已送达，重试会返回 `invalid_grant` 且不会改变任何内容。在这种情况下，请[检查客户是否已关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#check-a-links-status)：

* **未关联：** 从推送授权请求重新开始。
* **已关联：** 您没有丢失的响应中所包含的连接 ID 或 OAuth 令牌。如果您只请求了 `billing:attribution`，请引导客户以相同的作用域再次批准。这会返回相同的连接 ID 和一个新的访问令牌。如果您还请求了 `offline_access`，请按照[重新关联客户](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)操作。

### 如果您为每个人都有一个配置文件

您可能已经为使用您产品的每个人都创建了用户配置文件，而其中几个人可能供职于同一个客户。每个组织只能关联一个配置文件，因此 Anthropic 建议您为该客户的组织再创建一个配置文件并关联它。然后，您发送的配置文件 ID 取决于由谁付款：

* **由客户付款的请求：** 发送客户配置文件的 ID。若还要告知 Anthropic 请求来自哪个人，请在 `metadata.user_id` 中发送该人员的不透明 ID，切勿发送姓名、电子邮件地址或电话号码。请参阅[识别公司内的某一个人](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#identify-individual-users-within-a-company)。
* **由您付款的请求：** 您可以继续发送该人员自己的配置文件 ID。

### 如果您转售 API 访问权限

如果您的组织转售 API 访问权限，请在创建客户的配置文件时同时发送 `"access_type": "passthrough"`。示例省略了 `access_type`，这会创建一个 `application` 配置文件。有关这两种类型，请参阅[确定为谁创建配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles#decide-who-gets-a-profile)。

### 如果客户再次批准

当您引导客户为一个已有连接的配置文件进行批准时，客户就会再次批准。授权码交换的结果取决于该连接的作用域和新批准的作用域：

| 客户批准的内容 | 授权码交换的结果 | 您的操作 |
| ----- | ----- | ----- |
| 再次批准 `billing:attribution` | 返回相同的连接 ID 和一个新的访问令牌 | 存储新的访问令牌 |
| 再次批准 `billing:attribution offline_access` | 返回 `invalid_grant`，连接保持不变 | 要获取新的 OAuth 令牌，请[取消关联该客户并让其重新关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again) |
| 其他作用域，例如在 `billing:attribution` 基础上添加 `offline_access` | 终止先前的连接及其 OAuth 令牌，并创建一个具有新 ID 的新连接 | 存储新的连接 ID 和 OAuth 令牌 |

如果作用域变更的交换以 5xx 响应失败，先前的连接可能已经终止。在新的交换成功之前，您向该客户计费的请求都会被拒绝。要进行恢复：

1. 引导客户以新的作用域再次批准。
2. 如果该交换返回 `invalid_grant`，请[取消关联该客户并让其重新关联](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/manage-customers#link-a-customer-again)。

### 面向智能体

* 如果推送授权请求失败，请不要重定向浏览器。
* 当您发送带有 `bill_to: "user_profile"` 的请求时，请在您自己的日志中记录它：没有任何响应会说明由谁付款。

## 后续步骤

<CardGroup cols={1}>
  <Card title="您的客户会看到什么" icon="browser" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see">
    您的客户看到的批准页面、支出上限和电子邮件，以便您为他们提供支持
  </Card>
</CardGroup>
