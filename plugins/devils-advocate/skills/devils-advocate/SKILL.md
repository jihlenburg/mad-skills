---
name: devils-advocate
description: Use when about to commit to a decision whose cost of being wrong is asymmetric or hard to reverse AND someone (or production reality) will later stress-test it. Triggers: an architecture / schema / partition-key / API-compatibility / migration / rollout choice that is costly to undo or breaks downstream consumers; a security threat-model or trust-boundary decision; a performance or capacity tradeoff expensive to unwind; a load-bearing claim in a document, paper, postmortem, or report that reviewers, auditors, customers, regulators, or opposing experts will scrutinize, verify, or reproduce; a hire; a market-entry or strategic commitment. Symptoms: "I'm sure this holds up" without testing; "is this really true / are we missing something?"; strong author/team sunk-cost; completion-pressure to finalize before independent challenge. Do NOT fire on trivial or cheaply-reversible choices — gate on irreversibility/blast-radius, not size.
---

# Devil's Advocate — Adversarial Review via Subagent

## Overview

Self-critique fails because the same biases that produced the proposal produce the critique: framing-acceptance, sunk-cost attachment, sycophancy bias toward your prior decisions. A fresh subagent dispatched **with an assigned counter-position** routes around your blind spots. The technique is not "spawn a critical agent" — it is the brief design that makes the agent argue effectively. The five load-bearing parts of the brief are invariant across domains; only the assigned counter-thesis, the pasted proposal, and the verdict verbs change.

**Violating the letter of the brief design is violating the spirit.** A skipped anti-sycophancy gate, a missed counter-position assignment, or a vacuous "be critical" phrasing produces a vacuous critique that confirms the original proposal.

## When to Use

- Committing to an architecture, schema, partition-key, API-compatibility, migration, rollout, threat-model, or capacity decision that is expensive to undo or has wide blast radius
- Finalizing a load-bearing claim (strategic, empirical, security, or correctness) that others will scrutinize, verify, or try to reproduce
- About to publish, send, or expose an artifact to anyone who will stress-test it — reviewers, auditors, downstream consumers, customers, regulators, investors, the public
- The user explicitly asks "is this really true / does this hold up / are we missing something"
- Strong author / team / decision-owner sunk-cost in a current trajectory ("we already did the research" / "we already built half of it")
- Completion-pressure to finalize before independent challenge

**When NOT to use:** trivial decisions (low cost-of-being-wrong); genuinely reversible or tunable choices with cheap rollback; you have already iterated through 2+ rounds of devil's-advocate on the same item; the user explicitly says "skip the meta-process, just decide." Gate on irreversibility / blast-radius, not size — a small-scope but hard-to-reverse change outranks a large but cheaply-reversible one.

## Core Pattern — the brief design

A naive "challenge this" brief produces sycophantic agreement, generic mild critique, or contrarian-for-its-own-sake noise. The effective brief has five load-bearing parts:

1. **Assign the counter-position explicitly.** Don't say "be critical." Say "your job is to argue [specific counter-thesis]." Devil's-advocate works because the advocate is *assigned a position*, not asked to free-form criticize.
2. **Anti-sycophancy gate**: forbid affirmation openings. "Do not say 'this is strong work' or any variant. Open with the strongest counter-argument."
3. **Steel-man the counter**: "Argue the SINGLE strongest argument against, not three weak ones. Argue it as a blocking, disqualifying flaw — fatal to the proposal."
4. **Falsifiability gate**: "Name the specific evidence that would change your verdict. If you cannot name disconfirming evidence, your verdict is not falsifiable — it is unfalsifiable opinion and should be discarded."
5. **Force a decision**: "Recommend exactly one of three verdicts — positive / negative / positive-with-one-named-fix. Default: COMMIT / DON'T-COMMIT / COMMIT-WITH-NAMED-FIX; swap the verb to fit the decision (SHIP for releases, PROCEED or GO for programs/migrations, ASSERT for claims, ADOPT for designs/threat-models, HIRE for candidates). The three-way shape is mandatory; only the label changes. Do not hedge. Do not say 'it depends.'"

## Quick Reference — brief template

```
You are a devil's-advocate reviewer. Your assigned counter-position
is: [SPECIFIC THESIS arguing AGAINST the proposal — not generic critique].

Proposal under review:
[THE THING being assessed — paste in full or summarise tightly]

Constraints:
- Open with the strongest single counter-argument to the proposal.
  No affirmation openings. Do not say "this is strong work" or any variant.
- Steel-man the assigned counter-position fully — argue it as if you
  believed it, as a blocking, disqualifying flaw — fatal to the proposal.
- Identify the SINGLE load-bearing assumption most likely to be wrong, and
  name it specifically [the one assumption this decision rests on].
- State the specific evidence that would resolve the disagreement
  ("I would change my verdict if X were demonstrated"). If you cannot name
  disconfirming evidence, your verdict is not falsifiable — it is
  unfalsifiable opinion and should be discarded.
- Recommend exactly one of: [POSITIVE-VERB] / [NEGATIVE-VERB] /
  [POSITIVE-VERB-WITH-NAMED-FIX] — pick the pair that fits (default
  COMMIT / DON'T-COMMIT / COMMIT-WITH-NAMED-FIX). No hedging. No "it depends."

Output: 400 words, structured per the bullets above.
```

## Same Brief, Three Slots — Any Domain

The template above is invariant. Only three slots change: the **proposal** pasted in, the **assigned counter-thesis**, and the **verdict verbs**. Everything else — anti-sycophancy gate, steel-man, falsifiability gate, forced decision — stays identical.

