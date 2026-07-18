# Layer 7: Validation Scaffolding

## Purpose

Ensure AI output is correct, safe, and enterprise-ready before it reaches production.

Validation scaffolding is the quality gate that sits between AI-generated output and real-world use. Without it, organisations are left relying on human spot-checks and hoping that generated code compiles, generated content is accurate, and generated decisions are compliant. With a structured validation layer, every AI output passes through a deterministic, auditable pipeline that enforces technical correctness, business compliance, and factual accuracy before any human ever sees it — and certainly before it enters production.

---

## Architecture Overview

```
AI-Generated Output
        ↓
Validation Agent
  ┌─────────────────────────────────────┐
  │ Technical Validation                │
  │   Compilation · Unit Tests          │
  │   Security Scans · Performance      │
  ├─────────────────────────────────────┤
  │ Business Validation                 │
  │   Requirements · Policy             │
  │   Regulatory Compliance             │
  ├─────────────────────────────────────┤
  │ AI Validation                       │
  │   Hallucination Detection           │
  │   Confidence Scoring · Fact Check   │
  └─────────────────────────────────────┘
        ↓
Quality Score
        ↓
Human Approval (for scores below threshold or high-risk outputs)
        ↓
Production
```

The Validation Agent orchestrates all three validation dimensions. Outputs that pass above a configured quality threshold can proceed to production automatically. Outputs that fall below the threshold, or that touch high-risk domains, are routed to a human approver with a full validation report.

---

## Validation Dimensions

### 1. Technical Validation

Technical validation confirms that generated artefacts are functionally correct and safe to deploy.

#### 1a. Compilation

- Verify that generated code compiles successfully against the target runtime and version.
- Validate syntax, imports, and dependency declarations before any execution.
- Run in an isolated sandbox environment to prevent side effects from partial or malformed output.
- Report compilation errors with line-level diagnostics to enable targeted regeneration or human correction.

#### 1b. Unit Tests

- Execute an existing or AI-generated test suite against the generated code.
- Measure coverage thresholds and fail validation if critical paths are untested.
- Detect regressions by running the full existing test suite, not only tests written for the new output.
- Surface test failures with stack traces and assertion details for diagnosis.

#### 1c. Security Scans

- Run static application security testing (SAST) tools to identify vulnerabilities in generated code (e.g., injection flaws, insecure deserialization, hardcoded secrets).
- Apply dependency scanning to flag known CVEs in any libraries introduced by the generated output.
- Check for violations of the organisation's approved technology list — banned libraries, deprecated packages, or unapproved third-party services.
- Block promotion of output that introduces critical or high-severity security findings.

| Scan Type | Examples | What It Catches |
|---|---|---|
| SAST | Semgrep, SonarQube, CodeQL | Injection, auth flaws, insecure patterns |
| Dependency scan | OWASP Dependency-Check, Snyk | Known CVEs in third-party libraries |
| Secret detection | Trufflehog, GitLeaks | API keys, credentials in code or configs |
| Licence compliance | FOSSA, Black Duck | Incompatible open-source licences |

#### 1d. Performance Checks

- Profile the generated code against performance benchmarks defined for the target service.
- Detect obvious anti-patterns: N+1 database queries, unbounded loops, synchronous blocking in async contexts, missing pagination.
- Run load or stress tests if the output includes API endpoints or data-processing pipelines.
- Flag outputs that exceed latency or resource thresholds for human review before production deployment.

---

### 2. Business Validation

Business validation confirms that the generated output meets organisational requirements and stays within policy and regulatory boundaries.

#### 2a. Requirements Compliance

- Parse the original task or user story and verify that the generated output addresses all stated acceptance criteria.
- Use structured traceability: map each requirement to the code, configuration, or content that fulfils it.
- Detect omissions — requirements that have no corresponding output — and fail validation until they are addressed or explicitly deferred.
- Produce a requirements traceability matrix as part of the validation report for audit purposes.

#### 2b. Policy Checks

- Validate generated output against the organisation's internal policies: coding standards, architectural patterns, naming conventions, API design guidelines.
- Enforce the approved technology stack and flag deviations (e.g., use of an unapproved cloud provider SDK, introduction of a non-standard data store).
- Check that access control patterns, logging standards, and error-handling conventions match the enterprise baseline.
- Apply team-level policies (e.g., branch naming rules, commit message formats, infrastructure tagging requirements) when the output includes configuration or infrastructure-as-code.

