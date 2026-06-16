# mad-skills

A few [Agent Skills](https://agentskills.io) I use and figured were worth sharing.

Each one is a single `SKILL.md`: a markdown file with a short YAML header that tells the agent when to load it. There's no code and nothing to install beyond the file itself. They work in Claude Code, and in anything else that reads the Agent Skills format (Copilot CLI, Codex, Cursor, Gemini CLI).

The repo is a small marketplace, so you grab only the skills you want rather than the whole pile.

## Skills

| Skill | What it's for |
|---|---|
| `devils-advocate` | Have a subagent argue against a decision before you commit to it. Meant for the calls that are expensive to get wrong: an architecture or migration choice, a security model, a claim in a doc or paper, a hire. |

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

## Adding your own

Each skill is its own plugin so people can install them à la carte. To add one:

1. Make `plugins/<name>/` with a `.claude-plugin/plugin.json` and your skill at `skills/<name>/SKILL.md`.
2. Add an entry for it to `.claude-plugin/marketplace.json`.
3. Run `claude plugin validate .` to check the manifests, then open a PR.

## License

MIT. Do what you like with it.
