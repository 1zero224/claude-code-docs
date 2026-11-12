# 使用链接打开 Claude Desktop

Claude for macOS、Windows 和 Linux 响应 `claude://` URL 方案，就像浏览器响应 `https://` 方案一样。您可以从网站、脚本或其他应用使用这些链接来打开 Claude Desktop，并直接跳转到聊天、Cowork 会话或 Code 会话。

本文列出了 Claude Desktop 支持的链接格式以及每种格式接受的参数。

## 深层链接的工作原理

当您的操作系统打开 `claude://` URL 时，它会将 URL 传递给 Claude。如果应用未运行，macOS、Windows 和 Linux 将首先启动它。Claude 然后读取路径和查询参数，并导航到应用内的正确位置。

所有参数值必须进行 URL 编码。在 `q` 中传递的提示文本被截断为大约 14,000 个字符。

---

## 开始新聊天

```
claude://claude.ai/new
claude://claude.ai/new?q=Summarize%20this%20week%27s%20release%20notes
```

在 Claude Desktop 中打开新聊天。如果您包含 `q`，提示字段将预填充该文本，以便您可以查看并发送。

| **参数** | **必需** | **描述**         |
| ------ | ------ | -------------- |
| `q`    | 否      | 要在提示字段中预填充的文本。 |

---

## 打开现有聊天或项目

```
claude://claude.ai/chat/{conversation-id}
claude://claude.ai/project/{project-id}
```

按 ID 打开特定聊天或项目。ID 是您在 Claude 中聊天或项目 URL 末尾看到的 UUID。如果 ID 缺失或无效，Claude 将打开您最近的聊天或项目列表。

---

## 启动 Claude Code 会话

```
claude://code/new
claude://code/new?q=Fix%20the%20failing%20test&folder=%2FUsers%2Fme%2Frepo
```

在 Claude Desktop 中打开 Claude Code，并预填充编写器。在您自己的工具中使用此功能来实现"在 Claude Code 中打开"按钮。

| **参数**   | **必需** | **描述**                                                   |
| -------- | ------ | -------------------------------------------------------- |
| `q`      | 否      | 要在编写器中预填充的文本。prompt 可作为别名接受。                             |
| `folder` | 否      | 用作工作目录的绝对路径。Claude Desktop 会要求您在使用该文件夹之前确认。目前仅应用第一个文件夹值。 |
| `file`   | 否      | 要附加的文件的绝对路径。已接受但目前不支持。                                   |

**注意：**通过链接提供的任何文件夹都被视为不受信任。Claude Desktop 始终会显示确认对话框，然后才能将其作为工作目录采用，即使您之前已信任该文件夹。

---

## 启动 Claude Cowork 会话

```
claude://cowork/new
claude://cowork/new?q=Draft%20the%20Q2%20update&folder=%2FUsers%2Fme%2Fdocs&file=%2FUsers%2Fme
%2Fdocs%2Fnotes.md
```

打开新的 Cowork 会话，并预填充编写器。在您自己的工具中使用此功能来实现"在 Claude Cowork 中打开"按钮。

| **参数**   | **必需** | **描述**                                                   |
| -------- | ------ | -------------------------------------------------------- |
| `q`      | 否      | 要在编写器中预填充的文本。                                            |
| `folder` | 否      | 要附加的文件夹的绝对路径。重复该参数以附加多个。Claude Desktop 会要求您在使用每个文件夹之前确认。 |
| `file`   | 否      | 要附加的文件的绝对路径。重复该参数以附加多个。                                  |

**注意：**通过链接提供的任何文件夹都被视为不受信任。Claude Desktop 始终会显示确认对话框，然后才能将其作为工作目录采用，即使您之前已信任该文件夹。

---

## 测试深层链接

### 在 macOS 上

打开终端并运行：

```
open "claude://claude.ai/new?q=Hello"
```

### 在 Windows 上

打开命令提示符并运行：

```
start "" "claude://claude.ai/new?q=Hello"
```

### 在 Linux 上

打开终端并运行：

```
xdg-open "claude://claude.ai/new?q=Hello"
```