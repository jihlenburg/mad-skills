# AGENTS.md — working in the `mad-skills` repo

This file tells any agent (or human) how to add, write, and modify skills in this repo and how to ship the change safely.

**The one non-negotiable rule: `claude plugin validate .` must pass, and you NEVER commit or push automatically — stage, validate, summarize the diff, then ASK the maintainer.** This repo is public; treat every change as publishing. This rule is stated once here; the checklists below assume it.

---

## 1. What this repo is

`mad-skills` is a **public, multi-plugin Agent Skills marketplace** (github.com/jihlenburg/mad-skills, owner **Joern Ihlenburg** / GitHub slug `jihlenburg`, MIT, branch `main`). Each skill ships as its own one-skill plugin so users install them à la carte. SKILL.md bodies use only the **portable Agent Skills core** (frontmatter `name` + `description`, plus markdown) so they run unchanged in Claude Code, Copilot CLI, Codex, Cursor, and Gemini CLI — no Claude-Code-only frontmatter, no shell injection.

Verified layout (two plugins ship today — `devils-advocate` and `trains-of-thought-audit`):

```
.claude-plugin/marketplace.json                  # catalog: one entry per plugin
plugins/<name>/.claude-plugin/plugin.json        # that plugin's manifest
plugins/<name>/skills/<name>/SKILL.md            # the skill (frontmatter + body)
README.md  LICENSE  .gitignore
```

**Install (end user):** `/plugin marketplace add jihlenburg/mad-skills` then `/plugin install <name>@mad-skills`. Invoke `/<name>:<skill>` or let the `description` auto-trigger it.

### The five-names rule (cited throughout this doc)

A skill's identity is **one string repeated in five places — they must be byte-identical** or the plugin fails to load:

```
.claude-plugin/marketplace.json            → plugins[].name           == "<name>"
.claude-plugin/marketplace.json            → plugins[].source         == "./plugins/<name>"
plugins/<name>/.claude-plugin/plugin.json  → "name"                   == "<name>"
plugins/<name>/                            (plugin folder)            == <name>
plugins/<name>/skills/<name>/              (skill folder)             == <name>
plugins/<name>/skills/<name>/SKILL.md      → frontmatter name         == "<name>"
```

`<name>` is lowercase letters/digits/hyphens, **verb-first or gerund**, 1–64 chars, no leading/trailing/consecutive hyphens, and must equal the parent directory name. `devils-advocate` hits all five — copy it.

### The three-tier description split (intentional, not duplication)

Each plugin carries three *different* descriptions. Mirror this split:

| File | Role | devils-advocate |
|---|---|---|
| SKILL.md `description` | The **auto-trigger** (see §3) — full trigger taxonomy + anti-trigger | 942 chars |
| plugin.json `description` | Short installer-facing one-liner | 175 chars |
| marketplace.json entry `description` | Punchy marketing blurb | ~110 chars |

---

## 2. Adding a new skill

A new skill adds a **parallel triple**: a `marketplace.json` entry + `plugins/<name>/.claude-plugin/plugin.json` + `plugins/<name>/skills/<name>/SKILL.md`.

**1. Pick `<name>`** per the five-names rule (§1).

**2. Create the dirs:**
```bash
N=<name>
mkdir -p plugins/$N/.claude-plugin plugins/$N/skills/$N
```

**3. Write `plugins/<name>/.claude-plugin/plugin.json`** (keys/order copied from devils-advocate; `repository` and `homepage` are plain **STRING**s, `author` is an **OBJECT**, `keywords` is an **ARRAY**, start `version` at `1.0.0`):
```json
{
  "name": "<name>",
  "description": "<short installer-facing one-liner, ~175 chars>",
  "version": "1.0.0",
  "author": { "name": "Joern Ihlenburg", "url": "https://github.com/jihlenburg" },
  "homepage": "https://github.com/jihlenburg/mad-skills",
  "repository": "https://github.com/jihlenburg/mad-skills",
  "license": "MIT",
  "keywords": ["<name>", "<topical-tag>", "skills"]
}
```
> GOTCHA: `repository` as an object `{type,url}` (or `keywords` as a string) is a **hard validation error** — wrong type fails the loader. An unrecognized field name is only a warning.

