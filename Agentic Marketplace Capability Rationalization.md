# Agentic Marketplace Capability Rationalization

## Purpose

A meta-agentic capability for the Agent Assurance & Certification Platform that scans marketplace plugins, agents, skills and tools to identify duplicates, functional overlap, complementary capabilities and consolidation opportunities.

The MVP is intentionally **zero-additional-infrastructure**: no vector DB or graph DB is required. Embeddings are ephemeral and held in memory for each rationalization run.

## Architecture

```
Marketplace
   |
   v
Discovery / Scanner
   |
   v
Canonical Artifact Model
   |
   +--> Existing marketplace analysis skills
   |
   v
Capability Extraction
   |
   v
Embedding Model / Skill
   |
   v
In-memory vectors
   |
   v
Candidate reduction + cosine similarity
   |
   v
Comparison Agent
   |
   v
Critic Agent
   |
   v
Cluster / Consolidation Agent
   |
   v
Report Agent
   |
   v
Human Review
```

## Core principle

Use a hybrid architecture:

- deterministic scanning and normalization
- existing marketplace capabilities where reusable
- deterministic embedding/cosine calculations
- agentic semantic comparison
- critic-based false-positive challenge
- evidence-backed consolidation recommendations
- human approval for production changes

Do not make every operation an LLM agent.

## Why no vector database

This workload is a periodic marketplace scan, not continuous semantic retrieval over a massive corpus.

For each run:

1. discover artifacts
2. normalize artifacts
3. generate embeddings
4. hold vectors in memory
5. calculate cosine similarity
6. keep only candidate relationships
7. send candidates to comparison agents
8. discard vectors after the run

Persistent storage is still useful for artifact metadata, scan history, comparisons, evidence, reports and human decisions.

## Artifact types

Support:

- Plugin
- Agent
- Skill
- Tool
- MCP capability

Valid comparison combinations should be policy-driven.

## Canonical artifact model

Reuse the existing platform canonical model where possible. A normalized rationalization record should include:

```json
{
  "artifact_id": "customer-search",
  "artifact_type": "skill",
  "name": "Customer Search",
  "version": "1.2",
  "owner": "team-a",
  "description": "Search customer information",
  "instructions": "...",
  "business_domain": "customer",
  "capabilities": ["customer_search", "customer_profile_retrieval"],
  "actions": ["search", "retrieve"],
  "inputs": [{"name": "customer_id", "type": "string"}],
  "outputs": [{"name": "customer_profile", "type": "object"}],
  "tools": ["customer-api"],
  "apis": ["customer-api"],
  "permissions": ["customer.read"],
  "dependencies": [],
  "invocation_contract": {},
  "provenance": {}
}
```

Facts must remain separate from LLM conclusions.

## Provenance

Every material fact must identify its source.

Example:

```json
{
  "fact": "customer.update",
  "source": {
    "artifact_id": "customer-agent",
    "file": "tools/customer.py",
    "line_start": 21,
    "line_end": 42
  }
}
```

For marketplace-native metadata the source may identify the marketplace artifact and field.

## ADLC lifecycle

The rationalization workflow follows:

```
DISCOVER
  ->
UNDERSTAND
  ->
PLAN
  ->
EXECUTE
  ->
OBSERVE
  ->
EVALUATE
  ->
CRITIQUE
  ->
REFINE
  ->
REPORT
  ->
HUMAN REVIEW
```

The meta-agent should maintain explicit state and bounded execution.

## Reuse of existing marketplace capabilities

Before implementing a new analysis capability, the meta-agent should discover whether an existing marketplace skill or agent already provides it.

Example:

```
Need document parsing
   |
   v
Capability Registry
   |
   +--> Existing Document Analysis Skill
   +--> Existing Metadata Extraction Skill
   +--> Existing Report Skill
```

The rationalizer composes these capabilities through a standard adapter.

## Capability contract

Existing marketplace agents/skills should expose a common invocation contract:

```json
{
  "artifact_id": "customer-search",
  "artifact_type": "skill",
  "version": "1.2",
  "invoke": {
    "protocol": "mcp",
    "endpoint": "...",
    "input_schema": {},
    "output_schema": {}
  }
}
```

The platform should hide protocol details behind an adapter:

```python
class MarketplaceCapabilityAdapter:

    def invoke(
        self,
        artifact_id: str,
        version: str,
        input_data: dict
    ) -> dict:
        ...
```

Potential implementations:

- MarketplaceNativeAdapter
- MCPAdapter
- HTTPAdapter
- CIBRuntimeAdapter

## Capability understanding

