# Layer 6: Memory Scaffolding

## Purpose

Give AI systems the right memory at the right scope.

Memory scaffolding is the layer that enables AI agents to retain and recall information across turns, sessions, and users. Without it, every interaction starts from zero — the agent cannot remember the user's preferences, past decisions, or the organisation's standing policies. With a well-designed memory layer, agents provide continuity, personalisation, and institutional awareness while respecting the privacy and access boundaries that enterprise environments require.

---

## Architecture Overview

```
Agent
  ↓
Memory Manager
  (Read / write routing, scope resolution, TTL enforcement)
  ↓
-----------------------------------------
| Working Memory  | User Memory  | Enterprise Memory |
| (current turn)  | (per-user)   | (org-wide)        |
-----------------------------------------
  ↓
Vector DB / Knowledge Store
  (Semantic retrieval, structured lookup, access-controlled namespaces)
```

The Memory Manager sits between the agent and all memory stores. It decides which store to read from and write to based on the scope of the information, the identity of the user, and the lifetime of the data.

---

## Memory Types

### 1. Working Memory (Short-Term)

Holds the context of the current conversation session.

- Scoped to a single session or conversation thread
- Populated automatically from the conversation history as the session progresses
- Discarded or archived when the session ends
- Enables the agent to maintain coherence across multiple turns without re-reading the full history on every message

**Example:**
> *"Continue from the previous API design."*

The agent retrieves the API schema, technology choices, and constraints discussed earlier in the session, allowing it to continue without the user restating context.

**Key design decisions:**
- Define the working memory window — how many prior turns or tokens to retain in-context
- Apply summarisation for long sessions to compress older context without losing critical decisions
- Persist a session summary to long-term storage at the end of each conversation for future reference

---

### 2. User Memory (Long-Term)

Retains preferences, patterns, and personalisation signals for an individual user across sessions.

- Scoped to a single authenticated user
- Persists beyond session boundaries
- Updated incrementally as the user's preferences are stated or inferred
- Retrieved at the start of each session to personalise agent behaviour without requiring the user to repeat themselves

**Example:**
> *"Always generate Java 21 solutions."*

The agent stores this preference against the user's identity. In every future session, the Memory Manager injects the preference into the agent's context before code generation tasks begin.

**Typical user memory entries:**

| Category | Examples |
|---|---|
| Language and framework | "Prefer Java 21", "Use TypeScript strict mode" |
| Style preferences | "Concise explanations", "Include test cases by default" |
| Workflow preferences | "Always open a Jira ticket before code changes" |
| Communication style | "Respond in bullet points", "Skip boilerplate preamble" |
| Project context | "Primary project is the payments service on GitHub" |

**Key design decisions:**
- Require explicit user consent before capturing and persisting preferences
- Provide a user-accessible interface to review, edit, and delete stored preferences
- Apply staleness checks — preferences stated over 12 months ago may no longer be accurate
- Scope user memory to the individual; never share user-specific memory across users

---

### 3. Enterprise Memory (Long-Term, Org-Scoped)

Stores organisational knowledge, policies, and standards that apply across users and teams.

- Scoped to the organisation, department, or project depending on classification
- Maintained by designated knowledge owners rather than inferred from individual interactions
- Retrieved based on the task domain to enforce consistent organisational behaviour across all agents and users

**Example:**
> *"Banking systems require maker-checker approval."*

Every code generation, workflow automation, or architecture recommendation the agent produces for banking domains will include maker-checker controls, because the enterprise memory layer injects this policy into the agent's context automatically.

**Typical enterprise memory categories:**

| Category | Examples |
|---|---|
| Regulatory policies | PCI-DSS requirements, GDPR data handling rules |
| Architecture standards | Approved technology stack, banned libraries |
| Approval workflows | Maker-checker for financial transactions, change advisory board for production releases |
| Security baselines | Password policies, encryption standards, network segmentation rules |
| Domain terminology | Internal product names, team structures, process definitions |
| Runbooks | Incident response procedures, on-call escalation paths |

**Key design decisions:**
- Apply role-based access to enterprise memory — not all organisational knowledge is visible to every user
- Version enterprise memory entries so agents can be audited against the policy state that was active at decision time
- Define ownership and review cadences for each entry; unreviewed enterprise memory degrades into misinformation

---

## Memory Manager

The Memory Manager is the orchestration component that controls all memory read and write operations.

### Responsibilities

1. **Scope resolution** — Determine whether a piece of information belongs in working, user, or enterprise memory based on its content, source, and lifetime.
2. **Read routing** — On each agent turn, retrieve relevant context from the appropriate stores and assemble it into the agent's working context.
3. **Write routing** — Identify new information in agent outputs or user messages that should be persisted, and route writes to the correct store with appropriate metadata.
4. **TTL enforcement** — Apply time-to-live rules to memory entries; archive or expire entries that have exceeded their useful lifetime.
5. **Access control** — Enforce that each retrieval respects the user's identity and permission scope; prevent cross-user or cross-tenant memory leakage.
6. **Conflict resolution** — When working memory, user memory, and enterprise memory provide conflicting information, apply a defined precedence order (typically: enterprise policy > user preference > session context).

### Precedence Model

```
Enterprise Memory (highest authority — policy cannot be overridden by user preference)
  ↓
User Memory (personal preference within enterprise policy bounds)
  ↓
Working Memory (session-specific context, most recent and most specific)
```

---

## Vector DB / Knowledge Store

All three memory tiers are backed by a retrieval layer that supports both semantic search and structured lookup.

