# 安装 Claude Desktop

Claude 桌面应用将 Claude 的功能直接带到您的计算机，允许与您的工作流程无缝集成。

## 可用性

Claude Desktop 可在 macOS、Windows 和 Linux（测试版）上使用。您可以执行的操作取决于您的计划：

| **功能**        | **免费** | **专业版** | **Max** | **Ultra** | **团队** | **企业版** |
| ------------- | ------ | ------- | ------- | --------- | ------ | ------- |
| 聊天            | ✅      | ✅       | ✅       | ✅         | ✅      | ✅       |
| Claude Code   |        | ✅       | ✅       | ✅         | ✅      | ✅       |
| Claude Cowork |        | ✅       | ✅       | ✅         | ✅      | ✅       |

## 系统要求

- macOS：macOS 13（Ventura）或更高版本

- Windows：Windows 10 或更高版本

- Linux：Ubuntu 22.04 LTS+ 或 Debian 12（bookworm）+，在 x64 或 arm64 上

## macOS 和 Windows 的安装步骤

1. 访问 **[Claude 下载页面](https://claude.ai/download)**。

2. 为您的操作系统选择适当的版本：

  - 对于 Mac 用户，找到 **macOS** 并点击"下载"。

  - 对于 Windows 用户，找到 **Windows** 并点击"下载"。

下载完成后：

1. 打开文件以完成安装。

2. 从您的应用程序文件夹（Mac）或开始菜单（Windows）启动 Claude。

3. 使用您的账户登录以开始使用。

## Linux 的安装步骤

### 使用 apt 安装

在 Linux 上，您可以从 Anthropic 的 apt 存储库安装 Claude Desktop，而不是作为下载的 .deb 文件，这样更新将通过您系统的常规包更新而不是应用内更新到达。

1. 在 Ubuntu 或 Debian 上，打开终端并运行：
​

```
sudo curl -fsSLo /usr/share/keyrings/claude-desktop-archive-keyring.asc https://downloads.claude.ai/claude-desktop/key.asc
echo "deb [signed-by=/usr/share/keyrings/claude-desktop-archive-keyring.asc] https://downloads.claude.ai/claude-desktop/apt/stable stable main" | sudo tee /etc/apt/sources.list.d/claude-desktop.list
sudo apt update && sudo apt install claude-desktop
```

2. 从您的应用程序菜单启动 Claude，或从终端运行 `claude-desktop`，然后登录。

**在 Linux 上保持 Claude Desktop 最新**

该应用不会自动更新。更新随您系统的常规包更新到达（`sudo apt update && sudo apt upgrade`）。

### 使用文件下载安装

**注意：**下载 .deb 不会让您选择加入常规包更新。我们强烈建议您使用 apt 安装，或使用上述步骤设置 apt 存储库。

如果您无法使用 apt 安装 Claude Desktop，可以通过下载 .deb 文件来安装：

1. 访问 **[Claude 下载页面](https://claude.ai/download)**。

2. 找到您的架构（x64 或 arm64）的 .deb 并点击"下载"。

3. 从您下载 .deb 文件的文件夹中，运行：`sudo apt install ~/Downloads/claude-desktop_amd64.deb`

4. 从您的应用程序菜单启动 Claude（或运行 `claude-desktop`），然后登录。

### 在 Linux 上验证下载（可选）

在信任之前验证签名密钥指纹：

1. 运行：`gpg --show-keys /usr/share/keyrings/claude-desktop-archive-keyring.asc`

2. 确认输出匹配：`31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE`

### 在 Linux 上使用 Claude Desktop 时的当前限制

- 计算机使用不可用。

- 听写不可用。

- 快速输入（全局热键）在 X11 上有效。在原生 Wayland 上，它依赖于您的桌面的 GlobalShortcuts 门户。

---

## 桌面扩展

桌面扩展改变了您将 Claude 连接到桌面应用程序和数据的方式。与浏览器扩展类似，这些安全的可安装包让您能够：

- **一键将 Claude 连接到您的桌面应用**：安装与您的本地文件、日历、电子邮件和消息应用集成的扩展。

- **访问精选的扩展目录**：直接在 Claude Desktop 中浏览已验证的扩展，包括文件系统访问和 iMessage 等选项。

- **享受企业级安全**：扩展使用代码签名、API 密钥等敏感数据的加密存储以及企业策略控制。

要探索桌面扩展，请在安装后在 Claude Desktop 应用中导航到 **设置 > 桌面应用**，并在 **扩展**旁边选择"管理"（或在较旧的设置布局中选择 **设置 > 扩展**）。

**注意：**如果您是想在目录中列出桌面扩展（Claude Desktop 的本地 MCP 服务器）的开发者，请通过 **[桌面扩展提交表单](https://clau.de/desktop-extention-submission)**提交。要提交插件或远程连接器，请使用 **[开发者门户](https://claude.ai/directory/manage)**。请参阅 **[提交您的插件](https://claude.com/docs/plugins/submit)**了解每条路线接受的内容。

在我们的 **[Claude Desktop 上的本地 MCP 服务器入门](https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop)**文章中阅读有关桌面扩展的更多信息。

---

## Claude Cowork

Claude Cowork 可用于所有付费计划（专业版、Max、Ultra、团队、企业版），使用 macOS、Windows 和 Linux（测试版）上的 Claude Desktop 应用。

付费 Claude 计划的用户可以在 Claude Desktop 中访问 Cowork。Cowork 将 Claude Code 的代理功能带到可视界面，允许您在不使用终端的情况下交付复杂的多步骤任务。

使用 Cowork，Claude 可以：

- 直接访问您的本地文件，无需手动上传。

- 处理长期运行的任务，如研究综合、文件组织和文档生成。

- 协调复杂项目的并行工作流。

- 提供专业输出，如具有工作公式的电子表格和格式化演示文稿。

Cowork 在您计算机上的隔离虚拟机中运行代码。文件读写限制在您连接的文件夹中，网络访问遵循您的出口设置。

要了解更多信息，请参阅 **[Cowork 入门](https://support.claude.com/en/articles/13345190-get-started-with-cowork)**。

**Linux 上 Claude Cowork 的设置要求**

Linux 上的 Cowork 有其自己的设置要求。Cowork 在您计算机上的虚拟机中运行您的任务。在它启动之前，您的机器需要：

- **硬件虚拟化 (KVM)。**大多数计算机都支持这一点，但有时在固件中被关闭。如果 Cowork 说虚拟化不可用，请在您的 BIOS 或 UEFI 设置中启用它并重新启动。

- **使用虚拟化的权限。**将自己添加到 KVM 组，然后注销并重新登录：`sudo usermod -aG kvm $USER`

- **QEMU 和支持包。**使用 `apt install` 安装 Claude 会自动拉入这些。

  - 如果您在 x64 上直接安装了 .deb 文件，请运行 `sudo apt install qemu-system-x86 ovmf virtiofsd`

  - 在 arm64 上，改为运行 `qemu-system-arm qemu-efi-aarch64 virtiofsd`。

- **工作区映像约 25 GB 的可用磁盘空间，以及至少 8 GB 的 RAM。**工作区在运行时使用 4 GB。