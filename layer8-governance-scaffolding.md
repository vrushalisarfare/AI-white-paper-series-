# Layer 8: Governance Scaffolding

## Purpose

Operate the control plane for enterprise AI.

Governance scaffolding is the authoritative control layer that determines who an agent is, what it can access, what it is permitted to do, what it has done, and how much it has consumed. Without it, AI agents operate in a trust vacuum — any agent can claim any identity, access any resource, invoke any tool, and leave no record of what happened or what it cost. With a structured governance layer, every agent action flows through an auditable, policy-enforced pipeline that aligns AI behaviour with enterprise risk appetite, regulatory requirements, and operational budgets.

---

## Architecture Overview

```
Agent Identity
      ↓
Policy Engine
      ↓
Tool Authorization
      ↓
Audit Trail
      ↓
Monitoring
```

Each stage is a mandatory gate. Identity establishes the agent's verifiable persona before any policy can be evaluated. The policy engine determines what the authenticated agent is permitted to do. Tool authorization enforces those permissions at the point of execution. The audit trail records every decision and action. Monitoring aggregates signals into operational and cost dashboards that feed back into policy refinement.

---

## Components

### 1. AI Identity

**Who is the agent?**

AI identity establishes a verifiable, persistent, and revocable persona for every agent or copilot in the enterprise. Just as human employees have corporate identities with associated roles and permissions, AI agents must be first-class identity principals in the enterprise directory.

#### 1a. Agent Identity Records

Every deployed agent must have a registered identity containing:

| Attribute | Description |
|---|---|
| `agent_id` | Globally unique identifier (e.g., UUID or URN) |
| `agent_name` | Human-readable display name |
| `agent_type` | Classification: copilot, orchestrator, worker, evaluator |
| `owner_team` | The team responsible for the agent's behaviour |
| `created_at` | Provisioning timestamp |
| `environment` | dev / stage / prod |
| `model_binding` | The LLM or model version the agent is authorised to call |
| `trust_level` | Low / Medium / High — governs permitted action scope |

#### 1b. Authentication Mechanisms

Agents must authenticate to all downstream services using managed, non-human credentials:

- **Managed identities** — Platform-managed service principals (e.g., Azure Managed Identity, AWS IAM roles for services) that require no stored secrets.
- **Short-lived tokens** — OIDC or JWT tokens with bounded expiry; never long-lived static API keys.
- **Mutual TLS** — For agent-to-agent or agent-to-service communication in zero-trust network architectures.
- **Federated identity** — Link agent identity to a CI/CD system or orchestration platform to trace agent actions to the pipeline that spawned them.

#### 1c. Identity Lifecycle

| Lifecycle Stage | Requirement |
|---|---|
| Provisioning | Identity created through automated pipeline, not manually |
| Rotation | Credentials rotated on a defined schedule or on-demand after incidents |
| Suspension | Agent identity can be instantly disabled without code changes |
| Decommissioning | Identity revoked, audit records retained per data retention policy |

---

### 2. AI Authorization

**What can the agent access?**

Authorization translates a verified identity into a precise, least-privilege access grant. AI agents must not receive broad permissions by default; every access right must be justified, scoped, and time-bounded.

#### 2a. Role-Based Access Control (RBAC)

Define agent roles that mirror their operational function:

| Agent Role | Permitted Resources | Denied Resources |
|---|---|---|
| `read-only-analyst` | Knowledge base, metrics dashboards | Write APIs, user PII, financial records |
| `it-support-copilot` | ServiceNow tickets, IT runbooks, Active Directory (read) | Payroll systems, source control write |
| `code-generation-agent` | Repository read, CI pipeline trigger | Production deployment, secrets vault |
| `finance-forecasting-agent` | Financial data warehouse (read), report templates | General ledger write, HR records |

#### 2b. Attribute-Based Access Control (ABAC)

For fine-grained scenarios where role alone is insufficient, layer in attribute-based policies:

- Grant access only when `agent.environment == "prod"` and `request.data_classification <= "internal"`.
- Restrict access to customer PII to agents with `agent.trust_level == "High"` and an active data processing agreement on record.
- Allow tool invocation only when the originating request carries a human-in-the-loop approval token.

#### 2c. Scoped API and Tool Permissions

Each tool or API available to an agent must have explicitly bounded permissions:

