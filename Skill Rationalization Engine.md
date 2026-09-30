# Skill Rationalization Engine

## 1. Executive Summary

The **Skill Rationalization Engine (SRE)** is an enterprise capability-discovery and rationalization platform designed to analyze an existing AI/agent marketplace, identify duplicated or highly similar Skills and Agents across Plugins, establish canonical reusable capabilities, and enable governed versioning and composition.

The core principle is:

> **Build capabilities once. Compose them many times. Version independently. Govern centrally.**

In a traditional plugin-centric marketplace, each Plugin may package its own Agent, Skills, prompts, MCP tools, policies, and workflows. As the marketplace grows, the same capability is frequently implemented multiple times across different Plugins.

The Skill Rationalization Engine separates **capability** from **packaging**.

Instead of:

~~~
Plugin A
 ├── Agent A
 │    ├── Skill X
 │    ├── Skill Y
 │    └── Skill Z

Plugin B
 ├── Agent B
 │    ├── Skill X
 │    ├── Skill Y
 │    └── Skill Q
~~~

the target model is:

~~~
                    Skill Registry
                         │
        ┌────────────────┼────────────────┐
        │                │                │
      Skill X          Skill Y          Skill Z
        │                │                │
        └────────────────┼────────────────┘
                         │
                    Agent / Plugin
~~~

The Engine provides:

- Marketplace inventory and capability extraction
- Skill and Agent duplicate detection
- Semantic similarity and structural comparison
- Canonical Skill identification
- Skill dependency and lineage graphs
- Skill version management
- Fork/derive workflows
- Compatibility analysis
- Regression and evaluation testing
- Impact analysis
- Governance and certification metadata
- Plugin-to-Skill migration recommendations

---

## 2. Problem Statement

Enterprise AI marketplaces increasingly contain Plugins that bundle:

- Agents
- Skills
- Prompts
- MCP servers
- MCP tools
- APIs
- Knowledge sources
- Policies
- Workflows

This packaging model is convenient for initial adoption but creates a significant duplication problem.

### Example

Plugin A:

~~~
Customer Support Plugin
 └── Customer Support Agent
      ├── Customer Lookup
      ├── Case Analysis
      ├── Email Generation
      └── Knowledge Search
~~~

Plugin B:

~~~
Operations Plugin
 └── Operations Agent
      ├── Customer Lookup
      ├── Incident Analysis
      ├── Email Generation
      └── Knowledge Search
~~~

The marketplace now contains multiple implementations of:

- Customer Lookup
- Email Generation
- Knowledge Search

This leads to:

1. Capability duplication
2. Inconsistent behavior
3. Higher maintenance cost
4. Security-control duplication
5. Evaluation duplication
6. Version fragmentation
7. Difficult dependency management
8. Poor discoverability
9. Inconsistent quality
10. Difficult impact analysis

The Skill Rationalization Engine addresses this by making **Skill the primary reusable capability unit**.

---

## 3. Vision

Create an enterprise **Capability Graph** where Plugins are compositions of reusable, governed Skills rather than isolated implementations.

Target model:

~~~
Skill
  ↓
Capability
  ↓
Skill Bundle
  ↓
Agent
  ↓
Plugin
  ↓
Marketplace
~~~

A Plugin becomes a packaging and deployment construct.

A Skill becomes the reusable enterprise capability.

---

## 4. Goals

### Primary Goals

1. Discover Skills and Agents embedded inside existing Plugins.
2. Detect duplicate and near-duplicate capabilities.
3. Identify canonical Skills.
4. Build a reusable Skill Registry.
5. Support semantic and structural Skill similarity.
6. Support Skill versioning using semantic versioning.
7. Support Skill derivation/forking.
8. Track parent-child Skill lineage.
9. Track Skill dependencies.
10. Identify impacted Agents and Plugins before a Skill change.
11. Execute compatibility and regression evaluations.
12. Provide governance and certification metadata.
13. Enable users to compose Agents from existing Skills.
14. Reduce duplicate capability development.
15. Provide migration recommendations for existing Plugins.

### Non-Goals

The initial platform should not attempt to:

