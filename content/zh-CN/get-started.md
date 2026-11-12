---
title: Claude 快速入门
url: https://platform.claude.com/docs/zh-CN/get-started
description: 向 Claude 发起您的第一次 API 调用，并构建一个简单的网页搜索助手。
---

## 开始之前

<Steps continues>
  <Step title="创建 Claude Console 账户">
    [注册](https://platform.claude.com/) Claude Console 账户，或登录您已有的账户。
  </Step>

  <Step title="选择身份验证方式">
    对于本地开发，建议使用 CLI 登录。

    Follow the steps for one of the following options.
  </Step>
</Steps>

**CLI**

<Steps continued>
  <Step title="安装 CLI" stepNumber={3}>
    <Tabs noDeepLinks>
      <Tab title="Homebrew (macOS)">
        ```bash
        brew install anthropics/tap/ant
        ```
      </Tab>

      <Tab title="curl (Linux/WSL)">
        对于 Linux 环境，请直接下载发布版二进制文件。

        ```bash
        VERSION=2.9.0
        OS=$(uname -s | tr '[:upper:]' '[:lower:]')
        case $(uname -m) in
          x86_64) ARCH=amd64 ;;
          aarch64) ARCH=arm64 ;;
        esac
        curl -fsSL "https://github.com/anthropics/anthropic-cli/releases/download/v${VERSION}/ant_${VERSION}_${OS}_${ARCH}.tar.gz" \
          | sudo tar -xz -C /usr/local/bin ant
        ```

        您可以在 [GitHub 发布页面](https://github.com/anthropics/anthropic-cli/releases)上找到所有发布版本。
      </Tab>

      <Tab title="go install">
        您也可以使用 `go install` 从源代码安装 CLI，这需要 Go 1.26.10 或更高版本。

        ```bash
        go install github.com/anthropics/anthropic-cli/v2/cmd/ant@latest
        ```

        `go install` 会将二进制文件放置在 `$(go env GOPATH)/bin` 中。如果该目录尚未添加到您的 `PATH` 中，请将其添加进去：

        ```bash
        export PATH="$PATH:$(go env GOPATH)/bin"
        ```
      </Tab>
    </Tabs>
  </Step>

  <Step title="登录" stepNumber={4}>
    ```bash
    ant auth login
    ```

    登录完成后，SDK 和 CLI 将自动获取您的凭据。
  </Step>
</Steps>

**API 密钥**

<Steps continued>
  <Step title="创建 API 密钥" stepNumber={3}>
    在 Claude Console 中创建 [API 密钥](https://platform.claude.com/settings/keys)。
  </Step>

  <Step title="设置环境变量" stepNumber={4}>
    SDK 和 CLI 从 `ANTHROPIC_API_KEY` 读取您的密钥：

    ```bash
    export ANTHROPIC_API_KEY="your-api-key-here"
    ```
  </Step>
</Steps>

## 调用 API

<Tabs>
  <Tab title="cURL">
    <Steps>
      <Step title="发起您的第一次 API 调用" stepNumber={5}>
        **CLI**

        向 Messages API 发送 `POST` 请求，并将[您通过 CLI 登录获得的访问令牌](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/cli/scripting#authenticate-curl-requests-with-cli-credentials)作为 bearer 令牌：

        ```bash cURL
        curl https://api.anthropic.com/v1/messages \
          -H "content-type: application/json" \
          -H "Authorization: Bearer $(ant auth print-credentials --access-token)" \
          -H "anthropic-version: 2026-10-21" \
          -d '{
            "model": "claude-opus-5-5",
            "max_tokens": 1000,
            "messages": [
              {
                "role": "user",
                "content": "What should I search for to find the latest developments in renewable energy?"
              }
            ]
          }'
        ```

        **API 密钥**

        向 Messages API 发送 `POST` 请求：

        ```bash cURL
        curl https://api.anthropic.com/v1/messages \
          -H "content-type: application/json" \
          -H "x-api-key: $ANTHROPIC_API_KEY" \
          -H "anthropic-version: 2026-10-21" \
          -d '{
            "model": "claude-opus-5-5",
            "max_tokens": 1000,
            "messages": [
              {
                "role": "user",
                "content": "What should I search for to find the latest developments in renewable energy?"
              }
            ]
          }'
        ```

        Claude 会返回一个包含助手消息的 JSON 响应：

        ```json Output
        {
          "model": "claude-opus-5-5",
          "id": "msg_013mHbppMPd2PrVJzGMZPt2D",
          "type": "message",
          "role": "assistant",
          "content": [
            {
              "type": "text",
              "text": "Here are some effective search strategies to find the latest developments in renewable energy:\n\n## General Search Terms\n- \"Renewable energy news 2025\"\n- ..."
            }
          ],
          "stop_reason": "end_turn",
          "stop_sequence": null,
          "stop_details": null,
          "usage": {
            "input_tokens": 21,
            "output_tokens": 305
          }
        }
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="CLI">
    <Steps>
      <Step title="发起您的第一次 API 调用" stepNumber={5}>
        **API 密钥**

        如果您之前选择了 API 密钥，请先安装 CLI：

        <Tabs noDeepLinks>
          <Tab title="Homebrew (macOS)">
            ```bash
            brew install anthropics/tap/ant
            ```
          </Tab>

          <Tab title="curl (Linux/WSL)">
            对于 Linux 环境，请直接下载发布版二进制文件。

            ```bash
            VERSION=2.9.0
            OS=$(uname -s | tr '[:upper:]' '[:lower:]')
            case $(uname -m) in
              x86_64) ARCH=amd64 ;;
              aarch64) ARCH=arm64 ;;
            esac
            curl -fsSL "https://github.com/anthropics/anthropic-cli/releases/download/v${VERSION}/ant_${VERSION}_${OS}_${ARCH}.tar.gz" \
              | sudo tar -xz -C /usr/local/bin ant
            ```

            您可以在 [GitHub 发布页面](https://github.com/anthropics/anthropic-cli/releases)上找到所有发布版本。
          </Tab>

          <Tab title="go install">
            您也可以使用 `go install` 从源代码安装 CLI，这需要 Go 1.26.10 或更高版本。

            ```bash
            go install github.com/anthropics/anthropic-cli/v2/cmd/ant@latest
            ```

            `go install` 会将二进制文件放置在 `$(go env GOPATH)/bin` 中。如果该目录尚未添加到您的 `PATH` 中，请将其添加进去：

            ```bash
            export PATH="$PATH:$(go env GOPATH)/bin"
            ```
          </Tab>
        </Tabs>

        在终端中运行 `ant messages create`：

        ```bash CLI
        ant messages create \
          --model claude-opus-5-5 \
          --max-tokens 1000 \
          --message '{
            role: user,
            content: "What should I search for to find the latest developments in renewable energy?"
          }'
        ```

        CLI 会打印 JSON 响应：

        ```json Output
        {
          "model": "claude-opus-5-5",
          "id": "msg_01N1ycuCkM5Mzd7WhTU4fwST",
          "type": "message",
          "role": "assistant",
          "content": [
            {
              "type": "text",
              "text": "Here are some effective search strategies to find the latest developments in renewable energy:\n\n## General Search Terms\n- \"Renewable energy news 2025\"\n- ..."
            }
          ],
          "stop_reason": "end_turn",
          "stop_sequence": null,
          "stop_details": null,
          "usage": { "input_tokens": 21, "output_tokens": 305 }
        }
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="Python">
    <Steps>
      <Step title="创建项目并安装 SDK" stepNumber={5}>
        ```bash
        mkdir claude-quickstart && cd claude-quickstart
        python3 -m venv .venv && source .venv/bin/activate
        pip install anthropic
        ```
      </Step>

      <Step title="编写代码" stepNumber={6}>
        创建一个名为 `quickstart.py` 的文件：

        ```python Python
        import anthropic

        client = anthropic.Anthropic()

        message = client.messages.create(
            model="claude-opus-5-5",
            max_tokens=1000,
            messages=[
                {
                    "role": "user",
                    "content": "What should I search for to find the latest developments in renewable energy?",
                }
            ],
        )

        for block in message.content:
            if block.type == "text":
                print(block.text)
        ```
      </Step>

      <Step title="运行代码" stepNumber={7}>
        ```bash
        python quickstart.py
        ```

        ```text Output wrap
        Here are some effective search strategies to find the latest developments in renewable energy:

        ## General Search Terms
        - "Renewable energy news 2025"
        - ...
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="TypeScript">
    <Steps>
      <Step title="创建项目并安装 SDK" stepNumber={5}>
        ```bash
        mkdir claude-quickstart && cd claude-quickstart
        npm init -y
        npm pkg set type=module
        npm install @anthropic-ai/sdk
        ```
      </Step>

      <Step title="编写代码" stepNumber={6}>
        创建一个名为 `quickstart.ts` 的文件：

        ```typescript TypeScript
        import Anthropic from "@anthropic-ai/sdk";

        const client = new Anthropic();

        const message = await client.messages.create({
          model: "claude-opus-5-5",
          max_tokens: 1000,
          messages: [
            {
              role: "user",
              content: "What should I search for to find the latest developments in renewable energy?"
            }
          ]
        });

        for (const block of message.content) {
          if (block.type === "text") {
            console.log(block.text);
          }
        }
        ```
      </Step>

      <Step title="运行代码" stepNumber={7}>
        ```bash
        npx tsx quickstart.ts
        ```

        ```text Output wrap
        Here are some effective search strategies to find the latest developments in renewable energy:

        ## General Search Terms
        - "Renewable energy news 2025"
        - ...
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="C#">
    <Steps>
      <Step title="创建项目并安装 SDK" stepNumber={5}>
        创建一个新的控制台项目并添加 Anthropic 包：

        ```bash
        dotnet new console -n ClaudeQuickstart
        cd ClaudeQuickstart
        dotnet add package Anthropic
        ```
      </Step>

      <Step title="编写代码" stepNumber={6}>
        替换 `Program.cs` 的内容：

        ```csharp C#
        using Anthropic;
        using Anthropic.Models.Messages;

        var client = new AnthropicClient();

        var message = await client.Messages.Create(new MessageCreateParams
        {
            Model = Model.ClaudeOpus5_5,
            MaxTokens = 1000,
            Messages =
            [
                new UserMessageParam
                {
                    Content = "What should I search for to find the latest developments in renewable energy?",
                },
            ],
        });

        foreach (var block in message.Content)
        {
            if (block.TryPickText(out var textBlock))
            {
                Console.WriteLine(textBlock.Text);
            }
        }
        ```
      </Step>

      <Step title="运行代码" stepNumber={7}>
        ```bash
        dotnet run
        ```

        ```text Output wrap
        Here are some effective search strategies to find the latest developments in renewable energy:

        ## General Search Terms
        - "Renewable energy news 2025"
        - ...
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="Go">
    <Steps>
      <Step title="创建项目并安装 SDK" stepNumber={5}>
        创建一个新模块并添加 Anthropic SDK：

        ```bash
        mkdir claude-quickstart && cd claude-quickstart
        go mod init claude-quickstart
        go get github.com/anthropics/anthropic-sdk-go/v2
        ```
      </Step>

      <Step title="编写代码" stepNumber={6}>
        创建一个名为 `main.go` 的文件：

        ```go Go
        package main

        import (
        	"context"
        	"fmt"
        	"log"

        	"github.com/anthropics/anthropic-sdk-go/v2"
        )

        func main() {
        	client := anthropic.NewClient()

        	message, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
        		Model:     anthropic.ModelClaudeOpus5_5,
        		MaxTokens: 1000,
        		Messages: []anthropic.MessageParam{
        			anthropic.NewUserMessage(anthropic.NewTextBlock("What should I search for to find the latest developments in renewable energy?")),
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
        }
        ```
      </Step>

      <Step title="运行代码" stepNumber={7}>
        ```bash
        go run .
        ```

        ```text Output wrap
        Here are some effective search strategies to find the latest developments in renewable energy:

        ## General Search Terms
        - "Renewable energy news 2025"
        - ...
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="Java">
    <Steps>
      <Step title="设置您的项目" stepNumber={5}>
        您的 `PATH` 中需要有 JDK（25 或更高版本）以及 [Gradle](https://gradle.org/install/) 或 [Maven](https://maven.apache.org/install.html) 之一。为您的项目创建一个目录，并在其中创建 Java 源代码目录：

        ```bash
        mkdir -p claude-quickstart/src/main/java && cd claude-quickstart
        ```

        然后添加一个构建文件。在 [Maven Central](https://central.sonatype.com/artifact/com.anthropic/anthropic-java) 上查找当前的 SDK 版本。

        <Tabs>
          <Tab title="Gradle">
            将以下内容保存为 `build.gradle.kts`：

            ```kotlin
            plugins {
                application
            }

            repositories {
                mavenCentral()
            }

            java {
                toolchain {
                    languageVersion = JavaLanguageVersion.of(25)
                }
            }

            dependencies {
                implementation("com.anthropic:anthropic-java:3.8.0")
            }

            application {
                mainClass = "QuickStart"
            }
            ```
          </Tab>

          <Tab title="Maven">
            将以下内容保存为 `pom.xml`：

            ```xml
            <project xmlns="http://maven.apache.org/POM/4.0.0">
              <modelVersion>4.0.0</modelVersion>
              <groupId>com.example</groupId>
              <artifactId>quickstart</artifactId>
              <version>1.0-SNAPSHOT</version>
              <properties>
                <maven.compiler.release>25</maven.compiler.release>
                <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
              </properties>
              <dependencies>
                <dependency>
                  <groupId>com.anthropic</groupId>
                  <artifactId>anthropic-java</artifactId>
                  <version>3.8.0</version>
                </dependency>
              </dependencies>
            </project>
            ```
          </Tab>
        </Tabs>
      </Step>

      <Step title="编写代码" stepNumber={6}>
        将以下内容保存为项目 Java 源代码目录（通常为 `src/main/java/`）中的 `QuickStart.java`：

        ```java Java
        import com.anthropic.client.okhttp.AnthropicOkHttpClient;
        import com.anthropic.models.messages.Message;
        import com.anthropic.models.messages.MessageCreateParams;
        import com.anthropic.models.messages.Model;

        static void main() {
            var client = AnthropicOkHttpClient.fromEnv();

            var params = MessageCreateParams.builder()
                .model(Model.CLAUDE_OPUS_5_5)
                .maxTokens(1000)
                .addUserMessage(
                    "What should I search for to find the latest developments in renewable energy?"
                )
                .build();

            Message message = client.messages().create(params);
            for (var block : message.content()) {
                block.text().ifPresent(textBlock -> IO.println(textBlock.text()));
            }
        }
        ```
      </Step>

      <Step title="运行代码" stepNumber={7}>
        <Tabs>
          <Tab title="Gradle">
            ```bash
            gradle run
            ```
          </Tab>

          <Tab title="Maven">
            ```bash
            mvn compile exec:java -Dexec.mainClass=QuickStart
            ```
          </Tab>
        </Tabs>

        ```text Output wrap
        Here are some effective search strategies to find the latest developments in renewable energy:

        ## General Search Terms
        - "Renewable energy news 2025"
        - ...
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="PHP">
    <Steps>
      <Step title="创建项目并安装 SDK" stepNumber={5}>
        ```bash
        mkdir claude-quickstart && cd claude-quickstart
        composer require "anthropic-ai/sdk" "guzzlehttp/guzzle:^7"
        ```
      </Step>

      <Step title="编写代码" stepNumber={6}>
        创建一个名为 `quickstart.php` 的文件：

        ```php PHP
        <?php
        require 'vendor/autoload.php';

        use Anthropic\Client;
        use Anthropic\Messages\Model;
        use Anthropic\Messages\TextBlock;

        $client = new Client();

        $message = $client->messages->create(
            model: Model::CLAUDE_OPUS_5_5,
            maxTokens: 1000,
            messages: [
                [
                    'role' => 'user',
                    'content' => 'What should I search for to find the latest developments in renewable energy?',
                ],
            ],
        );

        foreach ($message->content as $block) {
            if ($block instanceof TextBlock) {
                echo $block->text . PHP_EOL;
            }
        }
        ```
      </Step>

      <Step title="运行代码" stepNumber={7}>
        ```bash
        php quickstart.php
        ```

        ```text Output wrap
        Here are some effective search strategies to find the latest developments in renewable energy:

        ## General Search Terms
        - "Renewable energy news 2025"
        - ...
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="Ruby">
    <Steps>
      <Step title="创建项目并安装 SDK" stepNumber={5}>
        ```bash
        mkdir claude-quickstart && cd claude-quickstart
        bundle init
        bundle add anthropic
        ```
      </Step>

      <Step title="编写代码" stepNumber={6}>
        创建一个名为 `quickstart.rb` 的文件：

        ```ruby Ruby
        require "anthropic"

        client = Anthropic::Client.new

        message = client.messages.create(
          model: Anthropic::Model::CLAUDE_OPUS_5_5,
          max_tokens: 1000,
          messages: [
            {
              role: "user",
              content: "What should I search for to find the latest developments in renewable energy?"
            }
          ]
        )

        message.content.each do |block|
          puts block.text if block.type == :text
        end
        ```
      </Step>

      <Step title="运行代码" stepNumber={7}>
        ```bash
        bundle exec ruby quickstart.rb
        ```

        ```text Output wrap
        Here are some effective search strategies to find the latest developments in renewable energy:

        ## General Search Terms
        - "Renewable energy news 2025"
        - ...
        ```
      </Step>
    </Steps>
  </Tab>
</Tabs>

## 后续步骤

您已经完成了第一次 API 调用。接下来，学习您将在每个 Claude 集成中使用的 Messages API 模式。

<Card title="使用 Messages API" icon="messages" href="https://platform.claude.com/docs/zh-CN/build-with-claude/working-with-messages">
  学习多轮对话、系统提示、停止原因以及其他核心模式。
</Card>

熟悉基础知识后，可进一步探索：

<CardGroup cols={2}>
  <Card title="模型概览" icon="brain" href="https://platform.claude.com/docs/zh-CN/models/overview">
    按能力和成本比较 Claude 模型。
  </Card>

  <Card title="功能概览" icon="list" href="https://platform.claude.com/docs/zh-CN/build-with-claude/overview">
    浏览 Claude 的所有功能：工具、上下文管理、结构化输出等。
  </Card>

  <Card title="客户端 SDK" icon="code-brackets" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview">
    Python、TypeScript、C# 及其他客户端库的参考文档。
  </Card>

  <Card title="身份验证" icon="lock" href="https://platform.claude.com/docs/zh-CN/manage-claude/authentication">
    比较 API 密钥与工作负载身份联合（Workload Identity Federation），并设置密钥过期时间。
  </Card>
</CardGroup>