Names are insufficient. Normalize:

- business purpose
- business domain
- actions
- entities
- inputs
- outputs
- tools
- APIs
- permissions
- data sources
- side effects
- consumers

Example:

```
Artifact: Customer Search
Capability: customer_information_retrieval
Actions: search_customer, retrieve_customer_profile
Input: customer_id
Output: customer_profile
Tool: customer-api
Permission: customer.read
```

## Embedding input

Do not indiscriminately embed entire repositories.

Create semantic text from:

```
name
description
purpose
capabilities
actions
tools
inputs
outputs
selected instruction text
```

Example:

```python
artifact_text = """
Name: Customer Search
Purpose: Search customer information
Capabilities: customer search, profile retrieval
Actions: search, retrieve
Tools: customer-api
Inputs: customer_id
Outputs: customer_profile
"""
```

Use an existing embedding skill/model where available.

Keep vectors in memory:

```python
embeddings = {
    "artifact-001": [...],
    "artifact-002": [...],
    "artifact-003": [...]
}
```

## Candidate reduction

Do not compare all artifacts with all artifacts.

Group by:

- artifact type
- business domain
- capability family
- entity
- tool/API family
- data domain
- input/output type

Example:

```
All artifacts
  |
  +-- Customer
  |    +-- Search
  |    +-- Update
  |    +-- Profile
  |
  +-- Documents
  |    +-- Search
  |    +-- Summarize
  |
  +-- Payments
       +-- Search
       +-- Payment
```

This reduces the candidate space before cosine similarity.

## Cosine similarity

Use cosine similarity as the first-pass semantic signal.

For vectors A and B:

```
              A · B
cos(theta) = -------
              |A||B|
```

Example candidate list:

```
Customer Search
  |
  +--> Customer Lookup   0.94
  +--> Customer Finder   0.91
  +--> Customer Update   0.72
  +--> Payment Search    0.31
```

Thresholds must be configurable and calibrated against the chosen embedding model and real marketplace data.

## Critical distinction

Cosine similarity means **semantic resemblance**, not duplication.

Example:

```
Customer Search
Customer Update

Cosine similarity = 0.89
```

These are related but may be materially different capabilities.

Final classification must use additional evidence.

## Multi-dimensional comparison

Candidate relationships should consider:

- semantic similarity
- capability similarity
- tool/API overlap
- input/output similarity
- instruction similarity
- permission overlap
- dependency overlap
- business-domain overlap
- side-effect similarity

Illustrative weighted score:

```
30% semantic similarity
30% capability similarity
15% tool/API similarity
10% input/output similarity
10% instruction similarity
 5% dependency similarity
```

Weights must be configurable.

## Comparison Agent

The Comparison Agent receives a candidate pair or cluster plus structured evidence.

Example:

```
Artifact A: Customer Search
Artifact B: Customer Lookup
Cosine: 0.94
Tool: customer-api
Input: customer_id
Output: customer_profile
Permission: customer.read
```

It classifies:

- EXACT_DUPLICATE
- FUNCTIONAL_DUPLICATE
- PARTIAL_OVERLAP
- COMPLEMENTARY
- UNIQUE
- UNCERTAIN

It must return evidence and confidence.

## Critic Agent

Every potential duplicate should be challenged.

Ask:

- Are permissions different?
- Are data sources different?
- Are users different?
- Are side effects different?
- Are business boundaries different?
- Are regulatory constraints different?
- Are SLAs materially different?
- Is the similarity mostly vocabulary rather than capability?

The critic can overturn the initial result.

Example:

```
Initial: FUNCTIONAL_DUPLICATE
Critic: PARTIAL_OVERLAP

Reason:
Artifact B accesses restricted customer data.
Artifact A does not.
```

## Duplication taxonomy

### Exact Duplicate

Same core implementation/capability with negligible material difference.

### Functional Duplicate

Different implementation or wording but materially equivalent business capability.

### Partial Overlap

Substantial shared capability while meaningful unique behavior remains.

### Complementary

Related artifacts that should coexist because they serve different capabilities.

### Unique

No material overlap.

### Uncertain

Evidence is insufficient or contradictory.

## Evidence model

Every material comparison should create an evidence record.

```json
{
  "comparison_id": "cmp-001",
  "artifact_a": "customer-search",
  "artifact_b": "customer-lookup",
  "semantic_similarity": 0.94,
  "capability_similarity": 0.91,
  "tool_overlap": 0.88,
  "io_similarity": 0.95,
  "classification": "FUNCTIONAL_DUPLICATE",
  "confidence": 0.93,
  "evidence": [
    {"type": "tool", "value": "customer-api"},
    {"type": "input", "value": "customer_id"},
    {"type": "output", "value": "customer_profile"}
  ],
  "critic_result": {
    "passed": true,
    "notes": "No material capability distinction found"
  }
}
```

