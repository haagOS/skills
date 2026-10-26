---
name: haagos-plan
description: Autonomous planning pass for any repo. Reconciles an issue/work item (or a rough feature sketch) against the code, researches its open questions itself, challenges the result, and only then presents a plan — blocking decisions first. Explore-only; no code changes. Use when the user asks to plan an issue, a work item or a feature idea before building it — "plan this issue", "plan #123", "make a plan for".
---

Input: $ARGUMENTS — an issue or work item (URL or number, in whatever tracker the repo uses), or a
rough feature sketch with no issue yet. A sketch's plan goes into a plan file (stage 7).

Stages 0–5 run without the user. They see the plan once, at stage 6. The point of the autonomy is
better options, not fewer round-trips — the round-trips were already cheap. Everything below
exists to stop the autonomy from quietly deciding things that were never yours to decide.

Once at stage 6 means once. Don't narrate the stages as they pass: a status line invites an answer
to a plan that does not exist yet, and it is the round trip this skill exists to spend better. Work
in the background so the user can interject if they want to — that is their call to make, not a
reason to check in.

## Stage 0 — Orient and reconcile

First learn the repo's rules. Where they conflict with a default in this skill, the repo wins:

- Working agreements: CLAUDE.md, AGENTS.md, CONTRIBUTING, and whatever they point to.
- Design rules: architecture docs, ADRs, a glossary or ubiquitous-language file if one exists.
- The tracker: read the git remote. GitHub → `gh`; Azure DevOps → `az`, or a tracker skill if one
  is installed.
- The project's stage: pre-production, or shipped with users, stored data or external consumers.
- Conventions for tasks, branches, commits and PRs.

Then read the issue and everything it links, the relevant docs, and the code. When an external spec
governs the feature (a protocol, a file format, a library's contract) read the authoritative source.
When the plan invents a domain model or a shared contract, survey at least one mature prior
implementation before designing our own. Check open bugs in the tracker touching the same area;
each becomes a predecessor task or an explicit deferral note. When UI is involved and the app can
be run, inspect it running.

Deliverable, before any question is asked: **what the code says that the issue does not.** Numbered.
Every stale line reference, every count that is now wrong, every claim the code disproves, every
capability the issue assumes exists and doesn't. An issue written more than a few days ago is
assumed stale until checked — this is the most common way a good plan gets built on sand.

If the issue's text is more wrong than right, the plan replaces it rather than amends it.

A rough sketch has no issue to reconcile. Reconcile the sketch instead, same output: what the code
says that the sketch does not.

## Stage 1 — Enumerate, classify, order

List every question the plan cannot answer from what stage 0 read. Then, per question:

- **Is it a fork?** Test it mechanically, not by feel: plan both answers to task *titles* only. Largely
  disjoint titles, different files touched, or one branch needing a doc the other doesn't → fork.
  Same titles differing in a parameter → not a fork, decide it later like anything else.
- **Does it gate another question?** Order accordingly. A question whose answer changes what a later
  question even means is settled first, and the later one is researched under the settled answer.
  Never research two ends of a dependency in parallel and reconcile afterwards.

Run the input-space sweep here. The axes:
- **Absent parts** — behavior when an assumed component is missing or empty.
- **User-typed variance** — casing/spacing/synonym variants; name the single normalization point.
- **Round-trip** — does every value this feature can *generate* re-read coherently through the
  system's own parsers?
- **Unknown-but-legal** — values outside the recognized set: explicit passthrough or fallback?

**Dropping is the default outcome, and the usual one.** A case earns a question or a pinned test
only by passing the triage gate — the repo's own if it has one, otherwise: reachable (no other bug
needed, some surface can produce the input), and consequential if reached (corrupts data, breaks a
user-visible flow, violates a standing invariant). Most will not, and one that fails it becomes at
most a line in the plan saying so: not a test, not an issue, not a TODO.

Report what the sweep dropped before what it kept. A sweep that turns up twelve cases and keeps one
has done its job; one that keeps twelve has moved the work rather than done it.

Queue check: if more than about seven items look headed for stage 3's escalation queue, that is a
signal the issue is under-decided — take it to stage 4's split test rather than asking twelve
questions. This counts questions, which exist now. It says nothing about tasks, which do not.

## Stage 2 — Research

One subagent per independent question; dependent ones wait their turn. Synthesis stays here — a
subagent cannot see how its answer interacts with the others.

**Every brief opens with whether, not how.** The first question is always whether the thing should
exist at all: what it buys, what already provides that, and what is actually lost by not building
it. Research that costs three ways to build something nobody needs has produced nothing,
expensively — and it will read as authoritative precisely because it is thorough. Costing the
implementations is the second half of the job, and it is wasted if the first half was skipped.

Each brief also carries the question, what stage 0 established, and the docs and code to read. Demand
evidence rather than preference: a recommendation with no file, no spec quote and no measurement is
not an answer. Measure instead of reasoning wherever measuring is possible — a throwaway probe
against the real packages, a query against a local database, a generated spec. Probes are deleted
after they answer.

Forks are researched as **two costed branches**, never toward one recommendation. Plan the
recommended branch to full depth and the other to task titles plus consequences. Fully planning both
doubles the work for a decision that mostly goes one way; if the user takes the other branch, re-run.

Record a runner-up only when the gap is genuinely small, or when the loser is the option a
reasonable person would have guessed. Everywhere else a runner-up is noise.

## Stage 3 — Decide, or escalate

Decide everything you can. Escalate on these six triggers and nothing else:

1. **Fork** — the branches lead to disjoint work.
2. **Preference with no repo evidence** — the answer depends on what the product is supposed to be.
   Test: could two competent engineers who read every doc land on opposite answers without either
   being wrong? Caps and limits, monetization posture, what counts as a social problem rather than a
   digital one, how much friction a destructive flow deserves.