- **Read vs. write scopes** — an agent that reads incident tickets should never be granted the scope to close or reassign them unless that is its defined function.
- **Row-level and column-level filtering** — database access policies should restrict agents to the rows and columns relevant to their task, not entire tables.
- **Quota-limited access** — cap the number of API calls per time window to prevent runaway loops or accidental data scraping.

---

### 3. AI Policies

**What can the agent do?**

Authorization answers *can this agent access this resource*; policy answers *can this agent take this action in this context*. Policies encode business rules, ethical guardrails, regulatory requirements, and operational constraints as executable, version-controlled artefacts.

#### 3a. Policy Categories

| Policy Type | Examples |
|---|---|
| **Safety** | Block generation of harmful, discriminatory, or misleading content |
| **Compliance** | Enforce GDPR data minimisation; prohibit retention of PII in logs |
| **Business rules** | Only escalate to a human when confidence score < 0.7 |
| **Operational** | Never call external APIs in the dev environment |
| **Cost** | Reject requests that would exceed the monthly LLM spend budget |
| **Security** | Block prompt injection patterns; sanitise all user-supplied input |

#### 3b. Policy Engine Architecture

```
Incoming Agent Action Request
      ↓
Input Normalisation
      ↓
Policy Evaluation Engine
  ┌───────────────────────────┐
  │ Safety Policies           │
  │ Compliance Policies       │
  │ Business Rule Policies    │
  │ Operational Policies      │
  │ Cost Policies             │
  └───────────────────────────┘
      ↓
Decision: Allow / Deny / Allow-with-conditions
      ↓
Action Execution (if allowed)
      ↓
Audit Log Entry
```

#### 3c. Policy Management Practices

- **Version control** — Policies are code. Store them in source control with peer review, change history, and rollback capability.
- **Policy testing** — Maintain a test suite of known-good and known-bad inputs to validate policy behaviour before deployment.
- **Policy simulation** — Run proposed policy changes against historical action logs to assess impact before going live.
- **Separation of duties** — Policy authoring, review, and deployment should involve different individuals or teams.
- **Policy expiry** — Temporary policies (e.g., a rate limit lifted during a planned event) must have defined expiry timestamps.

#### 3d. Open Policy Agent (OPA) Integration

OPA is the industry standard for decoupled, language-agnostic policy enforcement. In an AI governance context:

- Write policies in Rego, OPA's declarative language, to express complex multi-attribute rules.
- Deploy OPA as a sidecar or centralised service that agents query before executing any sensitive action.
- Return structured policy decisions that include the reason for denial — enabling meaningful error messages and audit records.

---

### 4. AI Observability

**What happened?**

Observability is the real-time and retrospective view of agent behaviour. It encompasses the audit trail of individual actions, the aggregated telemetry needed to detect anomalies, and the analytical capability to investigate incidents.

#### 4a. Audit Trail

Every agent action must produce an immutable, structured audit record:

| Audit Field | Description |
|---|---|
| `event_id` | Unique identifier for the audit event |
| `timestamp` | ISO 8601 timestamp of the action |
| `agent_id` | Identity of the acting agent |
| `session_id` | Conversation or workflow session |
| `user_id` | Human user who initiated the request (if applicable) |
| `action_type` | Read / Write / Tool call / LLM call / Escalation |
| `resource` | The specific resource or tool accessed |
| `input_summary` | Sanitised summary of the input (no PII) |
| `output_summary` | Sanitised summary of the output |
| `policy_decision` | Allow / Deny / Allow-with-conditions |
| `policy_id` | Reference to the policy that governed the decision |
| `latency_ms` | End-to-end latency of the action |
| `model_version` | LLM or model version called |
| `environment` | dev / stage / prod |

**Audit trail requirements:**
- Records must be append-only and tamper-evident (e.g., written to an immutable log store or signed with a cryptographic hash chain).
- Retention period must meet the most stringent applicable regulatory requirement (minimum 90 days recommended for operational use; 7 years for financial and regulated industries).
- Access to raw audit logs must itself be access-controlled and audited.

#### 4b. Telemetry and Metrics

Beyond individual event records, aggregate telemetry enables pattern detection and operational insight:

| Metric | What It Signals |
|---|---|
| Requests per agent per minute | Usage trends; runaway loops |
| Policy denial rate | Configuration gaps; attack attempts |
| Tool error rate | Integration failures; input quality issues |
| Hallucination detection rate | Model quality degradation; retrieval failures |
| Human escalation rate | Confidence calibration; task difficulty distribution |
| P95 / P99 latency | Performance degradation; upstream dependency issues |
| Model token consumption | Cost exposure; efficiency opportunities |

