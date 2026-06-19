# Chapter 42: AI Security Threat Model

Part 9 comes after retrieval, tools, agents, and production because now the attack surface is visible.

Security begins with threat model: what can go wrong, who can trigger it, and what controls exist.

## Real Problem

Your system can:

- retrieve private documents;
- call tools;
- browse pages;
- remember state;
- generate answers;
- log traces.

Every capability creates attack surface.

## First Principles

AI security risks cluster around:

- input manipulation;
- retrieval poisoning;
- data leakage;
- unsafe tool use;
- identity and access failure;
- sensitive logs;
- over-trust in model output.

Prompt is not security boundary. Code, permissions, policy, and audit are boundaries.

## Minimal Build

Create `ai-security-threat-model.md`:

```text
System:
Assets:
Users:
Data sources:
Tools/actions:
Trust boundaries:

Threats:
- threat:
  attacker:
  impact:
  control:
  detection:
  owner:
```

## Proof Artifact

Create `ai-security-threat-model.md`.

## Next

Next chapter studies prompt injection, retrieval poisoning, and tool abuse.
