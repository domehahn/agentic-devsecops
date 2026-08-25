---
title: "Part 4: The Birth of Agentic AI – Autonomous Loops and the Model Context Protocol"
description: "A comprehensive technical essay on agentic AI systems: from planning and reflection patterns to multi-agent architectures and the current architecture and security model of the Model Context Protocol (MCP)."
date: "2026-08-22"
author: "dome"
topics:
  - ai-engineering
  - agentic-engineering
tags:
  - Agentic AI
  - MCP
  - Multi-Agent Systems
  - Tool Integration
  - Software Architecture
draft: false
series:
  slug: "von-netzwerken-zu-agenten"
  title: "From Networks to Agents: The Evolution of AI"
  order: 4
ai:
  assisted: true
  humanReviewed: true
  reviewedBy: "Dominik Hahn"
  reviewedAt: "2026-08-22"
---

A language model with search is not yet an autonomous agent. Nor does a model that can generate exactly one API call automatically become agentic. The architectural break occurs only when a software system repeatedly allows the model—within clearly defined boundaries—to decide **which next step is useful for reaching a goal**: acquire context, select a tool, evaluate a result, revise a plan, correct an error, or terminate the task.[^1][^2]

This changes the role of the LLM. In a conventional chat system, it is primarily a generator. In an agentic system, it becomes part of a **control loop**. Around the model emerge state management, a tool registry, authorization logic, budgets, retry strategies, evaluation, human approval, and audit logging. Autonomy therefore does not reside exclusively “inside the model.” It is a system property of the scaffold in which the model is embedded.

This distinction is critical for both architecture and security. An agent that can only read has a different trust boundary from an agent with write access to Git, shell commands, cloud APIs, and production databases. The more discretion and side effects a system receives, the more important deterministic controls outside the probabilistic model become.[^3]

This fourth part of the **“From Networks to Agents”** series therefore examines two connected developments: first, the emergence of agentic control and planning patterns; second, the standardization of context and tool integrations through the **Model Context Protocol (MCP)**.

---

## 1. What “Agentic AI” Means Technically

The term *agent* predates modern LLMs. In classical AI, agents were described as systems that perceive the state of their environment and select actions in pursuit of goals. One well-known model is the BDI architecture (*Belief–Desire–Intention*), formalized by Rao and Georgeff in the 1990s.[^4]

A cautious analogy to today's LLM agents is:

```text
BELIEFS      -> observed state, tool results, memory, retrieval
DESIRES      -> goal / desired end state
INTENTIONS   -> currently pursued plan or next action
ACTIONS      -> tool calls with real or simulated side effects
```

This mapping is **an architectural analogy**, not a historical identity. Modern LLM agents do not automatically implement a formal BDI system. The comparison is still useful because it separates three things that prompts often conflate: What does the system know? What should be achieved? And what specific action is it currently pursuing?

A minimal agentic loop can look like this:

```text
while not done:
    observation = read_state()
    action = model.choose_next_action(goal, observation, policy)
    checked_action = policy_engine.validate(action)
    result = execute(checked_action)
    update_state(result)
```

In production systems, this loop also requires explicit limits:

```text
max_steps
max_tokens
max_cost
max_tool_calls
allowed_tools
authorized_resources
timeout
approval_required_for_high_impact_actions
```

Without such limits, “autonomy” can turn into infinite loops, runaway costs, uncontrolled side effects, or security violations.[^3]

---

## 2. ReAct: From Tool Use to Adaptive Execution

ReAct connected reasoning and acting in an iterative structure in the research literature.[^1] The original approach used explicit textual reasoning steps between actions and observations. For modern systems, however, the key point is less the disclosure of such reasoning text and more the underlying pattern:

```text
Goal
 |
Interpret state
 |
Select action
 |
Execute tool
 |
Observe result
 |
Update state
 |
continue or terminate
```

The difference from a hard-coded workflow engine is that not every branch has to be programmed in advance. Based on the current result, an agent can decide whether to inspect another file, rerun a test, or request additional information.

That is both the strength and the risk. A conventional pipeline

```text
A -> B -> C -> D
```

is easy to test. An agentic process can produce different paths depending on the model, context, and tool results:

```text
A -> B -> C
 \       |
  -> D -> E -> B -> ...
```

Evaluation of agentic systems must therefore examine not only the final answer but also the **trajectory**: Which tools were called? Which resources were read? Which side effects occurred? What did the run cost? Were policies violated?

---

## 3. Planning Beyond the Linear Loop

A linear ReAct pattern is sufficient for many tasks. It becomes harder when early decisions influence many later steps or when alternative solution paths must be compared.

### 3.1 Tree of Thoughts

In 2023, Yao et al. proposed **Tree of Thoughts (ToT)**, in which a model does not pursue only one next reasoning step but generates several candidates and evaluates them as a search tree.[^5]

