# Anthropic 托管的浏览器在代表用户浏览网络时如何识别自己？

某些 Claude 产品（包括 **[Claude Duet](https://claude.com/product/claude-duet)**）可以使用由 Anthropic 托管的浏览器按用户指示完成任务。这被称为代理浏览器使用。本文解释了网站所有者如何识别来自此类代理浏览器使用的流量，以及如何选择是否允许它。

代理浏览器使用不同于 Anthropic 的机器人（爬虫和获取器，包括 `ClaudeBot`、`Claude-User` 和 `Claude-SearchBot`）。要了解这些，请参阅 **[Anthropic 是否从网络爬取数据，网站所有者如何阻止爬虫？](https://support.claude.com/en/articles/8896518-does-anthropic-crawl-data-from-the-web-and-how-can-site-owners-block-the-crawler)**

## 如何识别代理浏览器流量

**User-Agent：**请求来自在 Anthropic 基础设施上运行的真实浏览器。它们携带浏览器本身的 `User-Agent` 标头，并添加了 `Claude-Agent` 令牌以指示浏览器由托管的 Claude 代理驱动。

例如：

```
Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko;
compatible; Claude-Agent/1.0) Chrome/154.0.0.0 Safari/537.36
```

此字符串的其他部分可能会更改，因此请匹配 `Claude-Agent` 令牌而不是完整字符串。

**IP 地址：**此流量来自发布在 **<https://claude.com/.well-known/sail>** 的 IP 地址，列在 `Claude-Agent` 下。这些地址可能会随时间变化，因此我们建议从此文件自动更新您的规则，而不是硬编码它们。

任何人都可以发送相同的 `User-Agent` 标头，因此请确保在得出流量是真实的结论之前，检查流量是否来自列出的 IP 之一。有关发布格式的技术详情和使用它的注意事项，请参阅 **[draft-kelley-webbotauth-sail-00](https://www.ietf.org/archive/id/draft-kelley-webbotauth-sail-00.html)**。这还不是标准，因此可能会随时间演变。

允许或禁止此流量将控制您的用户是否可以要求他们的 Claude 服务利用此类代理浏览器使用在您的网站上完成任务。您对 Anthropic 机器人（爬虫和获取器）的偏好是独立的，**[您可以单独管理它们](https://support.claude.com/en/articles/8896518-does-anthropic-crawl-data-from-the-web-and-how-can-site-owners-block-the-crawler)**。