- Replace every existing Plugin.
- Force immediate migration of existing Plugins.
- Become a general-purpose LLM platform.
- Own every model-serving capability.
- Require a vector database as a mandatory dependency.
- Replace MCP.
- Replace an existing enterprise marketplace.

---

## 5. Core Concepts

### 5.1 Skill

A reusable AI capability that performs a well-defined function.

Examples:

- Customer Lookup
- SQL Analysis
- Code Review
- Incident Analysis
- Document Classification
- PII Detection
- Data Lineage
- API Analysis
- Email Generation
- Security Review

A Skill should have explicit:

- Name
- Version
- Description
- Inputs
- Outputs
- Dependencies
- Permissions
- Tools
- Policies
- Evaluation criteria
- Risk classification
- Owner
- Lifecycle state

### 5.2 Agent

An execution/orchestration component that composes one or more Skills and tools to accomplish a task.

### 5.3 Plugin

A deployable or marketplace package containing an Agent, Skills, MCP integrations, configuration, policies, and supporting assets.

### 5.4 Skill Bundle

A predefined composition of Skills for a common use case.

Example:

~~~
Fraud Investigation Bundle
 ├── Transaction Analysis
 ├── Fraud Pattern Detection
 ├── Customer Lookup
 ├── Risk Scoring
 └── Evidence Summarization
~~~

### 5.5 Canonical Skill

The enterprise-approved reusable Skill representing the preferred implementation for a capability cluster.

### 5.6 Derived Skill

A Skill created from another Skill while maintaining explicit lineage.

Example:

~~~
CustomerLookup v2.0
        │
        └── CustomerLookup-FirmA v1.0
~~~

---

## 6. Target Architecture

~~~
                         ENTERPRISE AI MARKETPLACE
                                    │
                     ┌──────────────┴──────────────┐
                     │                             │
                Existing Plugins              New Skills
                     │                             │
                     ▼                             ▼
             Capability Extractor           Skill Registry
                     │                             │
                     └──────────────┬──────────────┘
                                    ▼
                         Rationalization Engine
                                    │
             ┌──────────────────────┼──────────────────────┐
             │                      │                      │
             ▼                      ▼                      ▼
       Similarity Engine      Dependency Graph       Lineage Engine
             │                      │                      │
             └──────────────────────┼──────────────────────┘
                                    ▼
                          Canonicalization Engine
                                    │
                                    ▼
                             Skill Registry
                                    │
                  ┌─────────────────┼─────────────────┐
                  ▼                 ▼                 ▼
             Versioning         Evaluation         Governance
                  │                 │                 │
                  └─────────────────┼─────────────────┘
                                    ▼
                            Capability Graph
                                    │
                     ┌──────────────┼──────────────┐
                     ▼              ▼              ▼
                  Agents        Bundles         Plugins
~~~

---

## 7. Major Components

### 7.1 Marketplace Ingestion Layer

Ingest existing marketplace artifacts.

Supported inputs may include:

- Plugin manifests
- Agent definitions
- SKILL.md files
- Prompt files
- Tool definitions
- MCP manifests
- OpenAPI specifications
- Configuration files
- Documentation
- Evaluation datasets

The ingestion layer should preserve source lineage.

Example:

~~~json
{
  "source": {
    "type": "plugin",
    "id": "customer-support-plugin",
    "version": "3.2.0"
  },
  "artifact": {
    "type": "skill",
    "path": "skills/customer-lookup/SKILL.md"
  }
}
~~~

---

## 8. Capability Extraction

The extraction engine converts Plugin contents into normalized capability records.

### Extraction targets

~~~
Plugin
 ├── Agents
 ├── Skills
 ├── Prompts
 ├── Tools
 ├── MCP Servers
 ├── MCP Resources
 ├── APIs
 ├── Policies
 └── Dependencies
~~~

The extractor should use deterministic parsing first and LLM-based analysis only where semantic interpretation is required.

### Recommended pipeline

~~~
Repository / Plugin
       ↓
Deterministic Parser
       ↓
Artifact Normalization
       ↓
Metadata Extraction
       ↓
