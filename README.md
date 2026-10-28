# haagOS skills

Two agent skills that find your blind spots before the plan is written, not after the code is.

| Skill | What it does |
| --- | --- |
| [`haagos-plan`](skills/haagos-plan/SKILL.md) | Turns an issue, or an idea you discussed in the session, into a plan. Checks it against the code, researches the open questions itself, attacks its own result, and only then brings you the decisions that are really yours. No code changes. |
| [`haagos-challenge`](skills/haagos-challenge/SKILL.md) | Adversarial review of a plan, issue, design draft or idea. Finds design problems, the option nobody wrote down, contradictions and open questions, with evidence. Runs blind when it reviews its own session's work. Discussion only. |

Both work in any repo. They read the repo's own rules first (CLAUDE.md, AGENTS.md, CONTRIBUTING,
ADRs, glossary). Where those conflict with a skill's defaults, the repo wins.

## Why these exist

For a few months, most of my sessions with coding agents started the same way: an issue, or just
an idea, feature or bug I had talked through in the session. Then, before any plan was worth
reading, I typed the same sentences again and again:

*Explore the code. Make suggestions. Do research on that topic. Challenge me on that. List the
options, give your recommendation and rationale. Present the options one at a time.*

These two skills are those sentences, written down once and sharpened over a few iterations. They
do things I have rarely seen elsewhere. Most of all, they don't treat what I said or wrote as
already decided.

## Where they fit

```text
grill session or issue  →  haagos-plan  →  haagos-challenge  (repeat as decisions change)
```

Matt Pocock's [skills](https://github.com/mattpocock/skills) are excellent at the step before this.
`grill-with-docs` finds out what you want and keeps the glossary and ADRs in line while it does.
`to-spec` turns that conversation into an issue. But an interview can only surface what you know to
ask about, and a spec is a synthesis of your answers. In my experience the plan that comes out
still has blind spots: things the code already answers or contradicts, questions with well-known
general answers, and decisions that don't fit together.

`haagos-plan` takes over there. It checks the issue or session against the code, answers what it
can by itself with evidence, attacks its own result, and only then brings you the questions that
are really yours.

## How `haagos-plan` works

Stages 0 to 5 run without you. You see the plan once, at stage 6. The autonomy is not there to
save round-trips. It is there to make the options better before you see them.

**0 — Reconcile.** Issues go stale, and many are a few sentences long. The code has moved on, or it
already answers the question, or it disagrees with the issue. The first output is a numbered list:
what the code says that the issue does not. The plan builds on the status quo, not on the issue's
memory of it.

**1 — Forks, gates and the input sweep.** Every open question is tested: is it a *fork* (the two
answers lead to different work) or just a parameter? Does it *gate* another question (its answer
changes what the other one means)? Those are the questions that change a plan materially, so they
are ordered first. Then a sweep over the inputs (missing parts, user-typed variants, round-trips,
unknown-but-legal values) finds the technical blind spots. Dropping a case is the default outcome,
and the sweep reports what it dropped before what it kept.

**2 — Research.** Many technical questions already have good answers at a general level. There is
no point asking you about them. One subagent per question, and every brief starts with *whether*
the thing should exist at all before *how* to build it. Answers need evidence: a file, a spec
quote, a measurement. The research is kept as its own record next to the plan, so you can see later
why the agent decided what it decided.

**3 — Decide, or escalate.** Everything from stages 0 to 2 comes together here, and this is the
first place anything gets decided. Most planning flows I have seen skip the first three stages and
take your input as the decision. Here the agent decides what it can and escalates only on six
triggers: forks, preferences with no evidence in the repo, one-way doors, precedents, real
contradictions, and things only you can verify. Every decision it takes goes on a **Decided
without you** list, with the trigger that allowed it.

**4 — Coherence.** The whole plan is read as one thing. Does a decision only work under an
assumption another decision broke? Is there a task that no decision produced? Should part of it be
its own issue?

**5 — Blind challenge.** `haagos-challenge` runs in a fresh subagent. It gets the plan as written
and nothing about why anything was chosen. If the plan can't defend itself from its own text,
that's a finding. Findings get fixed, and any the agent disagrees with come to you as questions.
They are never dropped silently.

**6 — Present.** A short header first: what changed about the issue, how many forks, how many
preference calls, how many decisions were made without you, and what the challenge found. Then one
decision at a time, blocking ones first, each with options, a recommendation, a rationale and, for
forks, the cost of reversing it in a month. Then the full plan.

**7 — Write it down.** Once decisions are settled, the plan goes into the issue (after you confirm)
or into a gitignored plan file. The research goes alongside it as an append-only record.

## What you get

- **Your blind spots, raised relentlessly.** You know what you want, but not everything the code,
  the docs or the wider world already says about it. The skill does.
- **Answers backed by evidence.** What the agent can find out by itself, it does, and it shows the
  file, quote or measurement behind each answer.
- **A plan that holds together.** It is much less likely to contradict the docs, the code, or
  itself.
- **Transparent autonomy.** Every decision made on your behalf is counted in the header and listed
  with its reason. Nothing is smuggled in as a "finding".
- **Less noise.** Edge cases have to earn their place. The plan says what it dropped instead of
  turning every case into a test or a ticket.

## `haagos-challenge` on its own

The challenge is its own skill, so you can run it again whenever you want: after you answer the
plan's questions, after a decision changes, or before you start building. It checks whether the
plan still holds.

It is very satisfying when the answer is "the plan holds", or when it finds only a few small
things the agent can fix by itself. Every run ends with a list of what was attacked, so "nothing
found" means something.

It works on anything, not only plans from `haagos-plan`: an issue, a design doc, or an idea you
paste into the chat.

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

Install both. `haagos-plan` still works without the challenger, but falls back to a generic blind
review.

Issue trackers: GitHub (`gh`) and Azure DevOps (`az`) are detected from the git remote. Without a
tracker, the input is the session or a plan file.

## Usage

On an issue:

```text
/haagos-plan #42
/haagos-challenge #42
```

Or in the middle of a session, after talking through an idea, feature or bug:

```text
run a /haagos-plan pass over that
run a /haagos-challenge pass to see if the plan holds
```

`haagos-challenge` switches to blind mode by itself when it reviews a plan written in the same
session.

As a plugin, the commands are namespaced: `/haagos:haagos-plan`, `/haagos:haagos-challenge`.

## License

[MIT](LICENSE)
