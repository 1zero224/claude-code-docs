---
title: Go SDK
url: https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/go
description: 安装和配置 Anthropic Go SDK，支持基于 context 的取消和函数式选项
---

Anthropic Go 库为使用 Go 编写的应用程序提供了便捷访问 Claude API 的方式。

<Info>
  有关包含代码示例的 API 功能文档，请参阅 [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)。本页面介绍 Go 特有的 SDK 功能和配置。
</Info>

## 安装

```go
import (
	"github.com/anthropics/anthropic-sdk-go/v2" // imported as anthropic
)
```

使用 `go get` 安装：

```bash
go get github.com/anthropics/anthropic-sdk-go/v2
```

## 要求

此库需要 Go 1.25+。

## 用法

```go
package main

import (
	"context"
	"fmt"

	"github.com/anthropics/anthropic-sdk-go/v2"
	"github.com/anthropics/anthropic-sdk-go/v2/option"
)

func main() {
	client := anthropic.NewClient(
		option.WithAPIKey("my-anthropic-api-key"), // defaults to os.LookupEnv("ANTHROPIC_API_KEY")
	)
	message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
		MaxTokens: 1024,
		Messages: []anthropic.MessageParam{
			anthropic.NewUserMessage(anthropic.NewTextBlock("What is a quaternion?")),
		},
		Model: anthropic.ModelClaudeOpus5_5,
	})
	if err != nil {
		panic(err.Error())
	}
	for _, block := range message.Content {
		if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
			fmt.Println(textBlock.Text)
		}
	}
}
```

