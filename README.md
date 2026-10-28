# haagOS skills

Agent skills for planning and challenging work before it gets built. They work in any repo and
read the repo's own rules first (CLAUDE.md, AGENTS.md, CONTRIBUTING, ADRs) — where those conflict
with a skill's defaults, the repo wins.

| Skill | What it does |
| --- | --- |
| [`haagos-plan`](skills/haagos-plan/SKILL.md) | Autonomous planning pass. Reconciles an issue or a rough feature sketch against the code, researches the open questions itself, challenges the result, and only then presents the plan — blocking decisions first, plus a list of what it decided without you. No code changes. |
| [`haagos-challenge`](skills/haagos-challenge/SKILL.md) | Adversarial review of a plan, issue, design draft or idea. Hunts for design problems, the option nobody wrote down, contradictions and open questions — with evidence, and with a list of what it attacked. Runs blind (in a fresh subagent) when it is reviewing its own session's work. Discussion only. |

## How they fit together

`haagos-plan` runs `haagos-challenge` on its own draft in blind mode before you ever see it, so the
plan reaches you already attacked once. You can also run `haagos-challenge` on its own, on any plan
or issue. Install both: `haagos-plan` still works without the challenger, but falls back to a
generic blind review.

Issue trackers: GitHub (`gh`) and Azure DevOps (`az`) are detected from the git remote. Without a
tracker, the input is a pasted sketch or a plan file.

## Install

With the [`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
npx skills add haagOS/skills
```

As a Claude Code plugin:

```text
/plugin marketplace add haagOS/skills
/plugin install haagos@haagos
```

Or copy the folders under `skills/` into `~/.claude/skills/` (or your agent's skills directory).

## Usage

```text
/haagos-plan https://github.com/owner/repo/issues/42
/haagos-plan add CSV export to the reports page
/haagos-challenge docs/plans/csv-export.md
/haagos-challenge blind #42
```

As a plugin, the commands are namespaced: `/haagos:haagos-plan`, `/haagos:haagos-challenge`.

## License

[MIT](LICENSE)