Semantic Analysis
       ↓
Capability Record
~~~

---

## 9. Skill Normalization

Two Skills may use different names while implementing essentially the same capability.

Examples:

~~~
Customer Finder
Customer Lookup
Retrieve Customer
Get Customer Details
Customer Information Resolver
~~~

The normalization engine should generate a canonical representation.

### Normalized representation

~~~json
{
  "capability": "customer_lookup",
  "intent": "retrieve_customer_information",
  "domain": "customer",
  "inputs": ["customer_id"],
  "outputs": ["customer_profile"],
  "actions": ["read"],
  "systems": ["crm"],
  "risk": "medium"
}
~~~

---

## 10. Duplicate Detection

Duplicate detection should combine multiple signals.

### 10.1 Name similarity

Compare:

- Skill names
- Descriptions
- Titles

### 10.2 Semantic similarity

Compare:

- Skill purpose
- Instructions
- Inputs
- Outputs
- Examples

### 10.3 Structural similarity

Compare:

- Tools
- MCP dependencies
- APIs
- Data sources
- Workflow steps

### 10.4 Behavioral similarity

Compare evaluation scenarios and observed behavior.

### 10.5 Dependency similarity

Compare underlying systems and capabilities.

### Similarity model

~~~text
Similarity =
  w1 * NameSimilarity
+ w2 * SemanticSimilarity
+ w3 * StructuralSimilarity
+ w4 * BehavioralSimilarity
+ w5 * DependencySimilarity
~~~

Weights should be configurable by domain.

---

## 11. Rationalization Pipeline

~~~
Marketplace
    ↓
Extract
    ↓
Normalize
    ↓
Cluster
    ↓
Detect Similarity
    ↓
Human / Policy Review
    ↓
Select Canonical Skill
    ↓
Create Skill Registry Entry
    ↓
Map Existing Plugins
    ↓
Track Dependencies
    ↓
Recommend Migration
~~~

The system should not automatically merge capabilities solely from a similarity score.

High similarity should generate a **rationalization candidate** for review.

---

## 12. Canonical Skill Selection

For each capability cluster, the Engine should identify candidate canonical implementations based on configurable evidence.

Possible evidence:

- Existing adoption
- Evaluation quality
- Security posture
- Reliability
- Documentation quality
- Dependency health
- Ownership
- Version maturity
- Maintenance status
- Certification status

The platform should expose the evidence rather than hiding the rationale.

Example:

~~~
Capability Cluster:
customer_lookup

Implementations:
 ├── Plugin A / Skill X
 ├── Plugin B / Skill Y
 └── Plugin C / Skill Z

Candidate canonical Skill:
customer-lookup

Evidence:
 ├── Shared capability intent
 ├── Common CRM dependency
 ├── Compatible input/output contract
 ├── Existing evaluation suite
 └── Existing enterprise ownership
~~~

---

## 13. Skill Registry

The Skill Registry becomes the system of record for reusable capabilities.

Example:

~~~
skills/
├── customer-lookup/
│   ├── 1.0.0
│   ├── 1.1.0
│   └── 2.0.0
│
├── sql-analysis/
│   ├── 1.0.0
│   └── 2.0.0
│
└── incident-analysis/
    ├── 1.0.0
    └── 1.2.0
~~~

Each version should be immutable after publication.

---

## 14. Skill Manifest

Example:

~~~yaml
name: customer-lookup
version: 1.2.0

description: >
  Retrieve customer information from the enterprise CRM.

inputs:
  customer_id:
    type: string
    required: true

outputs:
  customer:
    type: object

dependencies:
  mcp:
    - crm-mcp: ">=2.1,<3.0"

permissions:
  - crm.customer.read

risk:
  classification: medium

evaluation:
  suite: customer-lookup-regression
  minimum_score: 0.90

compatibility:
  runtime: ">=1.4"

lineage:
  parent: null

ownership:
  team: customer-platform

lifecycle:
  status: certified
~~~

---

## 15. Skill Versioning

Use semantic versioning:

~~~
MAJOR.MINOR.PATCH
~~~

### Patch

