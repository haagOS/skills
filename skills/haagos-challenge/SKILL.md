---
name: haagos-challenge
description: Adversarial review of a plan, issue/work item, plan file, design draft, or idea in any repo — including one written in this same session, which it attacks blind. Finds open questions, contradictions, unconsidered options and genuine design problems before anything is built. Discussion-only. Use when the user asks to challenge, stress-test or poke holes in a plan or proposal — "challenge this", "what's wrong with this plan", "review this design before we build it".
---

Input: $ARGUMENTS — an issue or work item (URL or number, in whatever tracker the repo uses), a doc
or plan-file path, or a pasted idea/plan. The thing under challenge is the "proposal", whoever
wrote it. Add `blind` to force blind mode.

## Orient

The proposal is judged against the repo's own rules, so learn them first. Where they conflict with
a default in this skill, the repo wins.

- Working agreements: CLAUDE.md, AGENTS.md, CONTRIBUTING, and whatever they point to.
- Design rules: architecture docs, ADRs, a glossary or ubiquitous-language file if one exists.
- The tracker: read the git remote. GitHub → `gh`; Azure DevOps → `az`, or a tracker skill if one
  is installed. No tracker → the proposal is the text or file you were given.
- The project's stage: pre-production, or shipped with users, stored data or external consumers.
  It decides how much compatibility weighs (step 7).

## Blind mode

When the proposal was authored by this session, or by whoever invoked the skill, run the challenge
in a **subagent with a fresh context**. Give it the proposal as written, the repo, its docs and
these instructions — and nothing about why anything was chosen. Rationale that lives only in the
author's head is not evidence, and a challenger that has heard the reasoning will confirm it. If the
plan cannot defend itself from its own artifact, that is a finding, and one the implementor would
have hit anyway.

Blind mode changes who reads, not what gets attacked. Everything below applies unchanged.

## The pass

1. Read everything first: the proposal, everything it links, the relevant docs, and the code it
   touches. When UI is involved and the app can be run, look at it running. A challenge without
   evidence is an opinion — don't present it.
2. Attack along four axes, in this order:
   - **Genuine design problems** — the proposal's model is smaller than the system it describes:
     missing actors/axes, rules its own examples break, claims the code already disproves.
   - **The option not on the list** — the proposal offers A and B; C exists and was never written
     down. This is where an author's blind spot lives, so it is the axis that earns blind mode.
     Also: an option that was rejected for a reason that no longer holds, and a recommendation whose
     rationale is "consistent with X" where X was chosen to answer a different question.
   - **Contradictions** — the proposal's invariants against each other, against shipped behavior,
     against the repo's docs and glossary. Quote both sides.
   - **Open questions** — cases the proposal doesn't decide (degenerate inputs, absent parts,
     who-owns-what). Only questions whose answer changes the design.
3. Against a *plan* specifically, also check the seams a prose proposal doesn't have: every task has
   a stated verification; every edge case the plan raised has an owner (a pinned test or an
   explicit out-of-scope note); no task exists that no decision produced; no decision is
   load-bearing for a task that never cites it; the evidence behind a decision isn't one
   measurement generalized.
4. Presentation: findings in chat, worst first, numbered, under those headings. Each finding: a
   one-sentence claim, then the evidence with file:line references.
5. **Name what you attacked.** Finish with the attack list — the axes swept, what was checked and
   held, and what you deliberately did not look at. "Nothing found" is a legitimate outcome and a
   valuable one, but only with that list attached; without it, a clean sweep is indistinguishable
   from a lazy one. This matters most in blind mode, where nobody else saw the work.
6. End with "What I'd change before blessing" — the shortest set of amendments that would make you
   stop objecting.
7. Triage before presenting: drop nitpicks. Judge against model coherence and standing invariants.
   Compatibility counts only where something real would break: before production it counts for
   nothing; once there are users, stored data or external consumers, breaking them is a genuine
   design problem. If the stage is unclear, say which you assumed. Naming, wording and style go in
   one short list at the end or not at all.
8. The challenge runs both ways. If the user pushes back, test their argument against the code —
   measure when you can — and either concede with the evidence or hold the position and say why.
   Never cave to authority, only to evidence. "Don't blindly accept" applies to the user's
   counter-arguments too.
9. After each round of answers: a convergence recap — what is now settled (one line each), what
   remains open. Keep the remaining list shrinking.
10. Hard fence: discussion only. No code, no mockups, no issue or plan-file edits until the user
    says settled. Then write the outcome where the proposal lives — the issue, or the plan file.
    Posting to a tracker publishes it, so confirm target and content first. Genuinely separate
    problems go through the triage gate below.

## Triage gate for separate problems

Use the repo's own gate if it has one. Otherwise, a problem outside the proposal is filed only if
it corrupts data, breaks a user-visible flow, or violates a standing invariant; trivial ones are
fixed inline; everything else is dropped — the usual outcome.

- **Reachability.** If reaching the state needs another bug to be present, or input no surface can
  produce, it is a documented limitation: a sentence in the doc that owns the rule, not an issue.
- **A missing test is never an issue.** It has no user consequence by construction.
- One session's findings in one area become **one** consolidated issue with a checklist, never one
  issue per finding.

Planning stays a separate step (the `haagos-plan` skill).

Style: simple English, short sentences, no filler. Each finding must stand alone — the reader
shouldn't need to look anything up to understand the claim.