## Consolidation Agent

After comparison, group relationships into clusters.

Example:

```
Cluster C-017

Skill A: Customer Search
Skill B: Customer Lookup
Skill C: Find Customer
Agent X: Customer Finder
```

The Consolidation Agent proposes a potential unified capability and identifies:

- affected artifacts
- potential retained artifact
- capabilities that would be lost
- permission differences
- dependencies
- migration considerations
- ownership conflicts
- human review requirements

V1 must not automatically retire, merge or delete production artifacts.

## Cluster analysis

Pairwise similarity is not sufficient.

If:

```
A ~ B
B ~ C
```

then A+B+C should be considered as a possible cluster even if A/C are slightly below the pair threshold.

For V1, represent relationships in memory rather than introducing a graph database.

## Report outputs

Generate:

```
rationalization-report.json
rationalization-report.md
rationalization-report.html
```

Report sections:

1. Marketplace inventory
2. Duplicate clusters
3. Functional overlap
4. Capability gaps
5. Consolidation opportunities
6. Risk/governance considerations
7. Uncertain cases
8. Human review queue

Every report must distinguish:

```
Observed fact
Agent interpretation
Recommendation
```

## Example report entry

```
Cluster C-017

Artifacts
1. customer-search
2. customer-lookup
3. customer-finder

Semantic similarity
customer-search <-> customer-lookup  0.94
customer-search <-> customer-finder  0.91
customer-lookup <-> customer-finder   0.89

Capability overlap: 93%
Tool overlap: 88%
Input/output overlap: 95%

Classification: FUNCTIONAL DUPLICATE
Confidence: 93%

Evidence:
- same customer retrieval purpose
- same customer API
- same customer_id input
- equivalent customer profile output

Critic: PASSED

Proposed action:
Human review for consolidation into a reusable customer-search capability.

Risks:
- ownership differs
- Artifact 2 has broader permission scope
```

## Security model

Default to read-only analysis.

Do not grant arbitrary production permissions to marketplace artifacts.

Preferred:

```
Marketplace metadata
    |
    v
Read-only definitions
    |
    v
Analysis
```

Only execute an artifact if static metadata is insufficient or empirical behavior is required.

When execution is necessary, use the existing CIB Agent Runtime through a controlled adapter.

## Circular invocation protection

Because the rationalizer can invoke existing marketplace capabilities, protect against cycles.

Track:

- execution_id
- parent_execution_id
- invocation_depth
- max_depth
- visited_artifacts
- timeout
- token_budget
- tool_budget
- cost_budget

Example:

```json
{
  "execution_id": "exec-001",
  "max_depth": 3,
  "max_tool_calls": 50,
  "max_tokens": 50000,
  "visited_artifacts": [
    "rationalizer",
    "capability-analysis"
  ]
}
```

Terminate when budgets or safety limits are exceeded.

## Versioning and reproducibility

Comparisons must be version-aware.

Record:

- artifact_id
- version
- commit
- owner
- model
- prompt version
- skill version
- tool version
- scan timestamp

Reports should identify the exact versions used.

## Bias control

No marketplace artifact should be treated as authoritative on whether another artifact is redundant.

Use independent evidence sources:

```
Capability analysis
       +
Semantic analysis
       +
Structural comparison
       +
Critic
       |
       v
Rationalization conclusion
```

## Agentic roles

### Discovery Agent

What exists?

### Capability Analysis Agent

What does this artifact actually do?

### Comparison Agent

How similar are these capabilities?

### Critic Agent

Why might they not be duplicates?

### Consolidation Agent

How could overlapping capabilities be rationalized?

### Report Agent

How should findings be communicated?

Candidate filtering and cosine similarity remain deterministic.

## Do not make everything an agent

Avoid:

```
Discovery LLM
Embedding LLM
Cosine LLM
Filtering LLM
Scoring LLM
Aggregation LLM
```

Prefer:

```
Scanner
   |
Normalizer
   |
Embedding Model
   |
Cosine Similarity
   |
Agentic Comparison
   |
Critic
   |
Consolidation
   |
Report
```

## ADLC execution state

Maintain:

- rationalization_run_id
- inventory_version
- artifact_count
- current_phase
- current_cluster
- current_pair
- candidate_queue
- comparisons_completed
- critic_results
- consolidation_candidates
- findings
- report_status

