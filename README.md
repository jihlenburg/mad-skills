# mad-skills

A few [Agent Skills](https://agentskills.io) I use and figured were worth sharing.

Each one is a single `SKILL.md`: a markdown file with a short YAML header that tells the agent when to load it. There's no code and nothing to install beyond the file itself. They work in Claude Code, and in anything else that reads the Agent Skills format (Copilot CLI, Codex, Cursor, Gemini CLI).

The repo is a small marketplace, so you grab only the skills you want rather than the whole pile.

## Skills

| Skill | What it's for |
|---|---|
| `devils-advocate` | Have a subagent argue against a decision before you commit to it. Meant for the calls that are expensive to get wrong: an architecture or migration choice, a security model, a claim in a doc or paper, a hire. |
| `trains-of-thought-audit` | Audit a long, much-edited or LLM-written document for places where it contradicts itself — numbers that drift between sections, invention/figure numbering that breaks, claims it rejects in one place and uses in another. |

More will show up here over time.

## Installing

In Claude Code, add the marketplace once:

```text
/plugin marketplace add jihlenburg/mad-skills
```

Then install whatever you want from it:

```text
/plugin install devils-advocate@mad-skills
```

After that the skill loads on its own when it's relevant. You can also call it by hand with `/devils-advocate:devils-advocate`.

Not using the plugin system, or on a different agent? Copy the folder instead:

```bash
cp -r plugins/devils-advocate/skills/devils-advocate ~/.claude/skills/
```

## About devils-advocate

When you ask a model to check its own work, the assumptions that produced the answer tend to produce the review too, so you mostly get agreement with extra steps.

This skill gets around that by spinning up a fresh subagent and handing it a side to argue, a specific thesis against your proposal, rather than asking it to "be critical." It also makes the reviewer lead with its strongest objection instead of softening into "this looks great, but…", say out loud what evidence would change its mind, and finish on an actual call rather than "it depends."

I originally wrote it for pitch and strategy decisions and then generalized it. The same brief now holds up for engineering, research, security, and hiring calls; the only things that really change between domains are the proposal, the counter-thesis, and the words you use for the verdict.

One caveat on when to bother: it's worth running before something costly or hard to undo. For a one-line fix or a reversible toggle it's overkill, and the skill says as much itself.

## About trains-of-thought-audit

Long documents that have been edited a lot — or written by an LLM in one long stretch — quietly contradict themselves. A number is 20,000 in one section and 24,000 forty pages later; a concept is defined one way early and used differently later; an "Invention #8" is declared folded in one place and cited as live in three others. The cause is that the document's actual specifications are never written down and tracked, so each section re-derives them from fading memory.

This skill audits for that. It builds a typed registry of every claim (numbers, definitions, repudiations, invention/figure numbering, the sums that should add up), reconciles every mention against it, and separately does a directed read for the contradictions a registry misses. Every finding is then re-checked against the document by re-quoting both sides, so what comes back is verified, not guessed.

I tested it on a real 75k-word report, and it's honestly a precision aid rather than a magic wand: it found a stack of real contradictions at zero false positives, but a plain careful read still caught a couple it missed (internal arithmetic is its weak spot). So run it alongside a read, not instead of one — the skill spells this out in its own coverage-limits section.

## Adding your own

Each skill is its own plugin so people can install them à la carte. To add one:

1. Make `plugins/<name>/` with a `.claude-plugin/plugin.json` and your skill at `skills/<name>/SKILL.md`.
2. Add an entry for it to `.claude-plugin/marketplace.json`.
3. Run `claude plugin validate .` to check the manifests, then open a PR.

## License

MIT. Do what you like with it.
