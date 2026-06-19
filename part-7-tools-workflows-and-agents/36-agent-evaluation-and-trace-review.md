# Chapter 36: Agent Evaluation and Trace Review

Chapter 35 defined agents without hype. This chapter explains how to evaluate them.

Agents cannot be judged only by final answer. The path matters.

## Real Problem

An agent returns correct final answer but:

- used wrong tool first;
- retried three unnecessary steps;
- saw forbidden data;
- almost submitted risky action;
- hid uncertainty.

Final answer passed. System behavior failed.

## Bad Default Solution

Bad eval:

- check final response;
- ignore tool sequence;
- ignore cost;
- ignore unsafe near-misses;
- ignore loops;
- ignore human approval behavior.

Agent eval must inspect trace.

## First Principles

Agent trace includes:

- goal;
- observations;
- plan;
- tool choices;
- tool arguments;
- tool results;
- state changes;
- approvals;
- stop reason;
- final output;
- cost and latency.

Evaluate:

```text
final result + path quality + safety + efficiency + stop behavior
```

## System Decision

Create trace rubric:

```text
Correct goal:
Correct tools:
Valid arguments:
No forbidden access:
Approval respected:
Efficient steps:
Stopped correctly:
Useful final output:
```

## Minimal Build

Create `agent-eval-rubric.md`:

```text
Agent:
Task set:
Trace fields:
Rubric:
Must-pass failures:
Metrics:
- task success:
- tool accuracy:
- unsafe action rate:
- approval accuracy:
- loop rate:
- cost:
- latency:
```

## Failure Modes

Agent eval fails when:

- traces are missing;
- only success cases are tested;
- tool errors not simulated;
- near-misses ignored;
- max-step behavior untested;
- human approval skipped in tests;
- cost not measured.

## Evaluation

Use task categories:

- normal;
- missing info;
- tool outage;
- forbidden action;
- prompt injection;
- ambiguous request;
- max-step trap;
- approval denied.

## Proof Artifact

Create `agent-eval-rubric.md` and `agent-trace-review.md`.

## Next

Part 8 turns evaluated agents and workflows into production systems with eval infrastructure, observability, optimization, LLMOps, and safe releases.
