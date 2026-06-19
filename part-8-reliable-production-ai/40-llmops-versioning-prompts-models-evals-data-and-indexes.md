# Chapter 40: LLMOps: Versioning Prompts, Models, Evals, Data, and Indexes

Chapter 39 optimized runtime behavior. This chapter controls change.

LLMOps is not a fancy platform. It is discipline around versions, releases, regressions, and rollback.

## Real Problem

An assistant quality drops after release.

Possible changes:

- prompt changed;
- model changed;
- eval set changed;
- document corpus changed;
- embedding model changed;
- index rebuilt;
- tool schema changed.

No one knows which change caused failure.

## Bad Default Solution

Bad operations:

- edit prompt in dashboard;
- change model by environment variable;
- rebuild index without version;
- update eval examples without changelog;
- no rollback package.

This makes production behavior unexplainable.

## First Principles

Version every behavior-shaping asset:

- prompt;
- model;
- tool schema;
- eval set;
- data corpus;
- retrieval index;
- embedding model;
- routing rules;
- safety policy.

Release should identify exact combination.

## System Decision

Create release manifest:

```text
System version:
Prompt version:
Model version:
Tool schema version:
Eval version:
Corpus version:
Index version:
Embedding model:
Risk policy:
Release owner:
Rollback target:
```

## Minimal Build

Create `llmops-release-manifest.md`:

```text
Release:
Changed assets:
Eval results:
Known regressions:
Approval:
Deployment date:
Rollback:
```

## Failure Modes

LLMOps fails when:

- prompt changes are untracked;
- eval data changes during comparison;
- index version unknown;
- tool schema change breaks agent;
- rollback restores app code but not index;
- production trace lacks version IDs.

## Evaluation

Test:

- prompt rollback;
- model rollback;
- index rollback;
- eval rerun by version;
- trace points to manifest;
- old failure stays fixed.

## Proof Artifact

Create `llmops-release-manifest.md` and `versioning-policy.md`.

## Next

Next chapter closes production loop with deployment, rollback, feedback, and continuous improvement.