#### 2c. Regulatory Checks

- Evaluate generated output for compliance with applicable regulations and standards: GDPR, HIPAA, PCI-DSS, SOX, FCA, and others relevant to the organisation's industry.
- Detect the presence of personal data (PII) in generated code, configuration, or content and verify that it is handled in accordance with data protection requirements.
- Check that consent mechanisms, data retention controls, audit logging, and encryption requirements are present where mandated.
- Flag regulatory concerns with specific citation to the relevant rule or standard so that human reviewers can make informed approval decisions.

| Regulatory Domain | Key Checks |
|---|---|
| GDPR | PII handling, consent, right-to-erasure support, data minimisation |
| PCI-DSS | Cardholder data encryption, access controls, audit logging |
| HIPAA | PHI handling, minimum necessary access, transmission security |
| SOX | Financial data integrity, change management audit trail |
| FCA / MiFID II | Transaction reporting, data retention, suitability assessments |

---

### 3. AI Validation

AI validation addresses the unique risks that arise from generative model outputs: hallucinations, miscalibrated confidence, and factual inaccuracies.

#### 3a. Hallucination Detection

- Compare generated factual claims against authoritative enterprise knowledge sources (the knowledge scaffolding layer).
- Use a secondary AI model or rule-based checker to identify statements that are not grounded in retrieved context or verified facts.
- Classify detected hallucinations by severity: factual errors, invented citations, fabricated API names, or plausible-sounding but incorrect logic.
- Require human review for any output containing unresolved hallucinations before it is used or published.

**Hallucination risk signals:**
- Claims about external libraries, APIs, or standards that cannot be verified against the knowledge base
- Numeric values (dates, thresholds, limits) that appear in the output but were not present in the source context
- Named entities (people, systems, services) referenced with specificity that was not provided in the prompt
- Code that calls functions or methods that do not exist in the target codebase

#### 3b. Confidence Scoring

- Assign a composite confidence score to each generated output based on signals from the generation process and validation checks.
- Incorporate model-reported token probabilities, retrieval relevance scores, and validation pass/fail rates into the composite score.
- Define tiered thresholds that determine the routing of the output:

| Confidence Tier | Score Range | Action |
|---|---|---|
| **High** | 90–100 | Auto-approve and proceed to production |
| **Medium** | 70–89 | Proceed with lightweight human spot-check |
| **Low** | 50–69 | Full human review required before promotion |
| **Rejected** | < 50 | Return to generation with validation report |

- Surface confidence scores transparently in the human approval interface so that reviewers can calibrate their level of scrutiny.
- Track confidence score calibration over time — measure whether high-confidence outputs actually perform better in production and adjust thresholds accordingly.

#### 3c. Fact Verification

- Cross-reference factual claims in generated output against authoritative sources: enterprise knowledge bases, official documentation, verified APIs.
- Apply citation checking for generated content that references external standards, regulations, or research: verify that the cited source exists and that the cited claim is accurately represented.
- Use structured fact extraction to isolate verifiable claims (dates, figures, named entities, technical specifications) and check each one independently.
- Produce a fact verification report that distinguishes between verified facts, unverifiable claims, and detected inaccuracies.

---

## Quality Score

The validation pipeline produces a composite Quality Score that aggregates results across all three validation dimensions.

```
Quality Score = f(
  Technical Score    (compilation, tests, security, performance),
  Business Score     (requirements, policy, regulatory),
  AI Score           (hallucination rate, confidence, fact accuracy)
)
```

### Scoring Model

| Dimension | Weight | Key Metrics |
|---|---|---|
| Technical | 40% | Test pass rate, security finding severity, compilation success |
| Business | 35% | Requirements coverage, policy violations, regulatory flags |
| AI | 25% | Hallucination count, confidence score, fact verification rate |

Weights should be adjusted to match the risk profile of the use case. Code generation for financial services may weight regulatory checks more heavily; documentation generation may weight AI validation more heavily.

### Quality Gates

Define explicit quality gates that an output must pass before proceeding to the next stage:

- **Critical gate** — Any critical security finding, confirmed hallucination, or regulatory violation automatically fails the output regardless of the composite score.
- **Threshold gate** — The composite Quality Score must exceed the configured threshold for the use case and risk tier.
- **Human gate** — Outputs above the threshold in regulated domains (financial, medical, legal) still require human sign-off before production deployment.

---

## Human Approval

Human approval is not a fallback for failed automation — it is a designed checkpoint for outputs that require human judgment, accountability, or regulatory sign-off.

### When Human Approval Is Triggered

- Composite Quality Score falls below the threshold
- Any critical gate failure is detected
- The output touches a domain designated as requiring human oversight (financial transactions, medical records, legal documents, infrastructure changes)
- The output is novel — no similar validated output exists in the approval history for calibration
- A regulatory requirement mandates human sign-off (e.g., maker-checker for financial controls)

### Human Approval Interface

Present the reviewer with a structured approval package:

1. **Summary** — One-paragraph description of what the AI generated and its purpose
2. **Quality Score** — Composite score with dimension breakdown
3. **Validation Report** — Full results from all three validation dimensions, highlighting failures and warnings
4. **Diff View** — Side-by-side comparison of the generated output against the relevant baseline (existing code, previous version, or blank)
5. **Risk Flags** — Any critical gate failures or regulatory concerns surfaced prominently
6. **Approve / Request Revision / Reject** — Clear actions with mandatory comment for rejections

### Approval Audit Trail

- Record every approval and rejection with timestamp, reviewer identity, and decision rationale.
- Link the approval record to the specific output version, Quality Score, and validation report.
- Retain approval records for the duration required by applicable regulations.
- Surface approval history in downstream incident investigations to trace the origin and sign-off chain of any production artefact.

---

## Why Validation Scaffolding Matters

| Without Validation Scaffolding | With Validation Scaffolding |
|---|---|
| AI errors reach production undetected | Quality gates catch failures before deployment |
| Human reviewers must evaluate raw AI output | Reviewers receive structured reports and confidence scores |
| Compliance depends on manual audit | Regulatory checks run automatically on every output |
| Hallucinations are discovered by end users | Fact verification catches inaccuracies before publication |
| Trust in AI output is based on hope | Trust is based on measurable, auditable quality metrics |
| Incidents are hard to trace | Full approval chain links every production artefact to its validation record |

---

## Example: Enterprise Code Generation Pipeline

**Scenario:** A developer uses an AI coding assistant to generate a new payment processing endpoint.

### Step 1 — AI Generation

The AI coding assistant generates a Spring Boot controller and service layer based on the developer's description and the retrieved context from the knowledge scaffolding layer.

### Step 2 — Technical Validation

- **Compilation:** The generated code compiles successfully against Java 21 and the project's Maven dependencies.
- **Unit tests:** 18/20 tests pass; 2 tests covering edge cases in currency conversion logic fail.
- **Security scan:** SAST identifies one medium-severity finding: a missing input validation annotation on a request parameter.
- **Performance:** No N+1 queries detected; response time estimate is within the SLA threshold.

**Technical Score: 72** (test failures and security finding reduce the score)

### Step 3 — Business Validation

- **Requirements compliance:** All 6 acceptance criteria from the Jira story are addressed.
- **Policy check:** The implementation uses the approved payment SDK; logging follows the enterprise standard.
- **Regulatory check:** PCI-DSS cardholder data is tokenised; audit logging is present. GDPR: no PII stored beyond the transaction record retention policy.

**Business Score: 95**

### Step 4 — AI Validation

- **Hallucination detection:** No ungrounded claims detected; all referenced methods exist in the codebase.
- **Confidence score:** 84 — the model had moderate uncertainty on the currency conversion logic (flagged).
- **Fact verification:** API version references verified against the internal API registry.

**AI Score: 84**

### Step 5 — Quality Score

```
Quality Score = (72 × 0.40) + (95 × 0.35) + (84 × 0.25)
             = 28.8 + 33.25 + 21.0
             = 83.05
```

The score is above the 70-point medium-confidence threshold but below the 90-point auto-approve threshold. The security finding and test failures prevent auto-approval.

### Step 6 — Human Approval

