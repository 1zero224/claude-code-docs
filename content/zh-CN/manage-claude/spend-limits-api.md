---
title: 支出限额 API
url: https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api
description: 为每位 Claude Enterprise 成员设置支出限额，查看每位成员的支出限额继承自何处，并审核或处理成员提出的提高限额请求。
---

Spend Limits API（支出限额 API）让您可以为每位 Claude Enterprise 成员设置支出限额，查看每位成员的支出限额继承自何处，并审核或处理成员提出的提高限额请求。

如需按用户和按时间分桶的用量与成本*报告*，请参阅[分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/analytics-api) 或 [用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api)。

<Check>
  **需要具有相应作用域的 Admin API 密钥**

  这些端点需要具有 `read:spend_limits` 权限范围（用于 `GET` 端点）或 `write:spend_limits` 权限范围（用于 `POST` 和 `DELETE` 端点）的 Admin API 密钥。请参阅[创建 Admin API 密钥](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#create-a-key-for-a-claude-enterprise-organization)，了解您的主要所有者在何处创建密钥以及应选择哪些权限范围。请在每个请求的 `x-api-key` 标头中传递该密钥，并同时传递 [`anthropic-version`](https://platform.claude.com/docs/zh-CN/api/versioning) 标头。
</Check>

<Note>
  支出限额 API 仅适用于 Claude Enterprise 组织。它不适用于 Claude Platform（Claude Console）组织。
</Note>

## 概述

该 API 在两种资源上公开了八个端点：

| 资源           | 端点                                                                                                                                                                                                                                                                   | 用途                                       |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| **支出限额**     | `GET /v1/organizations/spend_limits/effective`<br />`GET /v1/organizations/spend_limits/{spend_limit_id}`<br />`POST /v1/organizations/spend_limits`<br />`DELETE /v1/organizations/spend_limits/{spend_limit_id}`                                                   | 读取每位成员的有效支出限额和本期至今支出；设置或清除按用户的覆盖值。       |
| **支出限额提高请求** | `GET /v1/organizations/spend_limit_increase_requests`<br />`GET /v1/organizations/spend_limit_increase_requests/{id}`<br />`POST /v1/organizations/spend_limit_increase_requests/{id}/approve`<br />`POST /v1/organizations/spend_limit_increase_requests/{id}/deny` | 列出成员提出的提高支出限额请求，并附带做出决定所需的上下文；批准或拒绝每个请求。 |

使用**支出限额**端点来回答"每位成员适用什么支出限额、它来自哪里、以及他们距离限额还有多远？"这类问题，并设置按用户的覆盖值。使用**支出限额提高请求**端点来处理成员提交的请求队列。

## 前提条件

* 您的组织必须使用 Claude Enterprise 计划。
* 您的组织必须已开启用量额度（usage credits）。您的主要所有者可以在 claude.ai 账单设置中开启。

SDK 在 `client.beta.organization.spend_limits` (typescript: `client.beta.organization.spendLimits`; csharp, go: `client.Beta.Organization.SpendLimits`; java: `client.beta().organization().spendLimits()`; php: `$client->beta->organization->spendLimits`) 下提供这些端点，`ant` CLI 则在 `ant beta:organization:spend-limits` 下提供。两者都会为您发送 `anthropic-version` 标头。示例从 `ANTHROPIC_API_KEY` 环境变量中读取 Admin API 密钥。在您遍历结果时，SDK 和 CLI 会自动为您获取更多页面，因此 `limit` 设置的是每页大小，而不是总数（请参阅[分页](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api#pagination)）。

## 快速开始

列出每位成员的有效月度支出限额和本期至今支出：

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/spend_limits/effective?limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:spend-limits:effective list --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  effective_limits = client.beta.organization.spend_limits.effective.list(limit=20)

  for effective_limit in effective_limits:
      scope = effective_limit.scope
      if scope.type == "user":
          amount = effective_limit.amount or "unlimited"
          print(f"{scope.user_id}: {effective_limit.period_to_date_spend} of {amount}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const effectiveLimits = await client.beta.organization.spendLimits.effective.list({
    limit: 20
  });

  for await (const effectiveLimit of effectiveLimits) {
    const { scope } = effectiveLimit;
    if (scope.type === "user") {
      const amount = effectiveLimit.amount ?? "unlimited";
      console.log(`${scope.user_id}: ${effectiveLimit.period_to_date_spend} of ${amount}`);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.SpendLimits.Effective.List(
      new() { Limit = 20 }
  );

  await foreach (var effectiveLimit in page.Paginate())
  {
      if (effectiveLimit.Scope.TryPickBetaSpendLimitUser(out var userScope))
      {
          var amount = effectiveLimit.Amount ?? "unlimited";
          Console.WriteLine($"{userScope.UserID}: {effectiveLimit.PeriodToDateSpend} of {amount}");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  effectiveLimits := client.Beta.Organization.SpendLimits.Effective.ListAutoPaging(context.Background(), anthropic.BetaOrganizationSpendLimitEffectiveListParams{
  	Limit: anthropic.Int(20),
  })

  for effectiveLimits.Next() {
  	effectiveLimit := effectiveLimits.Current()
  	if userScope, ok := effectiveLimit.Scope.AsAny().(anthropic.BetaSpendLimitUserScope); ok {
  		amount := effectiveLimit.Amount
  		if !effectiveLimit.JSON.Amount.Valid() {
  			amount = "unlimited"
  		}
  		fmt.Printf("%s: %s of %s\n", userScope.UserID, effectiveLimit.PeriodToDateSpend, amount)
  	}
  }
  if err := effectiveLimits.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.spendlimits.effective.EffectiveListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = EffectiveListParams.builder()
          .limit(20)
          .build();
      var effectiveLimits = client.beta().organization().spendLimits().effective().list(params);

      for (var effectiveLimit : effectiveLimits.autoPager()) {
          var amount = effectiveLimit.amount().orElse("unlimited");
          effectiveLimit.scope().user().ifPresent(userScope ->
              IO.println(userScope.userId() + ": " + effectiveLimit.periodToDateSpend() + " of " + amount)
          );
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Organization\SpendLimits\SpendLimitUserScope;
  // ...

  $client = new Client();

  $effectiveLimits = $client->beta->organization->spendLimits->effective->list(limit: 20);

  foreach ($effectiveLimits->pagingEachItem() as $effectiveLimit) {
      $scope = $effectiveLimit->scope;
      if ($scope instanceof SpendLimitUserScope) {
          $amount = $effectiveLimit->amount ?? 'unlimited';
          echo "{$scope->userID}: {$effectiveLimit->periodToDateSpend} of {$amount}\n";
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.spend_limits.effective.list(limit: 20)

  page.auto_paging_each do |effective_limit|
    scope = effective_limit.scope
    if scope.is_a?(Anthropic::Models::Beta::Organization::BetaSpendLimitUserScope)
      amount = effective_limit.amount || "unlimited"
      puts "#{scope.user_id}: #{effective_limit.period_to_date_spend} of #{amount}"
    end
  end
  ```
</CodeGroup>

## 关键概念

### 支出限额层级

每位成员的支出都适用一个 **effective spend limit**（有效支出限额），它从一个由多个作用域级别组成的层级中解析得出。当成员没有按用户的覆盖值时，他们会继承为其所在组（如果您的组织使用基于组的限额）、其席位层级或组织范围默认值所配置的支出限额。组支出限额是按成员的默认值：每位继承该限额的成员都以其自身的支出为准进行限制，而不是共享的组预算。

读取 `GET /v1/organizations/spend_limits/effective` 会返回每位当前成员及其解析后的有效支出限额、该限额的解析来源（`source`）以及其本期至今支出。使用 `POST /v1/organizations/spend_limits` 设置按用户的覆盖值会将成员固定在特定的支出限额上，而不管他们原本会继承什么。删除覆盖值会使他们恢复到继承的支出限额（如果不存在继承限额，则保持无限制）。

每位成员所在行的 `source` 字段告诉您其支出限额是从哪个级别解析出来的：`user`（按用户的覆盖值）、`seat_tier`、`rbac_group` 或 `organization`。请将作用域类型视为开放集合；遇到未知值时应跳过处理而不是报错。

### 周期

`period`（周期）是执行支出限额并重置支出的循环窗口。支出限额由其 `(scope, period)` 对来标识。目前 `monthly` 是唯一支持的周期；月度支出在每个日历月第一天的 00:00 UTC 重置。请将 `period` 视为开放集合。

### 金额与货币

所有金额值均为字符串，以**组织计费货币的最小单位**表示（对于 USD 而言为美分）。例如，`"50000"` 表示 $500.00 USD。请将其解析为十进制数并除以 100 以显示美元金额；对于较大的值，请避免使用二进制浮点数。

`amount` **可为空**。在成员的有效行中，`null` 表示**无限制**（没有支出限额），而 `"0"` 表示该成员不能在其计划所含用量之外使用 Claude。在已配置的支出限额行上（如 `GET /v1/organizations/spend_limits/{id}` 返回的），`null` 仅表示未设置数值型支出限额；请读取成员的有效行以区分无限制与仅限所含用量。

`period_to_date_spend` 是该成员自当前 `period` 开始以来累计的支出，采用相同的最小单位格式；它可能包含小数部分（例如 `"41280.125"`）。如果支出读数暂时不可用，它可能显示为 `"0"`；请将其视为参考信息，而非交易数据。

### 提高请求的生命周期

当成员在 claude.ai 中点击 **Request more usage**（请求更多用量）时，会创建一个 **spend limit increase request**（支出限额提高请求）。请求不会通过此 API 创建。请求的 `status` 为以下之一：

| 状态         | 含义                                                                                                   |
| ---------- | ---------------------------------------------------------------------------------------------------- |
| `pending`  | 等待管理员处理。该请求通常附带实时的 `spend_summary`，以便您在做决定时查看该成员当前生效的支出限额和本期至今的支出；如果无法计算，`spend_summary` 可能为 `null`。 |
| `approved` | 请求已以批准方式解决：要么管理员明确批准了它，要么另一项管理员操作提高了该成员的支出限额，要么 Anthropic 支持团队代表组织提高了支出限额。`spend_summary` 为 `null`。  |
| `denied`   | 管理员已拒绝。`spend_summary` 为 `null`。成员可以立即发送新请求；只有 `pending` 状态的请求才会阻止新请求。管理员仍可随时直接提高该成员的支出限额。           |

`approved` 和 `denied` 均为终态。一位成员同一时间最多只有一个 `pending` 请求。

使用 `POST /v1/organizations/spend_limit_increase_requests/{id}/approve` 批准时，会写入与 `POST /v1/organizations/spend_limits` 所写入的相同的按用户支出限额行。直接设置支出限额**不会**转换待处理请求的状态；请使用批准端点来解决请求。

默认情况下，当成员的请求被批准或拒绝时，Anthropic 会向该成员发送电子邮件。在批准或拒绝时传递 `suppress_notification: true` 可抑制该邮件（例如，当您自己的系统会通知成员时）。

## 版本控制

请在每个请求中发送 `anthropic-version` 标头；有关可用版本，请参阅 [API 版本](https://platform.claude.com/docs/zh-CN/api/versioning)。

## 速率限制

所有八个端点共享一个按组织计算的速率限制，即**每分钟 60 个请求**。超出限制的请求将返回 **429 Too Many Requests**。

## 分页

`GET /v1/organizations/spend_limits/effective` 和 `GET /v1/organizations/spend_limit_increase_requests` 使用**不透明游标**进行分页。第一个请求返回最多 `limit` 行以及一个 `next_page` 游标；在下一个请求中将该游标原样作为 `page` 参数传递，并重复此过程直到 `next_page` 为 `null`。

**不要在序列中途更改查询参数。**&#x6E38;标与签发它们的筛选条件绑定。如果您更改了 `user_ids[]`、`period[]`、`status[]` 或 `actor_ids[]` 并传递旧游标，您将收到 400 错误，提示 *“cursor does not match current query parameters”*。请改为从第一页开始新的序列。

## 序列化列表参数

列表参数使用方括号表示法：为每个值重复带 `[]` 的参数名。

```text wrap
user_ids[]=user_01AbCdEfGh&user_ids[]=user_01JkLmNoPq
```

## 错误响应

错误响应遵循 [Claude API 错误](https://platform.claude.com/docs/zh-CN/api/errors)中记录的标准格式。联系支持团队时，请提供响应正文中的 `request_id`。

## 支出限额

### 列出每位成员的有效支出限额

`GET /v1/organizations/spend_limits/effective` 为每位当前成员返回一行，反映每位成员的有效支出限额、其在作用域层级中的 `source` 以及其 `period_to_date_spend`。需要 `read:spend_limits` 作用域。

有关完整的参数详情和响应模式，请参阅 API 参考中的[列出有效支出限额](https://platform.claude.com/docs/zh-CN/api/beta/organization/spend_limits/effective/list)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/spend_limits/effective?limit=20" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:spend-limits:effective list --limit 20
  ```

  ```python Python
  client = anthropic.Anthropic()

  effective_limits = client.beta.organization.spend_limits.effective.list(limit=20)

  for effective_limit in effective_limits:
      scope = effective_limit.scope
      if scope.type == "user":
          amount = effective_limit.amount or "unlimited"
          print(f"{scope.user_id}: {effective_limit.period_to_date_spend} of {amount}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const effectiveLimits = await client.beta.organization.spendLimits.effective.list({
    limit: 20
  });

  for await (const effectiveLimit of effectiveLimits) {
    const { scope } = effectiveLimit;
    if (scope.type === "user") {
      const amount = effectiveLimit.amount ?? "unlimited";
      console.log(`${scope.user_id}: ${effectiveLimit.period_to_date_spend} of ${amount}`);
    }
  }
  ```

  ```csharp C#
  AnthropicClient client = new();

  var page = await client.Beta.Organization.SpendLimits.Effective.List(
      new() { Limit = 20 }
  );

  await foreach (var effectiveLimit in page.Paginate())
  {
      if (effectiveLimit.Scope.TryPickBetaSpendLimitUser(out var userScope))
      {
          var amount = effectiveLimit.Amount ?? "unlimited";
          Console.WriteLine($"{userScope.UserID}: {effectiveLimit.PeriodToDateSpend} of {amount}");
      }
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  effectiveLimits := client.Beta.Organization.SpendLimits.Effective.ListAutoPaging(context.Background(), anthropic.BetaOrganizationSpendLimitEffectiveListParams{
  	Limit: anthropic.Int(20),
  })

  for effectiveLimits.Next() {
  	effectiveLimit := effectiveLimits.Current()
  	if userScope, ok := effectiveLimit.Scope.AsAny().(anthropic.BetaSpendLimitUserScope); ok {
  		amount := effectiveLimit.Amount
  		if !effectiveLimit.JSON.Amount.Valid() {
  			amount = "unlimited"
  		}
  		fmt.Printf("%s: %s of %s\n", userScope.UserID, effectiveLimit.PeriodToDateSpend, amount)
  	}
  }
  if err := effectiveLimits.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.spendlimits.effective.EffectiveListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = EffectiveListParams.builder()
          .limit(20)
          .build();
      var effectiveLimits = client.beta().organization().spendLimits().effective().list(params);

      for (var effectiveLimit : effectiveLimits.autoPager()) {
          var amount = effectiveLimit.amount().orElse("unlimited");
          effectiveLimit.scope().user().ifPresent(userScope ->
              IO.println(userScope.userId() + ": " + effectiveLimit.periodToDateSpend() + " of " + amount)
          );
      }
  }
  ```

  ```php PHP
  use Anthropic\Beta\Organization\SpendLimits\SpendLimitUserScope;
  // ...

  $client = new Client();

  $effectiveLimits = $client->beta->organization->spendLimits->effective->list(limit: 20);

  foreach ($effectiveLimits->pagingEachItem() as $effectiveLimit) {
      $scope = $effectiveLimit->scope;
      if ($scope instanceof SpendLimitUserScope) {
          $amount = $effectiveLimit->amount ?? 'unlimited';
          echo "{$scope->userID}: {$effectiveLimit->periodToDateSpend} of {$amount}\n";
      }
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.spend_limits.effective.list(limit: 20)

  page.auto_paging_each do |effective_limit|
    scope = effective_limit.scope
    if scope.is_a?(Anthropic::Models::Beta::Organization::BetaSpendLimitUserScope)
      amount = effective_limit.amount || "unlimited"
      puts "#{scope.user_id}: #{effective_limit.period_to_date_spend} of #{amount}"
    end
  end
  ```
</CodeGroup>

```json
{
  "data": [
    {
      "scope": { "type": "user", "user_id": "user_01AbCdEfGh" },
      "actor": {
        "type": "user_actor",
        "user_id": "user_01AbCdEfGh",
        "name": "Jane Smith",
        "email_address": "jane@example.com",
        "deleted": false
      },
      "amount": "50000",
      "currency": "USD",
      "period": "monthly",
      "source": { "type": "seat_tier", "seat_tier": "enterprise_standard" },
      "spend_limit_id": "spl_01XyZaBcDeFgHiJkLmNoPq",
      "period_to_date_spend": "31402.5"
    }
  ],
  "next_page": "page_..."
}
```

### 获取单个支出限额

`GET /v1/organizations/spend_limits/{spend_limit_id}` 按 ID 返回一个已配置的支出限额。使用它来检查某个 `spend_limit_id` 字段所引用的行。需要 `read:spend_limits` 作用域。

有关完整的参数详情和响应模式，请参阅 API 参考中的[检索支出限额](https://platform.claude.com/docs/zh-CN/api/beta/organization/spend_limits/retrieve)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/spend_limits/spl_01AbCdEfGhIjKlMnOpQrSt" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:spend-limits retrieve \
    --spend-limit-id spl_01AbCdEfGhIjKlMnOpQrSt
  ```

  ```python Python
  client = anthropic.Anthropic()

  spend_limit = client.beta.organization.spend_limits.retrieve(
      "spl_01AbCdEfGhIjKlMnOpQrSt"
  )

  amount = spend_limit.amount or "none"
  print(f"id: {spend_limit.id}")
  print(f"amount: {amount}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const spendLimit = await client.beta.organization.spendLimits.retrieve(
    "spl_01AbCdEfGhIjKlMnOpQrSt"
  );

  const amount = spendLimit.amount ?? "none";
  console.log(`id: ${spendLimit.id}`);
  console.log(`amount: ${amount}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var spendLimit = await client.Beta.Organization.SpendLimits.Retrieve(
      "spl_01AbCdEfGhIjKlMnOpQrSt"
  );

  var amount = spendLimit.Amount ?? "none";
  Console.WriteLine($"id: {spendLimit.ID}");
  Console.WriteLine($"amount: {amount}");
  ```

  ```go Go
  client := anthropic.NewClient()

  spendLimit, err := client.Beta.Organization.SpendLimits.Get(context.Background(), "spl_01AbCdEfGhIjKlMnOpQrSt")
  if err != nil {
  	log.Fatal(err)
  }

  amount := spendLimit.Amount
  if !spendLimit.JSON.Amount.Valid() {
  	amount = "none"
  }

  fmt.Printf("id: %s\n", spendLimit.ID)
  fmt.Printf("amount: %s\n", amount)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var spendLimit = client.beta().organization().spendLimits()
      .retrieve("spl_01AbCdEfGhIjKlMnOpQrSt");

  var amount = spendLimit.amount().orElse("none");
  IO.println("id: " + spendLimit.id());
  IO.println("amount: " + amount);
  ```

  ```php PHP
  $client = new Client();

  $spendLimit = $client->beta->organization->spendLimits->retrieve('spl_01AbCdEfGhIjKlMnOpQrSt');

  $amount = $spendLimit->amount ?? 'none';
  echo "id: {$spendLimit->id}\n";
  echo "amount: {$amount}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  spend_limit = client.beta.organization.spend_limits.retrieve("spl_01AbCdEfGhIjKlMnOpQrSt")

  amount = spend_limit.amount || "none"
  puts "id: #{spend_limit.id}"
  puts "amount: #{amount}"
  ```
</CodeGroup>

### 设置按用户的覆盖值

`POST /v1/organizations/spend_limits` 设置按用户的支出限额覆盖值。这是一个以 `(scope, period)` 为键的 upsert 操作：为已有限额的用户和周期设置限额会就地覆盖它。此端点仅接受 `scope.type: "user"`；席位层级、组和组织级别的默认值在 claude.ai 设置中配置。需要 `write:spend_limits` 作用域。

有关完整的参数详情和响应模式，请参阅 API 参考中的[创建支出限额](https://platform.claude.com/docs/zh-CN/api/beta/organization/spend_limits/set)。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/spend_limits" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{
      "scope": {"type": "user", "user_id": "user_01AbCdEfGh"},
      "amount": "75000"
    }'
  ```

  ```bash CLI
  ant beta:organization:spend-limits set \
    --scope '{type: user, user_id: user_01AbCdEfGh}' \
    --amount 75000
  ```

  ```python Python
  client = anthropic.Anthropic()

  spend_limit = client.beta.organization.spend_limits.set(
      scope={"type": "user", "user_id": "user_01AbCdEfGh"}, amount="75000"
  )

  print(f"id: {spend_limit.id}")
  print(f"amount: {spend_limit.amount}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const spendLimit = await client.beta.organization.spendLimits.set({
    scope: { type: "user", user_id: "user_01AbCdEfGh" },
    amount: "75000"
  });

  console.log(`id: ${spendLimit.id}`);
  console.log(`amount: ${spendLimit.amount}`);
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.SpendLimits;

  AnthropicClient client = new();

  var spendLimit = await client.Beta.Organization.SpendLimits.Set(
      new()
      {
          Scope = new BetaSpendLimitUserScope("user_01AbCdEfGh"),
          Amount = "75000",
      }
  );

  Console.WriteLine($"id: {spendLimit.ID}");
  Console.WriteLine($"amount: {spendLimit.Amount}");
  ```

  ```go Go
  client := anthropic.NewClient()

  spendLimit, err := client.Beta.Organization.SpendLimits.Set(context.Background(), anthropic.BetaOrganizationSpendLimitSetParams{
  	Scope: anthropic.BetaOrganizationSpendLimitSetParamsScopeUnion{
  		OfUser: &anthropic.BetaSpendLimitUserScopeParam{UserID: "user_01AbCdEfGh"},
  	},
  	Amount: anthropic.String("75000"),
  })
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", spendLimit.ID)
  fmt.Printf("amount: %s\n", spendLimit.Amount)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.spendlimits.SpendLimitSetParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = SpendLimitSetParams.builder()
          .userScope("user_01AbCdEfGh")
          .amount("75000")
          .build();
      var spendLimit = client.beta().organization().spendLimits().set(params);

      IO.println("id: " + spendLimit.id());
      IO.println("amount: " + spendLimit.amount().orElseThrow());
  }
  ```

  ```php PHP
  use Anthropic\Beta\Organization\SpendLimits\SpendLimitUserScope;
  // ...

  $client = new Client();

  $spendLimit = $client->beta->organization->spendLimits->set(
      scope: SpendLimitUserScope::with(userID: 'user_01AbCdEfGh'),
      amount: '75000',
  );

  echo "id: {$spendLimit->id}\n";
  echo "amount: {$spendLimit->amount}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  spend_limit = client.beta.organization.spend_limits.set(
    scope: { type: :user, user_id: "user_01AbCdEfGh" },
    amount: "75000"
  )

  puts "id: #{spend_limit.id}"
  puts "amount: #{spend_limit.amount}"
  ```
</CodeGroup>

```json
{
  "type": "spend_limit",
  "id": "spl_01RsTuVwXyZaBcDeFgHiJk",
  "created_at": "2026-05-11T10:02:44Z",
  "updated_at": "2026-05-11T10:02:44Z",
  "scope": { "type": "user", "user_id": "user_01AbCdEfGh" },
  "amount": "75000",
  "currency": "USD",
  "period": "monthly"
}
```

### 移除按用户的覆盖值

`DELETE /v1/organizations/spend_limits/{spend_limit_id}` 移除按用户的覆盖值，之后该成员将回退到任何继承的席位层级、组或组织默认值。席位层级、组和组织级别的行不能通过此端点删除。需要 `write:spend_limits` 作用域。

有关完整的参数详情和响应模式，请参阅 API 参考中的[删除支出限额](https://platform.claude.com/docs/zh-CN/api/beta/organization/spend_limits/delete)。

<CodeGroup>
  ```bash cURL
  curl -X DELETE "https://api.anthropic.com/v1/organizations/spend_limits/spl_01RsTuVwXyZaBcDeFgHiJk" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:spend-limits delete \
    --spend-limit-id spl_01RsTuVwXyZaBcDeFgHiJk
  ```

  ```python Python
  client = anthropic.Anthropic()

  deleted_spend_limit = client.beta.organization.spend_limits.delete(
      "spl_01RsTuVwXyZaBcDeFgHiJk"
  )

  print(f"id: {deleted_spend_limit.id}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const deletedSpendLimit = await client.beta.organization.spendLimits.delete(
    "spl_01RsTuVwXyZaBcDeFgHiJk"
  );

  console.log(`id: ${deletedSpendLimit.id}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var deletedSpendLimit = await client.Beta.Organization.SpendLimits.Delete(
      "spl_01RsTuVwXyZaBcDeFgHiJk"
  );

  Console.WriteLine($"id: {deletedSpendLimit.ID}");
  ```

  ```go Go
  client := anthropic.NewClient()

  deletedSpendLimit, err := client.Beta.Organization.SpendLimits.Delete(context.Background(), "spl_01RsTuVwXyZaBcDeFgHiJk")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", deletedSpendLimit.ID)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var deletedSpendLimit = client.beta().organization().spendLimits()
      .delete("spl_01RsTuVwXyZaBcDeFgHiJk");

  IO.println("id: " + deletedSpendLimit.id());
  ```

  ```php PHP
  $client = new Client();

  $deletedSpendLimit = $client->beta->organization->spendLimits->delete('spl_01RsTuVwXyZaBcDeFgHiJk');

  echo "id: {$deletedSpendLimit->id}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  deleted_spend_limit = client.beta.organization.spend_limits.delete("spl_01RsTuVwXyZaBcDeFgHiJk")

  puts "id: #{deleted_spend_limit.id}"
  ```
</CodeGroup>

## 支出限额提高请求

### 列出提高请求

`GET /v1/organizations/spend_limit_increase_requests` 列出请求，最新的排在最前。可按 `status[]`（`pending`、`approved`、`denied`）和 `actor_ids[]` 筛选。该列表不包含请求者已不再是组织成员的请求。需要 `read:spend_limits` 作用域。

有关完整的参数详情和响应模式，请参阅 API 参考中的[列出支出限额提高请求](https://platform.claude.com/docs/zh-CN/api/beta/organization/spend_limits/increase_requests/list)。

<CodeGroup>
  ```bash cURL
  curl -g "https://api.anthropic.com/v1/organizations/spend_limit_increase_requests?status[]=pending&limit=50" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:spend-limits:increase-requests list \
    --status pending \
    --limit 50
  ```

  ```python Python
  client = anthropic.Anthropic()

  increase_requests = client.beta.organization.spend_limits.increase_requests.list(
      status=["pending"], limit=50
  )

  for increase_request in increase_requests:
      print(f"{increase_request.id}: {increase_request.status}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const increaseRequests = await client.beta.organization.spendLimits.increaseRequests.list({
    status: ["pending"],
    limit: 50
  });

  for await (const increaseRequest of increaseRequests) {
    console.log(`${increaseRequest.id}: ${increaseRequest.status}`);
  }
  ```

  ```csharp C#
  using Anthropic.Models.Beta.Organization.SpendLimits.IncreaseRequests;

  AnthropicClient client = new();

  var page = await client.Beta.Organization.SpendLimits.IncreaseRequests.List(
      new() { Status = [BetaSpendLimitIncreaseRequestStatus.Pending], Limit = 50 }
  );

  await foreach (var increaseRequest in page.Paginate())
  {
      Console.WriteLine($"{increaseRequest.ID}: {increaseRequest.Status.Raw()}");
  }
  ```

  ```go Go
  client := anthropic.NewClient()

  increaseRequests := client.Beta.Organization.SpendLimits.IncreaseRequests.ListAutoPaging(context.Background(), anthropic.BetaOrganizationSpendLimitIncreaseRequestListParams{
  	Status: []anthropic.BetaSpendLimitIncreaseRequestStatus{anthropic.BetaSpendLimitIncreaseRequestStatusPending},
  	Limit:  anthropic.Int(50),
  })

  for increaseRequests.Next() {
  	increaseRequest := increaseRequests.Current()
  	fmt.Printf("%s: %s\n", increaseRequest.ID, increaseRequest.Status)
  }
  if err := increaseRequests.Err(); err != nil {
  	log.Fatal(err)
  }
  ```

  ```java Java
  import com.anthropic.models.beta.organization.spendlimits.increaserequests.BetaSpendLimitIncreaseRequestStatus;
  import com.anthropic.models.beta.organization.spendlimits.increaserequests.IncreaseRequestListParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = IncreaseRequestListParams.builder()
          .addStatus(BetaSpendLimitIncreaseRequestStatus.PENDING)
          .limit(50)
          .build();
      var increaseRequests = client.beta().organization().spendLimits().increaseRequests().list(params);

      for (var increaseRequest : increaseRequests.autoPager()) {
          IO.println(increaseRequest.id() + ": " + increaseRequest.status().asString());
      }
  }
  ```

  ```php PHP
  $client = new Client();

  $increaseRequests = $client->beta->organization->spendLimits->increaseRequests->list(
      status: ['pending'],
      limit: 50,
  );

  foreach ($increaseRequests->pagingEachItem() as $increaseRequest) {
      echo "{$increaseRequest->id}: {$increaseRequest->status}\n";
  }
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  page = client.beta.organization.spend_limits.increase_requests.list(status: [:pending], limit: 50)

  page.auto_paging_each do |increase_request|
    puts "#{increase_request.id}: #{increase_request.status}"
  end
  ```
</CodeGroup>

每个待处理请求都携带实时的 `spend_summary`，显示请求者当前的有效支出限额和本期至今支出，足以在无需单独查询的情况下做出决定。

### 获取单个提高请求

`GET /v1/organizations/spend_limit_increase_requests/{id}` 按 ID 返回一个请求。需要 `read:spend_limits` 作用域。

有关完整的参数详情和响应模式，请参阅 API 参考中的[检索支出限额提高请求](https://platform.claude.com/docs/zh-CN/api/beta/organization/spend_limits/increase_requests/retrieve)。

<CodeGroup>
  ```bash cURL
  curl "https://api.anthropic.com/v1/organizations/spend_limit_increase_requests/slir_01AbCdEfGhIjKlMnOpQrSt" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21"
  ```

  ```bash CLI
  ant beta:organization:spend-limits:increase-requests retrieve \
    --spend-limit-increase-request-id slir_01AbCdEfGhIjKlMnOpQrSt
  ```

  ```python Python
  client = anthropic.Anthropic()

  increase_request = client.beta.organization.spend_limits.increase_requests.retrieve(
      "slir_01AbCdEfGhIjKlMnOpQrSt"
  )

  print(f"id: {increase_request.id}")
  print(f"status: {increase_request.status}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const increaseRequest = await client.beta.organization.spendLimits.increaseRequests.retrieve(
    "slir_01AbCdEfGhIjKlMnOpQrSt"
  );

  console.log(`id: ${increaseRequest.id}`);
  console.log(`status: ${increaseRequest.status}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var increaseRequest = await client.Beta.Organization.SpendLimits.IncreaseRequests.Retrieve(
      "slir_01AbCdEfGhIjKlMnOpQrSt"
  );

  Console.WriteLine($"id: {increaseRequest.ID}");
  Console.WriteLine($"status: {increaseRequest.Status.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  increaseRequest, err := client.Beta.Organization.SpendLimits.IncreaseRequests.Get(context.Background(), "slir_01AbCdEfGhIjKlMnOpQrSt")
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", increaseRequest.ID)
  fmt.Printf("status: %s\n", increaseRequest.Status)
  ```

  ```java Java
  AnthropicClient client = AnthropicOkHttpClient.fromEnv();

  var increaseRequest = client.beta().organization().spendLimits().increaseRequests()
      .retrieve("slir_01AbCdEfGhIjKlMnOpQrSt");

  IO.println("id: " + increaseRequest.id());
  IO.println("status: " + increaseRequest.status().asString());
  ```

  ```php PHP
  $client = new Client();

  $increaseRequest = $client->beta->organization->spendLimits->increaseRequests->retrieve(
      'slir_01AbCdEfGhIjKlMnOpQrSt',
  );

  echo "id: {$increaseRequest->id}\n";
  echo "status: {$increaseRequest->status}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  increase_request = client.beta.organization.spend_limits.increase_requests.retrieve(
    "slir_01AbCdEfGhIjKlMnOpQrSt"
  )

  puts "id: #{increase_request.id}"
  puts "status: #{increase_request.status}"
  ```
</CodeGroup>

### 批准提高请求

`POST /v1/organizations/spend_limit_increase_requests/{id}/approve` 批准一个待处理请求：它以管理员提供的 `amount` 为请求者写入按用户的支出限额，并将请求转换为 `approved`。请求本身不携带所请求的金额；您在批准时提供新的支出限额。需要 `write:spend_limits` 作用域。

有关完整的参数详情和响应模式，请参阅 API 参考中的[批准支出限额提高请求](https://platform.claude.com/docs/zh-CN/api/beta/organization/spend_limits/increase_requests/approve)。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/spend_limit_increase_requests/slir_01AbCdEfGhIjKlMnOpQrSt/approve" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{"amount": "75000", "suppress_notification": true}'
  ```

  ```bash CLI
  ant beta:organization:spend-limits:increase-requests approve \
    --spend-limit-increase-request-id slir_01AbCdEfGhIjKlMnOpQrSt \
    --amount 75000 \
    --suppress-notification
  ```

  ```python Python
  client = anthropic.Anthropic()

  increase_request = client.beta.organization.spend_limits.increase_requests.approve(
      "slir_01AbCdEfGhIjKlMnOpQrSt", amount="75000", suppress_notification=True
  )

  print(f"id: {increase_request.id}")
  print(f"status: {increase_request.status}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const increaseRequest = await client.beta.organization.spendLimits.increaseRequests.approve(
    "slir_01AbCdEfGhIjKlMnOpQrSt",
    {
      amount: "75000",
      suppress_notification: true
    }
  );

  console.log(`id: ${increaseRequest.id}`);
  console.log(`status: ${increaseRequest.status}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var increaseRequest = await client.Beta.Organization.SpendLimits.IncreaseRequests.Approve(
      "slir_01AbCdEfGhIjKlMnOpQrSt",
      new() { Amount = "75000", SuppressNotification = true }
  );

  Console.WriteLine($"id: {increaseRequest.ID}");
  Console.WriteLine($"status: {increaseRequest.Status.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  increaseRequest, err := client.Beta.Organization.SpendLimits.IncreaseRequests.Approve(
  	context.Background(),
  	"slir_01AbCdEfGhIjKlMnOpQrSt",
  	anthropic.BetaOrganizationSpendLimitIncreaseRequestApproveParams{
  		Amount:               "75000",
  		SuppressNotification: anthropic.Bool(true),
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", increaseRequest.ID)
  fmt.Printf("status: %s\n", increaseRequest.Status)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.spendlimits.increaserequests.IncreaseRequestApproveParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = IncreaseRequestApproveParams.builder()
          .amount("75000")
          .suppressNotification(true)
          .build();
      var increaseRequest = client.beta().organization().spendLimits().increaseRequests()
          .approve("slir_01AbCdEfGhIjKlMnOpQrSt", params);

      IO.println("id: " + increaseRequest.id());
      IO.println("status: " + increaseRequest.status().asString());
  }
  ```

  ```php PHP
  $client = new Client();

  $increaseRequest = $client->beta->organization->spendLimits->increaseRequests->approve(
      'slir_01AbCdEfGhIjKlMnOpQrSt',
      amount: '75000',
      suppressNotification: true,
  );

  echo "id: {$increaseRequest->id}\n";
  echo "status: {$increaseRequest->status}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  increase_request = client.beta.organization.spend_limits.increase_requests.approve(
    "slir_01AbCdEfGhIjKlMnOpQrSt",
    amount: "75000",
    suppress_notification: true
  )

  puts "id: #{increase_request.id}"
  puts "status: #{increase_request.status}"
  ```
</CodeGroup>

### 拒绝提高请求

`POST /v1/organizations/spend_limit_increase_requests/{id}/deny` 拒绝一个待处理请求。对 `denied` 状态是幂等的：拒绝一个已被拒绝的请求会返回 200 及现有资源。该端点会拒绝对已批准请求执行拒绝操作的尝试，以便自动化流程能够区分重试与冲突的决定。需要 `write:spend_limits` 作用域。

有关完整的参数详情和响应模式，请参阅 API 参考中的[拒绝支出限额提高请求](https://platform.claude.com/docs/zh-CN/api/beta/organization/spend_limits/increase_requests/deny)。

<CodeGroup>
  ```bash cURL
  curl -X POST "https://api.anthropic.com/v1/organizations/spend_limit_increase_requests/slir_01AbCdEfGhIjKlMnOpQrSt/deny" \
    -H "content-type: application/json" \
    -H "x-api-key: $ANTHROPIC_API_KEY" \
    -H "anthropic-version: 2026-10-21" \
    -d '{"suppress_notification": true}'
  ```

  ```bash CLI
  ant beta:organization:spend-limits:increase-requests deny \
    --spend-limit-increase-request-id slir_01AbCdEfGhIjKlMnOpQrSt \
    --suppress-notification
  ```

  ```python Python
  client = anthropic.Anthropic()

  increase_request = client.beta.organization.spend_limits.increase_requests.deny(
      "slir_01AbCdEfGhIjKlMnOpQrSt", suppress_notification=True
  )

  print(f"id: {increase_request.id}")
  print(f"status: {increase_request.status}")
  ```

  ```typescript TypeScript
  const client = new Anthropic();

  const increaseRequest = await client.beta.organization.spendLimits.increaseRequests.deny(
    "slir_01AbCdEfGhIjKlMnOpQrSt",
    {
      suppress_notification: true
    }
  );

  console.log(`id: ${increaseRequest.id}`);
  console.log(`status: ${increaseRequest.status}`);
  ```

  ```csharp C#
  AnthropicClient client = new();

  var increaseRequest = await client.Beta.Organization.SpendLimits.IncreaseRequests.Deny(
      "slir_01AbCdEfGhIjKlMnOpQrSt",
      new() { SuppressNotification = true }
  );

  Console.WriteLine($"id: {increaseRequest.ID}");
  Console.WriteLine($"status: {increaseRequest.Status.Raw()}");
  ```

  ```go Go
  client := anthropic.NewClient()

  increaseRequest, err := client.Beta.Organization.SpendLimits.IncreaseRequests.Deny(
  	context.Background(),
  	"slir_01AbCdEfGhIjKlMnOpQrSt",
  	anthropic.BetaOrganizationSpendLimitIncreaseRequestDenyParams{
  		SuppressNotification: anthropic.Bool(true),
  	},
  )
  if err != nil {
  	log.Fatal(err)
  }

  fmt.Printf("id: %s\n", increaseRequest.ID)
  fmt.Printf("status: %s\n", increaseRequest.Status)
  ```

  ```java Java
  import com.anthropic.models.beta.organization.spendlimits.increaserequests.IncreaseRequestDenyParams;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      var params = IncreaseRequestDenyParams.builder()
          .suppressNotification(true)
          .build();
      var increaseRequest = client.beta().organization().spendLimits().increaseRequests()
          .deny("slir_01AbCdEfGhIjKlMnOpQrSt", params);

      IO.println("id: " + increaseRequest.id());
      IO.println("status: " + increaseRequest.status().asString());
  }
  ```

  ```php PHP
  $client = new Client();

  $increaseRequest = $client->beta->organization->spendLimits->increaseRequests->deny(
      'slir_01AbCdEfGhIjKlMnOpQrSt',
      suppressNotification: true,
  );

  echo "id: {$increaseRequest->id}\n";
  echo "status: {$increaseRequest->status}\n";
  ```

  ```ruby Ruby
  client = Anthropic::Client.new

  increase_request = client.beta.organization.spend_limits.increase_requests.deny(
    "slir_01AbCdEfGhIjKlMnOpQrSt",
    suppress_notification: true
  )

  puts "id: #{increase_request.id}"
  puts "status: #{increase_request.status}"
  ```
</CodeGroup>

## 示例工作流

其中一些工作流将支出限额 API 与[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 的成本端点结合使用。该成本端点专为跨日期范围的组织级支出报告而设计。`GET /spend_limits/effective` 返回当前适用于每位成员的上限。先用成本端点进行一次扫描以发现需要关注哪些成员，然后用 `/effective` 读取他们当前的上限。

支出限额端点需要 `spend_limits` 作用域，而成本端点需要 `read:analytics`。示例使用一个具有其所调用的每个 API 所需作用域的 Admin API 密钥。主要所有者[在创建密钥时选择这些作用域](https://platform.claude.com/docs/zh-CN/manage-claude/admin-api-keys#choose-scopes-for-a-claude-enterprise-key)。两者中的所有货币值都是以最小单位（美分）表示的十进制字符串。两个 API 都使用不透明游标进行分页。请设置明确的 `limit` 并通过 `next_page` 翻页，直到其为 `null`，以覆盖整个组织。

### 自动化提高请求审核流程

运行一个定时任务，获取待处理请求，应用您组织的批准策略，并解决每个请求。

1. 列出待处理请求：

   <CodeGroup>
     ```bash cURL
     curl -g "https://api.anthropic.com/v1/organizations/spend_limit_increase_requests?status[]=pending&limit=100" \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "anthropic-version: 2026-10-21"
     ```

     ```bash CLI
     ant beta:organization:spend-limits:increase-requests list \
       --status pending \
       --limit 100
     ```

     ```python Python
     client = anthropic.Anthropic()

     increase_requests = client.beta.organization.spend_limits.increase_requests.list(
         status=["pending"], limit=100
     )

     for increase_request in increase_requests:
         print(f"{increase_request.id}: {increase_request.status}")
     ```

     ```typescript TypeScript
     const client = new Anthropic();

     const increaseRequests = await client.beta.organization.spendLimits.increaseRequests.list({
       status: ["pending"],
       limit: 100
     });

     for await (const increaseRequest of increaseRequests) {
       console.log(`${increaseRequest.id}: ${increaseRequest.status}`);
     }
     ```

     ```csharp C#
     using Anthropic.Models.Beta.Organization.SpendLimits.IncreaseRequests;

     AnthropicClient client = new();

     var page = await client.Beta.Organization.SpendLimits.IncreaseRequests.List(
         new() { Status = [BetaSpendLimitIncreaseRequestStatus.Pending], Limit = 100 }
     );

     await foreach (var increaseRequest in page.Paginate())
     {
         Console.WriteLine($"{increaseRequest.ID}: {increaseRequest.Status.Raw()}");
     }
     ```

     ```go Go
     client := anthropic.NewClient()

     increaseRequests := client.Beta.Organization.SpendLimits.IncreaseRequests.ListAutoPaging(context.Background(), anthropic.BetaOrganizationSpendLimitIncreaseRequestListParams{
     	Status: []anthropic.BetaSpendLimitIncreaseRequestStatus{anthropic.BetaSpendLimitIncreaseRequestStatusPending},
     	Limit:  anthropic.Int(100),
     })

     for increaseRequests.Next() {
     	increaseRequest := increaseRequests.Current()
     	fmt.Printf("%s: %s\n", increaseRequest.ID, increaseRequest.Status)
     }
     if err := increaseRequests.Err(); err != nil {
     	log.Fatal(err)
     }
     ```

     ```java Java
     import com.anthropic.models.beta.organization.spendlimits.increaserequests.BetaSpendLimitIncreaseRequestStatus;
     import com.anthropic.models.beta.organization.spendlimits.increaserequests.IncreaseRequestListParams;

     void main() {
         AnthropicClient client = AnthropicOkHttpClient.fromEnv();

         var params = IncreaseRequestListParams.builder()
             .addStatus(BetaSpendLimitIncreaseRequestStatus.PENDING)
             .limit(100)
             .build();
         var increaseRequests = client.beta().organization().spendLimits().increaseRequests().list(params);

         for (var increaseRequest : increaseRequests.autoPager()) {
             IO.println(increaseRequest.id() + ": " + increaseRequest.status().asString());
         }
     }
     ```

     ```php PHP
     $client = new Client();

     $increaseRequests = $client->beta->organization->spendLimits->increaseRequests->list(
         status: ['pending'],
         limit: 100,
     );

     foreach ($increaseRequests->pagingEachItem() as $increaseRequest) {
         echo "{$increaseRequest->id}: {$increaseRequest->status}\n";
     }
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     page = client.beta.organization.spend_limits.increase_requests.list(status: [:pending], limit: 100)

     page.auto_paging_each do |increase_request|
       puts "#{increase_request.id}: #{increase_request.status}"
     end
     ```
   </CodeGroup>

   每个请求都携带请求者的 `actor.user_id` 和实时的 `spend_summary`，其中包含其当前有效的 `amount` 和 `period_to_date_spend`，足以在无需单独查询的情况下做出决定。

2. 应用您的策略。例如，当成员当前的 `amount` 低于某个阈值时自动批准，并将较大的上限转交人工审核。

3. 解决每个请求。要批准，请提供新的上限：

   <CodeGroup>
     ```bash cURL
     curl -X POST "https://api.anthropic.com/v1/organizations/spend_limit_increase_requests/slir_01AbCdEfGhIjKlMnOpQrSt/approve" \
       -H "content-type: application/json" \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "anthropic-version: 2026-10-21" \
       -d '{"amount": "75000", "suppress_notification": true}'
     ```

     ```bash CLI
     ant beta:organization:spend-limits:increase-requests approve \
       --spend-limit-increase-request-id slir_01AbCdEfGhIjKlMnOpQrSt \
       --amount 75000 \
       --suppress-notification
     ```

     ```python Python
     client = anthropic.Anthropic()

     increase_request = client.beta.organization.spend_limits.increase_requests.approve(
         "slir_01AbCdEfGhIjKlMnOpQrSt", amount="75000", suppress_notification=True
     )

     print(f"id: {increase_request.id}")
     print(f"status: {increase_request.status}")
     ```

     ```typescript TypeScript
     const client = new Anthropic();

     const increaseRequest = await client.beta.organization.spendLimits.increaseRequests.approve(
       "slir_01AbCdEfGhIjKlMnOpQrSt",
       {
         amount: "75000",
         suppress_notification: true
       }
     );

     console.log(`id: ${increaseRequest.id}`);
     console.log(`status: ${increaseRequest.status}`);
     ```

     ```csharp C#
     AnthropicClient client = new();

     var increaseRequest = await client.Beta.Organization.SpendLimits.IncreaseRequests.Approve(
         "slir_01AbCdEfGhIjKlMnOpQrSt",
         new() { Amount = "75000", SuppressNotification = true }
     );

     Console.WriteLine($"id: {increaseRequest.ID}");
     Console.WriteLine($"status: {increaseRequest.Status.Raw()}");
     ```

     ```go Go
     client := anthropic.NewClient()

     increaseRequest, err := client.Beta.Organization.SpendLimits.IncreaseRequests.Approve(
     	context.Background(),
     	"slir_01AbCdEfGhIjKlMnOpQrSt",
     	anthropic.BetaOrganizationSpendLimitIncreaseRequestApproveParams{
     		Amount:               "75000",
     		SuppressNotification: anthropic.Bool(true),
     	},
     )
     if err != nil {
     	log.Fatal(err)
     }

     fmt.Printf("id: %s\n", increaseRequest.ID)
     fmt.Printf("status: %s\n", increaseRequest.Status)
     ```

     ```java Java
     import com.anthropic.models.beta.organization.spendlimits.increaserequests.IncreaseRequestApproveParams;

     void main() {
         AnthropicClient client = AnthropicOkHttpClient.fromEnv();

         var params = IncreaseRequestApproveParams.builder()
             .amount("75000")
             .suppressNotification(true)
             .build();
         var increaseRequest = client.beta().organization().spendLimits().increaseRequests()
             .approve("slir_01AbCdEfGhIjKlMnOpQrSt", params);

         IO.println("id: " + increaseRequest.id());
         IO.println("status: " + increaseRequest.status().asString());
     }
     ```

     ```php PHP
     $client = new Client();

     $increaseRequest = $client->beta->organization->spendLimits->increaseRequests->approve(
         'slir_01AbCdEfGhIjKlMnOpQrSt',
         amount: '75000',
         suppressNotification: true,
     );

     echo "id: {$increaseRequest->id}\n";
     echo "status: {$increaseRequest->status}\n";
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     increase_request = client.beta.organization.spend_limits.increase_requests.approve(
       "slir_01AbCdEfGhIjKlMnOpQrSt",
       amount: "75000",
       suppress_notification: true
     )

     puts "id: #{increase_request.id}"
     puts "status: #{increase_request.status}"
     ```
   </CodeGroup>

   要拒绝，请改为使用相同的 ID 调用[拒绝端点](https://platform.claude.com/docs/zh-CN/manage-claude/spend-limits-api#deny-an-increase-request)。当您自己的系统会通知请求者时，请传递 `suppress_notification: true`。

### 识别接近支出限额的成员

找出接近上限的成员，以便在他们被阻止之前提高上限。

1. 从用量和成本分析 API 获取每个成员本月至今的支出（每个成员一行，默认按支出从高到低排列）：

   <CodeGroup>
     ```bash cURL
     curl -g "https://api.anthropic.com/v1/organizations/analytics/cost?\
     starting_at=2026-06-01T00:00:00Z&\
     group_by[]=actor&\
     actor_types[]=user_actor&\
     limit=1000" \
       -H "anthropic-version: 2026-10-21" \
       -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
       -H "x-api-key: $ANTHROPIC_API_KEY"
     ```

     ```bash CLI
     ant beta:organization:analytics:cost list \
       --starting-at 2026-06-01T00:00:00Z \
       --actor-type user_actor \
       --group-by actor \
       --limit 1000
     ```

     ```python Python
     client = anthropic.Anthropic()

     cost_rows = client.beta.organization.analytics.cost.list(
         starting_at="2026-06-01T00:00:00Z",
         actor_types=["user_actor"],
         group_by=["actor"],
         limit=1000,
     )

     for cost_row in cost_rows:
         actor = cost_row.group.actor
         if actor is not None and actor.type == "user_actor":
             print(f"{actor.user_id}: {cost_row.cost.amount}")
     ```

     ```typescript TypeScript
     const client = new Anthropic();

     const costRows = await client.beta.organization.analytics.cost.list({
       starting_at: "2026-06-01T00:00:00Z",
       actor_types: ["user_actor"],
       group_by: ["actor"],
       limit: 1000
     });

     for await (const costRow of costRows) {
       const { actor } = costRow.group;
       if (actor?.type === "user_actor") {
         console.log(`${actor.user_id}: ${costRow.cost.amount}`);
       }
     }
     ```

     ```csharp C#
     using Anthropic.Models.Beta.Organization.Analytics;
     using Anthropic.Models.Beta.Organization.Analytics.Cost;

     AnthropicClient client = new();

     var page = await client.Beta.Organization.Analytics.Cost.List(
         new()
         {
             StartingAt = DateTimeOffset.Parse("2026-06-01T00:00:00Z"),
             ActorTypes = [BetaAnalyticsActorType.UserActor],
             GroupBy = [BetaAnalyticsCostGroupBy.Actor],
             Limit = 1000,
         }
     );

     await foreach (var costRow in page.Paginate())
     {
         if (costRow.Group.Actor is { } actor && actor.TryPickReportUser(out var user))
         {
             Console.WriteLine($"{user.UserID}: {costRow.Cost.Amount}");
         }
     }
     ```

     ```go Go
     client := anthropic.NewClient()

     costRows := client.Beta.Organization.Analytics.Cost.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsCostListParams{
     	StartingAt: time.Date(2026, time.June, 1, 0, 0, 0, 0, time.UTC),
     	ActorTypes: []anthropic.BetaAnalyticsActorType{anthropic.BetaAnalyticsActorTypeUserActor},
     	GroupBy:    []anthropic.BetaAnalyticsCostGroupBy{anthropic.BetaAnalyticsCostGroupByActor},
     	Limit:      anthropic.Int(1000),
     })

     for costRows.Next() {
     	costRow := costRows.Current()
     	if user, ok := costRow.Group.Actor.AsAny().(anthropic.BetaAnalyticsReportUserActor); ok {
     		fmt.Printf("%s: %s\n", user.UserID, costRow.Cost.Amount)
     	}
     }
     if err := costRows.Err(); err != nil {
     	log.Fatal(err)
     }
     ```

     ```java Java
     import com.anthropic.models.beta.organization.analytics.BetaAnalyticsActorType;
     import com.anthropic.models.beta.organization.analytics.cost.BetaAnalyticsCostGroupBy;
     import com.anthropic.models.beta.organization.analytics.cost.CostListParams;
     import java.time.OffsetDateTime;

     void main() {
         AnthropicClient client = AnthropicOkHttpClient.fromEnv();

         var params = CostListParams.builder()
             .startingAt(OffsetDateTime.parse("2026-06-01T00:00:00Z"))
             .addActorType(BetaAnalyticsActorType.USER_ACTOR)
             .addGroupBy(BetaAnalyticsCostGroupBy.ACTOR)
             .limit(1000)
             .build();
         var costRows = client.beta().organization().analytics().cost().list(params);

         for (var costRow : costRows.autoPager()) {
             costRow.group().actor().flatMap(actor -> actor.user()).ifPresent(user ->
                 IO.println(user.userId() + ": " + costRow.cost().amount())
             );
         }
     }
     ```

     ```php PHP
     use Anthropic\Beta\Organization\Analytics\AnalyticsReportUserActor;
     // ...

     $client = new Client();

     $costRows = $client->beta->organization->analytics->cost->list(
         startingAt: new \DateTimeImmutable('2026-06-01T00:00:00Z'),
         actorTypes: ['user_actor'],
         groupBy: ['actor'],
         limit: 1000,
     );

     foreach ($costRows->pagingEachItem() as $costRow) {
         $actor = $costRow->group->actor;
         if ($actor instanceof AnalyticsReportUserActor) {
             echo "{$actor->userID}: {$costRow->cost->amount}\n";
         }
     }
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     page = client.beta.organization.analytics.cost.list(
       starting_at: Time.utc(2026, 6, 1),
       actor_types: [:user_actor],
       group_by: [:actor],
       limit: 1000
     )

     page.auto_paging_each do |cost_row|
       actor = cost_row.group.actor
       if actor.is_a?(Anthropic::Models::Beta::Organization::BetaAnalyticsReportUserActor)
         puts "#{actor.user_id}: #{cost_row.cost.amount}"
       end
     end
     ```
   </CodeGroup>

   每行包含 `group.actor.user_id`、`group.actor.email_address` 和 `cost.amount`（成员的支出，以美分计）。通过 `next_page` 翻页以覆盖整个组织。

2. 对于支出最高的成员（或所有超过某个美元阈值的成员），分批获取有效上限：

   <CodeGroup>
     ```bash cURL
     curl -g "https://api.anthropic.com/v1/organizations/spend_limits/effective?user_ids[]=user_01AbCdEfGh&user_ids[]=user_01JkLmNoPq&limit=100" \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "anthropic-version: 2026-10-21"
     ```

     ```bash CLI
     ant beta:organization:spend-limits:effective list \
       --user-id user_01AbCdEfGh \
       --user-id user_01JkLmNoPq \
       --limit 100
     ```

     ```python Python
     client = anthropic.Anthropic()

     effective_limits = client.beta.organization.spend_limits.effective.list(
         user_ids=["user_01AbCdEfGh", "user_01JkLmNoPq"], limit=100
     )

     for effective_limit in effective_limits:
         scope = effective_limit.scope
         if scope.type == "user":
             amount = effective_limit.amount or "unlimited"
             print(f"{scope.user_id}: {effective_limit.period_to_date_spend} of {amount}")
     ```

     ```typescript TypeScript
     const client = new Anthropic();

     const effectiveLimits = await client.beta.organization.spendLimits.effective.list({
       user_ids: ["user_01AbCdEfGh", "user_01JkLmNoPq"],
       limit: 100
     });

     for await (const effectiveLimit of effectiveLimits) {
       const { scope } = effectiveLimit;
       if (scope.type === "user") {
         const amount = effectiveLimit.amount ?? "unlimited";
         console.log(`${scope.user_id}: ${effectiveLimit.period_to_date_spend} of ${amount}`);
       }
     }
     ```

     ```csharp C#
     AnthropicClient client = new();

     var page = await client.Beta.Organization.SpendLimits.Effective.List(
         new() { UserIds = ["user_01AbCdEfGh", "user_01JkLmNoPq"], Limit = 100 }
     );

     await foreach (var effectiveLimit in page.Paginate())
     {
         if (effectiveLimit.Scope.TryPickBetaSpendLimitUser(out var userScope))
         {
             var amount = effectiveLimit.Amount ?? "unlimited";
             Console.WriteLine($"{userScope.UserID}: {effectiveLimit.PeriodToDateSpend} of {amount}");
         }
     }
     ```

     ```go Go
     client := anthropic.NewClient()

     effectiveLimits := client.Beta.Organization.SpendLimits.Effective.ListAutoPaging(context.Background(), anthropic.BetaOrganizationSpendLimitEffectiveListParams{
     	UserIDs: []string{"user_01AbCdEfGh", "user_01JkLmNoPq"},
     	Limit:   anthropic.Int(100),
     })

     for effectiveLimits.Next() {
     	effectiveLimit := effectiveLimits.Current()
     	if userScope, ok := effectiveLimit.Scope.AsAny().(anthropic.BetaSpendLimitUserScope); ok {
     		amount := effectiveLimit.Amount
     		if !effectiveLimit.JSON.Amount.Valid() {
     			amount = "unlimited"
     		}
     		fmt.Printf("%s: %s of %s\n", userScope.UserID, effectiveLimit.PeriodToDateSpend, amount)
     	}
     }
     if err := effectiveLimits.Err(); err != nil {
     	log.Fatal(err)
     }
     ```

     ```java Java
     import com.anthropic.models.beta.organization.spendlimits.effective.EffectiveListParams;

     void main() {
         AnthropicClient client = AnthropicOkHttpClient.fromEnv();

         var params = EffectiveListParams.builder()
             .addUserId("user_01AbCdEfGh")
             .addUserId("user_01JkLmNoPq")
             .limit(100)
             .build();
         var effectiveLimits = client.beta().organization().spendLimits().effective().list(params);

         for (var effectiveLimit : effectiveLimits.autoPager()) {
             var amount = effectiveLimit.amount().orElse("unlimited");
             effectiveLimit.scope().user().ifPresent(userScope ->
                 IO.println(userScope.userId() + ": " + effectiveLimit.periodToDateSpend() + " of " + amount)
             );
         }
     }
     ```

     ```php PHP
     use Anthropic\Beta\Organization\SpendLimits\SpendLimitUserScope;
     // ...

     $client = new Client();

     $effectiveLimits = $client->beta->organization->spendLimits->effective->list(
         userIDs: ['user_01AbCdEfGh', 'user_01JkLmNoPq'],
         limit: 100,
     );

     foreach ($effectiveLimits->pagingEachItem() as $effectiveLimit) {
         $scope = $effectiveLimit->scope;
         if ($scope instanceof SpendLimitUserScope) {
             $amount = $effectiveLimit->amount ?? 'unlimited';
             echo "{$scope->userID}: {$effectiveLimit->periodToDateSpend} of {$amount}\n";
         }
     }
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     page = client.beta.organization.spend_limits.effective.list(
       user_ids: ["user_01AbCdEfGh", "user_01JkLmNoPq"],
       limit: 100
     )

     page.auto_paging_each do |effective_limit|
       scope = effective_limit.scope
       if scope.is_a?(Anthropic::Models::Beta::Organization::BetaSpendLimitUserScope)
         amount = effective_limit.amount || "unlimited"
         puts "#{scope.user_id}: #{effective_limit.period_to_date_spend} of #{amount}"
       end
     end
     ```
   </CodeGroup>

   每行以 `amount` 返回上限（`null` = 无限制，`"0"` = 仅限所含用量），同时返回 `period_to_date_spend`。

3. 对于每位上限为正数的成员，计算 `period_to_date_spend / amount`，并标记达到或超过您阈值（例如 80%）的成员。将 `"0"` 上限视为已达限额。服务器端没有针对此比率的筛选器。

4. 对被标记的成员采取行动：使用 `POST /v1/organizations/spend_limits` 提高上限、批准其待处理的提额请求（如果存在），或联系该成员。

### 查找用量快速变化的成员

找出支出环比上周大幅跃升的成员。

1. 从用量和成本分析 API 获取过去 2 周内每个成员的每日成本：

   <CodeGroup>
     ```bash cURL
     curl -g "https://api.anthropic.com/v1/organizations/analytics/cost?\
     starting_at=2026-06-09T00:00:00Z&\
     ending_at=2026-06-23T00:00:00Z&\
     time_bucket_width=1d&\
     group_by[]=actor&\
     actor_types[]=user_actor&\
     limit=1000" \
       -H "anthropic-version: 2026-10-21" \
       -H "anthropic-beta: analytics-usage-cost-2026-08-24" \
       -H "x-api-key: $ANTHROPIC_API_KEY"
     ```

     ```bash CLI
     ant beta:organization:analytics:cost list \
       --starting-at 2026-06-09T00:00:00Z \
       --ending-at 2026-06-23T00:00:00Z \
       --time-bucket-width 1d \
       --actor-type user_actor \
       --group-by actor \
       --limit 1000
     ```

     ```python Python
     client = anthropic.Anthropic()

     cost_rows = client.beta.organization.analytics.cost.list(
         starting_at="2026-06-09T00:00:00Z",
         ending_at="2026-06-23T00:00:00Z",
         time_bucket_width="1d",
         actor_types=["user_actor"],
         group_by=["actor"],
         limit=1000,
     )

     for cost_row in cost_rows:
         actor = cost_row.group.actor
         if actor is not None and actor.type == "user_actor":
             print(f"{actor.user_id} {cost_row.starting_at}: {cost_row.cost.amount}")
     ```

     ```typescript TypeScript
     const client = new Anthropic();

     const costRows = await client.beta.organization.analytics.cost.list({
       starting_at: "2026-06-09T00:00:00Z",
       ending_at: "2026-06-23T00:00:00Z",
       time_bucket_width: "1d",
       actor_types: ["user_actor"],
       group_by: ["actor"],
       limit: 1000
     });

     for await (const costRow of costRows) {
       const { actor } = costRow.group;
       if (actor?.type === "user_actor") {
         console.log(`${actor.user_id} ${costRow.starting_at}: ${costRow.cost.amount}`);
       }
     }
     ```

     ```csharp C#
     using Anthropic.Models.Beta.Organization.Analytics;
     using Anthropic.Models.Beta.Organization.Analytics.Cost;

     AnthropicClient client = new();

     var page = await client.Beta.Organization.Analytics.Cost.List(
         new()
         {
             StartingAt = DateTimeOffset.Parse("2026-06-09T00:00:00Z"),
             EndingAt = DateTimeOffset.Parse("2026-06-23T00:00:00Z"),
             TimeBucketWidth = BetaAnalyticsTimeBucketWidth.Day,
             ActorTypes = [BetaAnalyticsActorType.UserActor],
             GroupBy = [BetaAnalyticsCostGroupBy.Actor],
             Limit = 1000,
         }
     );

     await foreach (var costRow in page.Paginate())
     {
         if (costRow.Group.Actor is { } actor && actor.TryPickReportUser(out var user))
         {
             Console.WriteLine($"{user.UserID} {costRow.StartingAt}: {costRow.Cost.Amount}");
         }
     }
     ```

     ```go Go
     client := anthropic.NewClient()

     costRows := client.Beta.Organization.Analytics.Cost.ListAutoPaging(context.Background(), anthropic.BetaOrganizationAnalyticsCostListParams{
     	StartingAt:      time.Date(2026, time.June, 9, 0, 0, 0, 0, time.UTC),
     	EndingAt:        anthropic.Time(time.Date(2026, time.June, 23, 0, 0, 0, 0, time.UTC)),
     	TimeBucketWidth: anthropic.BetaAnalyticsTimeBucketWidthDay,
     	ActorTypes:      []anthropic.BetaAnalyticsActorType{anthropic.BetaAnalyticsActorTypeUserActor},
     	GroupBy:         []anthropic.BetaAnalyticsCostGroupBy{anthropic.BetaAnalyticsCostGroupByActor},
     	Limit:           anthropic.Int(1000),
     })

     for costRows.Next() {
     	costRow := costRows.Current()
     	if user, ok := costRow.Group.Actor.AsAny().(anthropic.BetaAnalyticsReportUserActor); ok {
     		fmt.Printf("%s %s: %s\n", user.UserID, costRow.StartingAt.Format(time.RFC3339), costRow.Cost.Amount)
     	}
     }
     if err := costRows.Err(); err != nil {
     	log.Fatal(err)
     }
     ```

     ```java Java
     import com.anthropic.models.beta.organization.analytics.BetaAnalyticsActorType;
     import com.anthropic.models.beta.organization.analytics.BetaAnalyticsTimeBucketWidth;
     import com.anthropic.models.beta.organization.analytics.cost.BetaAnalyticsCostGroupBy;
     import com.anthropic.models.beta.organization.analytics.cost.CostListParams;
     import java.time.OffsetDateTime;

     void main() {
         AnthropicClient client = AnthropicOkHttpClient.fromEnv();

         var params = CostListParams.builder()
             .startingAt(OffsetDateTime.parse("2026-06-09T00:00:00Z"))
             .endingAt(OffsetDateTime.parse("2026-06-23T00:00:00Z"))
             .timeBucketWidth(BetaAnalyticsTimeBucketWidth.DAY)
             .addActorType(BetaAnalyticsActorType.USER_ACTOR)
             .addGroupBy(BetaAnalyticsCostGroupBy.ACTOR)
             .limit(1000)
             .build();
         var costRows = client.beta().organization().analytics().cost().list(params);

         for (var costRow : costRows.autoPager()) {
             costRow.group().actor().flatMap(actor -> actor.user()).ifPresent(user ->
                 IO.println(user.userId() + " " + costRow.startingAt() + ": " + costRow.cost().amount())
             );
         }
     }
     ```

     ```php PHP
     use Anthropic\Beta\Organization\Analytics\AnalyticsReportUserActor;
     // ...

     $client = new Client();

     $costRows = $client->beta->organization->analytics->cost->list(
         startingAt: new \DateTimeImmutable('2026-06-09T00:00:00Z'),
         endingAt: new \DateTimeImmutable('2026-06-23T00:00:00Z'),
         timeBucketWidth: '1d',
         actorTypes: ['user_actor'],
         groupBy: ['actor'],
         limit: 1000,
     );

     foreach ($costRows->pagingEachItem() as $costRow) {
         $actor = $costRow->group->actor;
         if ($actor instanceof AnalyticsReportUserActor) {
             $startingAt = $costRow->startingAt->format(\DateTimeInterface::ATOM);
             echo "{$actor->userID} {$startingAt}: {$costRow->cost->amount}\n";
         }
     }
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     page = client.beta.organization.analytics.cost.list(
       starting_at: Time.utc(2026, 6, 9),
       ending_at: Time.utc(2026, 6, 23),
       time_bucket_width: :"1d",
       actor_types: [:user_actor],
       group_by: [:actor],
       limit: 1000
     )

     page.auto_paging_each do |cost_row|
       actor = cost_row.group.actor
       if actor.is_a?(Anthropic::Models::Beta::Organization::BetaAnalyticsReportUserActor)
         puts "#{actor.user_id} #{cost_row.starting_at}: #{cost_row.cost.amount}"
       end
     end
     ```
   </CodeGroup>

   设置 `time_bucket_width` 后，每个成员在有用量的每一天各占一行。请通过 `next_page` 翻页以收集每个成员的完整序列。

2. 按 `group.actor.user_id` 对行进行分组。对于每个成员，分别汇总最近 7 天和之前 7 天的数据。标记最近一周超出前一周达到您所选倍数（例如三倍）的成员。最近几天的成本仍可能发生变化。为了获得可重复的比较结果，请将 `ending_at` 设置为不晚于您已看到的某个 `data_refreshed_at`（请参阅[数据新鲜度](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api#data-freshness)）。

3. 对被标记的成员采取行动：使用 `POST /v1/organizations/spend_limits` 调整上限，或联系该成员。

### 在事件期间临时提高成员的支出限额

在事件处理期间为事件响应人员留出工作空间：在事件开始时提高其支出上限，并在事件关闭后将其恢复。请以您的事件管理系统作为提额的前提条件，例如，要求存在一个已分配给该成员的进行中事件 ID。

1. 读取成员当前的上限，并记录下来以便回滚：

   <CodeGroup>
     ```bash cURL
     curl -g "https://api.anthropic.com/v1/organizations/spend_limits/effective?user_ids[]=user_01AbCdEfGh&period[]=monthly" \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "anthropic-version: 2026-10-21"
     ```

     ```bash CLI
     ant beta:organization:spend-limits:effective list \
       --user-id user_01AbCdEfGh \
       --period monthly
     ```

     ```python Python
     client = anthropic.Anthropic()

     effective_limits = client.beta.organization.spend_limits.effective.list(
         user_ids=["user_01AbCdEfGh"], period=["monthly"]
     )

     for effective_limit in effective_limits:
         scope = effective_limit.scope
         if scope.type == "user":
             amount = effective_limit.amount or "unlimited"
             print(f"{scope.user_id}: {effective_limit.period_to_date_spend} of {amount}")
     ```

     ```typescript TypeScript
     const client = new Anthropic();

     const effectiveLimits = await client.beta.organization.spendLimits.effective.list({
       user_ids: ["user_01AbCdEfGh"],
       period: ["monthly"]
     });

     for await (const effectiveLimit of effectiveLimits) {
       const { scope } = effectiveLimit;
       if (scope.type === "user") {
         const amount = effectiveLimit.amount ?? "unlimited";
         console.log(`${scope.user_id}: ${effectiveLimit.period_to_date_spend} of ${amount}`);
       }
     }
     ```

     ```csharp C#
     using Anthropic.Models.Beta.Organization.SpendLimits;

     AnthropicClient client = new();

     var page = await client.Beta.Organization.SpendLimits.Effective.List(
         new() { UserIds = ["user_01AbCdEfGh"], Period = [BetaSpendLimitPeriod.Monthly] }
     );

     await foreach (var effectiveLimit in page.Paginate())
     {
         if (effectiveLimit.Scope.TryPickBetaSpendLimitUser(out var userScope))
         {
             var amount = effectiveLimit.Amount ?? "unlimited";
             Console.WriteLine($"{userScope.UserID}: {effectiveLimit.PeriodToDateSpend} of {amount}");
         }
     }
     ```

     ```go Go
     client := anthropic.NewClient()

     effectiveLimits := client.Beta.Organization.SpendLimits.Effective.ListAutoPaging(context.Background(), anthropic.BetaOrganizationSpendLimitEffectiveListParams{
     	UserIDs: []string{"user_01AbCdEfGh"},
     	Period:  []anthropic.BetaSpendLimitPeriod{anthropic.BetaSpendLimitPeriodMonthly},
     })

     for effectiveLimits.Next() {
     	effectiveLimit := effectiveLimits.Current()
     	if userScope, ok := effectiveLimit.Scope.AsAny().(anthropic.BetaSpendLimitUserScope); ok {
     		amount := effectiveLimit.Amount
     		if !effectiveLimit.JSON.Amount.Valid() {
     			amount = "unlimited"
     		}
     		fmt.Printf("%s: %s of %s\n", userScope.UserID, effectiveLimit.PeriodToDateSpend, amount)
     	}
     }
     if err := effectiveLimits.Err(); err != nil {
     	log.Fatal(err)
     }
     ```

     ```java Java
     import com.anthropic.models.beta.organization.spendlimits.BetaSpendLimitPeriod;
     import com.anthropic.models.beta.organization.spendlimits.effective.EffectiveListParams;

     void main() {
         AnthropicClient client = AnthropicOkHttpClient.fromEnv();

         var params = EffectiveListParams.builder()
             .addUserId("user_01AbCdEfGh")
             .addPeriod(BetaSpendLimitPeriod.MONTHLY)
             .build();
         var effectiveLimits = client.beta().organization().spendLimits().effective().list(params);

         for (var effectiveLimit : effectiveLimits.autoPager()) {
             var amount = effectiveLimit.amount().orElse("unlimited");
             effectiveLimit.scope().user().ifPresent(userScope ->
                 IO.println(userScope.userId() + ": " + effectiveLimit.periodToDateSpend() + " of " + amount)
             );
         }
     }
     ```

     ```php PHP
     use Anthropic\Beta\Organization\SpendLimits\SpendLimitUserScope;
     // ...

     $client = new Client();

     $effectiveLimits = $client->beta->organization->spendLimits->effective->list(
         userIDs: ['user_01AbCdEfGh'],
         period: ['monthly'],
     );

     foreach ($effectiveLimits->pagingEachItem() as $effectiveLimit) {
         $scope = $effectiveLimit->scope;
         if ($scope instanceof SpendLimitUserScope) {
             $amount = $effectiveLimit->amount ?? 'unlimited';
             echo "{$scope->userID}: {$effectiveLimit->periodToDateSpend} of {$amount}\n";
         }
     }
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     page = client.beta.organization.spend_limits.effective.list(
       user_ids: ["user_01AbCdEfGh"],
       period: [:monthly]
     )

     page.auto_paging_each do |effective_limit|
       scope = effective_limit.scope
       if scope.is_a?(Anthropic::Models::Beta::Organization::BetaSpendLimitUserScope)
         amount = effective_limit.amount || "unlimited"
         puts "#{scope.user_id}: #{effective_limit.period_to_date_spend} of #{amount}"
       end
     end
     ```
   </CodeGroup>

2. 提高上限：

   <CodeGroup>
     ```bash cURL
     curl -X POST "https://api.anthropic.com/v1/organizations/spend_limits" \
       -H "content-type: application/json" \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "anthropic-version: 2026-10-21" \
       -d '{
         "scope": {"type": "user", "user_id": "user_01AbCdEfGh"},
         "amount": "500000",
         "period": "monthly"
       }'
     ```

     ```bash CLI
     ant beta:organization:spend-limits set \
       --scope '{type: user, user_id: user_01AbCdEfGh}' \
       --amount 500000 \
       --period monthly
     ```

     ```python Python
     client = anthropic.Anthropic()

     spend_limit = client.beta.organization.spend_limits.set(
         scope={"type": "user", "user_id": "user_01AbCdEfGh"},
         amount="500000",
         period="monthly",
     )

     print(f"id: {spend_limit.id}")
     print(f"amount: {spend_limit.amount}")
     ```

     ```typescript TypeScript
     const client = new Anthropic();

     const spendLimit = await client.beta.organization.spendLimits.set({
       scope: { type: "user", user_id: "user_01AbCdEfGh" },
       amount: "500000",
       period: "monthly"
     });

     console.log(`id: ${spendLimit.id}`);
     console.log(`amount: ${spendLimit.amount}`);
     ```

     ```csharp C#
     using Anthropic.Models.Beta.Organization.SpendLimits;

     AnthropicClient client = new();

     var spendLimit = await client.Beta.Organization.SpendLimits.Set(
         new()
         {
             Scope = new BetaSpendLimitUserScope("user_01AbCdEfGh"),
             Amount = "500000",
             Period = BetaSpendLimitPeriod.Monthly,
         }
     );

     Console.WriteLine($"id: {spendLimit.ID}");
     Console.WriteLine($"amount: {spendLimit.Amount}");
     ```

     ```go Go
     client := anthropic.NewClient()

     spendLimit, err := client.Beta.Organization.SpendLimits.Set(context.Background(), anthropic.BetaOrganizationSpendLimitSetParams{
     	Scope: anthropic.BetaOrganizationSpendLimitSetParamsScopeUnion{
     		OfUser: &anthropic.BetaSpendLimitUserScopeParam{UserID: "user_01AbCdEfGh"},
     	},
     	Amount: anthropic.String("500000"),
     	Period: anthropic.BetaSpendLimitPeriodMonthly,
     })
     if err != nil {
     	log.Fatal(err)
     }

     fmt.Printf("id: %s\n", spendLimit.ID)
     fmt.Printf("amount: %s\n", spendLimit.Amount)
     ```

     ```java Java
     import com.anthropic.models.beta.organization.spendlimits.BetaSpendLimitPeriod;
     import com.anthropic.models.beta.organization.spendlimits.SpendLimitSetParams;

     void main() {
         AnthropicClient client = AnthropicOkHttpClient.fromEnv();

         var params = SpendLimitSetParams.builder()
             .userScope("user_01AbCdEfGh")
             .amount("500000")
             .period(BetaSpendLimitPeriod.MONTHLY)
             .build();
         var spendLimit = client.beta().organization().spendLimits().set(params);

         IO.println("id: " + spendLimit.id());
         IO.println("amount: " + spendLimit.amount().orElseThrow());
     }
     ```

     ```php PHP
     use Anthropic\Beta\Organization\SpendLimits\SpendLimitUserScope;
     // ...

     $client = new Client();

     $spendLimit = $client->beta->organization->spendLimits->set(
         scope: SpendLimitUserScope::with(userID: 'user_01AbCdEfGh'),
         amount: '500000',
         period: 'monthly',
     );

     echo "id: {$spendLimit->id}\n";
     echo "amount: {$spendLimit->amount}\n";
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     spend_limit = client.beta.organization.spend_limits.set(
       scope: { type: :user, user_id: "user_01AbCdEfGh" },
       amount: "500000",
       period: :monthly
     )

     puts "id: #{spend_limit.id}"
     puts "amount: #{spend_limit.amount}"
     ```
   </CodeGroup>

3. 如果响应人员在事件期间需要更广泛的访问权限，请预先配置一个事件响应人员组，其[自定义角色](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#custom-roles)授予该权限，并在事件持续期间将该成员加入其中：

   <CodeGroup>
     ```bash cURL
     curl -X POST "https://api.anthropic.com/v1/organizations/rbac_groups/rbac_group_01UvWxYzAbCdEfGhIjKlMn/members" \
       -H "content-type: application/json" \
       -H "x-api-key: $ANTHROPIC_API_KEY" \
       -H "anthropic-version: 2026-10-21" \
       -d '{"user_id": "user_01AbCdEfGh"}'
     ```

     ```bash CLI
     ant beta:organization:rbac-groups:members add \
       --rbac-group-id rbac_group_01UvWxYzAbCdEfGhIjKlMn \
       --user-id user_01AbCdEfGh
     ```

     ```python Python
     client = anthropic.Anthropic()

     member = client.beta.organization.rbac_groups.members.add(
         "rbac_group_01UvWxYzAbCdEfGhIjKlMn", user_id="user_01AbCdEfGh"
     )

     print(f"user_id: {member.user_id}")
     print(f"rbac_group_id: {member.rbac_group_id}")
     ```

     ```typescript TypeScript
     const client = new Anthropic();

     const member = await client.beta.organization.rbacGroups.members.add(
       "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
       {
         user_id: "user_01AbCdEfGh"
       }
     );

     console.log(`user_id: ${member.user_id}`);
     console.log(`rbac_group_id: ${member.rbac_group_id}`);
     ```

     ```csharp C#
     AnthropicClient client = new();

     var member = await client.Beta.Organization.RbacGroups.Members.Add(
         "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
         new() { UserID = "user_01AbCdEfGh" }
     );

     Console.WriteLine($"user_id: {member.UserID}");
     Console.WriteLine($"rbac_group_id: {member.RbacGroupID}");
     ```

     ```go Go
     client := anthropic.NewClient()

     member, err := client.Beta.Organization.RBACGroups.Members.Add(
     	context.Background(),
     	"rbac_group_01UvWxYzAbCdEfGhIjKlMn",
     	anthropic.BetaOrganizationRBACGroupMemberAddParams{
     		UserID: "user_01AbCdEfGh",
     	},
     )
     if err != nil {
     	log.Fatal(err)
     }

     fmt.Printf("user_id: %s\n", member.UserID)
     fmt.Printf("rbac_group_id: %s\n", member.RBACGroupID)
     ```

     ```java Java
     import com.anthropic.models.beta.organization.rbacgroups.members.MemberAddParams;

     void main() {
         AnthropicClient client = AnthropicOkHttpClient.fromEnv();

         var params = MemberAddParams.builder()
             .userId("user_01AbCdEfGh")
             .build();
         var member = client.beta().organization().rbacGroups().members()
             .add("rbac_group_01UvWxYzAbCdEfGhIjKlMn", params);

         IO.println("user_id: " + member.userId());
         IO.println("rbac_group_id: " + member.rbacGroupId());
     }
     ```

     ```php PHP
     $client = new Client();

     $member = $client->beta->organization->rbacGroups->members->add(
         'rbac_group_01UvWxYzAbCdEfGhIjKlMn',
         userID: 'user_01AbCdEfGh',
     );

     echo "user_id: {$member->userID}\n";
     echo "rbac_group_id: {$member->rbacGroupID}\n";
     ```

     ```ruby Ruby
     client = Anthropic::Client.new

     member = client.beta.organization.rbac_groups.members.add(
       "rbac_group_01UvWxYzAbCdEfGhIjKlMn",
       user_id: "user_01AbCdEfGh"
     )

     puts "user_id: #{member.user_id}"
     puts "rbac_group_id: #{member.rbac_group_id}"
     ```
   </CodeGroup>

   有关组端点，请参阅[用户管理](https://platform.claude.com/docs/zh-CN/manage-claude/user-management#groups)。

4. 当您的事件系统将事件标记为已关闭时，回滚这两项更改：恢复您在步骤 1 中记录的支出限额（如果该成员原本没有覆盖值，则使用 `DELETE /v1/organizations/spend_limits/{spend_limit_id}` 删除覆盖值），并使用 `DELETE /v1/organizations/rbac_groups/{rbac_group_id}/members/{user_id}` 将该成员从组中移除。

## 常见问题

### 直接设置支出限额会解决成员的待处理提高请求吗？

不会。`POST /v1/organizations/spend_limits` 会写入覆盖值，但不会触及待处理请求。请使用 `POST /v1/organizations/spend_limit_increase_requests/{id}/approve` 在一次调用中解决请求并写入覆盖值。

### 删除按用户的覆盖值后会发生什么？

该成员会回退到他们从层级中继承的任何值：其组、席位层级或组织默认值。如果任何级别都不存在默认值，则该成员无限制。

### 我可以通过此 API 设置席位层级或组织范围的默认值吗？

不可以。只有按用户的覆盖值可以通过此 API 写入。席位层级、组和组织级别的默认值在 claude.ai 组织设置中配置。

### 为什么活跃成员的 `period_to_date_spend` 有时显示为 `"0"`？

支出读数可能暂时不可用，在这种情况下该字段显示为 `"0"` 而不是报错。请将其视为参考信息。

## 另请参阅

<CardGroup cols={2}>
  <Card title="支出限额 API 参考" href="https://platform.claude.com/docs/zh-CN/api/beta/organization/spend_limits">
    每个支出限额 API 端点的生成请求和响应模式。
  </Card>

  <Card title="支出限额提高请求 API 参考" href="https://platform.claude.com/docs/zh-CN/api/beta/organization/spend_limits/increase_requests">
    提高请求端点的生成请求和响应模式。
  </Card>

  <Card title="用量和成本分析 API" href="https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api">
    跟踪您组织的令牌用量和成本。
  </Card>
</CardGroup>
