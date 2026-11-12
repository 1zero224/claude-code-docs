---
title: Claude Models on Google Cloud (Anthropic managed)
url: https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed
description: Claude Models on Google Cloud (Anthropic managed) 是由 Anthropic 运行、通过您的 Google Cloud 项目访问的 Claude API。
---

Google Cloud 负责处理 Claude Models on Google Cloud (Anthropic managed) 的访问和计费。要比较这两种 Google Cloud 产品，或查看您使用的是哪一种，请参阅 [Claude Models on Google Cloud](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/overview)。

## 设置您的账户

要为 Claude Models on Google Cloud (Anthropic managed) 设置账户，您需要具备以下条件：

* 在您的结算账户上于 Google Cloud Marketplace 中进行购买的权限
* 在项目上启用服务的权限
* 一个[允许您打开 Claude Console 的 IAM 角色](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#using-the-claude-console)，第 4 步需要用到

这些步骤会为您的 Google Cloud 项目创建一个新的 Anthropic 组织。它独立于您公司已有的任何 Anthropic 组织，包括 Claude Enterprise 组织。要从现有组织迁移，请参阅[从现有 Anthropic 组织迁移](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#moving-from-an-existing-anthropic-organization)。

如果您已有折扣，请在开始这些步骤之前联系您的 Anthropic 或 Google Cloud 客户代表。折扣无法应用于过去的使用量。

<Steps>
  <Step title="在 Marketplace 中订阅">
    1. 打开 [Google Cloud Marketplace](https://console.cloud.google.com/marketplace)，找到 **Claude Models on Google Cloud (Anthropic managed)** 商品页面。
    2. 点击 **Subscribe** 并完成购买流程。
    3. 返回商品页面并点击 **Manage on Console**。这会在 Google Cloud 控制台中打开 Claude 页面。

    以后要返回 Claude 页面，请在 Google Cloud 控制台中选择您的项目。然后前往 **Agent Platform > Models > Claude Platform** 并选择 **Anthropic managed**。
  </Step>

  <Step title="启用服务">
    在每个调用 Claude Models on Google Cloud (Anthropic managed) 的项目上启用两项服务：

    * `claude.googleapis.com` 提供 Claude Models on Google Cloud (Anthropic managed) 的 API。该服务还定义了 IAM 权限。没有它，调用会失败。
    * `anthropic-managed-claude.cloudpartnerservices.goog` 是将项目的使用量归属到您的订阅的 Marketplace 服务。

    ```bash CLI
    gcloud services enable claude.googleapis.com \
      anthropic-managed-claude.cloudpartnerservices.goog \
      --project=my-project-id
    ```

    将 `my-project-id` 替换为您的项目 ID。

    您也可以从 Claude 页面或 **APIs & Services** 启用这两项服务。每个项目只需启用一次。要检查两项服务是否都已启用，请运行 `gcloud services list --enabled --project=my-project-id` 并查找这两个名称。
  </Step>

  <Step title="设置您的 Anthropic 组织">
    1. 在 Claude 页面上，点击 **Activate**。
    2. 在 Anthropic 设置流程中，输入联系电子邮件地址。
    3. 打开发送到该地址的设置链接。
    4. 填写组织名称、实体类型、国家/地区和预期用途。
    5. 接受 Anthropic 的商业服务条款和使用政策。

    联系地址仅用于设置和服务通知。关于设置完成后用户如何打开 Claude Console，请参阅[使用 Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#using-the-claude-console)。
  </Step>

  <Step title="打开 Claude Console">
    当您的 Anthropic 组织处于活动状态时，Claude 页面上的按钮会变为 **Open Claude Console**。点击它。Google Cloud 会检查您的 IAM 访问权限，然后将您的浏览器重定向到 `platform.claude.com`。您将登录到新组织。

    在 Claude Console 中，[找到您的工作区 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#find-your-workspace-id) 并复制它。每个调用 Claude Models on Google Cloud (Anthropic managed) 的人都需要它。请将其转交给您的开发人员。
  </Step>
</Steps>

要添加另一个项目，请在该项目中从第 2 步开始重复这些步骤。

### 账户设置故障排除

如果您在设置 Claude Models on Google Cloud (Anthropic managed) 时遇到以下问题之一，请应用相应的解决方法：

* **`gcloud services enable` 找不到 `anthropic-managed-claude.cloudpartnerservices.goog`：** 使用完成 Marketplace 购买的账户运行该命令，或联系您的 Anthropic 客户代表。
* **首次 API 调用时出现权限错误：** 检查您调用的项目上是否已启用 `claude.googleapis.com`。检查调用方在该项目上是否拥有 `roles/claude.user` 或 `roles/claude.admin`。关于每个角色允许的操作，请参阅 [IAM 角色](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#iam-roles)。新授予的权限或新启用的服务可能需要片刻才能生效。
* **点击设置链接后显示"已使用其他账户登录"：** 点击**退出登录并继续**。然后使用您在第 3 步中输入的电子邮件地址。
* **按钮显示"Finish Setup"而不是"Open Claude Console"：** 您的 Anthropic 组织已存在，但其设置尚未完成。管理员需要填写完组织详细信息表单。
* **首次 API 调用后用量页面没有显示数据：** 使用数据可能需要几分钟才会出现在 Claude Console 中。

## 管理访问权限

### IAM 角色

Claude Models on Google Cloud (Anthropic managed) 有三个预定义的 IAM 角色。请在项目上授予角色。

| 角色                    | 显示名称          | 允许的操作                                                                                                   |
| --------------------- | ------------- | ------------------------------------------------------------------------------------------------------- |
| `roles/claude.viewer` | Claude Viewer | 工作区资源上的所有 `get` 和 `list` 权限<br />此角色为只读。                                                                |
| `roles/claude.user`   | Claude User   | `roles/claude.viewer` 允许的所有操作，以及创建和更改工作区资源<br />这包括发送消息和计算令牌数，还包括管理批处理、文件、技能和 Claude Managed Agents 资源。 |
| `roles/claude.admin`  | Claude Admin  | `roles/claude.user` 允许的所有操作，以及在您的 Anthropic 组织上检查的权限<br />组织级路由作用于整个 Anthropic 组织，而不是单个工作区。             |

例如，以下命令允许服务账号调用 Claude Models on Google Cloud (Anthropic managed)：

```bash CLI
gcloud projects add-iam-policy-binding my-project-id \
  --member="serviceAccount:my-app@my-project-id.iam.gserviceaccount.com" \
  --role="roles/claude.user"
```

关于权限、预定义角色和审计日志类型，请参阅 [Claude Models on Google Cloud (Anthropic managed) 的 IAM 权限](https://platform.claude.com/docs/zh-CN/api/claude-models-on-google-cloud-anthropic-managed-iam-permissions)。

启用 `claude.googleapis.com` 会为项目创建一个服务代理。请勿移除其角色授权。移除它们会导致 Claude Models on Google Cloud (Anthropic managed) 在该项目中停止工作。

### 使用 Claude Console

使用 Claude Models on Google Cloud (Anthropic managed) 时，您无法使用电子邮件地址登录您的 Anthropic 组织。相反，请在 Google Cloud 控制台的 [Claude 页面](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#set-up-your-account)上点击 **Open Claude Console**。Google Cloud IAM 控制谁可以打开 Claude Console。在三个预定义的 [IAM 角色](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#iam-roles)中，只有项目上的 `roles/claude.admin` 允许您打开 Claude Console。

### 工作区

在 Claude Models on Google Cloud (Anthropic managed) 中，设置过程会创建带有一个工作区（即默认工作区）的 Anthropic 组织。要创建更多工作区，请[打开 Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#using-the-claude-console) 并前往**设置 > 工作区**。在同一页面上，您可以重命名或归档除默认工作区之外的任何工作区。每个工作区都有自己的[工作区 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#find-your-workspace-id)。文件、技能、批处理、Claude Managed Agents 会话和其他工作区资源属于您[为请求设置](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)的 ID 所对应的工作区。

您在项目上授予 [IAM 角色](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#iam-roles)，因此一个角色适用于该项目中的每个工作区。当团队需要独立的访问权限时，请为每个团队提供各自的 Google Cloud 项目。每个项目都有自己的 Anthropic 组织，因此请在每个项目中从第 2 步开始重复[设置步骤](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#set-up-your-account)。

## 进行 API 调用之前

要调用 Claude Models on Google Cloud (Anthropic managed)，您需要具备以下条件：

1. 一个拥有有效订阅且 Anthropic 组织已完成设置的 Google Cloud 项目。请参阅[设置您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#set-up-your-account)。
2. 已安装 [gcloud CLI](https://cloud.google.com/sdk/docs/install)，用于本地开发。
3. 在项目上授予进行调用的身份的 `roles/claude.user` 角色。
4. 该身份的 Application Default Credentials。请参阅[身份验证](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#authentication)。
5. 您的项目 ID 和[工作区 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#find-your-workspace-id)。

要为本地开发向您自己的账户授予该角色，请运行以下命令。您需要拥有更改项目 IAM 策略的权限。

```bash CLI
gcloud projects add-iam-policy-binding my-project-id \
  --member="user:you@example.com" \
  --role="roles/claude.user"
```

将 `my-project-id` 替换为您的项目 ID。将 `you@example.com` 替换为您的 Google 账户电子邮件地址。要查看 gcloud 正在使用的项目，请运行 `gcloud config get-value project`。关于服务账号和其他角色，请参阅 [IAM 角色](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#iam-roles)。

### 找到您的工作区 ID

要找到 Claude Models on Google Cloud (Anthropic managed) 的工作区 ID，请[打开 Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#using-the-claude-console) 并前往**设置 > 工作区**。该 ID 以 `wrkspc_` 开头。如果您无法打开 Claude Console，请让能够打开它的人将 ID 发送给您。

将项目和工作区设置为环境变量，以便 SDK 客户端和 [`ant` CLI](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart) 能够读取它们：

```bash CLI
export ANTHROPIC_GOOGLE_CLOUD_PROJECT="my-project-id"
export ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID="wrkspc_01AbCdEf23GhIj"
```

### 安装 SDK

Claude Models on Google Cloud (Anthropic managed) 的 [SDK 客户端](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview)是 `AnthropicGoogleCloud` (csharp: `AnthropicGoogleCloudClient`; go: `googlecloud.NewClient()`; java: `GoogleCloudBackend`; php: `Anthropic\GoogleCloud\Client`; ruby: `Anthropic::GoogleCloudClient`)。该客户端会解析您的 Google 凭据，还会根据您的项目、位置和工作区 ID 构建基础 URL。

<Note>
  Claude Models on Google Cloud (Anthropic managed) 的 SDK 客户端目前处于 beta 阶段。
</Note>

安装提供该客户端的软件包：

<Tabs>
  <Tab title="Python">
    ```bash
    pip install -U "anthropic[google_cloud]"
    ```
  </Tab>

  <Tab title="TypeScript">
    ```bash
    npm install @anthropic-ai/google-cloud-sdk
    ```
  </Tab>

  <Tab title="C#">
    ```bash
    dotnet add package Anthropic.GoogleCloud
    ```
  </Tab>

  <Tab title="Go">
    ```bash
    go get github.com/anthropics/anthropic-sdk-go/v2/googlecloud
    ```
  </Tab>

  <Tab title="Java">
    <Tabs>
      <Tab title="Gradle">
        ```kotlin
        implementation("com.anthropic:anthropic-java:3.8.0")
        implementation("com.anthropic:anthropic-java-google-cloud:3.8.0")
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
            <artifactId>anthropic-java-google-cloud</artifactId>
            <version>3.8.0</version>
        </dependency>
        ```
      </Tab>
    </Tabs>
  </Tab>

  <Tab title="PHP">
    ```bash
    composer require anthropic-ai/sdk google/auth
    ```
  </Tab>

  <Tab title="Ruby">
    ```bash
    gem install anthropic googleauth
    ```
  </Tab>
</Tabs>

如果要改用 `ant` CLI，请[安装或更新 CLI](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/quickstart#installation)到最新版本。不支持 Claude Models on Google Cloud (Anthropic managed) 的版本会忽略 `ANTHROPIC_GOOGLE_CLOUD_*` 变量，然后执行以下操作之一：

* 使用您设置的任何其他凭据将请求发送到 Claude API
* 因身份验证错误而失败

## 发出请求

Claude Models on Google Cloud (Anthropic managed) 在一个包含您的项目、位置和工作区的基础 URL 下提供 Claude API 端点：

```text wrap
https://claude.googleapis.com/v1/projects/{project}/locations/global/workspaces/{workspace}/invoke
```

[组织级路由](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#iam-roles)（其路径以 `/v1/organizations/` 开头）以及 `/v1/user_profiles` 下的[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)路由使用不含工作区的基础 URL：

```text wrap
https://claude.googleapis.com/v1/projects/{project}/locations/global/claudeOrganization/invoke
```

如果您没有[设置基础 URL](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)，SDK 客户端会为每个请求在这两者之间进行选择。

将 Claude API 路径附加到基础 URL 之后。例如，[Messages API](https://platform.claude.com/docs/zh-CN/api/messages/create) 位于 `.../invoke/v1/messages`。与 Claude API 请求相比，有三处不同：

* **基础 URL：** 使用前面所示的项目范围 URL，而不是 `https://api.anthropic.com`。
* **身份验证：** 使用带有 Google 访问令牌的 `Authorization: Bearer` 标头。不要发送 `x-api-key` 标头。
* **工作区：** 工作区 ID 是 URL 的一部分。不要发送 `anthropic-workspace-id` 标头。

其他一切都与 Claude API 相同，包括以下内容：

* `anthropic-version` 和 `anthropic-beta` 标头
* 请求正文
* 流式传输
* 响应

本文档其他地方的代码示例会创建 Claude API 客户端并读取 API 密钥。要在 Claude Models on Google Cloud (Anthropic managed) 上运行这些示例，请将其导入语句和客户端构造函数替换为以下示例中的内容。对于 cURL 示例，请应用前面列出的三处不同。

在运行这些示例之前，请完成[进行 API 调用之前](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#before-making-api-calls)中的步骤。

<CodeGroup>
  ```bash cURL
  curl "https://claude.googleapis.com/v1/projects/$ANTHROPIC_GOOGLE_CLOUD_PROJECT/locations/global/workspaces/$ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID/invoke/v1/messages" \
    -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
    -H "content-type: application/json" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "model": "claude-opus-5-5",
      "max_tokens": 1024,
      "messages": [
        {"role": "user", "content": "Hello, Claude"}
      ]
    }'
  ```

  ```bash CLI
  ant messages create \
    --model claude-opus-5-5 \
    --max-tokens 1024 \
    --message '{role: user, content: "Hello, Claude"}'
  ```

  ```python Python
  from anthropic import AnthropicGoogleCloud

  client = AnthropicGoogleCloud()

  message = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )
  print(message)
  ```

  ```typescript TypeScript
  import { AnthropicGoogleCloud } from "@anthropic-ai/google-cloud-sdk";

  const client = new AnthropicGoogleCloud();

  const message = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  });
  console.log(message);
  ```

  ```csharp C#
  using Anthropic.GoogleCloud;
  using Anthropic.Models.Messages;

  var client = new AnthropicGoogleCloudClient();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }]
  };

  var message = await client.Messages.Create(parameters);
  Console.WriteLine(message);
  ```

  ```go Go
  import (
  	"context"
  	"fmt"

  	"github.com/anthropics/anthropic-sdk-go/v2"
  	"github.com/anthropics/anthropic-sdk-go/v2/googlecloud"
  )

  func main() {
  	client, err := googlecloud.NewClient(context.Background(), googlecloud.ClientConfig{})
  	if err != nil {
  		panic(err)
  	}

  	message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 1024,
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  		},
  	})
  	if err != nil {
  		panic(err)
  	}

  	fmt.Println(message)
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.googlecloud.backends.GoogleCloudBackend;
  import com.anthropic.models.messages.Message;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.builder()
          .backend(GoogleCloudBackend.fromEnv())
          .build();

      Message message = client.messages().create(
          MessageCreateParams.builder()
              .model(Model.CLAUDE_OPUS_5_5)
              .maxTokens(1024)
              .addUserMessage("Hello, Claude")
              .build()
      );

      IO.println(message);
  }
  ```

  ```php PHP
  use Anthropic\GoogleCloud\Client;
  use Anthropic\Messages\Model;

  $client = new Client();

  $message = $client->messages->create(
      model: Model::CLAUDE_OPUS_5_5,
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
  );

  echo $message;
  ```

  ```ruby Ruby
  require "anthropic"

  client = Anthropic::GoogleCloudClient.new

  message = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  )

  puts message
  ```
</CodeGroup>

cURL 示例会向 gcloud 请求一个 Application Default Credentials 令牌。访问令牌的有效期很短。请为每个会话生成一个令牌，而不是存储它。

### API 模型 ID

Claude Models on Google Cloud (Anthropic managed) 使用与 Claude API 相同的模型 ID，例如 `claude-opus-5-5`。关于模型及其 ID，请参阅[模型概览](https://platform.claude.com/docs/zh-CN/models/overview)。

在 Claude Models on Google Cloud (Anthropic managed) 中，请求您的 Anthropic 组织无法使用的模型会返回错误。错误消息为 `Model {model} is not available for this organization's inference configuration.`

## 项目、位置和工作区解析

Claude Models on Google Cloud (Anthropic managed) 的客户端接受下表中的设置。创建客户端时传入的值优先于环境变量。要显式设置项目和工作区 ID，请传入 `project` (go: `Project`; java: `.project()`; csharp: `GoogleCloudClientOptions.Project`; cli: `--google-cloud-project`) 和 `workspace_id` (typescript, php: `workspaceId`; go: `WorkspaceID`; java: `.workspaceId()`; csharp: `GoogleCloudClientOptions.WorkspaceId`; cli: `--google-cloud-workspace-id`)。

| 设置     | 环境变量                                                        | 未设置时                                                                                                                                                                                                                                         |
| ------ | ----------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 项目 ID  | `ANTHROPIC_GOOGLE_CLOUD_PROJECT`，然后是 `GOOGLE_CLOUD_PROJECT` | Application Default Credentials 报告的项目（当凭据类型包含项目时）                                                                                                                                                                                            |
| 工作区 ID | `ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID`                       | 除非您设置了基础 URL，否则客户端会引发错误。[组织级路由](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#iam-roles)和[用户配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/user-profiles)路由不需要工作区 ID。 |
| 位置     | `ANTHROPIC_GOOGLE_CLOUD_LOCATION`                           | `global`                                                                                                                                                                                                                                     |
| 基础 URL | `ANTHROPIC_GOOGLE_CLOUD_BASE_URL`                           | 根据项目、位置和工作区 ID 派生                                                                                                                                                                                                                            |

请将位置保留为 `global`。位置与推理运行的地点无关，因此如需仅在美国进行推理，请在每个请求上设置 `inference_geo: "us"`，如[数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency#inference-geo)中所述。

`ant` CLI 以标志的形式接受相同的设置：

* **项目 ID：** `--google-cloud-project`
* **工作区 ID：** `--google-cloud-workspace-id`
* **位置：** `--google-cloud-location`
* **基础 URL：** `--google-cloud-base-url`

当设置了项目、工作区 ID 或位置时，CLI 会使用 Claude Models on Google Cloud (Anthropic managed)。仅设置 `GOOGLE_CLOUD_PROJECT`、基础 URL 或两者是不够的。

## 身份验证

发往 Claude Models on Google Cloud (Anthropic managed) 的每个请求都在 `Authorization: Bearer` 标头中携带一个 Google OAuth 2.0 访问令牌。该令牌具有 `https://www.googleapis.com/auth/cloud-platform` 范围。

Google Cloud 负责处理访问。其位于 `claude.googleapis.com` 的网关会对每个请求执行以下操作：

1. 检查您的 Google Cloud 凭据。
2. 检查调用方是否拥有请求所需的 IAM 权限。如果没有，Google Cloud 会返回 HTTP 403 错误。
3. 将请求转发给 Anthropic。Anthropic 会验证该请求。

要授予权限，请参阅 [IAM 角色](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#iam-roles)。

SDK 客户端从 [Application Default Credentials](https://cloud.google.com/docs/authentication/application-default-credentials)（ADC）获取令牌。ADC 支持的任何凭据来源都可以使用：

* **本地开发：** 运行 `gcloud auth application-default login`。
* **在 Google Cloud 上：** 客户端会自动使用附加到工作负载的服务账号，例如在 Compute Engine、GKE 或 Cloud Run 上。
* **服务账号密钥文件：** 将 `GOOGLE_APPLICATION_CREDENTIALS` 设置为其路径。
* **工作负载身份联合和服务账号模拟：** 按照配置任何 Google Cloud API 的方式进行配置。

检查 ADC 是否正常工作：

```bash CLI
gcloud auth application-default print-access-token
```

Claude Models on Google Cloud (Anthropic managed) 不使用 API 密钥。在 Claude Console 中，您无法为设置过程创建的 Anthropic 组织创建 API 密钥。Claude Models on Google Cloud (Anthropic managed) 的 SDK 客户端会忽略以下环境变量：

* `ANTHROPIC_API_KEY`
* `ANTHROPIC_AUTH_TOKEN`
* `ANTHROPIC_BASE_URL`

### 凭据优先级

Claude Models on Google Cloud (Anthropic managed) 的 SDK 客户端按以下顺序检查凭据，并使用找到的第一个：

1. 创建客户端时以 `token_provider` (typescript: `bearerTokenProvider`; csharp: `GoogleCloudClientOptions.TokenProvider`; go: `TokenSource`; java: `.bearerTokenSupplier()`; php: `tokenProvider`) 传入的令牌提供程序。它是一个返回访问令牌的函数或令牌源。客户端会在每个请求时向它请求令牌。令牌的缓存和刷新由您负责。
2. 创建客户端时以 `credentials` (typescript: `googleAuth`; csharp: `GoogleCloudClientOptions.GoogleCredential`; java: `.googleCredentials()`; php: `googleCredentials`; ruby: `google_credentials`) 传入的 Google 凭据对象。Go SDK 没有此选项。
3. Application Default Credentials。

Go 和 Java 与其他 SDK 有所不同：

* **Go：** 对于选项 1，客户端接受一个 `oauth2.TokenSource`。它会缓存令牌直到令牌过期。
* **Java：** 使用 `GoogleCloudBackend.fromEnv()` 读取 Application Default Credentials 以及[项目、位置和工作区解析](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)中的环境变量。

客户端没有用于凭据的专属环境变量。请改用 ADC 的 `GOOGLE_APPLICATION_CREDENTIALS`。

如果您的请求经过一个会为您附加令牌的代理，请设置 `skip_auth` (typescript, php: `skipAuth`; go: `SkipAuth`; java: `.skipAuth()`; csharp: `GoogleCloudClientOptions.SkipAuth`)。这样客户端就不会发送 `Authorization` 标头。此设置不能与前面的凭据选项组合使用。

当 `ant` CLI 使用 Claude Models on Google Cloud (Anthropic managed) 时，它始终从 Application Default Credentials 获取令牌。它不接受令牌提供程序、凭据对象或跳过身份验证的设置。

在将 `ant` 与 Claude Models on Google Cloud (Anthropic managed) 一起使用之前，请确保以下各项均未设置：

* API 密钥

* 身份验证令牌

* `--profile` 或 `ANTHROPIC_PROFILE`（如果它指定了 CLI 可用于身份验证的[配置文件](https://platform.claude.com/docs/zh-CN/manage-claude/wif-reference#profile-configuration-file)）

* Anthropic 的任何联合身份变量：

  * `ANTHROPIC_FEDERATION_RULE_ID`
  * `ANTHROPIC_ORGANIZATION_ID`
  * `ANTHROPIC_SERVICE_ACCOUNT_ID`
  * `ANTHROPIC_IDENTITY_TOKEN`
  * `ANTHROPIC_IDENTITY_TOKEN_FILE`

如果设置了其中任何一项，CLI 会使用该凭据，而不是[找到您的工作区 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#find-your-workspace-id) 中的 Google Cloud 设置。如果只设置了部分联合身份变量，CLI 会报错并停止。当设置了多个凭据时，CLI 会打印一条说明，指出它使用的是哪一个。

此外，请不要设置 `ANTHROPIC_WORKSPACE_ID` 和 `--workspace-id` 标志。CLI 仍会将该值作为 `anthropic-workspace-id` 标头发送，而发往 Claude Models on Google Cloud (Anthropic managed) 的请求不得携带该标头。请改用 `--google-cloud-workspace-id` 或 `ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID` 设置工作区。

与 SDK 客户端一样，CLI 不会将 Claude Models on Google Cloud (Anthropic managed) 的请求发送到 `--base-url` 或 `ANTHROPIC_BASE_URL`。

## 功能支持

本节列出了在 Claude Models on Google Cloud (Anthropic managed) 上可用的功能以及不受支持的功能。它并未列出所有 Claude API 功能，也不说明未列出的功能是否可用。

Models API 以及以下功能可在 Claude Models on Google Cloud (Anthropic managed) 上使用：

* [Advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool) (Beta)
* [Agent Skills](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview)
  * [自定义技能](https://platform.claude.com/docs/zh-CN/build-with-claude/skills-guide#managing-custom-skills)
  * [预构建 Agent Skills](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/overview#pre-built-agent-skills)
* [批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)
* [分类器拒绝](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback)
  * [客户端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#client-side-fallback) (Beta)
  * [服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback) (Beta)
* [代码执行工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)
* [自定义工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/define-tools)
* [数据驻留](https://platform.claude.com/docs/zh-CN/manage-claude/data-residency)
* [Effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)
  * [按消息设置 effort](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message) (Beta)
* [快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)
* [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files)
* [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector) (Beta)
  * [客户端 MCP 辅助工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector#client-side-mcp-helpers) (Beta)
* [记忆工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool)
  * [记忆工具辅助工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/memory-tool#implement-the-memory-handler) (Beta)
* [消息流式传输](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)
* [消息线程](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads) (Beta)
* [对话中途的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)
  * [对话中途的工具变更](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#mid-conversation-tool-changes) (Beta)
  * [轮次范围的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages#turn-scoped-system-messages) (Beta)
* [OpenTelemetry](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry) (Beta)
* [PDF 支持](https://platform.claude.com/docs/zh-CN/build-with-claude/pdf-support)
* [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)
  * [1 小时有效期](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#1-hour-cache-duration)
  * [5 分钟有效期（默认）](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#how-prompt-caching-works)
  * [自动缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#automatic-caching)
* [SDK 中间件](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/middleware)
* [结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs)
  * [类型化结构化输出辅助工具](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#usage)
* [思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)
  * [自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking-steering-and-cost)
  * [扩展思考](https://platform.claude.com/docs/zh-CN/build-with-claude/extended-thinking)
  * [保留思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)
* [令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)
* [工具运行器](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-runner) (Beta)
* [视觉（图像）](https://platform.claude.com/docs/zh-CN/build-with-claude/vision)
* [Web fetch 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool)
* [网页搜索工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool)

如果某项 beta 功能在 Claude API 上仅限特定组织使用，则设置过程中创建的 Anthropic 组织也需要具备相同的访问权限。

要为设置过程创建的 Anthropic 组织开启或关闭 [Compliance API](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api)，请先[打开 Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#using-the-claude-console)。然后前往 [Claude Console 中的合规页面](https://platform.claude.com/settings/security)（`platform.claude.com/settings/security`），并使用 **Compliance API** 开关。

“Customer-managed encryption keys”（客户管理的加密密钥），即 CMEK，可与 Google Cloud KMS 密钥一起使用。要为设置过程创建的 Anthropic 组织激活 CMEK，请参阅[谁可以使用 CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek#availability)。要设置密钥，请按照[使用 Google Cloud KMS 设置 CMEK](https://platform.claude.com/docs/zh-CN/manage-claude/cmek-google-cloud-kms) 操作。然后按照设置页面的 **Claude Platform** 选项卡中的说明，在 [Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#using-the-claude-console) 中注册、验证并附加密钥。

### Claude Managed Agents

[Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 可通过 Claude Models on Google Cloud (Anthropic managed) 使用，包括 [sessions](https://platform.claude.com/docs/zh-CN/managed-agents/sessions)（会话）以及以下功能：

* [智能体配置](https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup) (Beta)
* [云沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/environments) (Beta)
* [事件增量](https://platform.claude.com/docs/zh-CN/managed-agents/event-deltas) (Beta)
* [GitHub 访问](https://platform.claude.com/docs/zh-CN/managed-agents/github) (Beta)
* [记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/memory) (Beta)
* [多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration) (Beta)
  * [动态工作流](https://platform.claude.com/docs/zh-CN/managed-agents/workflow-runs) (Beta)
  * [会话线程](https://platform.claude.com/docs/zh-CN/managed-agents/session-threads) (Beta)
* [结果](https://platform.claude.com/docs/zh-CN/managed-agents/define-outcomes) (Beta)
* [权限策略](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies) (Beta)
* [计划部署](https://platform.claude.com/docs/zh-CN/managed-agents/scheduled-deployments) (Beta)
* [自托管沙箱](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes) (Beta)
* [会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets) (Beta)
* [保管库](https://platform.claude.com/docs/zh-CN/managed-agents/vaults) (Beta)
* [网页域名限制](https://platform.claude.com/docs/zh-CN/managed-agents/tools-web-restrictions) (Beta)

访问权限在创建会话或部署时检查，而不是在其运行期间检查：

* **正在运行的会话会继续运行。** 如果您撤销了创建会话的身份的 Google Cloud 访问权限，会话不会停止。要停止它，请单独向其发送一个 [`user.interrupt` 事件](https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming#integrating-events)。会话变为 `idle` 后，您可以将其归档或删除。
* **计划部署会继续运行。** 部署不会在每次计划运行时重新检查其创建者的访问权限。要停止它，请暂停、更新或归档它。

当您移除用户或轮换服务账号时，请检查该身份创建的会话和部署。

### 不支持的功能

Claude Models on Google Cloud (Anthropic managed) 不支持以下内容：

* **API 密钥：** 仅通过 Google Cloud 凭据进行身份验证。
* **MCP 隧道：** [MCP 隧道](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-tunnels/overview)在 Claude Models on Google Cloud (Anthropic managed) 上不可用。
* **[自托管环境上的记忆存储](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-memory)：** 自托管环境上的会话不支持记忆存储。
* **预构建的自托管沙箱工作器：** `ant` CLI 以及 Python、TypeScript 和 Go SDK 中的[自托管工作器](https://platform.claude.com/docs/zh-CN/managed-agents/self-hosted-sandboxes-workers)不支持 Claude Models on Google Cloud (Anthropic managed)。请改为实现您自己的工作器，直接调用[环境工作端点](https://platform.claude.com/docs/zh-CN/api/beta/environments/work)。有关您的工作器所需的角色和权限，请参阅[环境的 IAM 权限](https://platform.claude.com/docs/zh-CN/api/claude-models-on-google-cloud-anthropic-managed-iam-permissions#environments)。

### 上下文窗口

Claude Models on Google Cloud (Anthropic managed) 上的"context window"（上下文窗口）大小与 Claude API 上相同。关于每个模型的限制，请参阅[上下文窗口](https://platform.claude.com/docs/zh-CN/build-with-claude/context-windows)。

## 计费、速率限制和日志记录

### 速率限制

Anthropic 管理 Claude Models on Google Cloud (Anthropic managed) 的"rate limit"（速率限制）。要申请更高的限制，请联系您的 Anthropic 客户代表或[联系销售](https://claude.com/contact-sales)。关于速率限制的衡量和报告方式，请参阅[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits)。

### 计费

Claude Models on Google Cloud (Anthropic managed) 通过 Google Cloud Marketplace 计费。Anthropic 会计量您的用量，并以 Claude Consumption Units（Claude 消费单位）向 Google Cloud 报告用量。一个 Claude Consumption Unit 为 0.01 美元。费用显示在您的 Google Cloud 账单上。Anthropic 不会另行开具发票。

在 [Claude Console](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#using-the-claude-console) 的**用量**和**费用**页面上查看使用量和费用数据，或使用[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 查询这些数据。

如果您有 Google Cloud 支出承诺，使用量会像其他 Marketplace 购买一样计入该承诺。

### 数据保留

Anthropic 是 Claude Models on Google Cloud (Anthropic managed) 的数据处理者。一些支持服务（例如 Anthropic 的信任与安全系统）会在 Google Cloud 之外处理数据。

Claude Models on Google Cloud (Anthropic managed) 遵循与 Claude API 相同的数据保留政策。请参阅 [API 和数据保留](https://platform.claude.com/docs/zh-CN/manage-claude/api-and-data-retention)。"Zero Data Retention"（零数据保留），即 ZDR，可应要求提供。请联系您的 Anthropic 客户代表。

其他 Anthropic 组织上的 ZDR 安排不会延续到您用于 Claude Models on Google Cloud (Anthropic managed) 的组织。

### 监控和日志记录

[Cloud Audit Logs](https://cloud.google.com/logging/docs/audit) 会记录您项目中 Claude Models on Google Cloud (Anthropic managed) 的 API 活动。这些条目显示在 `claude.googleapis.com` 服务下。每个 API 路由属于以下两种日志类型之一：

* **数据访问日志：** 涵盖大多数路由，包括消息、批处理、文件、技能、模型以及大多数 Claude Managed Agents 资源。默认处于关闭状态。
* **管理员活动日志：** 涵盖其他路由，包括加密密钥路由和大多数凭据保管库路由。例外是 `vaults.verify` 的路由，它属于数据访问。管理员活动日志始终处于开启状态。

要保留请求记录，请在项目的审计日志设置中为 `claude.googleapis.com` 开启数据访问日志。

关于各路由的审计日志类型，请参阅 [Claude Models on Google Cloud (Anthropic managed) 的 IAM 权限](https://platform.claude.com/docs/zh-CN/api/claude-models-on-google-cloud-anthropic-managed-iam-permissions)。

#### 请求 ID

来自 Claude Models on Google Cloud (Anthropic managed) 的成功响应会携带 [`request-id`](https://platform.claude.com/docs/zh-CN/api/errors#request-id) 标头。

## 从现有 Anthropic 组织迁移

您无法将现有的 Anthropic 组织转换为 Claude Models on Google Cloud (Anthropic managed) 的组织，包括您直接用于 Claude API 的组织。

请将迁移规划为切换到新组织：

* **先创建新组织。** 按照[设置您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#set-up-your-account)操作。
* **重新创建访问权限和配置。** API 密钥、工作区和 Claude Console 设置不会延续。请在项目上授予 [IAM 角色](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#iam-roles)，而不是创建 API 密钥。要重新创建工作区，请参阅[工作区](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#workspaces)。关于 ZDR 安排，请参阅[数据保留](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#data-retention)。
* **规划已存储的资源。** 文件、技能、批处理和 Claude Managed Agents 资源会保留在现有组织中。
* **更新您的集成。** 关于哪些内容会改变、哪些保持不变，请参阅[发出请求](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#making-requests)。关于不受支持的功能，请参阅[不支持的功能](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#features-not-supported)。
* **按您自己的时间表进行切换。** 两个组织相互独立，可以并行处理流量。您可以逐步迁移工作负载。现有组织保持其当前的计费方式。在转移生产流量之前，请检查 [Claude Models on Google Cloud (Anthropic managed) 的速率限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#rate-limits)。

## 从 Claude Models on Google Cloud (Google managed) 迁移

要迁移到 Claude Models on Google Cloud (Anthropic managed)，请完成以下步骤。在迁移依赖 ZDR 安排的工作负载之前，请参阅[数据保留](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#data-retention)。有关迁移时会发生哪些变化，请参阅[两种 Google Cloud 产品的区别](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/overview#how-the-two-google-cloud-offerings-differ)。

<Steps>
  <Step title="设置您的账户">
    完成[设置您的账户](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#set-up-your-account)中的每个步骤。
  </Step>

  <Step title="设置访问权限">
    向每个调用 Claude Models on Google Cloud (Anthropic managed) 的身份授予 `roles/claude.user`。将您的项目和工作区 ID 设置为环境变量。有关这两项操作，请参阅[在进行 API 调用之前](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#before-making-api-calls)。
  </Step>

  <Step title="安装 SDK 包">
    安装为 Claude Models on Google Cloud (Anthropic managed) 提供客户端的包。请参阅[安装 SDK](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#install-an-sdk)。
  </Step>

  <Step title="替换导入和客户端">
    修改导入和创建客户端的代码。使用 `AnthropicGoogleCloud` (csharp: `AnthropicGoogleCloudClient`; go: `googlecloud.NewClient()`; java: `GoogleCloudBackend`; php: `Anthropic\GoogleCloud\Client`; ruby: `Anthropic::GoogleCloudClient`) 代替 `AnthropicVertex` (csharp: `AnthropicVertexClient`; go: `vertex.WithGoogleAuth()`; java: `VertexBackend`; php: `Anthropic\Vertex\Client`; ruby: `Anthropic::VertexClient`)。在 cURL 请求中，修改 URL、请求头和请求体。您当前的代码会为 Claude Models on Google Cloud (Google managed) 创建客户端。例如，以下代码使用全局端点创建该客户端：

    <CodeGroup exclude="shell:CLI">
      ```bash cURL
      curl https://aiplatform.googleapis.com/v1/projects/MY_PROJECT_ID/locations/global/publishers/anthropic/models/claude-opus-5-5:rawPredict \
        -H "Authorization: Bearer $(gcloud auth print-access-token)" \
        -H "Content-Type: application/json" \
        -d '{
          "anthropic_version": "2026-10-21",
          "messages": [{"role": "user", "content": "Hey Claude!"}],
          "max_tokens": 100
        }'
      ```

      <CodeGroupItem language="CLI">
        `ant` CLI 不支持 Claude Models on Google Cloud (Google managed)。
      </CodeGroupItem>

      ```python Python
      from anthropic import AnthropicVertex

      project_id = "MY_PROJECT_ID"
      region = "global"

      client = AnthropicVertex(project_id=project_id, region=region)
      ```

      ```typescript TypeScript
      import { AnthropicVertex } from "@anthropic-ai/vertex-sdk";

      const projectId = "MY_PROJECT_ID";
      const region = "global";

      const client = new AnthropicVertex({
        projectId,
        region
      });
      ```

      ```csharp C#
      using Anthropic.Vertex;

      var projectId = "MY_PROJECT_ID";
      var region = "global";

      var client = new AnthropicVertexClient(new AnthropicVertexCredentials(region, projectId));
      ```

      ```go Go
      import (
      	"context"

      	"github.com/anthropics/anthropic-sdk-go/v2"
      	"github.com/anthropics/anthropic-sdk-go/v2/vertex"
      )
      // ...
      	// 使用默认的 Google Cloud 凭据
      	client := anthropic.NewClient(
      		vertex.WithGoogleAuth(context.Background(), "global", "MY_PROJECT_ID"),
      	)
      ```

      ```java Java
      import com.anthropic.client.AnthropicClient;
      import com.anthropic.client.okhttp.AnthropicOkHttpClient;
      import com.anthropic.vertex.backends.VertexBackend;
      import com.google.auth.oauth2.GoogleCredentials;

      void main() throws Exception {
          AnthropicClient client = AnthropicOkHttpClient.builder()
              .backend(
                  VertexBackend.builder()
                      .googleCredentials(GoogleCredentials.getApplicationDefault())
                      .region("global")
                      .project("MY_PROJECT_ID")
                      .build()
              )
              .build();
      }
      ```

      ```php PHP
      use Anthropic\Vertex;

      $client = Vertex\Client::fromEnvironment(
          region: 'global',
          projectId: 'MY_PROJECT_ID',
      );
      ```

      ```ruby Ruby
      require "anthropic"

      client = Anthropic::VertexClient.new(
        region: "global",
        project_id: "MY_PROJECT_ID"
      )
      ```
    </CodeGroup>

    以下代码为 Claude Models on Google Cloud (Anthropic managed) 创建客户端：

    <CodeGroup>
      ```bash cURL
      curl "https://claude.googleapis.com/v1/projects/$ANTHROPIC_GOOGLE_CLOUD_PROJECT/locations/global/workspaces/$ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID/invoke/v1/messages" \
        -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
        -H "content-type: application/json" \
        -H "anthropic-version: 2026-10-21" \
        -d '{
          "model": "claude-opus-5-5",
          "max_tokens": 1024,
          "messages": [
            {"role": "user", "content": "Hello, Claude"}
          ]
        }'
      ```

      ```bash CLI
      ant messages create \
        --model claude-opus-5-5 \
        --max-tokens 1024 \
        --message '{role: user, content: "Hello, Claude"}'
      ```

      ```python Python
      from anthropic import AnthropicGoogleCloud

      client = AnthropicGoogleCloud()
      ```

      ```typescript TypeScript
      import { AnthropicGoogleCloud } from "@anthropic-ai/google-cloud-sdk";

      const client = new AnthropicGoogleCloud();
      ```

      ```csharp C#
      using Anthropic.GoogleCloud;

      var client = new AnthropicGoogleCloudClient();
      ```

      ```go Go
      import (
      	"context"

      	"github.com/anthropics/anthropic-sdk-go/v2"
      	"github.com/anthropics/anthropic-sdk-go/v2/googlecloud"
      )
      // ...
      	client, err := googlecloud.NewClient(context.Background(), googlecloud.ClientConfig{})
      	if err != nil {
      		panic(err)
      	}
      ```

      ```java Java
      import com.anthropic.client.AnthropicClient;
      import com.anthropic.client.okhttp.AnthropicOkHttpClient;
      import com.anthropic.googlecloud.backends.GoogleCloudBackend;

      void main() {
          AnthropicClient client = AnthropicOkHttpClient.builder()
              .backend(GoogleCloudBackend.fromEnv())
              .build();
      }
      ```

      ```php PHP
      use Anthropic\GoogleCloud\Client;

      $client = new Client();
      ```

      ```ruby Ruby
      require "anthropic"

      client = Anthropic::GoogleCloudClient.new
      ```
    </CodeGroup>

    新客户端会从您在[查找您的工作区 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#find-your-workspace-id) 中设置的环境变量读取项目和工作区 ID。创建客户端时无需传入它们。除了下一步中的模型 ID 和功能之外，您的 Messages API 调用保持不变。
  </Step>

  <Step title="检查模型 ID 和功能">
    * **模型 ID：** 如果您使用的是早于 Claude 4.6 的模型，请将其 ID 中快照日期前的 `@` 替换为连字符。例如，`claude-haiku-4-5@20251001` 变为 `claude-haiku-4-5-20251001`。有关模型及其 ID，请参阅 [API 模型 ID](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#api-model-ids)。
    * **功能：** 针对您的代码使用的每项功能，查看[功能支持](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#feature-support)。
    * **位置：** 如果您目前使用多区域或区域端点，请检查 Claude Models on Google Cloud (Anthropic managed) 使用哪个位置。请参阅[项目、位置和工作区解析](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)。
  </Step>

  <Step title="运行测试请求">
    运行[发出请求](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#making-requests)中的请求，以检查您的设置是否正常工作。然后运行您自己的代码。如果调用因权限错误而失败，请参阅[账户设置故障排除](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#troubleshooting-account-setup)。在迁移生产流量之前，请查看 [Claude Models on Google Cloud (Anthropic managed) 的速率限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#rate-limits)。
  </Step>
</Steps>

## 支持

[Google Cloud 支持团队](https://cloud.google.com/support)负责处理您的 Google Cloud 账户、访问权限和计费。[Anthropic 支持团队](https://support.claude.com/en/articles/9015913-how-to-get-support)负责处理 Claude 本身的问题，包括模型、API 错误和速率限制。如需申请更高的速率限制，请参阅[速率限制](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#rate-limits)。

## 后续步骤

<CardGroup cols={2}>
  <Card title="功能概览" icon="stack" href="https://platform.claude.com/docs/zh-CN/build-with-claude/overview">
    探索 Claude 的功能。
  </Card>

  <Card title="IAM 权限" icon="lock" href="https://platform.claude.com/docs/zh-CN/api/claude-models-on-google-cloud-anthropic-managed-iam-permissions">
    查找权限、角色和审计日志类型。
  </Card>

  <Card title="模型弃用" icon="arrow-clockwise" href="https://platform.claude.com/docs/zh-CN/about-claude/model-deprecations">
    随着更安全、更强大的模型推出，Anthropic 会定期停用较旧的模型。查看所有 API 弃用信息以及推荐的替代模型。
  </Card>

  <Card title="Google Cloud Marketplace" icon="coins" href="https://console.cloud.google.com/marketplace">
    管理您的订阅和账单。
  </Card>
</CardGroup>