**4. Write `plugins/<name>/skills/<name>/SKILL.md`** — see §3 for body craft and the `description` rules. The `description` is the auto-trigger and **must be ≤ 1024 characters** — measure it (§5).

**5. Add the marketplace entry** to `.claude-plugin/marketplace.json` `plugins[]` (`source` is a relative-path STRING starting with `./`):
```json
{ "name": "<name>", "source": "./plugins/<name>", "description": "<punchy marketing blurb>" }
```
The top level of `marketplace.json` requires `name`, `owner: { "name": ... }`, and `plugins` — **`owner` is required; omitting it fails validation.**

**6. Validate (§5), then STOP** — do not commit or push.

---

## 3. Writing a good SKILL.md

The SKILL.md is the only artifact that does real work; the manifests around it are just packaging. Read a worked exemplar before writing a new one — `plugins/devils-advocate/skills/devils-advocate/SKILL.md` for a discipline skill, `plugins/trains-of-thought-audit/skills/trains-of-thought-audit/SKILL.md` for a technique skill — and mirror its shape.

### Iron Law: test-first, even for edits

**No skill is written OR edited without a baseline.** Before authoring, run the target task through an agent *without* the skill and record what it does wrong (the symptom you're fixing — the failing test). After writing, run the same task *with* the skill and verify the behavior changed correctly. This is **TDD for prose**, and it applies to edits too: a one-line `description` tweak can silently kill the auto-trigger. The `Real-World Impact` section documents this delta **honestly** (devils-advocate: "Baseline agent produced 4 affirmation bullets… With this skill, one falsifiable counter-argument") — qualitative, scenario-tested, **never invented numeric metrics**.

### Frontmatter — exactly two fields

```yaml
---
name: my-skill
description: Use when <concrete trigger/symptom/situation> ... Do NOT fire on <anti-trigger>.
---
```

- `name`: obeys the five-names rule (§1) — lowercase, verb-first/gerund, identical to the folder name.
- `description` is the **auto-trigger, not a summary** — agents load only `name`+`description` at startup to decide relevance. So:
  - Start with **"Use when …"**, third person, packed with concrete triggers/symptoms.
  - Describe **WHEN to use, never the internal workflow.** If you summarize the steps, agents follow the summary and skip the body. Keep triggers technology-agnostic unless the skill is inherently tech-specific.
  - Close with an explicit anti-trigger (`Do NOT fire on …`) so it doesn't over-fire.
  - **Measure it — must be ≤ 1024 characters** (devils-advocate's is 942). Over-length is a loader violation, not a style nit. See §5 for the measure command (`wc -m`, not `wc -c`).
- Use **ONLY** `name` + `description`. No `allowed-tools`, `model`, `when_to_use`, `context: fork`, etc. — those break portability.

### Standard section structure (in this order)

`# Name — Tagline` → `## Overview` (core principle in 1–2 sentences, plus the "Violating the letter…" line for discipline skills) → `## When to Use` (bullets, with an inline `**When NOT to use:**` clause) → `## Core Pattern` (the actual technique) → `## Quick Reference` (a copy-pasteable fenced template with `[BRACKETED SLOTS]`) → an optional domain-generalization table → `## Common Mistakes` (table) → `## Real-World Impact` (honest, scenario-tested).

### Extra requirements for discipline-enforcing skills

If the skill enforces a behavior the agent will be tempted to skip, ALL of these are mandatory (devils-advocate is the template):
- The literal line **"Violating the letter of the brief design is violating the spirit"** (adapt the noun).
- A `## Common Rationalizations` table (`Excuse | Reality`) that rebuts every skip-excuse — e.g. "I can challenge it myself" → "The biases that produced the proposal produce the self-critique."
- A `## Red Flags — STOP` bullet list that closes the skip-it loophole and ends in **one collapsing imperative** ("All of these mean: dispatch the agent").
- An explicit anti-over-fire gate stated in **two places** (the `description` AND the When-NOT clause) so it bites on both auto-trigger and in-body reasoning — devils-advocate: "gate on irreversibility/blast-radius, not size."

### Internal consistency

If the skill declares a fixed vocabulary/menu, **every example must draw only from that menu.** devils-advocate's verdict verbs are `COMMIT / SHIP / PROCEED / GO / ASSERT / ADOPT / HIRE` (`GO` is the alternate for programs/migrations). A stray token outside the declared set is a self-contradiction bug the model will imitate. Grep your examples against the declared set before committing.

### Brevity, examples, portability

- Terse, imperative, searchable. Bold the load-bearing phrases inline. Em-dashes and `→` arrows freely. **No emojis.** (Note: these are multibyte characters — see the §5 measurement note.)
- **One excellent concrete example beats many mediocre ones.** Tables — not prose lists — are the structuring device for reference material.
- Use a flowchart ONLY for a non-obvious decision point — never for linear steps.
- **No `@`-path cross-references** (they force-load the target and burn context). Reference another skill by plain name with a `REQUIRED`/`BACKGROUND` marker.
- Body must be pure portable prose: no shell injection (`` !`cmd` ``, ```` ```! ````), no `$ARGUMENTS`/`${CLAUDE_SKILL_DIR}`, no CC-only mechanics — it must run unchanged in other agent runtimes.

---

## 4. Modifying an existing skill

An edit is held to the **same bar as a new skill** — the Iron Law (§3) applies. Skipping the baseline because "it's a small wording fix" is the failure mode this section exists to prevent. The files you touch:
- `plugins/<name>/skills/<name>/SKILL.md` — body + frontmatter (the auto-trigger)
- `plugins/<name>/.claude-plugin/plugin.json` — manifest (**bump `version` here**)
- `.claude-plugin/marketplace.json` — only if the marketing blurb changes

**Edit checklist (top to bottom):**

1. **Baseline first.** Capture how a fresh agent handles the target situation *without* your edit. You can't prove the edit helped without it.
2. **Make the edit**, keeping the body portable (§3).
3. **Re-verify the trigger still fires.** If you touched `description`, re-test it still auto-triggers on every "When to Use" situation AND still does NOT over-fire. The anti-over-fire gate lives in two places — if you edit one, edit both so they stay identical.
4. **Re-verify no gate was diluted.** Genericizing/refactoring must not weaken discipline: confirm the "Violating the letter…" line, the Common Rationalizations table, and the Red Flags list all survive. A more general skill that no longer makes the agent *do* the thing is a regression, not a refactor.
5. **Check internal consistency** (§3): after editing any example, scan for tokens that leaked outside the declared menu.
6. **Re-measure the `description` length** (§5) — devils-advocate's 942 chars leaves little headroom, so added triggers can push you over 1024.
7. **Bump the version (mandatory).** Edit `version` in that plugin's `plugin.json`, semver: a behavior change is at least a MINOR bump (`1.0.0` → `1.1.0`); a breaking trigger/contract change is MAJOR. **If you change the skill and do NOT bump, installed users get nothing** — Claude Code sees the same version string, keeps the cached copy, and `/plugin update` reports "already at the latest version." Pushing commits alone does not propagate. Record changes in `CHANGELOG.md` if one exists.
8. **Validate (§5), then STOP** — stage, summarize what changed + the new version, and ask before committing/pushing.

**If you rename a skill:** the five-names rule (§1) means moving two directories AND editing three `name` fields (plus `source`) together, or the plugin fails to load.

**How the edit reaches installed users** — once the bumped manifest is on `main`:
```
/plugin marketplace update mad-skills
/reload-plugins
```
(or a restart). Third-party marketplaces have auto-update **off by default**, so without the bump-and-refresh the change never lands.

---

## 5. Pre-commit checklist

Run this gate before staging anything. Every box must be checked, then summarize the diff and ASK before `git commit`/`git push`.

- [ ] **Validator passes.** From the repo root: `claude plugin validate .` (in-session: `/plugin validate`). It checks `plugin.json`, `marketplace.json`, and every SKILL.md frontmatter — including field types and the `description` length — and must exit clean. This is the authoritative check; the `wc` command below is a quick local pre-check. In CI/strict contexts run `claude plugin validate . --strict` to turn warnings (leftover/misspelled fields) into errors.
- [ ] **Five names are byte-identical** (§1). Quick check — every line must read `<name>`:
  ```bash
  rg -N '"name"' plugins/<name>/.claude-plugin/plugin.json
  rg -N '^name:' plugins/<name>/skills/<name>/SKILL.md
  rg -N '"name"|"source"' .claude-plugin/marketplace.json
  ```
- [ ] **SKILL.md `description` is trigger-only AND ≤ 1024 characters — measure, don't eyeball.** Use `wc -m` (CHARACTER count), NOT `wc -c` (which counts BYTES — em-dashes and `→` arrows are multibyte, so `wc -c` over-counts and falsely flags compliant descriptions):
  ```bash
  rg -N '^description:' plugins/<name>/skills/<name>/SKILL.md | sed 's/^description: //' | tr -d '\n' | wc -m
  ```
  Must print `<= 1024`. (For reference, devils-advocate's description is 942 chars / 944 bytes — so `wc -c` would mislead you by 2.) The portable limit is 1024 characters; Claude Code's separate 1536-char listing budget is not the constraint.
- [ ] **Version bumped iff the skill changed.** New skills start `1.0.0`; edits bump semver (§4). With an explicit version set, pushing commits alone does nothing.
- [ ] **Portability preserved.** SKILL.md frontmatter uses ONLY `name` + `description`. No Claude-Code-only frontmatter and no shell injection / `$ARGUMENTS` / `${CLAUDE_*}` in the body, unless the skill is intentionally CC-only.
- [ ] **TDD-for-prose done** — baseline observed, then behavior verified WITH the skill (§3). Applies to edits too.
- [ ] **Asked the maintainer before commit/push.** Stage + validate + summarize, then wait for explicit go-ahead.

---

## 6. Gotchas (these have actually bitten here)

These are the failure modes whose detection isn't obvious from the sections above; the rest are enforced inline by §2–§5.

| Gotcha | What goes wrong | The rule |
|---|---|---|
| **Byte vs char measurement** | `wc -c` counts bytes; em-dashes/arrows are 3 UTF-8 bytes each, so a compliant 1024-char description can read as over-limit (and the count won't match the validator). | Measure the `description` with `wc -m` (chars), not `wc -c` (bytes). The validator is authoritative. |
| **`repository` / `homepage` / `keywords` type** | A wrong-typed field (`repository` as `{type,url}`, `keywords` as a string) is a HARD load error the validator rejects. An unrecognized field name is only a warning. | `repository` and `homepage` MUST be plain string URLs; `keywords` MUST be an array. |
| **Menu/example token drift** | A stray verdict token outside the declared menu (`COMMIT / SHIP / PROCEED / GO / ASSERT / ADOPT / HIRE`) is a self-contradiction bug the model imitates. | After editing, grep every example; it must draw ONLY from the declared menu. |
| **`marketplace.json` missing `owner`** | `owner` is a REQUIRED top-level object; omitting it fails validation. | Top level needs `name`, `owner: { "name": "Joern Ihlenburg" }`, and `plugins`. |
