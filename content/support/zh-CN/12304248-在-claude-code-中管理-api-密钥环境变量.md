# 在 Claude Code 中管理 API 密钥环境变量

使用 Claude Code 时，了解身份验证方法的优先级很重要，这样可以避免意外的 API 费用，并确保您使用的是预期的账户。

## 身份验证的工作原理

- Claude Code 优先使用环境变量 API 密钥而不是已认证的订阅。

- 这是有意设计的行为，旨在让您灵活选择身份验证方法。

- 当 API 密钥设置为环境变量时，您将通过与该密钥关联的 API 账户按 API 按量付费费率收费。

- 即使您使用 claude.ai 订阅或不同的 Console 账户登录 Claude Code，也会发生这种情况。

## 最佳实践

**要使用 Claude Code 和您的 Claude 订阅：**保持 ANTHROPIC_API_KEY 环境变量未设置。

- 这可以防止意外的 API 费用，并确保您使用订阅包含的使用额。

- 如果您偶尔需要使用特定的 API 密钥，请仅在需要时临时设置。

- 定期在 Claude Code 中运行 /status 以验证您当前的身份验证方法。

## 身份验证冲突警告

当您的已认证订阅与环境变量 API 密钥之间存在冲突时，Claude Code 会通知您：

1. 在初始设置期间，如果在您的环境变量中检测到 API 密钥，Claude Code 将要求您确认要使用的身份验证方法。

2. 成功登录后，如果两个凭证都处于活跃状态，您将看到一条通知，提醒您可能产生意外的 API 费用。

## 检查您当前的配置

要验证是否将 API 密钥设置为环境变量，请在 Claude Code 中运行 /status。这将显示当前活跃的身份验证方法。

要直接检查您的环境变量，请在终端中运行以下命令之一（在 Claude Code 外部）：

macOS/Linux：

```
echo $ANTHROPIC_API_KEY
```

Windows CMD：

```
echo %ANTHROPIC_API_KEY%
```

Windows PowerShell：

```
echo $env:ANTHROPIC_API_KEY
```

## 临时设置 API 密钥

如果您只需要为当前终端会话使用 API 密钥：

macOS/Linux：

```
export ANTHROPIC_API_KEY='your-api-key-here'
```

Windows CMD：

```
set ANTHROPIC_API_KEY=your-api-key-here
```

Windows PowerShell：

```
$env:ANTHROPIC_API_KEY="your-api-key-here"
```

## 永久设置 API 密钥环境变量

macOS/Linux：

```
For zsh (default on macOS):
bash
# Add to shell config file
echo 'export ANTHROPIC_API_KEY="your-api-key-here"' >> ~/.zshrc

# Apply changes
source ~/.zshrc
For bash:
bash
# Add to shell config file
echo 'export ANTHROPIC_API_KEY="your-api-key-here"' >> ~/.bash_profile

# Apply changes
source ~/.bash_profile
```

Windows：

1. 打开系统属性 → 高级 → 环境变量

2. 在"用户变量"下，单击"新建"

3. 变量名：ANTHROPIC_API_KEY

4. 变量值：your-api-key-here

5. 单击确定并重启终端

## 删除 API 密钥环境变量

macOS/Linux（临时）：

```
unset ANTHROPIC_API_KEY
```

macOS（永久）：

```
# Remove from config file
sed -i '' '/ANTHROPIC_API_KEY/d' ~/.zshrc
source ~/.zshrc
```

Linux（永久）

```
sed -i '/ANTHROPIC_API_KEY/d' ~/.zshrc
```

Windows CMD：

```
set ANTHROPIC_API_KEY=
```

Windows PowerShell：

```
Remove-Item Env:ANTHROPIC_API_KEY
```

Windows（永久）：从系统环境变量设置中删除该变量。

## 要避免的常见问题

- 在 shell 配置文件中设置环境变量并忘记它们。

- 更改环境变量后未重启终端。

- 当您的环境中配置了 API 密钥时，假设您正在使用订阅。

如有任何问题，请**[联系我们的支持团队](https://support.claude.com/en/articles/9015913-how-to-get-support)**。