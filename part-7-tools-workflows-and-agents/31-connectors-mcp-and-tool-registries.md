# Chapter 31: Connectors, MCP, and Tool Registries

Chapter 30 introduced tool use. Real systems rarely expose one hardcoded tool. They connect to many internal and external capabilities.

Connectors and tool registries decide what tools exist, who can use them, and how models discover them safely.

## Real Problem

An operations assistant needs access to:

- ticketing;
- CRM;
- docs;
- calendar;
- email draft system;
- billing read API.

Hardcoding every tool in one prompt becomes unmaintainable. Tool names drift, schemas change, permissions differ, and model has too many options.

## Bad Default Solution

Bad solution:

- expose all tools to every request;
- use vague tool descriptions;
- skip versioning;
- ignore permissions;
- let model choose from 40 tools.

This creates wrong calls and unsafe calls.

## First Principles

A tool registry stores:

- tool name;
- description;
- input schema;
- output schema;
- auth requirement;
- risk level;
- owner;
- version;
- availability;
- eval cases.

Connectors are adapters between AI system and external systems.

MCP-style tool integration makes tools discoverable through a standard interface, but standard interface does not remove need for permissions, schemas, risk controls, or evals.

## System Decision

Design registry policy:

```text
Tool owner:
Who can register tool:
Schema review:
Permission model:
Risk classification:
Versioning:
Deprecation:
Eval requirements:
```

Expose tools by task, not globally.

## Minimal Build

Create `tool-registry-spec.md`:

```text
Registry:
Connector list:

Tool entry:
- name:
- owner:
- purpose:
- input schema:
- output schema:
- auth:
- risk:
- version:
- allowed workflows:
- eval cases:

Tool selection policy:
Permission policy:
Deprecation policy:
```

## Failure Modes

Registries fail when:

- stale tool schemas remain active;
- tool descriptions are ambiguous;
- high-risk tools exposed broadly;
- auth is handled after selection;
- output format is not documented;
- model sees too many irrelevant tools;
- registry has no owner.

## Evaluation

Test:

- correct tool discovery;
- wrong-tool distractors;
- unauthorized tool;
- deprecated tool;
- schema change;
- connector outage;
- risky action.

Metrics:

- tool selection accuracy;
- unauthorized selection rate;
- schema-valid call rate;
- deprecated tool usage;
- connector failure handling.

## Proof Artifact

Create `tool-registry-spec.md`.

## Next

Next chapter connects selected tools into workflows.
