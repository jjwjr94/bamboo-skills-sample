---
name: author-opportunities
description: Turn audit findings, session notes, failed runs, or messy gaps into a grouped opportunities roadmap — scan the repo/runtime evidence first, resolve what can be resolved, group remaining gaps into answerable modules, rank by judgment, and walk unresolved questions one at a time. Use after an audit, retro, batch of answers, meaningful context change, or escaped failure. Never on a calendar.
version: 0.2
stage: run
emits: [opportunities-roadmap]
consumes: [audit-findings, session-notes, run-outputs, context-diff, prior-roadmap, annotations]
---

# Author opportunities

> **External-share copy** (2026-08-10): customer identifiers and person names anonymized. v0.2 makes the default host plain Claude Code: write a Markdown opportunities roadmap.

The producer for an **opportunities roadmap**. `audit-agent` or a messy session finds
gaps; this skill is the judgment in between discovery and action — deciding what is worth
asking, in what shape, and just as often deciding what to *retire*.

The goal, in the owner's words: **give the customer attainable bites — improvements they
can make or questions they can answer.** Most answers should come from the repo, runtime,
or notes before the customer is bothered. Not a health score, not a checklist, not a
homework packet.

```
evidence (findings · notes · runs · diffs · answers) → [assess → scan → harvest → shape → rank → draft → Q&A] → roadmap.md / opportunities.md → human confirms → action
```

## Output target

Default in plain Claude Code: create or update `roadmap.md` or `opportunities.md` in the
project. If neither exists, draft `opportunities.md` and say it is a proposal.

If the host project has a first-class opportunity tracker, use the same module shape there
after human confirmation. Treat tracker rows, IDs, and delivery mechanics as adapters, not
as part of the core method.

## The one rule

**State what the corpus already has, then ask only for the gap.**

Scan before asking. Read the repo docs, skill files, run logs, issues, saved notes,
roadmap, and any runtime/tool output you can access. If the answer is in those artifacts,
use it and cite it. Ask the owner only for judgment calls, contradictions, missing facts,
or decisions the artifacts cannot settle.

Every one of the seven rows this skill is codified from does this, and it is why they read
as tasks instead of homework:

> *"The current notes already define ROAS, CAC/blended CAC, and LTV at a high level"* → ask only for
> nCAC, MER, LTV:CAC, churn, repeat rate, active subscriber.

> *"The updated growth chart names Morgan as Retention Lead; an older view names Riley.
> Should the older assignment be marked superseded?"*

A generic gap-finder emits *"unit economics: incomplete"* — a score. These emit a question
someone finishes tonight — a task. **That difference is the product.** An opportunity that
asks for something the corpus already holds teaches the reader the roadmap doesn't know the
work, and she stops reading it. This rule costs a corpus read per draft, and it is
never optional.

## What counts as an opportunity — the dividing rule

> **An opportunity is something only the intended owner can resolve. Everything else is
> observability.**

A missing metric definition, an unresolved owner, a contradiction between two pages, a
stale forecast — only she can settle those. A dark drain, a failed skill sync, a connector
500, an ingestion bug — ours, and putting them in her roadmap trains her to ignore it. One
narrow exception: a *stalled* ingest means she believes we hold a document we don't —
that fact is hers to know, even though the fix is ours.

## Inputs — where opportunities come from

In rough order of value. Use whatever the project has; none require special infrastructure.

| Input | What it yields |
|---|---|
| **Repo/runtime scan** | Existing docs, `CLAUDE.md`, skills, issues, tests, logs, tool outputs, dashboards, and config often answer the obvious questions before a human needs to |
| **`[GAP]` markers in run outputs** | The agent names its own gaps *while answering a real question* — demand-shaped without a demand read. Measured 2026-08-06: one passing run named five (`Amazon owner, Analytics owner, Promotions owner, numeric budget threshold, formal tie-break rule`), all machine-readable, none harvested |
| **Context diffs** | A deletion is content, not just a trigger: *"these claims are now unbacked"* is itself an ask. 4 pages deleted 2026-08-07 were exactly one module's corpus |
| **Answers to prior opportunities** | An answer closes a subquestion, can invalidate a sibling, and often surfaces a new gap. Her questions *back* are the highest-quality demand signal available |
| **Contradictions between pages** | A static corpus check, distinct from a diff — the Morgan/Riley row came from one |
| **Annotations** | `eval_labels` + annotation-task dismissal reasons. "Failure but NOT an agent defect" dismissals are often knowledge gaps with no home |
| **Audit findings** | The classic source — sourced 6 of the 7 first rows |
| **Live-run feedback + `ask` answers** | What actually broke or surfaced in her real work |
| **Quiz per-module scores** | Module-granularity coverage, once the batch is complete — never interpret a partial one |
| **Escapes** | If the cause was a knowledge gap, it is an opportunity as well as a golden case |
| *Real user questions* | Future input when available, not a blocker |