State must be serializable.

## Orchestration pseudocode

```python
def rationalize_marketplace():

    inventory = discover_marketplace()

    canonical = normalize(inventory)

    capabilities = analyze_capabilities(canonical)

    groups = group_candidates(
        capabilities,
        fields=[
            "artifact_type",
            "business_domain",
            "capability_family"
        ]
    )

    embeddings = create_embeddings(groups)

    candidate_pairs = cosine_match(
        embeddings,
        threshold=CONFIG.CANDIDATE_THRESHOLD,
        top_k=CONFIG.TOP_K
    )

    comparisons = []

    for pair in candidate_pairs:
        comparison = comparison_agent.compare(pair)

        critique = critic_agent.review(
            pair,
            comparison
        )

        comparisons.append(
            reconcile(comparison, critique)
        )

    clusters = cluster_relationships(comparisons)

    consolidation = consolidation_agent.analyze(
        clusters
    )

    return report_agent.generate(
        inventory=canonical,
        comparisons=comparisons,
        clusters=clusters,
        consolidation=consolidation
    )
```

## Threshold configuration

Example:

```yaml
similarity:
  candidate_threshold: 0.80
  strong_candidate_threshold: 0.90

comparison:
  minimum_capability_overlap: 0.70

critic:
  required_for_duplicate: true

clustering:
  minimum_edge_score: 0.85
```

These are starting values only and must be calibrated using real marketplace data.

## Evaluate the rationalizer itself

Create a labeled benchmark with:

- known duplicates
- known non-duplicates
- known partial overlaps
- known complementary artifacts

Measure:

- precision
- recall
- false positive rate
- false negative rate
- critic overturn rate
- human agreement rate

The ADLC feedback loop is:

```
Scan
  |
Rationalize
  |
Human review
  |
Ground truth
  |
Evaluate rationalizer
  |
Tune thresholds/prompts
  |
Next scan
```

## Human-in-the-loop

Create a review queue.

High-confidence relationships can receive a quick review.

Medium-confidence cases should receive deeper review.

Contradictory evidence should require human review.

High-risk permissions should trigger security review.

Possible human decisions:

- CONFIRMED_DUPLICATE
- NOT_DUPLICATE
- PARTIAL_OVERLAP
- COMPLEMENTARY
- KEEP_BOTH
- MERGE_APPROVED

These decisions can later be used as ground truth for evaluation and tuning.

## Long-term capability graph

Conceptually:

```
Business Capability
       |
       +-- implemented by --> Agent
       +-- implemented by --> Skill
       +-- uses --> Tool
       +-- accesses --> API
       +-- owned by --> Team
       +-- similar to --> Capability
       +-- overlaps --> Artifact
```

Do not procure a graph database for V1. Use relational records or in-memory structures.

## Integration with Agent Assurance Platform

This capability should sit alongside the existing certification lifecycle.

```
Agent Assurance Platform
|
+-- Certification
|     +-- Security
|     +-- Quality
|     +-- Tool Safety
|     +-- Data Leakage
|     +-- Performance
|
+-- Marketplace Rationalization
      +-- Discovery
      +-- Capability Mapping
      +-- Similarity
      +-- Duplication
      +-- Consolidation
      +-- Governance
```

Reuse:

- canonical artifact model
- marketplace adapters
- runtime adapters
- evidence model
- trace model
- policy engine
- reporting infrastructure

## Recommended repository addition

Add this specification under:

```
docs/
  Agentic Marketplace Capability Rationalization.md
```

Future code:

```
apps/
  marketplace_rationalization/

packages/
  marketplace_model/
  similarity_engine/
  evidence_model/
  capability_graph/

skills/
  marketplace_discovery/
  capability_analysis/
  marketplace_comparison/
  marketplace_critic/
  marketplace_consolidation/
  marketplace_reporting/
```

## MVP scope

The MVP must:

1. Scan a marketplace catalog.
2. Normalize plugins, agents and skills.
3. Extract capabilities.
4. Generate embeddings using an existing model/skill.
5. Hold embeddings in memory.
6. Calculate cosine similarity.
7. Produce candidate pairs.
8. Invoke a comparison agent.
9. Invoke a critic agent.
10. Cluster relationships.
11. Invoke a consolidation agent.
12. Generate Markdown and JSON reports.
13. Persist evidence and findings using existing platform persistence.
14. Provide a human review queue.

No vector DB.

No graph DB.

No new cloud-specific infrastructure.

## Example end-to-end run

