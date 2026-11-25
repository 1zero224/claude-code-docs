---
title: 处理错误
url: https://platform.claude.com/docs/zh-CN/api/handle-errors
description: 使用 SDK 按错误代码、HTTP 状态或连接失败捕获 Claude API 错误，并处理在流中到达的错误。
---

使用 SDK 按错误代码、HTTP 状态或连接失败捕获 Claude API 错误，并处理在流中到达的错误。要了解每种错误的含义，或在不使用 SDK 的情况下调用 API，请参阅 [Claude API 错误](https://platform.claude.com/docs/zh-CN/api/errors)。

<Note>
  本页中的示例需要[最新版本的 SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview#latest-versions)。
</Note>

## 按类型捕获错误

此示例处理一个[错误代码](https://platform.claude.com/docs/zh-CN/api/errors#error-codes)、一个 HTTP 状态、一个连接失败，以及所有其他错误响应：

<CodeGroup exclude="shell:cURL, shell:CLI">
  ```python Python
  client = anthropic.Anthropic()

  try:
      client.messages.create(
          model="claude-opus-5-5",
          max_tokens=1024,
          messages=[{"role": "user", "content": "Hello, Claude"}],
      )
  except anthropic.EnforcedSpendLimitReachedError as e:
      # 一种错误代码：enforced_spend_limit_reached
      print("Monthly spend cap reached:", e.api_message)
  except anthropic.RateLimitError as e:
      # 一种 HTTP 状态：其他任何 429
      print("Rate limited. Request ID:", e.request_id)
  except anthropic.APIConnectionError:
      # 没有响应的错误，例如网络故障或超时
      print("Connection failed")
  except anthropic.APIStatusError as e:
      # 所有其他错误响应
      print(e.status_code)  # HTTP status
      print(e.type)  # error.type
      print(e.api_message)  # error.message
      print(e.error_code)  # error.details.error_code if any
      print(e.details)  # error.details if any
      print(e.request_id)  # request_id
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  try {
    await client.messages.create({
      model: "claude-opus-5-5",
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello, Claude" }]
    });
  } catch (err) {
    if (err instanceof Anthropic.EnforcedSpendLimitReachedError) {
      // 一种错误代码：enforced_spend_limit_reached
      console.log("Monthly spend cap reached:", err.apiMessage);
    } else if (err instanceof Anthropic.RateLimitError) {
      // 一种 HTTP 状态：其他任何 429
      console.log("Rate limited. Request ID:", err.requestID);
    } else if (err instanceof Anthropic.APIConnectionError) {
      // 没有响应的错误，例如网络故障或超时
      console.log("Connection failed");
    } else if (err instanceof Anthropic.APIError) {
      // 所有其他错误响应
      console.log(err.status); // HTTP status
      console.log(err.type); // error.type
      console.log(err.apiMessage); // error.message
      console.log(err.errorCode); // error.details.error_code if any
      console.log(err.details); // error.details if any
      console.log(err.requestID); // request_id
    } else {
      throw err;
    }
  }
  ```

  ```csharp C#
  using Anthropic.Exceptions;
  using Anthropic.Models;
  using Anthropic.Models.Messages;

  AnthropicClient client = new();

  try
  {
      await client.Messages.Create(new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new UserMessageParam { Content = "Hello, Claude" }]
      });
  }
  catch (AnthropicEnforcedSpendLimitReachedException e)
  {
      // 一种错误代码：enforced_spend_limit_reached
      Console.WriteLine($"Monthly spend cap reached: {e.ApiMessage}");
  }
  catch (AnthropicRateLimitException e)
  {
      // 一种 HTTP 状态：其他任何 429
      Console.WriteLine($"Rate limited. Request ID: {e.RequestID}");
  }
  catch (Exception e) when (e is AnthropicIOException or TaskCanceledException)
  {
      // 没有响应的错误，例如网络故障或超时
      Console.WriteLine("Connection failed");
  }
  catch (AnthropicApiException e)
  {
      // 所有其他错误响应
      Console.WriteLine((int)e.StatusCode); // HTTP status
      Console.WriteLine(e.Type?.Raw()); // error.type
      Console.WriteLine(e.ApiMessage); // error.message
      Console.WriteLine(e.ErrorCode); // error.details.error_code if any
      Console.WriteLine(e.Details); // error.details if any
      Console.WriteLine(e.RequestID); // request_id
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  _, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  })

  // Go SDK 只有一种错误类型。请根据错误的代码和状态进行分支处理。
  var apierr *anthropic.Error
  if errors.As(err, &apierr) {
  	switch {
  	case apierr.ErrorCode() == anthropic.ErrorCodeEnforcedSpendLimitReached:
  		// 一种错误代码：enforced_spend_limit_reached
  		fmt.Println("Monthly spend cap reached:", apierr.Message)
  	case apierr.StatusCode == http.StatusTooManyRequests:
  		// 一种 HTTP 状态：其他任何 429
  		fmt.Println("Rate limited. Request ID:", apierr.RequestID)
  	default:
  		// 所有其他错误响应
  		fmt.Println(apierr.StatusCode)        // HTTP status
  		fmt.Println(apierr.Type())            // error.type
  		fmt.Println(apierr.Message)           // error.message
  		fmt.Println(apierr.ErrorCode())       // error.details.error_code if any
  		fmt.Println(apierr.Details.RawJSON()) // error.details if any
  		fmt.Println(apierr.RequestID)         // request_id
  	}
  } else if err != nil {
  	// 任何其他错误，例如网络故障或超时
  	fmt.Println("Connection failed")
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.errors.AnthropicIoException;
  import com.anthropic.errors.AnthropicServiceException;
  import com.anthropic.errors.RateLimitException;
  import com.anthropic.models.EnforcedSpendLimitReachedException;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addUserMessage("Hello, Claude")
          .build();

      try {
          client.messages().create(params);
      } catch (EnforcedSpendLimitReachedException e) {
          // 一种错误代码：enforced_spend_limit_reached
          IO.println("Monthly spend cap reached: " + e.apiMessage().orElse(""));
      } catch (RateLimitException e) {
          // 一种 HTTP 状态：其他任何 429
          IO.println("Rate limited. Request ID: " + e.requestId().orElse(""));
      } catch (AnthropicIoException e) {
          // 没有响应的错误，例如网络故障或超时
          IO.println("Connection failed");
      } catch (AnthropicServiceException e) {
          // 所有其他错误响应
          IO.println(e.statusCode()); // HTTP status
          IO.println(e.type().orElse(null)); // error.type
          IO.println(e.apiMessage().orElse(null)); // error.message
          IO.println(e.errorCode().orElse(null)); // error.details.error_code if any
          IO.println(e.errorDetails().orElse(null)); // error.details if any
          IO.println(e.requestId().orElse(null)); // request_id
      }
  }
  ```

  ```php PHP
  use Anthropic\Core\Exceptions\APIConnectionException;
  use Anthropic\Core\Exceptions\APIStatusException;
  use Anthropic\Core\Exceptions\RateLimitException;
  use Anthropic\EnforcedSpendLimitReachedException;
  use Anthropic\Messages\Model;

  $client = new Client();

  try {
      $client->messages->create(
          model: Model::CLAUDE_OPUS_5_5,
          maxTokens: 1024,
          messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      );
  } catch (EnforcedSpendLimitReachedException $e) {
      // 一种错误代码：enforced_spend_limit_reached
      echo 'Monthly spend cap reached: ', $e->getAPIMessage(), PHP_EOL;
  } catch (RateLimitException $e) {
      // 一种 HTTP 状态：其他任何 429
      echo 'Rate limited. Request ID: ', $e->getRequestID(), PHP_EOL;
  } catch (APIConnectionException) {
      // 没有响应的错误，例如网络故障或超时
      echo 'Connection failed', PHP_EOL;
  } catch (APIStatusException $e) {
      // 所有其他错误响应
      echo $e->status, PHP_EOL; // HTTP status
      echo $e->getType(), PHP_EOL; // error.type
      echo $e->getAPIMessage(), PHP_EOL; // error.message
      echo $e->getErrorCode(), PHP_EOL; // error.details.error_code if any
      echo json_encode($e->getDetails()), PHP_EOL; // error.details if any
      echo $e->getRequestID(), PHP_EOL; // request_id
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  begin
    client.messages.create(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello, Claude" }]
    )
  rescue Anthropic::EnforcedSpendLimitReachedError => e
    # 一种错误代码：enforced_spend_limit_reached
    puts "Monthly spend cap reached: #{e.api_message}"
  rescue Anthropic::Errors::RateLimitError => e
    # 一种 HTTP 状态：其他任何 429
    puts "Rate limited. Request ID: #{e.request_id}"
  rescue Anthropic::Errors::APIConnectionError
    # 没有响应的错误，例如网络故障或超时
    puts "Connection failed"
  rescue Anthropic::Errors::APIStatusError => e
    # 所有其他错误响应
    puts e.status # HTTP status
    puts e.type # error.type as a symbol
    puts e.api_message # error.message
    puts e.error_code # error.details.error_code as a symbol if any
    puts e.details # error.details if any
    puts e.request_id # request_id
  end
  ```
</CodeGroup>

错误会在 SDK 自身的重试（如果有）失败之后到达您的代码（请参阅[重试](https://platform.claude.com/docs/zh-CN/api/errors#retries)）。要了解每种状态的含义，请参阅 [HTTP 错误](https://platform.claude.com/docs/zh-CN/api/errors#http-errors)。

请将范围较窄的分支放在范围较宽的分支之前。Messages API 会以 429 返回 `enforced_spend_limit_reached`，这是[每月支出上限](https://platform.claude.com/docs/zh-CN/api/rate-limits#reaching-your-spend-cap)对应的代码。如果 429 的分支放在前面，它也会捕获该代码。

要处理其他情况（例如 404），请参阅 [SDK 错误类型](https://platform.claude.com/docs/zh-CN/api/errors#sdk-error-types)。

## 处理特定错误代码

要处理其他错误代码，请将其分支添加到[按类型捕获错误](https://platform.claude.com/docs/zh-CN/api/handle-errors#catch-errors-by-kind)中，并放在任何范围更宽的分支之前。

每个代码都有自己的类或常量，以该代码命名，例如 `anthropic.EnforcedSpendLimitReachedError` (typescript: `Anthropic.EnforcedSpendLimitReachedError`; ruby: `Anthropic::EnforcedSpendLimitReachedError`; java: `com.anthropic.models.EnforcedSpendLimitReachedException`; php: `Anthropic\EnforcedSpendLimitReachedException`; csharp: `Anthropic.Models.AnthropicEnforcedSpendLimitReachedException`; go: `anthropic.ErrorCodeEnforcedSpendLimitReached`)。其他代码的类可能位于其他命名空间中。要找到某个类，请在 SDK 的源代码中搜索该类名。方法的文档注释会列出该方法可能返回的每个代码所对应的类或常量。

有关完整的代码列表，请参阅 API 参考中的 [`ErrorDetails`](https://platform.claude.com/docs/zh-CN/api/$shared#error_details)。

## 处理流中的错误

流开始后，API 已经返回了 200，因此它会以[错误事件](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming#error-events)的形式发送错误。SDK 不会重试该请求。

每个错误事件都会以 `APIStatusError` (typescript: `APIError`; csharp: `AnthropicSseException`; java: `SseException`; php: `APIStatusException`; go: `*anthropic.Error`) 的形式到达您的代码。它的 HTTP 状态是流的 200，或者不存在。因此，针对某个状态或某个代码的类的分支无法捕获它。请改为检查它的 `error.type` 是否为 [HTTP 错误](https://platform.claude.com/docs/zh-CN/api/errors#http-errors)中的类型之一：

<CodeGroup exclude="shell:cURL, shell:CLI">
  ```python Python
  client = anthropic.Anthropic()

  try:
      with client.messages.stream(
          model="claude-opus-5-5",
          max_tokens=1024,
          messages=[{"role": "user", "content": "Hello, Claude"}],
      ) as stream:
          for text in stream.text_stream:
              print(text, end="", flush=True)
  except anthropic.APIStatusError as e:
      if e.type == "overloaded_error":
          # 一种 error.type：overloaded_error
          print("\nOverloaded:", e.api_message)
      else:
          raise
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  try {
    await client.messages
      .stream({
        model: "claude-opus-5-5",
        max_tokens: 1024,
        messages: [{ role: "user", content: "Hello, Claude" }]
      })
      .on("text", (text) => {
        process.stdout.write(text);
      })
      .finalMessage();
  } catch (err) {
    if (err instanceof Anthropic.APIError && err.type === "overloaded_error") {
      // 一种 error.type：overloaded_error
      console.log("\nOverloaded:", err.apiMessage);
    } else {
      throw err;
    }
  }
  ```

  ```csharp C#
  using Anthropic.Exceptions;
  using Anthropic.Models.Messages;

  AnthropicClient client = new();

  var parameters = new MessageCreateParams
  {
      Model = Model.ClaudeOpus5_5,
      MaxTokens = 1024,
      Messages = [new UserMessageParam { Content = "Hello, Claude" }]
  };

  try
  {
      await foreach (var streamEvent in client.Messages.CreateStreaming(parameters))
      {
          if (
              streamEvent.TryPickContentBlockDelta(out var deltaEvent)
              && deltaEvent.Delta.TryPickText(out var textDelta)
          )
          {
              Console.Write(textDelta.Text);
          }
      }
  }
  // AnthropicSseException（错误事件）
  // 和 AnthropicApiException（错误响应）的基类
  catch (AnthropicServiceException e) when (e.Type?.Raw() == "overloaded_error")
  {
      // 一种 error.type：overloaded_error
      Console.WriteLine($"\nOverloaded: {e.ApiMessage}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  stream := client.Messages.NewStreaming(context.TODO(), anthropic.MessageNewParams{
  	Model:     anthropic.ModelClaudeOpus5_5,
  	MaxTokens: 1024,
  	Messages: []anthropic.MessageParam{
  		anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  	},
  })

  for stream.Next() {
  	event := stream.Current()
  	switch eventVariant := event.AsAny().(type) {
  	case anthropic.ContentBlockDeltaEvent:
  		switch deltaVariant := eventVariant.Delta.AsAny().(type) {
  		case anthropic.TextDelta:
  			fmt.Print(deltaVariant.Text)
  		}
  	}
  }

  err := stream.Err()
  var apierr *anthropic.Error
  if errors.As(err, &apierr) && apierr.Type() == anthropic.ErrorTypeOverloadedError {
  	// 一种 error.type：overloaded_error
  	fmt.Println("\nOverloaded:", apierr.Message)
  } else if err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.client.AnthropicClient;
  import com.anthropic.client.okhttp.AnthropicOkHttpClient;
  import com.anthropic.errors.AnthropicServiceException;
  import com.anthropic.models.ErrorType;
  import com.anthropic.models.messages.MessageCreateParams;
  import com.anthropic.models.messages.Model;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      MessageCreateParams params = MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024L)
          .addUserMessage("Hello, Claude")
          .build();

      try (var streamResponse = client.messages().createStreaming(params)) {
          streamResponse.stream().forEach(event -> {
              event.contentBlockDelta().ifPresent(deltaEvent ->
                  deltaEvent.delta().text().ifPresent(textDelta ->
                      IO.print(textDelta.text())
                  )
              );
          });
      } catch (AnthropicServiceException e) {
          // 捕获错误事件（SseException）和错误响应
          if (ErrorType.OVERLOADED_ERROR.equals(e.type().orElse(null))) {
              // 一种 error.type：overloaded_error
              IO.println("\nOverloaded: " + e.apiMessage().orElse(""));
          } else {
              throw e;
          }
      }
  }
  ```

  ```php PHP
  use Anthropic\Core\Exceptions\APIStatusException;
  use Anthropic\Messages\Model;
  use Anthropic\Messages\RawContentBlockDeltaEvent;
  use Anthropic\Messages\TextDelta;

  $client = new Client();

  try {
      $stream = $client->messages->createStream(
          model: Model::CLAUDE_OPUS_5_5,
          maxTokens: 1024,
          messages: [['role' => 'user', 'content' => 'Hello, Claude']],
      );

      foreach ($stream as $event) {
          if (
              $event instanceof RawContentBlockDeltaEvent
              && $event->delta instanceof TextDelta
          ) {
              echo $event->delta->text;
          }
      }
  } catch (APIStatusException $e) {
      if ($e->getType() === 'overloaded_error') {
          // 一种 error.type：overloaded_error
          echo PHP_EOL, 'Overloaded: ', $e->getAPIMessage(), PHP_EOL;
      } else {
          throw $e;
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  begin
    stream = client.messages.stream(
      model: Anthropic::Model::CLAUDE_OPUS_5_5,
      max_tokens: 1024,
      messages: [{ role: "user", content: "Hello, Claude" }]
    )

    stream.text.each { |text| print(text) }
  rescue Anthropic::Errors::APIStatusError => e
    if e.type == :overloaded_error
      # 一种 error.type：overloaded_error
      puts "\nOverloaded: #{e.api_message}"
    else
      raise
    end
  end
  ```
</CodeGroup>

错误事件也可能带有[错误代码](https://platform.claude.com/docs/zh-CN/api/errors#error-codes)。请从 `error_code` (typescript: `errorCode`; csharp: `ErrorCode`; java: `errorCode().orElse(null)`; php: `getErrorCode()`; go: `ErrorCode()`) 中读取它，并将其与您要处理的代码进行比较，例如 `"enforced_spend_limit_reached"` (ruby: `:enforced_spend_limit_reached`; go: `anthropic.ErrorCodeEnforcedSpendLimitReached`)。

流还可能以另外两种方式失败：

* **请求在流开始之前失败：** 您会收到[按类型捕获错误](https://platform.claude.com/docs/zh-CN/api/handle-errors#catch-errors-by-kind)中的错误。
* **连接在流传输中途断开：** 不会有错误事件到达，并且[按类型捕获错误](https://platform.claude.com/docs/zh-CN/api/handle-errors#catch-errors-by-kind)中的连接分支可能无法捕获该错误。要了解由什么捕获它，请参阅 [SDK 错误类型](https://platform.claude.com/docs/zh-CN/api/errors#sdk-error-types)。

要在流失败后继续响应，请参阅[错误恢复](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming#error-recovery)。
