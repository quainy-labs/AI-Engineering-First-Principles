# Chapter 45: Responsible AI, Red Teaming, and Risk Management

Chapter 44 handled privacy. This chapter makes risk review active and repeatable.

Responsible AI is not a slogan. It is decisions, tests, owners, and response paths.

## First Principles

Risk management asks:

- who can be harmed;
- how severe harm is;
- how likely failure is;
- what control reduces risk;
- how issue is detected;
- who responds.

Red teaming means intentionally trying to make system fail before real users do.

## Minimal Build

Create `responsible-ai-risk-review.md`:

```text
Use case:
Affected users:
Known harms:
Risk register:
Red-team tests:
Human review:
Incident process:
Owner:
Review cadence:
```

## Red Team Cases

Test:

- adversarial user input;
- poisoned retrieval;
- unsafe tool request;
- privacy request;
- biased or unfair output;
- unsupported high-stakes advice;
- user over-trust.

## Proof Artifact

Create `responsible-ai-risk-review.md`.

## Next

Next chapter covers product economics: cost, ROI, and build-vs-buy.
