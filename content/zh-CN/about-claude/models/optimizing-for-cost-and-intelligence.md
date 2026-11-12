---
title: 优化成本与智能
url: https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence
description: 在 Claude Platform 上平衡成本与智能，附有提示缓存、effort、模型选择、输出上限和多模型策略的实测结果。
---

能力最强的模型在大规模使用时可能过于昂贵，而成本最低的模型可能在质量上有所欠缺。Claude Platform 让您可以直接控制这种权衡。您可以为每个请求选择模型、"effort"（努力程度）级别和架构，从而将工作负载放置在"cost-to-intelligence frontier"（成本-智能前沿）上几乎任何位置。

成本与智能通常被描绘为一条前沿曲线，其中一方以另一方为代价。本页的第一组杠杆通过在不影响质量的情况下削减成本，将工作负载推向这条前沿；只有第二组杠杆是沿着前沿移动：

<Frame>
  ![cost-to-intelligence frontier（成本-智能前沿）示意图：一个箭头在相同质量下削减支出，另一个箭头以质量换取成本](https://platform.claude.com/docs/zh-CN/images/cost-intel/frontier.svg)
</Frame>

这些杠杆分为两类：

* **免费收益**在不影响质量的情况下削减支出："prompt caching"（提示缓存）、"token hygiene"（令牌精简）、针对您正在运行的模型进行提示审计，以及对可以等待最多 24 小时的工作使用享受 50% 折扣的[批处理](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing)。
* **权衡**以成本换取智能：模型选择、努力程度、输出上限和会话预算、已用时间计时，以及多模型架构。

[工作区支出限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#setting-lower-limits-for-workspaces)是这两类杠杆的兜底：参见[设置预算和输出上限](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#set-budgets-and-output-caps)。

每个杠杆都附有实测结果以及何时值得使用的规则。在 Anthropic 的测量中，提示缓存通常是最大的杠杆。多模型杠杆的适用范围较窄；第二个模型以两种形态进行了测量：顾问（advisor）和编排器（orchestrator）。

## 从这里开始

将您的情况与某一行对应。

| 您的情况                             | 这样做                                                                               | 参见                                                                                                                                                                                                                                                                         |
| -------------------------------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 任何工作负载、任何模型                      | 开启提示缓存并精简不需要的令牌；两者都是免费的                                                           | [缓存重复的上下文](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#cache-repeated-context) · [精简令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens) |
| 轮次之间有人在等待                        | 统计请求之间的间隔，然后根据该部分的表格选择缓存时长                                                        | [选择缓存时长](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#pick-the-cache-duration)                                                                                                                                          |
| 成本过高；质量尚可                        | 在您当前使用的模型上尝试更低的努力程度                                                               | [调整努力程度](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#tune-effort)                                                                                                                                                      |
| 您没有使用最新模型                        | 升级；在该部分的编码基准测试中，每个较新的模型解决的任务至少与前一个模型一样多，且每个已解决任务的成本通常更低                           | [升级模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#upgrade-the-model)                                                                                                                                                  |
| 您正在选择或切换模型                       | 按每个已完成任务的成本进行比较，而不是按每个令牌                                                          | [比较模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#compare-models-on-cost-per-task)                                                                                                                                    |
| 输出可检查的大批量工作                      | 先尝试 Claude Haiku 5.5                                                              | [比较模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#compare-models-on-cost-per-task)                                                                                                                                    |
| 质量不够好                            | 如果您降低了努力程度，请恢复它；否则尝试以默认努力程度使用更高一级的模型                                              | [调整努力程度](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#tune-effort) · [比较模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#compare-models-on-cost-per-task)            |
| 尝试以 `stop_reason: max_tokens` 结束 | 将 `max_tokens` 提高到 64,000：在 Anthropic 的测试中，只有约 18,000 个中的 7 个轮次更长，且每个已解决任务的成本没有上升 | [设置预算](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#set-budgets-and-output-caps)                                                                                                                                        |
| 您可以检查输出（测试、验证器）                  | 以 `low` 努力程度运行，并以 `high` 重新运行失败的任务：在 Anthropic 的测试中，成本比全部使用 `high` 低约 40%，通过率相同   | [重新运行失败任务](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#re-run-failures-at-higher-effort)                                                                                                                               |
| 智能体循环中有少数运行成本极高                  | 设置 Claude Managed Agents 会话预算和工作区支出限制                                             | [设置预算](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#set-budgets-and-output-caps)                                                                                                                                        |
| 您希望智能体运行更快完成                     | 告诉模型时间很重要，并向其展示已用时间：在 Anthropic 的测试中用时减少 7% 到 47%，代价是得分降低几个百分点                    | [向模型显示已用时间](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#show-the-model-elapsed-time)                                                                                                                                   |
| 低成本模型仅在困难决策上停滞                   | 尝试顾问。只有当您的模型咨询它时它才能提供帮助；先单独以低努力程度为其模型定价，并测量咨询率                                    | [顾问策略](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#advisor-strategy-escalate-hard-decisions)                                                                                                                           |
| 工作超出一个上下文窗口                      | 拆分工作，并将各部分交给较低成本的模型                                                               | [编排器策略](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#orchestrator-strategy-delegate-bulk-work)                                                                                                                          |

这些结果来自 Anthropic 内部测量（[引用的基准测试](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)），仅供方向性参考，并非保证，因此请使用[四步方法](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#measure-on-your-own-workload)在您自己的工作负载上进行测量。

## 在不损失质量的情况下削减支出

提示缓存、令牌精简、批处理以及针对当前模型的提示审计都能降低您的支出而不降低输出质量。有两点需要注意：批处理以延迟换取折扣，而上下文编辑这一令牌精简杠杆在本节测量的短运行中花费超过了它节省的，在长运行中则没有带来任何变化。

### 缓存重复的上下文

#### 为什么缓存排在首位

在使用任何其他杠杆之前先开启[提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching)，因为默认情况下，智能体任务的每个轮次都会重新发送整个不断增长的对话：系统提示、工具定义以及之前的每个轮次。

* 一个 40 轮的任务会将其第一轮发送 40 次，因此任务成本大致随轮次数的平方增长。
* 缓存并不会停止重新发送，但每次重新发送的成本约为原来的十分之一（在 Claude Opus 5.5 和 Claude Sonnet 5.5 上为二十分之一，在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1 和 Claude Mythos 5.1 上为四十分之一），并且处理速度更快。
* 前缀按[缓存读取费率](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#pricing)而非输入价格计费，每个轮次只需为新增内容支付 1.25 倍的缓存写入费率。

处于 beta 阶段的[消息线程](https://platform.claude.com/docs/zh-CN/build-with-claude/message-threads)会在线程存续期间停止重新发送，线程一直存续到其最后一次请求后 5 分钟为止。它们降低的是首个令牌时间，而不是成本：模型仍会读取整个对话，因此请在每个请求上保留 `cache_control`。线程不支持压缩，也不支持大多数上下文编辑。

**良好的表现是什么样的。**

* 在一整天的真实流量中，智能体循环的输入中位数有 84% 从缓存读取，其中排名前 10% 的循环（无论是否为编码任务）读取比例达到 94% 或更高<sup>[17](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。
* 在任务深处，构建良好的循环只有不到 1% 的输入需要支付全价。
* 如果低于约 80%，请查找破坏缓存的因素（参见[什么会破坏缓存](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#what-breaks-the-cache)）。

Anthropic 对 DeepResearch Bench II<sup>[7](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 运行在享有和不享有缓存折扣的情况下分别进行了定价：

<Frame>
  ![图表，DeepResearch Bench II：缓存将 Fable 5.1 的每任务成本从 $40.58 USD 降至 $5.64 USD；Sonnet 5.5 从 $5.13 USD 降至 $1.05 USD](https://platform.claude.com/docs/zh-CN/images/cost-intel/caching-repriced.svg)
</Frame>

缓存的默认生命周期为 5 分钟，而智能体循环的轮次间隔只有几秒，因此折扣适用于每个轮次的大部分令牌。缓存图表中的运行有 89% 到 93% 的输入令牌从缓存读取。

以同样的方式对其他运行记录的令牌数进行定价（每个模型均使用其默认努力程度），可以得出无缓存时每个任务的平均令牌成本：

| 基准测试                                                                                                                                                                                                                                                                                | 模型                | 无缓存时          |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | ------------- |
| SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>                                                                                                                                                   | Claude Opus 5.5   | 有缓存时成本的 3.3 倍 |
| SWE-bench Pro                                                                                                                                                                                                                                                                       | Claude Sonnet 5.5 | 有缓存时成本的 3.1 倍 |
| Claude Managed Agents 任务集<sup>[25](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>，其会话较短                                                                                                                                | Claude Haiku 5.5  | 有缓存时成本的 1.7 倍 |
| 一个对错误报告进行分诊的小型智能体（[精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)）<sup>[16](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> | Claude Sonnet 5.5 | 有缓存时成本的 7.3 倍 |

节省幅度随任务所需的轮次数而变化，因为较短的循环重复读取的内容更少。

<Note>
  在 Messages API 上使用 Claude Haiku 5.5 时，请在任务之间、在提示超过 100,000 个令牌之前[压缩长对话](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#manage-the-context-lifecycle)。超过该值后，请求的每个令牌（包括输出）都要支付 5 倍的费用。缓存读取和写入都计入这 100,000 个令牌（请参阅[长上下文定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)）。

  如果您改用[在令牌阈值处进行压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold#parameters)，请将其 `trigger` 设置为略低于 100,000 减去提示在两次请求之间的最大增长量（默认值为 150,000）。在 Anthropic 使用 10,000 令牌轮次的测试中，90,000 的触发值使每个请求的成本比默认值低 70%<sup>[18](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。
</Note>

#### 选择缓存时长

如果您的循环在轮次之间需要等待人工操作，那么当大约每 20 个轮次中有 1 个是在 5 分钟到 1 小时之间的暂停之后、且很少有间隔超过 1 小时时，请使用 [1 小时缓存时长](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#1-hour-cache-duration)。在 Claude Fable 5.2、Claude Fable 5.1、Claude Opus 5.5 和 Claude Sonnet 5.5 上，保持 5 分钟缓存预热的成本可能更低。

1 小时时长的写入成本更高（输入价格的 2 倍，而不是 1.25 倍）。无论哪种时长，一次未命中都会按写入价格而非读取价格对整个前缀计费。

请在对话开始时选择缓存时长。在 Anthropic 的测试中，对于一个以 5 分钟时长缓存的对话，以 1 小时时长添加的第一个轮次重新写入了整个提示一次：在 Claude Opus 5.5 上，100,000 个令牌时为 $0.80 USD，而读取缓存的轮次输入成本为 $0.02 USD<sup>[16](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。

Anthropic 测量了一个对错误报告进行分诊的小型智能体（[精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)），在部分轮次之前插入暂停以模拟人工延迟<sup>[16](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。在所测量的每种设置下，准确率都保持在运行间噪声范围内。Anthropic 还测量了保持 5 分钟缓存预热的额外请求（保活请求）。

要做出决定，请统计对话中连续请求之间的间隔，并在下表中找到对应的行。指明您所用模型的行优先于"模型"列为"任意"的行：

| 您的请求间隔                                           | 模型                                  | 使用                      |
| ------------------------------------------------ | ----------------------------------- | ----------------------- |
| 轮次间隔只有几秒                                         | 任意                                  | 5 分钟默认值                 |
| 大约每 20 个间隔中有 1 个或更多落在 5 分钟到 1 小时之间，且很少有间隔超过 1 小时 | 任意                                  | 1 小时时长                  |
| 超过 1 小时的间隔很常见：超过 5 分钟的暂停中约 60% 或更多也超过了 1 小时      | 任意                                  | 5 分钟默认值                 |
| 每 20 个间隔中只有 1 或 2 个落在 5 分钟到 1 小时之间，且没有一个超过约半小时   | Claude Opus 5.5 和 Claude Sonnet 5.5 | 保持预热的 5 分钟缓存，而不是 1 小时时长 |
| 每个轮次之前都有超过 5 分钟的暂停，且很少有暂停超过 1 小时                 | Claude Opus 5.5 和 Claude Sonnet 5.5 | 1 小时时长                  |
| 暂停持续数分钟                                          | Claude Fable 5.1 和 Claude Fable 5.2 | 保持预热的 5 分钟缓存，而不是 1 小时时长 |
| 暂停接近 1 小时，且很少有暂停超过 1 小时                          | Claude Fable 5.1 和 Claude Fable 5.2 | 1 小时时长                  |

在 Claude Haiku 5.5 上，请使用"任意"行：它以输入价格的标准 0.1 倍读取缓存，是 Claude Opus 5.5 和 Claude Sonnet 5.5 的 0.05 倍的两倍，因此根据标价，每个保活请求相对于输入价格的成本是这两个模型上的两倍。在另一项测试中，`max_tokens: 0` 请求刷新了其缓存（参考文献 16）。

**逐行说明原因。**

* **任意模型。**

  * **轮次间隔只有几秒。** 没有暂停：在 Claude Sonnet 5.5 上，默认值的成本比 1 小时设置低约 19%，在 Claude Opus 5.5 上低约 15% 到 18%。
  * **大约每 20 个间隔中有 1 个或更多。** 根据令牌数计算，在 Claude Sonnet 5.5 和 Claude Opus 5.5 上的盈亏平衡点约为每 30 个轮次中有 1 个。
  * **超过 1 小时的间隔很常见。** 在任何模型上，超过 1 小时的间隔都会使两种时长的缓存过期，而 1 小时设置随后会以其更高的写入价格重新写入前缀。

* **Claude Opus 5.5 和 Claude Sonnet 5.5**，其缓存读取成本为输入价格的 0.05 倍。

  * **每 20 个间隔中只有 1 或 2 个。** 在默认努力程度下，保活请求的成本在 Claude Opus 5.5 上比 1 小时时长低 8% 到 13%，在 Claude Sonnet 5.5 上低 8% 到 16%。
  * **每个轮次之前都有暂停。** 在 Claude Opus 5.5 和 Claude Sonnet 5.5 上，暂停 6 分钟时保活请求的成本比 1 小时时长高几个百分点，暂停 45 分钟时高出 50% 以上。

* **Claude Fable 5.1 和 Claude Fable 5.2**，其[缓存读取](https://platform.claude.com/docs/zh-CN/about-claude/pricing#prompt-caching)成本为输入价格的 0.025 倍。

  * **暂停持续数分钟。** 在 Claude Fable 5.2 上，无论是 10% 的轮次有暂停，还是每个轮次之前都有 6 分钟暂停，保活请求<sup>[19](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>的成本都比 1 小时缓存低约 4.5% 到 19%。
  * **暂停接近 1 小时。** 在 Claude Fable 5.2 上，只有在每个轮次之前都有 45 分钟暂停时，1 小时缓存才胜出，在包含 5 个错误报告的会话中节省约 $0.12 USD<sup>[19](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。

下图绘制了 Claude Sonnet 5.5 上每个会话的成本与暂停轮次占比之间的关系：

<Frame>
  ![折线图：按暂停后轮次占比划分的每个分诊会话成本；超过约每 30 个轮次中 1 个后，1 小时缓存更便宜](https://platform.claude.com/docs/zh-CN/images/cost-intel/cache-ttl-sonnet-5-5.svg)
</Frame>

下图在 Claude Fable 5.1、Claude Opus 5.5 和 Claude Sonnet 5.5 上加入了通过保活请求保持预热的 5 分钟缓存：

<Frame>
  ![成本图表：在所绘制的范围内，保活在 Fable 5.1 上优于 1 小时缓存；在 Opus 5.5 和 Sonnet 5.5 上仅在少数轮次暂停时更优](https://platform.claude.com/docs/zh-CN/images/cost-intel/cache-keepalive-sonnet-5-5.svg)
</Frame>

**需要了解的内容。**

* "每 20 个中 1 个"的规则相对于约每 30 个轮次中 1 个的盈亏平衡点留有余量。在每 20 个轮次中 1 个时，5 分钟和 1 小时设置在 Claude Sonnet 5.5 上的成本大致相同，处于运行间差异范围内；在每 10 个轮次中 1 个时，5 分钟设置的成本高出 54%（参考文献 16）。
* 在 Claude Sonnet 5.5 和 Claude Opus 5.5 以外的模型上，价格使盈亏平衡点处于相同范围：所有当前模型都使用相同的缓存写入倍数，并且除 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5 和 Claude Sonnet 5.5 之外的所有模型都使用相同的读取价格。
* Claude Opus 5.5 和 Claude Sonnet 5.5 的保活数据来自以 `max_tokens: 1` 发送的请求。在 Claude Fable 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 上，`max_tokens: 0` 请求也刷新了现有缓存条目（参考文献 16）。

要保持缓存预热：

1. 取上一个请求，将 `max_tokens` 设置为 0，如果设置了 `stream` 则将其删除。不要更改前缀的任何一个字节。
2. 将请求的标头与其正文一起重新发送。如果没有原始请求的 `anthropic-beta` 标头，重放正文中受 beta 限制的字段将被拒绝。
3. 在上一个请求开始后的 4 分钟内发送，之后每 4 分钟发送一次，直到下一个轮次到来或会话结束。从请求开始时计时，因为[缓存的 5 分钟生命周期](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#how-prompt-caching-works)从那时开始计算。

这就是[预热请求](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#pre-warming-the-cache)：它会刷新缓存的生命周期，不返回任何输出，成本仅略高于缓存读取。一旦缓存过期，下一个请求（无论是否为保活请求）都会以缓存写入价格重新写入前缀<sup>[16](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。

在以下情况下，`max_tokens: 0` 请求会被拒绝：

* 请求设置了 `thinking.type: "enabled"`（Claude Fable 5.1 和 Claude Fable 5.2 上默认的自适应思考没有问题）、结构化输出或强制工具选择（[其限制](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#limitations)）。对于这些工作负载，请改为购买 1 小时时长。
* 请求带有顶层 `compaction` 参数，因此不要将来自[按需压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-on-demand)的压缩请求作为保活请求重新发送。

<CodeGroup exclude="shell:CLI, python, typescript, csharp, go, java, php, ruby">
  ```bash cURL
  # 在上一个请求开始后的 4 分钟内（生成所用时间计入
  # 缓存的生命周期），重新发送该请求，并将 max_tokens 设为
  # 0，同时去掉 stream（max_tokens: 0 的请求无法流式传输）。发送与
  # 原始请求相同的请求头，包括任何 anthropic-beta 请求头。
  jq '.max_tokens = 0 | del(.stream)' last_request.json | \
    curl https://api.anthropic.com/v1/messages \
      -H "x-api-key: $ANTHROPIC_API_KEY" \
      -H "anthropic-version: 2026-10-21" \
      -H "content-type: application/json" \
      --data-binary @-
  ```
</CodeGroup>

#### 开启缓存

设置工作量很小。[自动缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#automatic-caching)会为您放置断点；或者，Claude Code 附带的 [Claude API 技能](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/claude-api-skill)可以通过一条提示为现有集成添加缓存：

让 Claude Code 为您添加缓存

```text
/claude-api 为此集成添加提示缓存
```

在生成这些测量结果的代码上，该 skill 在系统块上放置了一个断点，并在最近的两个用户轮次上各放置了一个断点；[显式缓存断点](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#explicit-cache-breakpoints)说明了应将它们放在何处。

#### 什么会破坏缓存

将每个请求特有的文本（例如状态行）放在最新用户轮次的末尾、最后一个缓存断点之后。在后续请求中，保持每个较早的轮次与您发送时完全一致，其中仍保留其旧的状态行。在自然的断点处进行会使缓存失效的更改，例如压缩后的第一个请求。在运行[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)的模型上，即使在这样的断点处，编辑系统提示或 `tools` 的请求也可能收到 400 错误。在您的平台支持的情况下，请改为在 `messages` 中发送这些更改（参见[在不编辑前缀的情况下进行更改](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#replace-prefix-edits)）。缓存是按顺序（工具、系统提示、消息）对请求进行的逐字节前缀匹配，因此任何位置的更改都会使其后的所有内容失效。

当必须在对话中途更改某些内容时，请在模型支持的情况下，以保持缓存的形式发送。Anthropic 在 Claude Fable 5.1、Claude Opus 5.5 和 Claude Sonnet 5.5 上，针对一个约 100,000 个令牌、包含模型首次回复及其思考内容的已缓存对话，以两种方式测试了每项更改。编辑系统提示的成本约为以对话中途系统消息发送相同文本的 25 到 50 倍。在顶层更改努力程度的成本是在消息中更改的 20 到 40 倍<sup>[18](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>：

<Frame>
  ![条形图：编辑系统提示最高 $1.26 USD，顶层努力程度更改最高 $1.02 USD，两者在消息中进行时最高 $0.026 USD](https://platform.claude.com/docs/zh-CN/images/cost-intel/cache-break-cost.svg)
</Frame>

当某项更改使缓存失效时，前缀中受其影响的部分会被重新写入，而不是被读取。写入 5 分钟缓存的成本为输入价格的 1.25 倍，而缓存读取的成本仅为输入价格的一小部分。按标价计算，在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1 和 Claude Mythos 5.1 上，这次写入的成本是一次读取的 50 倍，因为在这些模型上缓存读取的成本为输入价格的 0.025 倍。在 Claude Opus 5.5 和 Claude Sonnet 5.5 上（0.05 倍），这次写入的成本是一次读取的 25 倍；在 Claude Haiku 5.5 上（0.1 倍）为 12.5 倍。图表中的数据来自 Messages API。

编辑系统提示或 `tools` 的请求也可能直接失败。在运行[前缀检查](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)的模型上，进行其中任一编辑并回传较早思考块的请求可能会收到 400 错误<sup>[18](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。在 Anthropic 的另一项测试中，等待压缩并不能避免该错误：在 Claude Opus 5.5 和 Claude Fable 5.2 上，这样的请求作为在令牌阈值处压缩后的第一个请求时仍然收到了该错误<sup>[18](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。在 Claude API 上，默认情况下，如果模型是 Claude Fable 5.2、Claude Mythos 5.2 或 Claude Mythos 5.1，未通过检查的请求在每个账户上都会收到该错误。如果模型是 Claude Fable 5.1、Claude Opus 5.5、Claude Sonnet 5.5 或 Claude Haiku 5.5，则该请求在某些账户上会收到该错误，在其他账户上则不会。有关按模型和账户划分的结果，请参阅[未设置 `prefix_mismatch_behavior` 时](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#enforcement)。

**应该怎么做。**

* 不要在稳定前缀之前放置任何随请求变化的内容，例如时间戳或队列位置：这会使每个请求都变成完整的缓存写入。在 Anthropic 的测试中，系统提示开头的一行 25 个令牌的状态行使每个请求都写入整个对话；将其移到最新用户轮次的末尾后，缓存读取保持不变<sup>[18](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。
* [提示缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#what-invalidates-the-cache)页面列出了会使缓存失效的更改，例如更改 [`effort`](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)、思考配置、[`speed`](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode) 设置、系统提示或工具定义。更改输出格式会使该对话的缓存失效（参见[结构化输出](https://platform.claude.com/docs/zh-CN/build-with-claude/structured-outputs#prompt-modification-and-token-costs)）。
* 请查看哪些模型支持每种形式：[对话中途的系统消息](https://platform.claude.com/docs/zh-CN/build-with-claude/mid-conversation-system-messages)，以及处于 beta 阶段的[按消息设置努力程度](https://platform.claude.com/docs/zh-CN/build-with-claude/effort-per-message)。
* 在进行会使缓存失效的更改后，确认缓存读取没有下降；如果下降了，[缓存诊断](https://platform.claude.com/docs/zh-CN/build-with-claude/cache-diagnostics)会显示前缀在何处出现分歧。

Anthropic 在一个早期模型上、针对[精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)中所述问题分诊智能体的长会话，测量了进行此类更改的时机<sup>[18](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。进行更改的会话以低努力程度开始，然后切换到默认努力程度并添加一个工具。在会话中途进行这两项更改的成本最高。在触发压缩的请求上进行更改的成本几乎一样高，因为该请求仍携带整个对话，并再次将其写入缓存。在下一个请求（即压缩后的第一个请求）上进行更改的成本最低。准确率在运行间噪声范围内。该模型不运行前缀检查。在运行前缀检查的模型上，请改为在消息中添加工具：参见[使用 `tool_addition` 和 `tool_removal` 添加或删除工具](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#tool-changes)，该功能处于 beta 阶段。下图还显示了没有任何更改的会话。它们并不是同等条件下的基线，因为它们全程以默认努力程度运行，且其中两个会话进行了第二次压缩：

<Frame>
  ![条形图，每个分诊会话的成本：无更改 $0.81 USD，会话中途 $0.95 USD，compaction（压缩）时 $0.92 USD，之后 $0.75 USD](https://platform.claude.com/docs/zh-CN/images/cost-intel/compaction-timing.svg)
</Frame>

### 精简输入和上下文令牌

大多数智能体请求都携带着对答案毫无影响的令牌。精简这些令牌很少会损失输出质量，不过在测量中，并非这里的每个杠杆都节省了成本。有两个方面值得关注：

* **输入精简。**

  * Web fetch 工具中的[动态过滤](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/web-fetch-tool#dynamic-filtering)可将样板内容排除在获取的页面之外。
  * [图像缩放](https://platform.claude.com/docs/zh-CN/build-with-claude/vision#evaluate-image-size)可将视觉输入调整到合适的尺寸。
  * [带延迟加载的工具搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)仅在需要时加载工具定义（参见[延迟加载未使用的工具定义](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#defer-unused-tool-definitions)）。
  * [编程式工具调用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/programmatic-tool-calling)让 Claude 通过代码运行多个工具调用，只有过滤后的结果才会进入上下文；其文档报告称，在智能体搜索基准测试中输入令牌减少了 24%，且得分更高。
  * [管理工具上下文](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/manage-tool-context)比较了工具搜索、程序化工具调用、提示缓存和上下文编辑。

* **上下文生命周期。** [上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)会清除过时的工具结果，而["automatic compaction"（自动压缩）](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold)及其阈值可防止长循环一直携带全部历史记录。

这些杠杆会与缓存以及彼此相互作用，因此请根据净效果来评判它们，并使用[缓存诊断](https://platform.claude.com/docs/zh-CN/build-with-claude/cache-diagnostics)确认您的缓存前缀在每次更改后仍然有效。

在针对早期模型的测量中<sup>[28](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>，Anthropic 在一个问题分诊智能体上运行了这些杠杆，该智能体处理来自公共代码仓库的 20 个带截图的真实错误报告；还在同一工作的一个更长变体上运行，其令牌数为前者的 2.6 倍。在开启缓存的情况下，输入精简（图像缩放和工具搜索）进一步降低了两次运行的成本。

#### 延迟加载未使用的工具定义

附加到请求的每个工具定义在每个轮次都是输入，而几个 MCP 服务器加起来就可能有数百个工具定义。

在针对早期模型的测量中<sup>[27](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>，Anthropic 运行了分诊智能体，使用其自身的两个工具加上来自公共 MCP 服务器的真实工具定义目录，总计最多 502 个工具，分别采用全部加载，或将额外工具标记为 `defer_loading` 并置于[工具搜索](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool)之后的方式：

<Frame>
  ![折线图：加载所有工具时，运行成本在 502 个工具时从 $0.55 USD 上升到 $1.02 USD；tool search（工具搜索）使其保持在 $0.56 USD](https://platform.claude.com/docs/zh-CN/images/cost-intel/tool-search.svg)
</Frame>

* 加载所有定义时，运行成本随目录增长而上升，与每个请求上的 schema 令牌数同步变化。
* 使用工具搜索时，运行成本在每种目录规模下都保持平稳。
* 两种方式的准确率处于相同范围，且模型从未调用错误的工具，因此在这种规模下，未使用的定义影响的是成本，而不是正确性。
* [保持最常用的 3 到 5 个工具处于加载状态](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/tool-search-tool#deferred-tool-loading)，这样 Claude 无需先搜索即可调用它们。
* 在同一早期模型上，延迟加载一个公共 GitHub MCP 服务器的工具集（在 [MCP 连接器](https://platform.claude.com/docs/zh-CN/agents-and-tools/mcp-connector)上使用 `default_config: {defer_loading: true}`）在相同准确率下降低了运行成本。在请求中没有工具搜索工具的情况下，延迟加载工具集时，没有任何运行调用该服务器的工具。

#### 将数据文件排除在提示之外

当模型需要对表格进行计算时，请使用 [Files API](https://platform.claude.com/docs/zh-CN/build-with-claude/files) 上传表格，并让模型通过[代码执行](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/code-execution-tool)查询它，而不是将其粘贴到提示中。Anthropic 针对一个 1,862 行的公共 CSV 提出了 25 个聚合问题<sup>[15](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>（求和、过滤计数、分组以及一个日期过滤），答案由 pandas 计算：

<Frame>
  ![散点图：上传文件并使用代码执行，25 题全对，成本 $0.40 USD；粘贴到提示中，25 题对 7 题，成本 $5.01 USD](https://platform.claude.com/docs/zh-CN/images/cost-intel/data-files.svg)
</Frame>

粘贴到提示中时，表格在每个请求上约占 91,000 个输入令牌，Claude Sonnet 5 正确回答了 25 个问题中的 7 个。上传并使用代码执行时，它回答对了全部 25 个问题，运行成本约为前者的十二分之一。Claude Opus 5 也表现出相同的模式。两种方法都没有使用提示缓存。

#### 管理上下文生命周期

上下文杠杆只有在会话足够长、确实需要它们时才会带来回报：

<Frame>
  ![按运行长度划分的柱状图：上下文编辑在短运行上增加 73% 成本；在长运行上压缩节省 32%，修剪节省 39%](https://platform.claude.com/docs/zh-CN/images/cost-intel/hygiene.svg)
</Frame>

Anthropic 在一个早期模型上测量了这些杠杆<sup>[28](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>：

* 在这些运行中，修剪是问题分诊智能体自身代码中的一个简短函数：在每个任务边界处，它将每个超过 2,000 个字符的搜索结果替换为一行摘要。
* 在 20 个问题的运行中，这些杠杆几乎没有节省，而上下文编辑使运行成本更高。
* 在长运行中，修剪和压缩都降低了成本，而上下文编辑没有带来任何变化。
* 如果您使用上下文编辑，请[分几次大批量清除](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing#context-editing-and-prompt-caching)，因为每次清除都会产生一次缓存写入的费用。

<Note>
  不要在 Claude Fable 5.2、Claude Mythos 5.2、Claude Fable 5.1、Claude Mythos 5.1、Claude Opus 5.5、Claude Sonnet 5.5 或 Claude Haiku 5.5 上使用修剪。在这些模型上，如果您的应用程序重写较早的工具结果，会[使该结果之后的所有思考块失效](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#prefix-check)，并且请求[可能以 400 错误失败](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#enforcement)。该修剪是在一个不具备[保留思考](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking)的早期模型上测量的。请改用[压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction)：在同一测试中，它也降低了长运行的成本。压缩和[上下文编辑](https://platform.claude.com/docs/zh-CN/build-with-claude/context-editing)都在服务器端运行，因此两者都不会使思考块失效。[按返回时的原样回传助手轮次](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#append-assistant-turns-exactly-as-returned)：丢弃其中的思考块会在每个轮次中消耗输出令牌，因为 Claude 会[重新生成推理过程](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#drop-block-cost)。
</Note>

### 批处理可以等待的工作

将所有无人等待的请求通过 [Batch API](https://platform.claude.com/docs/zh-CN/build-with-claude/batch-processing) 路由，其余请求保留交互式路径。Batch API 对请求的每个令牌（包括缓存令牌）提供 50% 的折扣，代价是结果会在 24 小时内的任意时间返回。

* 对于无人值守的智能体工作，批处理是仅次于缓存的第二大免费杠杆：评估运行、回填以及定时作业，例如定期运行[令牌精简测量](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)中的问题分诊智能体。
* 批处理可以与本页上的大部分内容结合使用，但不能与[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)、消息线程或 Claude Managed Agents 会话结合使用（参见 [Claude Managed Agents 定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#claude-managed-agents-pricing)）。

### 针对当前模型审计提示

针对您当前运行的模型审计提示，并在每次更换模型时再次审计：这是一项免费收益。

* 每一代模型对提示的响应方式都不同，因此提示中会积累为您已不再使用的模型编写的文本。
* 常见情况是为弥补旧模型不足而添加的过于具体的指令："验证两次"、"尽可能详尽"、强制性的分步流程，或手工编写的推理草稿区。
* 较新的模型会逐字遵循这些指令，产生额外的工具调用轮次和额外的文字输出，您要为此付费，而准确率没有任何提升。

审计只需一条命令。Claude Code 附带的 [Claude API 技能](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/claude-api-skill)有一个 `prompt-audit` 命令，它会读取项目的提示和请求代码，并报告哪些内容是为其他模型编写的：

让 Claude Code 为您审计提示

```text
/claude-api prompt-audit
```

以下简短摘录展示了该命令针对包含这些模式的客服台提示和请求代码运行的情况：

```text wrap
$ claude
> /claude-api prompt-audit

10 findings. The three that matter most:

1. budget_tokens returns a 400 on Opus 5 (support_agent.py:10). This code
   does not run against its own target model. Everything else here is
   quality; this one is an outage.
2. The prompt contains two live contradictions. Line 15 says confirm before
   every refund, line 17 says process every eligible refund immediately.
   Line 19 asks for a complete recap *and* a three-sentence maximum.
3. The reasoning scaffold and the 6-step script fight the model rather than
   steer it. <scratchpad> + "reason step by step" is now a request
   parameter, not prose; the mandatory 6-step procedure plus "investigate
   fully even when the ticket looks simple" forces four tool calls on a
   "where's my package" ticket.
...
-After any refund or escalation, verify twice before submitting: re-fetch
-the order, re-check every figure in your reply against the fresh lookup,
-and review the reply a second time for errors.
+Before submitting a refund or an escalation, re-fetch the order and confirm
+every figure in your reply matches the fresh lookup.
```

该命令随后以 diff 形式提出修改建议（显示了一个 hunk），并列出它有意保留不动的内容：退款期限、语气要求和质量标准。您审查的是一个补丁，而不是一次重写。

Anthropic 在一个客服台评估<sup>[14](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>上测量了其效果，涵盖两次早期模型之间的迁移：

<Frame>
  ![散点图，客服台评估：旧提示在新模型上成本更高；经过 audit（审计）后，它更便宜且同样准确](https://platform.claude.com/docs/zh-CN/images/cost-intel/prompt-audit.svg)
</Frame>

* 从 Claude Opus 4.8 迁移到 Claude Opus 5 时，为旧模型编写的提示在新模型上每张工单的成本更高，而准确率没有变化。
* 对相同的提示运行审计后，Claude Opus 5 比未审计版本更便宜，也更准确，且提升超出了噪声范围。
* 从 Claude Sonnet 4.6 迁移到 Claude Sonnet 5 时，审计在相同准确率下降低了每张工单的成本。

两类过时文本的代价不同：

<Frame>
  ![按遗留模式划分的条形图：被过度遵循的指令消耗金钱；失效的设置和相互矛盾的规则消耗准确率](https://platform.claude.com/docs/zh-CN/images/cost-intel/prompt-audit-patterns.svg)
</Frame>

* 新模型过于字面地遵循的指令会耗费成本：删除"验证两次"降低了 Claude Opus 5 每张工单的成本，删除"尽可能详尽"的降幅几乎相同。
* 不再适合模型的文本则会损害准确率：已弃用的思考设置、相互矛盾的规则，以及与模型自身思考相冲突的手工草稿区，在删除后都恢复了 Claude Opus 5 的准确率。

相同的模式往往也会出现在工具描述和 skill 中，它们同样值得审计。

## 在成本与智能之间权衡

这些杠杆决定了单个模型在成本与智能之间所处的位置：模型选择、努力程度、以更高设置重新运行失败项、模型工作时所受的预算和上限，以及它能否看到已经过去了多少时间。如果您落后一两个模型版本，请先升级（[升级模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#upgrade-the-model)）；否则，请从在当前模型上进行努力程度扫描开始（[调整努力程度](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#tune-effort)）。按成本和能力从低到高，当前模型依次为 Claude Haiku 5.5、Claude Sonnet 5.5、Claude Opus 5.5 和 Claude Fable 5.2（前沿模型）；[模型概览](https://platform.claude.com/docs/zh-CN/models/overview)列出了完整的模型阵容和价格。

### 按每任务成本比较模型

请按每个已完成任务的成本来比较模型，而不是按每个令牌的成本，因为您是为已完成的任务付费。对于大多数智能体工作负载，请从默认努力程度（`medium`）下的 Claude Opus 5.5 开始。

能力更强的模型可以用更少的工作量完成任务，这可以抵消其每令牌溢价的很大一部分。Anthropic 在 SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 子集上测量了这一点（其分数与公开排行榜不可比），并按客户实际计费方式定价：

<Frame>
  ![散点图，SWE-bench Pro：低 effort（努力程度）下的 Claude Opus 5.5 与 Sonnet 5.5 默认设置表现相当，每个已解决任务的成本约高 7%](https://platform.claude.com/docs/zh-CN/images/cost-intel/cost-per-task-sonnet-5-5.svg)
</Frame>

下表列出了结果：

| 基准测试                                                                                                                                                  | 模型                | 努力程度         | 分数                                                   | 成本                                                         |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | ------------ | ---------------------------------------------------- | ---------------------------------------------------------- |
| SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 子集                  | Claude Sonnet 5.5 | `high`（默认）   | 解决了 87.6% 的任务                                        | 每个已解决任务 $0.11 USD                                          |
| SWE-bench Pro 子集                                                                                                                                      | Claude Opus 5.5   | `low`        | 解决了 87.4% 的任务                                        | 每个已解决任务 $0.12 USD                                          |
| SWE-bench Pro 子集                                                                                                                                      | Claude Opus 5.5   | `medium`（默认） | 解决了 92.8% 的任务                                        | 每个已解决任务 $0.22 USD                                          |
| SWE-bench Pro 子集                                                                                                                                      | Claude Fable 5.1  | `high`（默认）   | 解决了 92.3% 的任务（单次运行）                                  | 每个已解决任务 $1.19 USD                                          |
| 内部智能体编码基准测试<sup>[11](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>                      | Claude Opus 5.5   | `medium`（默认） | 86.6%                                                | 每次尝试 $0.84 USD                                             |
| 内部智能体编码基准测试                                                                                                                                           | Claude Fable 5.2  | `medium`     | 88.9%（单次运行）                                          | 每次尝试 $1.34 USD                                             |
| GPQA Diamond<sup>[9](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>                      | Claude Haiku 5.5  | `medium`（默认） | 准确率 85%，而 Claude Opus 5.5 和 Claude Sonnet 5.5 均为 91% | 每个正确答案的成本约为 Claude Opus 5.5 的二十分之一、Claude Sonnet 5.5 的十分之一 |
| Claude Managed Agents 任务集<sup>[25](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>，79 个任务 | Claude Haiku 5.5  | `medium`     | 93% 的尝试通过（单次运行）                                      | 每次通过的尝试约 $0.011 USD                                        |

在 SWE-bench Pro 子集 478 个问题中的 463 个上（即参考解决方案在无网络访问时也能通过的问题，参考文献 3），`high`（其在 Messages API 上的默认值）下的 Claude Fable 5.2 解决了 96.3% 的任务，每个已解决任务 $1.39 USD。同一次默认设置下的 Claude Fable 5.1 运行在这 463 个问题上统计，解决了 92.4%，每个已解决任务 $1.17 USD。在 `low` 下，Fable 5.2 解决了 94.6%，每个已解决任务 $0.39 USD：比默认设置下的 Fable 5.1 解决的任务更多，成本约为其三分之一。在相近的通过率下，Claude Opus 5.5 在这些问题上每个已解决任务的成本仍然更低：默认设置下解决了 93.1%，$0.22 USD；`high` 下为 95.5%，$0.30 USD；`xhigh` 下为 96.5%，$0.75 USD，与 `high` 下的 Fable 5.2 持平，成本略高于其一半。

**应该怎么做。**

* 从默认努力程度下的 Claude Opus 5.5 开始：在 SWE-bench Pro 子集上，它与默认设置下的 Claude Fable 5.1 表现相当（在运行间噪声范围内），而每个已解决任务的成本约为其五分之一。Opus 5.5 和 Fable 5.1 在 SWE-bench Pro 子集上都已基本饱和。对于要求严苛的推理和长周期智能体工作，或者当您在更高努力程度下对 Claude Opus 5.5 的评估仍然不达标时，请使用 Claude Fable 5.2。
* 在另一端，Claude Haiku 5.5 适合输出可检验的大批量和延迟敏感型工作。
* 在您自己的流量上，以每个已完成任务的成本为每个候选方案定价，包括默认努力程度下的 Claude Opus 5.5 以及降低努力程度的前沿模型。Claude Opus 5.5 的每项令牌价格（包括缓存读取）都是 Claude Sonnet 5.5 的两倍，但在 SWE-bench Pro 子集上，`low` 下的 Opus 5.5 解决的任务比例与默认设置下的 Sonnet 5.5 大致相同（在运行间噪声范围内），每个已解决任务的成本约高 7%。
* 请在您自己的工作负载上测量更昂贵的模型能换来什么。在为每个模型都评了分的 44 个 DeepResearch Bench II<sup>[7](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 任务上，在 Messages API 的默认努力程度下，Claude Opus 5.5 的得分比 Claude Sonnet 5.5 高 2.4 分，每任务成本为其 1.5 倍。Claude Fable 5.2 的成本是 Claude Sonnet 5.5 的 4.3 倍，而得分没有可测量的提升；是 Claude Opus 5.5 的 2.9 倍，而得分低 1.9 分。与 Claude Fable 5.1 相比，它在每个努力程度下的成本低 20% 到 61%，得分低 4.7 到 7.0 分。在全部 50 个任务上，Claude Opus 5.5 没有可测量地领先于 Claude Sonnet 5.5：在 Claude Sonnet 5.5 作答的四个生物学任务上，安全防护几乎总是中止它，而被中止的尝试计为失败。
* 为工作负载的尾部定价，而不是中位数：在最难的十分之一任务上比较模型，而不是在典型任务上。

账单由较便宜模型失败的任务决定，因为失败的任务仍然会计费其令牌，然后是重试，再然后是失败在下游造成的任何成本。即使没有任何失败，尾部也是资金流向的地方。在一次 20 个问题的 WideSearch<sup>[1](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 运行中，两个问题占了 43% 的支出：

<Frame>
  ![按成本排序的 20 个 WideSearch 问题的条形图：前两个占支出的 43%，最便宜的一半占 10%](https://platform.claude.com/docs/zh-CN/images/cost-intel/tail.svg)
</Frame>

[多模型策略](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#combine-models)的存在，正是为了将前沿智能用在这一尾部上，而不必为其余部分支付前沿价格。

### 升级模型

如果您落后一两个模型版本，最便宜的杠杆就是模型字符串。在本部分以及[比较模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#compare-models-on-cost-per-task)中的编码基准测试上，每个较新的模型解决的任务至少与前一个模型一样多；大多数模型每个已解决任务的成本更低，而 `high` 下的 Claude Fable 5.2 成本高于 Claude Fable 5.1。在一个研究基准测试上，Claude Fable 5.2 的情况正好相反：它在每个努力程度下的每任务成本都低于 Claude Fable 5.1，得分也更低（[比较模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#compare-models-on-cost-per-task)）。图表绘制了近期 Claude 模型在两个基准测试上的表现，在这两个基准中，智能体单独处理一项书面任务，并由测试检查结果：SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 子集（每个模型都通过相同的测试设置运行）和 Terminal-Bench 3<sup>[20](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。表格列出了每次升级带来的变化：

<Frame>
  ![图表：在 SWE-bench Pro 上，Opus 5.5 和 Sonnet 5.5 对每个已解决任务成本的削减最大；在 Terminal-Bench 3 上，Opus 降至 $28 USD](https://platform.claude.com/docs/zh-CN/images/cost-intel/upgrade-ladder-sonnet-5-5.svg)
</Frame>

| 基准测试                                                                                                                                           | 升级                                    | 努力程度                               | 分数                        | 成本                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- | ---------------------------------- | ------------------------- | ----------------------------------------------------- |
| SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 子集           | Claude Sonnet 4.6 升级到 Claude Sonnet 5 | 默认                                 | 高 5 分                     | 每个已解决任务成本降低 16%                                       |
| SWE-bench Pro 子集                                                                                                                               | Claude Sonnet 5 升级到 Claude Sonnet 5.5 | 默认                                 | 约高 10 分                   | 每个已解决任务成本降低 87%；按 Claude Sonnet 5 的价格计算降低 85%         |
| SWE-bench Pro 子集                                                                                                                               | Claude Opus 5 升级到 Claude Opus 5.5     | 默认：`high` 到 `medium`               | 大致相同：解决的任务从 91.9% 到 92.8% | 每个已解决任务成本降低 78%；按 Claude Opus 5 的价格计算约降低 69%          |
| SWE-bench Pro 子集                                                                                                                               | Claude Fable 5 升级到 Claude Fable 5.1   | 默认                                 | 分数大致相同                    | 每个已解决任务成本降低 43%                                       |
| Terminal-Bench 3<sup>[20](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>          | Claude Opus 4.7 升级到 Claude Opus 4.8   | `high`                             | 解决的任务从 7% 到 15%           | 每个已解决任务从 $183 USD 降至 $63 USD                          |
| Terminal-Bench 3                                                                                                                               | Claude Opus 4.8 升级到 Claude Opus 5     | `high`                             | 解决的任务从 15% 到 41%          | 每个已解决任务从 $63 USD 降至 $28 USD，节省 56%                    |
| Claude Managed Agents 任务集<sup>[25](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> | Claude Haiku 4.5 升级到 Claude Haiku 5.5 | 按服务默认；Claude Haiku 5.5 上为 `medium` | 通过的尝试从 62% 到 93%，每个模型运行一次 | 每次尝试的令牌成本从 $0.041 USD 降至约 $0.010 USD；每次通过的尝试成本约降低 84% |

**何时划算。**

* 节省的幅度取决于工作内容。从 Claude Opus 5 到 Claude Opus 5.5，在 SWE-bench Pro 子集上每个已解决任务的成本降低 78%，在典型工作负载上降低 40%（[推出 Claude Opus 5.5](https://www.anthropic.com/news/claude-opus-5-5)）。从 Claude Sonnet 5 到 Claude Sonnet 5.5，在同一子集上降低 87%，在大多数工作上最多降低 30%（[推出 Claude Sonnet 5.5](https://www.anthropic.com/news/claude-sonnet-5-5)）。
* 当旧模型已经能解决大多数任务时，每次尝试的成本决定了节省：在 SWE-bench Pro 子集上，Claude Opus 5.5 的得分与 Claude Opus 5 大致相同，而每个已解决任务的成本降低 78%。
* 当旧模型大多失败时，通过率决定了节省：在 Terminal-Bench 3<sup>[20](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 上，从 Claude Opus 4.8 升级到 Claude Opus 5 将解决的任务从 15% 提高到 41%，每个已解决任务的成本降低 56%。
* 按每个已解决任务的成本比较，而不是按每个令牌：相同的文本在 Claude 4.7 及更高版本的模型上大约多消耗 30% 的令牌，因此按令牌比较必然会让较新的模型看起来更贵。

### 调整努力程度

在添加第二个模型之前，先在您自己的工作负载上扫描努力程度。较低的努力程度成本更低，往往也更快，而它会牺牲多少准确率取决于任务：在长周期编码中，努力程度换来了准确率；在研究任务中，它在一次测量中换来了很多，在另一次测量中则只换来了一点。[`effort`](https://platform.claude.com/docs/zh-CN/build-with-claude/effort) 参数控制模型进行多少思考、工具调用和自我验证；在 Messages API 上，大多数模型的默认值为 `high`，Claude Opus 5.5 和 Claude Haiku 5.5 的默认值为 `medium`。

下表列出了 Anthropic 使用当前模型在两个基准测试上的测量结果；两个基准测试之间的分数和成本不可比。成本按每个任务计算，而不是像[按每任务成本比较模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#compare-models-on-cost-per-task)中那样按每个已解决任务计算。除非单元格另有说明，每个分数在 SWE-bench Pro 上汇总了两次运行，在 DeepResearch Bench II 上汇总了三次运行：

| 基准测试                                                                                                                                      | 模型               | 努力程度                       | 分数                  | 每任务成本     |
| ----------------------------------------------------------------------------------------------------------------------------------------- | ---------------- | -------------------------- | ------------------- | --------- |
| SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 子集      | Claude Opus 5.5  | `low`                      | 解决了 87.4% 的任务       | $0.11 USD |
| SWE-bench Pro 子集                                                                                                                          | Claude Opus 5.5  | `medium`（默认）               | 解决了 92.8% 的任务       | $0.21 USD |
| SWE-bench Pro 子集                                                                                                                          | Claude Opus 5.5  | `high`                     | 解决了 95.3% 的任务       | $0.29 USD |
| SWE-bench Pro 子集                                                                                                                          | Claude Opus 5.5  | `xhigh`                    | 解决了 96.7% 的任务（单次运行） | $0.74 USD |
| DeepResearch Bench II<sup>[7](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> | Claude Fable 5.2 | `low`                      | 评分标准得分 54.5%        | $1.49 USD |
| DeepResearch Bench II                                                                                                                     | Claude Fable 5.2 | `medium`                   | 评分标准得分 55.0%        | $2.16 USD |
| DeepResearch Bench II                                                                                                                     | Claude Fable 5.2 | `high`（Messages API 上的默认值） | 评分标准得分 57.7%        | $4.82 USD |

下图绘制了 Claude Opus 5.5 在 SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 子集上的准确率与每任务成本的关系：

<Frame>
  ![按 effort（努力程度）绘制的准确率与每任务成本折线图：Claude Opus 5.5 在 SWE-bench Pro 上从 low 到 high 陡峭上升](https://platform.claude.com/docs/zh-CN/images/cost-intel/effort-sweep-opus-5-5-swe-bench-pro.svg)
</Frame>

下图绘制了 Claude Fable 5.1 在 DeepResearch Bench II<sup>[7](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 上的评分标准得分与每任务成本的关系：

<Frame>
  ![折线图，DeepResearch Bench II，Fable 5.1：随着成本上升，评分标准得分在 low 努力程度下为 60.5%，在 medium 下为 60.5%，在 high 下为 63.0%](https://platform.claude.com/docs/zh-CN/images/cost-intel/effort-limit-repriced.svg)
</Frame>

**应该怎么做。**

* 在您自己流量的样本上扫描两到三个努力程度级别，每个级别在单独的会话中进行：在会话中途更改顶层努力程度会[使缓存失效](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#cache-repeated-context)并扭曲比较结果。这条曲线是任何多模型策略都必须超越的基线（[在您自己的工作负载上测量的第 2 步](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#measure-on-your-own-workload)）。
* 在长周期编码中，努力程度是一种真实的权衡，而[以更高的努力程度重新运行失败的任务](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#re-run-failures-at-higher-effort)可以将其重新转化为节省。在 `medium` 下，Claude Sonnet 5.5 在 SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 子集上的得分比其默认值 `high` 低约 7 分，每任务成本约为其 59%（各运行 2 次）；在内部智能体编码基准测试<sup>[11](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>上得分为 62.7%，而默认值下为 77.8%（每个任务 1 次尝试）。
* 在 `xhigh` 和 `max` 下，Claude Sonnet 5.5 可能会在完成任务后自行开始多轮审查，这会花费更多时间和令牌。为了让这些工作聚焦于任务，请将[为 Claude Sonnet 5.5 编写提示](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5-5#steer-initiative-and-scope)中"`xhigh` 和 `max` effort 下的彻底性"下的提示添加到您的系统提示中。
* 在研究任务上，请测量您实际部署的模型。Claude Fable 5.2 在 DeepResearch Bench II 上 `high` 的得分比 `low` 高 3.3 分，成本为其 3.2 倍，而 `medium` 下的得分并没有可测量地高于 `low`；Claude Opus 5.5 在 DRACO<sup>[21](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 上 `low` 的得分比 `medium` 低 13.3 分。这两个结果来自不同基准测试上的不同模型，因此它们表明答案会变化，但不能说明原因。
* 在 `low` 下，Claude Opus 5.5 的典型任务耗时比 `medium` 下在 DRACO<sup>[21](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 上少 84%，在 HLE<sup>[22](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 上少 30%，在一组内部物理问题<sup>[23](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>上少 47%，而得分分别低 13.3、3.4 和 14.0 分（每个任务分别 3、3 和 4 次尝试）。
* 在一个内部缺陷搜索基准测试<sup>[8](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>上（每种设置运行 10 次），Claude Opus 5.5 在 `low` 下的得分比在 `medium` 下低 34 分，成本约为其五分之一。Claude Fable 5.2 在 `low` 下的得分比在 `high` 下低 4.3 分，成本低 37%。

### 以更高的努力程度重新运行失败的任务

当任务结果可检验时，努力程度曲线上最便宜的策略并不是某个固定设置：以较低设置运行每个任务，只以较高设置重新运行失败的任务。

Anthropic 根据[调整努力程度](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#tune-effort)中 Claude Opus 5.5 在 SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 子集上的努力程度运行结果，逐个任务计算了这一策略：

<Frame>
  ![图表，SWE-bench Pro，Opus 5.5：以 low 或 medium 努力程度（effort）运行并以 high 重新运行失败任务，可以以低于 high 的成本达到任何固定努力程度的水平](https://platform.claude.com/docs/zh-CN/images/cost-intel/escalation-opus-5-5.svg)
</Frame>

在下表中，重新运行的成本计入了失败的低成本尝试：

| 策略                         | 解决的任务 | 每任务成本       |
| -------------------------- | ----- | ----------- |
| `low`，失败任务以 `high` 重新运行    | 约 97% | 约 $0.17 USD |
| `medium`，失败任务以 `high` 重新运行 | 约 97% | 约 $0.24 USD |
| 全部使用 `high`                | 95.3% | $0.29 USD   |

* 在 `low` 下，13% 的任务失败。将这些任务以 `high` 重新运行后，通过率略高于全部使用 `high`，而成本仅略高于一半。
* 使用此策略是为了节省成本，而不是为了提升分数。这一小幅提升大部分来自第二次尝试：将一次 `high` 运行中的失败任务以 `high` 重新运行，得分大致相同，但花费更多。

这需要满足两个条件。第一，您需要一个失败信号（此处为基准自带的测试）；一个会放过糟糕工作的检查器会让这些失败漏过去。第二，每个首轮失败都需要两次运行的实际耗时，因此节省的成本是以失败任务上的延迟为代价的。

### 设置预算和输出上限

两种控制手段承担两种不同的职责：会话预算限制一个 Claude Managed Agents 会话可以花费的金额，而 `max_tokens` 限制一次响应的长度。在各自适用的地方设置它们，并以[工作区支出限制](https://platform.claude.com/docs/zh-CN/api/rate-limits#setting-lower-limits-for-workspaces)作为最后的保障。

* **Claude Managed Agents 上的会话预算**是对支出的硬性限制。[会话预算](https://platform.claude.com/docs/zh-CN/managed-agents/budgets)是对一个会话的[标价成本](https://platform.claude.com/docs/zh-CN/managed-agents/budgets#how-list-cost-is-measured)设置的美元上限，平台会对令牌、网络搜索和会话运行时间进行计量。
* 达到上限或略微超过上限时，会话会以 `stop_reason: budget_reached` 暂停；提高预算即可恢复。它由平台强制执行，适用于任何有标价的模型。部署会对其启动的每个会话应用相同的预算。
* **`max_tokens`** 限制单次响应，且对模型不可见，因此降低它并不会让模型节省。需要更多空间的轮次会被丢弃，但仍会计费。重试被截断的尝试很少有帮助：在相同上限下，大多数会再次失败；在更高上限下，您还要为浪费的尝试付费。
* 对于智能体工作，将 `max_tokens` 设置为 64,000；当单次被截断的尝试代价高昂时，设置为最大值 128,000。对如此大的响应使用[流式传输](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming)，将 [`stop_reason: max_tokens`](https://platform.claude.com/docs/zh-CN/build-with-claude/handling-stop-reasons#max-tokens) 视为失败，并通过模型可见的努力程度来节省成本。
* 在 Claude Haiku 5.5 上，请调高较小的 `max_tokens` 或降低努力程度：思考默认开启并计入上限，因此响应可能在输出任何文本之前就结束（请参阅 [Claude Haiku 5.5 迁移指南](https://platform.claude.com/docs/zh-CN/models/haiku-5-5/migration-guide#configure-thinking)）。

下表显示了在一个内部代码仓库任务基准测试<sup>[12](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>上，每个模型在每个上限下解决了多少任务以及每个已解决任务的成本，每个任务一次尝试，每个模型使用其在 Messages API 上的默认努力程度。16,384 的行是两次运行的平均值，其他行各为一次运行：

| 模型               | `max_tokens` | 因上限而结束的尝试 | 解决的任务 | 每个已解决任务的成本     |
| ---------------- | ------------ | --------- | ----- | -------------- |
| Claude Opus 5.5  | 16,384       | 25%       | 57.6% | 按标价 $5.38 USD  |
| Claude Opus 5.5  | 64,000       | 0%        | 75.6% | 按标价 $5.38 USD  |
| Claude Opus 5.5  | 128,000      | 0%        | 68.9% | 按标价 $5.94 USD  |
| Claude Fable 5.2 | 16,384       | 85%       | 20.3% | 按标价 $27.96 USD |
| Claude Fable 5.2 | 64,000       | 2%        | 77.4% | 按标价 $24.42 USD |
| Claude Fable 5.2 | 128,000      | 0%        | 78.2% | 按标价 $24.52 USD |

**Claude Opus 5.5 在 64,000 和 128,000 下：** 在这两个上限下都没有尝试被截断，因此上限无法解释这两行在解决的任务或每个已解决任务成本上的差异。每行为一次运行。

请在每个模型内部比较不同上限。这两个模型是在不同的运行中测量的。

在内部代码仓库任务<sup>[12](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>上，几乎每一轮都远低于 16,384 令牌的上限就结束了，但只要有一个较长的轮次就会终止整个尝试：在更高的上限下，Claude Fable 5.2 有 1.7% 的轮次超过 16,384 个令牌，而在 16,384 下，该上限终止了其 85% 的尝试。更高的上限换来的是罕见的长轮次（此图表中的两个模型为 Claude Opus 5.5 和 Claude Fable 5.1）：

<Frame>
  ![Opus 5.5 和 Fable 5.1 每轮输出的点图：中位数为几百个令牌，最长为 61k 和 128k，与上限对比](https://platform.claude.com/docs/zh-CN/images/cost-intel/max-tokens-ladder-opus-5-5.svg)
</Frame>

要求更简短的回答。在 Claude Haiku 5.5、Claude Sonnet 5.5、Claude Opus 5.5、Claude Fable 5.2 和 Claude Fable 5.1 上，输出令牌的价格是输入令牌的五倍；而在智能体循环中，模型写出的每个令牌都会在之后的每一轮中作为输入返回，因此您会为一个长回答反复付费。

在一次针对较早模型的测量<sup>[26](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>中，Anthropic 运行了一个对错误报告进行分诊的小型智能体（[精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)），使用三种最终回答指令，每种运行三次，模型和工具相同：

* 原始版本要求两行。
* 较短的变体要求一行。
* 较长的变体要求一份包含六个带标题章节的备忘录。

下图绘制了每种格式每次运行的成本：

<Frame>
  ![条形图：单行格式每次运行 $0.49 USD，原始两行格式 $0.57 USD，备忘录 $1.40 USD，正确率均为 78% 到 85%](https://platform.claude.com/docs/zh-CN/images/cost-intel/output-format.svg)
</Frame>

对照正确标签，三者的得分彼此都在运行间噪声范围内，因此这些格式在您支付的费用上的差异远大于在正确性上的差异。请要求您真正会阅读的回答，而不是看起来详尽的回答。

### 向模型显示已用时间

要让智能体更快完成并降低成本，请告诉它时间很重要，并在每个请求中向它显示已用时间。在 Anthropic 的测量中，这两项更改使 `medium` 努力程度下的单个 Claude Opus 5.5 智能体在典型（中位数）DRACO<sup>[21](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 研究任务上的耗时减少了 47%，成本减少了 60%，而其平均得分低了 4.1 分。

在 HLE<sup>[22](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 和一组内部物理问题<sup>[23](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>上，典型任务的耗时分别减少了 14% 和 7%，物理问题的得分低了 3.0 分。将努力程度降至 `low` 比这些更改节省了更多时间和成本，但损失了更多分数（参考文献 21）。在采用这些更改之前，请在您自己的任务上检查得分。

参考文献 21 给出了所测量运行中这两项更改所用的确切文本。有关可运行的代码，请参阅 Claude Cookbook 中的示例[延迟压力和预算限制下的多智能体团队](https://github.com/anthropics/claude-cookbooks/blob/main/patterns/agents/latency_multi_agent.ipynb)，它将这两项更改应用于一个智能体团队。如果要改为使用系统提示而不是第一条 `user` 消息来说明时间很重要，请参阅[多智能体 harness 的时间信号](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-opus-5-5#time-signals-for-multi-agent-harnesses)。

下图还绘制了一个团队（其主智能体很少启动辅助智能体）以及一个 `low` 努力程度下的单个智能体：

<Frame caption="采用这两项更改后，单个智能体在典型 DRACO 任务上的耗时减少了 47%，其平均得分低了 4.1 分。">
  ![散点图和条形图，DRACO、HLE 和物理集：这些改动在 DRACO 上节省的时间最多，代价是几个百分点的得分](https://platform.claude.com/docs/zh-CN/images/cost-intel/time-awareness.svg)
</Frame>

如果要改为花钱换取速度，请在 Claude Fable 5.2 或 Claude Opus 5.5 上设置 `speed: "fast"`。[快速模式](https://platform.claude.com/docs/zh-CN/build-with-claude/fast-mode)以标准令牌价格的两倍，提供最高 2.5 倍的每秒输出令牌数。在对话中途切换速度会导致提示缓存未命中。

## 组合模型

在添加第二个模型之前，先调优一个模型：首先在当前模型上扫描努力程度，因为大多数工作负载到此为止；如果扫描显示存在差距，再为低努力程度下单独使用的更强模型定价。只有当组合方案超越单个模型的整条得分-支出曲线时，才构建组合。在您自己的工作负载上，将组合方案与较低努力程度下的同一模型以及单独运行的低一级模型进行比较。

下表比较了两种策略：

| 策略                    | 控制流              | 前沿模型的角色     | 适合                                   | 前沿成本随之增长的因素 |
| --------------------- | ---------------- | ----------- | ------------------------------------ | ----------- |
| **顾问（Advisor）**       | 较小模型运行循环，按需升级    | 被咨询以获取计划和纠正 | 局部困难的串行工作，例如编码智能体在少数真正决策之间的许多轮次      | 执行器卡住的频率    |
| **编排器（Orchestrator）** | 前沿模型运行循环，委派大批量工作 | 规划、分派和综合    | 可扇出到真正独立的文件、文档或案例的工作，尤其是超过一个上下文窗口的工作 | 各部分协调的难度    |

### 顾问策略：升级困难决策

只有当 “executor”（执行器）咨询顾问时，顾问才能提供帮助。

在 Anthropic 的测量中，唯一一个执行器频繁咨询的组合（在一个编码基准测试上）比相同努力程度下单独的执行器高 1.7 分，处于运行间噪声的边缘，成本约为 2.1 倍。在执行器很少或从不咨询的情况下，组合的得分与单独的执行器相差在运行间噪声范围内，或者更低。

要启用它，请在您的请求中添加 [advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool)（一项 beta 功能）。在 Claude Managed Agents 上，请[为会话配置顾问](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration#give-the-session-an-advisor)。对于 Claude Code，请参阅[使用 advisor 工具升级困难决策](https://code.claude.com/docs/zh-CN/advisor)。

<Frame>
  ![advisor strategy（顾问策略）示意图：executor（执行器）模型运行主循环，并按需调用 Claude Fable 5.1 advisor（顾问）](https://platform.claude.com/docs/zh-CN/images/model-routing-advisor-strategy.svg)
</Frame>

下表将每个组合与相同努力程度下单独的执行器进行比较：

| 基准测试                                                                                                                              | 分数                              | 成本                                                     | 顾问被咨询情况                         |
| --------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- | ------------------------------------------------------ | ------------------------------- |
| 内部智能体编码基准测试<sup>[11](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>  | 90.1%，比单独的执行器高 1.7 分，处于运行间噪声的边缘 | 每次尝试 $2.92 USD，约为单独执行器的 2.1 倍                          | 每次尝试约 1.4 次；4% 的尝试未获得建议         |
| GPQA Diamond<sup>[9](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>  | 与单独的执行器相差在运行间噪声范围内              | Claude Haiku 5.5 每个问题成本增加 12%，Claude Sonnet 5.5 增加 25% | 在 198 个问题中均未咨询，各运行两次；没有系统提示要求咨询 |
| Chartography<sup>[13](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> | 61.7，比单独的执行器低 7 分，超出运行间噪声       | 与单独的执行器大致相同                                            | 300 个任务中有 1 个                   |

* **内部智能体编码基准测试。** `high` 下的 Claude Opus 5.5，配合 Claude Fable 5.1 顾问。
* **GPQA Diamond。** 各自默认设置（`medium` 和 `high`）下的 Claude Haiku 5.5 或 Claude Sonnet 5.5，配合 Claude Opus 5.5 顾问。
* **Chartography。** `low` 下的 Claude Opus 5.5，配合 Claude Fable 5.1 顾问。

顾问只能提供执行器所缺乏的能力：在 GPQA Diamond 上，将拒绝回答计为错误时，单独的 Claude Sonnet 5.5 与单独的 Claude Opus 5.5 以 91% 打平，但这只是因为 Opus 5.5 每次运行拒绝了六个生物学问题，而 Sonnet 5.5 只拒绝了一个。在各自回答的问题上，Opus 5.5 领先约 2 分。

在编码基准测试上，组合的提升处于运行间噪声的边缘，并且大致落在 Claude Opus 5.5 自身的努力程度曲线上，因此顾问带来的效果与提高努力程度大致相同。下图将该组合与这条曲线以及 Claude Fable 5.1 八月份的曲线进行对比：

<Frame>
  ![编码基准测试：high 下的 Opus 5.5 搭配 Fable 5.1 advisor（顾问）以 2.1 倍成本提升 1.7 个百分点，接近其 effort（努力程度）曲线](https://platform.claude.com/docs/zh-CN/images/cost-intel/internal-coding-advisor-opus-5-5.svg)
</Frame>

**应该怎么做。**

* 在您自己的任务上测量咨询率。advisor 工具的定义每个请求约 1,000 个提示令牌，即使执行器从不咨询也会增加成本。使用[提示缓存](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#cache-repeated-context)时，多轮智能体循环在第一个请求之后以缓存读取价格为该定义付费，因此这项开销对简短的单请求任务影响最大。
* 如果执行器很少咨询，请参阅[针对编码和智能体任务的提示](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#prompting-for-coding-and-agent-tasks)，了解要求咨询的系统提示，并在添加后重新测量。在 20 个 SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 问题上，建议的提示对 Claude Sonnet 5.5 没有产生明显差异。
* 如果咨询率骤降，请恢复执行器的努力程度。在对较早模型的测量中，一个在默认努力程度下对大多数任务进行咨询的组合，在降低努力程度后很少咨询，随后得分低于默认努力程度下单独的执行器（SWE-bench Pro<sup>[3](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>）。
* 在每次模型发布时重新检查，因为发布会同时改变能力差距和价格比。

### 编排器策略：委派批量工作

在 Anthropic 的测量中，委派只在两种情况下更胜一筹：

* 具有长成本尾部的常规工作（BrowseComp<sup>[4](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 的一个简单切片，在较早模型上测得）
* 超过一个上下文窗口的工作（一个内部缺陷搜索基准测试<sup>[8](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>）

当工作是一条相互依赖的链，或者能放入单个上下文且没有长成本尾部时，在较早模型上测量的所有情况下（BrowseComp 的较难题集以及 DeepWideSearch<sup>[6](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>），较低努力程度下单独使用编排器的模型都更胜一筹；独立的外部研究也报告了相同的模式<sup>[5](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。

要构建编排器，请在 Claude Managed Agents 中使用[多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)：配置一个编排器智能体以及它可以调用的子智能体，每个智能体都有自己的模型。

<Frame>
  ![orchestrator（编排器）策略示意图：前沿模型将子任务分发给三个成本较低的工作者模型](https://platform.claude.com/docs/zh-CN/images/model-routing-orchestrator-strategy.svg)
</Frame>

下图按成本和得分绘制了缺陷搜索基准测试十七种配置中的九种（不包括 Claude Fable 5.2 的五种）：

<Frame>
  ![散点图，缺陷搜索：Opus 5.5 搭配 Sonnet 5.5 子智能体与 medium 下单独使用的 Opus 5.5 得分相当，成本低约三分之一](https://platform.claude.com/docs/zh-CN/images/cost-intel/corpus-pareto.svg)
</Frame>

**应该怎么做。**

* **具有长成本尾部的常规工作：** 首先测量这种情况，既要测量成本，也要测量准确率。其背后的测量只涵盖了委派配置在每次运行中都回答正确的问题（参考文献 4）。
* **超过一个上下文窗口的工作：** 将其拆分为多个部分，每个部分一个会话或一个子智能体。在缺陷搜索基准测试<sup>[8](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>上，Claude Opus 5.5 搭配 Claude Sonnet 5.5 子智能体的成本，比默认努力程度下每个部分一个 Claude Opus 5.5 会话低约三分之一，得分大致相同。用 Claude Fable 5.2 代替 Claude Opus 5.5 作为编排器，成本高 19%，得分大致相同。单独使用时，从 `low` 到 `high`，Claude Fable 5.2 的得分比 `high` 下单独使用的 Claude Opus 5.5 高 1.6 到 5.9 分，成本为其 1.5 到 2.4 倍；比其搭配 Claude Sonnet 5.5 子智能体时高 3.2 到 7.5 分，成本为其 2.2 到 3.4 倍。在每个努力程度下，它的得分也都高于单独使用的 Claude Fable 5.1：`low` 下高 3.6 分，成本低 32%；`medium` 下高 3.8 分，成本低 37%；`high` 下高 6.1 分，成本低 21%。
* **编码任务上的 Claude Haiku 5.5 子智能体：** 先将其与较低努力程度进行比较。在 Claude Managed Agents 上的一项内部编码基准测试中，它们是降低努力程度的一种替代方案，而非更优方案。将它们添加到 `medium` 下的 Claude Opus 5.5 并配以简短的委派提示后，每次尝试的成本降低了 32%，通过率降低了约 4 个百分点（95% 区间为 1.3 到 7.1），大致相当于在该成本下不使用子智能体、混合使用 `low` 和 `medium` 所能达到的效果<sup>[29](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup>。

## 在您自己的工作负载上进行测量

请在您自己的工作负载上进行测量：本页中的数字反映的是测量时的标价，会随着模型和价格的变化而漂移，而您的升级率、任务拆分的干净程度以及对话记录（transcript）的长度也会影响这些数字。方法保持不变：

1. 从生产日志中抽取一些任务，按真实流量加权，并为每个任务[编写结果检查](https://platform.claude.com/docs/zh-CN/test-and-evaluate/develop-tests)：测试通过、工单关闭、行数正确。在分数旁边记录每个任务的成本。
2. 在各努力程度级别（而不仅是默认级别）上为各模型层级建立基线，并绘制得分与支出的关系图。多模型配置必须超越单个模型的整条曲线。
3. 如果曲线显示存在努力程度无法弥补的差距，请添加合适的多模型策略，并重新运行测试套件。
4. 在正式切换之前，先在一部分流量上以影子模式运行胜出方案，之后保持测试套件持续运行。

**如何为任务定价。** 按各自的费率为响应的 `usage` 中的五个令牌计数分别定价：未缓存的输入、5 分钟和 1 小时缓存写入（分别为输入价格的 1.25 倍和 2 倍）、缓存读取以及输出。将任务中所有请求的这些费用相加（[用量和成本分析 API](https://platform.claude.com/docs/zh-CN/manage-claude/usage-cost-analytics-api) 会报告汇总数据）。在 Claude Haiku 5.5 上，请按每个请求的提示长度（缓存读取和写入也计入该长度）使用相应的费率（请参阅[长上下文定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)）。当 `usage.speed` 为 `"fast"` 时，请使用[快速模式费率](https://platform.claude.com/docs/zh-CN/about-claude/pricing#fast-mode-pricing)。

以下示例按 Claude Opus 5.5 的标价计算单个请求的成本。如需用于其他模型，请更改这三个价格：

<CodeGroup exclude="shell:cURL, shell:CLI">
  <CodeGroupItem language="cURL">
    此计算需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  <CodeGroupItem language="CLI">
    此计算需要的逻辑超出了单个 shell 命令所能实现的范围。请使用 [Anthropic SDK](https://platform.claude.com/docs/zh-CN/cli-sdks-libraries/overview) 运行它。
  </CodeGroupItem>

  ```python Python
  INPUT_PER_MTOK = 4.00  # Claude Opus 5.5
  # 在 Claude Opus 5.5 上为输入价格的 0.05 倍；该倍数因模型而异
  CACHE_READ_PER_MTOK = 0.20
  OUTPUT_PER_MTOK = 20.00

  client = anthropic.Anthropic()
  response = client.messages.create(
      model="claude-opus-5-5",
      max_tokens=1024,
      messages=[{"role": "user", "content": "Hello, Claude"}],
  )
  usage = response.usage
  cache_writes = usage.cache_creation
  writes_1h = cache_writes.ephemeral_1h_input_tokens if cache_writes else 0
  writes_5m = cache_writes.ephemeral_5m_input_tokens if cache_writes else 0
  cost = (
      usage.input_tokens * INPUT_PER_MTOK
      # 1 小时缓存写入按输入价格的 2 倍计费，5 分钟缓存写入按 1.25 倍计费；读取按缓存读取价格计费。
      + writes_1h * INPUT_PER_MTOK * 2.0
      + writes_5m * INPUT_PER_MTOK * 1.25
      + (usage.cache_read_input_tokens or 0) * CACHE_READ_PER_MTOK
      + usage.output_tokens * OUTPUT_PER_MTOK
  ) / 1_000_000
  print(f"Request cost: ${cost:.6f}")
  ```

  ```typescript TypeScript
  const INPUT_PER_MTOK = 4.0; // Claude Opus 5.5
  const CACHE_READ_PER_MTOK = 0.2; // 0.05x the input price on Claude Opus 5.5; the multiplier differs by model
  const OUTPUT_PER_MTOK = 20.0;

  const client = new Anthropic();
  const response = await client.messages.create({
    model: "claude-opus-5-5",
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  });
  const usage = response.usage;
  const cost =
    (usage.input_tokens * INPUT_PER_MTOK +
      (usage.cache_creation?.ephemeral_1h_input_tokens ?? 0) * INPUT_PER_MTOK * 2 + // 1-hour cache write
      (usage.cache_creation?.ephemeral_5m_input_tokens ?? 0) * INPUT_PER_MTOK * 1.25 + // 5-minute cache write
      (usage.cache_read_input_tokens ?? 0) * CACHE_READ_PER_MTOK + // cache read
      usage.output_tokens * OUTPUT_PER_MTOK) /
    1_000_000;
  console.log(`Request cost: $${cost.toFixed(6)}`);
  ```

  ```csharp C#
  const double InputPerMtok = 4.00; // Claude Opus 5.5
  const double CacheReadPerMtok = 0.20; // 0.05x the input price on Claude Opus 5.5; the multiplier differs by model
  const double OutputPerMtok = 20.00;

  AnthropicClient client = new();
  var response = await client.Messages.Create(
      new MessageCreateParams
      {
          Model = Model.ClaudeOpus5_5,
          MaxTokens = 1024,
          Messages = [new UserMessageParam { Content = "Hello, Claude" }],
      }
  );
  var usage = response.Usage;
  double cost =
      (
          usage.InputTokens * InputPerMtok
          + (usage.CacheCreation?.Ephemeral1hInputTokens ?? 0) * InputPerMtok * 2.00 // 1-hour cache write
          + (usage.CacheCreation?.Ephemeral5mInputTokens ?? 0) * InputPerMtok * 1.25 // 5-minute cache write
          + (usage.CacheReadInputTokens ?? 0) * CacheReadPerMtok // cache read
          + usage.OutputTokens * OutputPerMtok
      ) / 1_000_000;
  Console.WriteLine($"Request cost: ${cost:F6}");
  ```

  ```go Go
  const (
  	inputPerMTok     = 4.00 // Claude Opus 5.5
  	cacheReadPerMTok = 0.20 // 0.05x the input price on Claude Opus 5.5; the multiplier differs by model
  	outputPerMTok    = 20.00
  )

  // ...
  	client := anthropic.NewClient()

  	response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
  		Model:     anthropic.ModelClaudeOpus5_5,
  		MaxTokens: 1024,
  		Messages: []anthropic.MessageParam{
  			anthropic.NewUserMessage(anthropic.NewTextBlock("Hello, Claude")),
  		},
  	})
  	if err != nil {
  		log.Fatal(err)
  	}

  	usage := response.Usage
  	cost := (float64(usage.InputTokens)*inputPerMTok +
  		float64(usage.CacheCreation.Ephemeral1hInputTokens)*inputPerMTok*2.00 + // 1-hour cache write
  		float64(usage.CacheCreation.Ephemeral5mInputTokens)*inputPerMTok*1.25 + // 5-minute cache write
  		float64(usage.CacheReadInputTokens)*cacheReadPerMTok + // cache read
  		float64(usage.OutputTokens)*outputPerMTok) / 1_000_000
  	fmt.Printf("Request cost: $%.6f\n", cost)
  ```

  ```java Java
  static final double INPUT_PER_MTOK = 4.00; // Claude Opus 5.5
  static final double CACHE_READ_PER_MTOK = 0.20; // 0.05x the input price on Claude Opus 5.5; the multiplier differs by model
  static final double OUTPUT_PER_MTOK = 20.00;

  void main() {
      AnthropicClient client = AnthropicOkHttpClient.fromEnv();

      Message response = client.messages().create(MessageCreateParams.builder()
          .model(Model.CLAUDE_OPUS_5_5)
          .maxTokens(1024)
          .addUserMessage("Hello, Claude")
          .build());

      Usage usage = response.usage();
      long writes1h = usage.cacheCreation().map(CacheCreation::ephemeral1hInputTokens).orElse(0L);
      long writes5m = usage.cacheCreation().map(CacheCreation::ephemeral5mInputTokens).orElse(0L);
      double cost = (usage.inputTokens() * INPUT_PER_MTOK
          + writes1h * INPUT_PER_MTOK * 2.00 // 1-hour cache write
          + writes5m * INPUT_PER_MTOK * 1.25 // 5-minute cache write
          + usage.cacheReadInputTokens().orElse(0L) * CACHE_READ_PER_MTOK // cache read
          + usage.outputTokens() * OUTPUT_PER_MTOK) / 1_000_000;
      IO.println("Request cost: $%.6f".formatted(cost));
  }
  ```

  ```php PHP
  const INPUT_PER_MTOK = 4.00; // Claude Opus 5.5
  const CACHE_READ_PER_MTOK = 0.20; // 0.05x the input price on Claude Opus 5.5; the multiplier differs by model
  const OUTPUT_PER_MTOK = 20.00;

  $client = new Client();
  $response = $client->messages->create(
      model: 'claude-opus-5-5',
      maxTokens: 1024,
      messages: [['role' => 'user', 'content' => 'Hello, Claude']],
  );
  $usage = $response->usage;
  $cost = (
      $usage->inputTokens * INPUT_PER_MTOK
      + ($usage->cacheCreation?->ephemeral1hInputTokens ?? 0) * INPUT_PER_MTOK * 2.00 // 1-hour cache write
      + ($usage->cacheCreation?->ephemeral5mInputTokens ?? 0) * INPUT_PER_MTOK * 1.25 // 5-minute cache write
      + ($usage->cacheReadInputTokens ?? 0) * CACHE_READ_PER_MTOK // cache read
      + $usage->outputTokens * OUTPUT_PER_MTOK
  ) / 1_000_000;
  printf("Request cost: \$%.6f\n", $cost);
  ```

  ```ruby Ruby
  INPUT_PER_MTOK = 4.00 # Claude Opus 5.5
  CACHE_READ_PER_MTOK = 0.20 # 0.05x the input price on Claude Opus 5.5; the multiplier differs by model
  OUTPUT_PER_MTOK = 20.00

  client = Anthropic::Client.new
  response = client.messages.create(
    model: Anthropic::Model::CLAUDE_OPUS_5_5,
    max_tokens: 1024,
    messages: [{ role: "user", content: "Hello, Claude" }]
  )
  usage = response.usage
  cost = (
    usage.input_tokens * INPUT_PER_MTOK +
    usage.cache_creation&.ephemeral_1h_input_tokens.to_i * INPUT_PER_MTOK * 2.00 + # 1-hour cache write
    usage.cache_creation&.ephemeral_5m_input_tokens.to_i * INPUT_PER_MTOK * 1.25 + # 5-minute cache write
    usage.cache_read_input_tokens.to_i * CACHE_READ_PER_MTOK + # cache read
    usage.output_tokens * OUTPUT_PER_MTOK
  ) / 1_000_000
  puts format("Request cost: $%.6f", cost)
  ```
</CodeGroup>

**需要检查的内容。**

* **缓存读取占比。** 在智能体循环中，大多数输入令牌应为缓存读取。如果 `cache_read_input_tokens` 相对于 `input_tokens` 加上 `cache_creation_input_tokens` 很小，请检查缓存是否已启用，以及前缀在请求之间是否保持不变。
* **Advisor 工具和压缩。** 启用 [advisor 工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/advisor-tool#usage-and-billing)或[压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold#understanding-usage)时，部分令牌仅在 `usage.iterations` 中报告，而不计入顶层总数。请改为对 `usage.iterations` 求和，并按顾问模型的费率为 `advisor_message` 条目定价。

下表按建议尝试的顺序列出了各项优化手段。这些数据来自 Anthropic 的测试、发布公告以及公布的价格和速度：

| 手段              | 节省                                                                 | 质量代价                                                                                                                                        | 延迟                                                                                                                                     | 位置                                                                                                                                                    |
| --------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| 提示缓存            | 在一个研究智能体上成本降低为原来的 1/4.8 到 1/7.2，在一个小型分诊智能体上降幅为 83% 到 86%           | 无                                                                                                                                           | 更快                                                                                                                                     | [缓存重复的上下文](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#cache-repeated-context)                    |
| 1 小时缓存时长        | 当每 10 轮中有 1 轮发生在暂停之后时，5 分钟默认时长的成本高出 54%；每 20 轮中有 1 轮时，成本大致相同       | 无                                                                                                                                           | 在早期模型上，暂停后缓存仍保持预热<sup>[16](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> | [选择缓存时长](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#pick-the-cache-duration)                     |
| 精简输入            | 在小型分诊智能体的账单上再节省 21% 到 26%                                          | 使用工具搜索时，准确率与不使用时处于相同范围<sup>[27](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> | 无影响                                                                                                                                    | [精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)           |
| 工具搜索            | 附加 500 个工具定义时节省 45%；延迟加载 GitHub MCP 服务器的工具集节省 20%                  | 对于任务从不需要的工具，无代价                                                                                                                             | 无影响                                                                                                                                    | [精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)           |
| 通过代码执行处理数据文件    | 在一个包含 25 个问题的数据任务上节省 92%（关闭提示缓存）                                   | 有提升：25 题全对，而非 7 题                                                                                                                           | 更快                                                                                                                                     | [精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)           |
| Batch API       | 50%                                                                | 无                                                                                                                                           | 24 小时内返回结果                                                                                                                             | [批处理可以等待的工作](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#batch-work-that-can-wait)                |
| 针对当前模型审计提示      | 在测试的两次模型升级中均节省 14%                                                 | 无；其中一次有提升                                                                                                                                   | 更快（工具调用轮次更少）                                                                                                                           | [针对当前模型审计提示](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#audit-prompts-against-the-current-model) |
| 升级模型            | 通常更低：在典型工作负载上最多降低 40%，在一个编码基准上每个已解决任务的成本最多降低 87%                   | 在编码上无代价，大多数情况下有提升；在一个研究基准上，Claude Fable 5.2 的得分更低                                                                                           | 无影响                                                                                                                                    | [升级模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#upgrade-the-model)                             |
| 降低努力程度          | 在长编码任务上，`medium` 的成本比 `high` 低约 30% 到 52%，`low` 则低约三分之二            | 视任务和模型而定，最多低 34 分；在一个研究智能体上低 3.3 分                                                                                                          | 更快                                                                                                                                     | [调整努力程度](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#tune-effort)                                 |
| 重新运行失败任务        | 比全部以 `high` 运行低约 40%，通过率相同或略高                                      | 无                                                                                                                                           | 对失败的任务进行第二次尝试                                                                                                                          | [以更高的努力程度重新运行失败的任务](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#re-run-failures-at-higher-effort) |
| 要求更简短的回答        | 输出令牌减少 39%，成本降低约 13%                                               | 无                                                                                                                                           | 更快                                                                                                                                     | [设置预算和输出上限](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#set-budgets-and-output-caps)              |
| 提高 `max_tokens` | 无：每个已解决任务的成本在 64,000 时没有上升                                         | 有提升：在一个编码基准上已解决任务最多增加 57 个百分点（在 64,000 时）；在另一个基准上无变化                                                                                        | 无影响                                                                                                                                    | [设置预算和输出上限](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#set-budgets-and-output-caps)              |
| 顾问              | 无：在一个编码基准上，成本约为单独使用该模型的 2.1 倍                                      | 在编码上有小幅提升，但可能是偶然；在图表阅读上有损失                                                                                                                  | 在编码上，每次尝试约多 1.4 次调用                                                                                                                    | [顾问策略](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#advisor-strategy-escalate-hard-decisions)      |
| 编排器             | 在超出单个上下文窗口的工作上，使用低一级模型的子智能体时，成本比默认努力程度下单独使用编排器模型低约三分之一；对于常规工作，请先测量 | 在该类工作上没有可测量的损失                                                                                                                              | 最多同时运行 25 个子智能体                                                                                                                        | [编排器策略](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#orchestrator-strategy-delegate-bulk-work)     |
| 快速模式            | 无：标准令牌价格的两倍                                                        | 无                                                                                                                                           | 每秒输出令牌数最高提升 2.5 倍                                                                                                                      | [向模型显示已用时间](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#show-the-model-elapsed-time)              |

### 使用 Claude API skill 自动化搜索

在搜索节省空间之前，请先运行 [Claude API 技能](https://platform.claude.com/docs/zh-CN/agents-and-tools/agent-skills/claude-api-skill)的 `cost-optimize` 命令。它会分析您的令牌花在了哪里，并按顺序应用各项手段，首先是缓存：

让 Claude Code 为您寻找节省空间

```text
/claude-api cost-optimize
```

然后在存放评估的目录中运行 `hillclimb` 命令：

让 Claude Code 为您运行搜索

```text
/claude-api hillclimb
```

Anthropic 在 tau2-bench retail<sup>[24](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#refs)</sup> 上测量了 `cost-optimize` 的第一项手段以及两次搜索，均基于较早的模型。`cost-optimize` 生成的缓存断点降低了每个任务的成本，而通过率的下降幅度小于相同运行之间的波动范围。

一次搜索通过降低努力程度并下调一个模型层级，找到了更便宜的配置。其通过率仅以不到一个任务的差距保持在搜索前设定的区间内，但最终低于搜索自身判定所用的更严格下限。第二次搜索从较低成本模型的默认努力程度开始，没有保留任何更改。参考文献 24 给出了相关数据以及每次搜索的运行成本。

下图绘制了第一次搜索的每一步：

<Frame caption="在较早模型上进行的一次搜索，逐步展示。节省来自努力程度和模型的更改；提示编辑没有带来持久的提升。">
  ![散点图，tau2-bench retail：从 Claude Opus 5 开始的每个搜索步骤；low 级别的 Sonnet 5 便宜 61%，差距在 2 个百分点以内](https://platform.claude.com/docs/zh-CN/images/cost-intel/hillclimb.svg)
</Frame>

有两项设置决定了您能否信任搜索给出的答案：

* **保留一个搜索从不读取的数据划分。** 搜索在该划分上判定是否采纳更改，因此这一确认步骤能够发现仅适用于搜索所见任务的更改，正如它发现了提示编辑的问题一样（参考文献 24）。
* **在第一次更改之前重复运行基线。** 在这些测量中，同一起始配置的相同运行之间相差了好几个任务（参考文献 24）。重复运行为搜索的采纳门槛提供了噪声基准。

## 引用的基准测试

除非某条参考文献另有说明，测量结果均为 Anthropic 内部对这些基准测试的运行。除非另有注明，成本均以美元计，按各基准测试运行时有效的标价计算；Claude Sonnet 5 的数据按每百万输入和输出令牌 $2 USD 和 $10 USD 计算。2026 年 9 月 24 日运行的 Claude Sonnet 5.5 数据使用相同的输入和输出价格，以及其当前的缓存读取价格（每百万令牌 $0.10 USD，而非其 $0.20 USD 的发布价格），并应用于每次运行记录的令牌数。这些运行使用的是 Claude Sonnet 5.5 的预发布版本，其中没有任何尝试由回退模型作答。标注为"notional USD"（名义美元）的图表按这些费率为每个请求的令牌数定价，而不是报告实际账单。

1. **WideSearch：** Wong et al., “WideSearch: Benchmarking Agentic Broad Info-Seeking,” arXiv:2508.07999, 2025。广泛的网络研究任务，按多行表格的完整性和准确性评分；200 个问题，每种配置运行 3 次，运行于 2026 年 8 月 1 日至 2 日。在此基准测试、DeepWideSearch（参考文献 6）、BrowseComp（参考文献 4）和 GDPval（参考文献 2）上使用 Claude Fable 5 时，`low` 努力程度损失 1 到 3 分，换来每任务成本降低三分之一到一半；`medium` 以默认设置约 70% 到 87% 的成本达到了与默认设置相同的准确率；而在这四项中的任何一项上，默认设置相比 `medium` 都没有带来可测量的提升。成本集中度图表来自一次单独的 20 个问题的运行，每个问题运行 3 次，运行于 2026 年 8 月 3 日至 4 日，成本依据逐请求计费记录计算。它涵盖 60 次运行中的 59 次。第 60 次运行在 $432.31 USD 总额中计费 $10.91 USD，其结果未记录成本，因此未包含在内。

2. **GDPval：** OpenAI, “GDPval: Evaluating AI Model Performance on Real-World Economically Valuable Tasks,” 2025。知识工作交付物根据任务评分标准评分；对已发布黄金集的 210 个任务运行，每个任务尝试一次，于 2026 年 8 月 2 日运行。由 Claude 模型评分，因此绝对分数可能与已发布的结果不同。

3. **SWE-bench Pro：** Scale AI, “SWE-Bench Pro: Can AI Agents Solve Long-Horizon Software Engineering Tasks?”, 2025。

   * **测量内容：** 为与 Anthropic 评估框架兼容而选取的 482 个问题的子集；分数不可与公开排行榜比较。它们也不可与 Claude Opus 5.5 系统卡中的 SWE-bench Pro 结果比较，后者来自在不同问题集上以 `max` 努力程度进行的运行。
   * **定价方式：** 此子集上的成本按客户组织的计量方式定价：每个请求的先前提示按缓存读取计价，其新令牌按 5 分钟缓存写入计价，数据来自运行自身的使用记录，并与客户账本进行了核对。
   * **Claude Opus 5.5 的运行：** Claude Opus 5.5 的数据为 `low`、`medium`（其默认值）和 `high` 下两次运行的平均值，以及 `xhigh` 下的一次运行，均运行于 2026 年 9 月 19 日至 20 日，每轮上限与 8 月 Claude Opus 5 的运行相同，为 16,384 个令牌；该上限截断了 2 次 `xhigh` 尝试，其他设置下没有截断。以 `high` 为基准，Claude Opus 5.5 在其默认值 `medium` 下得分约低 2.5 分，每任务成本约为其 70%；在 `low` 下得分约低 8 分，成本约为其三分之一；`xhigh` 得分约高 1.4 分，成本为 `high` 的 2.5 倍。这些比率来自未经四舍五入的成本。
   * **478 个问题：** Opus 5.5 的运行使用了该基准测试的一个版本，其评分容器只能访问内部软件包镜像。该版本去掉了一个测试需要访问在线网站的问题，另有三个问题的参考解决方案在该版本中失败，因此 Opus 5.5 的比较以及与之并列的 Claude Fable 5.1 数据使用剩余的 478 个问题。
   * **Claude Sonnet 5.5 的运行：** Claude Sonnet 5.5 运行了相同版本的基准测试：在其默认值 `high` 下运行两次，在 `medium` 下运行两次，均于 2026 年 9 月 24 日，每轮上限同样为 16,384 个令牌，该上限未截断其任何尝试。其数据使用这 478 个问题，与之并列的 Claude Sonnet 5 数据也是如此，后者根据其 8 月的运行在这些问题上统计。
   * **Claude Opus 5 的运行：** [升级模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#upgrade-the-model)和本参考文献中的 Claude Opus 5 SWE-bench Pro 数据为其默认努力程度下两次运行的平均值，以及 `low` 下的一次运行，均运行于 2026 年 8 月 4 日。
   * **顾问，Claude Sonnet 5 配对：** [顾问策略](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#advisor-strategy-escalate-hard-decisions)中咨询率示例背后的 Claude Sonnet 5 执行器与 Claude Opus 5 顾问的配对在此子集上通过相同的测试设置运行：默认努力程度下的配对运行了两次（2026 年 8 月 7 日和 8 月 8 日，一次运行及一次完全复现），低努力程度配对运行一次（2026 年 8 月 8 日），十天后，单独的 Claude Sonnet 5 在其默认努力程度下运行两次（77.4%，两种配对的基线）。
   * **顾问，Claude Sonnet 5.5 咨询检查：** [顾问策略](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#advisor-strategy-escalate-hard-decisions)中引用的 Claude Sonnet 5.5 咨询检查于 2026 年 9 月 24 日在 478 个问题中的 20 个上各尝试一次，通过 advisor 工具配备 Claude Opus 5.5 顾问。共运行四种配置：`high` 和 `xhigh`，每种分别带有和不带 advisor 工具文档中为编码任务建议的系统提示。在其默认努力程度 `high` 下，无论有无该提示，Claude Sonnet 5.5 在 20 个问题中都没有咨询顾问。在 `xhigh` 下，不带提示时它在 20 个问题中的 3 个上进行了咨询，带提示时为 4 个，每次都发生在轮次最多的尝试上。
   * **升级数据：** 升级数据逐任务来自 Opus 5.5 的运行：先用 `low`，失败后再用 `high`，在各运行配对中解决了 96.4% 到 97.5%，成本约 $0.17 USD；先用 `medium`，为 96.0% 到 97.1%，约 $0.24 USD；`high` 在自身失败的任务上重新运行，为 96.9%，$0.31 USD；全部使用 `high`，为 94.8% 到 95.8%，$0.29 USD。
   * **比较模型：** [比较模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#compare-models-on-cost-per-task)中的 Claude Fable 5.1 数据来自在同一子集上以默认努力程度进行的一次运行，运行于 2026 年 8 月 26 日：在 478 个问题上统计，它解决了 92.3%，每个已解决任务 $1.19 USD。该部分的图表及其表格中的 Claude Opus 5.5 和 Claude Sonnet 5.5 数据合并了所绘每个设置下的两次运行，均在 478 个问题上统计：Claude Opus 5.5 的 `low`、`medium` 和 `high`，以及 Claude Sonnet 5.5 的 `medium` 和 `high`。Claude Fable 5.2 的数据为 `low` 和 `high`（其在 Messages API 上的默认值）下各一次运行，每轮上限同样为 16,384 个令牌，该上限截断了 463 次 `high` 尝试中的 8 次，`low` 下没有截断；被截断的尝试计为失败。另有一次 `high` 尝试在完成前被评估环境中的安全监控器中止，也计为失败，尽管它到那时所做的更改通过了测试；若将其排除，`high` 的结果为 96.5%。这些运行使用了该基准测试的一个版本，其评分容器没有网络访问。在用于 Opus 5.5 比较的 478 个问题中，有 15 个问题的参考解决方案在无网络访问时失败，因此 Fable 5.2 的比较使用剩余的 463 个问题。在这些比较中，Claude Opus 5.5 的数据是相同的运行在 463 个问题上统计的结果：`medium` 和 `high` 下各两次运行，`xhigh` 下一次运行。与之并列的 Claude Fable 5.1 和 Claude Opus 5.5 数据是在有网络访问的情况下评分的，因此只有问题集是匹配的，评分设置并不相同。Claude Fable 5.2 的成本使用"定价方式"中描述的计量方式，但应用于每次尝试的令牌总数而非逐请求计算，因此为估算值。
   * **比较模型，每任务工作量：** 在这些数据中，`low` 下的 Claude Opus 5.5 在每个任务上所做的工作少于默认设置下的 Claude Sonnet 5.5：输出令牌略多于 Sonnet 5.5 的一半，缓存读取略多于其三分之一。在各自默认设置下，`medium` 的 Opus 5.5 比 Sonnet 5.5 多解决约 5 个百分点，成本约为其两倍。在该设置下，它每个任务所做的工作略多于 Sonnet 5.5。
   * **升级模型，运行：** 升级模型中的 Claude Fable 5 数据点合并了默认努力程度下的三次运行，运行于 2026 年 8 月 26 日，按客户组织的计量方式定价。升级阶梯在 478 个问题上统计每个模型，每个模型均使用其出厂默认设置。除 Opus 5、Opus 5.5、Sonnet 5 和 Sonnet 5.5（各两次运行）以及 Fable 5（上述三次运行）外，每个模型使用一次运行。Opus 4.7、Opus 4.8、Sonnet 4.6 和 Sonnet 5 的运行于 2026 年 8 月 17 日至 19 日进行，Opus 5 的运行于 8 月 4 日进行，全部在同一评估框架和组织中；Opus 5.5 和 Sonnet 5.5 的运行即本参考文献前面描述的那些。
   * **升级模型，每个已解决任务的成本：** 将这些 Claude Opus 5 运行与 8 月 17 日至 19 日系列中一次采用出厂默认设置的 Claude Opus 4.8 运行进行比较，并在相同的 478 个问题上统计时，Claude Opus 5 每个已解决任务的成本比 Claude Opus 4.8 高 21%；图表的 SWE-bench Pro 面板既未显示 Claude Opus 4.7，也未显示 Claude Opus 4.8。在相同的测量中，按客户计费方式定价，Claude Opus 5 以每个已解决任务高出 21% 的成本，比 Claude Opus 4.8 多解决 12 个百分点的任务；Claude Opus 4.8 解决的任务比例与 Claude Opus 4.7 相同，每个已解决任务的成本低 14%；而 `low` 努力程度下的 Claude Opus 5 在此基准测试上胜过 Claude Opus 4.8 的默认设置，每个已解决任务的成本约为后者的 30%。
   * **升级模型，价格：** 升级模型中 Claude Sonnet 5 相对 Claude Sonnet 4.6 的节省来自其更低的每令牌价格，这足以抵消它每个任务多使用的令牌。Claude Sonnet 5.5 除缓存读取外的所有收费与 Claude Sonnet 5 相同，缓存读取价格为其一半。Claude Opus 5.5 的令牌价格低于 Claude Opus 5，其中缓存读取降幅最大。与 Claude Sonnet 5 一样，前沿层级的收益来自价格：Claude Fable 5.1 的大部分节省来自更低的缓存读取价格。
   * **不使用缓存的成本，根据记录的令牌数定价：** Anthropic 针对[为什么缓存排在首位](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#why-caching-comes-first)中的缓存系数，在 478 个问题上对 Claude Opus 5.5 和 Claude Sonnet 5.5 运行所记录的令牌数重新定价。
   * **各模型的系数：** 每个系数是将所有输入令牌按输入价格计价、输出令牌不变时的令牌成本，除以按客户组织计量方式定价的成本：对于 Claude Opus 5.5，`medium` 下为 3.3，`low` 下为 2.4，`high` 下为 3.8，`xhigh` 下为 5.4（一次运行）；对于 Claude Sonnet 5.5，缓存读取按每百万令牌 $0.10 USD 计，`high` 下为 3.1，`medium` 下为 2.4，若按 $0.20 USD 的发布价格计，`high` 下为 2.8。任务的第一个请求按缓存写入定价。

4. **BrowseComp：** Wei et al., “BrowseComp: A Simple Yet Challenging Benchmark for Browsing Agents,” OpenAI, 2025。努力程度数据使用 500 个问题的切分，每个设置运行一到三次，运行于 2026 年 8 月 3 日，默认数据点合并了 2026 年 7 月 26 日至 27 日的两次运行。[编排器策略](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#orchestrator-strategy-delegate-bulk-work)中对具有长成本尾部的常规工作的测量，使用了在确定协议时模型能够独立可靠解决的 10 个问题，取自一个 26 个问题的切片，包括 50 次委派运行（每个问题 5 次；2026 年 8 月 1 日至 2 日）和 70 次模型单独运行（50 次来自 2026 年 8 月 2 日至 3 日；20 次为 2026 年 7 月 12 日至 13 日及 8 月 1 日的存档运行），期望每次运行成本为 $6.45 USD 对比 $11.99 USD；委派数据的测量区间约为 20%。这 10 个问题是委派配置（一个 Claude Fable 5 编排器加一个 Claude Sonnet 5 工作者）在全部 5 次运行中都答对、且单独的 Claude Fable 5 在两次存档运行中都答对的问题；委派运行就是这些运行本身，只有单独的 Claude Fable 5 在这十个问题上重新运行。在其全部 70 次运行（存档的和新的）中，单独的 Claude Fable 5 答对了 68 次。在这些问题上，委派配置的平均成本约为单独的 Claude Fable 5 的一半（即前面给出的 $6.45 USD 和 $11.99 USD），在第 90 百分位约为其三分之一（每次运行 $12 USD 对比 $33 USD），其最昂贵的一次运行成本为 $42 USD，对比 $84 USD；那次 $84 USD 的运行也答错了。在完整且更难的 BrowseComp 集合上，单独的 Claude Fable 5 以低 22% 到 30% 的成本达到了在该集合上测量的编排器配置的准确率。

5. **智能体架构扩展：** Kim et al., “Towards a Science of Scaling Agent Systems,” arXiv:2512.08296, 2025。独立的外部研究，仅引用其关于委派何时不划算这一发现的方向，不引用任何数据。

6. **DeepWideSearch：** “DeepWideSearch: Benchmarking Depth and Width in Agentic Information Seeking,” arXiv:2510.20168, 2025。220 个问题涵盖 15 个领域，每个问题都结合了多行收集与多跳检索；在该基准测试的常设行集上测量，每种配置运行 3 次，运行于 2026 年 8 月 2 日（单工作者团队数据点运行于 2026 年 7 月 26 日至 27 日）。使用 Claude Fable 5 时，`low` 努力程度以低 29% 的成本达到了配备一个 Claude Sonnet 5 工作者的编排器的水平，每个问题耗时 4.5 分钟，而默认设置下为 7.9 分钟。

7. **DeepResearch Bench II：** Li et al., “DeepResearch Bench II: Diagnosing Deep Research Agents via Rubrics from Expert Report,” arXiv:2601.08536, 2026。

   * **测量内容：** 其 22 个领域的 132 个研究任务按源自专家的二元评分标准评分。于 2026 年 11 月 3 日在一个从其所有主题中抽取的 50 个任务子集上测量，其中一半为英文、一半为中文，每个任务尝试一次，每个设置运行 3 次：`high` 下的 Claude Sonnet 5.5、`medium` 下的 Claude Opus 5.5，以及 `low`、`medium` 和 `high` 下的 Claude Fable 5.1 和 Claude Fable 5.2。所有设置都在 Messages API 上运行，采用自适应思考，三个已发布模型开启了生产安全分类器。Claude Fable 5.2 在其发布前运行，覆盖 50 个任务中的 46 个，不包括四个生物学任务。
   * **工具：** 智能体使用评估框架基于 Anthropic 网络索引托管的搜索和抓取工具，该索引不含 2026 年 3 月 17 日之后的任何内容，且智能体没有 shell。超过 120 个字符的搜索查询会在最后一个完整单词处截断，并告知智能体这一点：Claude Fable 5.2 有 57% 到 59% 的搜索被截断，Claude Sonnet 5.5 为 49%，其他设置为 26% 到 31%。在同一任务内，Claude Fable 5.2 被截断搜索较多的运行得分并不更低。之后于 2026 年 11 月 6 日和 7 日又对 Claude Fable 5.2 和 Claude Opus 5.5 进行了更多运行，分为两组，每组每个模型三次。在一组中，工具像之前一样截断较长的查询；在另一组中则拒绝这些查询。在两组中都被评分的 43 个任务上，采用截断时，`high` 下的 Claude Fable 5.2 得分比 Claude Opus 5.5 低 0.9 分（95% 区间为高 1.0 分到低 2.7 分）。该区间包含 11 月 3 日在 44 个任务上测得的 1.9 分差距。采用拒绝时，其得分低 1.9 分（0.0 到 3.7）。这一差距比采用截断时大 1.0 分（小 0.8 分到大 2.8 分），并非可测量的变化。约四分之一的抓取返回错误。每个任务的来源文章以及该基准测试自身的论文和数据集均被过滤掉。
   * **评分方式：** Claude Opus 4.6 按该基准测试的评分标准协议进行评判；原始基准使用不同的评判模型，而 Anthropic 的评判模型可能偏好自家风格。分数为每个设置都作答且评判模型完整评分的 44 个任务上的平均值。其余六个任务中：四个生物学任务，Claude Sonnet 5.5 在每次运行中都作答了，而安全分类器中止了 Claude Fable 5.1 的每次尝试（36 次）以及 Claude Opus 5.5 12 次中的 11 次；一个任务是计算机安全研究综述，分类器中止了 `low` 下 Claude Fable 5.1 的一次尝试以及 `high` 下 Claude Fable 5.2 三次中的两次；另一个任务上，Anthropic 在其自身评估中添加的一项安全检查中止了 Claude Sonnet 5.5 的一次尝试。
   * **分数：** 在 44 个任务上以及在全部 50 个任务上（被拒绝或被中止的尝试计为 0，一次已作答但评判模型未完整评分的尝试保留其部分得分）：Claude Sonnet 5.5 为 57.2% 和 56.8%；Claude Opus 5.5 为 59.6% 和 54.6%；Claude Fable 5.1 在 `low` 下为 60.2% 和 54.7%，在 `medium` 下为 62.1% 和 56.6%，在 `high` 下为 62.5% 和 57.0%。Claude Fable 5.2 在 44 个任务上：`low` 下为 54.5%，`medium` 下为 55.0%，`high` 下为 57.7%。在 44 个任务上逐任务的差异（四舍五入前计算，附 95% 区间）：`high` 下的 Claude Fable 5.1 比 `low` 高 2.3 分（1.3 到 3.3）；Claude Opus 5.5 比 Claude Sonnet 5.5 高 2.4 分（1.0 到 3.8）；`high` 下的 Claude Fable 5.1 比 Claude Opus 5.5 高 2.8 分（1.4 到 4.3），比 Claude Sonnet 5.5 高 5.2 分（4.0 到 6.5）；`medium` 下的 Claude Fable 5.1 比 `low` 高 1.9 分（0.6 到 3.1）；`high` 下的 Claude Fable 5.2 比 `low` 高 3.3 分（1.7 到 4.9），`medium` 下比 `low` 高 0.6 分（-0.6 到 1.8）；`high` 下的 Claude Fable 5.2 相对于 Claude Sonnet 5.5 的差异为 0.5 分（-1.2 到 2.2），相对于 Claude Opus 5.5 为 -1.9 分（-3.6 到 -0.2）；Claude Fable 5.2 相对于 Claude Fable 5.1 的差异在 `low` 下为 -5.8 分（-8.0 到 -3.5），在 `medium` 下为 -7.0 分（-9.3 到 -4.8），在 `high` 下为 -4.7 分（-6.4 到 -3.1）。在全部 50 个任务上：Claude Opus 5.5 相对于 Claude Sonnet 5.5 的差异为 -2.2 分（-7.6 到 3.2）；`high` 下的 Claude Fable 5.1 相对于 Claude Sonnet 5.5 的差异为 0.2 分（-5.5 到 5.9）。
   * **定价方式：** 每个任务的成本是模型自身令牌按标价计算的费用，数据来自各次尝试的使用记录，涵盖全部 50 个任务和每次尝试，再对三次运行取平均：Claude Sonnet 5.5 为 $1.11 USD，Claude Opus 5.5 为 $1.61 USD，Claude Fable 5.1 为 $3.55 USD、$4.44 USD 和 $5.69 USD。Claude Fable 5.2 的成本基于其 46 个任务：$1.49 USD、$2.16 USD 和 $4.82 USD；在相同的 46 个任务上，其他五个设置的成本为 $1.13 USD、$1.69 USD、$3.85 USD、$4.82 USD 和 $6.18 USD。在四个生物学任务上，Claude Fable 5.1 的每次尝试都在其第一个请求时被中止，尝试的平均成本在 Claude Fable 5.1 上约为一美分，在 Claude Opus 5.5 上为 $0.74 USD。仅在 44 个任务上，成本分别为 $1.13 USD、$1.68 USD、$3.85 USD、$4.77 USD 和 $6.10 USD，Claude Fable 5.2 为 $1.50 USD、$2.20 USD 和 $4.86 USD。[比较模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#compare-models-on-cost-per-task)中的成本比较（1.5、4.3 和 2.9 倍；比 `low`、`medium` 和 `high` 下的 Claude Fable 5.1 低 61%、54% 和 20%）以及[调整努力程度](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#tune-effort)中的成本比较（3.2 倍）与旁边的分数一样，都基于这 44 个任务。成本仅涵盖模型令牌；评估框架的工具不产生额外费用。智能体每个任务进行 12 到 33 次搜索（计入被拒绝的尝试）。
   * **图表：** 缓存图表和努力程度图表显示的是 2026 年 10 月 22 日 Claude Fable 5.1、Claude Opus 5.5 和 Claude Sonnet 5.5 在相同努力程度下的运行。缓存图表将这些运行的请求按所有输入令牌均以未缓存费率计价重新定价。缓存断点由评估框架设置。
   * **可比性：** 这些分数不能与该基准测试的排行榜进行比较。

8. **内部缺陷搜索基准测试：**

   * **测量内容：** Anthropic 内部测试，针对超过一个上下文窗口的工作：一个约 2,200 万令牌的语料库，来自 14 个公开 Python 软件包源码，分为 201 个部分，植入了 130 个缺陷，采用确定性评分。每种配置在 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 上运行十次：170 次运行中有 51 次于 2026 年 10 月 27 日至 29 日运行，69 次于 2026 年 11 月 2 日至 3 日运行，Claude Fable 5.2 的 50 次于其发布前的 2026 年 11 月 3 日至 4 日运行。
   * **配置：** 单独使用时，每个模型对每个部分运行一个会话，每个会话都获得了此前已有的发现：Claude Fable 5.2、Claude Fable 5.1 和 Claude Opus 5.5 分别在 `low`、`medium` 和 `high` 下运行，Claude Sonnet 5.5 和 Claude Haiku 5.5 在各自模型的默认努力程度（`high` 和 `medium`）下运行。使用子智能体时，Claude Fable 5.2、Claude Fable 5.1 或 Claude Opus 5.5 在 `high` 下作为编排器运行，通过[多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)（`multiagent_20261001` 类型）：一个预定义智能体作为子智能体，采用其模型的默认努力程度，每个部分调用一次，最多同时运行 25 个，内联智能体和动态工作流均已关闭。每种配置最后都以另一个不使用子智能体的会话结束，由该会话汇总发现。思考按平台默认方式运行：开启，并由模型选择何时思考。
   * **工具：** 每个智能体都使用基准测试自身的 shell 和文件工具，运行在一台隔离的虚拟机中。在任何智能体启动之前，评估从该机器中移除了语料库软件包的已安装副本，并确认了从该机器发出的连接均未成功。
   * **结果：** 平均 F1（范围）和每次运行的平均成本。单独使用 Claude Haiku 5.5：0.577（0.550 到 0.606），$25 USD。单独使用 Claude Sonnet 5.5：0.760（0.669 到 0.788），$103 USD。单独使用 `low` 下的 Claude Opus 5.5：0.489（0.437 到 0.551），$38 USD；`medium` 下：0.827（0.797 到 0.850），$200 USD；`high` 下：0.850（0.829 到 0.876），$236 USD。单独使用 `low` 下的 Claude Fable 5.1：0.829（0.807 到 0.841），$527 USD；`medium` 下：0.839（0.817 到 0.856），$603 USD；`high` 下：0.847（0.829 到 0.861），$716 USD。单独使用 `low` 下的 Claude Fable 5.2：0.866（0.852 到 0.884），$357 USD；`medium` 下：0.877（0.863 到 0.884），$378 USD；`high` 下：0.909（0.896 到 0.925），$563 USD。Claude Opus 5.5 搭配 Claude Sonnet 5.5 子智能体：0.831（0.814 到 0.850），$139 USD；搭配 Claude Haiku 5.5 子智能体：0.677（0.640 到 0.718），$47 USD。Claude Fable 5.1 搭配 Claude Sonnet 5.5 子智能体：0.825（0.812 到 0.858），$172 USD；搭配 Claude Haiku 5.5 子智能体：0.687（0.660 到 0.723），$88 USD。Claude Fable 5.2 搭配 Claude Sonnet 5.5 子智能体：0.833（0.795 到 0.856），$165 USD；搭配 Claude Haiku 5.5 子智能体：0.716（0.683 到 0.773），$85 USD。在其他模型得分高于 0.80 的配置中，任意两个配置的范围都相互重叠。平均 F1 的差异及 95% 区间：搭配 Claude Sonnet 5.5 子智能体的 Claude Opus 5.5 得分比 `medium` 下单独使用的 Claude Opus 5.5 高 0.4 分（低 1.1 分到高 1.9 分），比 `high` 下单独使用时低 1.9 分（0.6 到 3.2），比单独使用的 Claude Sonnet 5.5 高 7.1 分（4.6 到 9.7）。单独使用时，Claude Opus 5.5 在 `high` 下的得分比在 `medium` 下高 2.3 分（0.7 到 3.9）。单独使用的 Claude Fable 5.1 在 `low` 下的得分比在 `high` 下低 1.8 分（0.8 到 2.8）；在 `high` 下，其得分比 `medium` 下单独使用的 Claude Opus 5.5 高 2.0 分（0.7 到 3.4），比 `high` 下单独使用的 Claude Opus 5.5 低 0.3 分（低 1.5 分到高 1.0 分）。搭配 Claude Sonnet 5.5 子智能体时，Claude Fable 5.1 的得分比 Claude Opus 5.5 低 0.6 分（低 1.9 分到高 0.6 分）。与 `medium` 和 `high` 下单独使用的 Claude Opus 5.5 相比，单独使用的 Claude Fable 5.2 在每个级别下得分都更高（差异在四舍五入前计算）：在 `low` 下分别高 3.9 分（2.4 到 5.3）和 1.6 分（0.3 到 2.9），成本为其 1.8 倍和 1.5 倍；在 `medium` 下分别高 5.0 分（3.6 到 6.3）和 2.7 分（1.5 到 3.8），成本为 1.9 倍和 1.6 倍；在 `high` 下分别高 8.2 分（6.8 到 9.6）和 5.9 分（4.7 到 7.1），成本为 2.8 倍和 2.4 倍。它的得分高于相同级别下单独使用的 Claude Fable 5.1：`low` 下高 3.6 分（2.6 到 4.7），`medium` 下高 3.8 分（2.8 到 4.8），`high` 下高 6.1 分（5.3 到 7.0），成本分别低 32%、37% 和 21%。它在 `low` 下的得分比在 `high` 下低 4.3 分（3.3 到 5.3），成本低 37%。搭配 Claude Sonnet 5.5 子智能体时，Claude Fable 5.2 的得分比 Claude Opus 5.5 高 0.2 分（低 1.3 分到高 1.7 分），成本高 19%；其得分比 `low` 下单独使用的 Claude Fable 5.2 低 3.2 分（1.8 到 4.7），比 `medium` 下低 4.3 分（3.0 到 5.7），比 `high` 下低 7.5 分（6.1 到 9.0），成本分别低 54%、56% 和 71%。单独使用 `high` 下 Claude Fable 5.2 的一次运行提前结束且没有得分，已被替换。Claude Opus 5.5 组合方案的成本比 `medium` 下单独使用的 Claude Opus 5.5 低 31%，比 `high` 下单独使用时低 41%。单独使用时，Claude Opus 5.5 在 `medium` 下的成本比在 `high` 下低 15%。单独使用的 Claude Sonnet 5.5 在 11 月的全部七次运行得分均低于 10 月的三次（0.669 到 0.775，对比 0.783 到 0.788）；原因未知，所有运行均已计入。Claude Fable 5.1 搭配 Claude Haiku 5.5 子智能体在 10 月每次运行的成本为 $85.77 USD，在 11 月为 $89.06 USD。
   * **定价方式：** 按平台标价计算的每次运行中各会话的成本：使用提示缓存的令牌，加上会话运行时间。在 `high` 下 Claude Opus 5.5 的运行中，有五次仅按其令牌数量、以 5 分钟缓存价格定价，因为这些运行各自有一部分使用了 1 小时缓存时长，而该时长的缓存写入成本更高。Claude Fable 5.2 自身的令牌按其数量、以标价和 5 分钟缓存写入定价；在其单独运行中，会话运行时间（不到每次运行成本的 0.2%）未计入。对工具调用的额外检查（评估安全设置的一部分）的成本未计入。

9. **GPQA Diamond：** Rein et al., “GPQA: A Graduate-Level Google-Proof Q\&A Benchmark,” 2023。198 个问题的 Diamond 子集，由模型对照参考答案评分，顾问令牌按请求计量。Claude Haiku 5.5 和 Claude Sonnet 5.5（各自单独运行以及配备 Claude Opus 5.5 顾问运行）以及单独运行的 Claude Opus 5.5，于 2026 年 10 月 7 日每种配置运行两次，均关闭[服务器端回退](https://platform.claude.com/docs/zh-CN/build-with-claude/refusals-and-fallback#server-side-fallback)，采用各模型的默认努力程度，拒绝回答计为错误。平台安全检查在单独运行的 Claude Haiku 5.5 和 Claude Sonnet 5.5 上每次运行拒绝了一道生物学问题，在配备顾问的 Haiku 5.5 上两次运行中有一次拒绝了一道，在配备顾问的 Sonnet 5.5 上没有拒绝，在 Opus 5.5 上每次运行拒绝六道；Opus 5.5 在将这些计为错误时得分 91%，在其回答的问题上得分 93.5%。单独运行的 Claude Haiku 5.5 得分 85%，比 Opus 5.5 低 6 分。顾问配置添加了该工具，但没有要求进行咨询的提示，两个执行器在任何问题上都没有调用顾问。配对的得分比单独的执行器高 0.5 分（Claude Haiku 5.5）和 1.0 分（Claude Sonnet 5.5），均在运行间噪声范围内。

10. **DeepSWE：** Datacurve, “DeepSWE: Measuring Frontier Coding Agents on Original, Long-Horizon Engineering Tasks,” arXiv:2607.07946, 2026。该集合包含五种语言的 113 个原创任务，配有基于程序的验证器。配对各运行两次，运行于 2026 年 8 月 7 日，顾问令牌按请求计量，使用客户端顾问循环而非 advisor 工具，计费方式相同。单模型努力程度扫描为单次运行，根据令牌数定价，这是一种考虑缓存的近似。每任务成本为运行总额除以 113。在此基准测试上，配备顾问的低努力程度 Claude Sonnet 5 执行器比单独的同一执行器高 23 分。

11. **内部智能体编码基准测试：**

    * **测量内容：** Anthropic 内部测试：370 个代码仓库任务，由仓库自身的测试评分。API 数据使用一个普通 API 智能体和 128,000 令牌的输出上限测量，每种配置运行一次：单独的 Opus 5 在默认努力程度下于 2026 年 8 月 9 日至 10 日运行，在 `low` 和 `medium` 下于 2026 年 8 月 10 日运行；单独的 Claude Fable 5.1 在五个显式设置的努力程度值下于 2026 年 8 月 20 日运行（图表显示其中三个）；配对于 2026 年 8 月 24 日至 25 日运行。
    * **8 月的运行：** 8 月的运行中，配对和 Claude Opus 5 对照组每个任务尝试五次，其他数据点尝试一次。
    * **定价方式：** 成本按每次尝试计算。成本按客户组织的计量方式定价：每个智能体循环请求的先前提示按缓存读取计价，其新令牌按 5 分钟缓存写入计价，数据来自运行自身的使用记录；每次顾问调用不使用缓存，按其记录的令牌计价，全部按标价计算。
    * **单独的 Claude Opus 5.5：** 单独的 Claude Opus 5.5 于 2026 年 9 月 19 日至 20 日在全部 370 个任务上运行：在其默认努力程度（`medium`）和 `high` 下每个任务尝试五次，在 `low` 和 `xhigh` 下尝试一次（由于一次设置检查失败，每个设置下 370 个任务中有 369 个被评分）。单独的 Claude Opus 5.5 在 `low` 下解决了 74.8% 的尝试，每次尝试 $0.35 USD；`medium` 下为 86.6%，$0.84 USD；`high` 下为 88.4%，$1.38 USD；`xhigh` 下为 91.1%，$4.11 USD。
    * **配备 Claude Fable 5.1 顾问的 Claude Opus 5.5，运行：** `high` 下的 Claude Opus 5.5 执行器以已发布的 Claude Fable 5.1 作为顾问（8 月的运行使用的是预发布快照），在相同日期每个任务尝试五次；一个任务未通过设置检查，因此共评分 1,845 次尝试。顾问因负载而被拒绝的 279 次尝试被重新运行，咨询超时的尝试则被保留，与 8 月相同。
    * **单独的 Claude Fable 5.1 和 Claude Fable 5.2：** 作为比较，已发布的 Claude Fable 5.1 在其默认值 `high` 下单独运行，得分 85.7%（370 个任务中的 317 个），于 2026 年 10 月 7 日每个任务尝试一次。单独的 Claude Fable 5.2 在全部 370 个任务上运行，在 `low`、`medium` 和 `high` 下每个任务各尝试一次（由于一次设置检查失败，`medium` 下 370 个任务中有 369 个被评分）。
    * **咨询：** 8 月的配对平均每次尝试约咨询顾问两次；Claude Opus 5.5 配对请求了 1.39 次，收到了 1.35 次。
    * **配备 Claude Fable 5.1 顾问的 Claude Opus 5.5，结果：** Claude Opus 5.5 配对得分 90.1%，每次尝试 $2.92 USD：与默认值 `medium` 下的 Claude Opus 5.5 相比，即以约 3.5 倍的花费换取 3.5 分。延迟成本就是咨询本身：每个任务约多出一到两次前沿模型调用，每次都位于任务的关键路径上。使用 Claude Opus 5.5 执行器时，建议几乎没有节省执行器成本：每次尝试 $1.36 USD，而 `high` 下单独的 Claude Opus 5.5 为 $1.38 USD，同时咨询花费了 $1.55 USD。
    * **8 月的配对：** 8 月，在 Claude Opus 5 执行器和 Claude Fable 5.1 顾问的配置下，执行器每次尝试比默认设置下单独的 Claude Opus 5 少花 $1.26 USD，而咨询花费 $2.47 USD，因此这部分节省抵消了约一半的建议成本。该 8 月配对是当时测量到的最准确配置，每次尝试 $6.21 USD。
    * **单独的 Claude Sonnet 5.5，运行：** 单独的 Claude Sonnet 5.5 于 2026 年 9 月 24 日在全部 370 个任务上运行，采用其默认努力程度（`high`）和 `medium`，每个任务尝试一次，输出上限同样为 128,000 个令牌，该上限未截断其任何尝试。`medium` 下全部 370 个任务均被评分，`high` 下由于一次设置检查失败，370 个中有 369 个被评分。其成本按客户组织的计量方式定价。
    * **单独的 Claude Sonnet 5.5，结果：** 在没有顾问的情况下，Claude Sonnet 5.5 在其默认值 `high` 下得分 77.8%，每次尝试 $0.33 USD，比默认设置下的 Claude Opus 5.5 低约 9 分，成本约为其五分之二；在 `medium` 下得分 62.7%，$0.16 USD。在 `medium` 下，Claude Sonnet 5.5 的得分比其默认值 `high` 低约 15 分，每次尝试成本约为其 48%。
    * **比较模型：** 在[比较模型](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#compare-models-on-cost-per-task)中，默认设置下的 Claude Opus 5.5 每次尝试的成本不到 `medium` 下 Claude Fable 5.2 的三分之二，后者为单次运行。在 `low` 下，Fable 5.2 同样得分 88.9%，每次尝试 $1.05 USD。在 `high`（其在 Messages API 上的默认值）下，它得分 93.5%，每次尝试 $5.39 USD。

12. **内部代码仓库任务基准测试（上限测量）：**

    * **测量内容：** 一个单独的 Anthropic 内部集合，包含约 130 个代码仓库任务，运行于 2026 年 8 月 20 日（Claude Fable 5.1）和 2026 年 9 月 19 日（Claude Opus 5.5，采用其默认努力程度 `medium`），使用普通 API 智能体循环，每个任务尝试一次。Claude Fable 5.2 的运行使用相同的循环，每个任务尝试一次；与 Claude Opus 5.5 的运行一样，在这些运行中，达到上限的轮次会终止其尝试。
    * **定价方式：** 每个已解决任务的成本，对于 Claude Fable 5.1 按客户计费方式定价，使用 5 分钟提示缓存；对于 Claude Opus 5.5，则根据其运行记录的令牌数按标价定价。
    * **Claude Fable 5.1 和 Claude Fable 5.2 的运行：** Claude Fable 5.1 的运行在显式设置的默认努力程度下，每个上限 135 个任务。Claude Fable 5.2 的运行在显式设置的 `high`（其在 Messages API 上的默认值）下，每个上限 134 个任务；在 Claude Fable 5.1 和 Claude Opus 5.5 的运行之后，有一个任务从集合中撤出。另有一个任务在每次运行中、模型开始之前都未通过基准测试自身对其参考解决方案的检查，因此被排除，每个上限评分 133 个任务。16,384 令牌的数据为两次运行的平均值（19.5% 和 21.1%）；64,000 和 128,000 的数据为单次运行（77.4% 和 78.2%）。在 64,000 下，有一个任务在第一次运行时未通过该检查并被重新运行；它通过了检查并被评分。在 16,384 或 64,000 下，没有任何 Fable 5.2 尝试引发安全拒绝。在 128,000 下，有一次尝试引发了安全拒绝，另一次在完成前被评估环境中的安全监控器中止；两者都计为失败。被中止的尝试到那时所做的更改通过了测试；若将其排除，128,000 下的结果为 78.8%。表中 Claude Fable 5.2 和 Claude Opus 5.5 的成本按模型标价对每次尝试记录的令牌用量定价。两个模型的结果由不同版本的基准测试评分代码评分。
    * **Claude Opus 5.5 的运行：** Claude Opus 5.5 的 16,384 令牌数据为两次运行的平均值（分别评分 134 和 135 个任务），其 64,000 和 128,000 数据为单次运行（各 135 个任务）；每次 16,384 令牌运行中有两次尝试、64,000 和 128,000 令牌运行中各有一次尝试以安全拒绝结束，计为失败。Claude Opus 5.5 在 16,384 下解决了 57.6%（58.2% 和 57.0%），在 64,000 下为 75.6%，在 128,000 下为 68.9%。其尝试在 64,000 或 128,000 下均未被截断，因此上限无法解释这两次运行之间的差异。
    * **被上限终止的尝试：** 在被 16,384 令牌上限终止的 66 次 Claude Opus 5.5 尝试中只有 1 次、在 117 次 Claude Fable 5.1 尝试中只有 9 次仍然通过。在 226 次 Claude Fable 5.2 尝试中也有 20 次仍然通过。
    * **每次尝试的成本：** 按标价计算，Claude Opus 5.5 每次尝试的成本在 16,384 下为 $3 USD，在 64,000 下为 $4 USD；按客户计费方式计算，Claude Fable 5.1 分别为 $8 USD 和 $13 USD。按标价计算，Claude Fable 5.2 分别为 $6 USD 和 $19 USD。
    * **SWE-bench Pro 上限数据：** SWE-bench Pro 上限数据为默认努力程度下每个上限一次 Claude Fable 5.1 运行，运行于 2026 年 8 月 26 日，使用从参考文献 3 的 482 个问题集中分层抽取的 100 个问题子集，不可与其分数比较；在默认努力程度下，16,384 和 64,000 下的运行各解决了 100 个问题中的 94 个。
    * **轮次长度：** 该部分点图中的每轮分布来自 128,000 下的 Claude Opus 5.5 和 Claude Fable 5.1 运行：没有任何 Opus 5.5 轮次达到上限（最长约 61,000 个令牌，0.56% 的轮次超过 16,384），有一个 Fable 5.1 轮次达到 128,000（0.46% 的轮次超过 16,384）。上限为 128,000 时，Claude Fable 5.1 的 14,331 个轮次中有 2 个超过 64,000 个令牌，Claude Opus 5.5 则没有。上限为 128,000 时，Claude Fable 5.2 的 17,750 个轮次中有 303 个（1.7%）超过 16,384 个令牌，7 个超过 64,000 个令牌，其中计入了被取消并重新运行的尝试的轮次。

13. **Chartography：** Surge AI, “Chartography,” 2026。

    * **测量内容：** 完整发布的 100 个问题集，测量于 2026 年 8 月 6 日和 9 日（单独的 Claude Opus 5）以及 2026 年 9 月 20 日（Claude Opus 5.5），使用 Anthropic 在 Claude Managed Agents 上的实现（标准云沙箱；顾问配置使用 Managed Agents 顾问）。
    * **评分方式：** 由 Claude Sonnet 4.6 代替参考评判模型进行评分，且基准测试在使用工具的情况下运行，因此这里的分数可在配置之间比较，但不可与已发布的排行榜比较。它们也不可与 Claude Opus 5.5 系统卡中的 Chartography 结果比较，后者使用不同的评分器并以 `max` 努力程度运行。每种配置运行两次（Claude Opus 5.5 为三次），合并计算；运行间差异最多达 10 分。
    * **定价方式：** 成本是客户日常运行该智能体时被收取的费用：每个图表的第一个请求从缓存中读取智能体共享的系统提示和工具，就像同一智能体的另一个会话在前 5 分钟内运行过时那样。单独运行一个图表时，使用 Claude Opus 5 或 Claude Opus 5.5 约多花 $0.03 USD，使用 Claude Fable 5.1 约多花 $0.12 USD。8 月的数据根据运行的使用记录按此方式重新定价。成本不包括沙箱时间，沙箱时间使 8 月运行的成本增加不到 1%。
    * **单独的 Claude Fable 5.1，8 月的运行：** 单独的 Claude Fable 5.1 的运行来自 2026 年 8 月 24 日，采用平台的发布服务设置，每个设置运行两次；有六次尝试触及 15 分钟的会话上限，得分为 0，每次运行中有两个图表在安全拒绝后由 Claude Opus 5 作答。
    * **配备 Claude Fable 5.1 顾问的 Claude Opus 5：** 配备 Claude Fable 5.1 顾问的 Claude Opus 5 低努力程度执行器于 2026 年 8 月 30 日在相同设置下运行了两次（63.0 和 67.0，平均 65.0，每个图表 $0.47 USD；每次运行中 88% 的任务咨询了顾问，其 219 条回复中有 4 条改由 Claude Opus 5 给出，每次都是在生产安全过滤器阻止了顾问自身的回复之后）。该配对在运行间噪声范围内与 `medium` 下单独的 Claude Fable 5.1 持平（65.0 对 67.5），每任务成本约为其 1.8 倍：在大多数任务上都会触发的咨询，比直接运行更强的模型成本更高。
    * **Claude Opus 5.5 的运行：** Claude Opus 5.5 在 `low` 下运行，关闭服务器端回退，并由安全分类器评判每次工具调用：单独运行三次（70、68 和 68），配置 Claude Fable 5.1 顾问运行三次（59、63 和 63），其中它在 300 个任务中的 1 个上咨询了顾问。
    * **`low` 努力程度下的结果：** 在这个图表阅读基准测试上，`low` 下的 Claude Opus 5.5 得分 68.7，每个图表约 $0.03 USD，而 `low` 下的 Claude Fable 5.1 为 62.5，$0.15 USD，`low` 下的 Claude Opus 5 为 49，$0.16 USD。
    * **单独的 Claude Fable 5.1，10 月的运行：** 为与 Claude Opus 5.5 配对进行比较，单独的 Claude Fable 5.1 在其默认值 `high` 下于 2026 年 10 月 7 日运行了三次，使用与 Claude Opus 5.5 运行相同的实现和平台设置：82、79 和 79，平均 80.0。
    * **咨询率：** 早期配对的咨询率比较来自于 2026 年 8 月 10 日至 11 日在 Messages API 上使用容器工具集重新运行相同配置。

14. **客服台提示审计评估：**

    * **测量内容：** 一个由 Anthropic 构建的包含 44 个支持工单的集合，采用确定性评分，于 2026 年 8 月初运行，并于 2026 年 8 月 8 日报告，使用六个系统提示，每个提示都在同一个干净提示的基础上添加一种在为 Claude Opus 4.8 和 Claude Sonnet 4.6 编写的提示中常见的模式。
    * **图表：** 每个图表点是三种情况之一（旧模型、使用相同提示的新模型、审计后的新模型），在六个提示和 44 个工单上取平均。
    * **结果，Claude Opus 5：** 在 Claude Opus 5 上，为 Claude Opus 4.8 编写的提示每个工单的成本比在 Claude Opus 4.8 上高 36%，准确率没有变化。审计后，它们在 Claude Opus 5 上的成本比审计前低 14%（根据未经四舍五入的平均值；图表中四舍五入后的成本为每工单 3.43 和 2.93 美分，得出 14.6%），且 Claude Opus 5 正确解决了 97% 的工单，高于之前的 92%。按模式划分，在 Claude Opus 5 上：删除"verify twice"（验证两次）使每工单成本降低三分之一，删除"be maximally thorough"（尽可能详尽）的降幅几乎相同；删除已停用的思考设置、相互矛盾的规则和手工编写的草稿区，各自恢复了 7 到 11 分的准确率。
    * **结果，Claude Sonnet 5：** 在从 Claude Sonnet 4.6 到 Claude Sonnet 5 的迁移中，审计在准确率不变的情况下使每工单成本降低了 14%。
    * **注意事项：** Opus 5 准确率提升的 95% 置信区间为 3 到 8 分；Sonnet 的准确率差异在噪声范围内。

15. **数据文件问题集：** 一个由 Anthropic 构建的集合，包含针对某公开酒类销售 CSV 的 1,862 行切片的 25 个聚合问题，真实答案由 pandas 计算，采用精确匹配评分，在 Claude Sonnet 5 和 Claude Opus 5 上运行，禁用思考（上下文内方案在默认设置下无法完成）、输出上限 4,000 个令牌、不使用提示缓存，每种配置运行三次，运行于 2026 年 8 月 19 日。文件方案通过 Files API 上传 CSV 并使用 `code_execution_20260120` 工具。上下文内方案的三次 Claude Sonnet 5 运行分别正确回答了 25 个问题中的 6、7 和 7 个；图表标签和正文给出的是其平均值 6.67 取整到最接近的问题数，图表中的点绘制在平均值 26.7% 处。

16. **缓存时长测量：**

    * **测量内容：** [精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)中的 20 个问题分诊任务，于 2026 年 8 月 23 日在 Claude Sonnet 5 上运行，于 2026 年 9 月 19 日和 20 日在 Claude Opus 5.5 上以其默认努力程度（`medium`）和 `high` 运行，在 Messages API 上使用相同的评估框架（对于 Claude Opus 5.5，使用其移植版本，发送相同的请求体），Claude Opus 5.5 单元格的 `max_tokens` 提高到 4,096，并在随机选择的一部分轮次之前插入暂停（在两个模型上，对全部 20 个问题分别设置无暂停、5%、10% 以及每轮 6 分钟暂停，另在 Claude Sonnet 5 上设置每轮 2 分钟暂停；在两个模型上，对 5 个问题的子集设置 20 分钟和 45 分钟暂停）。本页中 Claude Opus 5.5 的数据涵盖两个努力程度。
    * **其他模型和日期：** 下文中 Claude Opus 5 的保活数据来自 2026 年 8 月 23 日的同一任务，`max_tokens` 提高到 4,096，采用相同的暂停计划，但不包括 2 分钟和 45 分钟暂停。Claude Sonnet 5.5 运行了与 Claude Sonnet 5 相同的测量（相同的暂停计划和三种缓存设置，每个单元格运行三次），使用为 Claude Opus 5.5 移植的评估框架，采用其默认努力程度 `high`，于 2026 年 9 月 24 日运行。
    * **评分和定价方式：** 每个单元格运行三次，成本根据每个响应的 `usage` 字段按标价计算（对于 Claude Opus 5.5，每百万令牌输入 $4 USD、5 分钟写入 $5 USD、1 小时写入 $8 USD、缓存读取 $0.20 USD、输出 $20 USD；Claude Sonnet 5 在一个 Anthropic 内部组织上运行，其使用量的计量方式与客户组织相同），准确率对照相同的黄金标签计算。
    * **运行和定价方式，Claude Sonnet 5.5：** Claude Sonnet 5.5 在会话请求上将 `max_tokens` 设为 4,096，在保活请求上设为 1，在一个使用量计量方式与客户组织相同的 Anthropic 内部组织上运行，并按其当前标价定价（每百万令牌输入 $2 USD、5 分钟写入 $2.50 USD、1 小时写入 $4 USD、缓存读取 $0.10 USD、输出 $10 USD）。除了更高的 `max_tokens`（没有任何轮次达到该值）之外，有一项输入与 Claude Sonnet 5 的运行不同：任务中唯一的动画 GIF 以其第一帧的 PNG 形式发送，这是 API 使用的唯一一帧（[视觉](https://platform.claude.com/docs/zh-CN/build-with-claude/vision)），两者计为相同数量的输入令牌。
    * **盈亏平衡点：** 盈亏平衡点在 Claude Sonnet 5 上约为 3.3% 的轮次，在 Claude Opus 5.5 上为 3.1% 到 3.2%：这是每个会话盈亏平衡比例的中位数，由成本模型根据该会话逐轮的上下文大小计算，涵盖全部 45 个 Claude Sonnet 5 的二十问题会话，以及每个努力程度下 36 个 Claude Opus 5.5 的二十问题会话（每种暂停计划都在完整任务上、在全部三种缓存设置下各运行三次；5 个问题的单元格不包括在内）。Claude Sonnet 5.5 上的盈亏平衡点约为 3.1% 的轮次，为其 45 个二十问题会话的中位数。缓存时长图表绘制了这些 Claude Sonnet 5.5 会话，每个点三个会话。
    * **5 分钟和 1 小时设置，Claude Sonnet 5 和 Claude Opus 5.5：** 在 5% 单元格中，5 分钟和 1 小时设置在 Claude Sonnet 5 上持平，因为该次抽样的暂停落在较小的前缀上；在 Claude Opus 5.5 上两者几乎持平。在 Claude Sonnet 5 上，按其每百万令牌 $0.20 USD 的缓存读取价格，在 5 分钟设置下，无暂停、5% 和 10% 轮次暂停时每个会话的成本分别为 $0.60 USD、$0.69 USD 和 $0.99 USD，在 1 小时设置下分别为 $0.70 USD、$0.70 USD 和 $0.71 USD。
    * **保活请求，Claude Sonnet 5 和 Claude Opus 5：** Anthropic 于 2026 年 8 月 23 日在 Claude Sonnet 5 和 Claude Opus 5 上、并在上述运行中在 Claude Opus 5.5 上测量了刷新 5 分钟缓存的保活请求，这些请求始终以 `max_tokens: 1` 发送。在 Claude Sonnet 5 上，5% 轮次暂停时它们的成本比 1 小时设置低 7.7%，10% 时大致相同；在 Claude Opus 5 上，两种比例下都没有可测量的差异；在两个模型上，每轮之前暂停 6 分钟或更长时间时，它们的成本都更高。因此在这两个模型上，应使用的设置是 1 小时时长，而不是保活请求。
    * **保活请求，Claude Opus 5.5：** 在 Claude Opus 5.5 上，5% 和 10% 轮次暂停时，保活请求的成本比 1 小时设置低 8% 到 18%（在其默认努力程度 `medium` 下低 8% 到 13%，在 `high` 下低 10% 到 18%；通过按 1 小时缓存价格对每个保活会话自身的令牌重新计费以消除会话间噪声后，约低 10% 到 15%），而每轮之前都暂停时成本更高：6 分钟时高 4% 到 6%，20 分钟时高 9% 到 10%，45 分钟时高 56% 到 58%。
    * **使用 `max_tokens: 0` 的请求：** 于 2026 年 10 月 20 日在 Claude Fable 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 上测量，每个模型三次，使用一个 10,000 令牌的系统提示和 5 分钟缓存。在缓存写入 4 分钟后发送的 `max_tokens: 0` 请求没有返回任何输出，读取了整个条目，且没有写入任何内容。再过 4 分钟（即写入后 8 分钟）的请求仍读取了整个条目。如果没有 `max_tokens: 0` 请求，8 分钟时的请求什么也没有读取，并重新写入了前缀，12 次全部如此。
    * **5 分钟和 1 小时设置，Claude Sonnet 5.5：** 5% 轮次暂停时，5 分钟设置的成本平均比 1 小时设置高 5%（三次运行中从低 2% 到高 12%），这在运行间差异范围内：与 Claude Sonnet 5 上一样，该次抽样的暂停落在较小的前缀上，按 1 小时缓存价格对每个 5 分钟会话自身的令牌重新计费，约高 1%。10% 暂停时，其成本高 54%（高 36% 到 73%）。无暂停时，5 分钟设置的成本平均比 1 小时设置低 19%（三次运行中低 3% 到 32%；1 小时设置更高的写入价格预测约为 20%），在 2 分钟暂停的每次运行中低 21% 到 24%。
    * **保活请求，Claude Sonnet 5.5：** 5% 轮次暂停时，保活请求的成本平均比 1 小时设置低 16%（三次运行中低 13% 到 20%），10% 时低 8%（低 3% 到 10%），每轮之前暂停 6 分钟或更长时间时成本更高。在 5% 和 10% 轮次暂停时，Claude Sonnet 5.5 和 Claude Opus 5.5 上的暂停持续 6 到 32 分钟。与上文 Claude Opus 5.5 一样，按 1 小时缓存价格对每个保活会话自身的令牌重新计费，5% 暂停时低 15% 到 16%，10% 时约低 10%；每轮之前都暂停时，6 分钟高 7%，20 分钟高 9% 到 10%，45 分钟高 66% 到 73%。6 分钟的数据不包括第 3 次运行，在该次运行中，该会话的 39 个保活请求中有 2 个超时：缓存在这两个轮次之前过期，该会话向缓存写入了 214,000 个令牌，而第 1 和第 2 次运行为 80,000 到 85,000 个，其成本比第 3 次运行的 1 小时会话高 38%。
    * **保活请求，Claude Sonnet 5.5，按运行：** 若改为将每次运行的保活会话与同一次运行的 1 小时会话进行比较，保活的成本在 6 分钟时从大致相同到高 10%（第 1 和第 2 次运行；平均高 4%），在 20 分钟时从大致相同到高 16%，在 45 分钟时高 64% 到 80%。
    * **保活请求，Claude Sonnet 5.5，样本量：** 保活图表中 6 分钟的点对全部三种设置都取第 1 和第 2 次运行的平均值。每个单元格为三个会话（上述 6 分钟数据为两个）。无暂停时，保活不发送额外请求，因此与 5 分钟设置相同，但两者在单次运行中的差异高达 23%，因此三次运行平均值中几个百分点的差异并不是明确的效应。
    * **关闭和开启缓存，Claude Sonnet 5：** 在 Claude Sonnet 5 上，同一任务分别在关闭缓存和开启缓存的情况下运行，于 2026 年 8 月 19 日各运行三次，`max_tokens` 为 700：平均每个会话 $3.36 USD 对 $0.57 USD，节省 83%。
    * **根据记录的令牌数定价，Claude Sonnet 5.5：** Anthropic 还针对[为什么缓存排在首位](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#why-caching-comes-first)中的缓存数据，对使用 5 分钟缓存且无暂停的三个 Claude Sonnet 5.5 会话所记录的令牌数重新定价。这三个会话中的每一个都是一次包含 40 个请求、覆盖任务全部 20 个问题的对话。按截至 2026 年 10 月 12 日的标价，根据使用量字段，这些会话平均每个成本 $0.51 USD，而将所有输入令牌按输入价格计价、输出令牌不变时为 $3.70 USD：节省 86%（三个会话中为 86% 到 87%），即系数为 7.3。按 $0.20 USD 的发布缓存读取价格计算，节省为 82%，因此相对 Claude Sonnet 5 数据的提升来自更低的缓存读取价格。
    * **根据记录的令牌数定价，Claude Opus 5.5：** 以同样方式重新定价，在其默认努力程度 `medium` 下、使用 5 分钟缓存且无暂停的三个 Claude Opus 5.5 会话平均每个成本 $1.02 USD，对比 $6.93 USD，节省 85%，即系数为 6.8；该模型在 79% 的请求中使用了思考。
    * **思考的设置方式，仅限两组运行：** 使用 5 分钟缓存且无暂停的三个 Claude Sonnet 5.5 会话的请求未设置任何思考选项，因此思考保持模型默认值，模型在 38% 的请求中使用了思考。关闭缓存和开启缓存的 Claude Sonnet 5 运行的请求同样未设置思考选项，因此思考保持模型默认值，模型在关闭缓存时的 121 个请求中有 41 个、开启缓存时的 120 个请求中有 40 个使用了思考。
    * **暂停后的延迟：** 在 Claude Sonnet 5 和 Claude Opus 5 上，在 1 小时设置下，暂停后的轮次保持了热缓存延迟。
    * **在对话中途更改时长：** 于 2026 年 10 月 21 日在 Claude Fable 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 上测量，使用 Messages API，不带工具，采用自适应思考、`high` 努力程度、系统提示上的缓存断点，以及[自动缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#automatic-caching)或最后一个块上的显式断点。对话包含约 5,000 或 100,000 个令牌的填充文本，助手轮次为固定的单词回复，且没有思考块。在以 5 分钟时长缓存的对话中，第一个使用 1 小时缓存时长并添加一个轮次的请求什么也没有读取，并按 1 小时价格写入了整个提示（55 次中 55 次）。下一个使用 1 小时缓存时长的请求读取了全部内容（36 次中 36 次）。从第一个请求起以 1 小时时长缓存的系统提示仍被读取（12 次中 12 次）。使用 1 小时缓存时长但未添加任何内容的请求读取了 5 分钟时长的条目（31 次中 31 次），但没有延长它们：8 分钟后，相同的请求什么也没有读取（12 次中 12 次）。在以 1 小时时长缓存的对话中，使用 5 分钟缓存时长的请求读取了全部内容，且只写入了其新轮次（12 次中 12 次）。这些请求在 Claude API 上的一个 Anthropic 内部组织中运行。
    * **在对话中途更改时长，定价方式：** 根据 `usage` 字段按标价计算，在略超过 100,000 个令牌、`max_tokens` 为 16 时，第一个使用 1 小时缓存时长并添加一个轮次的请求在上述四个模型上（按所列顺序）的成本分别为 $2.01 USD、$0.80 USD、$0.40 USD 和 $0.10 USD。读取缓存的轮次成本分别为 $0.026 USD、$0.020 USD、$0.010 USD 和 $0.005 USD。Claude Haiku 5.5 按其[长上下文价格](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)计价。

17. **生产环境中的缓存读取占比：** 截至 2026 年 8 月 23 日的 14 天内第一方 Claude API 使用量的汇总，仅限直接 API 产品，排除 Anthropic 内部组织，不识别任何组织。当一个组织日的请求携带工具定义和工具结果、其提示平均包含 9 次或更多先前的工具调用、使用了缓存，且至少发出 10 个此类请求时，该组织日计为一个智能体循环（API 没有对话标识符，因此以此代替对话长度）：106,487 个组织的 303,003 个组织日，缓存读取占全部输入令牌的中位数比例为 84.2%，上四分位数为 91.7%。用例标签（组织声明的用例，否则为其分类得出的用例）覆盖了这些组织日的 74% 及其令牌的 99%；约 72% 的编码组织日读取比例达到 80% 或更高。组织日中排名前十分之一的，编码类读取比例为 95.9% 或更高，支持、研究、数据及其他智能体为 94.2% 到 94.8%。在 25 次或更多先前工具调用时的请求级拆分来自一个六小时样本：编码类 92% 为读取，7% 为写入，不到 1% 未缓存。

18. **缓存破坏、压缩时机和状态行测量：**

    * **2026 年 10 月测试，对长对话进行一项更改：** 于 2026 年 10 月 20 日在 Claude Fable 5.1、Claude Opus 5.5、Claude Sonnet 5.5 和 Claude Haiku 5.5 上测量，使用 Messages API，不带工具，采用自适应思考、各模型的默认努力程度、5 分钟缓存、系统提示上的缓存断点以及[自动缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#automatic-caching)。对话包含约 101,000 个令牌：一个 20,000 令牌的系统提示、13 个填充文本的用户轮次、模型对第一个轮次的自身回复及其思考块，以及对其余轮次的单词回复。在一个请求缓存了该对话后，第二个请求添加了一个简短轮次，每个被测量的请求都发送带有一项更改的这个较长对话。每个模型的两个对话得出了相同的结果。
    * **2026 年 10 月测试，读取和写入的令牌：** 在无更改以及按消息更改努力程度时，请求读取了全部 101,000 个令牌，且没有写入任何令牌。使用对话中途系统消息（25 个令牌），或在最新用户轮次末尾放置 25 令牌的状态行时，请求读取了全部令牌，且只写入了新文本。顶层努力程度更改（在 Claude Fable 5.1 和 Claude Sonnet 5.5 上改为 `low`，在 Claude Opus 5.5 和 Claude Haiku 5.5 上改为 `high`）读取了 20,000 令牌的系统提示，并写入了 81,000 个令牌。将相同的 25 个令牌放在系统提示开头或末尾时，什么也没有读取，并写入了全部 101,000 个令牌。在对话中途系统消息、顶层努力程度更改或在系统提示末尾添加 25 个令牌之后的请求读取了新前缀；开头的新状态行则再次写入了全部内容。2026 年 10 月 21 日的另一项测试在相同的四个模型上使用 5,000 令牌的对话，并在除 Claude Fable 5.1 之外的模型上使用 100,000 令牌的对话，全部采用单词回复，在对话中途首次发送 beta 标头（就像首次按消息更改努力程度时可能发生的那样）：该请求读取了前一个请求所缓存的全部内容，54 次全部如此。不带 beta 标头发送的系统消息也是如此，15 次全部如此。
    * **2026 年 10 月测试，定价方式：** 图表给出了一个请求的提示部分成本，根据其 `usage` 字段按标价计算。图表中的数值，依次为编辑系统提示、系统消息、顶层努力程度更改和按消息更改努力程度：Claude Fable 5.1 为 $1.26 USD、$0.026 USD、$1.02 USD 和 $0.025 USD；Claude Opus 5.5 为 $0.50 USD、$0.020 USD、$0.41 USD 和 $0.020 USD；Claude Sonnet 5.5 为 $0.25 USD、$0.010 USD、$0.20 USD 和 $0.010 USD。未更改的请求与较早的请求逐字节相同；一个约 25 令牌的新轮次增加的成本不到 $0.001 USD。在 Claude Haiku 5.5 上，其请求超过 100,000 个提示令牌，因此按其[长上下文价格](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)计价：编辑系统提示为 $0.063 USD，努力程度更改为 $0.052 USD，读取为 $0.005 USD。这些请求在一个 Anthropic 内部组织上运行，其用量的计量方式与客户组织相同。对于该组织，当请求未设置 `prefix_mismatch_behavior` 时，API 会放行未通过检查的思考块，因此这些测试中的系统提示编辑被接受。在 2026 年 8 月 31 日 00:00 UTC 或之后创建的账户上，同样的编辑在这四个模型上都会收到 400 错误，因为每个请求都会回传一个较早的思考块。
    * **2026 年 11 月测试，压缩后的编辑：** 于 2026 年 11 月 9 日在 Claude Opus 5.5 和 Claude Fable 5.2 上运行，使用 Messages API，采用自适应思考、`high` 努力程度、不使用缓存、定义一个工具、`tool_choice` 设为 `none`、[`prefix_mismatch_behavior`](https://platform.claude.com/docs/zh-CN/build-with-claude/preserved-thinking#mismatch-behavior) 设为 `"error"`，以及触发值为 50,000 个令牌的[在令牌阈值处进行压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold)。第一个约 55,700 令牌的请求触发了一次压缩，其回复包含一个压缩块，后跟一个思考块。下一个请求将该回复发回，并附带一个新的用户轮次。未作更改时，该请求成功。在 `tools` 中添加一个工具，或改写系统提示后，它收到了 400 错误。在没有压缩的对话中，同样的两项编辑收到了 400 错误，从较早的用户轮次中删除一个文本块也是如此。每个模型每种对话各一个。在 Claude Fable 5.2 上，第一个大型对话的回复不包含思考块。第二个对话取代了第一个，使用了更难的问题、更高的 `max_tokens` 以及自定义的压缩 `instructions`，只有第二个对话计入。这些请求在 Claude API 上运行。
    * **测量内容：** [精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)中分诊智能体的长版本，于 2026 年 8 月 24 日在 Claude Sonnet 5 上使用 5 分钟缓存运行，成本根据使用量字段按标价计算，每组五个会话：一个全程采用默认努力程度的无变更组（每会话 $0.81 USD），以及两个从低努力程度开始并进行相同两项破坏缓存的变更（切换到默认努力程度并添加一个工具）的组，变更要么在会话中途的第 12 和第 17 个请求进行（$0.95 USD），要么在第一次压缩后的第一个请求上一起进行（$0.75 USD）。
    * **第四组：** 第四组包含六个会话，于 2026 年 8 月 25 日运行，在触发第一次压缩的请求上进行相同的两项变更（每会话 $0.92 USD）：该请求的摘要过程将 81,000 令牌的上下文写入缓存而不是读取它，因此该过程花费 $0.21 USD，而边界组中相同过程为 $0.04 USD。
    * **压缩：** 会话在提示超过 80,000 令牌的压缩触发阈值后，于第 21 到 25 个请求首次压缩（21 个会话中有 16 个在第 22 个请求），有两个无变更会话在接近结束时进行了第二次压缩。边界组的总成本低于无变更组，反映的是其变更前的低努力程度请求以及那些第二次压缩，而非缓存：两组的重写成本相差不到 $0.01 USD。
    * **会话中途组：** 会话中途组每会话在缓存重写上花费 $0.23 USD；会话中途组与边界组之间的差异为 $0.20 USD，95% 置信区间为 $0.11 USD 到 $0.29 USD。在会话中途组中，努力程度变更和添加的工具分别重写了 39,000 和 60,000 个缓存令牌。其中一个会话成本较低（$0.82 USD），原因是其模型在压缩后错误调用了搜索工具并得到空结果；该会话被包含在内，若排除它，该组平均为 $0.98 USD。
    * **准确率和缓存读取：** 8 月 24 日的每组准确率平均为 20 个标签中的 14.2 个，8 月 25 日的组为 14.7 个；缓存读取占提示令牌的比例在无变更时为 91%，会话中途为 85%，边界处为 91%，在触发请求上进行变更时为 86%。
    * **单独测量，状态行：** 另一项测量针对[精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)中的 20 个问题分诊任务，于 2026 年 8 月 18 日运行，同样在 Claude Sonnet 5 上使用 5 分钟缓存，以相同方式定价，每组运行三次：在系统提示开头放置一个 25 令牌的状态行时，每次运行平均成本 $4.24 USD（三次运行中为 $4.12 USD 到 $4.47 USD），且没有从缓存中读取任何内容；将同一行放在最新用户轮次的末尾、最后一个缓存断点之后时，每次运行成本 $0.59 USD（$0.55 USD 到 $0.61 USD）。前者比关闭缓存时同一任务的成本还高。之后以相同配置重新运行，得到 $4.28 USD 和 $0.61 USD。
    * **Claude Haiku 5.5 上的压缩触发值：** 于 2026 年 10 月 21 日测量，使用 Messages API，采用[在令牌阈值处进行压缩](https://platform.claude.com/docs/zh-CN/build-with-claude/compaction-threshold)，不带工具，采用自适应思考、默认努力程度（`medium`）、5 分钟缓存、系统提示上的缓存断点以及[自动缓存](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching#automatic-caching)。每个会话是一次包含 45 个请求的对话：一个 5,000 令牌的系统提示，每个用户轮次附带 10,000 个令牌的填充文本，以及单词形式的助手轮次，但带有压缩块的回复会被完整发回。四种设置各运行三个会话：触发值为 150,000 个令牌（默认值）、90,000 和 50,000（允许的最低值），以及不压缩。同一设置内的成本相差在 4% 以内。使用默认触发值时，每个会话的 45 个回复中有 15 个以及全部 3 个摘要的提示超过 100,000 个令牌。触发值为 90,000 时，最大的提示包含 97,269 个令牌。压缩在整个提示（包括系统提示、缓存读取和缓存写入）首次达到触发值的请求上运行。摘要根据该提示加上 44 个令牌（两者合计最多超出触发值 7,269 个令牌）生成。另外三个会话使用 22,000 令牌的系统提示和 50,000 的触发值，表明系统提示会被计入。由于文本是填充内容，摘要较短（184 到 640 个输出令牌）。这些请求在 Claude API 上的一个 Anthropic 内部组织中运行。
    * **Claude Haiku 5.5 上的压缩触发值，定价方式：** 根据 `usage` 字段按标价计算，每个会话的成本在默认触发值下为 $0.272 USD，90,000 时为 $0.082 USD，50,000 时为 $0.075 USD，不压缩时为 $0.729 USD。按请求计算，从第一次压缩之后的请求到最后一次压缩，三种触发值的成本分别为 $0.00605 USD、$0.00181 USD 和 $0.00166 USD。如果每个提示都按不超过 100,000 令牌的提示价格计价，默认值的成本为每个请求 $0.00206 USD，而 90,000 的触发值将节省 12%。压缩的摘要和回复各自按其自身提示的大小定价。

19. **Claude Fable 5.1 和 Claude Fable 5.2 上的缓存时长测量：** 与参考文献 16 相同的 20 个问题分诊任务和评估框架，`max_tokens` 提高到 4,096，于 2026 年 8 月 23 日和 8 月 26 日在 Claude Fable 5.1 发布快照上按其发布价格运行（每百万令牌输入 $10 USD、5 分钟写入 $12.50 USD、1 小时写入 $20 USD、缓存读取 $0.25 USD、输出 $50 USD），每种计划三种设置：5 分钟缓存、1 小时缓存，以及每 4 分钟（从上一个请求开始时计时）对未变更前缀发送一个 `max_tokens: 0` 请求以保持预热的 5 分钟缓存（8 月 23 日的运行以 `max_tokens: 1` 发送保活请求；在此处报告的 8 月 26 日单元格中，每个得到应答的保活请求都刷新了缓存，且其响应报告的输出令牌为零）。计划：在全部 20 个问题上设置无暂停、10% 轮次暂停和每轮 6 分钟暂停，在 5 个问题的子集上设置 45 分钟暂停；每个单元格运行三次（8 月 26 日 45 分钟暂停的保活单元格为六次），成本根据每个响应的 `usage` 字段按标价计算，准确率对照相同的黄金标签计算（20 个中精确匹配 12 到 18 个）。8 月 26 日 5 分钟、1 小时和保活设置的每会话平均值：无暂停 $2.42 USD、$3.09 USD、$2.29 USD；10% 暂停 $4.50 USD、$2.96 USD、$2.36 USD；每轮暂停 $22.89 USD、$3.01 USD、$2.62 USD；8 月 23 日的单元格与之相差在 6% 以内。45 分钟的数据（每个五问题会话 $1.68 USD、$0.59 USD 和 $0.71 USD）来自 8 月 26 日较晚的会话：当天的第一组会话因其机器关闭而中断，第二组受到服务故障影响而被搁置。第三组给出了 5 分钟和 1 小时的数据；其三个保活会话每个保活请求只发送一次，因速率限制拒绝而丢失了请求，因此不计入平均值。保活数据是另外六个会话的平均值，这些会话会在两秒内重新发送任何被拒绝的保活请求。8 月 23 日的运行得出 $1.67 USD、$0.58 USD 和 $0.70 USD。5 分钟和 1 小时设置之间的盈亏平衡点为 3.1% 的轮次，与参考文献 16 的衡量方式相同。Claude Fable 5.1 的缓存读取成本为输入价格的 0.025 倍，而其缓存写入保持标准倍数，因此重新读取前缀的保活请求很便宜，1 小时时长的写入溢价才是更大的开销，但 45 分钟暂停时除外，此时保活请求的成本高于该溢价。Claude Fable 5.2 在 `high`（其在 Messages API 上的默认值）下运行了相同的任务，并按标价计费（每百万令牌输入 $8 USD、5 分钟写入 $10 USD、1 小时写入 $16 USD、缓存读取 $0.20 USD、输出 $40 USD）。与参考文献 16 中 Claude Opus 5.5 的单元格一样，它使用了发送相同请求体的评估框架移植版本，`max_tokens` 提高到 4,096。计划：在全部 20 个问题上设置无暂停、5% 和 10% 轮次暂停以及每轮 6 分钟暂停，在 5 个问题的子集上设置 20 分钟和 45 分钟暂停，每个单元格运行三次。45 分钟的单元格测量了两次；其数据合并了两组各三个会话，并且与 Claude Fable 5.1 一样，排除了丢失保活请求的保活会话（此处原因为超时；每组各一个），因此 45 分钟的保活数据是四个会话的平均值，5 分钟和 1 小时的数据是六个会话的平均值。全程以 `max_tokens: 0` 发送保活请求。测试客户端对保活请求等待 30 秒，且不重试。在上述所有计划中，它在 886 个保活请求中有 3 个停止等待，全部出现在 45 分钟的单元格中，每次缓存都过期了，下一个保活请求重新写入了前缀。5 分钟、1 小时和保活设置的每会话平均值：无暂停 $2.18 USD、$2.66 USD、$2.23 USD；10% 暂停 $4.16 USD、$2.82 USD、$2.29 USD；每轮 6 分钟暂停 $20.36 USD、$2.61 USD、$2.50 USD；45 分钟暂停时每个五问题会话 $1.43 USD、$0.53 USD 和 $0.65 USD。[选择缓存时长](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#pick-the-cache-duration)中 Claude Fable 5.2 的节省幅度在四舍五入前计算。若计入丢失保活请求的两个会话，45 分钟保活平均值为 $0.73 USD，比 1 小时设置高 $0.20 USD。

20. **Terminal-Bench 3：** 该公开终端智能体基准测试的 74 个任务，在 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 上运行，使用两个自定义工具——一个 shell 和一个文件编辑器，由评估框架在每个任务自己的容器中运行——代替平台的内置工具，其他方面采用平台面向外部账户的默认设置，每个模型在 `high` 努力程度下运行两次，于 2026 年 8 月 27 日至 28 日。这些运行使用 Terminal-Bench 3.0 版本，其分数不可与公开的 Terminal-Bench 排行榜比较，也不可与 Claude Opus 5.5 系统卡中的 Terminal-Bench 4.0 结果比较，后者来自在 Claude Code 中以 `max` 努力程度进行的运行。每个任务的时间限制是基准测试自身限制的 2.5 倍，这使智能体每个任务有 75 分钟到 20 小时的时间（中位任务为 5 小时），每个任务获得其指定内存的三倍，从 6 GiB 到 96 GiB，运行辅助服务的 12 个任务另有额外内存。智能体没有一般的互联网访问：其容器可以访问一个内部软件包镜像、一小份下载站点列表（包括 GitHub 和 Python Package Index），以及少数特定于某些任务的站点，其中八个任务完全没有网络访问。分数为每个模型 148 次尝试的原始通过率；单次运行的波动为 5 到 11 分。成本是客户按标价会被收取的费用，根据运行的使用记录按 5 分钟缓存生命周期逐请求重新定价。Claude Opus 4.7、Claude Opus 4.8 和 Claude Opus 5 每个任务各花费 $8 USD 到 $15 USD。Claude Opus 4.7 的 148 次尝试中有 11 次因达到输出上限而结束。

21. **DRACO：** Perplexity, “DRACO: a Cross-Domain Benchmark for Deep Research Accuracy, Completeness, and Objectivity,” arXiv:2602.11685, 2026。

    * **测量内容：** 其 10 个领域的 100 个研究任务按专家编写的评分标准评分，每个标准约有 40 个加权条目，分数为满足条目的加权比例，范围为 0 到 100。
    * **配置：** 所有配置都在 Claude API 上使用 Claude Opus 5.5 运行，采用默认的自适应思考，开启生产安全分类器，`max_tokens` 为 128,000 并使用流式传输：`medium` 努力程度（模型默认值）和 `low` 下的单智能体、`medium` 下带有该句子和时钟的单智能体，以及 `medium` 下带有和不带这两项的团队。团队是一个主智能体，可以通过工具启动同一模型的辅助智能体，数量不设上限。
    * **句子：** 两项变更是一个句子和一个时钟。该句子"Time matters here: do not spend time that can be avoided, and the earlier a correct result is obtained, the better."（时间很重要：不要花费可以避免的时间，越早得到正确结果越好。）放在智能体第一条 `user` 消息中任务的末尾，空一行之后，并放在每个辅助智能体简报的末尾。
    * **时钟：** 时钟是以整秒计的已用时间，例如 `[elapsed 252s]`，在每个请求（包括第一个）之前添加到最新 `user` 消息的末尾：在第一个请求上，作为任务之后的独立文本块；当消息携带工具结果时，添加在最后一个工具结果文本的末尾，空一行之后。较早的时钟保留在原位，因此缓存前缀在下一个请求上仍然匹配。团队中的每个智能体读取同一个时钟，系统提示不变。
    * **在测量的运行中：** 测量的运行使用了这一措辞和格式。它们的时钟从第一个响应的第一个令牌开始计时，而不是从任务开始时。在测量的团队运行中，辅助智能体的报告以时钟之后的单独消息送达主智能体，而不是在最后一个工具结果内。
    * **时间：** 每种配置对每个任务进行三次尝试，每次尝试限时四小时，运行于 2026 年 9 月 24 日，同一组的所有配置同时开始。
    * **工具：** 智能体使用评估框架基于 Anthropic 网络索引托管的搜索和抓取工具，以及在无网络访问的沙箱中的 Python 和 shell。这些工具决定了部分时间，而您的工具运行速度会不同，因此本页以配置之间的比率给出时间，而不是以分钟计。
    * **计时方式：** 时间是一次尝试的挂钟时间，从其第一个请求到任何智能体最后一个请求结束，包括工具运行，在一个任务的各次尝试上取平均。典型任务数据是各任务在两种配置之间比率的中位数。
    * **定价方式：** 每个任务的成本是其请求按 Claude Opus 5.5 公开标价计算的费用，提示缓存按客户在每个请求末尾设置缓存断点并使用 5 分钟缓存生命周期时的方式计费。每个请求从缓存中读取上一个请求的提示，并写入其余部分。一次尝试的第一个请求，以及任何在前一个请求之后超过 5 分钟才开始的请求，会写入其整个提示。成本仅涵盖模型令牌；评估框架的工具不产生额外费用。
    * **评分方式：** 所有区间均为基于任务的 95% bootstrap 区间，分数变化为基于任务的配对差异。由 Claude Opus 5 对答案评分，因此 Claude Opus 5.5 不会给自己的工作评分，每个答案的分数为五次评判运行的平均值。
    * **注意事项：** 生产安全分类器在每种配置中都停止了三次尝试，全部在同一个任务上，这些尝试得分为 0；排除该任务不会改变任何结论。这些运行还让每个智能体的工具使用经过一项客户请求所没有的 Anthropic 内部安全检查，这在每种配置中都为每个请求增加了时间。在 DRACO 上，不带变更的单智能体发出的请求约为两倍，团队约多三分之一，因此该检查可能使 DRACO 的时间节省看起来略大。
    * **结果，单智能体：** [向模型显示已用时间](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#show-the-model-elapsed-time)的结果：带有变更时，单智能体在典型任务上用时减少 47%，成本降低 60%，其平均每任务成本从 $1.13 USD 降至 $0.44 USD，分数低 4.1 分（95% 区间为低 2.9 到 5.2 分）。
    * **结果，更低的努力程度：** 降低单智能体的努力程度比这些变更节省了更多时间和成本，也损失了更多分数。在 `low` 努力程度下，它在 DRACO 上比 `medium` 低 13.3 分，在 HLE 上低 3.4 分，在物理集上低 14.0 分，典型任务用时分别减少 84%、30% 和 47%。在 `medium` 努力程度下带有变更时，它比 `low` 分别高 9.2、2.5 和 11.0 分，典型任务用时是 `low` 的 1.2 到 3.2 倍。
    * **结果，团队，辅助智能体：** 团队的主智能体很少启动辅助智能体。不带变更时，它在 DRACO 的 300 次尝试中只有 5 次、HLE 的 1,479 次中只有 13 次启动了辅助智能体，在物理集的 280 次中一次也没有。
    * **结果，团队：** 因此团队结果主要反映的是主智能体使用团队自身的指令和工具独自工作。不带变更时，团队在典型 DRACO 任务上的用时已约为单智能体的一半，分数低 3.3 分（95% 区间为低 2.1 到 4.6 分）。带有变更时，团队在典型 DRACO 任务上用时减少 17%，成本降低 20%，分数低 1.6 分（95% 区间为低 0.3 到 2.8 分）。

22. **HLE：** Phan et al., “Humanity’s Last Exam,” arXiv:2501.14249, 2025。专家编写的具有精确答案的问题，对照参考答案评分。在 2025 年 9 月 10 日版本的前 500 个问题上测量，包括图像，搜索中屏蔽了该基准测试自身的来源，其余设置与参考文献 21 相同。每种配置对每个问题进行三次尝试，运行于 2026 年 9 月 24 日。Claude Opus 5 使用该基准测试公布的评判提示，将每个答案与参考答案进行比较，以三次评判的多数结果为准。分数在每种配置中涵盖 493 个问题。在其余问题中，有五个的评分请求超过了评判模型 1M 令牌的限制，另外两个被排除，因为不带变更的单智能体运行在这两个问题上有三次尝试尚未完成时就停止了。时间和成本比率对单智能体涵盖 492 个问题，对团队涵盖 491 个，因为有一两个问题没有计时数据。生产安全分类器停止了每种配置 3% 到 4% 的尝试，这些尝试得分为 0；排除它们不会改变任何结论。[向模型显示已用时间](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#show-the-model-elapsed-time)的结果：带有变更时，单智能体在典型问题上用时减少 14%，成本降低 9%。使用 Claude Opus 5 评判时，其分数没有明显变化（低 0.9 分，95% 区间为低 2.5 到高 0.7）。使用第二个评分器 Claude Opus 4.6 时，其分数低 1.9 分（95% 区间为低 0.2 到 3.7 分）。平均时间的降幅大于典型问题：平均时间减少 43%，平均成本减少 37%，因为这些变更缩短了少数耗时很长的问题。团队在典型问题上的用时变化不超过 3%，其分数没有明显变化。

23. **物理集：** 一个包含 70 个研究级物理问题的内部集合，改编自公开的 CritPt 基准测试：Zhu et al., “Probing the Critical Point (CritPt) of AI Reasoning: a Frontier Physics Research Benchmark,” arXiv:2509.26574, 2025。专家审阅者修正了问题陈述。Claude Opus 4.6 使用该集合的评分提示，对照未公开的专家参考解答，以部分得分方式为每个答案评分，因此分数不可与已发布的结果比较。分数为一个问题各次尝试的平均得分，再在各问题上取平均，满分 100 分。在全部 70 个问题上测量，每个问题尝试四次，运行于 2026 年 9 月 24 日。每个智能体在无网络访问的沙箱容器中拥有 Python 工具、shell 和文件编辑器，没有搜索或抓取工具。在物理集上，没有安全分类器停止任何尝试。其他设置与参考文献 21 相同。[向模型显示已用时间](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#show-the-model-elapsed-time)的结果：带有变更时，单智能体在典型问题上用时减少 7%，成本降低 4%，分数低 3.0 分（95% 区间为低 0.4 到 5.6 分）。团队在典型问题上的用时变化不超过 3%，其分数没有明显变化。单智能体的平均时间减少 14%。单智能体的全部平均时间节省都来自一个问题，在该问题上，不带变更的智能体每次尝试运行代码约半小时；排除该问题后，有无变更时其平均时间大致相同。

24. **tau2-bench retail：** Barres et al., "τ²-Bench: Evaluating Conversational Agents in a Dual-Control Environment," arXiv:2506.07982, 2025。

    * **测量内容：** 零售领域的 114 个任务在公开评估框架上针对 Claude API 运行，以 Claude Haiku 4.5 作为模拟用户，Claude Sonnet 5 作为自然语言评判模型，并开启提示缓存；任务被分为 57 个搜索集和 57 个保留集。
    * **时间：** 两次搜索于 2026 年 9 月 6 日运行，由 Claude API skill 的 `hillclimb` 命令驱动。缓存前后对比于 2026 年 9 月 8 日运行。
    * **评分方式：** 在搜索中，起点和确认的配置为三次保留集运行的平均值，保留的步骤基于两次运行判断，其他候选基于一到两次运行评分。缓存对比：在全部 114 个任务上各运行三次，仅切换评估框架的缓存断点，所用配置是 `cost-optimize` 命令在较早一次运行中生成的配置。保留集数据基于搜索从未读取过的 57 个任务。
    * **定价方式：** 每任务成本按标价对智能体和模拟用户的令牌数定价（Claude Opus 5 每百万输入和输出令牌 $5 USD 和 $25 USD，Claude Sonnet 5 为 $2 USD 和 $10 USD，缓存读取和写入按其公布费率），不包括评判模型；百分比变化来自未经四舍五入的成本；月度数据仅为智能体的令牌，每任务 $0.135 USD 和 $0.047 USD，乘以 10,000。按每月 10,000 次对话计算，第一次搜索前约为 $1,350 USD，之后为 $470 USD。
    * **注意事项：** 这些测量在上一代 Claude Opus 和 Claude Sonnet 模型上运行：缓存对比在 Claude Sonnet 5 上运行，第二次搜索从 Claude Sonnet 5 开始，第一次搜索从 Claude Opus 5 转到 Claude Sonnet 5。
    * **缓存对比：** 该对比在默认努力程度下的 Claude Sonnet 5 上运行。公开评估框架按原样在每一轮都以未缓存方式发送整个对话。缓存配置在系统提示上设置一个断点，在最新消息上设置一个断点。两次搜索都在开启该缓存的情况下运行。每任务成本从未缓存的 $0.223 USD 降至缓存后的 $0.062 USD，节省 72%，通过率从 79.2% 变为 76.0%。该配置的相同缓存运行在不同日期的结果为 114 个中的 85 到 93 个。每次缓存运行的得分都等于或低于每次未缓存运行，但不到 4 个任务的差距处于相同缓存运行的波动范围之内，因此在此样本量下，准确率变化太小，无法测量。
    * **搜索的运行方式。** `hillclimb` 命令会提出四个问题，给出一个计划，并在花费任何费用之前等待批准。在第一次搜索中，该计划保持用户模拟器、评分器、缓存、温度和任务不变。四个问题中的最后一个以如下内容开头：`One pass over the full set is 114 cases x 1 rep on claude-opus-5, roughly 22 minutes.` 每一轮，搜索提出一项变更并运行完整集合。只有当数值发生变化且预测的机制在使用数据中得到体现时，它才保留该变更。当连续三轮没有任何收获时，它停止，然后在新的运行上确认胜出者。
    * **第一次搜索，结果。** 该搜索将每任务成本降低了 61%，同时将通过率保持在搜索前设定的 2 分区间内：从默认努力程度下的 Claude Opus 5（每任务 $0.142 USD，通过 76.0%）到 `low` 下带有一项成本中性提示编辑的 Claude Sonnet 5（$0.055 USD，74.3%）。该搜索在评估运行上花费 $179 USD，在其自身令牌上花费 $228 USD。
    * **第一次搜索，起点：Claude Opus 5，默认努力程度。** 76.0%，$0.142 USD。三次相同运行的结果为 57 个中的 43 到 44 个。搜索将下限设为低 2 分，即 74.0%，并在判定时将其向上取整为 57 个中的 43 个（75.4%）。
    * **第一次搜索，第 1 步，努力程度改为 `low`：保留。** 在保留它的两次运行上为 77.2%，在搜索后的三次运行确认中为 77.8%，$0.119 USD，便宜 17%。搜索自身的状态行部分内容如下：`two runs held-out 42 and 46, mean 44.0 >= floor 43; cost mean $0.1187 <= gate $0.1208; mechanism confirmed (output tokens -34% with calls flat)`。
    * **第一次搜索，第 2 步，`low` 下的 Claude Sonnet 5：保留。** 75.4%，$0.055 USD，便宜 61%。
    * **第一次搜索，第 3 步，一项提示编辑（登录后查询客户的订单，而不是询问订单 ID）：在选择时保留，之后未能维持。** 两次运行上为 79.8%；三次新运行上为 74.3%。搜索将其作为成本中性项保留：省下的轮次变成了另一个查询轮次。
    * **第一次搜索，第 4 步，一项提示编辑（仅在被要求时转接人工）：被拒绝并回退。** 71.9%。它修复了所针对的 3 个案例，却因一个通用的"说明您能做什么和不能做什么"模板破坏了另外 10 个。
    * **第一次搜索，第 5 步，一项提示编辑：未运行。** 其预测收益为 0.5 个任务，低于搜索 2 个任务的采纳门槛。
    * **第一次搜索，确认：`low` 下带有第 3 步提示的 Claude Sonnet 5，三次新运行。** 74.3%，$0.055 USD，以 0.14 个任务之差处于 2 分区间内，比起点少 57 个中的 1 个任务，且低于搜索登记的 57 个中 43 个的下限。该配置的五次运行结果为 42 到 46，比起点的 43 到 44 更宽。
    * **第一次搜索，说明了什么。** 努力程度和模型这两个杠杆带来了节省，且各自的机制都得到了确认。提示杠杆带来了真实的行为变化，但没有持久的收益。在此工作负载上，决定每任务成本的是轮次数量以及每轮重新读取的缓存前缀，而不是措辞。
    * **第二次搜索。** 它从开启缓存、默认努力程度下的 Claude Sonnet 5 开始：在保留集上为 79.5%，$0.062 USD，三次运行结果为 57 个中的 42 到 47 个，因此下限为 57 个中的 45 个（78.9%）。在这里，起始配置的相同运行之间相差最多 57 个中的 5 个任务。第二次搜索没有保留任何变更。`low` 努力程度、两项提示编辑以及两者结合，各自得分 77.2%，低于下限，节省 10% 到 14%，未达到搜索 15% 的成本门槛。一项针对准确率的提示编辑在两次运行上得分 79.8%，在三次新运行上得分 73.7%，被回退。该搜索约进行了 19 次评估运行，花费 $120 USD。

25. **Claude Managed Agents 任务集：**

    * **测量内容：** 一个 Anthropic 内部集合，包含八个领域的 79 个脚本化智能体任务，包括记忆、多智能体和长上下文工作，以 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 会话的形式运行，由测试应答每次工具调用。每个任务在每个模型上运行五次。
    * **运行：** 数据来自 2026 年 10 月 7 日每个模型的一次运行，并于当天评分。在 2026 年 10 月 8 日和 9 日对相同任务进行的另外三次运行中，Claude Haiku 5.5 通过了 91% 到 93% 的尝试；在 10 月 8 日的另一次运行中，Claude Haiku 4.5 通过了 63%。
    * **评分方式：** 只有当评分器 Claude Sonnet 4.6（评分一次）将任务检查清单的每一行都标记为满足时，一次尝试才算通过，因此通过率取决于评分器。除一个任务外，在所有任务上都由一次单独的调用（向其完整展示工具结果）对照这些结果，为检查智能体通过其工具写入的文件和消息的那一行评分。
    * **定价方式：** 成本为按标价、使用 5 分钟提示缓存、逐请求定价的令牌成本，不包括会话运行时间。
    * **结果，令牌价格和成本：** Claude Haiku 4.5 有 80% 的请求没有缓存：每个请求的提示约为 4,100 个令牌或更少，而其最小可缓存提示长度为 4,096 个令牌（Claude Haiku 5.5 为 512）。从 Claude Haiku 4.5 到 Claude Haiku 5.5，令牌价格下降了最多 90%，79 个任务的令牌成本下降了约 75%。令牌成本的降幅小于价格，因为 Claude Haiku 5.5 约 5% 的请求提示超过 100,000 个令牌，需支付更高的价格（[长上下文定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)），而这些请求占其成本的 73%。
    * **思考：** Anthropic 将 Claude Haiku 5.5 的努力程度设为 `medium`，且没有为任一模型发送思考设置或输出限制，因此思考由 Claude Managed Agents 针对每个模型的自身设置决定。Claude Haiku 5.5 在 59% 的请求中使用了思考，Claude Haiku 4.5 为 22%。该比较针对的是各模型的实际服务状态，而非相同设置。
    * **不使用缓存的成本，根据记录的令牌数定价：** Anthropic 还针对[为什么缓存排在首位](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#why-caching-comes-first)中的缓存系数，对 Claude Haiku 5.5 在 10 月 7 日运行中记录的令牌数重新定价。Claude Managed Agents 会自行应用提示缓存，因此该系数比较的是同一批请求的两种价格，而不是两种设置。
    * **系数：** 该系数为 1.7，是将所有输入令牌按输入价格计价、输出令牌不变时该运行的令牌成本，除以每个会话单独定价时的令牌成本：每个请求的先前提示按缓存读取计价，其新令牌按 5 分钟缓存写入计价。在两种成本中，每个请求都按其自身的提示长度定价，因此超过 100,000 个令牌的 87 个请求在两种成本中都支付更高的价格。按实际记录的令牌数（本指南中该运行的成本即使用此数据）计算，系数为 1.9，因为一个任务的五次尝试是并行运行的，并读取了彼此的缓存条目。
    * **系数为何较低：** 系数较低是因为会话很短，中位数为 4 个请求：大多数输入令牌作为工具结果在第二或第三个请求中到达，只从缓存中被读取一两次。八个长上下文任务占了四分之三的成本。10 月 8 日的第二次运行同样得出 1.7。

26. **答案格式测量：**

    * **测量内容：** [精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)中的 20 个问题分诊任务，在 Claude Sonnet 5 上运行，开启提示缓存，请求中没有思考设置，因此思考保持模型默认值，使用三种最终答案指令，每种指令运行三次。
    * **两行指令：** 原始指令要求两行：`4. Finish with exactly two lines: LABEL: <one of: bug-confirmed, needs-more-info, duplicate-candidate, feature-request, upstream-issue, perf, ui-polish> SUMMARY: <one or two sentences for the engineering team>`
    * **一行指令：** 较短的变体要求一行：`4. Finish with exactly one line in this form: DECISION | LABEL | REASON where DECISION is one of: triage-now, needs-info, close-duplicate; LABEL is one of: bug-confirmed, needs-more-info, duplicate-candidate, feature-request, upstream-issue, perf, ui-polish; REASON is one clause under 15 words. Output nothing after that line.`
    * **备忘录指令：** 较长的变体要求一份包含六个带标题部分的备忘录：问题摘要、证据、重复检查、推荐标签、后续步骤和置信度。
    * **实际发送的指令中还包括：** 实际发送时，两行指令还写有"Be concise."（简明扼要。），备忘录指令还写有"Be thorough and explicit."（详尽而明确。）
    * **时间：** 两行和一行指令于 2026 年 8 月 18 日运行，`max_tokens` 为 700；备忘录指令于 2026 年 8 月 19 日运行，`max_tokens` 为 3,000。没有任何响应达到其上限。
    * **评分和定价方式：** 数据为三次运行按 Claude Sonnet 5 标价计算的平均值，准确率为对照黄金标签的精确匹配。
    * **结果：** 一行答案比两行原始答案少用 39% 的输出令牌，每次运行成本低 13%（根据未经四舍五入的平均值；图表中四舍五入后的成本 $0.49 USD 和 $0.57 USD 得出 14%）。备忘录使用的输出令牌是两行原始答案的六倍，成本是一行答案的 2.8 倍。
    * **思考：** 约一半的响应包含思考，其令牌计入输出令牌。

27. **工具搜索测量：**

    * **测量内容：** [精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)中的 20 个问题分诊任务，于 2026 年 8 月 19 日在 Claude Sonnet 5 上运行，开启提示缓存，请求中没有思考设置，因此思考保持模型默认值，每种配置运行三次。智能体拥有自己的两个工具，外加来自公开 MCP 服务器的真实工具定义，总计 7、17、52、102、252 或 502 个工具，要么加载所有定义，要么将新增的定义标记为 `defer_loading` 并置于工具搜索之后。
    * **评分和定价方式：** 数据为三次运行按 Claude Sonnet 5 标价计算的平均值，准确率为对照黄金标签的精确匹配。
    * **结果：** 加载所有定义时，每次运行的成本从 7 个工具时的 $0.55 USD 升至 502 个工具时的 $1.02 USD，几乎翻倍。使用工具搜索时，在每种目录规模下均为 $0.55 USD 到 $0.56 USD，在 502 个工具时低 45%（根据图表中四舍五入后的成本 $0.56 USD 和 $1.02 USD；未经四舍五入的平均值得出 45.6%）。无论是否使用工具搜索，每次运行都正确回答了 20 个问题中的 15 到 18 个。
    * **注意事项：** 没有任何运行调用目录中的工具或工具搜索工具：智能体自己的两个工具始终处于加载状态，每次工具调用都指向其中之一。该测量显示的是携带未使用定义的成本。
    * **单独测量：** 在同一模型和日期的一项单独测量中，通过 MCP 连接器接入一个公开的 GitHub MCP 服务器且不使用目录，将服务器工具集延迟加载的三次运行比不延迟的三次运行成本低 20%（每次运行 $0.56 USD 和 $0.70 USD），每组的三次运行分别正确回答了 17、16 和 16 个问题。此单独测量中的请求不包含工具搜索工具。加载工具集时，每次运行调用服务器的一个工具 `issue_read` 3 或 4 次；延迟加载时，没有任何运行调用服务器的任何工具。

28. **输入精简和上下文生命周期测量：**

    * **测量内容：** [精简输入和上下文令牌](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#trim-input-and-context-tokens)中的 20 个问题分诊任务及其长版本，于 2026 年 8 月 19 日（按 UTC 计）在 Claude Sonnet 5 上运行，每种设置运行三次，请求中没有思考设置，因此思考保持模型默认值；约十分之三的响应包含思考。
    * **图表：** [管理上下文生命周期](https://platform.claude.com/docs/zh-CN/about-claude/models/optimizing-for-cost-and-intelligence#manage-the-context-lifecycle)中的图表在每个任务上比较四种设置：开启提示缓存和输入精简（图像缩放和工具搜索）的任务，以及在此基础上添加一个杠杆的相同任务，该杠杆为上下文编辑、压缩或修剪。图表将低于 10% 的变化视为无变化。
    * **杠杆设置：** 两个任务的杠杆设置不同：在 20 个问题的任务上，上下文编辑在提示超过 20,000 个令牌后至少清除 2,000 个令牌；在长任务上，在超过 50,000 个令牌后至少清除 40,000 个令牌，并保留最近的三次工具使用；压缩在 20 个问题的任务上于 50,000 个输入令牌时触发，在长任务上于 80,000 个时触发。
    * **定价方式：** 每项成本为三次运行的平均值，根据响应的使用量字段按 Claude Sonnet 5 标价计算，缓存写入为每百万令牌 $2.50 USD，缓存读取为 $0.20 USD。
    * **结果，输入精简：** 第五种设置——开启提示缓存但不进行输入精简——在 20 个问题的任务上每次运行成本 $0.57 USD，在长任务上为 $1.38 USD，因此输入精简使前者降低 26%，后者降低 21%。与关闭缓存时同一 20 个问题的任务每次运行 $3.36 USD 的成本（参考文献 16）相比，提示缓存和输入精简共同节省了 88%（未经四舍五入的平均值得出 87.5%）。
    * **结果，20 个问题的运行：** 在 20 个问题的运行中，不使用杠杆的任务每次运行成本 $0.42 USD，上下文编辑的成本高 73%，压缩在每次运行接近结束时运行一次，成本低 7%，修剪没有重写任何内容，因为没有任何搜索结果超过其 2,000 字符的阈值。
    * **结果，长运行：** 在长运行中，不使用杠杆的任务每次运行成本 $1.09 USD，修剪的成本低 39%，压缩低 32%，上下文编辑高 4%。
    * **缓存读取，长运行：** 在使用修剪的长运行中，任务边界后的第一个请求有 89% 的提示令牌从缓存读取，其他请求为 81%。在整个长运行中，使用修剪时 85% 的提示令牌从缓存读取，使用上下文编辑时为 85%，而不使用杠杆时为 95%。

29. **内部编码基准测试（子智能体测量）：**

    * **测量内容：** 134 个代码仓库编码任务，每个任务由智能体看不到的测试评为通过或失败；在 2026 年 10 月 21 日至 23 日的三次运行中，每次运行对每个任务进行五次尝试，运行于 [Claude Managed Agents](https://platform.claude.com/docs/zh-CN/managed-agents/overview) 上，并使用其自身的[多智能体编排](https://platform.claude.com/docs/zh-CN/managed-agents/multiagent-orchestration)（`multiagent_20261001` 类型）：一个预定义智能体作为子智能体，内联智能体和动态工作流均已关闭。
    * **配置：** 单独使用 Claude Opus 5.5，分别在 `medium`（其默认值）和 `low` 下运行；以及 `medium` 下的 Claude Opus 5.5 搭配 `medium` 下的 Claude Haiku 5.5 子智能体，一次不添加提示，一次使用一个 113 词的委派提示作为编排器的系统提示。第五种配置使用了更长的提示，未予报告。评估未设置任何思考选项，因此思考按默认方式运行：开启，并由模型选择何时思考。
    * **工具：** 每个智能体都使用两个自定义工具来代替平台的内置工具：一个 shell 和一个文件编辑器，由评估在每次尝试自己的沙箱中运行。作为评估安全设置的一部分，对它们的调用会经过一次额外检查。在文档所述的设置中，[权限策略不适用于自定义工具](https://platform.claude.com/docs/zh-CN/managed-agents/permission-policies#custom-tools)。
    * **结果：** 通过的尝试比例和每次尝试的成本分别为：单独使用 `medium` 下的 Claude Opus 5.5 时为 72.1% 和 $4.78 USD；使用子智能体且不添加提示时为 68.8% 和 $3.35 USD；使用子智能体并添加提示时为 67.9% 和 $3.27 USD；单独使用 `low` 时为 63.0% 和 $1.84 USD。
    * **与单独使用 `medium` 下的 Claude Opus 5.5 相比：** 添加提示时，成本降低 32%（95% 区间为 29% 到 35%），通过率降低 4.2 个百分点（1.3 到 7.1）。不添加提示时，成本降低 30%（27% 到 33%），通过率降低 3.3 个百分点（0.8 到 5.8）。单独使用 `low` 时，成本降低 62%（59% 到 64%），通过率降低 9.1 个百分点（5.0 到 13.3），这一比较是在运行后添加的。差异在每次运行内按任务配对，区间为基于 134 个任务的 bootstrap 区间。
    * **与较低努力程度相比：** 将一部分任务发送到 `low`、其余发送到 `medium`，会在这两个结果之间得到一条直线，这是计算结果而非实际运行。在各自的成本下，使用子智能体的配置在添加提示时比该直线高 0.5 个百分点（低 2.3 到高 3.2），不添加提示时高 1.1 个百分点（低 1.2 到高 3.3）：既不高于也不低于该直线。每通过一次尝试所需的成本为：单独使用 `medium` 时 $6.63 USD，使用子智能体且不添加提示时 $4.86 USD，添加提示时 $4.81 USD，使用 `low` 时 $2.92 USD。
    * **定价方式：** 成本为按标价、使用 5 分钟提示缓存的令牌成本，每个请求按其自身的提示长度定价（[长上下文定价](https://platform.claude.com/docs/zh-CN/about-claude/pricing#long-context-pricing)），不包括会话运行时间。成本未计入评估对工具调用的额外检查；若计入该检查，添加提示时的节省为 27%（25% 到 30%）。
    * **何为失败：** 如果一次尝试被安全检查终止、平台的安全防护拒绝了某一轮，或者评估因无法确认其安全检查看到了每一步而将其作废，则该尝试计为失败。作废的尝试只出现在使用子智能体的配置上：添加提示时 2,010 次中有 72 次，其中 57 次出现在同一次运行中。重新运行该配置在那次运行中的部分后，其通过率比单独使用 Claude Opus 5.5 低 3.6 个百分点（1.0 到 6.3）。若改为在双方都排除所有被安全检查终止或被评估作废的尝试，其通过率低 1.7 个百分点（低 4.4 到高 1.1）。因此，部分损失（但并非全部）来自评估自身的规则。单独使用 `medium` 下的 Claude Opus 5.5 时，有一次尝试在三小时限制处被截断，没有结果，因此被排除；而在使用子智能体时，同样的截断计为失败。

## 后续步骤

<CardGroup cols={2}>
  <Card title="提示缓存" icon="database" href="https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-caching">
    本页最大的免费收益：设置、生命周期和诊断。
  </Card>

  <Card title="Effort" icon="gauge" href="https://platform.claude.com/docs/zh-CN/build-with-claude/effort">
    在单个模型内以智能换取延迟和成本。
  </Card>

  <Card title="选择合适的模型" icon="settings" href="https://platform.claude.com/docs/zh-CN/about-claude/models/choosing-a-model">
    评估整个 Claude 模型家族的能力、速度和成本。
  </Card>

  <Card title="会话预算" icon="coins" href="https://platform.claude.com/docs/zh-CN/managed-agents/budgets">
    为 Managed Agents 会话设置硬性美元上限。
  </Card>

  <Card title="定价" icon="dollar-sign" href="https://platform.claude.com/docs/zh-CN/about-claude/pricing">
    查看每个 Claude 模型当前的每令牌定价。
  </Card>

  <Card title="Cookbook：Claude API 上的成本优化" icon="book" href="https://platform.claude.com/cookbook/cost-optimization-cost-optimization">
    在可运行的 notebook 中将这些手段逐一应用于一个可工作的智能体，并在每一步后查看每任务成本。
  </Card>

  <Card title="网络研讨会：在 Claude Platform 上构建" icon="play" href="https://www.anthropic.com/webinars/building-on-the-claude-platform-claude-fable-5-and-model-orchestration-patterns">
    观看顾问和编排器模式的演示讲解。
  </Card>
</CardGroup>
