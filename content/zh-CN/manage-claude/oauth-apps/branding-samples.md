---
title: 品牌示例
url: https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/branding-samples
description: 用于 Linked billing 的 Link your Claude organization 按钮：它的 HTML 和 CSS、颜色以及 Claude 徽标。
---

<Warning>
  Claude Platform OAuth 是一项 Beta Service：根据您与 Anthropic 签订的协议（例如[商业服务条款](https://www.anthropic.com/legal/commercial-terms)和[特定服务条款](https://www.anthropic.com/legal/service-specific-terms)）提供的预发布软件。在Beta 期间，端点、参数和行为可能会发生变化。它可能不适合用于生产环境。
</Warning>

为使用 [Linked billing](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections) 的产品添加 **Link your Claude organization** 按钮。将该按钮放在您的客户管理计费或集成的位置。当有人点击该按钮时，您的后端会[启动计费连接](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/organization-billing-connections#runtime-step-2)。

<Frame caption="按钮的浅色（light）和深色（dark）版本">
  ![浅色和深色的 Link your Claude organization 按钮，文字前有 Claude 徽标](https://platform.claude.com/docs/images/claude-link-button.svg)
</Frame>

## 复制按钮的 HTML 和 CSS

打开 **按钮 CSS**，将其 `<style>` 块放在页面的 `<head>` 中，或者将其中的规则移到您的样式表中。然后，打开 **Link your Claude organization 按钮的示例 HTML**，并将链接放在您希望显示按钮的位置。将 `/claude/billing/link` 更改为您后端上启动连接的地址。

该 CSS 和 HTML 共同包含徽标和按钮的所有样式，因此如果您的网站有"Content Security Policy"（内容安全策略），则无需向其中添加 Anthropic 主机。

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

<Accordion title="Link your Claude organization 按钮的示例 HTML">
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

  对于深色按钮，请向链接添加 `claude-button-dark` 类。
</Accordion>

如果您的页面通过提交表单来启动连接，请使用 `<button type="submit" class="claude-button">` 代替链接。同样的样式适用。您样式表中为容器内所有按钮设置样式的规则仍可能会改变该按钮的样式。请将结果与图片进行比较。

## 颜色

| 部分 | 浅色按钮 | 深色按钮 |
| ----- | ----- | ----- |
| 背景 | `#FFFFFF` | `#141413` |
| 文字 | `#141413` | `#FAF9F5` |
| 边框 | `#87867F` | `#73726C` |
| 指针悬停在按钮上时的背景 | `#F5F4ED` | `#262624` |
| 徽标 | `#D97757` | `#D97757` |

## 下载徽标

如需在不使用 HTML 的情况下构建相同的按钮（例如在移动应用或桌面应用中），请将以下文件保存为 `claude-logo.svg`。使用[颜色](https://platform.claude.com/docs/zh-CN/manage-claude/oauth-apps/branding-samples#button-colors)中的颜色，并对屏幕阅读器隐藏徽标，因为按钮上的文字已经说明了按钮的名称。

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