#### 4c. Anomaly Detection

- Define baseline behaviour profiles per agent type and alert on statistically significant deviations.
- Flag agents that suddenly increase their policy denial rate — this can indicate a prompt injection attempt or a misconfigured update.
- Alert when an agent accesses a resource it has not accessed in its prior history, even if the access is technically permitted.
- Correlate anomaly signals across agents to detect coordinated or supply-chain threats.

#### 4d. Incident Investigation

- Provide a replay capability: given an `event_id` or `session_id`, reconstruct the full sequence of agent actions, inputs, outputs, and policy decisions.
- Enable cross-layer querying: join audit trail entries with model call logs, tool execution logs, and cost records to build a complete picture of any incident.
- Maintain chain-of-custody documentation for any audit records involved in a regulatory or legal investigation.

---

### 5. AI Cost Governance

**How much did the agent consume?**

AI cost governance makes consumption visible, attributable, and controllable. Without it, LLM costs grow silently and unpredictably, with no mechanism to allocate them to business outcomes or intervene before budgets are exhausted.

#### 5a. Cost Attribution Model

Every unit of AI consumption must be attributed to a cost centre:

| Attribution Dimension | Example Values |
|---|---|
| `agent_id` | it-support-copilot-prod |
| `team` | platform-engineering, hr-operations |
| `business_unit` | EMEA IT, Global Finance |
| `use_case` | ticket-triage, code-review, report-generation |
| `model` | gpt-4o, claude-3-opus, llama-3-70b |
| `environment` | dev, stage, prod |
| `request_type` | Interactive, batch, scheduled |

#### 5b. Cost Metrics

Track consumption at multiple granularities:

| Metric | Unit | Purpose |
|---|---|---|
| Input tokens | tokens | Primary cost driver for most LLM APIs |
| Output tokens | tokens | Often priced differently from input tokens |
| Embedding calls | calls / tokens | Cost of knowledge retrieval |
| Tool execution calls | calls | Cost of external API and database access |
| Compute time | seconds | For self-hosted or fine-tuned models |
| Storage | GB-months | Vector store, audit log, and model artefact storage |

#### 5c. Budget Controls

- **Soft limits** — Alert the owning team when consumption reaches 80% of the monthly budget.
- **Hard limits** — Automatically block new agent requests when the budget ceiling is reached, with a defined fallback (queue, degrade gracefully, escalate to human).
- **Per-agent caps** — Set individual consumption ceilings so a single misconfigured or runaway agent cannot exhaust the shared budget.
- **Context window optimisation** — Enforce maximum prompt lengths and output token caps per request type to prevent unnecessarily expensive calls.

#### 5d. Cost Visibility and Chargeback

- Publish a real-time cost dashboard showing consumption by agent, team, use case, and model.
- Generate monthly chargeback reports attributing AI costs to business units for budget reconciliation.
- Surface cost-per-outcome metrics: cost per ticket resolved, cost per PR reviewed, cost per report generated — linking spend to business value.
- Benchmark costs against industry baselines and highlight efficiency outliers for optimisation.

#### 5e. Cost Optimisation Levers

| Lever | Description | Typical Saving |
|---|---|---|
| Model right-sizing | Route simple tasks to smaller, cheaper models | 40–70% |
| Caching | Cache identical or semantically similar responses | 20–50% |
| Prompt compression | Reduce context window usage through summarisation | 15–30% |
| Batching | Aggregate low-urgency requests for batch processing | 10–25% |
| Retrieval filtering | Return fewer, higher-precision chunks to the model | 10–20% |
| Output length limits | Cap maximum token output per request type | 10–15% |

---

## Why Governance Scaffolding Matters

| Without Governance Scaffolding | With Governance Scaffolding |
|---|---|
| Agents operate with unchecked, broad permissions | Every agent action is authorised against least-privilege policies |
| No audit trail — incidents are unrecoverable | Full audit replay capability for every agent session |
| Compliance evidence requires manual reconstruction | Automated, tamper-evident records available on demand |
| AI costs are invisible until the invoice arrives | Real-time cost attribution with budget controls and alerts |
| Rogue or compromised agents have unlimited blast radius | Identity revocation and policy enforcement contain damage instantly |
| Policy changes require code deployments | Policies are independently versioned and deployable |

---

## Example: IT Support Copilot — Governance Layer in Action

**Scenario:** The enterprise IT support copilot assists employees with password resets, VPN access, and device provisioning.