```text
              state A
           /     |      \
         B1      B2      B3
        / \              |
      C1  C2             C3
```

The concept combines language models with classical search. Candidates can be expanded, scored, and discarded. Depending on the implementation, Breadth-First Search, Depth-First Search, or other strategies can be used.

ToT is not, however, universal proof that more search is always better. More candidates mean more model calls, higher latency, and higher cost. The evaluation function itself can also be wrong. A model that generates poor solutions is not automatically a reliable judge of its own solutions.

### 3.2 Reflexion

**Reflexion** explores a different problem: How can an agent learn from a failed attempt without changing its model weights? Shinn et al. stored verbal feedback about previous trajectories and reused that information in later attempts.[^6]

```text
Attempt
  |
Failure / Evaluation
  |
Reflection
  |
Memory
  |
New Attempt
```

The term “verbal reinforcement learning” should be understood as a description of the approach, not as classical reinforcement learning with gradient updates to the model. Learning occurs at the level of **context or episodic memory**.

### 3.3 LATS and Search-Based Agents

Language Agent Tree Search (LATS) combined ideas from search, reflection, and Monte-Carlo-Tree-Search-style methods.[^7] Such work is important because it shows that agentic reasoning can increasingly be treated as a **search problem over possible action sequences**.

It would nevertheless be unjustified to call LATS the “current gold standard” for agents. Practice is heterogeneous. Many successful coding agents use comparatively simple loops with strong tools, good repository instructions, tests, and clear stopping conditions. More complex search methods are especially interesting where the additional compute investment is justified by a higher probability of success.

---

## 4. Multi-Agent Systems: Division of Labor with Additional Trust Boundaries

A single agent does not have to solve every task itself. Multi-agent systems distribute work among several specialized roles.

```text
                 +------------------+
                 |   Orchestrator   |
                 +------------------+
                   /      |       \
                  v       v        v
             Research   Coding   Security
              Agent     Agent     Reviewer
```

Research and framework work such as AutoGen examined conversations among several LLM-based agents; *Generative Agents* demonstrated a different form of multiple autonomous instances in a simulated environment.[^8][^9]

The potential advantages are clear:

- **Context separation:** A reviewer may need only the diff, policies, and test results.
- **Specialization:** Different roles can receive different tools and instructions.
- **Parallelization:** Independent subtasks can be processed simultaneously.
- **Separation of duties:** An implementing agent can be checked through a separate verification path.

But multi-agent architectures do not automatically improve quality. They create new classes of failure:

```text
Agent A produces a false assumption
        |
        v
Agent B adopts it without verification
        |
        v
Agent C approves a derived change
```

Messages from other agents must therefore be treated as **inputs with provenance**, not as inherently trustworthy truth. OWASP addresses exactly such risks in its agentic threat-modeling resources.[^3][^10]

A robust system should therefore record:

```text
who produced this message?
which tool/data source supports it?
which permissions did the sender have?
was the artifact independently verified?
```

---

## 5. The Integration Problem: N Models × M Tools

As tool use expanded, a practical architecture problem emerged. Without a common standard, every host needs custom adapters for every external system.

```text
            GitHub  Slack  DB  Files  Browser
Claude        x      x     x    x       x
Codex         x      x     x    x       x
Gemini        x      x     x    x       x
Custom        x      x     x    x       x
```

This matrix creates duplicated integration code, inconsistent authentication models, and inconsistent metadata.

Anthropic released the **Model Context Protocol (MCP)** in late 2024 as an open protocol for standardized connections between AI applications, tools, and data sources.[^11] In December 2025, MCP was donated to the **Agentic AI Foundation (AAIF)** under the Linux Foundation. The foundation was co-founded by organizations including Anthropic, Block, and OpenAI; according to Anthropic and the Linux Foundation, MCP had already been adopted by a broad range of AI products at that time.[^12][^13]

MCP is therefore a widely adopted open integration standard. The phrase “universal industry standard” should still be used cautiously: broad adoption is not the same as formal standardization by ISO, the IETF, or a comparable standards body.

---

## 6. MCP Architecture: Distinguishing Host, Client, and Server

One of the most common simplifications is:

```text
MCP Client <-> MCP Server
```

The official architecture, however, distinguishes **host, client, and server**.[^14]

```text
+--------------------------------------------------+
| MCP Host                                         |
|  - User interaction                              |
|  - Policy / Consent                              |
|  - LLM / Agent Orchestration                     |
|                                                  |
|   +-------------+      +-------------+           |
|   | MCP Client A|      | MCP Client B|           |
|   +------+------+      +------+------+           |
+----------|---------------------|------------------+
           |                     |
           v                     v
     +-----------+         +-----------+
     | Server A  |         | Server B  |
     | Git / SCM |         | Database  |
     +-----------+         +-----------+
```