```
START
 |
 +-- Scan marketplace
 |
 +-- Discover N artifacts
 |
 +-- Normalize metadata
 |
 +-- Extract capability models
 |
 +-- Generate N embeddings
 |
 +-- Group artifacts by domain/type/capability
 |
 +-- Calculate cosine similarity
 |
 +-- Produce candidate relationships
 |
 +-- Comparison Agent evaluates candidates
 |
 +-- Critic Agent challenges classifications
 |
 +-- Build candidate clusters
 |
 +-- Consolidation Agent produces proposals
 |
 +-- Report Agent generates output
 |
END
```

Counts are illustrative only.

## First implementation sequence

### Phase 1 — Marketplace Inventory

- marketplace adapter
- artifact model
- scanner
- normalizer
- inventory API

### Phase 2 — Capability Model

- capability extraction
- tool/API extraction
- permission extraction
- input/output extraction
- provenance

### Phase 3 — Similarity

- embedding adapter
- in-memory vector store
- cosine similarity
- deterministic candidate reduction
- configurable thresholds

### Phase 4 — Agentic Comparison

- comparison agent
- critic agent
- structured output schema
- evidence model

### Phase 5 — Cluster / Consolidation

- relationship graph in memory
- cluster generation
- consolidation agent

### Phase 6 — Reporting

- Markdown
- JSON
- HTML
- human review queue

### Phase 7 — Evaluation

- ground-truth benchmark
- precision/recall
- false-positive analysis
- prompt/threshold tuning

## Development rules for office VS Code / agentic development

1. Read the existing platform specification before changing the platform.
2. Reuse the existing canonical plugin model wherever possible.
3. Do not introduce a vector database for this capability.
4. Do not introduce a graph database for V1.
5. Keep semantic vectors ephemeral.
6. Keep deterministic calculations outside the LLM.
7. Use existing marketplace skills before creating new ones.
8. Treat marketplace artifacts as untrusted inputs.
9. Do not allow arbitrary side-effecting execution during static rationalization.
10. Use read-only marketplace metadata wherever possible.
11. Keep agent outputs schema-constrained.
12. Store evidence for every material conclusion.
13. Preserve artifact version and provenance.
14. Require critic review before classifying a pair as a duplicate.
15. Keep consolidation recommendations advisory.
16. Never automatically delete, retire or merge production marketplace artifacts in V1.
17. Add tests for false positives and false negatives.
18. Make the workflow resumable and bounded.
19. Reuse the existing Assurance Platform trace/evidence model.
20. Keep the implementation modular so the capability can later scale without changing its domain contracts.

## Definition of done

The MVP is complete when the platform can:

```
1. Discover marketplace artifacts
2. Normalize them
3. Understand capabilities
4. Generate in-memory embeddings
5. Calculate cosine similarity
6. Select candidate relationships
7. Agentically compare candidates
8. Critically challenge the comparison
9. Create rationalization clusters
10. Propose consolidation opportunities
11. Generate evidence-backed reports
12. Persist results using existing platform persistence
13. Produce a human review queue
```

A successful implementation demonstrates marketplace rationalization using **agent composition + deterministic similarity + evidence-driven evaluation**, without new vector or graph infrastructure.

## Long-term vision

The end-state is an **AI Marketplace Capability Governance Layer**:

```
Developer creates capability
             |
             v
       Marketplace Intake
             |
             v
        Assurance Scan
             |
             +-----------------------+
             |                       |
             v                       v
       Certification         Capability Mapping
                                     |
                                     v
                               Duplication Check
                                     |
                                     v
                              Consolidation Check
                                     |
                                     v
                              Marketplace Governance
```

This enables continuous answers to:

- What AI capabilities do we already have?
- Where are we duplicating capabilities?
- Which teams own equivalent capabilities?
- Which skills should be reused?
- Which agents are composing duplicate skills?
- Where should the marketplace converge on common capabilities?
- What should be retired or consolidated?
- What new submissions overlap with existing capabilities?

The long-term goal is to move the organization from an **artifact-centric marketplace** to a **capability-centric AI ecosystem**.

## Relationship to Agent Assurance & Certification

Certification answers:

> Is this capability safe and fit for use?

Rationalization answers:

> Do we already have this capability somewhere else?

Together:

```
                AI Marketplace
                      |
          +-----------+-----------+
          |                       |
          v                       v
   Assurance / Certification   Rationalization
          |                       |
          v                       v
      Is it safe?            Is it redundant?
          |                       |
          +-----------+-----------+
                      |
                      v
               Governance
```

This makes the overall platform a broader **AI supply-chain assurance and governance layer**, rather than only a certification tool.
