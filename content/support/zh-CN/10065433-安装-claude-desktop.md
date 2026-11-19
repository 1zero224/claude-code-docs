# 安装 Claude Desktop

Claude 桌面应用将 Claude 的功能直接带到您的计算机上，允许与您的工作流程无缝集成。

## 可用性

Claude Desktop 可在 macOS、Windows 和 Linux（测试版）上使用。您可以执行的操作取决于您的计划：

| **功能**        | **免费版** | **专业版** | **Max** | **Ultra** | **团队版** | **企业版** |
| ------------- | ------- | ------- | ------- | --------- | ------- | ------- |
| 聊天            | ✅       | ✅       | ✅       | ✅         | ✅       | ✅       |
| Claude Code   |         | ✅       | ✅       | ✅         | ✅       | ✅       |
| Claude Cowork |         | ✅       | ✅       | ✅         | ✅       | ✅       |

## 系统要求

- macOS：macOS 13 (Ventura) 或更高版本

- Windows：Windows 10 或更高版本

- Linux：Ubuntu 22.04 LTS+ 或 Debian 12 (bookworm)+，在 x64 或 arm64 上

## macOS 和 Windows 的安装步骤

1. 访问 **[Claude 下载页面](https://claude.ai/download)**。

2. 为您的操作系统选择合适的版本：

  - 对于 Mac 用户，找到 **macOS** 并点击 “下载。”

  - 对于 Windows 用户，找到 **Windows** 并点击 “下载。”

下载完成后：

1. 打开文件以完成安装。

2. 从您的应用程序文件夹 (Mac) 或开始菜单 (Windows) 启动 Claude。

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

该应用不会自动更新。更新随您系统的常规包更新到达 (`sudo apt update && sudo apt upgrade`)。

### 使用文件下载进行安装

**注意：**下载 .deb 文件不会让您自动获得定期的软件包更新。我们强烈建议您使用 apt 进行安装，或按照上述步骤设置 apt 仓库。

如果您无法使用 apt 安装 Claude Desktop，可以通过下载 .deb 文件来安装：

1. 访问 **[Claude 下载页面](https://claude.ai/download)**。

2. 找到适合您架构的 .deb 文件（x64 或 arm64）并点击 “下载。”

3. 从您下载 .deb 文件的文件夹中运行：`sudo apt install ~/Downloads/claude-desktop_amd64.deb`

4. 从应用程序菜单启动 Claude（或运行 `claude-desktop`），然后登录。

### 在 Linux 上验证下载（可选）

在信任签名密钥之前验证其指纹：

1. 运行：`gpg --show-keys /usr/share/keyrings/claude-desktop-archive-keyring.asc`

2. 确认输出与以下内容匹配：`31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE`

### 在 Linux 上使用 Claude Desktop 时的当前限制

- 计算机使用功能不可用。

- 听写功能不可用。

- 快速输入（全局快捷键）在 X11 上可用。在原生 Wayland 上，它依赖于您桌面的 GlobalShortcuts 门户。

---

## 选择发布渠道

发布渠道在所有计划中都可用，包括 Claude for Mac 和 Claude for Windows。在 Linux、Microsoft Store 安装或您的 IT 团队设置了 Claude 策略的计算机上不可用。

发布渠道设置了应用更新时您获得的 Claude Desktop 版本：

- **稳定版：**默认版本。获得新版本的频率较低，经过更多测试，因此更可靠。修复和新功能比最新版本晚几天到达。

- **最新版：**大约每天获得一个新版本，因此修复和新功能更快到达您。可能不如稳定版可靠。

### 将发布渠道更改为最新版

要更快获得新版本，请将发布渠道更改为最新版：

1. 导航到 **设置 > 桌面应用**。

2. 在 **发布渠道**下，选择 “最新版。”

更改将在您使用的计算机上应用，下次 Claude Desktop 检查更新时生效。要立即检查，请关闭设置，然后在 Mac 上选择 **Claude > 检查更新**，或在 Windows 上选择 **帮助 > 检查更新**。当您使用最新版时，**关于 Claude** 会在版本号后显示 “latest”。

### 将发布渠道更改为稳定版

要将发布渠道更改为稳定版：

1. 导航到 **设置 > 桌面应用**。

2. 在 **发布渠道**旁边，选择 “稳定版。”

**重要：**切换回稳定版不会替换您当前的版本。Claude Desktop 只会升级到更新的版本，因此您将保持最新版本，直到发布更新的稳定版本。在此之前，您不会收到稳定版更新，包括可能包含安全修复的紧急修复。如果最新版更新已下载，重启时仍会安装。不要重新安装 Claude Desktop 以加快恢复速度，因为重新安装可能会危及存储在计算机上的数据，特别是在 Windows 上。

## 如果您无法选择发布渠道

- **您的计算机受管理：**如果您的 IT 团队在您的计算机上设置了任何 Claude 策略，**发布渠道**不会显示。Claude Desktop 遵循稳定版，即使您之前选择了最新版。

- **您使用较旧的安装程序安装了 Claude for Windows：****发布渠道**呈灰显状态。要更改发布渠道，请从 **[Claude 下载页面](https://claude.ai/download)**重新安装 Claude Desktop。重新安装可能会危及存储在计算机上的数据，因此请先保存您需要的任何内容。

- **您的版本已过期：**更新到 Claude Desktop 4.29.0 或更高版本。

- **您在云提供商上使用 Claude Desktop：****发布渠道**不会显示。您的 IT 团队设置渠道。详细了解 **[发布渠道设置](https://claude.com/docs/third-party/claude-desktop/configuration#releasechannel)**。

---

## 桌面扩展

桌面扩展改变了您将 Claude 连接到桌面应用程序和数据的方式。与浏览器扩展类似，这些安全的可安装软件包让您能够：

- **一键将 Claude 连接到您的桌面应用**：安装与本地文件、日历、电子邮件和消息应用集成的扩展。

- **访问精选的扩展目录**：直接在 Claude Desktop 中浏览已验证的扩展，包括文件系统访问和 iMessage 等选项。

- **享受企业级安全**：扩展使用代码签名、敏感数据（如 API 密钥）的加密存储和企业策略控制。

要探索桌面扩展，请导航到 **设置 > 桌面应用**，并在安装后在 Claude Desktop 应用中的 **扩展**旁边选择 “管理”（或在较旧的设置布局中选择 **设置 > 扩展**）。

**注意：**如果您是想在目录中列出桌面扩展（Claude Desktop 的本地 MCP 服务器）的开发者，请通过 **[桌面扩展提交表单](https://clau.de/desktop-extention-submission)**提交。要提交插件或远程连接器，请使用 **[开发者门户](https://claude.ai/directory/manage)**。请参阅 **[提交您的插件](https://claude.com/docs/plugins/submit)**了解每条路线接受的内容。

在我们的 **[Claude Desktop 本地 MCP 服务器入门](https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop)**文章中了解有关桌面扩展的更多信息。

---

## Claude Cowork

Claude Cowork 适用于所有付费计划（Pro、Max、Ultra、Team、Enterprise），可在 macOS、Windows 和 Linux（测试版）上的 Claude Desktop 应用中使用。

付费 Claude 计划的用户可以在 Claude Desktop 中访问 Cowork。Cowork 将 Claude Code 的代理功能带到可视界面，让您可以在不使用终端的情况下交付复杂的多步骤任务。

使用 Cowork，Claude 可以：

- 直接访问您的本地文件，无需手动上传。

- 处理研究综合、文件组织和文档生成等长期运行的任务。

- 为复杂项目协调并行工作流。

- 提供专业输出，如包含有效公式的电子表格和格式化演示文稿。

Cowork 在您计算机上的隔离虚拟机中运行代码。文件读写仅限于您连接的文件夹，网络访问遵循您的出站设置。

如需了解更多信息，请参阅 **[Cowork 入门](https://support.claude.com/en/articles/13345190-get-started-with-cowork)**。

**Linux 上 Claude Cowork 的设置要求**

Linux 上的 Cowork 有其自己的设置要求。Cowork 在您计算机上的虚拟机中运行您的任务。在启动之前，您的计算机需要：

- **硬件虚拟化 (KVM)。**大多数计算机都支持此功能，但有时在固件中被关闭。如果 Cowork 显示虚拟化不可用，请在 BIOS 或 UEFI 设置中启用它并重新启动。

- **使用虚拟化的权限。**将自己添加到 KVM 组，然后注销并重新登录：`sudo usermod -aG kvm $USER`

- **QEMU 和支持包。**使用 `apt install` 安装 Claude 会自动拉取这些包。

  - 如果您在 x64 上直接安装了 .deb 文件，请运行 `sudo apt install qemu-system-x86 ovmf virtiofsd`

  - 在 arm64 上，请改为运行 `qemu-system-arm qemu-efi-aarch64 virtiofsd`。

- **约 25 GB 的工作区镜像可用磁盘空间，以及至少 8 GB 的 RAM。**工作区运行时使用 4 GB。