3. **One-way door** — data loss, destructive migration, external-system state, anything that cannot
   be undone by editing code.
4. **Precedent** — first instance of a pattern in this repo, which the next task like it will copy.
   Before escalating, do a short design pass: survey mature prior art outside the repo; separate
   what the platform or data model forces from what we are choosing; write the *decision rule* —
   when to reach for this, when not, and the signs for each — not just an answer; walk the
   degenerate and concurrent cases through it. The rule is what gets escalated.
5. **Unresolvable contradiction** — two repo docs disagree, or the issue contradicts shipped
   behavior. Don't pick which doc wins.
6. **Verification gap** — the outcome cannot be checked without the user's eyes, device, or
   environment. Autonomy extends exactly as far as verification does.

Not triggers: the answer is in a doc you can read, or in an external spec you can fetch; one option
strictly dominates; being wrong costs a cheap rewrite. Decide those.

Every decision taken here goes on the **Decided without you** list — one line each, naming the
trigger that permitted it. A decision that isn't on that list wasn't made, it was smuggled.

## Stage 4 — Coherence

Assemble the whole plan under the decided answers, then read it as one artifact and hunt for:
a decision that only works under an assumption a different decision broke; a task in the breakdown
that no decision produced; a decided answer whose evidence came from code that a later decision
changes. Fix, or promote to a question.

**Then the split test, which is not a count.** The task list exists for the first time here, so this
is where to ask whether it should be one issue. Length is a smell worth checking, never a verdict —
six tasks that all serve one set of decisions are one issue. What earns its own issue is a piece
with its own **design surface**: decisions that aren't this issue's to make, a rule the rest of the
plan doesn't need settled to proceed, a precedent that deserves its own pass. Extract that and
nothing else. Three tasks where one is quietly a second design problem are two issues; ten tasks
serving one decision are one issue with a long list.

## Stage 5 — Self-challenge

Dispatch the `haagos-challenge` skill in blind mode — a subagent, given the plan as written and
nothing about why anything was chosen. Loading the skill into this context instead would have the
author challenge itself, which is the one thing this stage cannot be. If that skill is not
installed, the subagent still runs blind: it attacks the plan for design problems, the option not on
the list, contradictions and open questions, with evidence for each, and names what it attacked.

Its findings are yours to fix before presenting. A finding you disagree with is carried to stage 6
as a question — never dismissed silently, because dismissing your own challenger is the failure this
stage exists to prevent.

## Stage 6 — Present

Header first, in this fixed order, short:

1. What changed about the issue — stage 0, one line each.
2. Forks — count only.
3. Preference calls — count only.
4. Decided without you — count only; the list on request.
5. What the challenge found, including "nothing" and what it attacked.

Then one item per message, blocking first: forks, then one-way doors and precedent, then
preferences. Each item stands alone — the question, what you found, options labeled A/B/C, your
recommendation and its rationale. Don't make the user look anything up. For a fork, per branch:
what it makes easy, what it forecloses, and the **reversal cost** — what undoing it costs in a
month. Reversal cost is usually what actually decides these.

Prose in the chat. Never the AskUserQuestion tool — not for questions, not for option picks, not
for confirmations.

The full plan goes into the chat as well, after the header and the items: every section of the
draft as written, not a summary of it and not a pointer to a scratch file.

Ratification is the risk this skill creates. A plan that has already survived its own challenge is
harder to argue with than a plan that hasn't, and the standing habit is to approve and move on. So a
decision made on the user's behalf must never be dressed as a finding: different heading, different
ask, and the count in the header even when nothing on the list looks contentious.

Mockups or prototypes (a `prototype` skill, if one is installed) only once questions are resolved and the user
approves creating one.

## Stage 7 — Write it down

Only once stage 6 has settled. Where it goes:

- **Input was an issue** → the issue. Posting to a tracker publishes the plan, so confirm before
  writing.
- **Input was a sketch** → a plan file. Look for a gitignored plans folder anywhere in the repo —
  a directory named `plans`, typically `docs/plans` or `plans`, possibly nested (e.g.
  `Client/docs/plans`); skip `node_modules`, `bin`, `obj` and the like, and keep only those
  `git check-ignore` confirms are ignored. Exactly one → write `<short-slug>.md` there. Several, or
  none → ask where. Never put a plan file in a tracked location without being told to.

Plan content: the reconciliation, the decisions with a one-line rationale each, what the sweep
dropped, and a task breakdown — each task sized for one commit and carrying its own stated
verification. Follow the repo's branch/commit/PR conventions; the default is one branch, one
commit per task, one PR. A task whose verification looks likely to outweigh the production code it
adds says why in that line; if it cannot, the cases it is pinning were probably not triaged. Two to
five tasks is the common case, not a gate; a longer list is a prompt to re-run stage 4's split test,
and never something to report as a breach. Name dependencies and blockers. A plan that renames,
moves or deletes a UI control names the tests that address it by name or selector (e2e, UI tests)
— migrating them is part of the work, not a follow-up.

Research dossiers are kept apart from the plan. On an issue: **one collapsed comment** (`<details>`
where the tracker renders it; otherwise one comment headed as the research record), not the body.
In a plan file: a final `## Research` section. The plan gets rewritten freely as decisions settle;
the research record is the append-only audit trail. Resolved forks stay recorded there — they are
the seeds of the next ADR.

Style, everywhere in this skill: simple English, short sentences, no filler. Say the thing once.
Being complete beats being brief; being elaborate does not.

Hard rule: no code changes, throwaway probes excepted, and those are deleted once they have
answered. Implementation is a separate step and starts only after approval.
