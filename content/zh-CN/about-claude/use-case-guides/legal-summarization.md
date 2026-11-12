---
title: 法律文件摘要
url: https://platform.claude.com/docs/zh-CN/about-claude/use-case-guides/legal-summarization
description: 本指南介绍如何利用 Claude 先进的自然语言处理能力高效地总结法律文档，提取关键信息并加快法律研究。借助 Claude，您可以简化合同审查、诉讼准备和监管工作，在法律流程中节省时间并确保准确性。
---

> 访问 Claude Cookbook 中的[摘要示例](https://platform.claude.com/cookbook/capabilities-summarization-guide)，查看使用 Claude 实现法律文档摘要的示例。

## 使用 Claude 构建之前

### 决定是否使用 Claude 进行法律文件摘要

以下是一些关键指标，表明您应该使用 Claude 这样的"large language model"（大型语言模型），即 LLM 来对法律文件进行摘要：

<AccordionGroup>
  <Accordion title="您希望高效且经济地审查大量文档">
    手动进行大规模文档审查既耗时又昂贵。Claude 可以快速处理和总结大量法律文档，显著降低文档审查相关的时间和成本。这一能力对于尽职调查、合同分析或诉讼证据开示等对效率要求很高的任务尤其有价值。
  </Accordion>

  <Accordion title="您需要自动提取关键元数据">
    Claude 可以高效地从法律文档中提取重要元数据并进行分类，例如涉及的各方、日期、合同条款或特定条款。这种自动提取有助于整理信息，使大型文档集的搜索、分析和管理变得更加容易。它对于合同管理、合规检查或创建可搜索的法律信息数据库尤其有用。
  </Accordion>

  <Accordion title="您希望生成清晰、简洁且标准化的摘要">
    Claude 可以生成遵循预定格式的结构化摘要，使法律专业人士更容易快速掌握各类文件的要点。这些标准化摘要可以提高可读性，便于文件之间的比较，并增强整体理解，尤其是在处理复杂的法律语言或技术术语时。
  </Accordion>

  <Accordion title="您的摘要需要精确的引用">
    在创建法律摘要时，适当的出处说明和引用对于确保可信度和符合法律标准至关重要。可以通过提示让 Claude 为所有引用的法律要点提供准确的引用，使法律专业人士更容易审查和核实摘要信息。
  </Accordion>

  <Accordion title="您希望简化并加快法律研究流程">
    Claude 可以通过快速分析大量判例法、成文法和法律评论来协助法律研究。它可以识别相关先例、提取关键法律原则，并总结复杂的法律论证。这一能力可以显著加快研究进程，使法律专业人士能够专注于更高层次的分析和策略制定。
  </Accordion>
</AccordionGroup>

### 确定您希望摘要提取的细节

对于任何给定的文件，都不存在唯一正确的摘要。如果没有明确的指引，Claude 可能难以确定应包含哪些细节。为了获得最佳结果，请确定您希望在摘要中包含的具体信息。

例如，在对转租协议进行摘要时，您可能希望提取以下要点：

<CodeGroup exclude="shell">
  ```python Python
  details_to_extract = [
      "Parties involved (sublessor, sublessee, and original lessor)",
      "Property details (address, description, and permitted use)",
      "Term and rent (start date, end date, monthly rent, and security deposit)",
      "Responsibilities (utilities, maintenance, and repairs)",
      "Consent and notices (landlord's consent, and notice requirements)",
      "Special provisions (furniture, parking, and subletting restrictions)",
  ]
  ```

  ```typescript TypeScript
  const detailsToExtract = [
    "Parties involved (sublessor, sublessee, and original lessor)",
    "Property details (address, description, and permitted use)",
    "Term and rent (start date, end date, monthly rent, and security deposit)",
    "Responsibilities (utilities, maintenance, and repairs)",
    "Consent and notices (landlord's consent, and notice requirements)",
    "Special provisions (furniture, parking, and subletting restrictions)"
  ];
  ```

  ```csharp C#
  List<string> detailsToExtract =
  [
      "Parties involved (sublessor, sublessee, and original lessor)",
      "Property details (address, description, and permitted use)",
      "Term and rent (start date, end date, monthly rent, and security deposit)",
      "Responsibilities (utilities, maintenance, and repairs)",
      "Consent and notices (landlord's consent, and notice requirements)",
      "Special provisions (furniture, parking, and subletting restrictions)",
  ];
  ```

  ```go Go
  var detailsToExtract = []string{
  	"Parties involved (sublessor, sublessee, and original lessor)",
  	"Property details (address, description, and permitted use)",
  	"Term and rent (start date, end date, monthly rent, and security deposit)",
  	"Responsibilities (utilities, maintenance, and repairs)",
  	"Consent and notices (landlord's consent, and notice requirements)",
  	"Special provisions (furniture, parking, and subletting restrictions)",
  }
  ```

  ```java Java
  List<String> detailsToExtract = List.of(
      "Parties involved (sublessor, sublessee, and original lessor)",
      "Property details (address, description, and permitted use)",
      "Term and rent (start date, end date, monthly rent, and security deposit)",
      "Responsibilities (utilities, maintenance, and repairs)",
      "Consent and notices (landlord's consent, and notice requirements)",
      "Special provisions (furniture, parking, and subletting restrictions)");
  ```

  ```php PHP
  $detailsToExtract = [
      'Parties involved (sublessor, sublessee, and original lessor)',
      'Property details (address, description, and permitted use)',
      'Term and rent (start date, end date, monthly rent, and security deposit)',
      'Responsibilities (utilities, maintenance, and repairs)',
      "Consent and notices (landlord's consent, and notice requirements)",
      'Special provisions (furniture, parking, and subletting restrictions)',
  ];
  ```

  ```ruby Ruby
  details_to_extract = [
    "Parties involved (sublessor, sublessee, and original lessor)",
    "Property details (address, description, and permitted use)",
    "Term and rent (start date, end date, monthly rent, and security deposit)",
    "Responsibilities (utilities, maintenance, and repairs)",
    "Consent and notices (landlord's consent, and notice requirements)",
    "Special provisions (furniture, parking, and subletting restrictions)"
  ]
  ```
</CodeGroup>

### 建立成功标准

评估摘要质量是一项众所周知的困难任务。与许多其他自然语言处理任务不同，摘要评估通常缺乏明确、客观的指标。这一过程可能非常主观，不同的读者会看重摘要的不同方面。以下是您在评估 Claude 法律文档摘要表现时可能需要考虑的标准。

<AccordionGroup>
  <Accordion title="事实正确性">
    摘要应准确呈现文件中的事实、法律概念和要点。
  </Accordion>

  <Accordion title="法律精确性">
    术语以及对法规、判例法或规章的引用必须正确并符合法律标准。
  </Accordion>

  <Accordion title="简洁性">
    摘要应将法律文件浓缩为其核心要点，同时不丢失重要细节。
  </Accordion>

  <Accordion title="一致性">
    如果对多份文件进行摘要，LLM 应对每份摘要保持一致的结构和方法。
  </Accordion>

  <Accordion title="可读性">
    文本应清晰易懂。如果受众不是法律专家，摘要不应包含可能使受众困惑的法律术语。
  </Accordion>

  <Accordion title="偏见与公正性">
    摘要应对法律论证和立场进行无偏见且公正的描述。
  </Accordion>
</AccordionGroup>

请参阅[建立成功标准](https://platform.claude.com/docs/zh-CN/test-and-evaluate/develop-tests)指南了解更多信息。

***

## 如何使用 Claude 对法律文件进行摘要

### 选择合适的 Claude 模型

在总结法律文档时，模型准确性极其重要。对于此类需要高准确性的用例，Claude Opus 5.5 是绝佳选择。如果您的文档规模和数量较大，以至于成本开始成为问题，您也可以尝试使用较小的模型，例如 Claude Haiku 5.5。

为帮助估算这些成本，以下是使用 Opus 和 Haiku 模型对 1,000 份转租协议进行摘要的成本比较：

* **内容规模**

  * 协议数量：1,000
  * 每份协议的字符数：300,000
  * 总字符数：3 亿

* **估算令牌数**

  * 输入令牌：1.08 亿，每份协议约 108,000 个（约每 2.8 个字符 1 个令牌，这是两个模型在本指南示例协议上表现出的比率）
  * 每份摘要的输出令牌（包括按输出计费的任何思考令牌）：Claude Opus 5.5 约 6,200 个，Claude Haiku 5.5 约 3,100 个
  * 总输出令牌：Claude Opus 5.5 为 620 万，Claude Haiku 5.5 为 310 万

* **Claude Opus 5.5 估算成本**

  * 输入令牌成本：108 MTok \* $4.00/MTok = $432.00 USD
  * 输出令牌成本：6.2 MTok \* $20.00/MTok = $124.00 USD
  * 总成本：$432.00 + $124.00 = $556.00 USD

* **Claude Haiku 5.5 估算成本**，按其超过 100,000 令牌的提示价格计算

  * 输入令牌成本：108 MTok \* $0.50/MTok = $54.00 USD
  * 输出令牌成本：3.1 MTok \* $2.50/MTok = $7.75 USD
  * 总成本：$54.00 + $7.75 = $61.75 USD

<Tip>
  实际成本可能与这些估算有所不同。令牌数来自在本指南的示例协议（约 105,000 个输入令牌）上运行

  [构建强大的提示](https://platform.claude.com/docs/zh-CN/about-claude/use-case-guides/legal-summarization#build-a-strong-prompt)

  部分中的示例，而输出长度会因文档不同以及每次运行而有所变化。此处每份协议是一个约 108,000 个令牌的提示，超过了 Claude Haiku 5.5 

  [长提示价格](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)

  的 100,000 令牌阈值。如果文档的提示保持在 100,000 个令牌或以下，则改为按每百万输入/输出令牌 $0.10 / $0.50 USD 计费。要了解适用哪种价格，请使用

  [令牌计数](https://platform.claude.com/docs/zh-CN/build-with-claude/token-counting)

  并将 

  `model`

   设置为 

  `claude-haiku-5-5`

   来统计您的文档。
</Tip>

### 将文件转换为 Claude 可以处理的格式

在开始总结文档之前，您需要准备数据。这包括从 PDF 中提取文本、清理文本，并确保其已准备好供 Claude 处理。

以下是在示例 PDF 上演示此过程：

<CodeGroup exclude="shell">
  <CodeGroupItem language="cURL">
    在代码中从 PDF 提取文本。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    在代码中从 PDF 提取文本。
  </CodeGroupItem>

  ```python Python
  from io import BytesIO
  import re

  import pypdf
  import requests


  def get_llm_text(pdf_file):
      reader = pypdf.PdfReader(pdf_file)
      text = "\n".join([page.extract_text() for page in reader.pages])

      # 移除页码
      text = re.sub(r"\n\s*\d+\s*\n", "\n", text)

      # 移除多余空白
      text = re.sub(r"\s+", " ", text)

      return text


  # 根据 GitHub 仓库构建完整 URL
  url = "https://raw.githubusercontent.com/anthropics/claude-cookbooks/main/capabilities/summarization/data/Sample Sublease Agreement.pdf"
  url = url.replace(" ", "%20")

  # 将 PDF 文件下载到内存中
  response = requests.get(url)

  # 从内存加载 PDF
  pdf_file = BytesIO(response.content)

  document_text = get_llm_text(pdf_file)
  print(document_text[:50000])
  ```

  ```typescript TypeScript
  import { PDFParse } from "pdf-parse";

  async function getLlmText(pdfData: Uint8Array): Promise<string> {
    const parser = new PDFParse({ data: pdfData });
    const result = await parser.getText();
    await parser.destroy();
    let text = result.pages.map((page) => page.text).join("\n");

    // 删除页码
    text = text.replace(/\n\s*\d+\s*\n/g, "\n");

    // 删除多余空白
    text = text.replace(/\s+/g, " ");

    return text;
  }

  // 从 Claude Cookbook 的摘要示例中下载示例转租协议
  const response = await fetch(
    "https://raw.githubusercontent.com/anthropics/claude-cookbooks/main/capabilities/summarization/data/Sample%20Sublease%20Agreement.pdf"
  );
  const pdfData = new Uint8Array(await response.arrayBuffer());

  const documentText = await getLlmText(pdfData);
  console.log(documentText.slice(0, 50000));
  ```

  ```csharp C#
  using System.Text.RegularExpressions;
  using UglyToad.PdfPig;
  using UglyToad.PdfPig.DocumentLayoutAnalysis.TextExtractor;

  string GetLlmText(byte[] pdfData)
  {
      using var document = PdfDocument.Open(pdfData);
      var text = string.Join("\n", document.GetPages().Select(page => ContentOrderTextExtractor.GetText(page)));

      // 删除页码
      text = Regex.Replace(text, @"\n\s*\d+\s*\n", "\n");

      // 删除多余的空白
      text = Regex.Replace(text, @"\s+", " ");

      return text;
  }

  // 从 Claude Cookbook 的摘要示例中下载示例转租协议
  using var httpClient = new HttpClient();
  var pdfData = await httpClient.GetByteArrayAsync(
      "https://raw.githubusercontent.com/anthropics/claude-cookbooks/main/capabilities/summarization/data/Sample%20Sublease%20Agreement.pdf"
  );

  var documentText = GetLlmText(pdfData);
  Console.WriteLine(documentText[..Math.Min(50000, documentText.Length)]);
  ```

  ```go Go
  import (
  	"bytes"
  	"io"
  	"net/http"
  	"regexp"

  	"github.com/ledongthuc/pdf"
  )

  // 示例转租协议的文本
  var documentText string

  func getLLMText(pdfData []byte) (string, error) {
  	reader, err := pdf.NewReader(bytes.NewReader(pdfData), int64(len(pdfData)))
  	if err != nil {
  		return "", err
  	}

  	var pages []string
  	for pageIndex := 1; pageIndex <= reader.NumPage(); pageIndex++ {
  		pageText, err := reader.Page(pageIndex).GetPlainText(nil)
  		if err != nil {
  			return "", err
  		}
  		pages = append(pages, pageText)
  	}
  	text := strings.Join(pages, "\n")

  	// 删除页码
  	text = regexp.MustCompile(`\n\s*\d+\s*\n`).ReplaceAllString(text, "\n")

  	// 删除多余空白
  	text = regexp.MustCompile(`\s+`).ReplaceAllString(text, " ")

  	return text, nil
  }

  // extractDocumentText 下载示例转租协议并提取其文本。
  func extractDocumentText() {
  	// 从 Claude Cookbook 的摘要示例中下载示例转租协议
  	response, err := http.Get("https://raw.githubusercontent.com/anthropics/claude-cookbooks/main/capabilities/summarization/data/Sample%20Sublease%20Agreement.pdf")
  	if err != nil {
  		log.Fatal(err)
  	}
  	defer response.Body.Close()

  	pdfData, err := io.ReadAll(response.Body)
  	if err != nil {
  		log.Fatal(err)
  	}

  	documentText, err = getLLMText(pdfData)
  	if err != nil {
  		log.Fatal(err)
  	}
  	fmt.Println(documentText[:min(50000, len(documentText))])
  }
  ```

  ```java Java
  import java.io.IOException;
  import java.net.URI;
  import java.net.http.HttpClient;
  import java.net.http.HttpRequest;
  import java.net.http.HttpResponse;
  import org.apache.pdfbox.Loader;
  import org.apache.pdfbox.pdmodel.PDDocument;
  import org.apache.pdfbox.text.PDFTextStripper;

  // 示例转租协议的文本
  String documentText;

  String getLlmText(byte[] pdfData) throws IOException {
      try (PDDocument document = Loader.loadPDF(pdfData)) {
          String text = new PDFTextStripper().getText(document);

          // 删除页码
          text = text.replaceAll("\\n\\s*\\d+\\s*\\n", "\n");

          // 删除多余的空白
          return text.replaceAll("\\s+", " ");
      }
  }

  void extractDocumentText() throws IOException, InterruptedException {
      // 从 Claude Cookbook 的摘要示例中下载示例转租协议
      HttpRequest request = HttpRequest.newBuilder(URI.create(
              "https://raw.githubusercontent.com/anthropics/claude-cookbooks/main/capabilities/summarization/data/Sample%20Sublease%20Agreement.pdf"))
          .build();
      byte[] pdfData;
      try (HttpClient httpClient = HttpClient.newHttpClient()) {
          pdfData = httpClient.send(request, HttpResponse.BodyHandlers.ofByteArray()).body();
      }

      documentText = getLlmText(pdfData);
      IO.println(documentText.substring(0, Math.min(50000, documentText.length())));
  }
  ```

  ```php PHP
  use Smalot\PdfParser\Parser;

  $getLlmText = function (string $pdfData): string {
      $document = (new Parser())->parseContent($pdfData);
      $text = implode("\n", array_map(fn ($page) => $page->getText(), $document->getPages()));

      // 删除页码
      $text = preg_replace('/\n\s*\d+\s*\n/', "\n", $text);

      // 删除多余空白
      return preg_replace('/\s+/', ' ', $text);
  };

  // 从 Claude Cookbook 的摘要示例中下载示例转租协议
  $pdfData = file_get_contents(
      'https://raw.githubusercontent.com/anthropics/claude-cookbooks/main/capabilities/summarization/data/Sample%20Sublease%20Agreement.pdf'
  );

  $documentText = $getLlmText($pdfData);
  echo substr($documentText, 0, 50000) . "\n";
  ```

  ```ruby Ruby
  require "net/http"
  require "pdf-reader"
  require "stringio"

  def get_llm_text(pdf_data)
    reader = PDF::Reader.new(StringIO.new(pdf_data))
    text = reader.pages.map(&:text).join("\n")

    # 删除页码
    text = text.gsub(/\n\s*\d+\s*\n/, "\n")

    # 删除多余空白
    text.gsub(/\s+/, " ")
  end

  # 从 Claude Cookbook 的摘要示例中下载示例转租协议
  pdf_data = Net::HTTP.get(
    URI("https://raw.githubusercontent.com/anthropics/claude-cookbooks/main/capabilities/summarization/data/Sample%20Sublease%20Agreement.pdf")
  )

  document_text = get_llm_text(pdf_data)
  puts document_text[0, 50_000]
  ```
</CodeGroup>

在此示例中，您首先下载 Claude Cookbook 的[摘要示例](https://platform.claude.com/cookbook/capabilities-summarization-guide)中使用的示例转租协议 PDF。该协议来源于 [sec.gov 网站](https://www.sec.gov/Archives/edgar/data/1045425/000119312507044370/dex1032.htm)上一份公开的转租协议。

该示例使用 `pypdf` (typescript: `pdf-parse`; csharp: `PdfPig`; go: `github.com/ledongthuc/pdf`; java: `org.apache.pdfbox:pdfbox`; php: `smalot/pdfparser`; ruby: `pdf-reader`) 库提取 PDF 的内容并将其转换为文本。然后通过删除页码和多余的空白来清理文本数据。

### 构建强大的提示

Claude 可以适应各种摘要风格。您可以更改提示的细节，引导 Claude 更详细或更简略、包含更多或更少的技术术语，或对当前上下文提供更高层次或更低层次的摘要。

以下示例展示了如何创建一个提示，以确保在分析转租协议时生成的摘要遵循一致的结构：

<CodeGroup exclude="shell">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 Anthropic SDK 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 Anthropic SDK 运行它。
  </CodeGroupItem>

  ```python Python
  # 初始化 Anthropic 客户端
  client = anthropic.Anthropic()


  def summarize_document(
      text, details_to_extract, model="claude-opus-5-5", max_tokens=16000
  ):
      # 格式化待提取的详细信息，以放入提示的上下文中
      details_to_extract_str = "\n".join(details_to_extract)

      # 提示模型总结转租协议
      prompt = f"""Summarize the following sublease agreement. Focus on these key aspects:

      {details_to_extract_str}

      Provide the summary in bullet points nested within the XML header for each section. For example:

      <parties involved>
      - Sublessor: [Name]
      // Add more details as needed
      </parties involved>

      If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.

      Sublease agreement text:
      {text}
      """

      response = client.messages.create(
          model=model,
          max_tokens=max_tokens,
          system="You are a legal analyst specializing in real estate law, known for highly accurate and detailed summaries of sublease agreements.",
          messages=[
              {"role": "user", "content": prompt},
          ],
      )

      return next(block.text for block in response.content if block.type == "text")


  sublease_summary = summarize_document(document_text, details_to_extract)
  print(sublease_summary)
  ```

  ```typescript TypeScript
  // 初始化 Anthropic 客户端
  const client = new Anthropic();

  async function summarizeDocument(
    text: string,
    detailsToExtract: string[],
    model: Anthropic.Model = "claude-opus-5-5",
    maxTokens = 16000
  ): Promise<string> {
    // 格式化要提取的详细信息，以放入提示的上下文中
    const detailsToExtractStr = detailsToExtract.join("\n");

    // 提示模型总结转租协议
    const prompt = `Summarize the following sublease agreement. Focus on these key aspects:

      ${detailsToExtractStr}

      Provide the summary in bullet points nested within the XML header for each section. For example:

      <parties involved>
      - Sublessor: [Name]
      // Add more details as needed
      </parties involved>

      If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.

      Sublease agreement text:
      ${text}
      `;

    const response = await client.messages.create({
      model,
      max_tokens: maxTokens,
      system:
        "You are a legal analyst specializing in real estate law, known for highly accurate and detailed summaries of sublease agreements.",
      messages: [{ role: "user", content: prompt }]
    });

    const textBlock = response.content.find((block) => block.type === "text");
    if (!textBlock) {
      throw new Error("Expected a text block in the response");
    }
    return textBlock.text;
  }

  const subleaseSummary = await summarizeDocument(documentText, detailsToExtract);
  console.log(subleaseSummary);
  ```

  ```csharp C#
  // 初始化 Anthropic 客户端
  var client = new AnthropicClient();

  async Task<string> SummarizeDocument(
      string text,
      IReadOnlyList<string> detailsToExtract,
      Model model = Model.ClaudeOpus5_5,
      long maxTokens = 16000
  )
  {
      // 格式化要提取的详细信息，以放入提示的上下文中
      var detailsToExtractStr = string.Join("\n", detailsToExtract);

      // 提示模型总结转租协议
      var prompt = $"""
          Summarize the following sublease agreement. Focus on these key aspects:

              {detailsToExtractStr}

              Provide the summary in bullet points nested within the XML header for each section. For example:

              <parties involved>
              - Sublessor: [Name]
              // Add more details as needed
              </parties involved>

              If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.

              Sublease agreement text:
              {text}
          """;

      var response = await client.Messages.Create(
          new()
          {
              Model = model,
              MaxTokens = maxTokens,
              System = "You are a legal analyst specializing in real estate law, known for highly accurate and detailed summaries of sublease agreements.",
              Messages = [new UserMessageParam { Content = prompt }],
          }
      );

      foreach (var block in response.Content)
      {
          if (block.TryPickText(out var textBlock))
          {
              return textBlock.Text;
          }
      }
      throw new InvalidOperationException("Expected a text block in the response.");
  }

  var subleaseSummary = await SummarizeDocument(documentText, detailsToExtract);
  Console.WriteLine(subleaseSummary);
  ```

  ```go Go
  // 初始化 Anthropic 客户端
  var client = anthropic.NewClient()

  func summarizeDocument(text string, detailsToExtract []string, model anthropic.Model, maxTokens int64) string {
  	// 格式化要提取的详细信息，以放入提示的上下文中
  	detailsToExtractStr := strings.Join(detailsToExtract, "\n")

  	// 提示模型总结转租协议
  	prompt := fmt.Sprintf(`Summarize the following sublease agreement. Focus on these key aspects:

      %s

      Provide the summary in bullet points nested within the XML header for each section. For example:

      <parties involved>
      - Sublessor: [Name]
      // Add more details as needed
      </parties involved>

      If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.

      Sublease agreement text:
      %s
      `, detailsToExtractStr, text)

  	response, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  		Model:     model,
  		MaxTokens: maxTokens,
  		System: []anthropic.TextBlockParam{
  			{Text: "You are a legal analyst specializing in real estate law, known for highly accurate and detailed summaries of sublease agreements."},
  		},
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock(prompt)),
  		},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	for _, block := range response.Content {
  		if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  			return textBlock.Text
  		}
  	}
  	panic("expected a text block in the response")
  }

  // printSubleaseSummary 总结转租协议并打印摘要。
  func printSubleaseSummary() {
  	subleaseSummary := summarizeDocument(documentText, detailsToExtract, anthropic.ModelClaudeOpus5_5, 16000)
  	fmt.Println(subleaseSummary)
  }
  ```

  ```java Java
  // 初始化 Anthropic 客户端
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  String summarizeDocument(String text, List<String> detailsToExtract, Model model, long maxTokens) {
      // 格式化要提取的详细信息，以便放入提示的上下文中
      String detailsToExtractStr = String.join("\n", detailsToExtract);

      // 提示模型总结转租协议
      String prompt = """
          Summarize the following sublease agreement. Focus on these key aspects:

              %s

              Provide the summary in bullet points nested within the XML header for each section. For example:

              <parties involved>
              - Sublessor: [Name]
              // Add more details as needed
              </parties involved>

              If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.

              Sublease agreement text:
              %s
          """
          .formatted(detailsToExtractStr, text);

      Message response = client.messages().create(MessageCreateParams.builder()
          .model(model)
          .maxTokens(maxTokens)
          .system("You are a legal analyst specializing in real estate law, known for highly accurate and detailed summaries of sublease agreements.")
          .addUserMessage(prompt)
          .build());

      return response.content().stream()
          .flatMap(block -> block.text().stream())
          .findFirst()
          .orElseThrow()
          .text();
  }

  void printSubleaseSummary() {
      String subleaseSummary = summarizeDocument(
          documentText, detailsToExtract, Model.CLAUDE_OPUS_5_5, 16000);
      IO.println(subleaseSummary);
  }
  ```

  ```php PHP
  // 初始化 Anthropic 客户端
  $client = new Client();

  $summarizeDocument = function (
      string $text,
      array $detailsToExtract,
      Model $model = Model::CLAUDE_OPUS_5_5,
      int $maxTokens = 16000,
  ) use ($client): string {
      // 格式化要提取的详细信息，以放入提示的上下文中
      $detailsToExtractStr = implode("\n", $detailsToExtract);

      // 提示模型总结转租协议
      $prompt = <<<PROMPT
          Summarize the following sublease agreement. Focus on these key aspects:

              {$detailsToExtractStr}

              Provide the summary in bullet points nested within the XML header for each section. For example:

              <parties involved>
              - Sublessor: [Name]
              // Add more details as needed
              </parties involved>

              If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.

              Sublease agreement text:
              {$text}
          PROMPT;

      $response = $client->messages->create(
          model: $model,
          maxTokens: $maxTokens,
          system: 'You are a legal analyst specializing in real estate law, known for highly accurate and detailed summaries of sublease agreements.',
          messages: [['role' => 'user', 'content' => $prompt]],
      );

      $textBlock = array_find($response->content, fn ($block) => $block instanceof \Anthropic\Messages\TextBlock)
          ?? throw new \RuntimeException('Expected a text block in the response.');

      return $textBlock->text;
  };

  $subleaseSummary = $summarizeDocument($documentText, $detailsToExtract);
  echo $subleaseSummary . "\n";
  ```

  ```ruby Ruby
  # 初始化 Anthropic 客户端
  CLIENT = Anthropic::Client.new

  def summarize_document(text, details_to_extract, model: Anthropic::Model::CLAUDE_OPUS_5_5, max_tokens: 16_000)
    # 格式化要提取的详细信息，以便放入提示的上下文中
    details_to_extract_str = details_to_extract.join("\n")

    # 提示模型总结转租协议
    prompt = <<~PROMPT
      Summarize the following sublease agreement. Focus on these key aspects:

          #{details_to_extract_str}

          Provide the summary in bullet points nested within the XML header for each section. For example:

          <parties involved>
          - Sublessor: [Name]
          // Add more details as needed
          </parties involved>

          If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.

          Sublease agreement text:
          #{text}
    PROMPT

    response = CLIENT.messages.create(
      model:,
      max_tokens:,
      system_: "You are a legal analyst specializing in real estate law, known for highly accurate and detailed summaries of sublease agreements.",
      messages: [{role: :user, content: prompt}]
    )

    response.content.find { |block| block.is_a?(Anthropic::Models::TextBlock) }.text
  end

  sublease_summary = summarize_document(document_text, details_to_extract)
  puts sublease_summary
  ```
</CodeGroup>

此代码实现了一个 `summarize_document()` (typescript, go, java: `summarizeDocument()`; php: `$summarizeDocument()`; csharp: `SummarizeDocument()`) 函数，该函数使用 Claude 对转租协议的内容进行摘要。该函数接受一个文本字符串和一个待提取细节的列表作为输入。在此示例中，代码使用前面代码片段中定义的 `document_text` (typescript, go, java, csharp: `documentText`; php: `$documentText`) 和 `details_to_extract` (typescript, go, java, csharp: `detailsToExtract`; php: `$detailsToExtract`) 变量调用该函数。

在函数内部，会为 Claude 生成一个提示，其中包括待摘要的文件、待提取的细节以及摘要文件的具体指令。该提示指示 Claude 以嵌套在 XML 标头中的形式返回每个待提取细节的摘要。

由于代码将摘要的每个部分输出在标签内，因此可以在后处理步骤中解析出每个部分。这种方法使您能够创建可根据用例进行调整的结构化摘要，从而使每份摘要都遵循相同的模式。

### 评估您的提示

提示通常需要经过测试和优化才能投入生产使用。为了确定您的解决方案是否已准备就绪，请使用结合定量和定性方法的系统化流程来评估摘要质量。根据您定义的成功标准创建[强大的实证评估](https://platform.claude.com/docs/zh-CN/test-and-evaluate/develop-tests#build-evaluations)，可以帮助您优化提示。以下是您可能希望在实证评估中纳入的一些指标：

<AccordionGroup>
  <Accordion title="ROUGE 分数">
    该指标衡量生成的摘要与专家创建的参考摘要之间的重叠程度。该指标主要关注召回率，适用于评估内容覆盖度。
  </Accordion>

  <Accordion title="BLEU 分数">
    虽然最初是为机器翻译开发的，但该指标可以适用于摘要任务。BLEU 分数衡量生成的摘要与参考摘要之间 n-gram 匹配的精确率。分数越高，表明生成的摘要包含与参考摘要相似的短语和术语。
  </Accordion>

  <Accordion title="上下文嵌入相似度">
    该指标涉及为生成的摘要和参考摘要创建向量表示（嵌入）。然后计算这些嵌入之间的相似度，通常使用余弦相似度。相似度分数越高，表明生成的摘要捕捉到了参考摘要的语义含义和上下文，即使确切措辞有所不同。
  </Accordion>

  <Accordion title="基于 LLM 的评分">
    该方法涉及使用 Claude 等 LLM 根据评分标准评估生成摘要的质量。评分标准可以根据您的具体需求进行定制，评估准确性、完整性和连贯性等关键因素。有关实施指导，请参阅

    [基于 LLM 评分的技巧](https://platform.claude.com/docs/zh-CN/test-and-evaluate/develop-tests#tips-for-llm-based-grading)

    。
  </Accordion>

  <Accordion title="人工评估">
    除了创建参考摘要外，法律专家还可以评估生成摘要的质量。虽然大规模进行这项工作既昂贵又耗时，但在部署到生产环境之前，通常会对少量摘要进行此类评估作为验证检查。
  </Accordion>
</AccordionGroup>

### 部署您的提示

以下是在将解决方案部署到生产环境时需要牢记的一些其他注意事项。

1. **确保无责任风险：** 了解摘要中的错误可能带来的法律影响，这些错误可能导致您的组织或客户承担法律责任。提供免责声明或法律声明，说明摘要由 AI 生成，应由法律专业人士审查。

2. **处理多种文档类型：** 本指南讨论了如何从 PDF 中提取文本。在实际场景中，文档可能采用多种格式（例如 PDF、Word 文档和文本文件）。请确保您的数据提取管道能够转换您预期接收的所有文件格式。

3. **并行调用 Claude API：** 包含大量令牌的长文档可能需要 Claude 最多一分钟才能生成摘要。对于大型文档集合，您可能希望并行向 Claude 发送 API 调用，以便在合理的时间内完成摘要。请参阅 Anthropic 的[速率限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#rate-limits)，以确定可以并行执行的最大 API 调用数。

***

## 提升性能

在复杂场景中，除了标准的[提示工程技术](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/overview)之外，考虑其他策略来提升性能可能会有所帮助。以下是一些高级策略：

### 执行元摘要以对长文件进行摘要

法律文件摘要通常涉及一次处理长文件或许多相关文件，以至于超出 Claude 的"context window"（上下文窗口）。您可以使用一种称为"meta-summarization"（元摘要）的分块方法来处理此用例。该技术涉及将文件分解为更小、更易管理的块，然后分别处理每个块。之后，您可以将每个块的摘要组合起来，创建整个文件的元摘要。

以下是如何执行元摘要的示例：

<CodeGroup exclude="shell">
  <CodeGroupItem language="cURL">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 Anthropic SDK 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此工作流需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 Anthropic SDK 运行它。
  </CodeGroupItem>

  ```python Python
  def chunk_text(text, chunk_size=20000):
      return [text[i : i + chunk_size] for i in range(0, len(text), chunk_size)]


  def summarize_long_document(
      text, details_to_extract, model="claude-opus-5-5", max_tokens=16000
  ):
      # 格式化待提取的详细信息，以放入提示的上下文中
      details_to_extract_str = "\n".join(details_to_extract)

      # 遍历各个分块并逐一总结
      chunk_summaries = [
          summarize_document(
              chunk, details_to_extract, model=model, max_tokens=max_tokens
          )
          for chunk in chunk_text(text)
      ]

      final_summary_prompt = f"""

      You are looking at the chunked summaries of multiple documents that are all related.
      Combine the following summaries of the document from different truthful sources into a coherent overall summary:

      <chunked_summaries>
      {"".join(chunk_summaries)}
      </chunked_summaries>

      Focus on these key aspects:
      {details_to_extract_str}

      Provide the summary in bullet points nested within the XML header for each section. For example:

      <parties involved>
      - Sublessor: [Name]
      // Add more details as needed
      </parties involved>

      If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.
      """

      response = client.messages.create(
          model=model,
          max_tokens=max_tokens,
          system="You are a legal expert that summarizes notes on one document.",
          messages=[
              {"role": "user", "content": final_summary_prompt},
          ],
      )

      return next(block.text for block in response.content if block.type == "text")


  long_summary = summarize_long_document(document_text, details_to_extract)
  print(long_summary)
  ```

  ```typescript TypeScript
  function chunkText(text: string, chunkSize = 20000): string[] {
    return Array.from({ length: Math.ceil(text.length / chunkSize) }, (_, index) =>
      text.slice(index * chunkSize, (index + 1) * chunkSize)
    );
  }

  async function summarizeLongDocument(
    text: string,
    detailsToExtract: string[],
    model: Anthropic.Model = "claude-opus-5-5",
    maxTokens = 16000
  ): Promise<string> {
    // 格式化要提取的详细信息，以放入提示的上下文中
    const detailsToExtractStr = detailsToExtract.join("\n");

    // 遍历各个分块并逐一总结
    const chunkSummaries: string[] = [];
    for (const chunk of chunkText(text)) {
      chunkSummaries.push(await summarizeDocument(chunk, detailsToExtract, model, maxTokens));
    }

    const finalSummaryPrompt = `

      You are looking at the chunked summaries of multiple documents that are all related.
      Combine the following summaries of the document from different truthful sources into a coherent overall summary:

      <chunked_summaries>
      ${chunkSummaries.join("")}
      </chunked_summaries>

      Focus on these key aspects:
      ${detailsToExtractStr}

      Provide the summary in bullet points nested within the XML header for each section. For example:

      <parties involved>
      - Sublessor: [Name]
      // Add more details as needed
      </parties involved>

      If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.
      `;

    const response = await client.messages.create({
      model,
      max_tokens: maxTokens,
      system: "You are a legal expert that summarizes notes on one document.",
      messages: [{ role: "user", content: finalSummaryPrompt }]
    });

    const textBlock = response.content.find((block) => block.type === "text");
    if (!textBlock) {
      throw new Error("Expected a text block in the response");
    }
    return textBlock.text;
  }

  const longSummary = await summarizeLongDocument(documentText, detailsToExtract);
  console.log(longSummary);
  ```

  ```csharp C#
  IEnumerable<string> ChunkText(string text, int chunkSize = 20000) =>
      text.Chunk(chunkSize).Select(chars => new string(chars));

  async Task<string> SummarizeLongDocument(
      string text,
      IReadOnlyList<string> detailsToExtract,
      Model model = Model.ClaudeOpus5_5,
      long maxTokens = 16000
  )
  {
      // 格式化要提取的详细信息，以放入提示的上下文中
      var detailsToExtractStr = string.Join("\n", detailsToExtract);

      // 遍历各个分块并逐一总结
      List<string> chunkSummaries = [];
      foreach (var chunk in ChunkText(text))
      {
          chunkSummaries.Add(await SummarizeDocument(chunk, detailsToExtract, model, maxTokens));
      }

      var finalSummaryPrompt = $"""

              You are looking at the chunked summaries of multiple documents that are all related.
              Combine the following summaries of the document from different truthful sources into a coherent overall summary:

              <chunked_summaries>
              {string.Concat(chunkSummaries)}
              </chunked_summaries>

              Focus on these key aspects:
              {detailsToExtractStr}

              Provide the summary in bullet points nested within the XML header for each section. For example:

              <parties involved>
              - Sublessor: [Name]
              // Add more details as needed
              </parties involved>

              If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.
          """;

      var response = await client.Messages.Create(
          new()
          {
              Model = model,
              MaxTokens = maxTokens,
              System = "You are a legal expert that summarizes notes on one document.",
              Messages = [new UserMessageParam { Content = finalSummaryPrompt }],
          }
      );

      foreach (var block in response.Content)
      {
          if (block.TryPickText(out var textBlock))
          {
              return textBlock.Text;
          }
      }
      throw new InvalidOperationException("Expected a text block in the response.");
  }

  var longSummary = await SummarizeLongDocument(documentText, detailsToExtract);
  Console.WriteLine(longSummary);
  ```

  ```go Go
  func chunkText(text string, chunkSize int) []string {
  	var chunks []string
  	for chunk := range slices.Chunk([]rune(text), chunkSize) {
  		chunks = append(chunks, string(chunk))
  	}
  	return chunks
  }

  func summarizeLongDocument(text string, detailsToExtract []string, model anthropic.Model, maxTokens int64) string {
  	// 格式化要提取的详细信息，以放入提示的上下文中
  	detailsToExtractStr := strings.Join(detailsToExtract, "\n")

  	// 遍历各个分块并逐一总结
  	var chunkSummaries []string
  	for _, chunk := range chunkText(text, 20000) {
  		chunkSummaries = append(chunkSummaries, summarizeDocument(chunk, detailsToExtract, model, maxTokens))
  	}

  	finalSummaryPrompt := fmt.Sprintf(`

      You are looking at the chunked summaries of multiple documents that are all related.
      Combine the following summaries of the document from different truthful sources into a coherent overall summary:

      <chunked_summaries>
      %s
      </chunked_summaries>

      Focus on these key aspects:
      %s

      Provide the summary in bullet points nested within the XML header for each section. For example:

      <parties involved>
      - Sublessor: [Name]
      // Add more details as needed
      </parties involved>

      If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.
      `, strings.Join(chunkSummaries, ""), detailsToExtractStr)

  	response, err := client.Messages.New(context.Background(), anthropic.MessageNewParams{
  		Model:     model,
  		MaxTokens: maxTokens,
  		System: []anthropic.TextBlockParam{
  			{Text: "You are a legal expert that summarizes notes on one document."},
  		},
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock(finalSummaryPrompt)),
  		},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	for _, block := range response.Content {
  		if textBlock, ok := block.AsAny().(anthropic.TextBlock); ok {
  			return textBlock.Text
  		}
  	}
  	panic("expected a text block in the response")
  }

  // printLongSummary 逐块总结转租协议并打印合并后的摘要。
  func printLongSummary() {
  	longSummary := summarizeLongDocument(documentText, detailsToExtract, anthropic.ModelClaudeOpus5_5, 16000)
  	fmt.Println(longSummary)
  }
  ```

  ```java Java
  List<String> chunkText(String text, int chunkSize) {
      return IntStream.iterate(0, start -> start < text.length(), start -> start + chunkSize)
          .mapToObj(start -> text.substring(start, Math.min(start + chunkSize, text.length())))
          .toList();
  }

  String summarizeLongDocument(String text, List<String> detailsToExtract, Model model, long maxTokens) {
      // 格式化要提取的详细信息，以便放入提示的上下文中
      String detailsToExtractStr = String.join("\n", detailsToExtract);

      // 遍历各个分块并逐一总结
      String chunkSummaries = chunkText(text, 20000).stream()
          .map(chunk -> summarizeDocument(chunk, detailsToExtract, model, maxTokens))
          .collect(Collectors.joining());

      String finalSummaryPrompt = """

              You are looking at the chunked summaries of multiple documents that are all related.
              Combine the following summaries of the document from different truthful sources into a coherent overall summary:

              <chunked_summaries>
              %s
              </chunked_summaries>

              Focus on these key aspects:
              %s

              Provide the summary in bullet points nested within the XML header for each section. For example:

              <parties involved>
              - Sublessor: [Name]
              // Add more details as needed
              </parties involved>

              If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.
          """
          .formatted(chunkSummaries, detailsToExtractStr);

      Message response = client.messages().create(MessageCreateParams.builder()
          .model(model)
          .maxTokens(maxTokens)
          .system("You are a legal expert that summarizes notes on one document.")
          .addUserMessage(finalSummaryPrompt)
          .build());

      return response.content().stream()
          .flatMap(block -> block.text().stream())
          .findFirst()
          .orElseThrow()
          .text();
  }

  void printLongSummary() {
      String longSummary = summarizeLongDocument(
          documentText, detailsToExtract, Model.CLAUDE_OPUS_5_5, 16000);
      IO.println(longSummary);
  }
  ```

  ```php PHP
  $chunkText = fn (string $text, int $chunkSize = 20000): array => mb_str_split($text, $chunkSize);

  $summarizeLongDocument = function (
      string $text,
      array $detailsToExtract,
      Model $model = Model::CLAUDE_OPUS_5_5,
      int $maxTokens = 16000,
  ) use ($client, $chunkText, $summarizeDocument): string {
      // 格式化要提取的详细信息，以放入提示的上下文中
      $detailsToExtractStr = implode("\n", $detailsToExtract);

      // 遍历各个分块并逐一总结
      $chunkSummaries = array_map(
          fn (string $chunk) => $summarizeDocument($chunk, $detailsToExtract, $model, $maxTokens),
          $chunkText($text),
      );
      $chunkSummariesStr = implode('', $chunkSummaries);

      $finalSummaryPrompt = <<<PROMPT

              You are looking at the chunked summaries of multiple documents that are all related.
              Combine the following summaries of the document from different truthful sources into a coherent overall summary:

              <chunked_summaries>
              {$chunkSummariesStr}
              </chunked_summaries>

              Focus on these key aspects:
              {$detailsToExtractStr}

              Provide the summary in bullet points nested within the XML header for each section. For example:

              <parties involved>
              - Sublessor: [Name]
              // Add more details as needed
              </parties involved>

              If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.
          PROMPT;

      $response = $client->messages->create(
          model: $model,
          maxTokens: $maxTokens,
          system: 'You are a legal expert that summarizes notes on one document.',
          messages: [['role' => 'user', 'content' => $finalSummaryPrompt]],
      );

      $textBlock = array_find($response->content, fn ($block) => $block instanceof \Anthropic\Messages\TextBlock)
          ?? throw new \RuntimeException('Expected a text block in the response.');

      return $textBlock->text;
  };

  $longSummary = $summarizeLongDocument($documentText, $detailsToExtract);
  echo $longSummary . "\n";
  ```

  ```ruby Ruby
  def chunk_text(text, chunk_size: 20_000)
    text.chars.each_slice(chunk_size).map(&:join)
  end

  def summarize_long_document(text, details_to_extract, model: Anthropic::Model::CLAUDE_OPUS_5_5, max_tokens: 16_000)
    # 格式化要提取的详细信息，以便放入提示的上下文中
    details_to_extract_str = details_to_extract.join("\n")

    # 遍历各个分块并分别进行总结
    chunk_summaries = chunk_text(text).map do |chunk|
      summarize_document(chunk, details_to_extract, model:, max_tokens:)
    end

    final_summary_prompt = <<~PROMPT

          You are looking at the chunked summaries of multiple documents that are all related.
          Combine the following summaries of the document from different truthful sources into a coherent overall summary:

          <chunked_summaries>
          #{chunk_summaries.join}
          </chunked_summaries>

          Focus on these key aspects:
          #{details_to_extract_str}

          Provide the summary in bullet points nested within the XML header for each section. For example:

          <parties involved>
          - Sublessor: [Name]
          // Add more details as needed
          </parties involved>

          If any information is not explicitly stated in the document, note it as "Not specified". Do not preamble.
    PROMPT

    response = CLIENT.messages.create(
      model:,
      max_tokens:,
      system_: "You are a legal expert that summarizes notes on one document.",
      messages: [{role: :user, content: final_summary_prompt}]
    )

    response.content.find { |block| block.is_a?(Anthropic::Models::TextBlock) }.text
  end

  long_summary = summarize_long_document(document_text, details_to_extract)
  puts long_summary
  ```
</CodeGroup>

`summarize_long_document()` (typescript, go, java: `summarizeLongDocument()`; php: `$summarizeLongDocument()`; csharp: `SummarizeLongDocument()`) 函数在前面的 `summarize_document()` (typescript, go, java: `summarizeDocument()`; php: `$summarizeDocument()`; csharp: `SummarizeDocument()`) 函数的基础上构建，它将文件拆分为更小的块，并分别对每个块进行摘要。

代码通过对原始文件中每个 20,000 字符的块应用 `summarize_document()` (typescript, go, java: `summarizeDocument()`; php: `$summarizeDocument()`; csharp: `SummarizeDocument()`) 函数来实现这一点。然后将各个摘要组合起来，并根据这些块摘要创建最终摘要。

请注意，对于示例 PDF 而言，`summarize_long_document()` (typescript, go, java: `summarizeLongDocument()`; php: `$summarizeLongDocument()`; csharp: `SummarizeLongDocument()`) 函数并非绝对必要，因为整个文件都能放入 Claude 的上下文窗口中。但是，对于超出 Claude 上下文窗口的文件，或者在对多份相关文件一起进行摘要时，它就变得必不可少。无论如何，这种元摘要技术通常能在最终摘要中捕捉到早期单一摘要方法所遗漏的其他重要细节。

### 使用摘要索引文件探索大型文件集合

使用 LLM 搜索文档集合通常涉及"retrieval-augmented generation"（检索增强生成），即 RAG。然而，在涉及大型文档或精确信息检索至关重要的场景中，基本的 RAG 方法可能不够用。摘要索引文档是一种高级 RAG 方法，它提供了一种更高效的文档检索排序方式，所使用的上下文比传统 RAG 方法更少。在这种方法中，您首先使用 Claude 为语料库中的每份文档生成简洁的摘要，然后使用 Claude 对每份摘要与所提问题的相关性进行排序。有关此方法的更多详细信息（包括基于代码的示例），请参阅[摘要示例](https://platform.claude.com/cookbook/capabilities-summarization-guide)中的摘要索引文档部分。

### 微调 Claude 以从您的数据集中学习

另一种提升 Claude 生成摘要能力的高级技术是"fine-tuning"（微调）。微调涉及在专门符合您法律摘要需求的自定义数据集上训练 Claude，确保 Claude 适应您的用例。以下是如何执行微调的概述：

1. **识别错误：** 首先收集 Claude 摘要表现不佳的实例。这可能包括遗漏关键法律细节、误解上下文或使用不恰当的法律术语。

2. **整理数据集：** 识别出这些问题后，将这些有问题的示例汇编成数据集。该数据集应包括原始法律文档以及您修正后的摘要，以确保 Claude 学习到您期望的行为。

3. **执行微调：** 微调涉及在您整理的数据集上重新训练模型，以调整其权重和参数。这种重新训练有助于 Claude 更好地适应您法律领域的特定要求，提升其按照您的标准对文件进行摘要的能力。

4. **迭代改进：** 微调并非一次性过程。随着 Claude 持续生成摘要，您可以迭代地添加其表现不佳的新示例，进一步完善其能力。随着时间的推移，这种持续的反馈循环将产生一个高度专门化、适用于您法律摘要任务的模型。

<Tip>
  微调目前仅通过 Amazon Bedrock 提供。更多详细信息请参阅 

  [AWS 发布博客](https://aws.amazon.com/blogs/machine-learning/fine-tune-anthropics-claude-3-haiku-in-amazon-bedrock-to-boost-model-accuracy-and-quality/)

  。
</Tip>

<CardGroup cols={2}>
  <Card title="摘要示例" icon="link" href="https://platform.claude.com/cookbook/capabilities-summarization-guide">
    查看如何使用 Claude 对合同进行摘要的完整实现代码示例。
  </Card>

  <Card title="引用示例" icon="link" href="https://platform.claude.com/cookbook/misc-using-citations">
    浏览引用示例，获取有关如何确保信息准确性和可解释性的指导。
  </Card>
</CardGroup>