Backward-compatible bug fixes.

~~~
1.0.0 → 1.0.1
~~~

### Minor

Backward-compatible functionality.

~~~
1.0.0 → 1.1.0
~~~

### Major

Breaking changes.

~~~
1.1.0 → 2.0.0
~~~

---

## 16. Dependency Resolution

Agents and Plugins should declare Skill dependencies.

Example:

~~~yaml
dependencies:
  skills:
    - customer-lookup: "^1.2"
    - email-generation: "~2.1.0"
~~~

The resolver should:

1. Identify compatible versions.
2. Detect conflicts.
3. Detect deprecated versions.
4. Detect security restrictions.
5. Identify transitive dependencies.
6. Produce a resolved dependency graph.

Example:

~~~
Agent A
 ├── customer-lookup ^1.2
 │     └── crm-mcp >=2.1
 │
 └── email-generation ^2.0
       └── notification-mcp >=3.0
~~~

---

## 17. Skill Forking / Derivation

Users should be able to derive a Skill when the canonical Skill does not fully satisfy their requirements.

Example:

~~~
Canonical:
customer-lookup v2.0.0
        │
        ▼
Derive
        │
        ▼
customer-lookup-enterpriseX v1.0.0
~~~

The derived Skill must preserve:

- Parent Skill
- Parent version
- Changes
- Owner
- Reason for derivation
- Compatibility information

Example:

~~~yaml
lineage:
  type: derived
  parent:
    name: customer-lookup
    version: 2.0.0

changes:
  - Added customer hierarchy resolution
  - Added regional routing
  - Added organization-specific validation
~~~

---

## 18. Skill Lineage Graph

The Engine should maintain lineage as a graph.

~~~
customer-lookup v1.0
        │
        ├── v1.1
        │
        └── v2.0
              │
              ├── enterpriseA v1.0
              │
              └── enterpriseB v1.0
~~~

This enables:

- Change impact
- Dependency analysis
- Security propagation
- Version history
- Ownership tracking
- Migration planning

---

## 19. Impact Analysis

Before publishing a breaking Skill version:

~~~
customer-lookup v2 → v3
~~~

the Engine should calculate:

~~~
Consumers:
 ├── 23 Agents
 ├── 11 Plugins
 ├── 7 Skill Bundles
 └── 4 Workflows

Potentially affected:
 ├── Agent A
 ├── Agent C
 ├── Plugin X
 └── Plugin Y
~~~

It should then run compatibility evaluations.

---

## 20. Compatibility Testing

Every Skill version should have a regression suite.

~~~
Skill v3
   ↓
Dependency Discovery
   ↓
Find Consumers
   ↓
Generate / Load Test Scenarios
   ↓
Execute
   ↓
Compare
   ↓
Compatibility Report
~~~

Example:

~~~
CustomerLookup v3

Agent A       PASS
Agent B       PASS
Agent C       FAIL
Agent D       PASS

Compatibility: 75%

Blocking consumer:
Agent C
Reason:
Output schema changed.
~~~

---

## 21. Evaluation Framework

Skill quality should be evaluated independently of the Plugin.

Evaluation dimensions:

- Functional correctness
- Instruction adherence
- Groundedness
- Tool selection
- Tool-call correctness
- Security
- Prompt injection resistance
- Data leakage
- Reliability
- Latency
- Cost
- Regression behavior

Example:

~~~
Skill Evaluation

Functional correctness       95%
Groundedness                 93%
Tool selection               91%
Security                     PASS
Prompt injection             PASS
Regression                   PASS
P95 latency                  2.8s
Average cost                 $0.021/task
~~~

---

## 22. Skill Assurance

A Skill should have an assurance lifecycle:

~~~
Draft
  ↓
Development
  ↓
Evaluation
  ↓
Security Testing
  ↓
Review
  ↓
Certified
  ↓
Published
  ↓
Deprecated
  ↓
Retired
~~~

Certification metadata should be attached to the Skill version rather than only the Plugin.

This allows one certified Skill to be reused by many Plugins.

---

## 23. Plugin-to-Skill Migration