有关包括 Workload Identity Federation（工作负载身份联合）在内的身份验证选项，请参阅[身份验证](https://platform.claude.com/docs/zh-CN/manage-claude/authentication)。如果您的 API 密钥是未限定到某个工作区的[个人或服务账户密钥](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#key-types)，请在 `anthropic-workspace-id` 请求标头中设置工作区 ID；[选择工作区](https://platform.claude.com/docs/zh-CN/manage-claude/authentication#select-a-workspace)介绍了此 SDK 的按请求设置选项。

<AccordionGroup>
  <Accordion title="对话">
    ```go
    messages := []anthropic.MessageParam{
    	anthropic.NewUserMessage(anthropic.NewTextBlock("What is my first name?")),
    }

    message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    	Model:     anthropic.ModelClaudeOpus5_5,
    	Messages:  messages,
    	MaxTokens: 1024,
    })
    if err != nil {
    	panic(err)
    }

    fmt.Printf("%+v\n", message.Content)

    messages = append(messages, message.ToParam())
    messages = append(messages, anthropic.NewUserMessage(
    	anthropic.NewTextBlock("My full name is John Doe"),
    ))

    message, err = client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    	Model:     anthropic.ModelClaudeOpus5_5,
    	Messages:  messages,
    	MaxTokens: 1024,
    })
    if err != nil {
    	panic(err)
    }

    fmt.Printf("%+v\n", message.Content)
    ```
  </Accordion>

  <Accordion title="系统提示">
    ```go
    message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    	Model:     anthropic.ModelClaudeOpus5_5,
    	MaxTokens: 1024,
    	System: []anthropic.TextBlockParam{
    		{Text: "Be very serious at all times."},
    	},
    	Messages: messages,
    })
    if err != nil {
    	panic(err)
    }
    fmt.Printf("%+v\n", message.Content)
    ```
  </Accordion>

  <Accordion title="流式传输">
    ```go
    content := "What is a quaternion?"

    stream := client.Messages.NewStreaming(context.TODO(), anthropic.MessageNewParams{
    	Model:     anthropic.ModelClaudeOpus5_5,
    	MaxTokens: 1024,
    	Messages: []anthropic.MessageParam{
    		anthropic.NewUserMessage(anthropic.NewTextBlock(content)),
    	},
    })

    message := anthropic.Message{}
    for stream.Next() {
    	event := stream.Current()
    	err := message.Accumulate(event)
    	if err != nil {
    		panic(err)
    	}

    	switch eventVariant := event.AsAny().(type) {
    	case anthropic.ContentBlockDeltaEvent:
    		switch deltaVariant := eventVariant.Delta.AsAny().(type) {
    		case anthropic.TextDelta:
    			print(deltaVariant.Text)
    		}

    	}
    }

    if stream.Err() != nil {
    	panic(stream.Err())
    }
    ```
  </Accordion>

  <Accordion title="工具调用">
    ```go
    messages := []anthropic.MessageParam{
    	anthropic.NewUserMessage(anthropic.NewTextBlock(content)),
    }

    toolParams := []anthropic.ToolParam{
    	{
    		Name:        "get_coordinates",
    		Description: anthropic.String("Accepts a place as an address, then returns the latitude and longitude coordinates."),
    		InputSchema: GetCoordinatesInputSchema,
    	},
    }
    tools := make([]anthropic.ToolUnionParam, len(toolParams))
    for i, toolParam := range toolParams {
    	tools[i] = anthropic.ToolUnionParam{OfTool: &toolParam}
    }

    for {
    	message, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    		Model:     anthropic.ModelClaudeOpus5_5,
    		MaxTokens: 1024,
    		Messages:  messages,
    		Tools:     tools,
    	})

    	if err != nil {
    		panic(err)
    	}

    	print(color("[assistant]: "))
    	for _, block := range message.Content {
    		switch block := block.AsAny().(type) {
    		case anthropic.TextBlock:
    			println(block.Text)
    			println()
    		case anthropic.ToolUseBlock:
    			inputJSON, _ := json.Marshal(block.Input)
    			println(block.Name + ": " + string(inputJSON))
    			println()
    		}
    	}

    	messages = append(messages, message.ToParam())
    	toolResults := []anthropic.ContentBlockParamUnion{}

    	for _, block := range message.Content {
    		switch variant := block.AsAny().(type) {
    		case anthropic.ToolUseBlock:
    			print(color("[user (" + block.Name + ")]: "))

    			var response interface{}
    			switch block.Name {
    			case "get_coordinates":
    				var input struct {
    					Location string `json:"location"`
    				}

    				err := json.Unmarshal([]byte(variant.JSON.Input.Raw()), &input)
    				if err != nil {
    					panic(err)
    				}

    				response = GetCoordinates(input.Location)
    			}

    			b, err := json.Marshal(response)
    			if err != nil {
    				panic(err)
    			}

    			println(string(b))

    			toolResults = append(toolResults, anthropic.NewToolResultBlock(block.ID, string(b), false))
    		}

    	}
    	if len(toolResults) == 0 {
    		break
    	}
    	messages = append(messages, anthropic.NewUserMessage(toolResults...))
    }
    ```
  </Accordion>
</AccordionGroup>

## 请求字段

Anthropic 库对请求字段使用 Go 1.24+ `encoding/json` 版本中的 [`omitzero`](https://tip.golang.org/doc/go1.24#encodingjsonpkgencodingjson) 语义。

必需的基本类型字段（例如 `int64` 或 `string`）带有标签 `` `json:"...,required"` ``。这些字段始终会被序列化，即使是零值也是如此。

可选的基本类型被包装在 `param.Opt[T]` 中。这些字段可以使用提供的构造函数进行设置，例如 `anthropic.String(string)` 或 `anthropic.Int(int64)`。

任何 `param.Opt[T]`、map、slice、struct 或字符串枚举都使用标签 `` `json:"...,omitzero"` ``。其零值被视为省略。

`param.IsOmitted(any)` 函数可以确认任何 `omitzero` 字段是否存在。

```go
p := anthropic.ExampleParams{
	ID:   "id_xxx",                // required property
	Name: anthropic.String("..."), // optional property

	Point: anthropic.Point{
		X: 0,                // required field will serialize as 0
		Y: anthropic.Int(1), // optional field will serialize as 1
		// ... 省略的非必需字段将不会被序列化
	},

	Origin: anthropic.Origin{}, // the zero value of [Origin] is considered omitted
}
```

要发送 `null` 而不是 `param.Opt[T]`，请使用 `param.Null[T]()`。 要发送 `null` 而不是结构体 `T`，请使用 `param.NullStruct[T]()`。

```go
p.Name = param.Null[string]()       // 'null' instead of string
p.Point = param.NullStruct[Point]() // 'null' instead of struct

param.IsNull(p.Name)  // true
param.IsNull(p.Point) // true
```

请求结构体包含一个 `.SetExtraFields(map[string]any)` 方法，可以在请求体中发送不符合规范的 字段。额外字段会覆盖任何具有匹配 键的结构体字段。

<Warning>
  出于安全原因，请仅对受信任的数据使用 `SetExtraFields`。
</Warning>

要发送自定义值而不是结构体，请使用泛型函数 `param.Override`（例如 `param.Override[anthropic.FooParams](12)`）。

```go
// 如果 API 指定了某个类型，
// 但您想发送其他内容，请使用 [SetExtraFields]：
p.SetExtraFields(map[string]any{
	"x": 0.01, // send "x" as a float instead of int
})

// 发送数字而非对象
custom := param.Override[anthropic.FooParams](12)
```

### 请求联合类型

联合类型（union）表示为一个结构体，其每个变体对应一个以 “Of” 为前缀的字段。 只能有一个字段为非零值。非零字段会被序列化。

可以通过联合结构体上的方法访问联合类型的子属性。 如果值存在，这些方法会返回指向该值的指针。

```go
// 只能有一个字段为非零值，请使用 param.IsOmitted() 检查字段是否已设置
type AnimalUnionParam struct {
	OfCat *Cat `json:",omitzero,inline"`
	OfDog *Dog `json:",omitzero,inline"`
}

animal := AnimalUnionParam{
	OfCat: &Cat{
		Name: "Whiskers",
		Owner: PersonParam{
			Address: AddressParam{Street: "3333 Coyote Hill Rd", ZipCode: 0},
		},
	},
}

// 修改字段
if address := animal.GetOwner().GetAddress(); address != nil {
	address.ZipCode = 94304
}
```

### 反序列化参数

<Note>
  `param.SetJSON` 需要 SDK v1.20.0 或更高版本。
</Note>

参数类型（以 `Param` 结尾的类型，例如 `MessageNewParams` 或 `ToolUnionParam`）仅设计用于发出的请求。它们可以正确地序列化为 JSON，但不完全支持往返反序列化。如果您将原始 JSON 反序列化到参数结构体中，即使底层 JSON 有效，`OfBashTool20250124` 等类型化联合字段也将为 nil。

如果您需要从原始 JSON（例如来自数据库、中间件或之前的请求）重建参数，请调用 `UnmarshalJSON` 来填充非联合字段，然后使用 `param.SetJSON` 附加原始字节以实现正确的重新序列化：

```go
// 序列化 params（例如用于存储或转发）
b, err := json.Marshal(original)
if err != nil {
	panic(err)
}

// 之后，从存储的 JSON 重建 params
var params anthropic.MessageNewParams
if err := params.UnmarshalJSON(b); err != nil {
	panic(err)
}
param.SetJSON(b, &params)

// params.Model 及其他标量字段由 UnmarshalJSON 填充。
// params.Tools[0].OfBashTool20250124 为 nil（联合类型的限制），
// 但原始 JSON 会被保留。当再次序列化 params
// 以进行 API 调用时，tools 会被正确序列化。
b2, _ := json.Marshal(params)
fmt.Println(string(b) == string(b2)) // true
```

对于此用例，`param.SetJSON`（自 v1.20.0 起可用）优于更通用的 `param.Override[T](any)`，因为它不需要显式写出类型参数，并且使往返意图更加明确。

## 响应对象

响应结构体中的所有字段都是普通值类型（不是指针或包装器）。 响应结构体还包含一个特殊的 `JSON` 字段，其中包含有关每个属性的元数据。

```go
type Animal struct {
	Name   string `json:"name,nullable"`
	Owners int    `json:"owners"`
	Age    int    `json:"age"`
	JSON   struct {
		Name        respjson.Field
		Owners      respjson.Field
		Age         respjson.Field
		ExtraFields map[string]respjson.Field
	} `json:"-"`
}
```

要处理可选数据，请在 JSON 字段上使用 `.Valid()` 方法。 当字段存在、非 `null` 且已成功反序列化时，`.Valid()` 返回 true。

如果 `.Valid()` 为 false，则相应字段将为其零值。

```go
raw := `{"owners": 1, "name": null}`

var res Animal
json.Unmarshal([]byte(raw), &res)

// 访问常规字段

res.Owners // 1
res.Name   // ""
res.Age    // 0

// 可选字段检查

res.JSON.Owners.Valid() // true
res.JSON.Name.Valid()   // false
res.JSON.Age.Valid()    // false

// 原始 JSON 值

res.JSON.Owners.Raw()                  // "1"
res.JSON.Name.Raw() == "null"          // true
res.JSON.Name.Raw() == respjson.Null   // true
res.JSON.Age.Raw() == ""               // true
res.JSON.Age.Raw() == respjson.Omitted // true
```

这些 `.JSON` 结构体还包含一个 `ExtraFields` map，其中包含 JSON 响应中未在结构体中指定的任何属性。这对于 SDK 中尚未提供的 API 功能很有用。

```go
body := res.JSON.ExtraFields["my_unexpected_field"].Raw()
```

### 响应联合类型

在响应中，联合类型表示为一个扁平化的结构体，包含每个对象变体的所有可能字段。 要将其转换为某个变体，请使用 `.AsFooVariant()` 方法，或者在存在时使用 `.AsAny()` 方法。

如果响应值联合类型包含基本类型值，则基本类型字段将与属性并列，但以 `Of` 为前缀，并带有标签 `json:"...,inline"`。

```go
type AnimalUnion struct {
	// 来自变体 [Dog]、[Cat]
	Owner Person `json:"owner"`
	// 来自变体 [Dog]
	DogBreed string `json:"dog_breed"`
	// 来自变体 [Cat]
	CatBreed string `json:"cat_breed"`
	// ...

	JSON struct {
		Owner respjson.Field
		// ...
	} `json:"-"`
}

// 如果是 animal 变体
if animal.Owner.Address.ZipCode == "" {
	panic("missing zip code")
}

// 根据变体进行分支
switch variant := animal.AsAny().(type) {
case Dog:
case Cat:
default:
	panic("unexpected type")
}
```

## 错误处理

当 API 返回非成功状态码时，SDK 会返回类型为 `*anthropic.Error` 的错误。其中包含请求的 `StatusCode`、`*http.Request` 和 `*http.Response` 值，以及错误体的 JSON（与 SDK 中的其他响应对象非常相似）。该错误还包含来自响应头的 `RequestID`，这在与 Anthropic 支持团队一起排查问题时很有用。

要处理错误，请使用 `errors.As` 模式：

```go
_, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
	MaxTokens: 1024,
	Messages: []anthropic.MessageParam{
		anthropic.NewUserMessage(anthropic.NewTextBlock("What is a quaternion?")),
	},
	Model: anthropic.ModelClaudeOpus5_5,
})
if err != nil {
	var apierr *anthropic.Error
	if errors.As(err, &apierr) {
		println("Request ID:", apierr.RequestID)
		println(string(apierr.DumpRequest(true)))  // Prints the serialized HTTP request
		println(string(apierr.DumpResponse(true))) // Prints the serialized HTTP response
	}
	panic(err.Error()) // POST "/v1/messages": 400 Bad Request (Request-ID: req_xxx) { ... }
}
```

当发生其他错误时，它们会以未包装的形式返回；例如，如果 HTTP 传输失败，您可能会收到包装了 `*net.OpError` 的 `*url.Error`。

## 重试

默认情况下，某些错误会自动重试 2 次，并采用短暂的指数退避。 SDK 默认会重试所有连接错误、408 Request Timeout、409 Conflict、429 Rate Limit 以及 >=500 Internal 错误。

您可以使用 `WithMaxRetries` 选项来配置或禁用此功能：

```go
// 为所有请求配置默认值：
client := anthropic.NewClient(
	option.WithMaxRetries(0), // default is 2
)

// 按请求覆盖：
// ...
	client.Messages.New(
		context.TODO(),
		anthropic.MessageNewParams{
			MaxTokens: 1024,
			Messages: []anthropic.MessageParam{
				anthropic.NewUserMessage(anthropic.NewTextBlock("What is a quaternion?")),
			},
			Model: anthropic.ModelClaudeOpus5_5,
		},
		option.WithMaxRetries(5),
	)
```

## 超时

默认情况下，非流式 Messages 请求在 10 分钟后超时；其他请求没有默认超时。使用 context 为请求生命周期配置超时。

请注意，如果请求被[重试](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/go#retries)，context 超时不会重新开始计时。 要设置每次重试的超时，请使用 `option.WithRequestTimeout()`。

```go
// 此项设置请求的超时时间，包括所有重试。
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Minute)
defer cancel()
// ...
	client.Messages.New(
		ctx,
		anthropic.MessageNewParams{
			MaxTokens: 1024,
			Messages: []anthropic.MessageParam{
				anthropic.NewUserMessage(anthropic.NewTextBlock("What is a quaternion?")),
			},
			Model: anthropic.ModelClaudeOpus5_5,
		},
		// 此项设置每次重试的超时时间
		option.WithRequestTimeout(20*time.Second),
	)
```

## 长时间请求

<Warning>
  对于运行时间较长的请求，请考虑使用流式 Messages API。
</Warning>

避免在不使用流式传输的情况下设置较大的 `MaxTokens` 值，因为某些网络可能会在一段时间后断开空闲连接，这 可能导致请求失败或在未收到 Anthropic 响应的情况下[超时](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/go#timeouts)。

如果预计非流式请求的时长超过大约 10 分钟，此 SDK 也会返回错误。 调用 `.Messages.NewStreaming()` 或[设置自定义超时](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/sdks/go#timeouts)可禁用此错误。

## 文件上传

与 multipart 请求中的文件上传相对应的请求参数的类型为 `io.Reader`。默认情况下，`io.Reader` 的内容会作为 multipart 表单 部分发送，文件名为 `anonymous_file`，内容类型为 `application/octet-stream`。因此，推荐的做法是使用 `anthropic.File(reader io.Reader, filename string, contentType string)` 辅助函数指定自定义内容类型。此辅助函数会为任何 `io.Reader` 包装上适当的文件名和内容类型。

```go
// 来自文件系统的文件
file, err := os.Open("/path/to/file.json")
anthropic.FileUploadParams{
	File: anthropic.File(file, "custom-name.json", "application/json"),
}

// 来自字符串的文件
anthropic.FileUploadParams{
	File: anthropic.File(strings.NewReader("my file contents"), "custom-name.json", "application/json"),
}
```

也可以通过在 `io.Reader` 的运行时类型上实现 `Name() string` 或 `ContentType() string` 来自定义文件名和内容类型。请注意，`os.File` 实现了 `Name() string`，因此 由 `os.Open` 返回的文件会使用磁盘上的文件名发送。

## 分页

此库为处理分页列表端点提供了一些便利功能。

您可以使用 `.ListAutoPaging()` 方法遍历所有页面中的项目：

```go
iter := client.Messages.Batches.ListAutoPaging(context.TODO(), anthropic.MessageBatchListParams{
	Limit: anthropic.Int(20),
})
// 根据需要自动获取更多页面。
for iter.Next() {
	messageBatch := iter.Current()
	fmt.Println(messageBatch.ID)
}
if err := iter.Err(); err != nil {
	panic(err.Error())
}
```

或者，您可以使用简单的 `.List()` 方法获取单个页面，并接收一个标准响应对象， 其中包含 `.GetNextPage()` 等额外的辅助方法：

```go
page, err := client.Messages.Batches.List(context.TODO(), anthropic.MessageBatchListParams{
	Limit: anthropic.Int(20),
})
for page != nil {
	for _, batch := range page.Data {
		fmt.Println(batch.ID)
	}
	page, err = page.GetNextPage()
}
if err != nil {
	panic(err.Error())
}
```

## RequestOptions

此库使用函数式选项模式。`option` 包中定义的函数返回一个 `RequestOption`，它是一个修改 `RequestConfig` 的闭包。这些选项可以提供给客户端，也可以提供给单个请求。例如：

```go
client := anthropic.NewClient(
	// 为客户端发出的每个请求添加一个请求头
	option.WithHeader("X-Some-Header", "custom_header_info"),
)

client.Messages.New(context.TODO(), // ...,
	// 覆盖该请求头
	option.WithHeader("X-Some-Header", "some_other_custom_header_info"),
	// 使用 sjson 语法向请求体添加一个未记录在文档中的字段
	option.WithJSONSet("some.json.path", map[string]string{"my": "object"}),
)
```

请求选项 `option.WithDebugLog(nil)` 在调试时可能会有所帮助。

请参阅[完整的请求选项列表](https://pkg.go.dev/github.com/anthropics/anthropic-sdk-go/v2/option)。

## HTTP 客户端自定义

有关请求中间件（`option.WithMiddleware`）以及替换默认 `http.Client`（`option.WithHTTPClient`）的信息，请参阅 [SDK 中间件](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/middleware)。

## 平台集成

<Note>
  有关包含代码示例的详细平台设置指南，请参阅：

  * [Amazon Bedrock（`bedrock-mantle`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-amazon-bedrock)
  * [Amazon Bedrock（`bedrock-runtime`）](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-on-amazon-bedrock-legacy)
  * [Claude Platform on AWS](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-platform-on-aws)
  * [Claude Models on Google Cloud (Google managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/google-managed)
  * [Microsoft Foundry](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-in-microsoft-foundry)

  另请参阅 [Claude Models on Google Cloud (Anthropic managed)](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed)。
</Note>

Go SDK 支持以下平台：

* **Claude Models on Google Cloud (Google managed)：** `import "github.com/anthropics/anthropic-sdk-go/v2/vertex"`。使用 `vertex.WithGoogleAuth(ctx, region, projectID)` 或 `vertex.WithCredentials(ctx, region, projectID, creds)`。

* **Bedrock：** `import "github.com/anthropics/anthropic-sdk-go/v2/bedrock"`。使用哪种方式取决于端点：

  * **`bedrock-mantle`：** `bedrock.NewMantleClient`。它通过 SSE 进行流式传输。
  * **`bedrock-runtime`：** `bedrock.WithLoadDefaultConfig(ctx)` 或 `bedrock.WithConfig(cfg)`，用于 `InvokeModel` API。

* **Claude Platform on AWS：** `import anthropicaws "github.com/anthropics/anthropic-sdk-go/v2/aws"`。使用 `anthropicaws.NewClient(ctx, cfg)` 和一个 `anthropicaws.ClientConfig` 值来构造客户端；在配置中设置 `WorkspaceID`，或设置 `ANTHROPIC_AWS_WORKSPACE_ID` 环境变量。当同时导入 `github.com/aws/aws-sdk-go-v2/aws` 时，`anthropicaws` 导入别名可避免名称冲突。以 beta 形式提供。

* **Foundry：** `import "github.com/anthropics/anthropic-sdk-go/v2/foundry"`。使用 `foundry.NewClient(cfg)` 和一个 `foundry.ClientConfig` 值来构造客户端；在配置中设置 `APIKey`（对于 Microsoft Entra ID 则设置 `AzureADTokenProvider`）和 `Resource`，或设置 `ANTHROPIC_FOUNDRY_API_KEY` 和 `ANTHROPIC_FOUNDRY_RESOURCE` 环境变量。

Go SDK 还通过 `github.com/anthropics/anthropic-sdk-go/v2/googlecloud` 包支持 Claude Models on Google Cloud (Anthropic managed)。要构造客户端，请使用一个 `googlecloud.ClientConfig` 值调用 `googlecloud.NewClient(ctx, cfg)`。此客户端处于 beta 阶段。在配置中设置 `WorkspaceID`，或设置 `ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID` 环境变量。有关客户端如何解析工作区 ID，请参阅[项目、位置和工作区解析](https://platform.claude.com/docs/zh-CN/build-with-claude/claude-models-on-google-cloud/anthropic-managed#project-location-and-workspace-resolution)。

## 高级用法

### 访问原始响应数据（例如响应头）

您可以使用 `option.WithResponseInto()` 请求选项访问原始 HTTP 响应数据。当您需要检查响应头、状态码或其他详细信息时，这非常有用。

```go
// 创建一个变量来存储 HTTP 响应
var response *http.Response
message, err := client.Messages.New(
	context.TODO(),
	anthropic.MessageNewParams{
		MaxTokens: 1024,
		Messages: []anthropic.MessageParam{
			anthropic.NewUserMessage(anthropic.NewTextBlock("What is a quaternion?")),
		},
		Model: anthropic.ModelClaudeOpus5_5,
	},
	option.WithResponseInto(&response),
)
if err != nil {
	// 处理错误
}
fmt.Printf("%+v\n", message.Content)

fmt.Printf("Status Code: %d\n", response.StatusCode)
fmt.Printf("Headers: %+#v\n", response.Header)
```

### OpenTelemetry 追踪

当您的应用程序注册 OpenTelemetry tracer provider 时，SDK 会为其发送的每个请求发出一个 span。要将这些 span 发送到您的可观测性平台，请参阅[在 Anthropic SDK 中设置 OpenTelemetry](https://platform.claude.com/docs/zh-CN/manage-claude/opentelemetry/sdk)。

### 发出自定义/未记录的请求

此库具有类型定义，便于访问已记录的 API。如果您需要访问未记录的端点、参数或响应属性，仍然可以使用此库。

#### 未记录的端点

要向未记录的端点发出请求，您可以使用 `client.Get`、`client.Post` 和其他 HTTP 动词。 发出这些请求时，客户端上的 `RequestOptions`（例如重试）会被遵循。

```go
var (
	// params 可以是 io.Reader、[]byte、可由 encoding/json 序列化的对象，
	// 或本库中定义的 "...Params" 结构体。
	params map[string]any

	// result 可以是 []byte、*http.Response、可由 encoding/json 反序列化的对象，
	// 或本库中定义的模型。
	result *http.Response
)
err := client.Post(context.Background(), "/unspecified", params, &result)
if err != nil {
	// ...
}
```

#### 未记录的请求参数

要使用未记录的参数发出请求，您可以使用 `option.WithQuerySet()` 或 `option.WithJSONSet()` 函数。

```go
params := FooNewParams{
	ID: "id_xxxx",
	Data: FooNewParamsData{
		FirstName: anthropic.String("John"),
	},
}
client.Foo.New(context.Background(), params, option.WithJSONSet("data.last_name", "Doe"))
```

#### 未记录的响应属性

要访问未记录的响应属性，您可以使用 `result.JSON.RawJSON()` 以字符串形式访问响应的原始 JSON， 或者使用 `result.JSON.Foo.Raw()` 获取结果中特定字段的原始 JSON。

响应结构体中不存在的任何字段都会被保存，并可以通过 `result.JSON.ExtraFields` 访问，它是一个 `map[string]respjson.Field`。

## 语义化版本控制

此软件包通常遵循 [SemVer](https://semver.org/spec/v2.0.0.html) 约定，但某些向后不兼容的更改可能会作为次要版本发布：

1. 对库内部的更改，这些内部内容在技术上是公开的，但并非有意或有文档说明供外部使用。
2. 预计在实践中不会影响绝大多数用户的更改。

我们非常重视向后兼容性，以确保您可以获得顺畅的升级体验。

对 Claude API 的每个请求都使用一个 API 版本。使用此 SDK 时，您的请求会使用 SDK 主版本发布时的当前 API 版本，因此您需要通过升级到新的主版本来迁移到新的 API 版本。版本 2.0.0 是首个使用 `2026-10-21` 的版本。其模块路径为 `github.com/anthropics/anthropic-sdk-go/v2`。有关如何升级，请参阅[迁移到 API 版本 2026-10-21](https://platform.claude.com/docs/zh-CN/api/migration-2026-10-21#upgrade)。

欢迎您提供反馈；如有问题、错误或建议，请提交 [issue](https://github.com/anthropics/anthropic-sdk-go/issues)。

## 其他资源

* [GitHub 仓库](https://github.com/anthropics/anthropic-sdk-go)
* [Go 包文档](https://pkg.go.dev/github.com/anthropics/anthropic-sdk-go/v2)
* [API 参考](https://platform.claude.com/docs/zh-CN/api/overview)
* [流式 Messages](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)
