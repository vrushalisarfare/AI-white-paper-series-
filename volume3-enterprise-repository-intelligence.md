# Volume 3: Enterprise Repository Intelligence

*Enterprise AI Blueprint Series — Reference Architectures, Frameworks and Playbooks for Enterprise AI*

---

## Purpose

Define the reference architecture for AI-driven code intelligence across enterprise codebases — enabling semantic search, knowledge graph construction, developer productivity tooling, and automated insight generation at the scale of large polyglot monorepos and multi-repository enterprise estates.

---

## The Problem: Enterprise Codebases Are Unmapped

Enterprise software estates grow over decades. A typical large enterprise has:

- Hundreds to thousands of repositories across dozens of teams
- Multiple programming languages, frameworks, and vintage generations
- Implicit architectural knowledge trapped in the heads of long-tenured engineers
- Dependencies that are undocumented, circular, or unknown to current maintainers
- Security and compliance risk embedded in code that no one has reviewed in years

Traditional code search (lexical grep) and manual documentation cannot scale to this reality. Enterprise Repository Intelligence (ERI) uses AI to make the entire codebase queryable, navigable, and understandable — treating the repository estate as a first-class enterprise knowledge asset.

---

## Reference Architecture

### Architectural Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    Developer Experience                      │
│   (IDE Plugins · Chat Interface · PR Insights · CLI)        │
├─────────────────────────────────────────────────────────────┤
│                    Intelligence Services                     │
│  Semantic Search · Code Q&A · Impact Analysis ·             │
│  Dependency Mapping · Security Intelligence                  │
├─────────────────────────────────────────────────────────────┤
│                    Knowledge Representation                  │
│  Code Knowledge Graph · Vector Index · Symbol Index ·       │
│  Dependency Graph · Semantic Embeddings                      │
├─────────────────────────────────────────────────────────────┤
│                    Processing Pipeline                       │
│  Ingestion · Parsing · Chunking · Embedding ·               │
│  Graph Construction · Incremental Updates                    │
├─────────────────────────────────────────────────────────────┤
│                    Source Layer                              │
│  Git Repositories · CI/CD Artifacts · Issue Trackers ·      │
│  Pull Requests · Documentation · Test Results               │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. Ingestion Pipeline

The foundation of ERI is a continuous ingestion pipeline that keeps the knowledge representation current as code evolves.

#### Ingestion Sources

| Source | Content Captured |
|---|---|
| Git repositories | All branches, commits, files, and history |
| Pull requests and reviews | Change intent, review discussions, approval decisions |
| CI/CD pipelines | Build metadata, test results, deployment history |
| Issue and work item trackers | Feature intent, bug reports, acceptance criteria |
| Wiki and documentation | Architecture decisions, runbooks, API documentation |
| Dependency manifests | Package dependencies, transitive graphs, version locks |

#### Ingestion Strategies

- **Full ingestion:** Complete re-indexing of the repository estate on initial setup or after major restructuring.
- **Incremental ingestion:** Event-driven updates triggered by push, merge, or release events — keeping the index fresh within minutes of code changes.
- **Priority ingestion:** Real-time indexing of files touched in active pull requests to support PR-time intelligence.

---

### 2. Code Parsing and Chunking

Raw source files must be parsed into semantically meaningful units before embedding or graph construction.

#### Parsing Approaches

