# Claude Code 安装和身份验证故障排除

这十个问题占了与 Claude Code 安装和身份验证相关的大多数支持工单。每个条目都包含最可靠的解决方案。

**注意：**如果您在 Claude Desktop 中使用 Claude Code，无需安装任何内容。使用您的付费 Claude 计划登录应用，然后打开 “Code” 标签页。如果终端安装遇到问题，Claude Desktop 也是快速解决问题的方式。

## 1. 安装后立即出现 claude: command not found。

安装程序已将 `claude` 添加到您的 PATH，但您当前的 shell 尚未读取。打开新的终端，或运行 `source ~/.zshrc`（或 `~/.bashrc`）。在 Windows 上，关闭并重新打开 PowerShell。

## 2. npm install 失败，出现 `EACCES` / 权限被拒绝。

这通常意味着安装时使用了 `sudo`，或您的全局 npm 目录由 root 拥有。不要使用 sudo。改为使用原生安装程序（`curl -fsSL https://claude.ai/install.sh | bash`），或使用 `npm config set prefix ~/.npm-global` 修复 npm 的前缀，并将该 `bin` 目录添加到您的 PATH。

## 3. “Node 版本不受支持”或启动时无声崩溃。

Claude Code 需要 **Node 18 或更高版本**。使用 `node -v` 检查您的版本。如果版本较旧，请通过 `nvm install --lts` 安装当前版本，或使用原生安装程序，它包含自己的运行时，完全避免此问题。

## 4. WSL：**claude** 运行的是 Windows Node 而不是 Linux Node。

Windows PATH 泄漏到 WSL 并覆盖 nvm。在 `~/.bashrc` 中将您的 Linux node 添加到 PATH 的前面：`export PATH="$HOME/.nvm/versions/node/$(nvm current)/bin:$PATH"`

## 5. 安装程序在企业网络后面挂起或失败。

下载主机（`storage.googleapis.com`）可能被阻止。首先使用 `export HTTPS_PROXY=http://proxy.example.com:port` 设置您的代理，然后重新运行安装程序。如果无法进行，请向您的 IT 团队索要离线包。

## 6. **SELF_SIGNED_CERT_IN_CHAIN** 或其他 TLS 错误。

您的公司注入了自己的证书。将 Node 指向企业 CA 包：`export NODE_EXTRA_CA_CERTS=/path/to/company-ca.pem` 将其添加到您的 shell 配置文件以使其持久化。

## 7. **/login** 打开浏览器，但终端从未完成（“等待身份验证…”）。

这通常意味着 localhost 回调被阻止，这在远程 SSH、devcontainers 或严格防火墙后面很常见。改为使用手动流程：复制终端中打印的 URL，在任何浏览器中完成登录，然后将返回的代码粘贴回终端。

## 8. “未身份验证”，即使您设置了 `ANTHROPIC_API_KEY`。

有三个常见原因：密钥在不同的 shell 中导出（运行 `echo` `$ANTHROPIC_API_KEY` 检查），Claude Code 尚未被告知信任此密钥（它在首次使用时提示一次；在您批准之前，`ANTHROPIC_API_KEY` 优先于任何 OAuth 会话，仅在该批准之后），或密钥是控制台密钥，但您的组织要求改为使用 SSO 登录。

## 9. Bedrock / Vertex：“无法加载凭证。”

Claude Code 使用标准提供商 SDK，因此修复方法与任何 AWS/GCP CLI 工具相同。对于 Bedrock，确认 `aws sts get-caller-identity` 有效且 `AWS_REGION` 设置为启用您的模型的区域。对于 Vertex，确认已运行 `gcloud auth application-default login` 且 `ANTHROPIC_VERTEX_PROJECT_ID` 和 `CLOUD_ML_REGION` 已设置。

## 10. 已安装并身份验证，但每个请求都出现 403 / “模型不可用。”

您的账户存在，但无权访问 Claude Code 请求的模型。对于企业席位，确认您的席位在您组织的管理员设置中处于活跃状态。在 Bedrock 或 Vertex 上，确认特定的 Claude 模型在该区域或项目中已启用。作为快速解决方案，运行 `/model` 并选择您知道有权访问的模型。

## 仍然卡住？

从您的正常 shell（不是从 Claude 会话内部）运行 `claude doctor`。它会打印一份诊断报告，您可以将其附加到支持工单。有关已知问题的完整列表，请参阅 **[故障排除指南](https://code.claude.com/docs/en/troubleshooting)**。