The **host** coordinates the application and manages security and user decisions. A **client** maintains one connection to one server. The **server** exposes concrete capabilities.[^14]

This distinction is more than terminology. It prevents the mistaken idea that an MCP server is directly “the model.” The model can be hosted or invoked by the host; MCP standardizes the exchange of capabilities and context among components.

---

## 7. JSON-RPC and Capability Negotiation

MCP uses **JSON-RPC 2.0** as its message format and adds a defined lifecycle and capability negotiation.[^15]

During connection establishment, client and server communicate which features they support. A server may, for example, declare:

```text
resources
prompts
tools
logging
```

while a client or host may support functions such as sampling or elicitation, depending on the protocol revision.[^15]

The advantage is evolution without hard coupling: a client does not need to assume that every server implements every MCP capability. Capabilities are negotiated explicitly.

---

## 8. Tools, Resources, and Prompts: Three Different Control Models

The current MCP specification describes three central server-side primitives.[^16]

### 8.1 Tools

Tools are executable functions. A server can expose and invoke them through:

```text
tools/list
tools/call
```

A tool has a name, a description, and an input schema. Examples include:

```text
create_issue
run_query
read_pipeline_status
restart_service
```

In the official control model, tools are considered **model-controlled**: the model can propose or initiate their use, while the host and user must still enforce security and consent boundaries.[^16]

### 8.2 Resources

Resources expose data or context:

```text
resources/list
resources/read
```

They are addressed through URIs and are considered **application-controlled**. Examples include files, schemas, or Git histories.[^17]

### 8.3 Prompts

Prompts are reusable, parameterized templates:

```text
prompts/list
prompts/get
```

Within the MCP interaction model, they are **user-controlled**: the user or UI typically decides when a prompt is used.[^18]

<Architecture>
**The three primitives are not interchangeable:** A Resource is not a harmless Tool, a Tool is not a Prompt, and a Prompt is not an authorization rule. The separation makes control and security models more explicit.
</Architecture>

---

## 9. MCP Transports: Why “SSE” Is Outdated as the Current Standard Description

The first generation of MCP used HTTP+SSE for remote connections. That description is now outdated.

The stable **2025-11-25** specification defines two standard transports:[^19]

1. **stdio**
2. **Streamable HTTP**

### 9.1 stdio

With `stdio`, the client starts the server as a local subprocess and exchanges JSON-RPC messages through standard input and standard output.[^19]

```text
Host/Client
   |
start subprocess
   |
stdin/stdout
   |
MCP Server
```

This is attractive for local tools because no network listener is required. It does not, however, automatically mean “maximum isolation”: the spawned process has exactly the operating-system privileges and environmental access granted by its runtime. Sandboxing must be implemented separately.

### 9.2 Streamable HTTP

**Streamable HTTP** replaced the old HTTP+SSE transport with protocol revision 2025-03-26.[^19] A server exposes an HTTP endpoint; POST is used to send JSON-RPC messages, while GET can support server-side streaming via SSE.

SSE has therefore not disappeared completely—but today it is a possible mechanism **within Streamable HTTP**, not the correct name for the standard transport itself.

---

## 10. MCP Security: A Protocol Does Not Replace an Authorization Model

Standardizing a tool interface does not automatically make its use secure. On the contrary, a universal tool channel can increase blast radius if identity, authorization, or consent are implemented incorrectly.

The MCP specification therefore states explicit security principles: users should understand and control data access and operations; tools must be treated carefully because they may execute code; and tool descriptions from untrusted servers must not be accepted blindly as truth.[^20]

For HTTP-based MCP connections, the current specification defines an OAuth-based authorization model and requires, among other things, Resource Indicators and audience validation. Token passthrough to downstream services is explicitly problematic because it can create a confused-deputy scenario.[^21]

A secure design therefore looks more like this:

```text
User Identity
     |
     v
MCP Host Policy
     |
     +--> consent / approval
     +--> tool allowlist
     +--> resource scope
     +--> audit
     |
MCP Client
     |
MCP Server
     |
separate downstream credential
     |
Target API
```

The central rule remains the same as with Function Calling: **the language model must not make the final decision about its own privileges.**

---

## 11. Agents Become Distributed Software Systems

With MCP, multi-agent communication, and repeated tool loops, the boundary between “prompt engineering” and classical software architecture disappears almost entirely. A production agent suddenly requires the same disciplines familiar from distributed systems:

- identity and authentication
- authorization and least privilege
- state management
- error handling
- timeouts and retries
- idempotency
- observability
- rate limits
- cost budgets
- audit trails
- supply-chain management

On top of these come LLM-specific problems such as prompt injection, non-deterministic decisions, context poisoning, and trajectories that are harder to reproduce.[^3]