The Engine should provide a migration path rather than requiring immediate marketplace redesign.

### Current state

~~~
Plugin A
 └── Embedded Skill X
~~~

### Target state

~~~
Skill Registry
 └── Skill X v1.0

Plugin A
 └── dependency:
      Skill X ^1.0
~~~

Migration phases:

1. Discover
2. Extract
3. Normalize
4. Compare
5. Register
6. Validate
7. Link
8. Deprecate embedded copy
9. Migrate Plugin
10. Remove duplicate implementation

---

## 24. Capability Graph

The platform should maintain a graph connecting:

~~~
Plugin
  ↓
Agent
  ↓
Skill
  ↓
Skill Version
  ↓
MCP Tool
  ↓
MCP Server
  ↓
API / Data Source
  ↓
Policy
  ↓
Evaluation
  ↓
Certification
~~~

This graph enables queries such as:

- Which Plugins use this Skill?
- Which Agents depend on this MCP server?
- Which Skills access customer PII?
- What breaks if this Skill is deprecated?
- Which duplicate Skills exist?
- Which canonical Skill can replace this implementation?
- Which Skills are uncertified?
- Which Skill versions are vulnerable or outdated?

---

## 25. Discovery Experience

Users should be able to search for capabilities rather than only Plugins.

Example:

~~~
User:
"I need a capability that can analyze SQL queries and identify
performance problems."

Marketplace:

Relevant Skills
 ├── SQL Performance Analysis
 ├── SQL Query Optimization
 ├── Database Performance Analysis
 └── Query Plan Analysis
~~~

The user can then:

- Use Skill
- Add Skill to Agent
- Add Skill to Plugin
- Create Skill Bundle
- Fork Skill
- Compare versions

---

## 26. Skill Composition

A user should be able to compose an Agent from Skills.

Example:

~~~
Fraud Investigation Agent

Skills:
 ├── Transaction Analysis v2
 ├── Customer Lookup v1
 ├── Fraud Pattern Detection v3
 ├── Risk Scoring v2
 └── Evidence Summarization v1
~~~

The platform validates:

- Dependencies
- Permissions
- Version compatibility
- Risk
- Security
- Evaluation coverage

---

## 27. Skill Rationalization Report

A typical report should contain:

### Executive Summary

~~~
Marketplace inventory:
  500 Plugins

Extracted:
  2,000 Skills

Potential duplicate clusters:
  600

High-confidence duplicate clusters:
  185

Potential canonical Skills:
  160
~~~

### Duplicate Cluster

~~~
Capability:
Customer Lookup

Implementations:
 ├── Plugin A / Skill X
 ├── Plugin B / Skill Y
 ├── Plugin C / Skill Z

Semantic similarity:
94%

Structural similarity:
89%

Dependency similarity:
100%

Recommendation:
Create canonical Skill customer-lookup.
~~~

The report should provide evidence and allow human review.

---

## 28. Governance Model

Recommended roles:

### Skill Owner

Responsible for capability implementation and lifecycle.

### Skill Maintainer

Responsible for updates and defect fixes.

### Domain Owner

Responsible for domain-level standards.

### Platform/CoE

Responsible for:

- Registry
- Standards
- Evaluation framework
- Security framework
- Lifecycle controls

### Consumer Team

Uses Skills and may create derived versions.

---

## 29. Lifecycle States

Recommended lifecycle:

~~~
DISCOVERED
    ↓
CANDIDATE
    ↓
UNDER_REVIEW
    ↓
REGISTERED
    ↓
CERTIFIED
    ↓
PUBLISHED
    ↓
DEPRECATED
    ↓
RETIRED
~~~

A Skill can also be:

~~~
DERIVED
FORKED
BLOCKED
SUSPENDED
~~~

---

## 30. Security Requirements

The Engine must treat Skills as executable AI capabilities.

Security metadata should include:

- Required permissions
- Data classifications
- Tool access
- MCP servers
- Network access
- External communication
- PII handling
- Secrets access
- Human approval requirements
- Risk classification

A Skill should never automatically inherit broader permissions than its parent without explicit review.

---

## 31. Architecture Principles