| Language Type | Parsing Strategy |
|---|---|
| Strongly typed (Java, C#, Go, TypeScript) | AST-based parsing using tree-sitter or language-specific parsers |
| Dynamically typed (Python, JavaScript, Ruby) | AST parsing with type inference signals from existing type annotations |
| Infrastructure as Code (Terraform, Bicep, YAML) | Schema-aware parsing to extract resource declarations and dependencies |
| Legacy (COBOL, PL/I, FORTRAN) | Specialized parsers that extract program units, data declarations, and call graphs |
| Configuration and data (JSON, YAML, TOML) | Schema-guided parsing to extract meaningful key-value structures |

#### Chunking Strategy

- **Symbol-level chunks:** Individual functions, classes, interfaces, and modules as atomic units.
- **File-level chunks:** File summaries that describe purpose, key exports, and dependencies.
- **Cross-file chunks:** Related code groups identified by import relationships and call graphs.
- **Context-enriched chunks:** Chunks annotated with commit history, author, PR origin, test coverage, and quality signals.

---

### 3. Embedding and Vector Index

Semantic search requires a dense vector representation of code and documentation.

#### Embedding Considerations

| Consideration | Guidance |
|---|---|
| Model selection | Use code-specialized embedding models (e.g., trained on code corpora) for higher semantic fidelity on technical content |
| Embedding dimensionality | Balance retrieval quality against index size and query latency for your corpus size |
| Multi-modal embeddings | Embed code, comments, documentation, and test descriptions in the same space to enable cross-modal retrieval |
| Incremental updates | Support upsert operations so changed files update their embeddings without full re-indexing |

#### Vector Index Design

- Maintain separate sub-indexes for code symbols, file summaries, documentation, and test cases.
- Support filtered retrieval by language, repository, team, file path prefix, and modification date.
- Apply metadata governance: tag chunks with data classification (internal, confidential) and enforce retrieval access controls.

---

### 4. Code Knowledge Graph

The knowledge graph is the structural backbone of ERI — capturing relationships that vector search cannot express.

#### Graph Node Types

| Node Type | Represents |
|---|---|
| Repository | A distinct code repository in the estate |
| File | A source file within a repository |
| Symbol | A named entity: class, function, interface, variable |
| Dependency | An external package or library |
| Commit | A version-controlled change |
| Author | A developer identity |
| Test | A test case or test suite |
| Issue | A work item, bug report, or feature request |

#### Graph Edge Types

| Edge Type | Represents |
|---|---|
| `CONTAINS` | Repository contains file; file contains symbol |
| `CALLS` | Function calls another function |
| `IMPORTS` | File imports another module |
| `DEPENDS_ON` | Repository depends on external package |
| `MODIFIED_BY` | Symbol was last modified by a commit or author |
| `TESTED_BY` | Symbol is covered by a test case |
| `REFERENCED_IN` | Symbol is referenced in an issue or PR |

#### Graph Query Capabilities

- **Impact analysis:** "Which files and systems are affected if I change function X?"
- **Dependency tracing:** "Show all transitive dependencies of service Y, including indirect imports."
- **Ownership mapping:** "Who owns the code paths handling customer payment data?"
- **Test coverage gaps:** "Which public API functions have no test coverage?"
- **Change history:** "What changed in the authentication module in the last 30 days?"

---

### 5. Intelligence Services

The intelligence layer exposes repository knowledge through purpose-built services.

#### Semantic Code Search

Natural language queries over the combined vector index and knowledge graph.

- **Example queries:**
  - "Find all functions that parse user input without sanitization"
  - "Show me examples of how the retry pattern is implemented in the order service"
  - "Which modules handle PII data and what controls are applied?"

- **Results** include ranked code excerpts with source file, line range, relevance score, and cross-references to related symbols.

#### Code Q&A

Conversational answers to questions about the codebase, grounded in retrieved code context.

- "How does the authentication service validate JWT tokens?"
- "What is the data flow for a customer order from placement to fulfilment?"
- "Why does the billing service have a dependency on the inventory module?"

All answers cite the specific files and functions from which the answer is derived.

#### Impact Analysis

Automated assessment of the downstream effects of proposed changes.

- Parse a diff or proposed change and traverse the call graph and import graph.
- Identify all symbols, files, services, and systems that reference the changed entity.
- Produce a risk-ranked impact report: what is directly affected, what is indirectly affected, and what tests cover the affected scope.

#### Dependency Intelligence

Continuous visibility into the dependency estate.

- Map all first-party and third-party dependencies across the repository estate.
- Flag outdated packages, known CVEs, license violations, and deprecated APIs.
- Identify dependency duplication: the same library pulled in at multiple incompatible versions across services.

#### Security Intelligence

AI-assisted identification of security-relevant patterns in code.

- Surface code patterns associated with OWASP Top 10 vulnerabilities.
- Flag secrets, hardcoded credentials, and configuration anti-patterns.
- Identify data flows that touch sensitive data classifications without appropriate controls.
- Track security finding remediation rates over time.

---

## Polyglot Monorepo Patterns

Large enterprises often consolidate related services into monorepos. ERI must support monorepo-specific patterns:

### Monorepo Challenges

| Challenge | ERI Response |
|---|---|
| Massive repository size | Incremental indexing with priority lanes for active paths |
| Mixed language stacks | Per-language parsers with a unified graph schema |
| Shared library versions | Dependency graph tracks which services consume which version of shared libraries |
| Cross-team ownership | Graph ownership model maps code paths to team and service ownership |
| Long CI pipelines | Impact analysis narrows test scope to affected modules, accelerating CI |

### Multi-Repository Estate Patterns

For enterprises with many separate repositories:

- Federated graph: each repository has its own subgraph; cross-repository edges are created from API contracts and shared library dependencies.
- Unified search: a single query interface searches across all repositories with permission-filtered results.
- Cross-repository impact: change propagation analysis traces effects across repository boundaries through shared library version graphs.

---

## Developer Productivity Integration

### IDE Integration

- **Inline semantic search:** Developers query the codebase from within their IDE without context switching.
- **Function-level documentation generation:** On demand, generate documentation for any function by retrieving its context, callers, and test cases.
- **PR impact preview:** Before opening a PR, surface the impact analysis for the changed files.
- **Suggested test cases:** Based on changed code, suggest test cases derived from existing patterns in the codebase.

### Pull Request Intelligence

- Automatically attach an impact analysis report to every PR.
- Flag PRs that touch code with no test coverage.
- Surface relevant prior PRs that made similar changes.
- Identify reviewers who have the most context on the changed code paths, based on authorship history.

### Onboarding Acceleration

- New engineers ask natural language questions about the codebase and receive grounded, cited answers.
- "How do I set up a new service that follows our architecture patterns?"
- "What is the standard way to add a new API endpoint in this codebase?"

---

## Governance and Access Control

Repository intelligence contains sensitive information — proprietary code, security findings, and organizational knowledge. Access must be governed:

| Governance Control | Implementation |
|---|---|
| Query access control | Filter retrieval results to repositories and paths the querying user is authorized to view |
| Data classification tagging | Mark chunks containing PII, secrets references, or confidential logic with classification labels |
| Audit logging | Log all queries, retrieved chunks, and generated answers for audit and monitoring |
| Retention policies | Apply code retention policies from the source SCM to the knowledge index |
| Secret detection before indexing | Scan ingested content for secrets and redact before storing in the index |

---

## Working Example: Modernization Assessment

**Goal:** Assess 200 legacy Java services for modernization readiness.

| Step | ERI Capability Used | Output |
|---|---|---|
| 1. Map service estate | Dependency graph ingestion | Complete service inventory with inter-service dependency map |
| 2. Identify high-risk code | Security intelligence scan | Prioritized list of services with known vulnerability patterns |
| 3. Assess complexity | Symbol-level embedding + graph metrics | Complexity scores per service (cyclomatic complexity, fan-in/fan-out) |
| 4. Find modernization patterns | Semantic search for existing migration examples | Candidate patterns from services already modernized |
| 5. Produce readiness report | Code Q&A + impact analysis | Per-service modernization effort estimate and risk classification |

---

## Implementation Playbook (90 Days)

### Days 1–30: Foundation
- Deploy ingestion pipeline for top 20 highest-priority repositories.
- Implement AST-based parsing for primary language stack.
- Build basic vector index and enable semantic search.
- Establish access control and audit logging.

### Days 31–60: Graph and Intelligence
- Construct the code knowledge graph from parsed artifacts.
- Enable impact analysis and dependency intelligence services.
- Integrate semantic search into IDE and pull request workflow.
- Onboard 2–3 engineering teams as early adopters.

### Days 61–90: Scale and Expand
- Extend ingestion to full repository estate.
- Enable security intelligence and CI integration.
- Measure developer productivity signals: time-to-answer, onboarding speed, PR review time.
- Publish developer guidelines for using repository intelligence effectively.

---

## Success Metrics

| Metric | Target |
|---|---|
| Time to answer codebase questions (self-serve) | Reduced by ≥ 50% |
| Onboarding time for new engineers | Reduced by ≥ 30% |
| PRs with attached impact analysis | 100% of PRs in enrolled repositories |
| High-severity security findings surfaced pre-merge | ≥ 90% of known patterns detected |
| Developer adoption (weekly active users) | ≥ 70% of enrolled teams within 90 days |

---

## Relationship to Other EABS Volumes

| Volume | Connection |
|---|---|
| [Volume 1: Enterprise Agentic SDLC](./volume1-enterprise-agentic-sdlc.md) | Code Agent and Review Agent use repository intelligence for context and impact analysis |
| [Volume 2: Enterprise AI Skills & Scaffolding](./volume2-enterprise-ai-skills-scaffolding.md) | ERI is built on Knowledge Scaffolding (Layer 2) and Tool Scaffolding (Layer 5) |
| [Volume 4: Enterprise AI Modernization](./volume4-enterprise-ai-modernization.md) | Modernization programs use ERI for portfolio assessment and migration planning |
| [Volume 6: Responsible Enterprise AI](./volume6-responsible-enterprise-ai.md) | Access control and security intelligence serve the compliance and audit requirements |

---

## Key Message

**The codebase is the largest undiscovered knowledge asset in most enterprises.** Enterprise Repository Intelligence transforms it from an opaque archive into an interactive, queryable knowledge graph that accelerates every engineer, informs every architectural decision, and surfaces every risk — at a scale no human team could achieve alone.
