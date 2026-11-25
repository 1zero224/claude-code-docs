---
title: Link your Claude organization 按钮
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-button
description: 添加将客户从您的产品引导至 Claude 批准页面以启动 Linked billing 的按钮。
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

本页列出了用于 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing) 的 **Link your Claude organization** 按钮的 HTML、CSS、颜色、尺寸和徽标。

<Frame caption="浅色和深色按钮">
  ![Link your Claude organization 按钮，浅色和深色，文字前带有 Claude 徽标](https://platform.claude.com/docs/images/claude-link-button.svg)
</Frame>

## HTML 和 CSS

按钮外观中您唯一可以更改的部分是其尺寸，以及使用浅色按钮还是深色按钮。字体、颜色以及徽标相对于文字的大小须保持[颜色和尺寸](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-button#colors)中所示的样式。

按钮的 CSS 和 HTML 按如下方式放入您的页面：

* **CSS：** **按钮 CSS** 中的 `<style>` 块放在页面的 `<head>` 中。如果您的 Content Security Policy（内容安全策略）不允许内联样式，请将其中的规则放入您的样式表。
* **HTML：** **示例 HTML** 中的 `<a>` 元素放在按钮显示的位置。它包含徽标，因此该按钮无需在您的 Content Security Policy 中添加任何 Anthropic 主机。
* **路由：** 该元素的 `href`（示例中为 `/claude/billing/link`）指向您后端中[启动批准的路由](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-and-bill-a-customer#send-them-to-approve)。
* **深色按钮：** 在元素上的 `claude-button` 旁边添加 `claude-button-dark` 类，即可得到深色按钮。
* **尺寸：** 要调整按钮大小，请仅更改 **按钮 CSS** 中的 `font-size` 行，而不要更改 `font` 行中的大小。

有关如何使用 Claude 名称和徽标，请参阅 [Linked Billing Beta Terms](https://www.anthropic.com/legal/linked-billing-terms)。

<Accordion title="按钮 CSS">
  ```html claude-button.html
  <style>
    /* Each size is written as calc(Nem / 14), where N is its size in pixels
       when the text is 14 pixels. To make the button larger, change only font-size. */
    .claude-button {
      --claude-button-fill: #FFFFFF;
      --claude-button-fill-hover: #F5F4ED;
      --claude-button-text: #141413;
      --claude-button-border: #87867F;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: calc(10em / 14);
      box-sizing: border-box;
      min-height: calc(40em / 14);
      padding: calc(8em / 14) calc(12em / 14);
      border: 1px solid var(--claude-button-border);
      border-radius: calc(8em / 14);
      background: var(--claude-button-fill);
      font: 500 0.875rem/1.4286 ui-sans-serif, system-ui, -apple-system, "Segoe UI",
        Roboto, "Helvetica Neue", Arial, sans-serif;
      font-size: 0.875rem;
      letter-spacing: normal;
      text-transform: none;
      cursor: pointer;
    }
    .claude-button-dark {
      --claude-button-fill: #141413;
      --claude-button-fill-hover: #262624;
      --claude-button-text: #FAF9F5;
      --claude-button-border: #73726C;
    }
    .claude-button,
    .claude-button:link,
    .claude-button:visited,
    .claude-button:hover,
    .claude-button:focus,
    .claude-button:active {
      color: var(--claude-button-text);
      text-decoration: none;
    }
    .claude-button:hover {
      background: var(--claude-button-fill-hover);
    }
    .claude-button:focus-visible {
      outline: 2px solid var(--claude-button-text);
      outline-offset: 2px;
      box-shadow: 0 0 0 2px var(--claude-button-fill);
    }
    .claude-button svg {
      flex: none;
      width: calc(20em / 14);
      height: calc(20em / 14);
    }
    .claude-button path {
      fill: #D97757;
      stroke: none;
    }
  </style>
  ```
</Accordion>

<Accordion title="示例 HTML">
  ```html link-your-claude-organization.html
  <a class="claude-button" href="/claude/billing/link">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="20" height="20" aria-hidden="true">
      <path fill="#D97757" d="M11.376 24L10.776 23.544L10.44 22.8L10.776 21.312L11.16 19.392L11.472 17.856L11.76 15.96
        L11.928 15.336L11.904 15.288L11.784 15.312L10.344 17.28L8.16 20.232L6.432 22.056
        L6.024 22.224L5.304 21.864L5.376 21.192L5.784 20.616L8.16 17.568L9.6 15.672L10.536 14.592
        L10.512 14.448H10.464L4.128 18.576L3 18.72L2.496 18.264L2.568 17.52L2.808 17.28L4.704 15.96
        L9.432 13.32L9.504 13.08L9.432 12.96H9.192L8.4 12.912L5.712 12.84L3.384 12.744L1.104 12.624
        L0.528 12.504L0 11.784L0.048 11.424L0.528 11.112L1.224 11.16L2.736 11.28L5.016 11.424
        L6.672 11.52L9.12 11.784H9.504L9.552 11.616L9.432 11.52L9.336 11.424L6.96 9.84L4.416 8.16
        L3.072 7.176L2.352 6.672L1.992 6.216L1.848 5.208L2.496 4.488L3.384 4.56L3.6 4.608
        L4.488 5.304L6.384 6.768L8.88 8.616L9.24 8.904L9.408 8.808V8.736L9.24 8.472L7.896 6.024
        L6.456 3.528L5.808 2.496L5.64 1.872C5.576 1.656 5.544 1.416 5.544 1.152L6.288 0.144001
        L6.696 0L7.704 0.144001L8.112 0.504001L8.736 1.92L9.72 4.152L11.28 7.176L11.736 8.088
        L11.976 8.904L12.072 9.168H12.24V9.024L12.36 7.296L12.6 5.208L12.84 2.52L12.912 1.752
        L13.296 0.840001L14.04 0.360001L14.616 0.624001L15.096 1.32L15.024 1.752L14.76 3.6
        L14.184 6.504L13.824 8.472H14.04L14.28 8.208L15.264 6.912L16.92 4.848L17.64 4.032
        L18.504 3.12L19.056 2.688H20.088L20.832 3.816L20.496 4.992L19.44 6.336L18.552 7.464
        L17.28 9.168L16.512 10.536L16.584 10.632H16.752L19.608 10.008L21.168 9.744L22.992 9.432
        L23.832 9.816L23.928 10.2L23.592 11.016L21.624 11.496L19.32 11.952L15.888 12.768
        L15.84 12.792L15.888 12.864L17.424 13.008L18.096 13.056H19.728L22.752 13.272L23.544 13.8
        L24 14.424L23.928 14.928L22.704 15.528L21.072 15.144L17.232 14.232L15.936 13.92H15.744
        V14.016L16.848 15.096L18.84 16.896L21.36 19.224L21.48 19.8L21.168 20.28L20.832 20.232
        L18.624 18.552L17.76 17.808L15.84 16.2H15.72V16.368L16.152 17.016L18.504 20.544
        L18.624 21.624L18.456 21.96L17.832 22.176L17.184 22.056L15.792 20.136L14.376 17.952
        L13.224 16.008L13.104 16.104L12.408 23.352L12.096 23.712L11.376 24Z"/>
    </svg>
    Link your Claude organization
  </a>
  ```
</Accordion>

如果您的路由需要表单提交（例如 `POST`），请在表单中使用 `<button type="submit" class="claude-button">` 代替 `<a>` 元素。它包含相同的徽标和文字。表单的 action 为您的路由，并适用相同的样式。

您自己样式表中的规则（例如针对 `.nav a` 的规则）可能会覆盖按钮的样式。这样按钮就会与[颜色和尺寸](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-button#colors)中的图片和表格不一致。为保持其设计，请编写此类规则时让其跳过带有 `claude-button` 类的元素。

## 颜色和尺寸

| 部分 | 浅色按钮 | 深色按钮 |
| ----- | ----- | ----- |
| 背景 | `#FFFFFF` | `#141413` |
| 文字 | `#141413` | `#FAF9F5` |
| 边框 | `#87867F` | `#73726C` |
| 悬停时背景 | `#F5F4ED` | `#262624` |
| 徽标 | `#D97757` | `#D97757` |

在默认字体大小为 14 像素、页面根字体大小为 16 像素时，按钮的尺寸如下：

| 部分 | 尺寸 |
| ----- | ----- |
| 高度 | 40 像素 |
| 内边距 | 上下 8 像素，左右 12 像素 |
| 圆角半径 | 8 像素 |
| 边框 | 1 像素 |
| 徽标 | 20 × 20 像素 |
| 徽标与文字之间的间距 | 10 像素 |
| 文字 | 14 像素，字重 500，使用系统的无衬线字体 |
| 焦点轮廓 | 2 像素，使用文字颜色，距按钮 2 像素，并在按钮周围带有一圈 2 像素的填充色环 |

除边框和焦点轮廓外，每个尺寸都会随按钮的字体大小缩放。

## 徽标文件

对于不使用 HTML 构建的按钮（例如在移动或桌面应用中），徽标为以下文件 `claude-logo.svg`。该按钮采用[颜色和尺寸](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/link-button#colors)中的颜色和尺寸；如果您更改其大小，请按该部分所述将它们一起缩放。与示例 HTML 一样，在该按钮中也请对屏幕阅读器隐藏徽标，因为文字已经说明了它的名称。

<Accordion title="claude-logo.svg">
  <File filename="claude-logo.svg" size="sm">
    ```xml
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24">
      <path fill="#D97757"
        d="M11.376 24L10.776 23.544L10.44 22.8L10.776 21.312L11.16 19.392L11.472 17.856L11.76 15.96
          L11.928 15.336L11.904 15.288L11.784 15.312L10.344 17.28L8.16 20.232L6.432 22.056
          L6.024 22.224L5.304 21.864L5.376 21.192L5.784 20.616L8.16 17.568L9.6 15.672L10.536 14.592
          L10.512 14.448H10.464L4.128 18.576L3 18.72L2.496 18.264L2.568 17.52L2.808 17.28L4.704 15.96
          L9.432 13.32L9.504 13.08L9.432 12.96H9.192L8.4 12.912L5.712 12.84L3.384 12.744L1.104 12.624
          L0.528 12.504L0 11.784L0.048 11.424L0.528 11.112L1.224 11.16L2.736 11.28L5.016 11.424
          L6.672 11.52L9.12 11.784H9.504L9.552 11.616L9.432 11.52L9.336 11.424L6.96 9.84L4.416 8.16
          L3.072 7.176L2.352 6.672L1.992 6.216L1.848 5.208L2.496 4.488L3.384 4.56L3.6 4.608
          L4.488 5.304L6.384 6.768L8.88 8.616L9.24 8.904L9.408 8.808V8.736L9.24 8.472L7.896 6.024
          L6.456 3.528L5.808 2.496L5.64 1.872C5.576 1.656 5.544 1.416 5.544 1.152L6.288 0.144001
          L6.696 0L7.704 0.144001L8.112 0.504001L8.736 1.92L9.72 4.152L11.28 7.176L11.736 8.088
          L11.976 8.904L12.072 9.168H12.24V9.024L12.36 7.296L12.6 5.208L12.84 2.52L12.912 1.752
          L13.296 0.840001L14.04 0.360001L14.616 0.624001L15.096 1.32L15.024 1.752L14.76 3.6
          L14.184 6.504L13.824 8.472H14.04L14.28 8.208L15.264 6.912L16.92 4.848L17.64 4.032
          L18.504 3.12L19.056 2.688H20.088L20.832 3.816L20.496 4.992L19.44 6.336L18.552 7.464
          L17.28 9.168L16.512 10.536L16.584 10.632H16.752L19.608 10.008L21.168 9.744L22.992 9.432
          L23.832 9.816L23.928 10.2L23.592 11.016L21.624 11.496L19.32 11.952L15.888 12.768
          L15.84 12.792L15.888 12.864L17.424 13.008L18.096 13.056H19.728L22.752 13.272L23.544 13.8
          L24 14.424L23.928 14.928L22.704 15.528L21.072 15.144L17.232 14.232L15.936 13.92H15.744
          V14.016L16.848 15.096L18.84 16.896L21.36 19.224L21.48 19.8L21.168 20.28L20.832 20.232
          L18.624 18.552L17.76 17.808L15.84 16.2H15.72V16.368L16.152 17.016L18.504 20.544
          L18.624 21.624L18.456 21.96L17.832 22.176L17.184 22.056L15.792 20.136L14.376 17.952
          L13.224 16.008L13.104 16.104L12.408 23.352L12.096 23.712L11.376 24Z"/>
    </svg>
    ```
  </File>
</Accordion>

## 后续步骤

<CardGroup cols={2}>
  <Card title="您的客户会看到什么" icon="browser" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/what-your-customers-see">
    您的客户会看到的批准页面、支出上限和电子邮件，以便您为他们提供支持
  </Card>

  <Card title="测试并发布您的应用" icon="rocket" href="https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/linked-billing/test-and-publish">
    添加测试用户、发布您的应用，以及更改或删除应用
  </Card>
</CardGroup>