- **Semantic search** — Retrieve memory entries by meaning, not just keyword match, so the agent can surface relevant context even when the user's phrasing varies across sessions.
- **Structured lookup** — Retrieve memory entries by exact identifiers (user ID, policy ID, domain tag) for deterministic access control and permission enforcement.
- **Namespace isolation** — Separate namespaces for working, user, and enterprise memory prevent cross-scope contamination and enable targeted access controls.
- **Metadata filtering** — Filter retrievals by scope, owner, domain, sensitivity classification, and freshness to return the most relevant and authorised context.

---

## Why Memory Scaffolding Matters

| Without Memory Scaffolding | With Memory Scaffolding |
|---|---|
| Every session starts from scratch | Agent recalls user preferences and prior decisions |
| Users repeat preferences on every interaction | Personalisation is automatic and persistent |
| Organisational policies must be stated manually in each prompt | Enterprise standards are injected automatically |
| No continuity across long or multi-session tasks | Agent maintains state across conversation threads |
| Policy changes require prompt rewrites across all use cases | Updating enterprise memory propagates automatically |
| All users receive identical, generic responses | Responses are tailored to user context and role |

---

## Example: IT Support Copilot — Memory Layer in Action

**Scenario:** An employee returns to the IT support copilot two weeks after an initial session about network access issues.

### Session 1 (two weeks prior)

- The employee stated: *"I always work from the Singapore office on Wednesdays."*
- The copilot stored this as a **user memory** entry: `work_location: Singapore, day: Wednesday`.
- The session resolved a VPN issue. A session summary was archived to **working memory** storage.

### Session 2 (current)

1. **Memory Manager** retrieves the archived session summary (working memory) and the `work_location` preference (user memory) at session start.

2. **Enterprise Memory** contributes: `Singapore office uses a split-tunnel VPN profile; all traffic to internal banking APIs must route through the corporate proxy`.

3. The employee asks: *"My network access is slow again today."*

4. The agent, with all three memory layers active, responds:
   - Identifies that it is Wednesday and the user is likely in Singapore (user memory)
   - Recalls the previous VPN resolution (working memory summary)
   - Applies the Singapore split-tunnel policy (enterprise memory)
   - Suggests checking the corporate proxy configuration — the specific fix relevant to this user's context — without requiring the employee to re-explain their setup

### Result

A resolution in one turn rather than five, with no repeated context-gathering, because memory scaffolding assembled the relevant history and policy context automatically.

---

## Implementation Guidance

1. **Design memory scope boundaries first** — Determine the lifetime, ownership, and access rules for each memory tier before implementation. Retrofitting scope boundaries after go-live is costly.
2. **Obtain explicit consent for user memory** — Inform users when preferences are being captured and stored. Provide a self-service interface to review and delete entries.
3. **Separate enterprise memory from knowledge scaffolding** — Enterprise memory covers standing policies and standards that direct agent behaviour. Knowledge scaffolding covers factual document retrieval for answering questions. Both are needed; they serve different purposes.
4. **Apply a clear precedence model** — Define and document which memory tier takes precedence when entries conflict. Ambiguity in precedence leads to unpredictable agent behaviour.
5. **Version enterprise memory entries** — Treat policy entries as versioned records. Agents must be auditable against the policy state active at the time of their decisions.
6. **Instrument memory hit rates** — Track how often each memory tier contributes to agent responses. Low hit rates indicate poor relevance; high rates with poor user satisfaction indicate stale or inaccurate entries.
7. **Apply TTL to all entries** — Working memory expires at session end. User memory should have staleness checks at defined intervals. Enterprise memory entries must have scheduled review dates aligned to policy governance cycles.
8. **Test memory isolation rigorously** — Verify that no user memory is retrievable by a different user, and that enterprise memory respects department- and role-based access controls. Memory leakage is a privacy and compliance failure.

---

## Relationship to the 8-Dimensional Scaffolding Framework

Memory scaffolding provides the continuity and institutional knowledge layer that underpins consistent, compliant agent behaviour across the full framework:

- **Strategy and Use-Case Selection** — Enterprise memory encodes the organisational goals and constraints that define what agents are permitted to optimise for.
- **Data Foundation** — User and enterprise memory entries are data assets that require the same quality, lineage, and access control disciplines as other enterprise data.
- **Platform and Architecture** — The Memory Manager and its backing stores are shared platform services; all agents in the enterprise consume them rather than building isolated memory implementations.
- **Security, Governance, and Compliance** — Role-scoped retrieval, consent management, versioned policy entries, and memory isolation are the primary mechanisms through which the memory layer enforces governance requirements.
- **Model Lifecycle and MLOps** — Memory content influences agent outputs. Memory updates must be treated as configuration changes and validated in evaluation pipelines before production rollout.
- **Product Delivery and Change Management** — Product teams benefit from the memory layer without building it themselves; personalisation and policy enforcement are infrastructure concerns, not per-agent development tasks.
- **Operations and Continuous Improvement** — Memory hit rates, staleness metrics, and user feedback on personalisations are operational signals that drive continuous improvement of both the memory content and the retrieval strategy.
- **Value Realization and Portfolio Scaling** — Each new enterprise memory entry and each new user preference captured adds value to every agent that consumes the shared memory layer, compounding returns across the portfolio.

---

## Summary

Memory scaffolding gives AI agents the ability to remember — across turns, across sessions, and across the enterprise. By structuring memory into three distinct tiers — working memory for session continuity, user memory for personalisation, and enterprise memory for organisational policy — and by routing every read and write through a governed Memory Manager, enterprises can deploy agents that behave consistently, adapt to individual users, and enforce institutional standards automatically. The result is an agent experience that improves over time rather than resetting on every interaction.
