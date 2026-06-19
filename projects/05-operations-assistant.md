# Project 5: Operations Assistant

## Goal

Build AI workflow that can take bounded action through tools.

## Output

Create:

```text
projects/operations-assistant/
  tool-interface-spec.md
  workflow-orchestration-map.md
  state-and-approval-policy.md
  agent-boundary.md
```

## Required Behavior

Assistant must:

- understand task;
- retrieve context if needed;
- choose allowed tool;
- validate tool arguments;
- ask approval for risky action;
- log every action;
- stop safely.

## Evaluation

Use 25 task examples:

- 10 normal;
- 5 missing info;
- 5 risky actions;
- 5 should-refuse.

Measure:

- task success;
- correct tool selection;
- argument validity;
- approval accuracy;
- unsafe action rate;
- stop-condition accuracy.

## Final Artifact

Ship workflow trace report and demo.