This is precisely why Agentic AI does not move us away from software engineering, but **deeper into it**. The central question is no longer merely: “Which model should I use?” It becomes: “Which rules, instructions, skills, tests, and verification steps make the entire agentic system controllable?”

That question leads to the final part of the series: **[Part 5: Agentic Engineering in Practice – AGENTS.md, Skills, Flows, and the Future of Programming](/blog/05-agentic-engineering-agents-md-skills-und-flows)**.

<KeyTakeaway>
**Core message of Part 4:** Agentic AI emerges from a controlled execution loop around a model—not from a magical “autonomous” LLM. MCP standardizes important connections between hosts and external capabilities, but it replaces neither policy enforcement nor authentication, authorization, sandboxing, or human oversight.
</KeyTakeaway>

---

## Notes on the Source Situation

Agent research is evolving rapidly and has no single generally accepted “best algorithm.” Tree of Thoughts, Reflexion, and LATS are important research approaches, but their suitability depends on the problem and available compute budget. For MCP, this article follows the stable **2025-11-25** specification; at the time of revision, a draft with additional changes also exists. Statements about MCP are therefore explicitly tied to the named protocol revision.[^22]

---

## 📚 Literature & Scientific Sources

[^1]: Yao, S., et al. (2022/2023). *ReAct: Synergizing Reasoning and Acting in Language Models.* ICLR 2023. https://arxiv.org/abs/2210.03629
[^2]: Wang, L., et al. (2024). *A Survey on Large Language Model based Autonomous Agents.* Frontiers of Computer Science. https://arxiv.org/abs/2308.11432
[^3]: OWASP GenAI Security Project (2025). *Agentic AI – Threats and Mitigations.* https://genai.owasp.org/resource/agentic-ai-threats-and-mitigations/
[^4]: Rao, A. S., & Georgeff, M. P. (1995). *BDI Agents: From Theory to Practice.* Proceedings of ICMAS-95.
[^5]: Yao, S., Yu, D., Zhao, J., Shafran, I., Griffiths, T. L., Cao, Y., & Narasimhan, K. (2023). *Tree of Thoughts: Deliberate Problem Solving with Large Language Models.* NeurIPS 2023. https://arxiv.org/abs/2305.10601
[^6]: Shinn, N., Cassano, F., Gopinath, A., Narasimhan, K., & Yao, S. (2023). *Reflexion: Language Agents with Verbal Reinforcement Learning.* NeurIPS 2023. https://arxiv.org/abs/2303.11366
[^7]: Zhou, A., Yan, K., Shlapentokh-Rothman, M., Wang, H., & Wang, Y. (2023). *Language Agent Tree Search Unifies Reasoning, Acting, and Planning in Language Models.* https://arxiv.org/abs/2310.04406
[^8]: Wu, Q., et al. (2023). *AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation.* https://arxiv.org/abs/2308.08155
[^9]: Park, J. S., et al. (2023). *Generative Agents: Interactive Simulacra of Human Behavior.* UIST 2023. https://arxiv.org/abs/2304.03442
[^10]: OWASP GenAI Security Project (2025). *Multi-Agentic System Threat Modeling Guide v1.0.* https://genai.owasp.org/resource/multi-agentic-system-threat-modeling-guide-v1-0/
[^11]: Anthropic (2024). *Introducing the Model Context Protocol.* https://www.anthropic.com/news/model-context-protocol
[^12]: Anthropic (2025). *Donating the Model Context Protocol and establishing the Agentic AI Foundation.* https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation
[^13]: Linux Foundation (2025). *Linux Foundation Announces the Formation of the Agentic AI Foundation.* https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation
[^14]: Model Context Protocol. *Architecture.* https://modelcontextprotocol.io/specification/2025-11-25/architecture
[^15]: Model Context Protocol. *Lifecycle / Capability Negotiation.* https://modelcontextprotocol.io/specification/2025-11-25/basic
[^16]: Model Context Protocol. *Server Features Overview: Prompts, Resources, Tools.* https://modelcontextprotocol.io/specification/2025-11-25/server
[^17]: Model Context Protocol. *Resources.* https://modelcontextprotocol.io/specification/2025-11-25/server/resources
[^18]: Model Context Protocol. *Prompts.* https://modelcontextprotocol.io/specification/2025-11-25/server/prompts
[^19]: Model Context Protocol. *Transports – Protocol Revision 2025-11-25.* https://modelcontextprotocol.io/specification/2025-11-25/basic/transports
[^20]: Model Context Protocol. *Specification – Security and Trust & Safety.* https://modelcontextprotocol.io/specification/2025-11-25
[^21]: Model Context Protocol. *Authorization – Protocol Revision 2025-11-25.* https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization
[^22]: Model Context Protocol. *Key Changes / Draft Changelog.* https://modelcontextprotocol.io/specification/draft/changelog