## Phase 1 — Assess before you author

**Re-assessment is half the job, and it runs first.** The most valuable run of this skill
may *shorten* the roadmap.

For every open opportunity:

1. **Still valid?** Did a corpus change invalidate it? (The 4 deleted claims-and-compliance
   pages sit exactly under one open module.)
2. **Already answered elsewhere?** An answer that arrived through a different channel —
   a brain save, an `ask`, a meeting note — closes the subquestion; nothing does that
   automatically.
3. **Declined or ignored?** If it has sat unanswered across multiple touches, that is
   information: the ask may be too big, too vague, or not hers. Propose reshaping or
   retiring it, with the reason.
4. **Sibling effects.** One answer often settles part of another module — mark those,
   don't re-ask them.

⚠ The whole roadmap sitting untouched is not a reason to produce more. Measured: 7 modules,
~25 subquestions, zero answered in three days. Another task queue died the same way —
83 rows, all eventually dismissed. **Producing into an undrained roadmap is how a demand
signal becomes a backlog.**

## Phase 2 — Scan before you ask

For each candidate gap, do a bounded scan before drafting a question:

1. **Repo evidence.** Search docs, `CLAUDE.md`, skill files, configs, tests, examples,
   issue references, and existing roadmap/memory files.
2. **Runtime evidence.** Inspect run outputs, logs, traces, screenshots, tool output, or
   eval results when the project gives you access.
3. **Prior answers.** Check previous roadmap entries, meeting notes, user replies, and
   dismissed/declined items.
4. **Decision boundary.** If evidence answers it, cite the answer and remove or narrow the
   ask. If evidence conflicts, ask the owner to choose. If evidence is missing, ask the
   smallest question that would unlock the next action.

The scan is bounded by value: stop once the remaining uncertainty is clearly an owner
decision or the next search would cost more attention than asking.

## Phase 3 — Harvest and shape

For each candidate gap that survives the dividing rule:

- **Run the one rule.** Read the corpus first. What does it already hold on this?
  Quote it in the draft. Ask only for the delta.
- **Group into modules.** ~25 loose questions is a wall; the same 25 grouped into named
  modules ("Metrics and source rules", "Owners and approvals") with keyed subquestions is
  workable — the module is the unit the customer engages with, the subquestion is the unit
  of progress. Every subquestion carries `purpose` and `required`, so a partial answer is
  representable and still moves things.
- **Name the artifact.** `expected_artifacts` says what exists when this closes. If
  answering it wouldn't change what the agent does, it is trivia — drop it.
- **Point at evidence, never by restatement.** Evidence may be file paths, headings, issue
  links, run IDs, transcript excerpts, or audit finding IDs. **The finding is derived —
  reference it. The ask is authored — that is the only original content this skill
  produces.** A roadmap that restates findings is a stale copy waiting to drift.

## Phase 4 — Dedup: never ask twice, except on purpose

Before drafting, check:

| Case | Rule |
|---|---|
| Same gap, different wording | Key on the **underlying finding**, not the phrasing. If the evidence IDs overlap an existing row, it is the same opportunity |
| Previously answered | Never resurface. Note: `answered` is per-subquestion — check the parts, not just the parent |
| Asked and declined | Do not re-ask without a reason *she* would accept. Record the decline and why |
| Legitimately recurring | Some questions are *supposed* to return — *"is Scenario 2 still the operating forecast?"* goes stale annually. Mark these explicitly as recurring with a horizon, so their return is a policy, not a bug |

The precedent for getting this wrong is measured: annotation-task creation ignored scope
history and 78% of open tasks pointed at paused cases. **Check history before creating.
Every time.**

## Phase 5 — Rank, and say the ranking is judgment

`gap severity × observed demand × step unlocked` is the frame. **Observed demand is
currently dark** (no usage read), so the ranking is expert judgment — the draft must say
so out loud rather than implying measurement. A criteria gap nobody ever asks about
outranks nothing.

Tiebreak toward: unblocks a workflow step (the delegation ladder) > settles a
contradiction > fills a definition > refreshes recency.

## Phase 6 — Draft, and stop

**The skill drafts; a human confirms before anyone acts.** An opportunity spends the
owner's attention — a bad one costs trust, not just a cycle.
Drafting is judgment; sending is a decision.

Emit per module: `title` · `summary` · `why_it_matters` (customer language, never
rubric language) · `expected_artifacts` · keyed `subquestions` with `purpose` +
`required` · `evidence` as source-row references · `priority` with the one-line judgment
behind it. Plus the assessment verdicts from Phase 1: keep / reshape / retire per open
row, each with its reason.