### Principle 1 — Skill First

Treat Skill as the reusable capability primitive.

### Principle 2 — Plugin as Composition

Plugins should compose existing capabilities wherever possible.

### Principle 3 — Version Everything

Skill versions must be immutable and traceable.

### Principle 4 — Preserve Lineage

Every derived capability must identify its parent.

### Principle 5 — Evidence-Based Rationalization

Similarity detection generates candidates; governance decides canonicalization.

### Principle 6 — Evaluation Before Promotion

A new Skill version must pass its required evaluation suite.

### Principle 7 — Dependency Awareness

Every Skill should declare its dependencies.

### Principle 8 — No Mandatory Vector Database

The initial implementation should support deterministic and lightweight semantic techniques. A vector database may be introduced later if scale requires it.

### Principle 9 — Open Protocol Alignment

The platform should integrate with MCP, Skills, A2A and OpenTelemetry rather than creating proprietary replacements.

### Principle 10 — Human Governance

High-impact changes should remain reviewable and auditable.

---

## 32. Recommended Technical Architecture

A lightweight initial implementation can use:

~~~
API:
  FastAPI

Core:
  Python

Metadata:
  PostgreSQL or SQLite for MVP

Search:
  BM25 / Tantivy / lightweight lexical search

Semantic similarity:
  Embeddings + local similarity index

Graph:
  PostgreSQL recursive queries initially
  or a graph database later

Evaluation:
  Python evaluation framework

Telemetry:
  OpenTelemetry

Policy:
  OPA or Cedar

Protocol:
  MCP
  A2A where applicable

Runtime:
  Containers / Kubernetes where required

LLM:
  Enterprise-approved model gateway
~~~

The architecture should avoid introducing infrastructure that is not justified by scale.

---

## 33. MVP

### Phase 1 — Inventory

Build:

- Plugin ingestion
- Agent extraction
- Skill extraction
- Skill normalization
- Metadata store

Output:

~~~
Plugin → Agent → Skill inventory
~~~

### Phase 2 — Rationalization

Build:

- Duplicate detection
- Similarity clustering
- Capability normalization
- Candidate canonical Skills

Output:

~~~
Skill clusters
Duplicate candidates
Canonical candidates
~~~

### Phase 3 — Registry

Build:

- Skill registry
- Skill manifest
- Versioning
- Search
- Ownership
- Lifecycle

### Phase 4 — Dependency and Lineage

Build:

- Dependency graph
- Skill lineage
- Plugin relationships
- Impact analysis

### Phase 5 — Assurance

Build:

- Evaluation suites
- Regression testing
- Security validation
- Certification metadata

### Phase 6 — Composition

Build:

- Add Skill to Agent
- Skill bundles
- Plugin dependency resolution
- Version selection

---

## 34. Future Capabilities

### Intelligent Rationalization Agent

An Agent can continuously inspect the marketplace and identify:

- New duplicate Skills
- Deprecated capabilities
- Version fragmentation
- Missing canonical Skills
- Unused Skills
- High-risk Skills
- Dependency conflicts

### Automatic Migration Planner

Generate:

~~~
Plugin A
Current:
  embedded CustomerLookup

Target:
  customer-lookup v2.1

Migration:
  Replace embedded implementation.
  Update manifest.
  Run regression suite.
  Validate permissions.
~~~

### Continuous Capability Governance

~~~
New Plugin
   ↓
Capability Extraction
   ↓
Duplicate Detection
   ↓
Dependency Analysis
   ↓
Security Evaluation
   ↓
Rationalization
   ↓
Marketplace Publication
~~~

---

## 35. Success Metrics

Measure the platform using engineering outcomes.

### Reuse

~~~
Skill reuse ratio =
Skill consumer relationships / total Skill implementations
~~~

### Duplication reduction

~~~
Duplicate capabilities before
        vs.
Duplicate capabilities after
~~~

### Development acceleration

Measure reduction in time to create a new Agent.

### Governance coverage

Percentage of Skills with:

- Owner
- Version
- Evaluation
- Security classification
- Dependency metadata

### Migration

