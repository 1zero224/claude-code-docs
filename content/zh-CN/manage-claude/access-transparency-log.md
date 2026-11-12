---
title: 使用透明度日志验证访问透明度事件
url: https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log
description: 使用 Compliance API 提供的签名检查点和 Merkle 证明，验证没有任何访问透明度事件在提交到日志后被删除或篡改。
featureMetadata:
  status: beta
---

了解如何以密码学方式验证没有任何[访问透明度](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency)事件在提交到您组织的透明度日志后被删除或篡改。

<Note>
  透明度日志目前处于 beta 阶段，其端点和响应结构在 beta 期间可能会发生变化。它是访问透明度的一部分，访问透明度应请求向符合条件的客户提供，不支持自助开通（请参阅[访问透明度](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency)）。一旦启用访问透明度，Anthropic 就会为您的组织维护一个透明度日志。您可以通过 Compliance API 读取它，所用的密钥和权限范围与[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)相同。

  启用后，当您的组织记录第一个访问透明度事件时，您的日志即被创建。在此之前，所有透明度日志端点都会返回 `404`。

  如果您已经在定期运行 `axt-verify`，请参阅[为新的叶子版本升级](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#upgrade-for-a-new-leaf-version)。

  透明度日志仅供您参考。它并非为满足任何安全、隐私或监管要求而设计或认证。您有责任自行判断它是否符合您自身的义务。
</Note>

## 透明度日志的工作原理

“Transparency log”（透明度日志）是一种使对记录的任何篡改都能被检测到的技术。条目只会被追加。每当日志增长时，其运营方都会签署一份简短的声明，称为 “checkpoint”（检查点），它通过 Merkle 树哈希对迄今为止的所有条目作出承诺。任何保存了检查点的人之后都可以要求证明：当前日志仍然包含该检查点所覆盖的全部内容，且未被更改、顺序不变。因此，对于保存了覆盖某条目的检查点的验证者而言，删除或重写该条目不可能不被察觉。Certificate Transparency 和 Go 模块校验和数据库都基于同样的技术构建。[C2SP tlog-tiles](https://c2sp.org/tlog-tiles) 是一个开放规范，用于将此类日志以签名检查点加上静态、可缓存的哈希和条目瓦片（tile）的形式提供，使客户端能够获取哈希并自行计算每一个证明。

启用访问透明度后，Anthropic 会为您的组织维护一个透明度日志。它是访问透明度事件（`anthropic_access` 和 `cmek_preserve`）的一份仅追加、经密码学签名的记录。日志创建后为您的组织记录的每个此类事件都会被追加到其中。该日志遵循 C2SP tlog-tiles 格式，因此为该标准构建的工具可以解析其检查点、瓦片和证明。

* **每个组织一个日志。** 每个组织的日志都有一个固定的 origin 字符串：`axt.anthropic.com/<your organization UUID>`。在组织的整个生命周期中，origin 永不改变。
* **每个新事件都成为一个叶子。** 当某个访问透明度事件符合在您的活动源上显示的条件时，它会首先作为叶子追加到您的日志中，然后才会在活动源上提供。叶子是事件中文档所列字段的确定性序列化。您活动源上的事件带有 `transparency_log_leaf_index`，即它在您日志中从零开始的位置。
* **检查点对整个历史作出承诺。** 日志是一棵 Merkle 树。每当它增长时，Anthropic 都会发布一个签名检查点：一份简短的文本文档，说明日志的 origin、当前大小以及对所有叶子作出承诺的根哈希。每个检查点恰好带有一个来自日志签名密钥的签名。
* **由此产生两种证明。** 包含证明（inclusion proof）表明某个特定事件在某个检查点下存在于其位置上。一致性证明（consistency proof）表明较新的检查点是您保存的较早检查点的仅追加扩展，因此两者之间没有任何内容被删除或更改。
* **验证密钥在带内提供。** [验证者密钥端点](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-the-verifier-keys)返回用于签署检查点的公钥。在计划内的密钥轮换中，新密钥会在开始签名之前被添加到该列表中，较早的密钥仍保留在列表中。因此，您已持有的检查点仍可继续通过验证。

### 透明度日志能证明什么

* 您持有的事件的叶子字段（列于[事件如何成为叶子](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#how-an-event-becomes-a-leaf)中）与 Anthropic 提交到日志的内容逐字节一致。
* 您组织的日志只会增长。如果已提交到日志的事件之后在日志中被删除或重写，针对您持有的检查点的验证就会失败。如果向您提供的历史与之前提供给您的历史不同，验证也会失败。
* 新条目只能被追加。事件无法被插入到您已验证过的历史中。

### 它不能证明或改变什么

* 它不能证明每一次访问都被记录了，也不能证明已记录的事件准确描述了该访问。它只能证明 Anthropic 提交到日志的内容此后未被更改。
* 它不会改变[访问透明度涵盖的内容](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency#what-access-transparency-covers)或事件到达的时间。
* 任何[叶子版本](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#how-an-event-becomes-a-leaf)都不包含的已提供字段（例如 `workspace_uuid`）不在证明范围内。之后添加到事件中的字段，只有在某个叶子版本包含它之后才会被覆盖。
* 包含证明只针对提供给您的事件。它本身并不能证明活动源列出了日志所持有的每一个叶子。日志的[条目包](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-an-entry-bundle)包含每一个叶子，因此在需要时，您可以直接读取已提交事件的完整集合。
* 事件上存在 `transparency_log_leaf_index` 只是一个指针，而不是证明。在将事件视为已提交到日志之前，务必先验证其包含性。
* 针对历史被重写的保护来自您保存的检查点。检查点由[已发布的密钥指纹](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#published-key-fingerprints)中列出的密钥签名，这证明它来自 Anthropic 的日志。从您上次保存的检查点出发的一致性证明，则证明您已观察到的历史只是在增长。

## 开始之前

您需要：

* 一个具有 `read:compliance_activities` 权限范围的 Compliance Access Key，即您用于[活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)的同一密钥和权限范围。父组织密钥可以通过在每个请求中指定子组织来读取每个已注册子组织的日志。
* 您组织的 UUID。可在 Claude Console 的[组织](https://platform.claude.com/settings/organization)页面（`platform.claude.com/settings/organization`）上找到。它与活动源作为 `organization_uuid` 提供的值相同，但请从 Console 获取。正是这个值使检查点归属于您，因此它不能来自您正在验证的 API。您可以由它推导出日志的 origin：`axt.anthropic.com/<organization UUID>`。请自行推导此字符串，不要从 API 响应中读取。
* 一个可持久保存您上次验证的检查点的位置。正是这个保存的检查点，将"日志今天是一致的"变为"自您开始监控以来，日志一直是一致的"。

## 时间安排

* **事件：** 访问透明度事件会在访问发生后 2 个工作日内出现在您的活动源上。事件只有在符合提供条件后才会进入日志，因此日志永远不会提前暴露事件。由于日志在活动源提供事件之前写入，某个条目可能会在其事件出现在您的活动源上之前短暂地出现在日志中。这种时间差并非不一致。
* **检查点：** 每当您的日志增长时，都会发布一个新的检查点。
* **包含证明：** 一旦覆盖事件位置的检查点发布，新提供事件的证明即可获取，通常在事件出现后很快就会发布。如果您过早请求，会收到 `404`，请稍后重试。
* **验证频率：** 至少每天运行一次验证。每小时运行一次也是合理的。
* **取消注册：** 如果您的组织停止使用访问透明度，不会删除任何内容。您的日志仍可通过相同的端点读取。如果之后再次启用访问透明度，将继续使用同一个日志。

## 保留与删除

* **透明度日志：** Anthropic 不会从您的日志中删除条目，日志也没有过期时间。即使您的组织停止使用访问透明度，或在您的组织被删除之后，日志仍会保留，因为删除条目正是日志存在所要检测的变更。[条目包](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-an-entry-bundle)保存每个事件的[叶子字段](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#how-an-event-becomes-a-leaf)，因此这些字段会与日志保留同样长的时间。
* **活动源：** 活动源上的访问透明度事件遵循活动源的保留策略。活动会保留 6 年。请参阅[查询活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)。
* **您无法删除：** 透明度日志端点是只读的。无法删除或修改条目。

## 透明度日志端点

在 `https://api.anthropic.com/v1/compliance/transparency_log/` 下提供六个只读端点：

| 端点                                                                                                                               | 返回内容                 |
| -------------------------------------------------------------------------------------------------------------------------------- | -------------------- |
| [`GET /checkpoint`](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-the-latest-checkpoint)     | 最新的签名检查点             |
| [`GET /keys`](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-the-verifier-keys)               | 验证者密钥集               |
| [`GET /inclusion`](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#fetch-an-inclusion-proof)        | 单个事件的包含证明            |
| [`GET /consistency`](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#fetch-a-consistency-proof)     | 从您持有的检查点到最新检查点的一致性证明 |
| [`GET /tile/{level}/{index}`](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-a-hash-tile)     | Merkle 哈希瓦片          |
| [`GET /tile/entries/{index}`](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-an-entry-bundle) | 叶子字节的条目包             |

检查点、瓦片和条目包完全遵循 C2SP tlog-tiles 传输格式。两个证明端点只是为了方便：每个证明也都可以从瓦片计算得出，因此您永远不必信任证明端点的输出。您需要针对签名检查点验证它返回的哈希。

### 身份验证与范围

与每个 Compliance API 请求一样，在 `x-api-key` 标头中发送您的 Compliance Access Key，并发送 `anthropic-version` 标头（请参阅[版本控制](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api#versioning)）。您的组织必须已启用 Compliance API。

没有单独的透明度日志权限。任何能够读取您组织活动源的密钥（无论是您组织的还是其父组织的），都可以读取您的整个日志，包括其条目包中的事件字段。

每个请求恰好读取一个组织的日志：

* 组织级密钥读取其自身组织的日志。`organization_id` 查询参数是可选的。如果提供，它必须指定该密钥自身所属的组织。
* 父组织级密钥必须传递 `organization_id`，指定一个子组织。
* `organization_id` 接受 `org_...` 标记 ID 或组织 UUID。

`404` 表示不存在此密钥可读取的日志。超出密钥范围的组织、不存在的组织以及日志尚未创建的组织，被有意设计为无法区分。已停止使用访问透明度的组织的日志不属于这种情况：它会继续被提供。

### 错误

所有端点（包括返回文本和二进制内容的端点）的错误都使用标准的 Compliance API JSON 错误封装。有关封装格式和共享错误类型，请参阅[处理 Compliance API 错误](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors)。

| 状态    | 在此接口上的含义                                                                                                                        |
| ----- | ------------------------------------------------------------------------------------------------------------------------------- |
| `400` | `organization_id` 格式错误或在使用父组织密钥时被省略、未启用 Compliance API、查询参数未知，或某个端点特定的验证失败                                                      |
| `401` | API 密钥缺失或无效                                                                                                                     |
| `403` | 密钥缺少所需的权限范围                                                                                                                     |
| `404` | 没有此密钥可读取的日志，或端点特定的"未覆盖"和"超出树范围"情况                                                                                               |
| `429` | 已被限速。这些端点共享 Compliance API 按父组织计算的[速率限制](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api)。请遵守 `retry-after` |
| `503` | 日志暂时不可用。请使用退避策略重试                                                                                                               |

### 缓存

响应只能由发出请求的客户端缓存。`Cache-Control` 始终包含 `private`，且响应带有 `Vary: x-api-key`。不要在这些端点前放置共享缓存。完整瓦片和完整条目包永不改变，并以 `Cache-Control: private, max-age=604800, immutable` 提供。其他所有内容，包括检查点、证明、密钥、部分瓦片和错误，都以 `Cache-Control: private, no-store` 提供。

### 读取最新检查点

`GET /v1/compliance/transparency_log/checkpoint`

```bash
curl --fail-with-body -sS \
  "https://api.anthropic.com/v1/compliance/transparency_log/checkpoint" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

响应为 `text/plain`：一个 C2SP [签名注释（signed note）](https://c2sp.org/signed-note)。正文各行依次为 origin、十进制的树大小以及 base64 编码的根哈希。随后是一个空行，然后是签名行，签名行以长破折号（U+2014）开头，指明 origin，并以一个 base64 值结尾。此处的值仅作示例：

```text wrap
axt.anthropic.com/25f6429a-3293-49bf-afed-cb312911554b
1207
C6C4HzGRqDNlbu54LWCvpDX0NcB5DRTmLcjM4u5vWUI=

— axt.anthropic.com/25f6429a-3293-49bf-afed-cb312911554b q83vATBFAiEAvL8m…(base64)…
```

* 解码后签名值的前 4 个字节是签名密钥的 `key_hash`，它告诉您应使用[验证者密钥集](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-the-verifier-keys)中的哪个条目进行验证。其余字节是对注释正文的 SHA-256 所做的 ASN.1 DER ECDSA P-256 签名：注释正文指空行之前的所有字节，包括正文末尾的换行符。
* 检查点可以在根哈希之后带有额外的行。忽略您不理解的行。它们也在签名覆盖范围内。
* 忽略名称不是您的 origin 或您未持有其密钥哈希的签名行。
* 切勿缓存检查点。过时的检查点会隐藏日志的当前大小，导致新提供的事件看起来未被覆盖。

### 读取验证者密钥

`GET /v1/compliance/transparency_log/keys`

```bash
curl --fail-with-body -sS \
  "https://api.anthropic.com/v1/compliance/transparency_log/keys" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

```json
{
  "type": "transparency_log_keys",
  "origin": "axt.anthropic.com/25f6429a-3293-49bf-afed-cb312911554b",
  "log_keys": [
    {
      "verifier_key": "axt.anthropic.com/25f6429a-3293-49bf-afed-cb312911554b+<key_hash>+<base64 key>",
      "key_hash": "<8 lowercase hex digits>",
      "fingerprint": "<64 lowercase hex digits>",
      "algorithm": "ecdsa_p256_sha256",
      "public_key": "<base64 DER SubjectPublicKeyInfo>"
    }
  ]
}
```

| 字段                        | 类型     | 描述                                                                                                                                                                                            |
| ------------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`                    | string | 始终为 `transparency_log_keys`                                                                                                                                                                   |
| `origin`                  | string | 此日志每个检查点所带的 origin 行。仅供参考：请将检查点与您自行推导的 origin 进行比较，而不是与此字段比较                                                                                                                                  |
| `log_keys`                | array  | 首先是签署新检查点的密钥，然后是日志提供的其他所有密钥，按从新到旧排列。永不为空                                                                                                                                                      |
| `log_keys[].verifier_key` | string | 以 C2SP note-verifier 字符串形式表示的密钥，`<origin>+<key_hash>+<base64 key>`，支持 ECDSA 注释密钥的 tlog-tiles 工具（例如 Go 的 `github.com/transparency-dev/formats` 模块）可接受此格式。base64 部分解码后为算法字节 `0x02`，后跟 DER 编码的公钥 |
| `log_keys[].key_hash`     | string | 八位小写十六进制数字。用于将此密钥与检查点签名行匹配的 4 字节选择器。它是 `fingerprint` 的前 4 个字节，并不是身份标识                                                                                                                         |
| `log_keys[].fingerprint`  | string | 64 位小写十六进制数字：DER `SubjectPublicKeyInfo` 的 SHA-256                                                                                                                                             |
| `log_keys[].algorithm`    | string | 密钥类型，目前为 `ecdsa_p256_sha256`。可能会添加新值。跳过您不支持其算法的密钥                                                                                                                                             |
| `log_keys[].public_key`   | string | 以 base64 DER `SubjectPublicKeyInfo` 表示的公钥                                                                                                                                                     |

`key_hash` 只覆盖密钥字节：它是对 DER `SubjectPublicKeyInfo` 计算的 SHA-256 的前 4 个字节，这正是 ECDSA note-verifier 编码所使用的规则。它不是基础签名注释格式为 Ed25519 密钥定义的、依赖名称的密钥 ID，因此不会随 origin 改变。由[已发布的密钥指纹](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#published-key-fingerprints)中列出的密钥产生的有效签名，证明该检查点来自 Anthropic 的透明度日志服务。签名检查点内的 origin 行才是将其绑定到您组织的依据。这就是您需要将该行与您自行推导的 origin 进行比较的原因。

密钥可能会轮换：

* 轮换是一次切换。从某个检查点开始，新检查点由新密钥签名。
* 在计划内的轮换中，新密钥会在签署任何内容之前出现在 `log_keys` 中，较早的密钥仍保留在列表中。因此，您在轮换前保存的检查点仍可继续通过验证。
* 验证者可以在每次运行时获取密钥集，也可以在本地保存。在本地保存密钥集的验证者，在遇到其未持有 `key_hash` 的签名时，会重新读取此端点。

#### 已发布的密钥指纹

Anthropic 在此处（API 之外）发布每个签署检查点的密钥的指纹。这使您可以对照一个提供路径无法篡改的来源，检查您在本地持有的密钥。您持有的密钥可能来自之前的 `GET /keys` 响应，也可能来自固定（pin）该密钥的工具。

| 密钥哈希       | SHA-256 指纹                                                         | 算法                  | 开始签名日期     | 状态     |
| ---------- | ------------------------------------------------------------------ | ------------------- | ---------- | ------ |
| `1dff5fe4` | `1dff5fe420d49743fe444a04fc17f818eea856699dec2ebbc24df15602c74a58` | `ecdsa_p256_sha256` | 2026-08-17 | 当前签名密钥 |

计划内的轮换会在新密钥签署其第一个检查点前至少 30 天在本页公布。在此通知期内，新密钥会连同其切换日期一起列在 `log_keys` 和此表中。已退役的密钥会连同其服务日期继续列出。未出现在此表中的密钥都不是合法密钥，无论 `GET /keys` 返回什么。将无法在任何已列出密钥下通过验证的检查点视为验证失败，并将其报告给您的 Anthropic 客户代表或 [Anthropic 支持](https://support.claude.com)。

`axt-verify` 在每个版本中内置当前密钥，且从不从 API 读取密钥（请参阅[使用 axt-verify 进行验证](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#verify-with-axt-verify)）。每个版本恰好内置一个密钥。在切换日期，Anthropic 开始使用新密钥签名，并发布内置该密钥的 `axt-verify` 版本。在同一日期，Anthropic 会使用新密钥重新签发每个组织的最新检查点，即使日志没有增长也是如此。请在切换日期进行升级。在切换后运行旧版本会以退出状态 `1` 失败，在切换前运行新版本也会如此。一旦您运行了匹配的版本，这两种失败都会消失。对于您自行维护的验证者，需要在切换日期之前添加新指纹及其切换日期。

### 获取包含证明

`GET /v1/compliance/transparency_log/inclusion?leaf_index={index}`

| 参数                | 类型         | 描述                                                                                                                    |
| ----------------- | ---------- | --------------------------------------------------------------------------------------------------------------------- |
| `leaf_index`      | integer，必需 | 事件在日志中的位置：活动源在该事件上提供的 `transparency_log_leaf_index`。必须大于或等于零                                                          |
| `organization_id` | string，可选  | 请参阅[身份验证与范围](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#authentication-and-scoping) |

```bash
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/transparency_log/inclusion" \
  --data-urlencode "leaf_index=41" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

```json
{
  "type": "transparency_log_inclusion_proof",
  "leaf_index": 41,
  "hashes": [
    "mUdyOWMp0zXIq0CDMvSYDUSBl9yAvnTZzdm51RwWpUM=",
    "yR6tDHkAhKvdQSLqQATVjXOo4GM3FDyiKF2XCKTtMUI=",
    "..."
  ],
  "checkpoint": "axt.anthropic.com/25f6429a-3293-49bf-afed-cb312911554b\n42\nCsRlS31ITFHrX9GR5XjyPw8n0MkfrB8Yh2UDHl3Lr3E=\n\n— axt.anthropic.com/25f6429a-3293-49bf-afed-cb312911554b q83vATBEAiB0…(base64)…\n"
}
```

| 字段           | 类型               | 描述                                     |
| ------------ | ---------------- | -------------------------------------- |
| `type`       | string           | 始终为 `transparency_log_inclusion_proof` |
| `leaf_index` | integer          | 事件在日志中的位置，回显自请求                        |
| `hashes`     | array of strings | 审计路径的 base64 兄弟哈希，按从叶子到根的顺序排列          |
| `checkpoint` | string           | 最新的签名检查点，即证明所针对验证的检查点。它带有树大小           |

不支持按活动 ID 查找。您始终持有索引，因为它随事件一起到达，并且您需要针对从活动源获取的事件字节来检查证明。

`404` 表示最新发布的检查点未覆盖所提供的位置：

* 对于您从已提供事件中读取的索引，这是暂时的。覆盖它的检查点很快就会发布，因此请稍后重试。
* 对于任何其他未覆盖的位置（例如活动源从未提供过的索引），也会返回同样的 `404`。对于此类位置，不保证会发布覆盖它的检查点。响应不会说明您属于哪种情况。

缺失或不是非负整数的 `leaf_index` 会返回 `400`。

### 获取一致性证明

`GET /v1/compliance/transparency_log/consistency?from={size}`

| 参数                | 类型         | 描述                                                                                                                    |
| ----------------- | ---------- | --------------------------------------------------------------------------------------------------------------------- |
| `from`            | integer，必需 | 您持有的较早检查点的树大小。必须至少为 1，且不超过最新检查点的树大小                                                                                   |
| `organization_id` | string，可选  | 请参阅[身份验证与范围](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#authentication-and-scoping) |

```bash
curl --fail-with-body -sS -G \
  "https://api.anthropic.com/v1/compliance/transparency_log/consistency" \
  --data-urlencode "from=1180" \
  -H "x-api-key: $ANTHROPIC_COMPLIANCE_ACCESS_KEY" \
  -H "anthropic-version: 2026-10-21"
```

```json
{
  "type": "transparency_log_consistency_proof",
  "hashes": [
    "dGw0aPzu2N0pdc4C5ZAvNIbkXF7J6F9ZQLkPpV6v8Vg=",
    "9PSWm1T9RUmhjF6z6YQzB9CW6E2m2n3mK0aVgqf5Qm0=",
    "..."
  ],
  "checkpoint": "axt.anthropic.com/25f6429a-3293-49bf-afed-cb312911554b\n1207\nC6C4HzGRqDNlbu54LWCvpDX0NcB5DRTmLcjM4u5vWUI=\n\n— axt.anthropic.com/25f6429a-3293-49bf-afed-cb312911554b q83vATBFAiEAvL8m…(base64)…\n"
}
```

| 字段           | 类型               | 描述                                       |
| ------------ | ---------------- | ---------------------------------------- |
| `type`       | string           | 始终为 `transparency_log_consistency_proof` |
| `hashes`     | array of strings | base64 证明哈希，按 RFC 9162 顺序排列              |
| `checkpoint` | string           | 最新的签名检查点，即证明所延伸到的检查点。它带有树大小              |

* 证明始终延伸到最新发布的检查点。此 API 从不提供历史检查点：您需要自行保存提供给您的检查点。
* `from` 等于最新检查点的树大小时，返回空证明。
* 持有的树大小为 0 的检查点不需要一致性证明，因为每个日志都是空日志的扩展。在这种情况下，直接采用最新检查点即可。
* `from` 小于 1 或大于最新检查点的树大小时，返回 `400`。
* 如果日志无法再证明它扩展了曾经为您签署的某个检查点，请将其视为验证失败，而不是使用错误。

### 读取哈希瓦片

`GET /v1/compliance/transparency_log/tile/{level}/{index}`

返回 `application/octet-stream`：按 tlog-tiles 规范拼接的 32 字节 SHA-256 哈希。瓦片寻址完全遵循 tlog-tiles，包括 `{level}` 和 `{index}` 路径语法、大型树使用的 `x001/234` 索引形式，以及部分瓦片后缀 `.p/{width}`。哈希瓦片是 tlog-tiles 客户端自行计算证明的基本单位。

* 完整瓦片是不可变的，并以 `Cache-Control: private, max-age=604800, immutable` 提供。
* 部分瓦片会随着树的增长而被取代，并以 `Cache-Control: private, no-store` 提供。一旦瓦片填满，对其早期部分形式的请求可能会返回 `404`，即使完整瓦片已存在。按照 tlog-tiles 的规定，从部分瓦片回退到完整瓦片是客户端的职责，标准客户端已经这样做了。
* 格式错误的 `level`、`index` 或部分瓦片宽度会返回 `400`。超出当前树大小的瓦片位置会返回 `404`。

### 读取条目包

`GET /v1/compliance/transparency_log/tile/entries/{index}`

返回 `application/octet-stream`：按 tlog-tiles 规范排列的连续叶子条目，每个条目都以其大端 `uint16` 长度作为前缀。条目包包含事件明文：每个访问透明度事件的[规范字节](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#how-an-event-becomes-a-leaf)。这就是整个接口都需要活动源权限范围的原因。寻址、部分形式、缓存和错误均与哈希瓦片相同。

## 活动源事件上的 `transparency_log_leaf_index` 字段

`GET /v1/compliance/activities` 上的 `anthropic_access` 和 `cmek_preserve` 事件，只要该事件有叶子，就会带有整数类型的 `transparency_log_leaf_index`。其他活动类型从不带有此字段。

* **当事件没有叶子时，该键不存在，而不是 `null`。** 健壮的验证者会以相同方式处理键不存在和值为 `null` 的情况。
* **事件仅在两种情况下提供时不带 `transparency_log_leaf_index`。** 第一种是您的组织未注册访问透明度期间，即注册之前，或在取消注册与重新注册之间。第二种是事件在您组织的日志创建之前被记录。对于在透明度日志推出之前就已注册的组织，这包括其较早的历史。一旦您的日志存在且您处于注册状态，每个事件都会在活动源提供它之前被追加到日志中。如果某个故障导致活动源无法获取索引，活动源会延迟提供该事件，而不是在没有索引的情况下提供它。该事件不会丢失：它已经在日志中，并会在故障修复后出现在活动源上，且带有索引。日期在您的日志创建之后、并处于您注册期间内的事件，不应出现缺少索引的情况。
* **存在的索引只是一个指针，而不是证明。** 在将事件视为已提交到日志之前，请先验证其包含性。值得上报的异常是：存在索引，但在覆盖它的检查点本应发布很久之后，仍然无法获取其包含证明。
* 索引在事件被追加到日志时分配，它不属于构成叶子的字段。

## 事件如何成为叶子

叶子条目由一个 schema 版本字节加上该版本字段的规范 JSON 组成。该 JSON 遵循 [RFC 8785](https://www.rfc-editor.org/rfc/rfc8785)（JSON Canonicalization Scheme），字段完全按照活动源提供的事件内容取值。

叶子版本是一组固定的字段，每个字段要么是必需的，要么是可选的。

版本 `0x01` 有 11 个必需字段，没有可选字段：

* `id`、`type`、`created_at`、`accessed_at`、`organization_id`、`organization_uuid`、`workspace_id`、`accessor_department` 和 `reason_code`
* `actor`，及其嵌套的 `type` 和 `email_address`
* `resource_details`，及其嵌套的 `type`、`id` 和 `parent`

版本 `0x02` 具有相同的 11 个必需字段和一个可选字段：

* `user_profile_id`，即被访问或被保留的数据创建时所属的[用户配置文件](https://platform.claude.com/docs/zh-CN/api/beta/user_profiles)的 ID

一旦您的日志开始写入某个版本，它就会以该版本写入每个新叶子，直到开始写入下一个版本为止，即使事件不含该版本的任何可选字段也是如此。[为新的叶子版本升级](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#upgrade-for-a-new-leaf-version)说明了您的日志何时可以开始写入新版本。叶子一旦写入就永不改变，因此它会永久保持其版本。因此，一个日志中可能包含多个版本的叶子。活动源或证明端点提供的任何内容都不会说明叶子的版本（只有[条目包](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-an-entry-bundle)中叶子自身的字节会说明），而您也无需知道：以下规则中的最后一条说明了应为事件重建哪些条目。

规则如下：

* 版本字段集之外的已提供字段（例如 `workspace_uuid` 以及 `transparency_log_leaf_index` 本身）会被忽略。

* 已提供事件省略或以 `null` 提供的必需字段，以 `null` 进入叶子。空字符串与 `null` 不同。

* 可选字段仅在已提供事件中以字符串形式（包括空字符串）存在时才进入叶子，并以该字符串进入。当已提供事件省略它或以 `null` 提供时，叶子中完全不含该键。可选字段在叶子中永远不会是 `null`。如果已提供事件中某个可选字段为其他任何 JSON 值，请拒绝该事件。

* 当已提供事件带有 `actor` 和 `resource_details` 时，它们是恰好包含其已记录键的对象。`actor.email_address` 和 `resource_details.parent` 始终为 `null`。当已提供事件省略其中某个对象或以 `null` 提供时，整个值在叶子中为 `null`，而不是一个字段全为 `null` 的对象。许多访问事件不带 `resource_details`。

* 字符串值（包括两个时间戳和 `reason_code`）按提供的内容逐字节获取。如果您从其他表示形式重新推导时间戳，请精确重现所提供的呈现形式：

  * 带 `Z` 后缀的 RFC 3339 UTC。
  * 当 `created_at` 的微秒为零时，它没有小数位，否则恰好有六位小数。
  * `accessed_at` 有零、三、六或九位小数，取能精确保留其纳秒的最短形式。

* 规范 JSON 意味着对象键已排序、没有无意义的空白，并使用最少的字符串转义。叶子中任何地方都不会出现数字。

* 叶子哈希为 RFC 6962 的 `SHA-256(0x00 || entry)`。内部节点的哈希为 `SHA-256(0x01 || left || right)`。

* 读取叶子条目（例如从[条目包](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-an-entry-bundle)中读取）的验证者，会拒绝未知的版本字节，以及 `type` 不属于两种访问透明度类型之一的叶子。新字段、新事件类型和规则变更都以新的版本字节发布。现有叶子永不重新计算哈希。

* 对于已提供的事件，请在其字段包含该事件所有以字符串形式存在的可选字段的每个叶子版本下重建其条目（如果事件不含可选字段，则为每个版本），而不在其他任何版本下重建。如果其包含证明与其中任意一个的叶子哈希匹配，该事件即通过验证。如果向您提供的事件中有可选字段被删除、添加或更改，那么无论日志中保存的是哪个叶子，该事件在任何以包含该字段的叶子版本进行重建的验证者上都会验证失败。

例如，活动源提供的这个访问事件：

```json
{
  "id": "activity_01GPXmAhizavrUuoXNn3tzeA",
  "type": "anthropic_access",
  "created_at": "2025-07-08T18:40:00Z",
  "accessed_at": "2025-07-08T18:39:58Z",
  "organization_id": "org_015gtSHLz269eTwgrH8NX5yk",
  "organization_uuid": "25f6429a-3293-49bf-afed-cb312911554b",
  "workspace_id": "wrkspc_01PaGUP2rbg1XDh7Z9W1CEpd",
  "workspace_uuid": "b6ce2143-1083-d4a7-247c-17530f55a076",
  "accessor_department": "Trust & Safety",
  "reason_code": "safety_review",
  "actor": { "type": "anthropic_actor", "email_address": null },
  "resource_details": { "type": "message", "id": "msg_01HXAMPLE12345678" },
  "transparency_log_leaf_index": 17
}
```

会变为以下规范 JSON。它恰好包含 11 个必需键，已排序，位于一行中。`workspace_uuid` 和 `transparency_log_leaf_index` 被去除，而已提供事件中不存在的 `resource_details.parent` 以 `null` 进入：

```json
{"accessed_at":"2025-07-08T18:39:58Z","accessor_department":"Trust & Safety","actor":{"email_address":null,"type":"anthropic_actor"},"created_at":"2025-07-08T18:40:00Z","id":"activity_01GPXmAhizavrUuoXNn3tzeA","organization_id":"org_015gtSHLz269eTwgrH8NX5yk","organization_uuid":"25f6429a-3293-49bf-afed-cb312911554b","reason_code":"safety_review","resource_details":{"id":"msg_01HXAMPLE12345678","parent":null,"type":"message"},"type":"anthropic_access","workspace_id":"wrkspc_01PaGUP2rbg1XDh7Z9W1CEpd"}
```

此事件不含可选字段，因此其版本 `0x02` 和版本 `0x01` 的条目包含相同的 JSON，仅第一个字节不同。版本 `0x02` 条目是字节 `0x02` 后跟这些 UTF-8 字节。其叶子哈希 `SHA-256(0x00 || entry)` 的 base64 编码为 `1iKC1LbEakcNTfp8OwW4LtItQu4+/K/pzaDDLVIA5XE=`。版本 `0x01` 条目是字节 `0x01` 后跟相同的字节，叶子哈希为 `6ro7vTcFq+sYDZiiGvZaFetUIoMSLig3DGDrCKK1HFU=`。

同一事件如果带有一个可选字段（此处为 `"user_profile_id": "uprof_01H7Z9W1CEpdPaGUP2rbg1XD"`）提供，则其版本 `0x02` JSON 中有 12 个键。该可选键与其他键一起排序，位于 `type` 和 `workspace_id` 之间：

```json
{"accessed_at":"2025-07-08T18:39:58Z","accessor_department":"Trust & Safety","actor":{"email_address":null,"type":"anthropic_actor"},"created_at":"2025-07-08T18:40:00Z","id":"activity_01GPXmAhizavrUuoXNn3tzeA","organization_id":"org_015gtSHLz269eTwgrH8NX5yk","organization_uuid":"25f6429a-3293-49bf-afed-cb312911554b","reason_code":"safety_review","resource_details":{"id":"msg_01HXAMPLE12345678","parent":null,"type":"message"},"type":"anthropic_access","user_profile_id":"uprof_01H7Z9W1CEpdPaGUP2rbg1XD","workspace_id":"wrkspc_01PaGUP2rbg1XDh7Z9W1CEpd"}
```

其版本 `0x02` 条目（唯一需要重建的条目）是字节 `0x02` 后跟这些 UTF-8 字节，叶子哈希为 `onAlpkDfdXmHPvbffiTpHAgBacf/mJCzTfNDibDrYp4=`。请将这三个哈希用作您自己的规范化代码的测试向量。

## 验证您的日志

验证在您自己的基础设施上运行。API 返回的所有内容在针对您自己持有的两样东西验证通过之前，都是不可信的。第一样是您从组织 UUID 推导出的 origin。第二样是您在上次运行时保存的检查点。完整的验证运行按顺序执行以下操作：

1. 获取[验证者密钥集](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-the-verifier-keys)。自行计算每个密钥的指纹，即其 base64 解码后的 `public_key` 的 SHA-256。仅保留指纹出现在[已发布的密钥指纹](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#published-key-fingerprints)中的密钥，以及每个密钥在该表中的状态。仅在该表列为当日当前密钥的密钥下验证新获取的检查点。仅对您在某个已退役密钥退役日期之前保存的检查点接受该密钥。
2. 确定最新检查点。首次运行时，从[检查点端点](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-the-latest-checkpoint)获取。之后每次运行时，从您保存的树大小请求[一致性证明](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#fetch-a-consistency-proof)。响应会连同证明一起带有最新检查点。
3. 首先检查检查点的 origin 行。将其第一行与您推导出的 origin 逐字节比较。在执行任何其他操作之前，拒绝 origin 不同的任何检查点。
4. 验证检查点的签名。找到以您的 origin 命名、且解码后前四个字节等于您保留的某个当前有效密钥的 `key_hash` 的签名行。使用该密钥的 `public_key`，将其余字节作为对注释正文 SHA-256 的 ECDSA P-256 签名进行验证。如果没有签名行与此类密钥匹配，或签名验证失败，则拒绝该检查点。
5. 证明仅追加。如果新的树大小小于您保存的树大小，则失败。如果相等，根哈希必须匹配。如果更大，则验证从您保存的大小和根哈希到新大小和根哈希的 RFC 9162 一致性证明。
6. 证明每个事件都已包含。读取活动源提供的每个访问透明度事件。对于每个新事件，重建[需要为其重建的每个叶子](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#how-an-event-becomes-a-leaf)，分别计算哈希，并获取其[包含证明](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#fetch-an-inclusion-proof)。从事件索引处的每个叶子哈希沿审计路径向上计算到检查点的根哈希。如果其中之一重现了根哈希，该事件即通过验证。如果都没有，则向您提供的事件并非日志所提交的事件。证明响应可能带有与您持有的不同的检查点。在针对它验证任何内容之前，请检查其 origin 行和签名（第 3 步和第 4 步），并通过一致性证明将其链接到您的历史中（第 5 步）。
7. 重新检查您之前看到的内容。每当您再次读取某个事件时，它必须以与您验证时相同的索引和相同的叶子字节提供。任何事件都不得丢失其原有的索引，并且在您处于注册状态期间，任何事件都不得新出现而不带索引。首次运行时在没有索引的情况下提供的事件是您的日志前历史。只重新读取近期事件的运行只会重新检查这些事件。要重新检查较早的事件，请再次验证您保存的副本（第 8 步）。
8. 保存您验证过的检查点以及您验证过的每个事件的记录。它们是您的证据，也是下次运行的起点。

### 为新的叶子版本升级

每个 `axt-verify` 版本只会重建其发布时已定义的[叶子版本](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#how-an-event-becomes-a-leaf)。一旦能够重建某个新版本的发布版已经发布，就应假定您的日志随时可能开始写入该新版本。请在该发布版发布后尽快安装。

| 叶子版本   | 能够重建它的 `axt-verify` 发布版 |
| ------ | ----------------------- |
| `0x01` | 所有发布版                   |
| `0x02` | `v0.2.0` 及更高版本          |

`axt-verify version` 会打印您正在运行的发布版。新安装会获得最新的发布版。如果表格最后一行不包含您的发布版，请升级到该行中的某个发布版。如果该行中的发布版尚未发布，请在其发布后尽快升级：

1. 按照您之前的安装方式安装最新的发布版（即[使用 axt-verify 进行验证](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#verify-with-axt-verify)中的 `go install` 命令行）。
2. 像以前一样继续运行它：相同的 `--org`、相同的 `--state` 文件、相同的计划。

预期情况：

* **在您的日志开始写入新版本之前。** 较新的发布版会像以前一样验证您的日志，因为它也能重建较早的叶子版本。
* **在您的日志开始写入新版本之后，使用的是不能重建该版本的发布版。** 该发布版会将叶子属于该版本的每个事件报告为验证失败，退出状态为 `1`，即使该事件不包含该版本的任何可选字段。您的发布版能够重建的版本的叶子不受影响，它们会像以前一样通过验证。一旦您在同一个 `--state` 文件上运行能够重建该版本的发布版，这些失败就会消除，仅追加检查会从您上次运行结束的地方继续。
* **如果您回退到较早的发布版。** 不要将较新发布版保存的状态文件交给较早的发布版。较早的发布版会将每个正在等待检查点的事件报告为验证失败。如果您可能需要回退到较早的发布版，请在首次运行较新发布版之前复制状态文件，并将该副本交给较早的发布版使用。

如果您自行维护验证者，则需要在您的日志开始写入新版本之前，从[事件如何成为叶子](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#how-an-event-becomes-a-leaf)中获取该版本的字段，以及关于应重建哪些条目的规则。

### 使用 axt-verify 进行验证

如果您已经按计划运行 `axt-verify`，请先阅读[为新的叶子版本升级](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#upgrade-for-a-new-leaf-version)。

[`axt-verify`](https://github.com/anthropics/axt-verify) 是 Anthropic 为此日志提供的开源验证者。它是一个单独的 Go 二进制文件，您可以在自己的基础设施上运行。每个发布版都内置了[已发布的密钥指纹](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#published-key-fingerprints)中的恰好一个日志签名密钥，因此它从不会向 API 询问应信任哪个密钥。当 Anthropic 轮换密钥时，您需要在切换日期升级到携带新密钥的发布版。`axt-verify` 根据您通过 `--org` 传入的组织 UUID 推导出您的 origin，该 UUID 就是您在[开始之前](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#before-you-begin)中从 Console 获取的那个。它会拒绝任何 origin 行不一致的检查点。

使用 Go 1.26 或更高版本安装它。它从 `ANTHROPIC_COMPLIANCE_ACCESS_KEY` 环境变量读取您的 Compliance Access Key，从不从标志或文件读取。`run` 命令是需要定期调度的命令：

```bash
go install github.com/anthropics/axt-verify/cmd/axt-verify@latest

export ANTHROPIC_COMPLIANCE_ACCESS_KEY="<your Compliance Access Key>"
axt-verify --org 25f6429a-3293-49bf-afed-cb312911554b \
  --state /var/lib/axt-verify/25f6429a-3293-49bf-afed-cb312911554b.state \
  run
```

每次 `run` 都会获取最新的检查点，并验证其签名和 origin 行。然后它会证明日志是上一次运行所保存检查点的仅追加扩展。接下来，它会分页读取您活动源上的访问透明度事件。它从上一次运行读取的最新事件之前 7 天（按 `created_at` 计）开始读取，以便仍能捕获延迟列出或乱序的事件。它会重建每个[事件需要重建的叶子](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#how-an-event-becomes-a-leaf)，针对这些叶子验证包含证明，并将之前已验证过的任何事件与当时记录的内容进行比较。最后，它会将新的检查点及其进度保存到 `--state` 文件中，下一次运行将从这里开始。请至少每天运行一次，每小时运行一次也是合理的。使用父组织密钥时，请为每个子组织运行一个副本，每个副本都有自己的 `--org` 和 `--state` 文件。每个子组织的 UUID 也应来自 Claude Console，而不是来自您正在验证的 Compliance API 响应。要找到它，请在 Console 侧边栏底部的账户菜单中切换到该子组织，然后打开[组织](https://platform.claude.com/settings/organization)页面（`platform.claude.com/settings/organization`）。您也可以在 Console 中父组织的组织列表中找到它。`axt-verify checkpoint` 只执行检查点和仅追加这两个步骤。`axt-verify events FILE` 会验证您已持有的事件，例如审计员的样本或您自己的导出内容。它会证明文件中的每个事件在当前检查点下仍提交在日志中。它不会读取活动源，也不会改动状态文件。

这个时间窗口带来了一个限制：`run` 只会重新读取最近 7 天的事件，因此它重新检查的是最近提供的事件（第 7 步），而不是您的全部历史。请保留您导出的事件（请参阅[保留您自己的检查点存档](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#keep-your-own-checkpoint-archive)）。`axt-verify events FILE` 可以在之后的任何日期证明这些副本仍提交在日志中，但它不会重新读取活动源。要检测从活动源中删除或在活动源上被改写的较早事件，请从活动源重新导出该时间范围，并将其与您保留的副本进行比较。您也可以使用 `axt-verify events FILE` 验证重新导出的内容本身。7 天的重叠期长于活动源的 2 个工作日交付时间，因此延迟到达的事件仍会落在之后某次运行的时间窗口内。如有需要，可以使用 `--overlap` 更改该时长。

如果您需要自己的实现，请使用支持 ECDSA 注释密钥的 tlog-tiles 库，按照上述检查清单进行操作。

### 解读结果

`axt-verify` 会打印您的 origin、它所验证检查点的树大小和根哈希、仅追加检查开始时的树大小，以及按结果分类的事件计数。传入 `--json` 可获得相同的报告，以每行一个 JSON 对象的形式输出。每个事件都有以下四种结果之一：

* **Verified（已验证）：** 根据所提供事件重建的叶子已提交在已签名日志中该事件的索引位置。
* **Pending（待定）：** 事件的索引超出了最新发布的检查点。在事件出现后的短时间内，这是正常现象。在 `run` 中，`axt-verify` 会记录该事件，在之后某次运行中一旦有检查点覆盖它就进行验证，如果这一过程超过 24 小时则判定为失败。`events FILE` 没有后续运行来解决此问题，因此它会最多等待一分钟，等待覆盖该事件的检查点发布。如果没有检查点到达，它会将该事件报告为尚未覆盖，并以退出状态 `3` 结束。请稍后再次运行。如果 `events FILE` 在相隔至少一天的两次运行中都将同一事件报告为尚未覆盖，请将其视为验证失败，并按照退出状态 `1` 的方式上报。
* **Not logged（未记录）：** 该事件在提供时没有索引。只有在您的组织未注册访问透明度期间，或者事件记录于您组织的日志创建之前时，事件才会在没有 `transparency_log_leaf_index` 的情况下提供（请参阅[活动源事件上的 `transparency_log_leaf_index` 字段](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#the-transparency-log-leaf-index-field-on-activity-feed-events)）。`axt-verify` 会将此类事件报告为未记录，并且不会因此使运行失败。如果一个无索引事件的日期晚于您的日志创建时间，且处于您已注册的期间内，则这种情况不应出现。请查看运行摘要或 `--json` 输出中的未记录列表，而不要仅依赖退出状态。
* **Failed（失败）：** 请参阅退出状态 `1`。

退出状态会告诉您的调度程序发生了什么：

* **`0`：** 没有任何失败。未记录的事件只会被报告而不会判定为失败，`run` 中的待定事件也是如此。

* **`1`：** 验证失败。如果每个新事件都因其包含证明无法重现已签名的根哈希而失败，请首先排除所用发布版无法重建您日志最新叶子版本的可能性，具体如本列表中相应条目所述。如果不是这个原因，以及对于任何其他失败，请将其视为安全发现，而不是暂时性错误。请保留状态文件和输出，并将其报告给您的 Anthropic 客户代表或 [Anthropic 支持](https://support.claude.com)。原因包括：

  * 检查点的 origin 错误，或者签名无法使用您的 `axt-verify` 版本中内置的密钥验证。请将失败检查点签名行上的 `key_hash` 与[已发布的密钥指纹](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#published-key-fingerprints)进行比较。如果某个密钥列在其中，且其切换日期之后您尚未升级，则说明您需要对应的版本。如果某个密钥未列在其中，则无论您运行的是哪个版本，这都是一项安全发现。出现此失败时，`axt-verify` 会打印所提供检查点上每个签名的密钥哈希以及它所信任密钥的密钥哈希，每个均为八位十六进制数字。这些输出足以进行比较。
  * 检查点不是格式正确的签名注释，例如其根哈希不是 32 字节。
  * 日志缩小了，或者无法证明它扩展了您保存的检查点。此时输出会包含两个检查点以及证明，因此证据本身即可成立。
  * 同一树大小存在两个根哈希不同的签名检查点。输出会包含这两个检查点。
  * 通过 `--from` 传入的检查点文件的签名无法使用您的 `axt-verify` 版本中内置的密钥验证，或者 `--from` 或 `--from-trusted` 文件的 origin 不属于您。对于在密钥轮换之前签名的存档，请参阅[保留您自己的检查点存档](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#keep-your-own-checkpoint-archive)。
  * 包含证明无法为提供给您的事件重现已签名的根哈希。无法重建某个叶子版本的发布版会为叶子属于该版本的每个事件报告此问题。如果每个新事件同时以这种方式失败，请首先检查您运行的是否为最新的 `axt-verify` 发布版（请参阅[为新的叶子版本升级](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#upgrade-for-a-new-leaf-version)）。如果不是，请复制状态文件，保留该副本和输出，然后在原始 `--state` 文件上运行最新的发布版。一旦您运行能够重建该版本的发布版，这些失败就会消除。如果您运行的已经是最新发布版，或者升级后失败仍然存在，则这是一项安全发现。
  * 运行窗口内的某个事件的提供方式与之前某次运行所记录的不同：叶子字节不同、索引不同，或者原本有索引而现在没有。
  * 某个事件在运行首次看到它 24 小时后仍处于待定状态。
  * 对于活动源提供给您的事件，包含证明请求被拒绝（`400`、`401` 或 `403`）。
  * 返回的包含证明对应的 `leaf_index` 与请求的不同。
  * 在 `events FILE` 中，某个事件的索引在检查开始时已被最新检查点覆盖，但在等待结束前未提供其包含证明。
  * 某个事件的 `organization_uuid` 不是您通过 `--org` 传入的组织 UUID。当父组织对涵盖多个子组织的导出运行 `events FILE` 时，其他每个组织的事件都会以这种方式失败，因此请先按组织拆分导出，并使用各自的 `--org` 验证每个部分。
  * 在一次运行或一个 `events FILE` 输入中，同一活动 `id` 出现在两个不同的索引处，或在同一索引处出现两次但内容不同。
  * 某个事件的叶子无法重建：例如，某个文档所列的字段不是字符串、某个字段名出现了两次，或者 `type` 缺失或是访问透明度类型的无法识别的变体。`events FILE` 会跳过其他活动类型的行，且不会将其判定为失败。

* **`2`：** 用法或配置错误。原因包括：

  * 标志缺失或格式错误。
  * 没有 `ANTHROPIC_COMPLIANCE_ACCESS_KEY`。
  * 在任何检查点验证通过之前，密钥被 API 拒绝（`401` 或 `403`）。
  * 状态文件无法读取或属于另一个 origin。

* **`3`：** 运行无法完成。在 `run` 中，已验证的内容会保存到状态文件中。请再次运行。原因包括：

  * `events FILE` 中某个事件的索引在等待期间内未被任何已发布的检查点覆盖。对于此类事件，**Pending** 结果说明了何时应停止重新运行并上报。
  * 网络错误、速率限制或服务器错误持续时间超过了重试次数。
  * 意外的响应。
  * 状态文件或 `--save` 文件无法写入。

  在 Anthropic 为您的组织记录第一个访问透明度事件之前，出现带有 `404` 响应的退出状态 `3` 属于预期情况，因为在该事件创建日志之前，每个透明度日志端点都会返回 `404`。如果在您的活动源首次显示访问透明度事件几天后，透明度日志端点仍然返回 `404`，请上报，无论该事件是否带有 `transparency_log_leaf_index`。这种组合不在预期之内。一旦某次运行成功，如果退出状态 `3` 持续出现，请上报。

### 保留您自己的检查点存档

您能持有的最有力证据，是您自己对日志在某一天所述内容的记录。`axt-verify --save FILE` 会逐字写入一次运行所验证的检查点，而在后续运行中使用 `--from FILE` 会让日志证明它仍然扩展了该检查点。请将保存的检查点存档到您控制的存储中，例如每天一次。数月之后，从该存档检查点的树大小出发的[一致性证明](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#fetch-a-consistency-proof)仍必须能通向日志所提供的任何检查点，否则验证失败。在计划的轮换之后，早期密钥仍会列在[验证者密钥集](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-the-verifier-keys)中，因此存档的检查点可以继续通过验证。使用 `axt-verify` 时，如果签署某个存档检查点的密钥此后已被轮换掉，请使用 `--from-trusted` 而不是 `--from` 传入该存档。也请保留事件。您从活动源导出的访问透明度事件是 `axt-verify events FILE` 的有效输入，它可以在之后的任何日期证明这些副本仍在日志当前检查点下被提交。由于 `events FILE` 不会重新读取活动源，您保留的导出也可以作为参照，用于与之后对同一范围的重新导出进行比较。

## 常见问题

<AccordionGroup>
  <Accordion title="我必须验证透明度日志吗？">
    不必。无论是否有人验证，Anthropic 都会为您的组织维护该日志。验证是您亲自检查日志的方式。审计人员只要拥有具有活动源权限范围的 API 密钥，就可以使用相同的步骤验证您交给他们的事件样本。
  </Accordion>

  <Accordion title="为什么我刚收到的事件的包含证明返回了 404？">
    该事件是在覆盖其位置的检查点发布前片刻提供的。覆盖它的检查点很快就会发布，因此请稍等片刻后重试。如果一天后某个事件的证明仍不可用，这才是需要上报的异常。
  </Accordion>

  <Accordion title="Anthropic 轮换签名密钥时会发生什么？">
    计划的轮换会至少提前 30 天在[已发布的密钥指纹](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#published-key-fingerprints)中公布，并附有切换日期。在切换日期，Anthropic 开始使用新密钥签名，并发布包含该密钥的 `axt-verify` 版本。在同一日期，Anthropic 会使用新密钥重新签发每个组织的最新检查点，即使日志没有增长也是如此。每个 `axt-verify` 版本只包含一个密钥，因此请在切换日期升级。在切换后运行旧版本会以退出状态 `1` 失败，在切换前运行新版本也是如此。只要运行对应的版本，任一失败都会消除。您在旧密钥下保存的检查点仍然是有效的起点，因为从已保存检查点到新检查点的仅追加证明并不取决于旧检查点由哪个密钥签名。新密钥会出现在[验证者密钥集](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#read-the-verifier-keys)的最前面，早期密钥仍会保留在列表中。因此，您已验证或存档的检查点可以继续针对该密钥集通过验证。如果您自己的验证者固定了已发布的指纹，则需要在切换日期之前添加新指纹及其切换日期。如果验证者在本地保存密钥集，则当它遇到其未持有的密钥哈希的签名时，会重新获取密钥集。一个检查点会对整个历史进行承诺。一旦某个由新密钥签名的检查点通过验证，并且从您保存的检查点出发的一致性证明能通向它，所有早期条目也会随之重新确立。
  </Accordion>

  <Accordion title="什么是叶子版本？我需要跟踪某个叶子属于哪个版本吗？">
    叶子版本是叶子条目的第一个字节。它表明该叶子承诺的是哪一组字段（请参阅[事件如何成为叶子](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#how-an-event-becomes-a-leaf)）。您不需要跟踪它。您的日志可以包含多个版本的叶子，验证者会为事件重建该部分最后一条规则所指定的条目，只要事件的包含证明与其中一个匹配就接受该事件。您真正需要的是一个能够重建您的日志所写入最新版本的 `axt-verify` 发布版（请参阅[为新的叶子版本升级](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency-log#upgrade-for-a-new-leaf-version)）。
  </Accordion>

  <Accordion title="我可以自己计算证明，而不调用证明端点吗？">
    可以。哈希瓦片是主要接口，支持 ECDSA 注释密钥的 tlog-tiles 客户端可以根据它们计算包含证明和一致性证明。证明端点只是为了方便。通用客户端需要一个轻量封装来发送 `x-api-key` 标头，对于父组织密钥，还需发送 `organization_id` 参数。
  </Accordion>

  <Accordion title="如果我的组织停止使用访问透明度会怎样？">
    不会删除任何内容。您的日志仍可通过相同的端点读取和验证，因此请像以前一样继续验证。如果您的组织之后再次启用访问透明度，将继续使用同一个日志，并且一致性证明会跨越这段间隔。
  </Accordion>

  <Accordion title="我可以用一个密钥验证所有子组织吗？">
    可以。父组织密钥可以通过传入 `organization_id` 读取任何已加入的子组织的日志。每个子组织都有自己的日志、origin 和已保存的检查点。请为每个子组织分别运行单独的验证，并使用各自的状态。
  </Accordion>
</AccordionGroup>

## 相关资源

* [访问透明度](https://platform.claude.com/docs/zh-CN/manage-claude/access-transparency)
* [查询活动源](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-activity-feed)
* [Compliance API 概述](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-api)
* [处理 Compliance API 错误](https://platform.claude.com/docs/zh-CN/manage-claude/compliance-errors)
* [axt-verify](https://github.com/anthropics/axt-verify)，Anthropic 用于透明度日志的开源验证者
* [C2SP tlog-tiles](https://c2sp.org/tlog-tiles) 和 [C2SP signed note](https://c2sp.org/signed-note)，瓦片、条目包和检查点的传输格式
* [RFC 9162](https://www.rfc-editor.org/rfc/rfc9162)，Merkle 树、包含证明和一致性证明算法
* [RFC 8785](https://www.rfc-editor.org/rfc/rfc8785)，用于叶子的 JSON 规范化方案
