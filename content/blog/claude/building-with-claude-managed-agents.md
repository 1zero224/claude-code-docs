Getting an agent into production takes more than a good prompt. The agent needs somewhere to run the code it writes, credentials to reach your data, observable sessions, and infrastructure that scales with usage. On the Applied AI team, we work at the intersection of product, research, and the customers building on Claude—and we see the same pattern repeatedly: infrastructure is what separates a prototype from a production agent. All too often, teams burn development cycles on security, state management, permissioning, and harness tuning.
[Claude Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview), our suite of composable APIs for building and deploying production-grade agents, pairs an agent harness tuned for performance with production infrastructure, allowing teams to go from prototype to launch in days rather than months. In this post, we'll cover the evolution of Anthropic’s agentic building blocks, why we built Claude Managed Agents, and how teams are using it in production today.
## **Evolving the agent architecture**
When we opened up Claude to developers in 2023, the API was deliberately simple: tokens in, tokens out. You sent a prompt, Claude returned a completion, and you built the harness and underlying infrastructure.
The API grew steadily richer over the years, but the contract underneath never changed: one request, one model turn, and your application decides what happens next. For a long time, that was enough. Summarizing a document, classifying a support ticket, rewriting a block of text—the kind of work that fits comfortably in a single turn.
Over time, however, the tasks people wanted to hand off stopped fitting. They wanted Claude to carry a task all the way through, look something up, act on it, see what changed, and decide what to do next. And they wanted it to operate *in* the systems their work already ran on, like a codebase, internal wiki, or ticketing system.
With the API, turning Claude into an agent meant building your own loop: ask the model what to do, run the tool, feed the result back, and repeat. You were responsible for building and deploying the agent scaffolding, which may need tuning as models evolve. For agents that require full customization, this approach makes sense. For agentic workloads that are more predictable and less complex, optimizing harnesses as models and products evolved became tedious.
![__wf_reserved_inherit](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a298c28f950480f89a8dfcf_01%20_%20Messages%20API.png)
[Claude Code](https://claude.com/product/claude-code), the agentic coding tool we launched in 2025 that lets Claude interact directly with your codebase, contained our own version of that harness: the loop, tool execution, subagents, context management, and rich capabilities that made it an effective agent. Developers naturally wanted similar harness machinery for their own agents across various domains.
To enable teams to build agents on top of the Claude Code harness, we released [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview). Claude Agent SDK gives developers tools to build their own agents on the same machinery that runs Claude Code instead of maintaining a homegrown loop. For a lot of teams, this is when agents became practical: the harness arrived already tuned for Claude with infrastructure primitives and it kept improving as Claude Code did.
Even with a harness, though, deploying agents in production environments can be challenging for several reasons:
  - **Hosting and scaling.** Where does the agent run, how long can a process stay alive for a multi-hour task, and what scales it when usage grows?
  - **Session management.** Where does an agent's history and progress live? Can a run survive an interruption and resume unencumbered? Can you go back and inspect what happened in previous sessions?
  - **Filesystem management. **Doing real work means producing artifacts: editing code, writing files, building outputs. Where does the agent get a workspace to act on, and what happens to that workspace between runs?
  - **Execution isolation.** The code Claude writes has to execute somewhere. What's the blast radius if it's wrong, and what boundary would you actually trust in production?
  - **Credentials. **The agent needs access to your systems. How does it get that access without exposing proprietary information to the code it generates?
  - **Observability. **When an agent works autonomously for an hour and does something surprising, can you reconstruct every step it took?
With the Agent SDK, many elements of the aforementioned production infrastructure are provided through Claude Code’s machinery. The agent gets a real filesystem to work in, session state is persisted locally or on external storage, and observability is exportable through OpenTelemetry into whatever monitoring stack you already run.
![__wf_reserved_inherit](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a298c53aaeeee508f2b3166_02%20_%20Claude%20Agent%20SDK.png)
However, as teams increasingly built agents that moved out of local development into production, they needed a way to deploy them at scale and with managed infrastructure. And as models and their surrounding harnesses become more advanced–running longer, executing more code, touching more systems, and taking more actions– scaling, security, and sandboxing became more challenging.
Several of these hurdles stem from a common architectural choice: agent harnesses often run *inside the same container* as the filesystem it works on. A container has to spin up (paying a startup cost) before Claude can think, the agent along with code execution lives right next to your credentials, and when the container dies, the run dies with it.
Managed Agents solves these problems by [decoupling the brain from the hands](https://www.anthropic.com/engineering/managed-agents). The harness that calls Claude runs separately from the sandbox where code executes, and the session–an append-only log of every model call, tool call, and result–connects the two. Claude can start reasoning before any container exists, the sandbox stays far away from your credentials, and a whole run can be reconstructed from its session at any point.
![__wf_reserved_inherit](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a298c97d4a887f2666a50b6_03%20_%20Claude%20Managed%20Agents.png)
## **When and why to use Claude Managed Agents **
When building with Managed Agents, users define the task, the tools, and the guardrails, and Anthropic runs the agent on our infrastructure and handles the agentic loop underneath: how to give an agent an execution environment to call tools, how to recover when something fails, multi-agent orchestration, and more.
When the harness doesn’t evolve alongside model intelligence, [the agent breaks down](https://www.anthropic.com/engineering/harness-design-long-running-apps). On Claude Sonnet 4.5, an agent would rush to finish as it neared the end of its context, cutting work short rather than using the room it had left—a pattern called "context anxiety." Our fix was to add context resets to the harness, baking in an assumption that Claude needed help staying coherent near the limit. That assumption didn't survive the next model. On Claude Opus 4.5, the behavior was gone, and the resets we'd added were just overhead.
For most organizations, maintaining a harness is overhead that doesn't differentiate their product. Harnesses have to be tuned for certain model behaviors; primitives like compaction, tool execution, and caching works differently on Claude than other models. With Claude Managed Agents, the harness evolves alongside the model, allowing teams to focus on what will differentiate their agents: **context management and domain expertise.**
To enable developers to configure the context and tools necessary to build effective agents, Managed Agents is built around three primary resources: agents, environments, and sessions. An *agent* is a configuration: a model, a prompt, a set of tools, and the guardrails around them. An *environment* is the execution context the agent runs in: the sandbox container, its networking rules, and the packages pre-installed in it, hosted on our cloud or on infrastructure you control. Each run is a *session*, which pairs an agent with an environment and gets its own isolated sandbox instance. Sessions persist their full event history, sandbox state, and outputs server-side, so long-running work can pause, resume cleanly, and be traced step by step after the fact. With Managed Agents, you can define an agent and an environment once, then run many sessions against the same configuration as your workload grows.
![__wf_reserved_inherit](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a29a18bb07e245f8389acb9_04%20_%20Agents_%20environments_%20sessions%20(2).png)
## **Building for production and scale on Managed Agents**
Within Applied AI, we see agents go from prototype to production both inside Anthropic and across our customers’ systems, across coding, finance, support, legal, and a dozen other domains. This gives us a clear view of what separates a demo from a production-ready agent and where teams often get stuck.
Below, we share the most common reasons to build on a managed service like Claude Managed Agents:
**1. Credentials are kept out of the sandbox.** When everything runs in one container, the code Claude generates sits right next to your credentials, so prompt injections could lead the model to leak a token by convincing the model to read its own environment. We can protect against this by setting up robust guardrails within the same container, but decoupling the architecture enables a much more secure approach by keeping credentials out of the sandbox entirely. Tokens for tools like MCPs, CLIs, and GitHub repos live in a separate vault, and a proxy fetches them and decrypts them only on demand. Managed Agents provides [Vaults](https://platform.claude.com/docs/en/managed-agents/vaults) that handle credentials out-of-the-box, so you don’t need to run your own secret store, transmit tokens on every call, or lose track of which end user an agent acted on behalf of. Vault credentials are protected with envelope encryption before storage, and retrieval requires a signed request token for verification.
![__wf_reserved_inherit](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a29a19cebb4eb7adac0a8ec_05%20_%20Managed%20Agents%20runtime%20(1).png)
**2. Lower latency from eliminated sandbox overhead.** Latency is a metric that is top-of-mind for many enterprise teams, since users acutely feel when they’re waiting for Claude to respond. Without the Managed Agents architecture, a container has to be spun up for every session, even the ones where the agent only needs to think and never runs a tool. That setup time is wasted, and the user feels it as a delay before the first response. With Managed Agents, Claude begins reasoning immediately while the environment spins up in parallel, and sessions that never run a tool skip the container entirely. This means the user sees the first token without waiting on container startup, and the environment is ready by the time the agent needs to run something. In our testing, that cut the time-to-first-token by roughly 60% in the median case (p50) and by over 90% in the slowest cases (p95).
**3. Reliable, persistent sessions that enable session management, observability, and memory. **Instead of request/response, Managed Agents thinks in terms of *events. *A session is an ongoing stream of events: every model call, tool call, and result, are appended to a log that lives outside the process running the agent. With this architecture, you get real-time updates as events stream in while the agent works, and you can resume any session later with no database or save-points to manage. History is preserved between interactions unless you delete the session, and when a session goes idle its container is checkpointed so you can pick up cleanly from where it paused. And because the whole run is already a record of events, observability and memory come with it: the Claude Developer Console offers a native visual timeline view of your agent sessions, and a debugging experience that allows you to examine any transcript in-depth. Managed Agents also comes with features like Memory and Dreaming that also use this session durability. [Dreaming](https://platform.claude.com/docs/en/managed-agents/dreams) is a scheduled process that reviews your agent sessions and memory stores, extracts patterns, and curates memories so your agents improve over time. Dreaming refines memory between sessions so that it can improve from recurring mistakes and user preferences by reading from the persistent session logs.
**4. Flexibility in Anthropic-managed or self-hosted cloud containers.** By default, with Managed Agents, you can delegate both orchestration and tool execution to Anthropic-managed cloud containers. This makes hosting and scaling simple and easy, delivering a faster path to production. Because the brain is decoupled from the hands in Managed Agents, the hands can live anywhere, including inside your Virtual Private Cloud (VPC). Thus, we also offer [self-hosted sandboxes](https://platform.claude.com/docs/en/managed-agents/self-hosted-sandboxes) for teams that want control over tool execution, so the agent’s code, filesystem, and network egress never leave their environment. We also provide [MCP tunnels](https://platform.claude.com/docs/en/agents-and-tools/mcp-tunnels/overview), which let you connect Claude to Model Context Protocol (MCP) servers that run inside your private network. So self-hosted sandboxes control *where the agent’s code executes*, and MCP tunnels control *how Anthropic reaches MCP servers in your network*, giving you the ability to control exactly what stays inside your boundary.
![__wf_reserved_inherit](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a298e427c7a804ea4295163_image7.png)
*The built-in observability console for Claude Managed Agents records every event, so you can scrub the timeline, open any step, and read its raw payload.*
Beyond these features, additional capabilities include outcomes that let an agent grade its own work against a rubric, multiagent orchestration, permission policies, and webhooks. Learn more [here](https://platform.claude.com/docs/en/managed-agents/overview).
### **How customers are building on Managed Agents today**
Across industries, customers are already shipping agents in production with Claude Managed Agents. Here are a few examples:
  - [Notion](https://claude.com/customers/notion) runs its Custom Agents on Managed Agents: teams assign work to Claude straight from a task board, Claude picks up the docs, meeting notes, and connected data around each task, and the finished code, decks, and sites land back in the workspace for review. Dozens of tasks run in parallel, and their team has described an early prototype turning roughly twelve hours of work into twenty minutes.
  - [Rakuten](https://claude.com/customers/rakuten) used Managed Agents to ship specialist agents across product, sales, marketing, and finance, each live within about a week.[ ](https://claude.com/customers/sentry)
  - [Sentry](https://claude.com/customers/sentry) paired its Seer debugging agent with a Claude agent that writes the patch and opens the PR, built in weeks instead of months by a single engineer.[ ](https://claude.com/blog/claude-managed-agents)
  - [Asana](https://claude.com/blog/claude-managed-agents) built AI Teammates that pick up tasks inside projects, and[ Atlassian](https://claude.com/blog/claude-managed-agents) put developer agents into Jira workflows.
## **Getting started with Claude Managed Agents**
We built Managed Agents to make it as easy as possible to spin up agents through Claude Code and the Claude Developer Console at [platform.claude.com](http://platform.claude.com). The Console’s quickstart, for example, lets you start from an agent template or describe an agent in plain language, then turn it into a production-ready agent you can secure and deploy in minutes.
![__wf_reserved_inherit](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a298e9b866a4402a3c9bb5d_image5.png)
*The agent quickstart at platform.claude.com: start from a template or describe what you want to build.*
![__wf_reserved_inherit](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a298ebdff6d26839e052c63_image9.png)
*A few steps later: the agent is created, the environment is configured, and a session is live. The console streams the run as it happens.*
In Claude Code, the [/claude-api skill ](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/claude-api-skill)is provided by default and provides Claude with detailed, up-to-date reference material for building applications on Claude Managed Agents. We highly recommend that you utilize it for the best practices on setting up your Managed Agents application. Get started by running /claude-api managed-agents-onboard for an interview-driven walkthrough for setting up a new Managed Agent from scratch.
![__wf_reserved_inherit](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6a298ef3765ce453971174cd_image6.png)
## **The future of building managed agents**
As teams share what they’re building with Managed Agents, we see that the time they used to spend on production infrastructure now goes to what differentiates their agents: managing context and tailoring the experience to users. Now, when a new model comes out, you update your agent to use it, rerun your evals, and ship the improvement without touching the architecture underneath.
We’re excited to see what you build.
[***Get started***](https://platform.claude.com/docs/en/managed-agents/overview)*** with Claude Managed Agents.***
*This article was written by Gagan Bhat and Isabella He, Members of Technical Staff on Anthropic’s Applied AI team. They'd like to thank Hema Thanki, Jess Yan, and Molly Vorwerck for their contributions.*
No items found.
[Prev](https://claude.com/blog/building-with-claude-managed-agents#)Prev
0/5
[Next](https://claude.com/blog/building-with-claude-managed-agents#)Next
eBook
##
[](https://claude.com/blog/building-with-claude-managed-agents#)
![](https://cdn.prod.website-files.com/6889473510b50328dbb70ae6/6889473610b50328dbb70b58_placeholder.svg)
![](https://cdn.prod.website-files.com/6889473510b50328dbb70ae6/6889473610b50328dbb70b58_placeholder.svg)![](https://cdn.prod.website-files.com/6889473510b50328dbb70ae6/6889473610b50328dbb70b58_placeholder.svg)
FAQ
No items found.
[](https://claude.com/blog/building-with-claude-managed-agents#)
## Related posts
Explore more product news and best practices for teams building with Claude.
![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6903d228c83775fcc75f4e6d_74409af25137110ac04cc39e4d5ea0a2fbcea421-1000x1000.svg)
Sep 2, 2026
### Building commerce agents with Claude
Product announcements
[Building commerce agents with Claude](https://claude.com/blog/building-with-claude-managed-agents#)Building commerce agents with Claude
[Building commerce agents with Claude](https://claude.com/blog/claude-for-commerce-agents)Building commerce agents with Claude
![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6903d222061abf091318fb82_423062049d4676b41d52b16068cbb5e21603190e-1000x1000.svg)
Sep 2, 2026
### A guide to the anatomy of effective commerce agents
Agents
[A guide to the anatomy of effective commerce agents](https://claude.com/blog/building-with-claude-managed-agents#)A guide to the anatomy of effective commerce agents
[A guide to the anatomy of effective commerce agents](https://claude.com/blog/the-anatomy-of-effective-commerce-agents)A guide to the anatomy of effective commerce agents
![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6903d225485fe31f1ed2d9a1_db28a79c9f4492b8471009d4c20e900f234ece48-1000x1000.svg)
Aug 26, 2026
### How Warp builds self-improving agents on Claude
Agents
[How Warp builds self-improving agents on Claude](https://claude.com/blog/building-with-claude-managed-agents#)How Warp builds self-improving agents on Claude
[How Warp builds self-improving agents on Claude](https://claude.com/blog/how-warp-builds-self-improving-agents-on-claude)How Warp builds self-improving agents on Claude
![](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6903d2238ce207f9b2011d3f_e44a6b53398f189b9fd0d4f70516db614ac84db3-1000x1000.svg)
Aug 13, 2026
### Self-service data analytics in Slack: how Anthropic deploys Claude Tag for ad-hoc questions
Agents
[Self-service data analytics in Slack: how Anthropic deploys Claude Tag for ad-hoc questions](https://claude.com/blog/building-with-claude-managed-agents#)Self-service data analytics in Slack: how Anthropic deploys Claude Tag for ad-hoc questions
[Self-service data analytics in Slack: how Anthropic deploys Claude Tag for ad-hoc questions](https://claude.com/blog/self-service-data-analytics-in-slack-how-anthropic-deploys-claude-tag-for-ad-hoc-questions)Self-service data analytics in Slack: how Anthropic deploys Claude Tag for ad-hoc questions
## Transform how your organization operates with Claude
See pricing
[See pricing](https://claude.com/pricing#api)See pricing
Contact sales
[Contact sales](https://claude.com/contact-sales)Contact sales
Get the developer newsletter
Product updates, how-tos, community spotlights, and more. Delivered monthly to your inbox.
[Subscribe](https://claude.com/blog/building-with-claude-managed-agents#)Subscribe
Please provide your email address if you'd like to receive our monthly developer newsletter. You can unsubscribe at any time.
Thank you! You’re subscribed.
Sorry, there was a problem with your submission, please try again later.
[Homepage](https://claude.com)Homepage
[Next](https://claude.com/blog/building-with-claude-managed-agents#)Next
Thank you! Your submission has been received!
Oops! Something went wrong while submitting the form.
[Anthropic](https://www.anthropic.com/)Anthropic
© [year] Anthropic PBC
Products
  -
Claude
[Claude](https://claude.com/product/overview)Claude
  -
Claude Code
[Claude Code](https://claude.com/product/claude-code)Claude Code
  -
Claude Code for Enterprise
[Claude Code for Enterprise](https://claude.com/product/claude-code/enterprise)Claude Code for Enterprise
  -
Claude Cowork
[Claude Cowork](https://claude.com/product/cowork)Claude Cowork
  -
@Claude
[@Claude](https://claude.com/product/tag)@Claude
  -
Claude Design
[Claude Design](https://claude.com/product/design)Claude Design
  -
Claude Science
[Claude Science](https://claude.com/product/claude-science)Claude Science
  -
Claude Security
[Claude Security](https://claude.com/product/claude-security)Claude Security
  -
Download app
[Download app](https://claude.com/download)Download app
  -
Pricing
[Pricing](https://claude.com/pricing)Pricing
  -
Log in
[Log in](https://claude.ai/login)Log in
Features
  -
Claude in Chrome
[Claude in Chrome](https://claude.com/claude-in-chrome)Claude in Chrome
  -
Claude for Microsoft 365
[Claude for Microsoft 365](https://claude.com/claude-for-microsoft-365)Claude for Microsoft 365
  -
Skills
[Skills](https://claude.com/skills)Skills
Models
  -
Mythos
[Mythos](https://www.anthropic.com/claude/mythos)Mythos
  -
Fable
[Fable](https://www.anthropic.com/claude/fable)Fable
  -
Opus
[Opus](https://www.anthropic.com/claude/opus)Opus
  -
Sonnet
[Sonnet](https://www.anthropic.com/claude/sonnet)Sonnet
  -
Haiku
[Haiku](https://www.anthropic.com/claude/haiku)Haiku
Solutions
  -
AI agents
[AI agents](https://claude.com/solutions/agents)AI agents
  -
Code modernization
[Code modernization](https://claude.com/solutions/code-modernization)Code modernization
  -
Coding
[Coding](https://claude.com/solutions/coding)Coding
  -
Commerce
[Commerce](https://claude.com/solutions/commerce)Commerce
  -
Customer support
[Customer support](https://claude.com/solutions/customer-support)Customer support
  -
Cybersecurity
[Cybersecurity](https://claude.com/solutions/cybersecurity)Cybersecurity
  -
Enterprise
[Enterprise](https://claude.com/solutions/enterprise)Enterprise
  -
Financial services
[Financial services](https://claude.com/solutions/financial-services)Financial services
  -
Government
[Government](https://claude.com/solutions/government)Government
  -
Healthcare
[Healthcare](https://claude.com/solutions/healthcare)Healthcare
  -
Higher education
[Higher education](https://claude.com/solutions/education)Higher education
  -
K-12 teachers
[K-12 teachers](https://claude.com/solutions/teachers)K-12 teachers
  -
Legal
[Legal](https://claude.com/solutions/legal)Legal
  -
Life sciences
[Life sciences](https://claude.com/solutions/life-sciences)Life sciences
  -
Nonprofits
[Nonprofits](https://claude.com/solutions/nonprofits)Nonprofits
  -
Small business
[Small business](https://claude.com/solutions/small-business)Small business
Claude Platform
  -
Overview
[Overview](https://claude.com/platform/api)Overview
  -
Developer docs
[Developer docs](https://platform.claude.com/docs)Developer docs
  -
Pricing
[Pricing](https://claude.com/pricing#api)Pricing
  -
Ecosystem
[Ecosystem](https://claude.com/ecosystem)Ecosystem
  -
Marketplace
[Marketplace](https://claude.com/platform/marketplace)Marketplace
  -
Claude on AWS
[Claude on AWS](https://claude.com/partners/claude-on-aws)Claude on AWS
  -
Google Cloud
[Google Cloud](https://claude.com/partners/google-cloud)Google Cloud
  -
Microsoft Foundry
[Microsoft Foundry](https://claude.com/partners/microsoft-foundry)Microsoft Foundry
  -
Regional compliance
[Regional compliance](https://claude.com/regional-compliance)Regional compliance
  -
Console login
[Console login](https://platform.claude.com/)Console login
Resources
  -
Blog
[Blog](https://claude.com/blog)Blog
  -
Claude partner network
[Claude partner network](https://claude.com/partners)Claude partner network
  -
Community
[Community](https://claude.com/community)Community
  -
Connectors
[Connectors](https://claude.com/connectors)Connectors
  -
Courses
[Courses](https://academy.claude.com/courses)Courses
  -
Customer stories
[Customer stories](https://claude.com/customers)Customer stories
  -
Engineering at Anthropic
[Engineering at Anthropic](https://www.anthropic.com/engineering)Engineering at Anthropic
  -
Events
[Events](https://www.anthropic.com/events)Events
  -
Plugins
[Plugins](https://claude.com/plugins)Plugins
  -
Powered by Claude
[Powered by Claude](https://claude.com/partners/powered-by-claude)Powered by Claude
  -
Service partners
[Service partners](https://claude.com/blog/building-with-claude-managed-agents#)Service partners
  -
Tutorials
[Tutorials](https://academy.claude.com/tutorials)Tutorials
  -
Use cases
[Use cases](https://academy.claude.com/use-cases)Use cases
Company
  -
Anthropic
[Anthropic](https://www.anthropic.com/)Anthropic
  -
Careers
[Careers](https://www.anthropic.com/careers)Careers
  -
Policy
[Policy](https://www.anthropic.com/policy)Policy
  -
Economic Futures
[Economic Futures](https://www.anthropic.com/economic-futures)Economic Futures
  -
Research
[Research](https://www.anthropic.com/research)Research
  -
News
[News](https://www.anthropic.com/news)News
  -
Policy on the AI Exponential
[Policy on the AI Exponential](https://www.anthropic.com/policy-on-the-ai-exponential)Policy on the AI Exponential
  -
Responsible Scaling Policy
[Responsible Scaling Policy](https://www.anthropic.com/news/announcing-our-updated-responsible-scaling-policy)Responsible Scaling Policy
  -
Security and compliance
[Security and compliance](https://trust.anthropic.com/)Security and compliance
  -
Transparency
[Transparency](https://anthropic.com/transparency)Transparency
Programs
  -
Startups
[Startups](https://claude.com/programs/startups)Startups
  -
Scientists
[Scientists](https://claude.com/programs/team-plan-for-scientists)Scientists
Help and security
  -
Availability
[Availability](https://www.anthropic.com/supported-countries)Availability
  -
Check files
[Check files](https://claude.com/check-files)Check files
  -
Report abuse
[Report abuse](https://claude.com/form/anthropic-content-reporting)Report abuse
  -
Status
[Status](https://status.anthropic.com/)Status
  -
Support center
[Support center](https://support.claude.com/en/)Support center
Terms and policies
  -
Privacy choices

### Cookie settings

 We use cookies to deliver and improve our services, analyze site usage, and if you agree, to customize or personalize your experience and market our services to you. You can read our Cookie Policy [here](https://www.anthropic.com/legal/cookies).

  Customize cookie settings   Reject all cookies   Accept all cookies

###### Necessary

Enables security and basic functionality.

 Required

###### Analytics

Enables tracking of site performance.

 Off

###### Marketing

Enables ads personalization and tracking.

 Off

   Save preferences

  -
Privacy policy
[Privacy policy](https://www.anthropic.com/legal/privacy)Privacy policy
  -
Responsible disclosure policy
[Responsible disclosure policy](https://www.anthropic.com/responsible-disclosure-policy)Responsible disclosure policy
  -
Terms of service: Commercial
[Terms of service: Commercial](https://www.anthropic.com/legal/commercial-terms)Terms of service: Commercial
  -
Terms of service: Consumer
[Terms of service: Consumer](https://www.anthropic.com/legal/consumer-terms)Terms of service: Consumer
  -
Terms of Service: US K-12
[Terms of Service: US K-12](https://anthropic.com/legal/k12-terms)Terms of Service: US K-12
  -
Data Processing Agreement: US K-12
[Data Processing Agreement: US K-12](https://anthropic.com/legal/k12-dpa)Data Processing Agreement: US K-12
  -
Usage policy
[Usage policy](https://www.anthropic.com/legal/aup)Usage policy
[x.com](https://x.com/claudeai)x.com
[x.com](https://www.threads.com/@claudeai)x.com
[LinkedIn](https://www.linkedin.com/showcase/claude/)LinkedIn
[YouTube](https://www.youtube.com/@anthropic-ai)YouTube
[Instagram](https://www.instagram.com/claudeai)Instagram
English
[](https://claude.com/blog-product/claude-platform)
Claude Platform
[](https://claude.com/blog-usecases/agents)
Agents