In plain Claude Code, write this as Markdown:

```markdown
# Opportunities roadmap

## Assessment of existing opportunities
- Keep / reshape / retire: ...

## Module: <title>
- Why it matters:
- Expected artifact:
- Priority:
- Evidence:
- Already known:
- Questions / decisions needed:
  - [required] <question> — Purpose: ...
  - [optional] <question> — Purpose: ...
```

## Phase 7 — Work the questions one at a time

The roadmap may contain many subquestions. The conversation should not.

Default to the highest-priority incomplete module, unless the owner chooses another one.
Before asking, show a compact evidence block:

```markdown
Module: <title>
What I already found:
- <evidence-backed fact>
- <evidence-backed fact>
Still missing:
- <gap this question resolves>
```

Then ask **one focused question at a time**. A focused question can have tightly related
parts when one answer naturally covers them, but it must fit in one response. Do not dump
the whole module's question list.

After each answer:

- Record the answer in `roadmap.md` / `opportunities.md` under the relevant module.
- Mark the subquestion `answered`, `deferred`, `skipped`, or `unavailable`.
- If the answer resolves or invalidates sibling questions, update them immediately.
- Confirm the captured facts briefly before moving on.
- Ask the next highest-leverage unresolved question, or stop if the owner is done.

## Anti-patterns

- **Asking for what the corpus already has.** The one rule. Everything else here is
  recoverable; this is how the roadmap loses the reader.
- **Skipping the scan.** A question the repo or runtime could have answered is avoidable
  customer work.
- **Batching Q&A.** A roadmap can group many questions; the conversation asks one focused
  question at a time.
- **Producing into an undrained roadmap.** Assessment first, always.
- **Restating findings.** Point at them. The ask is the only authored content.
- **A health score wearing a question mark.** "Improve your metrics coverage (currently
  D+)" is not an opportunity.
- **Ungrouped question lists.** 25 loose questions is a wall; the module is the unit.
- **Re-asking a declined question because the finding fired again.** The finding firing
  is expected — she declined it, and that decision stands until something material changes.
- **Mixing our bugs into her roadmap.** The dividing rule.
- **Implying the ranking is measured.** It is judgment until a demand read exists.
- **Auto-sending.** Never.

## Hand-off

**You now have:** `opportunities-roadmap` — new/reshaped modules plus keep/reshape/retire
verdicts on existing rows, awaiting human confirmation.
**Next is usually:** the human confirms → the owner answers the questions or accepts the
work → the project updates its memory/docs/skill/eval artifacts → a follow-up check
measures the delta.
**Unless assessment dominated:** if the run mostly retired/reshaped, the next step is the
customer touch that drains what remains — not another authoring pass.
**Append to engagement state:**

    stage: opportunities
    emitted: opportunities-roadmap -> awaiting confirmation (n new, n reshaped, n retire-proposed)
    open: modules awaiting owner answers; declined items with reasons
    blocked: missing human confirmation, or "nothing"

## When NOT to use this

- **On a calendar.** Triggered only: brain structure change, a batch of answers landing,
  an escape tracing to a knowledge gap, or an audit run completing. `OPERATIONS.md`:
  *"tuned on triggers. Never on a calendar."*
- **When the roadmap is full and untouched.** Run Phase 1 alone, or nothing. The blocker is
  drainage, not supply.
- **To report our own system failures.** Dividing rule — that's observability.
- **To re-rank for its own sake.** Ranking churn with no new evidence is noise.

## Method changelog

- **v0.2** (2026-08-10) — external-share portability pass. The default output is now a
  Markdown opportunities roadmap (`roadmap.md` / `opportunities.md`) that works in plain
  Claude Code. Added the scan-first rule: inspect repo/runtime evidence and ask only for
  unresolved owner decisions or missing facts. Added one-question-at-a-time Q&A handling
  from the product skill pattern.
- **v0.1** (2026-08-07) — first written form. **Run count: 0 — but NOT evidence-free.**
  Codified from one full hand-run of the loop this skill automates: the 7 pilot-customer
  modules (authored 2026-08-04, live in prod), the 2026-08-03 customer report that
  rendered them, and the session that audited both. The one rule, the module
  grouping, the subquestion shape, and the evidence-source list are all lifted from those
  artifacts, not invented. The dividing rule and the derived-vs-authored rule come from
  the same session's review of ten candidate input sources. Deliberately deferred, to be
  earned from real runs rather than decided now: the priority formula, the
  absence-state taxonomy, the customer/operator swimlane, and whether active
  modules need a bound beyond grouping — the owner's call is grouping-over-cap, and the
  first drained round tests it. Expect Phase 1's assessment verdicts and Phase 3's
  recurring-question policy to move most; neither has survived contact with a customer.