Number of embedded Plugin capabilities migrated to canonical Skills.

### Quality

Regression failure rate after Skill upgrades.

---

## 36. Example End-to-End Scenario

### Current Marketplace

~~~
Plugin A
 └── Customer Lookup

Plugin B
 └── Customer Finder

Plugin C
 └── Retrieve Customer

Plugin D
 └── Customer Resolver
~~~

### Engine

~~~
Extract
  ↓
Normalize
  ↓
Semantic similarity
  ↓
Structural comparison
  ↓
Dependency comparison
  ↓
Cluster
~~~

Result:

~~~
Cluster:
Customer Lookup

Similarity:
92–96%

Common dependency:
CRM

Common input:
customer_id

Common output:
customer_profile
~~~

### Rationalization

Create:

~~~
customer-lookup v1.0.0
~~~

### Migration

~~~
Plugin A → customer-lookup ^1.0
Plugin B → customer-lookup ^1.0
Plugin C → customer-lookup ^1.0
Plugin D → customer-lookup ^1.0
~~~

### Custom requirement

Plugin D needs regional customer routing.

Instead of creating another independent Skill:

~~~
customer-lookup v1.0
        ↓
derive
        ↓
customer-lookup-regional v1.0
~~~

Lineage remains visible.

---

## 37. Strategic Positioning

The Skill Rationalization Engine should be positioned as an **enterprise capability layer**, not merely a duplicate detector.

Its value proposition is:

> **Discover capabilities. Rationalize duplication. Create canonical Skills. Version independently. Track lineage. Evaluate continuously. Compose everywhere.**

This enables a transition from:

~~~
Plugin-Centric AI Marketplace
~~~

to:

~~~
Capability-Centric AI Marketplace
~~~

The resulting enterprise model becomes:

~~~
                 CAPABILITY MARKETPLACE

                       Skills
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           Agents      Bundles     Plugins
              │          │          │
              └──────────┼──────────┘
                         ▼
                  Enterprise AI
~~~

---

## 38. Relationship to Agent Assurance

The Skill Rationalization Engine should become a foundational component of an **Agent Assurance Platform**.

~~~
                Agent Assurance Platform
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
       Skill Registry   Assurance     Runtime
             │             │             │
             ▼             ▼             ▼
       Rationalization   Evaluation   Control Plane
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                    Trust Registry
~~~

This creates a closed lifecycle:

~~~
Discover
   ↓
Rationalize
   ↓
Register
   ↓
Version
   ↓
Evaluate
   ↓
Certify
   ↓
Compose
   ↓
Deploy
   ↓
Observe
   ↓
Improve
~~~

---

## 39. Recommended Repository Evolution

The existing Agent Assurance platform can evolve toward:

~~~
Agent-Assurance-Certification-Platform/
│
├── docs/
│   ├── Skill Rationalization Engine.md
│   ├── Agent Runtime Control Plane.md
│   └── Agent Assurance Architecture.md
│
├── skill-registry/
│   ├── manifests/
│   ├── versions/
│   └── lineage/
│
├── rationalization/
│   ├── extractor/
│   ├── normalizer/
│   ├── similarity/
│   ├── clustering/
│   └── recommendations/
│
├── dependency-graph/
│
├── evaluation/
│
├── policy/
│
├── runtime/
│
└── observability/
~~~

The Skill Rationalization Engine should initially be implemented as an independent module so it can operate against the existing Plugin marketplace without requiring a marketplace rewrite.

---

## 40. Final Architectural Principle

The long-term platform should move from:

~~~
"Which Plugin should I install?"
~~~

to:

~~~
"What capability do I need?"
~~~

The marketplace then determines:

~~~
Capability
   ↓
Best available Skill
   ↓
Compatible version
   ↓
Required dependencies
   ↓
Security / policy requirements
   ↓
Evaluation status
   ↓
Agent / Plugin composition
~~~

This establishes **Skills as reusable enterprise AI building blocks** while preserving Plugins as convenient packaged experiences.

The strategic outcome is a marketplace where capabilities are developed once, governed once, evaluated once, and reused across many Agents and Plugins.
