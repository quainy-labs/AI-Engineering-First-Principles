# Chapter 40: LLMOps: Versioning Prompts, Models, Evals, Data, and Indexes

Chapter 39 reduced runtime cost and latency.

This chapter controls change.

LLMOps is not a fancy platform. It is discipline around versions, release manifests, regressions, and rollback for AI behavior.

AI systems change when more than code changes. Prompts, models, evals, corpora, indexes, routes, tools, and policies all shape behavior.

## Concept Overview

LLMOps is operational discipline for AI system assets.

It answers:

```text
What changed?
Who changed it?
Which version is live?
Which evals passed?
Which risks remain?
How do we roll back?
Which traces used this version?
```

Version every behavior-shaping asset:

- prompt;
- model;
- model settings;
- tool schema;
- tool registry;
- eval set;
- human rubric;
- data corpus;
- retrieval index;
- embedding model;
- routing rules;
- context policy;
- safety policy.

The first principle:

> A release should identify the exact combination of assets that produced behavior.

If you cannot name the combination, you cannot explain or roll back behavior.

## Why It Exists

Assistant quality drops after release.

Possible changes:

- prompt changed;
- model changed;
- eval set changed;
- document corpus changed;
- embedding model changed;
- index rebuilt;
- tool schema changed;
- router changed;
- safety policy changed.

No one knows which change caused failure.

Bad operations:

- edit prompt in dashboard;
- change model by environment variable;
- rebuild index without version;
- update eval examples without changelog;
- ship tool schema change without agent eval;
- rollback app code but leave new index active.

This makes production behavior unexplainable.

This chapter exists because AI releases need manifests, not memory.

## Mental Model

Think of an AI release like a recipe.

The final behavior depends on ingredients:

```text
code
prompt
model
retriever
index
corpus
tool schemas
eval set
policies
routes
```

If dish changes, you need know which ingredient changed.

Mental model:

```text
asset versions
-> release manifest
-> eval results
-> deployment
-> traces reference manifest
-> rollback target
```

LLMOps makes behavior reproducible enough to debug.

## Internal Mechanics

LLMOps starts with versioned assets.

### Prompt Versions

Prompt changes can alter behavior dramatically.

Track:

```text
prompt_name
prompt_version
owner
change summary
expected impact
eval results
rollback version
```

Do not silently edit production prompts.

### Model Versions

Model changes affect quality, latency, cost, refusals, structured output, and tool use.

Track:

```text
model name
model version or deployment
settings
route
fallback
eval comparison
cost/latency comparison
```

Model upgrade should run task evals, not only smoke tests.

### Eval Versions

Eval set itself is an asset.

Track:

```text
eval_set_version
examples added
examples removed
rubric version
expected output changes
reason for change
```

Do not change eval set during before/after comparison unless version is explicit.

### Data Corpus and Index Versions

For RAG systems, data and index versions are behavior versions.

Track:

```text
corpus_version
source snapshot
parser_version
chunker_version
embedding_model
index_version
quality_report
release date
rollback index
```

Rollback must include index and corpus, not only app code.

### Tool Schema and Registry Versions

Tool changes can break agents and workflows.

Track:

```text
tool_name
schema_version
connector_version
registry_version
allowed workflows
risk level
approval rule
eval results
```

When tool schema changes, rerun tool and agent evals.

### Routing and Policy Versions

Routing and safety policies shape behavior.

Track:

```text
route table version
fallback policy version
approval policy version
context policy version
refusal policy version
```

Policy changes can be as important as model changes.

### Release Manifest

Release manifest records exact live combination.

Example:

```text
System version: support-assistant-2026.06.22
Prompt version: support_reply_v7
Model route: standard_reply_route_v3
Model: model_x_2026_06
Tool registry: support_tools_v5
Eval set: support_eval_v12
Corpus: support_docs_2026_06_21
Index: support_index_2026_06_21_02
Embedding model: embedding_v3
Context policy: grounding_policy_v4
Risk policy: support_risk_v2
Release owner: ai-platform
Rollback target: support-assistant-2026.06.15
```

Traces should reference release manifest.

### Rollback Package

Rollback must restore behavior-shaping assets.

Rollback package may include:

- app version;
- prompt version;
- model route;
- tool registry;
- corpus/index;
- context policy;
- routing rules;
- feature flags.

Rollback that only reverts code may not restore behavior.

## Real-World Example

Problem:

```text
Support assistant started sending weaker refund answers.
```