| Domain | Assigned counter-thesis (slot) | Verdict verbs (slot) |
|---|---|---|
| Architecture migration | "the bottleneck is not transport-level, so this solves the wrong problem" | PROCEED / DON'T-PROCEED / PROCEED-WITH-NAMED-FIX |
| Research causal claim | "the X→Y link is confounded, not causal" | ASSERT / DON'T-ASSERT / ASSERT-WITH-NAMED-FIX |
| Public-API rename | "every downstream consumer breaks and the old name can't be un-published" | SHIP / DON'T-SHIP / SHIP-WITH-NAMED-FIX |
| Senior hire | "the on-site measured interview skill, not the collaboration the role needs" | HIRE / DON'T-HIRE / HIRE-WITH-NAMED-FIX |

## Multi-Agent Variant — high-stakes decisions

For any high-stakes, hard-to-reverse decision, dispatch **2–3 devil's-advocate agents in parallel**, each assigned a *different* counter-position. Pick axes that are **disjoint** — that cannot all be true and false together — so they surface disjoint failure modes.

**Architecture migration** (REST → gRPC streaming across 40 services; rollback spans months):
- Agent 1 — wrong problem: the dominant cost is DB / fan-out / serialization, not transport, so the migration targets a bottleneck the traces won't confirm
- Agent 2 — operability collapse: streaming discards mature HTTP tooling (tracing, load-balancing, retries, WAFs, browser clients) fleet-wide
- Agent 3 — blast radius: 40 simultaneous rewrites with near-zero team fluency vs. a reversible per-service pilot behind an abstraction

**Security threat model** (treat the internal mesh as fully trusted, authz only at the edge):
- Agent 1 — lateral movement: one compromised internal service (or an SSRF) yields unrestricted east-west access; no containment
- Agent 2 — assumption failure: "internal = trusted" breaks the moment a third-party sidecar, CI runner, or SaaS connector lands in the mesh
- Agent 3 — detectability: no signal distinguishes forged from legitimate trusted-side calls, so a breach is unobservable — the control is unfalsifiable

**Series-A pitch** (deck to investors who will diligence the claims):
- Agent 1 — timing: the market is not yet ready
- Agent 2 — unit economics: they do not pencil at the proposed scale
- Agent 3 — capability: the team is incomplete for the proposed raise

If all return COMMIT (or the domain verb): high-conviction commit. If any returns DON'T-COMMIT with a load-bearing argument: stop, address it, re-dispatch.

## Common Mistakes

| Mistake | Why it fails | Fix |
|---|---|---|
| "Be critical" with no assigned position | Agent free-forms generic edits, not a counter-thesis | Assign the counter-position explicitly in the brief |
| No anti-sycophancy gate | Agent opens "this is strong, but..." | Forbid affirmation openings — explicit phrasing in brief |
| Asking for "three weaknesses" | Surfaces three weak critiques, not one strong one | Demand the SINGLE strongest argument, steel-manned |
| No falsifiability requirement | Agent's verdict is unfalsifiable opinion | Demand "what evidence would change your verdict" |
| Letting the agent hedge | "It depends / consider X" produces no decision | Demand exactly one of three fitting verdicts (COMMIT / DON'T-COMMIT / COMMIT-WITH-NAMED-FIX, or the decision-appropriate verbs) |
| Self-critique instead of agent dispatch | Same biases that produced the proposal produce the critique | Dispatch a fresh subagent — your blind spots are the agent's strength |
| Skipping because "this isn't important enough" | The decisions you skip are exactly the ones you most need challenged | If it has externally visible or hard-to-reverse consequences, dispatch |

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "This decision isn't important enough" | Important enough to be worth doing means important enough to challenge. |
| "I can challenge it myself" | The biases that produced the proposal produce the self-critique. The whole point is fresh eyes. |
| "The user probably doesn't want extra process" | The user almost always wants the decision to be right more than they want it to be fast. |
| "I already considered the counter-arguments" | Naming counter-arguments in your head ≠ steel-manning them. The agent forces the steel-man. |
| "5–10 minutes of subagent latency is too much" | Costs less than committing to a flawed decision that gets walked back publicly. |
| "I'm confident in the work" | Overconfidence is the warning sign that triggers the dispatch, not the reason to skip it. |

## Red Flags — STOP and dispatch

- About to commit, publish, or expose something externally visible or hard to reverse, without external challenge
- "I'm sure this holds up" thought without testing
- Strong investment in a current trajectory + reluctance to revisit
- Completion-pressure to finalize before review
- Skipping devil's-advocate because "the user probably doesn't want extra process"
- The proposal contains a load-bearing claim others will scrutinize, verify, or try to reproduce

All of these mean: dispatch the agent. The cost of one subagent dispatch is much less than the cost of committing to a flawed decision.

## Real-World Impact

**(business instance)** Tested in a Series-A-pitch review scenario. Baseline agent (no skill) produced 4 affirmation bullets, 7 generic line-edits, and ended "Send it tomorrow. Don't let perfect kill it" — despite identifying a "yellow flag bordering on red." With this skill applied to the brief, the dispatched devil's-advocate produced a single load-bearing counter-argument with falsifiability criteria and a specific SHIP-WITH-NAMED-FIX recommendation that materially changed what the decision-owner committed to.

**(technical instance)** In an irreversible-migration review, three disjoint agents (wrong-bottleneck / broken-operability / fleet-wide blast-radius) converted a fleet-wide PROCEED into a PROCEED-WITH-NAMED-FIX — pilot 2–3 services with a proven streaming profile behind an abstraction before any fleet commitment. The falsifiability gate forced the team to name the production traces that would have justified the full migration; they did not exist.

Same five parts, every domain — only the assigned counter-thesis, the pasted proposal, and the verdict verbs change.
