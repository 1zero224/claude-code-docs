# 为 macOS 部署 Claude Desktop

Team 或 Enterprise 计划的管理员可以自动部署 Claude Desktop 以集中管理安装和更新。Claude Desktop 安装到 `/Applications`，除非通过企业策略禁用，否则在发布新版本时会自动更新。

# 可用的安装格式

- **`.pkg`** - 标准 macOS 安装程序包（推荐用于企业）

- **`.dmg`** - 拖放安装

# Cowork 要求

当您下载并安装 Claude Desktop for macOS 时，Cowork 将自动安装。

# 下载

**[通用版本（x64 或 arm64）Claude PKG](https://claude.ai/api/desktop/darwin/universal/pkg/latest/redirect)**

通用版本与 Intel 和 Apple Silicon 机器兼容，支持所有 Mac 硬件。

# 通过 MDM 部署

将 PKG 上传到您的 MDM 解决方案（Jamf、Kandji、Microsoft Intune）并部署到目标机器。

# 配置

要配置 Claude Desktop 设置（如自动更新、扩展和 MCP 服务器），请参阅 **[企业配置](https://support.claude.com/en/articles/12622667-enterprise-configuration)** 文章。

# 故障排除

## 用户无法更新 Claude Desktop

Claude Desktop 可以安装到用户的 Applications 文件夹或系统 Applications 文件夹，这会影响更新权限。

- `~/Applications`（用户文件夹）：用户可以在没有管理员权限的情况下更新

- `/Applications`（系统文件夹）：用户需要对 `/Applications`、`Claude.app` 和所有包含的文件具有管理员访问权限和写入权限

对于共享机器，通过企业策略禁用自动更新，并通过 MDM 解决方案集中管理更新。