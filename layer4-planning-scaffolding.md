# Layer 4: Planning Scaffolding

## Purpose

Enable agent reasoning through structured planning.

Planning scaffolding transforms a single-step agent execution into a governed, multi-stage reasoning workflow. Without planning scaffolding, an agent receives a goal and attempts to execute it in one pass — producing outputs that are incomplete, unvalidated, or misaligned with complex enterprise requirements. With a mature planning layer, goals are decomposed into ordered tasks, assigned to specialist agents, executed in a controlled sequence, and validated before results are accepted.

---

## The Shift to Enterprise Planning

### Simple Pattern

```
Agent → Execute
```

A single agent receives a goal and produces an output directly. This works for simple, well-scoped tasks but fails for goals that require multiple steps, domain expertise, or quality verification.

### Enterprise Pattern

```
Goal
  ↓
Planner Agent
  ↓
Task Decomposition
  ↓
Specialist Agents
  ↓
Execution
  ↓
Validation
```

A planner agent interprets the goal, decomposes it into structured tasks, delegates each task to the most appropriate specialist, oversees execution, and verifies the final output before accepting results.

---

## Components

### 1. Planner Agent

The central coordinator that interprets the goal and produces a structured plan.

- Analyse the goal to identify scope, constraints, and success criteria
- Break the goal into an ordered sequence of tasks with explicit dependencies
- Select and route tasks to appropriate specialist agents
- Monitor plan progress and adapt the sequence when tasks fail or produce unexpected results

### 2. Task Decomposition

The process of converting a high-level goal into discrete, executable steps.

- Define each task with clear inputs, expected outputs, and acceptance criteria
- Identify dependencies between tasks to establish execution order
- Assign effort and risk estimates to each task for prioritization and scheduling
- Represent the task graph in a structured format that agents and orchestration systems can process

### 3. Specialist Agents

Purpose-built agents responsible for executing individual tasks within the plan.

- Each specialist is scoped to a specific domain, skill, or tool set
- Specialists receive task context, constraints, and input artifacts from the planner
- Multiple specialists can execute independent tasks in parallel to reduce total time
- Specialist outputs are returned to the planner for integration and downstream use

### 4. Execution

The controlled process of running each task in the defined order.

- Enforce task sequencing to ensure dependencies are satisfied before a task begins
- Pass outputs from completed tasks as inputs to dependent tasks
- Handle failures, retries, and fallback paths without losing plan state
- Log task execution details for traceability and post-execution audit

### 5. Validation

The quality gate that verifies task outputs before accepting them as complete.

- Check each task output against the acceptance criteria defined during decomposition
- Apply domain-specific validators for correctness, completeness, and compliance
- Return failed tasks to the appropriate specialist agent for correction
- Produce a final validation report summarizing what was delivered and how it was verified

---

## Why Planning Scaffolding Matters

| Without Planning Scaffolding | With Planning Scaffolding |
|---|---|
| Agent attempts complex goals in a single pass | Goals are decomposed into structured, manageable tasks |
| No coordination between multiple agents | Planner routes tasks to the right specialist at the right time |
| Partial or inconsistent outputs with no verification | Each task output is validated before being accepted |
| Failures are opaque and hard to recover from | Task-level logging enables targeted diagnosis and retry |
| Delivery quality depends on prompt alone | Quality is enforced through decomposition and validation stages |
| Not suitable for regulated or high-risk workflows | Governance checkpoints are embedded throughout the plan |

---

## Example: Modernization Agent

**Goal:** "Migrate COBOL application to Java"

This is a complex, multi-stage goal that cannot be achieved in a single agent call. A planner agent decomposes it as follows:

### Plan

| Step | Task | Specialist Agent | Output |
|---|---|---|---|
| 1 | Analyse application | Code Analysis Agent | Application inventory and complexity report |
| 2 | Identify dependencies | Dependency Mapping Agent | Dependency graph and integration catalogue |
| 3 | Generate target architecture | Architecture Agent | Java target architecture document |
| 4 | Convert modules | Code Conversion Agent | Converted Java source modules |
| 5 | Generate tests | Test Generation Agent | Unit and integration test suite |
| 6 | Validate output | Validation Agent | Quality report with pass/fail results |

### Execution Flow

```
Goal: Migrate COBOL application to Java
  ↓
Planner Agent analyses goal and produces 6-step task graph
  ↓
Step 1: Code Analysis Agent → Application inventory
  ↓
Step 2: Dependency Mapping Agent → Dependency graph
  ↓
Step 3: Architecture Agent → Java target architecture
  ↓
Step 4: Code Conversion Agent → Converted Java modules
  ↓
Step 5: Test Generation Agent → Test suite
  ↓
Step 6: Validation Agent → Quality report
  ↓
Planner reviews validation report and closes plan
```

Each step produces a validated artifact that feeds the next. If Step 4 produces modules that fail Step 6 validation, the planner returns the failing modules to the Code Conversion Agent with the validation feedback before proceeding.

---

## Implementation Guidance

1. **Define the planner's goal grammar** — Establish how goals are expressed so the planner can consistently parse scope, constraints, success criteria, and output requirements.
2. **Standardize task structure** — Every decomposed task should carry a name, description, inputs, expected outputs, acceptance criteria, and dependency list.
3. **Build specialist agents with narrow scope** — Focused specialists are easier to test, validate, and replace than generalist agents asked to do too much.
4. **Make the plan state inspectable** — Store plan state externally so that humans and systems can observe progress, intervene on failures, and resume from the last successful step.
5. **Design validation as a first-class step** — Do not treat validation as optional post-processing; embed it explicitly in the task graph with defined criteria and escalation paths.
6. **Handle partial failures gracefully** — Build retry logic, fallback paths, and human escalation into the planner so that one failed task does not abort the entire plan.
7. **Log at the task level** — Capture inputs, outputs, agent decisions, and timestamps for every task to support audit trails, debugging, and continuous improvement.
8. **Separate planning from execution** — Keep the planner responsible for coordination and let specialist agents own execution. This separation makes each component independently testable and replaceable.

---

## Relationship to the 8-Dimensional Scaffolding Framework

Planning scaffolding is the reasoning and coordination layer that activates complex multi-agent enterprise workflows across the broader framework:

- **Strategy and Use-Case Selection** — Planning scaffolding is most valuable for complex use cases where a single prompt or agent cannot deliver the required outcome. Use-case selection should identify which goals require a planner.
- **Platform and Architecture** — Planner agents, task graphs, and execution state management are reusable platform components that can be shared across products and domains.
- **Model Lifecycle and MLOps** — Task-level success and failure metrics provide detailed evaluation signals for each specialist agent, improving model selection and prompt tuning.
- **Security, Governance, and Compliance** — Validation steps embedded in the plan are natural control points for compliance checks, human review, and regulatory approvals.
- **Product Delivery and Change Management** — Decomposed plans make AI-assisted workflows explainable and auditable to product owners, reducing resistance to adoption.
- **Operations and Continuous Improvement** — Task-level logs and validation reports provide granular operational data for identifying bottlenecks, improving specialist agents, and reducing failure rates over time.

---

## Summary

Planning scaffolding moves enterprise AI from prompt-and-execute to reason-and-coordinate. By introducing a planner agent that decomposes goals, delegates tasks to specialists, enforces execution order, and validates outputs, organizations can apply AI to complex, multi-stage workflows with the same rigour they apply to any other enterprise delivery process. The result is AI behaviour that is more reliable, auditable, and capable of handling the full complexity of real enterprise goals.
