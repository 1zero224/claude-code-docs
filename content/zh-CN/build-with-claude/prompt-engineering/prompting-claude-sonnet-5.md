---
title: 为 Claude Sonnet 5 编写提示
url: https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/prompting-claude-sonnet-5
description: Claude Sonnet 5 的行为差异和提示模式，涵盖 effort、自适应思考默认设置以及工具使用。
---

本指南介绍 Claude Sonnet 5 特有的提示模式。有关该模型的规格，请参阅 [Claude Sonnet 5](https://platform.claude.com/docs/zh-CN/models/sonnet-5/overview)。有关适用于所有当前 Claude 模型的技巧，请参阅[提示最佳实践](https://platform.claude.com/docs/zh-CN/build-with-claude/prompt-engineering/claude-prompting-best-practices)。

Claude Sonnet 5 在编码和智能体任务方面具有特别的优势。它在现有的 Claude Sonnet 4.6 提示上开箱即用，表现良好。本指南中的模式涵盖了最常需要调整的行为。

## 响应长度和冗长程度

Claude Sonnet 5 会根据任务的复杂程度来调整响应长度，而不是默认采用固定的冗长程度。这通常意味着在简单查询上给出较短的回答，在开放式分析上给出较长的回答。

如果您的产品依赖于某种特定的输出风格或冗长程度，您可能需要调整提示。例如，要降低冗长程度，您可以添加：

```text wrap
Provide concise, focused responses. Skip non-essential context, and keep examples minimal.
```

如果您发现特定类型的冗长（例如过度解释），可以在提示中添加额外的指令来防止这种情况。展示 Claude 如何以适当简洁程度进行沟通的正面示例，往往比负面示例或告诉模型不要做什么的指令更有效。

## 校准 effort 和思考深度

[effort 参数](https://platform.claude.com/docs/zh-CN/build-with-claude/effort)允许您在 Claude 的智能程度与令牌消耗之间进行调整，以牺牲部分能力换取更快的速度和更低的成本。在 Claude Sonnet 5 上，effort 默认为 `high`，与 Claude Sonnet 4.6 相同。对于最困难的编码和智能体任务，请将 effort 提高到 `xhigh`。您可以尝试其他 effort 级别，以进一步调整令牌使用量和智能程度：

* **`max`：** 绝对最大能力，对令牌消耗没有任何限制。
* **`xhigh`：** 超高 effort 是最困难的编码和智能体用例的推荐设置。
* **`high`：** 默认值。此设置在大多数用例中平衡了令牌使用量和智能程度。
* **`medium`：** 适用于对成本敏感、需要在牺牲部分智能的情况下减少令牌使用量的用例。
* **`low`：** 保留用于简短、范围明确的任务，以及对延迟敏感但对智能要求不高的工作负载。

迁移时的大致跨模型对应关系：medium 级别的 Claude Sonnet 5 在智能程度上与 high 级别的 Claude Sonnet 4.6 相当，而 high 级别的 Claude Sonnet 5 与 max 级别的 Claude Sonnet 4.6 相当。进行基准测试时，请按观察到的思考长度而非 effort 名称进行匹配。

Claude Sonnet 5 会严格遵守 effort 级别，尤其是在低端级别。在 `low` 和 `medium` 级别下，模型会将工作范围限定在所要求的内容上，而不会超出请求范围。这对延迟和成本有利，但在以 `low` effort 运行中等复杂度的任务时，存在一定的思考不足风险。

如果您在复杂问题上观察到推理较浅，请将 effort 提高到 `high` 或 `xhigh`，而不是通过提示来绕过这个问题。如果出于延迟考虑需要将 effort 保持在 `low`，请添加有针对性的指导：

```text wrap
This task involves multistep reasoning. Think carefully through the problem before responding.
```

在 Claude Sonnet 5 上，[自适应思考](https://platform.claude.com/docs/zh-CN/build-with-claude/thinking)默认开启。不带 `thinking` 字段的请求会以自适应思考方式运行。这与 Claude Sonnet 4.6 不同，在 Claude Sonnet 4.6 上，相同的请求会在不进行思考的情况下运行。要完全关闭思考，请传入 `thinking: {type: "disabled"}`。由于 `max_tokens` 是对总输出（思考加响应文本）的硬性限制，对于在 Claude Sonnet 4.6 上不进行思考运行的工作负载，请重新审视该值。如果您之前在 Claude Sonnet 4.6 上关闭了思考，请尝试在 Claude Sonnet 5 上开启思考并使用较低的 effort 级别。

自适应思考的触发行为是可引导的。如果您发现模型输出思考块的频率超出预期（这可能在使用大型或复杂的系统提示时发生），请添加指导来引导它。与往常一样，请衡量任何提示更改对性能的影响。示例：

```text wrap
Thinking adds latency and should only be used when it will meaningfully improve answer quality, typically for problems that require multistep reasoning. When in doubt, respond directly.
```

反之，如果您以 `medium` 运行困难的工作负载并发现思考不足，首要手段是提高 effort。如果您需要更精细的控制，请直接通过提示进行要求。

Claude Sonnet 5 不支持手动扩展思考（`thinking: {type: "enabled", budget_tokens: N}`），使用时会返回 400 错误。该功能在 Claude Sonnet 4.6 上已被弃用，现已移除。请改用自适应思考配合 effort 参数。

<Note>
  如果您以 `high`、`xhigh` 或 `max` effort 运行 Claude Sonnet 5，请在 `max_tokens` 中预留余量，以便模型有空间进行思考和工具调用。在长任务中，自适应思考可能会占用大量预算；如果预算紧张，您可能会看到一个几乎全部是思考内容的响应，随后是被截断的回答以及 `stop_reason: "max_tokens"`。提高 `max_tokens` 或降至 `medium` effort 即可解决此问题。由于 Claude Sonnet 5 使用了新的分词器，对于相同文本会产生约多 30% 的令牌，因此为 Claude Sonnet 4.6 调整的 `max_tokens` 限制可能会截断等量的输出。确切的增幅取决于内容和工作负载的形态。
</Note>

## 工具使用触发

默认情况下，Claude Sonnet 5 比 Claude Sonnet 4.6 更具智能体特性，更倾向于调用工具并运行自我验证循环。在禁用思考的情况下，模型不太可能调用工具或考虑进行搜索；如果您在关闭思考时依赖工具调用，请在系统提示中添加明确的引导。effort 也是影响工具使用的手段：`high` 或 `xhigh` effort 设置在智能体搜索和编码中会显著增加工具使用。对于您希望更多使用工具的场景，您还可以调整提示，明确指示模型何时以及如何正确使用其工具。例如，如果您发现模型没有使用您的网络搜索工具，请清楚地描述它应该使用的原因和方式。

## 面向用户的进度更新

Claude Sonnet 5 会在长时间的智能体执行轨迹中定期向用户提供更高质量的更新。如果您添加了强制输出中间状态消息的脚手架（"每进行 3 次工具调用后，总结进度"），请尝试将其移除。如果您发现 Claude Sonnet 5 面向用户的更新在长度或内容上与您的用例不太匹配，请在提示中明确描述这些更新应该是什么样子，并提供示例。

## 更字面化的指令遵循

Claude Sonnet 5 会字面化、明确地理解提示，尤其是在较低的 effort 级别下。它不会默默地将针对某一项的指令推广到另一项，也不会推断您没有提出的请求。这种字面化的好处是精确性，对于使用精心调整的提示、结构化提取以及需要可预测行为的流水线等 API 用例，它通常表现更好。如果您需要 Claude 广泛地应用某条指令，请明确说明其范围（例如，"将此格式应用于每个部分，而不仅仅是第一个部分"）。

## 语气和写作风格

与任何新模型一样，长篇写作的文风可能会发生变化。如果您的产品依赖于特定的语调，请根据新的基线重新评估风格提示。

例如，如果您的产品语气更温暖或更口语化，请添加：

```text wrap
Use a warm, collaborative tone. Acknowledge the user's framing before answering.
```

如果您之前依赖 `temperature` 来实现风格多样性，请注意，在 Claude Sonnet 5 上设置 `temperature`、`top_p` 或 `top_k` 会返回 400 错误。这一限制对于 Sonnet 级别的模型来说是新增的。迁移时请移除这些参数，改用系统提示指令来引导语气和多样性。

## 设计和前端默认设置

在开放式的前端和设计需求中，Claude Sonnet 5 可能会形成一种固定的默认视觉风格。默认的固有风格对某些需求来说可能效果不错，但可能不适合仪表板、开发者工具、金融科技、医疗保健或企业应用。

笼统的指令（"不要使用那种颜色"、"让它简洁、极简"）往往会让模型转向另一种固定的配色方案，而不是产生多样性。有两种方法可以可靠地奏效：

**1. 指定具体的替代方案。** 模型会精确遵循明确的规格：

```text wrap
Design a desktop landing page for a supplement brand called AEFRM.

The visual direction should come from a cold monochrome atmosphere using pale silver-gray tones that gradually deepen into blue-gray and near-black, similar to a misted metallic surface.

The page should feel sharp and controlled, with a strong sense of structure and restraint.

Use this tonal system across the full page instead of introducing bright accent colors.

Use the uploaded image on the hero design in black and white.

The layout should be built with clear horizontal sections and a centered max-width container. Use 4px corner radius consistently across cards, buttons, inputs, and media frames. Margins should feel generous, with enough empty space around each section so the page breathes.

Typography should use a square, angular sans-serif with wider letter spacing than usual, especially in headings and navigation, so the text feels more engineered and less compressed. Headline text can be large and uppercase, while supporting copy remains short and sparse. The sub texts should be written with Alumni Sans SC in 4-6px like tiny little texts on corners bottom centre like that.

For the structure, start with a hero section containing a strong product statement, one short supporting paragraph, and a clean product placeholder or packshot frame. Below that, add a benefit grid with three or four blocks, then a formulation or ingredients section, and finally a cta.

Buttons should be flat and precise, with subtle hover changes using transition: all 160ms ease out where brightness and border contrast shift slightly rather than using dramatic motion.

Color palette should stay within this range:
#E9ECEC, #C9D2D4, #8C9A9E, #44545B, #11171B.
```

**2. 让模型在构建之前提出选项。** 这可以打破默认风格，并让用户拥有控制权。由于 Claude Sonnet 5 不接受 `temperature`，因此这种方法是在多次运行中产生明显不同设计方向的推荐方式。示例提示：

```text wrap
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.
```

为了避开被用户称为"AI slop"美学的通用模式，您可以在系统提示中加入一条简短的指令。[frontend-design 技能](https://github.com/anthropics/claude-code/blob/main/plugins/frontend-design/skills/frontend-design/SKILL.md)提供了更完整的处理方式，但以下代码片段与前面的多样性方法配合使用效果很好：

```text wrap
<frontend_aesthetics>
NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.
</frontend_aesthetics>
```

## 交互式编码产品

在只有单个用户轮次的自主异步编码智能体，与具有多个用户轮次的交互式同步编码智能体之间，令牌用量和行为可能有所不同。为了在编码产品中同时最大化性能和令牌效率，请使用 `xhigh` 或 `high` effort，添加诸如自动模式之类的自主功能，并减少用户所需的人工交互次数。

在限制所需用户交互次数时，重要的是在第一个人类轮次中预先明确任务、意图和相关约束。预先提供明确、清晰且准确的任务描述，有助于最大化自主性和智能程度，同时最大限度地减少用户轮次之后的额外令牌使用。相比之下，在多个用户轮次中逐步传达的模糊或不够明确的提示，往往会相对降低令牌效率，有时还会降低性能。

## 代码审查框架

如果您的代码审查框架（harness）是针对早期模型调整的，您最初可能会在 Claude Sonnet 5 上看到较低的召回率。这很可能是框架的影响，而不是能力退化。当审查提示中包含诸如"仅报告高严重性问题"、"保守一些"或"不要吹毛求疵"之类的表述时，Claude Sonnet 5 可能会比早期模型更忠实地遵循该指令：它可能会同样彻底地调查代码、识别出 bug，然后不报告那些它判定低于您所述标准的发现。这可能表现为模型进行了同样深度的调查，但将更少的调查转化为报告的发现，尤其是对于较低严重性的 bug。精确率通常会上升，但测得的召回率可能会下降，尽管模型底层的 bug 发现能力实际上已经提高。

一些推荐的提示措辞：

```text wrap
Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage - a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a real bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them.
```

即使没有实际的第二步，也可以使用此提示，但将置信度过滤从发现步骤中移出通常会有所帮助。如果您的框架有单独的验证、去重或排序阶段，请明确告诉模型，它在发现阶段的工作是覆盖而不是过滤。

如果您确实希望模型在单次处理中自行过滤，请具体说明报告阈值，而不是使用诸如"重要"之类的定性词语：例如，"报告任何可能导致错误行为、测试失败或误导性结果的 bug；仅省略纯粹的风格或命名偏好之类的小问题。"

请针对您的评估或测试用例的一个子集迭代提示，以验证召回率或 F1 分数的提升。

## 计算机使用

Claude Sonnet 5 通过 `computer_toolset_20260801` 工具集以及 `computer_20251124` 工具版本支持计算机使用。它还支持用于网页内任务的[浏览器使用工具](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/browser-use-tool)（`browser_toolset_20260801`）。[计算机使用](https://platform.claude.com/docs/zh-CN/agents-and-tools/tool-use/computer-use-tool)功能可在各种分辨率下工作，最高支持 2576px / 3.75MP 的分辨率。内部计算机使用测试表明，以 1080p 发送图像可以在性能和成本之间取得良好的平衡。

对于特别注重成本的工作负载，720p 或 1366×768 是成本较低且性能强劲的选项。请自行进行测试，以找到适合您用例的理想设置；尝试不同的 effort 设置也有助于调整模型的行为。