### Agent Identity

The copilot is registered as `it-support-copilot-prod` in the enterprise identity directory with `trust_level: Medium` and `owner_team: platform-engineering`. It authenticates to all downstream services using a platform-managed identity — no stored credentials in code or configuration.

### Authorization

The copilot holds the `it-support-read-write` RBAC role, granting it:
- Read access to ServiceNow tickets and the IT knowledge base.
- Write access to create and update tickets on behalf of the requesting user.
- Read access to Active Directory to verify user identity and device registration status.

It is explicitly denied access to payroll systems, source code repositories, and any data classified above `internal`.

### Policies

Active policies for the copilot include:
- **Safety:** Block any response that includes credentials, even partially masked.
- **Compliance:** Do not log or surface user email addresses in output summaries.
- **Business rule:** Escalate to a human agent if the request involves privileged access (domain admin, firewall changes) or if the confidence score falls below 0.75.
- **Cost:** Reject requests that would exceed 4,000 output tokens; use GPT-4o Mini for initial triage and GPT-4o only for complex multi-step resolutions.

### Audit Trail

Every interaction generates an audit record capturing the agent identity, session ID, user ID, action type, resources accessed, policy decisions applied, and token consumption. Records are written to an immutable log store with a 3-year retention period to satisfy the organisation's IT change management policy.

### Cost Governance

The IT support copilot is allocated a monthly budget of $2,000. A real-time dashboard shows daily spend by request type. At 80% utilisation, the platform-engineering team receives an alert. If the ceiling is reached, the copilot degrades to a static FAQ responder until the next budget cycle begins, ensuring the service remains available at reduced capability rather than failing hard.

---

## Implementation Guidance

1. **Register all agents in a central identity registry** — treat agent provisioning with the same rigour as human onboarding: approvals, access reviews, and offboarding procedures.
2. **Adopt least-privilege as the default** — start with the minimum access set and expand based on demonstrated need, not anticipated requirement.
3. **Treat policies as code** — version-control every policy, enforce peer review before deployment, and maintain a test suite that covers both positive (allowed) and negative (blocked) cases.
4. **Make cost visible before it becomes a problem** — instrument token consumption from day one; retrofitting cost attribution into a production system is significantly harder than building it in from the start.
5. **Design for revocability** — ensure every agent identity, credential, and permission can be revoked within minutes. Practise revocation drills as part of incident response preparation.
6. **Separate governance from agent logic** — the policy engine, audit trail, and cost controls should be platform infrastructure that agents consume as a service, not logic embedded in individual agent codebases.
7. **Conduct regular governance reviews** — quarterly access reviews should include AI agents alongside human users; retire agents that no longer serve active use cases.

---

## Relationship to the 8-Dimensional Scaffolding Framework

Governance scaffolding is the operational expression of the **Security, Governance, and Compliance** dimension of the 8-Dimensional Enterprise AI Scaffolding Framework:

- **Strategy and Use-Case Selection** — Governance defines the risk appetite that constrains which use cases are permitted and under what conditions.
- **Data Foundation** — AI authorization policies enforce the data access controls and privacy requirements established in the data foundation layer.
- **Platform and Architecture** — The policy engine, identity directory, audit store, and cost dashboard are core reusable platform components shared across all agents.
- **Model Lifecycle and MLOps** — Model version bindings in agent identity records link governance controls to specific model versions, enabling safe rollout and rollback.
- **Security, Governance, and Compliance** — Governance scaffolding provides the runtime enforcement mechanism for the guardrails defined in this dimension, including audit evidence for compliance reviews.
- **Operations and Continuous Improvement** — Observability metrics and cost attribution data are primary operational signals; anomaly alerts and budget overruns trigger operational escalations.
- **Value Realization and Portfolio Scaling** — Cost-per-outcome metrics directly inform portfolio-level investment and scaling decisions.

---

## Summary

Governance scaffolding is the control plane that makes enterprise AI trustworthy at scale. Identity ensures every agent is accountable. Authorization ensures every agent is constrained. Policies ensure every agent action is evaluated against business and regulatory rules. Observability ensures every agent action is recorded and reviewable. Cost governance ensures every agent action is paid for intentionally and attributed correctly.

Without governance scaffolding, AI adoption creates an expanding surface of uncontrolled, unaccountable, and increasingly expensive autonomous behaviour. With it, enterprises can extend agents confidently — knowing that each new deployment inherits the same identity, policy, audit, and cost controls that govern the rest of the portfolio.