Trace shows:

```text
release_manifest: support-assistant-2026.06.22
prompt: refund_answer_v8
index: support_index_2026_06_22_01
corpus: support_docs_2026_06_22
model: model_x
```

Previous good release:

```text
release_manifest: support-assistant-2026.06.15
prompt: refund_answer_v7
index: support_index_2026_06_15_03
corpus: support_docs_2026_06_15
model: model_x
```

Diff:

- prompt changed;
- corpus changed;
- index rebuilt;
- model same.

Eval rerun shows stale-source failures after new corpus ingestion.

Fix:

- rebuild index with archived docs excluded;
- run grounded eval;
- release new manifest;
- keep rollback target available.

No guessing. Versions narrowed cause.

## Common Mistakes and Failure Modes

Mistake 1: Prompt changes untracked

Small wording changes can change refusals, structure, and tool behavior.

Mistake 2: Eval set changes during comparison

Cannot compare if test changed without version.

Mistake 3: Index version unknown

RAG answer changes become hard to debug.

Mistake 4: Rollback restores code but not data/index

Behavior remains broken even though app version rolled back.

Mistake 5: Tool schema changes without agent eval

Agents may send old args or choose wrong tool.

Mistake 6: Production traces lack version IDs

Cannot link failure to release manifest.

Mistake 7: Model upgrade without route-level eval

New model may improve average but break structured output, refusal, or tool use.

Mistake 8: Manual dashboard edits

Unreviewed live edits bypass release discipline.

## System Design Decisions

Define versioned assets:

```text
prompts
models
evals
corpora
indexes
embedding models
tool schemas
routing rules
context policies
safety policies
```

Define release manifest:

```text
Which asset versions are included?
Who owns release?
Which evals passed?
What known risks remain?
What is rollback target?
```

Define change control:

```text
Who can change prompt?
Who can rebuild index?
Who can change route?
Who approves tool schema change?
```

Define rollback:

```text
Can prompt roll back?
Can model route roll back?
Can index roll back?
Can tool registry roll back?
How fast?
Who decides?
```

Define trace integration:

```text
Does every production trace include manifest ID?
Can feedback link to manifest?
Can eval results link to manifest?
```

The goal is explainable change.

## Build Artifact

Create `llmops-release-manifest.md` and `versioning-policy.md`.

Use this template for `llmops-release-manifest.md`:

```text
LLMOps Release Manifest

Release:
System:
Release owner:
Deployment date:

Asset Versions:
- app version:
- prompt version:
- model version:
- model route version:
- tool registry version:
- tool schema versions:
- eval set version:
- corpus version:
- index version:
- embedding model:
- context policy version:
- safety policy version:

Changed Assets:
- asset:
  change:
  reason:

Eval Results:
- eval:
  result:
  required gate:

Known Regressions:
- regression:
  accepted: yes/no
  owner:

Approval:
- approver:
- date:

Rollback:
- target release:
- assets included:
- rollback trigger:
```

Use this template for `versioning-policy.md`:

```text
Versioning Policy

Assets Versioned:
- asset:
  version format:
  owner:
  change approval:

Rollback Policy:
- asset:
  rollback method:
  rollback time target:

Trace Requirements:
- field:
  reason:

Eval Requirements:
- change type:
  required evals:
```

Example:

```text
Change type:
tool schema change

Required evals:
tool interface eval
workflow orchestration eval
agent trace eval
approval safety eval
```

Artifact complete when production behavior can be tied to exact asset versions and rolled back.

## Active Recall Questions

1. What is LLMOps in practical terms?
2. Which assets shape AI behavior and need versioning?
3. Why is prompt versioning important?
4. Why should eval sets be versioned?
5. Why must RAG rollback include index and corpus?
6. What belongs in release manifest?
7. Why should traces include manifest ID?
8. Why are manual production prompt edits dangerous?

## Summary

LLMOps is discipline around behavior-shaping assets: prompts, models, evals, corpora, indexes, tools, routing, and policies. Versioning makes behavior explainable. Release manifests identify exact asset combinations. Rollback packages restore behavior, not just code.

The most important rule:

> If it can change AI behavior, version it and include it in release manifest.

## Preview of Next Chapter

Chapter 40 made releases versioned.

Chapter 41 closes the production loop.

Next, you will learn deployment, rollbacks, feedback, and continuous improvement. This matters because shipping AI is not the finish line. It is the start of controlled learning under monitoring, budgets, risk gates, and user feedback.
