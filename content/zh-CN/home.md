---
title: 文档
url: https://platform.claude.com/docs/zh-CN/home
description: Claude API 文档
---

<HomePage>
  <HomeHero
    eyebrow="Claude Platform"
    title="开始
使用 Claude 构建"
    subtitle="将 Claude 集成到您的应用程序中所需的一切。从第一次 API 调用到生产环境。"
  >
    <HomeQuickChip icon="Play" href="https://platform.claude.com/docs/zh-CN/get-started">
      快速入门
    </HomeQuickChip>

    <HomeQuickChip icon="Key" href="https://platform.claude.com/settings/keys">
      在 Claude Console 中获取 API 密钥
    </HomeQuickChip>

    <HomeQuickChip icon="CodeBrackets" href="https://platform.claude.com/docs/zh-CN/api/overview">
      API 参考
    </HomeQuickChip>

    ```python Python
    import anthropic

    client = anthropic.Anthropic()

    message = client.messages.create(
        model="claude-opus-5-5",
        max_tokens=1024,
        messages=[
            {
                "role": "user",
                "content": "Hello, Claude",
            }
        ],
    )
    for block in message.content:
        if block.type == "text":
            print(block.text)
    ```

    ```typescript TypeScript
    import Anthropic from "@anthropic-ai/sdk";

    const client = new Anthropic();

    const msg = await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [
        {
          role: "user",
          content: "Hello, Claude"
        }
      ]
    });
    for (const block of msg.content) {
      if (block.type === "text") {
        console.log(block.text);
      }
    }
    ```

    ```go Go
    import "github.com/anthropics/anthropic-sdk-go/v2"

    client := anthropic.NewClient()
    msg, _ := client.Messages.New(
      context.TODO(),
      anthropic.MessageNewParams{
        Model:     anthropic.ModelClaudeOpus5_5,
        MaxTokens: 1024,
        Messages: []anthropic.MessageParam{
          anthropic.NewUserMessage(
            anthropic.NewTextBlock("Hello, Claude"),
          ),
        },
      },
    )
    for _, block := range msg.Content {
      if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
        fmt.Println(textBlock.Text)
      }
    }
    ```

    ```java Java
    import com.anthropic.client.okhttp.AnthropicOkHttpClient;

    var client = AnthropicOkHttpClient
      .fromEnv();

    var msg = client.messages().create(
      MessageCreateParams.builder()
        .model(Model.CLAUDE_OPUS_5_5)
        .maxTokens(1024)
        .addUserMessage("Hello, Claude")
        .build()
    );
    for (var block : msg.content()) {
      block.text().ifPresent(
        textBlock -> System.out.println(textBlock.text()));
    }
    ```

    ```ruby Ruby
    require "anthropic"

    client = Anthropic::Client.new

    msg = client.messages.create(
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{
        role: "user",
        content: "Hello, Claude"
      }]
    )
    msg.content.each do |block|
      puts block.text if block.type == :text
    end
    ```

    ```php PHP
    use Anthropic\Client;

    $client = new Client();

    $message = $client->messages->create(
      model: "claude-opus-5-5",
      maxTokens: 1024,
      messages: [['role' => 'user',
        'content' => 'Hello, Claude']],
    );
    foreach ($message->content as $block) {
      if ($block->type === 'text') {
        echo $block->text, PHP_EOL;
      }
    }
    ```

    ```csharp C#
    using Anthropic;

    var client = new AnthropicClient();

    var msg = await client.Messages
      .Create(new() {
        Model = Model.ClaudeOpus5_5,
        MaxTokens = 1024,
        Messages = [new UserMessageParam {
          Content = "Hello, Claude"
        }]
      });
    foreach (var block in msg.Content)
    {
      if (block.TryPickText(out var textBlock))
      {
        Console.WriteLine(textBlock.Text);
      }
    }
    ```

    ```bash cURL
    curl https://api.anthropic.com/v1/messages \
      -H "content-type: application/json" \
      -H "x-api-key: $ANTHROPIC_API_KEY" \
      -H "anthropic-version: 2026-10-21" \
      -d '{
        "model": "claude-opus-5-5",
        "max_tokens": 1024,
        "messages": [{
          "role": "user",
          "content": "Hello, Claude"
        }]
      }'
    ```

    ```bash CLI
    ant messages create \
      --model claude-opus-5-5 \
      --max-tokens 1024 \
      --message '{
        role: user,
        content: "Hello, Claude"
      }'
    ```
  </HomeHero>

  <HomeSection>
    <HomeSectionHeader label="平台" title="选择您的构建方式" description="选择与您的方法相匹配的开发者界面，以及适合您技术栈的基础设施。" />

    <HomePlatformCards>
      <HomePlatformCard title="Messages" description="直接访问模型。您构建每一轮对话、管理对话状态，并编写自己的工具循环。" pictogram="code-terminal">
        <HomeCardLink icon="play" href="https://platform.claude.com/docs/zh-CN/get-started">
          快速入门
        </HomeCardLink>

        <HomeCardLink icon="book" href="https://platform.claude.com/docs/zh-CN/api/messages/create">
          API 参考
        </HomeCardLink>

        <HomeCardLink icon="code-brackets" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview">
          客户端 SDK
        </HomeCardLink>
      </HomePlatformCard>

      <HomePlatformCard title="Managed Agents" description="完全托管的智能体基础设施。在具有持久事件历史的有状态会话中部署和管理自主智能体。" pictogram="clouds">
        <HomeCardLink icon="play" href="https://platform.claude.com/docs/zh-CN/managed-agents/quickstart">
          快速入门
        </HomeCardLink>

        <HomeCardLink icon="book" href="https://platform.claude.com/docs/zh-CN/api/beta/sessions">
          API 参考
        </HomeCardLink>

        <HomeCardLink icon="brain" href="https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup">
          定义您的智能体
        </HomeCardLink>
      </HomePlatformCard>
    </HomePlatformCards>

    <HomePartnerLinks label="Claude 也可在以下云平台上使用：">
      <HomeCloudPartnerLink icon="cloud" href="https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock">
        Amazon Bedrock
      </HomeCloudPartnerLink>

      <HomeCloudPartnerLink icon="cloud" href="https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/overview">
        Google Cloud
      </HomeCloudPartnerLink>

      <HomeCloudPartnerLink icon="cloud" href="https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry">
        Microsoft Foundry
      </HomeCloudPartnerLink>
    </HomePartnerLinks>
  </HomeSection>

  <HomeSection>
    <HomeSectionHeader label="开发者旅程" title="从想法到生产" description="按照生命周期逐步进行，或直接跳转到您需要的内容。" />

    <HomeJourney>
      <HomeJourneyTab label="Messages">
        <HomeJourneyStep title="开始使用">
          <HomeJourneyLink icon="play" href="https://platform.claude.com/docs/zh-CN/get-started">
            快速入门
          </HomeJourneyLink>

          <HomeJourneyLink icon="lock" href="https://platform.claude.com/settings/keys">
            在 Claude Console 中获取 API 密钥
          </HomeJourneyLink>

          <HomeJourneyLink icon="settings" href="https://platform.claude.com/docs/zh-CN/models/overview">
            选择模型
          </HomeJourneyLink>

          <HomeJourneyLink icon="code-brackets" href="https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview">
            安装 SDK
          </HomeJourneyLink>

          <HomeJourneyLink icon="message" href="https://platform.claude.com/playground">
            在 Console playground 中试用 API
          </HomeJourneyLink>
        </HomeJourneyStep>

        <HomeJourneyStep title="构建">
          <HomeJourneyLink icon="message" href="https://platform.claude.com/docs/zh-CN/api/messages/create">
            Messages API
          </HomeJourneyLink>

          <HomeJourneyLink icon="brain" href="https://platform.claude.com/docs/zh-CN/build-with-claude/thinking">
            思考
          </HomeJourneyLink>

          <HomeJourneyLink icon="image" href="https://platform.claude.com/docs/zh-CN/build-with-claude/vision">
            视觉
          </HomeJourneyLink>

          <HomeJourneyLink icon="tool" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/overview">
            工具使用
          </HomeJourneyLink>

          <HomeJourneyLink icon="compass" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-search-tool">
            网页搜索
          </HomeJourneyLink>

          <HomeJourneyLink icon="code" href="https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool">
            代码执行
          </HomeJourneyLink>

          <HomeJourneyLink icon="database" href="https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs">
            结构化输出
          </HomeJourneyLink>

          <HomeJourneyLink icon="lightning" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching">
            提示缓存
          </HomeJourneyLink>

          <HomeJourneyLink icon="wifi-high" href="https://platform.claude.com/docs/zh-CN/build-with-claude/streaming">
            流式传输
          </HomeJourneyLink>
        </HomeJourneyStep>

        <HomeJourneyStep title="评估与发布">
          <HomeJourneyLink icon="lightbulb" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/overview">
            提示最佳实践
          </HomeJourneyLink>

          <HomeJourneyLink icon="chart" href="https://platform.claude.com/docs/zh-CN/test-and-evaluate/develop-tests">
            运行评估
          </HomeJourneyLink>

          <HomeJourneyLink icon="stack" href="https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing">
            批量测试
          </HomeJourneyLink>

          <HomeJourneyLink icon="verified" href="https://platform.claude.com/docs/zh-CN/test-and-evaluate/strengthen-guardrails/increase-consistency">
            安全与防护
          </HomeJourneyLink>

          <HomeJourneyLink icon="bolt" href="https://platform.claude.com/docs/zh-CN/api/rate-limits">
            速率限制与错误
          </HomeJourneyLink>

          <HomeJourneyLink icon="calculator" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing">
            成本优化
          </HomeJourneyLink>
        </HomeJourneyStep>

        <HomeJourneyStep title="运营">
          <HomeJourneyLink icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/workspaces">
            工作区与管理
          </HomeJourneyLink>

          <HomeJourneyLink icon="lock" href="https://platform.claude.com/settings/keys">
            Claude Console 中的 API 密钥
          </HomeJourneyLink>

          <HomeJourneyLink icon="chart" href="https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api">
            用量监控
          </HomeJourneyLink>

          <HomeJourneyLink icon="settings" href="https://platform.claude.com/docs/zh-CN/about-claude/models/migration-guide">
            模型迁移
          </HomeJourneyLink>
        </HomeJourneyStep>
      </HomeJourneyTab>

      <HomeJourneyTab label="Managed Agents">
        <HomeJourneyStep title="开始使用">
          <HomeJourneyLink icon="play" href="https://platform.claude.com/docs/zh-CN/managed-agents/quickstart">
            快速入门
          </HomeJourneyLink>

          <HomeJourneyLink icon="lock" href="https://platform.claude.com/settings/keys">
            在 Claude Console 中获取 API 密钥
          </HomeJourneyLink>

          <HomeJourneyLink icon="message" href="https://platform.claude.com/docs/zh-CN/managed-agents/onboarding">
            在 Console 中构建
          </HomeJourneyLink>
        </HomeJourneyStep>

        <HomeJourneyStep title="定义您的智能体">
          <HomeJourneyLink icon="brain" href="https://platform.claude.com/docs/zh-CN/managed-agents/agent-setup">
            智能体设置
          </HomeJourneyLink>

          <HomeJourneyLink icon="tool" href="https://platform.claude.com/docs/zh-CN/managed-agents/tools">
            工具
          </HomeJourneyLink>

          <HomeJourneyLink icon="lock" href="https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies">
            工具权限
          </HomeJourneyLink>
        </HomeJourneyStep>

        <HomeJourneyStep title="运行会话">
          <HomeJourneyLink icon="wifi-high" href="https://platform.claude.com/docs/zh-CN/managed-agents/events-and-streaming">
            流式传输与事件
          </HomeJourneyLink>

          <HomeJourneyLink icon="code-brackets" href="https://platform.claude.com/docs/zh-CN/api/beta/sessions">
            Sessions API 参考
          </HomeJourneyLink>
        </HomeJourneyStep>

        <HomeJourneyStep title="运营">
          <HomeJourneyLink icon="settings" href="https://platform.claude.com/docs/zh-CN/manage-claude/workspaces">
            工作区与管理
          </HomeJourneyLink>

          <HomeJourneyLink icon="lock" href="https://platform.claude.com/settings/keys">
            Claude Console 中的 API 密钥
          </HomeJourneyLink>

          <HomeJourneyLink icon="chart" href="https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api">
            用量监控
          </HomeJourneyLink>
        </HomeJourneyStep>
      </HomeJourneyTab>
    </HomeJourney>
  </HomeSection>

  <HomeSection>
    <HomeSectionHeader label="模型" title="Claude 模型家族" description="为您的用例选择合适的模型。" />

    * [Claude Fable 5.2](https://platform.claude.com/docs/zh-CN/models/fable-5-2/overview) (`claude-fable-5-2`) — New — *For demanding reasoning and long-horizon agentic work* — Most capable · Research · Multi-day tasks
    * [Claude Opus 5.5](https://platform.claude.com/docs/zh-CN/models/opus-5-5/overview) (`claude-opus-5-5`) — New — *For long-running agentic coding and knowledge work* — Complex projects · Agents · Coding
    * [Claude Sonnet 5.5](https://platform.claude.com/docs/zh-CN/models/sonnet-5-5/overview) (`claude-sonnet-5-5`) — New — *The best combination of speed and intelligence* — Everyday tasks · Writing · Cost-efficient
    * [Claude Haiku 5.5](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/overview) (`claude-haiku-5-5`) — New — *For high-volume, latency-sensitive tasks such as classification, extraction, and routing* — Fastest · Lowest cost · High volume
  </HomeSection>

  <HomeSection last>
    <HomeSectionHeader label="资源" title="继续学习" />

    <CardGroup cols={3}>
      <Card icon="graduation-cap" title="课程" href="https://academy.claude.com/courses">
        掌握 Claude 的互动课程。
      </Card>

      <Card icon="book" title="Claude Cookbook" href="https://platform.claude.com/cookbook">
        代码示例与模式。
      </Card>

      <Card icon="play" title="快速入门" href="https://github.com/anthropics/claude-quickstarts">
        可部署的入门应用。
      </Card>

      <Card icon="star" title="最新动态" href="https://platform.claude.com/docs/zh-CN/release-notes/overview">
        最新功能与更新。
      </Card>

      <Card icon="terminal" title="Claude Code" href="https://code.claude.com/docs">
        您终端中的智能体编码助手。
      </Card>
    </CardGroup>
  </HomeSection>
</HomePage>