The output is routed to a senior developer for review with a highlighted report:
- 2 failing tests (currency conversion edge cases)
- 1 medium security finding (missing input validation)
- Confidence flag on currency conversion logic

The reviewer corrects the two issues, re-runs the validation pipeline, and approves the revised output.

### Step 7 — Production

The approved, validated code is merged to the main branch and deployed via the standard CI/CD pipeline with the full validation and approval record attached to the pull request.

---

## Implementation Guidance

1. **Define quality gates before building validation tooling** — Agree on what constitutes a blocking failure versus a warning for each validation dimension. Tooling built without agreed gates will be calibrated inconsistently across teams.
2. **Start with the highest-risk output types** — Apply validation scaffolding first to generated code that touches production systems, financial data, or regulated domains. Lower-risk use cases (documentation drafts, internal summaries) can adopt lighter validation profiles.
3. **Automate critical gates fully** — Critical security findings, confirmed hallucinations, and regulatory violations should never reach human review undetected. Automation must catch them; human review is for judgment calls, not for finding obvious failures.
4. **Calibrate confidence thresholds empirically** — Set initial thresholds based on risk tolerance, then measure the false-positive and false-negative rates of your auto-approve decisions over time. Adjust thresholds as data accumulates.
5. **Keep the human approval interface focused** — Reviewers should see exactly what they need to make a decision and nothing more. Information overload leads to approval fatigue and rubber-stamping.
6. **Treat the validation report as an audit artefact** — Retain validation reports alongside the approved output. They are the evidence that due diligence was performed and will be required in incident post-mortems and regulatory audits.
7. **Feed validation failures back into generation** — Route outputs that fail validation back to the generation layer with the validation report as additional context. Structured failure feedback enables the AI to self-correct rather than requiring full human rework.
8. **Instrument validation pipeline performance** — Track validation throughput, gate failure rates by dimension, and human approval cycle times. Bottlenecks in validation are bottlenecks in the entire AI delivery pipeline.
9. **Version validation rules alongside code** — Validation rules are policy artefacts. Changes to security scan rules, regulatory checks, or quality thresholds must go through change management and be versioned so that outputs can be re-evaluated against the rules that were active when they were approved.

---

## Relationship to the 8-Dimensional Scaffolding Framework

Validation scaffolding is the enforcement layer that gives the 8-Dimensional Framework its teeth — it is the mechanism through which stated standards, policies, and regulatory requirements are actually applied to AI outputs:

- **Strategy and Use-Case Selection** — Quality gates and risk tiers align validation intensity to use-case risk, ensuring that strategic commitments to responsible AI are operationalised rather than aspirational.
- **Data Foundation** — Fact verification and hallucination detection depend on a well-maintained knowledge layer; validation quality is a direct function of knowledge scaffolding quality.
- **Platform and Architecture** — The Validation Agent and its supporting tools (SAST scanners, test runners, fact-check services) are shared platform components consumed by all AI pipelines, not rebuilt per use case.
- **Security, Governance, and Compliance** — Security scans, regulatory checks, and the human approval audit trail are the primary mechanisms through which governance commitments are demonstrated to auditors and regulators.
- **Model Lifecycle and MLOps** — Validation scores and failure rates are model evaluation signals; systematic validation failures across a category indicate a model performance or prompt engineering gap that feeds back into the MLOps pipeline.
- **Product Delivery and Change Management** — The human approval gate integrates with existing change management workflows; validated AI outputs go through the same promotion pipeline as human-authored changes.
- **Operations and Continuous Improvement** — Gate failure rates, confidence score distributions, and approval cycle times are operational metrics that drive continuous improvement of both the validation rules and the upstream generation pipeline.
- **Value Realization and Portfolio Scaling** — Trust in AI output, once established through a consistent and transparent validation process, is the prerequisite for expanding the AI use-case portfolio; organisations that cannot demonstrate output quality cannot scale responsibly.

---

## Summary

Validation scaffolding is the enterprise trust layer for AI output. By routing every generated artefact through structured technical, business, and AI validation — and by providing human reviewers with scored, evidence-backed approval packages rather than raw AI output — organisations can deploy AI at scale with the same quality standards they apply to human-authored work. The result is an AI delivery pipeline where quality is measurable, failures are caught early, compliance is automated, and every production artefact carries an auditable record of the validation and approval it passed through.
